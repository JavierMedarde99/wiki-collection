# Futuro — Importación de mazos Magic por texto

## Estado: Investigación / No implementado

## 1. Qué es "importar mazo por texto"

El usuario pega un texto representando un mazo completo de Magic: The Gathering
y el sistema lo parsea, busca cada carta en Scryfall, y crea el mazo con todas
sus cartas en una operación.

Esto es extremadamente común en la comunidad Commander: la mayoría de sitios de
deckbuilding (Moxfield, Deckstats, Archidekt, etc.) permiten exportar/importar
en formato texto plano.

### Formato típico de decklist

Varios formatos existen. Los más comunes:

**Formato básico (cantidad + nombre):**
```
4 Cultivate
2 Deep-rooted Haymakers
1 Narset, Parter of Veils
```

**Formato con "x":**
```
4x Cultivate
2x Deep-rooted Haymakers
1x Narset, Parter of Veils
```

**Formato con secciones (main + sideboard):**
```
// Commander
1 Kinnan, B indenture
// Main
4 Cultivate
2 Deep-rooted Haymakers
// Sideboard
2 Splinter Twin
```

**Formato con comentarios:**
```
4 Cultivate // Tierra boscosa
2 Deep-rooted Haymakers
1 Narset, Parter of Veils // Comandante
```

### ¿De dónde viene el texto?

El usuario puede:
- Copiar un decklist desde una web (Moxfield, Deckstats, etc.) y pegarlo
- Escribirlo manualmente línea por línea
- Importar desde un archivo .txt subido
- Pegar el texto directamente en un textarea

---

## 2. Análisis de viabilidad en el backend actual

El backend ya tiene:

- `ScryfallClient` — busca cartas por nombre (fuzzy) o por nombre exacto
- `DeckService` — crea/actualiza mazos
- `DeckCard` — entidad que representa una carta en un mazo (con cantidad)
- `DeckValidator` — valida las reglas Commander

**Lo que falta:** un servicio que tome un texto arbitrario, lo parseé, resuelva
cada nombre de carta a un `scryfallId`, y construya el mazo.

### 2.1 Flujo propuesto (backend)

```
1. Cliente → POST /api/v1/decks/import
   Body: {
     "name": "Mi Mazo",
     "description": "...",
     "decklistText": "4 Cultivate\n2 Deep-rooted Haymakers\n..."
   }

2. Backend → DeckImportService.parseDecklist(decklistText)
   → Devuelve lista de { name: string, quantity: number, isCommander: boolean }
   (líneas vacías, comentarios y secciones se ignoran)

3. Para cada carta única (nombre único):
   → ScryfallClient.searchByName(name)  // fuzzy search
   → Si no encuentra → ScryfallClient.searchExact(name)
   → Si no encuentra → registrar fallo, continuar con las demás

4. Backend → DeckService.create(deckDto) + DeckService.addCards(ids, quantities)
   → Crea el mazo
   → Añade las cartas con sus cantidades

5. Backend → DeckValidator.evaluate(deck)
   → Devuelve status (DRAFT/INCOMPLETE/COMPLETE/INVALID)

6. Backend → Devuelve DeckImportResult {
     deck: DeckResponse,
     cardsAdded: 98,
     cardsFailed: [
       { line: "2 SomeObscureCard", reason: "No encontrado en Scryfall" }
     ],
     status: "COMPLETE"  // o DRAFT/INCOMPLETE si faltan cartas
   }
```

### 2.2 Parsing del decklist

El parser es una utilidad pura (sin dependencias externas) que:

1. Divide el texto por líneas
2. Ignora líneas vacías
3. Ignora comentarios (todo lo que está después de `//` o entre `[ ]` o `( )`
   dependiendo del formato)
4. Extrae cantidad + nombre de cada línea con regex
5. Detecta si la línea es el comandante (convención: primera línea non-empty,
   o línea marcada explícitamente con "Commander:", o carta única con cantidad 1
   al inicio del mazo)
6. Agrupa cartas por nombre (si aparece "4 Cultivate" y luego "1 Cultivate",
   se suman → 5 Cultivate)

Regex sugerido para línea de carta:
```java
Pattern.compile("^\\s*(\\d+)\\s*(?:x|X)?\\s*(.+?)(?:\\s*[/].*)?$")
```

Esto captura:
- `4 Cultivate` → qty=4, name="Cultivate"
- `2x Black Lotus` → qty=2, name="Black Lotus"
- `1 Narset, Parter of Veils // Comandante` → qty=1, name="Narset, Parter of Veils"

### 2.3 Manejo de nombres de cartas ambiguas

Scryfall tiene métodos para buscar por nombre:
- `GET /cards/named?exact={name}` — nombre exacto (prefijo de búsqueda)
- `GET /cards/named?fuzzy={name}` — fuzzy (tolerante a errores de ortografía)
- `GET /cards/search?q={name}` — búsqueda avanzada

Estrategia de resolución:

1. Intentar `exact` primero (el nombre del decklist suele ser correcto)
2. Si no hay resultados, intentar `fuzzy`
3. Si siguen sin resultados, registrar como fallo
4. Si hay múltiples resultados (ej "Black Lotus" existe en varios sets),
   tomar el primero (o el más reciente por date)

**Carta no encontrada:** No es raro que un decklist tenga cartas muy específicas
(ediciones especiales, cartas muy nuevas, etc.) que Scryfall no tiene indexadas
aún, o nombres con errores tipográficos. El sistema debe ser resistente: si una
carta falla, las demás siguen siendo añadidas al mazo. Las fallidas se reportan
al usuario para que las corrija manualmente.

### 2.4 Rate limit de Scryfall — Estrategia asíncrona recomendada

Scryfall tiene un rate limit de ~10 requests/segundo (recomendado).
Un mazo típico tiene entre 30 y 50 cartas únicas (muchas se repiten).
50 requests a 10 req/s = 5 segundos mínimos si es secuencial.

Existen varias estrategias. La **recomendada** es paralelismo controlado:

#### Estrategia recomendada: Pool de hilos con Semaphore

Se hace paralelismo limitado en el mismo request HTTP. Una pool de 3-5 hilos
permite hacer varias búsquedas a la vez sin exceder el rate limit.

**Ventajas:**
- Simple — no requiere infraestructura adicional
- Las búsquedas son concurrentes pero controladas
- El tiempo de respuesta baja de ~10s a ~2-3s para 50 cartas únicas
- No hay estado persistente ni colas

**Implementación con Spring:**

```java
@Service
public class DeckImportService {

    private final ScryfallClient scryfallClient;
    private final DeckService deckService;
    private final DeckValidator validator;
    private final Semaphore scryfallSemaphore = new Semaphore(4); // 4 concurrentes

    public DeckImportResult importDeck(DeckImportRequest request) {
        List<ParsedLine> parsed = parseDecklist(request.decklistText());

        // Resolver cartas en paralelo (limitado a 4 concurrentes)
        List<CardResolution> resolved = new ArrayList<>();
        List<FailedCard> failed = new ArrayList<>();

        try (ExecutorService executor = Executors.newFixedThreadPool(4)) {
            List<Future<CardResolution>> futures = parsed.stream()
                .map(line -> executor.submit(() -> {
                    scryfallSemaphore.acquire();
                    try {
                        return resolveCard(line);
                    } finally {
                        scryfallSemaphore.release();
                    }
                }))
                .toList();

            for (Future<CardResolution> f : futures) {
                try {
                    resolved.add(f.get());
                } catch (ExecutionException e) {
                    failed.add(new FailedCard(...));
                }
            }
        }

        // Crear mazo + añadir cartas
        Deck deck = deckService.create(...);
        deck = deckService.addCards(deck.getId(), resolved);

        // Validar
        DeckStatus status = validator.evaluate(deck);

        return new DeckImportResult(..., resolved, failed, status);
    }

    private CardResolution resolveCard(ParsedLine line) {
        // 1. exact search
        // 2. fuzzy search si falla
        // 3. retornar o fallar
    }
}
```

Explicación de valores:
- `Semaphore(4)` → máximo 4 requests concurrentes a Scryfall
- 4 concurrentes = ~8-10 req/s efectivos (cada request tarda ~400ms)
- Queda por debajo del límite de 10 req/s recomendado por Scryfall
- Pool de 4 threads → suficiente para 50 cartas únicas

**Manejo de 429 (Too Many Requests):**
Aunque es poco probable con 4 concurrentes, si llega a ocurrir, hacer backoff
exponencial antes de reintentar.

```java
int maxRetries = 3;
for (int attempt = 0; attempt < maxRetries; attempt++) {
    try {
        return scryfallClient.searchByName(name);
    } catch (RateLimitException e) {
        long waitMs = (long) Math.pow(2, attempt) * 1000; // 1s, 2s, 4s
        Thread.sleep(waitMs);
    }
}
```

**Alternativas menos recomendadas:**
- **Secuencial con delay:** Simple pero lento (50 cartas × 200ms = 10s). Solo
  justificado si el rate limit de Scryfall fuera mucho más estricto.
- **Job asíncrono con ID:**má más robusto para importaciones muy grandes pero
  requiere storage de jobs (MongoDB, Redis) y polling por parte del frontend.
  Sobrekill para el caso de uso actual.

### 2.5 Validación del mazo post-import

Tras añadir las cartas, `DeckValidator` evalúa el mazo:
- ¿100 cartas exactas? (o menos si está en borrador)
- ¿Comandante presente?
- ¿Comandante dentro de la identidad de color?
- ¿Cartas dentro de los colores del comandante?

Si el mazo importado no tiene comandante (se perdió en el parsing), el status será
`DRAFT`. El usuario puede añadir el comandante manualmente después.

---

## 3. Endpoints propuestos

### 3.1 Import rápido de mazo (crear mazo nuevo)

```
POST /api/v1/decks/import
Content-Type: application/json

{
  "name": "Mi Mazo 시험을",
  "description": "Un decklist importado",
  "decklistText": "4 Cultivate\n2 Deep-rooted Haymakers\n1 Narset, Parter of Veils"
}
```

Response (201):
```json
{
  "id": "abc123",
  "name": "Mi Mazo 시험을",
  "description": "Un decklist importado",
  "commander": "Narset, Parter of Veils",
  "cards": [
    { "scryfallId": "...", "cardName": "Cultivate", "quantity": 4 },
    { "scryfallId": "...", "cardName": "Deep-rooted Haymakers", "quantity": 2 },
    { "scryfallId": "...", "cardName": "Narset, Parter of Veils", "quantity": 1 }
  ],
  "status": "COMPLETE",
  "importResult": {
    "cardsAdded": 3,
    "cardsFailed": [],
    "warnings": ["El mazo tiene 7 cartas, necesitas 99 más para Commander"]
  }
}
```

### 3.2 Import a un mazo existente (reemplaza cartas)

```
POST /api/v1/decks/{id}/import
Content-Type: application/json

{
  "decklistText": "4 Cultivate\n2 Deep-rooted Haymakers"
}
```

Response (200):
```json
{
  "id": "abc123",
  "name": "MazoTést",
  "cards": [...],
  "importResult": {
    "cardsAdded": 2,
    "cardsFailed": [],
    "cardsRemoved": 10  // vaciado anterior
  }
}
```

### 3.3 Validar decklist sin crear mazo

```
POST /api/v1/decks/validate-decklist
Content-Type: application/json

{
  "decklistText": "4 Cultivate\n2 Deep-rooted Haymakers"
}
```

Response (200):
```json
{
  "parsed": [
    { "name": "Cultivate", "quantity": 4 },
    { "name": "Deep-rooted Haymakers", "quantity": 2 }
  ],
  "resolved": [
    { "name": "Cultivate", "scryfallId": "...", "quantity": 4 },
    { "name": "Deep-rooted Haymakers", "scryfallId": "...", "quantity": 2 }
  ],
  "failed": [
    { "line": "2 DoesNotExist", "reason": "No encontrado en Scryfall" }
  ],
  "summary": {
    "totalCards": 6,
    "uniqueCards": 2,
    "missingCommander": true
  }
}
```

Este endpoint es útil para verificación previa antes de crear el mazo.

---

## 4. Frontend — Componente de importación

### 4.1 Componente `DeckImportModal.tsx`

```
┌─────────────────────────────────┐
│  Importar mazo desde texto  [✕]│
│─────────────────────────────────│
│                                 │
│  Pegá tu decklist aquí:         │
│  ┌───────────────────────────┐  │
│  │ 4 Cultivate              │  │
│  │ 2 Deep-rooted Haymakers   │  │
│  │ 1 Narset, Parter of Veils │  │
│  │                           │  │
│  └───────────────────────────┘  │
│                                 │
│  Nombre del mazo: [          ]  │
│  Descripción:     [          ]  │
│                                 │
│  [Importar mazo]  [Cancelar]   │
│                                 │
│  ⚠️ Ejemplo de formato:         │
│  4 Cultivate                    │
│  2x Black Lotus                 │
│  1 Narset, Parter of Veils      │
│                                 │
└─────────────────────────────────┘
```

### 4.2 Flujo de importación

```
DeckListPage → botón "Importar mazo" → DeckImportModal
  → usuario pega texto
  → click "Importar"
  → POST /api/v1/decks/import
  → spinner mientras Scryfall resuelve cartas (2-5 segundos)
  → si éxito: redirect a DeckDetailPage del mazo creado
  → si fallos: mostrar modal con cartas fallidas (clickeables para buscar
    manualmente en Scryfall)
```

### 4.3 Manejo de errores y cartas fallidas

Si el import tiene cartas que Scryfall no encontró:

```
┌────────────────────────────────────────┐
│  Mazo importado parcialmente            │
│────────────────────────────────────────│
│                                        │
│  3 cartas añadidas, 1 fallida:         │
│                                        │
│  ✗ "2 SomeObscureCard" — no encontrada │
│    [Buscar en Scryfall]  [Ignorar]     │
│                                        │
│  [Ver mazo]  [Importar de nuevo]       │
│                                        │
└────────────────────────────────────────┘
```

El usuario puede:
- Clickear "Buscar en Scryfall" para ir al flujo normal de búsqueda
- "Ignorar" para continuar con el mazo parcial
- "Importar de nuevo" para corregir el texto y reintentar

---

## 5. Resumen de cambios necesarios

### Backend

| Archivo | Cambio |
|---------|--------|
| `DeckImportService.java` (nuevo) | Servicio que parsea decklist + resuelve cartas en paralelo + crea mazo |
| `DeckImportUseCase.java` (nuevo) | Port de entrada para la operación de import |
| `DeckController.java` | Añadir endpoints: POST /decks/import, POST /decks/{id}/import, POST /decks/validate-decklist |
| `DeckImportRequest.java` (nuevo DTO) | `{ name?, description?, decklistText }` |
| `DeckImportResult.java` (nuevo DTO) | `{ cardsAdded, cardsFailed, warnings, deck? }` |
| `DeckParsedLine.java` (nuevo) | Internal: `{ name, quantity, isCommander }` |
| `DeckImportServiceTest.java` | Tests del parser, de resolución paralela, de casos de error |
| `DeckControllerTest.java` | Tests de los nuevos endpoints MockMvc |
| `DeckImportService` → `ScryfallClient` | Añadir métodos `searchByName` (exact + fuzzy) si no existen ya |

### Frontend

| Archivo | Cambio |
|---------|--------|
| `src/components/DeckImportModal.tsx` | NUEVO — modal de importación de mazo |
| `src/pages/DeckListPage.tsx` | Añadir botón "Importar mazo" |
| `src/api/deckApi.ts` | Añadir `importDeck(decklistText)`, `validateDecklist(text)`, `importToDeck(id, text)` |
| `src/types/Deck.ts` | Añadir tipos `DeckImportRequest`, `DeckImportResult` |
| `package.json` | No requiere librerías nuevas (solo textarea + fetch) |

### Frontend — Testing

El componente `DeckImportModal` es un componente stateful con fetch. Se puede
testear con Vitest + React Testing Library mockeando la API.

---

## 6. Forma de importación de texto: idea de UX

El usuario puede pegar el texto de varias formas:

1. **Directo en un textarea** — más sencillo, funciona siempre.
2. **Subir archivo .txt** — `<input type="file" accept=".txt" />` → leer con
   `FileReader.readAsText()` → obtener el contenido. Para usuarios que tienen
   el decklist guardado como archivo.
3. **Copia-desde-web** — el usuario copia desde Moxfield, Deckstats, etc. y pega.

La opción 1 es la más universal y no requiere UI extra. La opción 2 es un
"premium" que se puede añadir después si hay demanda.

---

## 7. Riesgos y consideraciones

**Nombres con caractéres especiales:** Algunas cartas tienen comas, dos puntos,
o nombres con caracteres no-ASCII. El parser debe ser robusto.

**Commander ambiguo:** Si el decklist no marca explícitamente el comandante,
el sistema puede asumir que es la primera carta con quantity=1 que aparece, o
preguntar al usuario. Mejor dejar el mazo en estado DRAFT y pedir al usuario
que elija el comandante manualmente.

**Scryfall puede no encontrar:** Dependiendo de la ortografía del nombre, Scryfall
puede no encontrar la carta. La búsqueda fuzzy ayuda, pero no es infalible.

**Coste de las llamadas a Scryfall:** 50 cartas únicas → 50 peticiones HTTP
con paralelismo de 4 → ~2-3 segundos. Aceptable para uso normal. Si el usuario
importa 100 mazos seguidos, puede toser rate limit. Implementar backoff simple
si es necesario.

**Mazo vacío o casi vacío:** Si el decklist está vacío o casi, el sistema debe
validarlo y avisar antes de crear un mazo vacío.

---

## 8. Relación con otras funcionalidades futuras

Este feature es independiente del escáner de barcodes para libros
(docs/22-futuro-barcode-scanner.md). Se puede implementar en cualquier orden.
Ambas son funcionalidades de "importación rápida" para diferentes colecciones
(books → barcode ISBN, decks → texto pegado).
