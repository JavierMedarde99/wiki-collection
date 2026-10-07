# Spec: Mazos Commander (Decks)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** Deck (domain/model/Deck.java)

---

## Dominio

### Entidad Deck

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| name | String | ✅ | Nombre del mazo |
| description | String | ❌ | Descripción del mazo |
| commander | String | ❌ | Nombre del comandante |
| commanderColors | List<String> | ❌ | Colores del comandante |
| cards | List<DeckCard> | ❌ | Cartas del mazo (subdocumentos embebidos) |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Entidad DeckCard (subdocumento embebido)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| cardName | String | Nombre de la carta |
| quantity | Integer | Cantidad de copias |
| inCollection | Boolean | Si está en la colección Magic |
| isProxy | Boolean | Si es un proxy |
| manaCost | String | Coste de maná |
| typeLine | String | Línea de tipo |
| colorIdentity | List<String> | Identidad de color |
| imageUrl | String | URL de imagen |
| scryfallId | String | ID en Scryfall |

### Enum

**DeckStatus:** `DRAFT`, `COMPLETE`, `INVALID`

**DeckImportFormat (Fase 23):** `TXT`, `JSON`, `CSV` — si no se declara, se deduce del contenido

**DeckImportMode (Fase 23):** `REPLACE` (por defecto, el mazo queda con las cartas del archivo), `MERGE` (se suman cantidades a las ya existentes)

**DeckImportStatus (Fase 23):** `PENDING`, `RUNNING`, `COMPLETED`, `FAILED`

> **Fase 23 — importación asíncrona:** resolver nombres contra Scryfall tarda, así que `POST /{id}/imports` responde **202** con la URL de un job en memoria (`GET /{id}/imports/{jobId}`). Fases del job: `PARSING → RESOLVING → SAVING → DONE`. El mazo solo se devuelve con `COMPLETED`; `FAILED` garantiza el mazo intacto (se valida antes de guardar).

### DeckStatusReport (record de validación)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| status | DeckStatus | Estado del mazo |
| reasons | List<String> | Razones de invalidación (si any) |

### Reglas de Negocio (Commander)

- **DeckValidator** evalúa el estado del mazo:
  - `DRAFT` — mazo en construcción (requisitos mínimos no cumplidos)
  - `COMPLETE` — mazo cumple todas las reglas Commander
  - `INVALID` — mazo incumple reglas (con reasons explicativas)
- **Reglas validadas:** número de cartas (100 + comandante = 101), identidad de color del comandante, etc.
- **AD-014:** Mazos Commander con validación de reglas

---

## Puertos (Interfaces de Dominio)

### DeckUseCase (in)
```java
public interface DeckUseCase {
    Deck save(Deck deck, String ownerId);
    Deck update(String id, Deck updates, String ownerId);
    void delete(String id, String ownerId);
    Deck findById(String id);
    Page<Deck> search(DeckSearchCriteria criteria, Pageable pageable);
    Deck addCard(String deckId, String scryfallId, int quantity, String ownerId);
    void removeCard(String deckId, String scryfallId, String ownerId);
}
```

### DeckSearchUseCase (in)
```java
public interface DeckSearchUseCase {
    List<MagicCardSearchResult> searchCommanders(String colors);
}
```

### DeckImportUseCase (in) — Fase 23
```java
public interface DeckImportUseCase {
    DeckImportJob startImport(String deckId, String content, DeckImportFormat format, DeckImportMode mode, String ownerId);
    DeckImportJob findJob(String deckId, String jobId, String ownerId);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `DeckService` | CRUD de mazos + gestión de cartas (add/remove) |
| `DeckSearchService` | Búsqueda de comandantes en Scryfall |
| `DeckValidator` | Validación de reglas Commander (DRAFT/COMPLETE/INVALID) |
| `DeckImportService` | Fase 23: parseo, resolución de nombres y guardado de la importación |
| `DeckImportWorker` + `DeckImportJobStore` | Fase 23: jobs en memoria y ejecución en segundo plano |

---

## API Externa: Scryfall (Comandantes)

- **Base URL:** `https://api.scryfall.com`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Estado:** Activa

| Uso | Endpoint |
|-----|----------|
| Buscar comandantes por colores | `GET /cards/search?q=is:commander+t:{colors}` |
| Buscar por nombre | `GET /cards/named?fuzzy={name}` |

---

## Controlador REST

**Base:** `/api/v1/decks`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/decks` | Listar mazos (filtro por nombre opcional) |
| GET | `/api/v1/decks/{id}` | Obtener mazo por ID |
| POST | `/api/v1/decks` | Crear mazo (auth requerido) |
| PUT | `/api/v1/decks/{id}` | Actualizar mazo (auth requerido, ownership verificado) |
| DELETE | `/api/v1/decks/{id}` | Eliminar mazo (204 No Content, auth requerido) |
| POST | `/api/v1/decks/{id}/cards` | Añadir carta al mazo desde Scryfall |
| DELETE | `/api/v1/decks/{id}/cards/{scryfallId}` | Quitar carta del mazo |
| GET | `/api/v1/decks/{id}/status` | Estado del mazo (DRAFT/COMPLETE/INVALID) |
| POST | `/api/v1/decks/{id}/imports` | Importar lista (multipart, responde 202) — Fase 23 |
| POST | `/api/v1/decks/{id}/imports/text` | Importar lista desde texto plano (202) — Fase 23 |
| GET | `/api/v1/decks/{id}/imports/{jobId}` | Estado de la importación — Fase 23 |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Filtrar por nombre exacto |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |

---

## DTOs

### DeckRequest (POST/PUT)

```json
{
  "name": "string (obligatorio)",
  "description": "string",
  "commander": "string",
  "commanderColors": ["string"]
}
```

### DeckResponse (GET)

```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "commander": "string",
  "commanderColors": ["string"],
  "cards": [
    {
      "cardName": "string",
      "quantity": "integer",
      "inCollection": "boolean",
      "isProxy": "boolean",
      "manaCost": "string",
      "typeLine": "string",
      "colorIdentity": ["string"],
      "imageUrl": "string",
      "scryfallId": "string"
    }
  ],
  "createdAt": "datetime",
  "updatedAt": "datetime",
  "ownerId": "string"
}
```

### DeckCardRequest (POST /{id}/cards)

```json
{
  "scryfallId": "string (obligatorio)",
  "quantity": "integer (min 1)"
}
```

### DeckStatusResponse (GET /{id}/status)

```json
{
  "status": "DRAFT | COMPLETE | INVALID",
  "message": "string"
}
```

### DeckImportAcceptedResponse (POST /{id}/imports y /imports/text → 202)

```json
{
  "jobId": "string",
  "status": "PENDING",
  "statusUrl": "/api/v1/decks/{id}/imports/{jobId}"
}
```

Parámetros: `format` (TXT|JSON|CSV, opcional — si se omite se deduce) y `mode` (replace por defecto | merge).

### DeckImportJobResponse (GET /{id}/imports/{jobId})

```json
{
  "jobId": "string",
  "status": "PENDING | RUNNING | COMPLETED | FAILED",
  "phase": "PARSING | RESOLVING | SAVING | DONE",
  "deck": "DeckResponse (solo cuando status=COMPLETED; null mientras corre)",
  "commander": "string",
  "commanderColors": ["w"],
  "unresolved": [
    {
      "line": 12,
      "raw": "1 Lightning Boltt",
      "quantity": 1,
      "name": "Lightning Boltt",
      "reason": "string",
      "candidates": ["..."]
    }
  ],
  "validation": { "status": "DRAFT | COMPLETE | INVALID", "reasons": ["string"] },
  "error": "string | null",
  "progress": { "total": 99, "processed": 40, "resolved": 38, "sideboardIgnored": 2 },
  "createdAt": "instant",
  "updatedAt": "instant",
  "completedAt": "instant | null"
}
```

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `DeckNotFoundException` | 404 | Mazo no encontrado |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `DeckServiceTest` | Unitario | CRUD de mazos |
| `DeckControllerTest` | Integración | Endpoints de mazos |
| `DeckSearchServiceTest` | Unitario | Búsqueda de comandantes |
| `DeckValidationTest` | Unitario | Validación de mazos |
| `DeckPersistenceAdapterTest` | Integración | Persistencia de mazos |
| `DeckDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
| `DeckModelTest` | Unitario | Modelo de dominio Deck |
| `DeckServiceOwnerFilterTest` | Unitario | Filtro owner en decks (Fase 9) |
| `DeckImportControllerTest` | Integración | Endpoints de importación (Fase 23) |
| `DeckImportServiceTest` | Unitario | Servicio de importación con validaciones (Fase 23) |
| `DeckImportWorkerTest` | Unitario | Worker en segundo plano (Fase 23) |
| `DeckImportJobStoreTest` | Unitario | Registro de jobs en memoria (Fase 23) |
| `DeckImportFormatDetectorTest` | Unitario | Detección de formato TXT/JSON/CSV (Fase 23) |
| `DeckListParserRegistryTest` | Unitario | Registro de parsers (Fase 23) |
| `DeckNameNormalizerTest` | Unitario | Normalización de nombres (Fase 23) |
| `MtgoTextDeckListParserTest` | Unitario | Parser de listas MTGO (Fase 23) |
| `JsonDeckListParserTest` | Unitario | Parser JSON (Fase 23) |
| `JsonDeckListParserContextTest` | Unitario | Parser JSON con contexto (Fase 23) |
| `CsvDeckListParserTest` | Unitario | Parser CSV (Fase 23) |
| `DeckCardFactoryTest` | Unitario | Fábrica de DeckCard (Fase 23) |
| `DeckCacheInvalidatorTest` | Unitario | Invalidación de caché al CRUD de mazos |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/decks` devuelve lista de mazos
- [x] El endpoint `GET /api/v1/decks/{id}` devuelve el mazo completo con cartas
- [x] Se puede crear un mazo vía `POST /api/v1/decks` (auth requerido)
- [x] Se puede actualizar/eliminar un mazo vía `PUT`/`DELETE /api/v1/decks/{id}` (auth + ownership)
- [x] Se puede añadir carta al mazo vía `POST /api/v1/decks/{id}/cards` (auth requerido)
- [x] Se puede quitar carta vía `DELETE /api/v1/decks/{id}/cards/{scryfallId}` (auth requerido)
- [x] El endpoint `GET /api/v1/decks/{id}/status` devuelve DRAFT/COMPLETE/INVALID con razones
- [x] Importación asíncrona: `POST /{id}/imports` responde 202 y el job se consulta con `GET /{id}/imports/{jobId}` (Fase 23)
- [x] Formatos TXT/JSON/CSV con detección automática y modos replace/merge (Fase 23)
- [x] `FAILED` deja el mazo intacto y `unresolved` lista las cartas con candidatos (Fase 23)
- [x] El DeckValidator evalúa las reglas Commander correctamente
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)

---

## Estado de Implementación

Fase 4.1 completada. Fase 23 (importación de mazos) completada — PR #453 backend, PR #528 frontend. Todos los componentes implementados:
- ✅ Domain: Deck.java, DeckCard.java, DeckStatus.java, DeckStatusReport.java, DeckImportFormat/Mode/Status
- ✅ Ports: DeckUseCase.java, DeckSearchUseCase.java, DeckImportUseCase.java, DeckRepository.java
- ✅ Application: DeckService.java, DeckSearchService.java, DeckValidator.java, DeckImportService.java, DeckImportWorker.java, DeckImportJobStore.java, parsers (MTGO/JSON/CSV)
- ✅ Infrastructure: DeckController.java (CRUD + imports), DeckEntity.java, DeckCardEntity.java, DeckPersistenceAdapter.java, SpringDataDeckRepository.java, DeckDtoMapper.java, DeckImportDtoMapper.java, respuestas de importación
- ✅ Frontend: DeckImportDialog.tsx (botón "Importar mazo" en el detalle)
- ✅ Tests: 21 archivos de test
