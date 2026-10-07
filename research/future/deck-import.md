# Futuro: Importación de Mazos Existentes (Deck Import)

**Fase:** 23
**Estado:** ✅ Completada — backend PR #453 (2026-10-06, issue #353) + frontend PR #528 (2026-10-07, issue #462)

## Descripción

Permitir importar mazos Commander existentes desde fuentes externas, para que el usuario no tenga que añadir cartas una a una.

### Fuentes soportadas (implementadas)
- **TXT (listas MTGO):** `4 Lightning Bolt` por línea, con BOM ignorado
- **JSON:** arrays de entradas de mazo
- **CSV:** con detección automática de formato si no se declara (`format=TXT|JSON|CSV`)

## Capacidades afectadas

- `decks` — Creación de mazos desde datos externos
- `magic-cards` — Posible integración con buscador de cartas por nombre

## APIs necesarias

- **Scryfall API** — ya integrada (búsqueda de cartas por nombre; `search?q=name:{nombre}`)
- Posible integración con **Moxfield API** o **Deckbox API** si existen y son públicas.

## Estado actual del código (implementado)

**Backend** (`DeckImportUseCase`, `DeckImportService`, `DeckImportWorker`):
- `POST /api/v1/decks/{id}/imports` — multipart `file`, params `format` (opcional) y `mode` (`replace` por defecto | `merge`). Responde **202** con `Location` y `{jobId, status, statusUrl}`.
- `POST /api/v1/decks/{id}/imports/text` — variante con la lista en el cuerpo (`text/plain`).
- `GET /api/v1/decks/{id}/imports/{jobId}` — estado del job: `{status, phase, deck, commander, commanderColors, unresolved, validation, error, progress, createdAt, updatedAt, completedAt}`. El `deck` solo viene cuando `status=COMPLETED`.
- Estados: `PENDING → RUNNING → COMPLETED | FAILED`; fases: `PARSING → RESOLVING → SAVING → DONE`.
- Jobs en memoria (registro de jobs + executor con worker en segundo plano); **429** si la cola está llena.
- `FAILED` significa mazo intacto: se valida antes de guardar.
- Cartas sin resolver: `{line, raw, quantity, name, reason, candidates[]}` — los candidatos son la respuesta a la ambigüedad de nombres.
- Al completar, el mazo pasa por el mismo `DeckValidator` que uno manual: `validation = {status, reasons}`.

**Frontend:** `DeckImportDialog.tsx` con botón "Importar mazo" en el detalle del mazo; `importDeckText`, `importDeckFile` y `getDeckImportJob` en `deckApi.ts`.

**Tests:** 16 archivos nuevos en backend (parsers, job store, worker, service, controller, normalizador) + `DeckImportDialog.test.tsx` y tests de API en frontend.

## Consideraciones de diseño

- **Importación por nombre de carta:** dado que Scryfall soporta búsqueda por nombre con fuzzy search, la importación por lista de nombres es factible. Pero hay cartas con nombres duplicados o similares (ej: "Lightning Bolt" vs "Shock"). El importador debe permitir al usuario confirmar o elegir entre resultados ambiguos.
- **Identidad de color del comandante:** el importador debe inferir los colores del comandante a partir de los datos importados.
- **Validación de reglas:** después de la importación, el mazo debe pasar por la misma validación que un mazo creado manualmente (ver `DeckValidator`).
- **Ritmo de importación:** si el mazo tiene 200 cartas, y cada carta requiere una búsqueda en Scryfall, el importador podría tardar varios segundos. Podríamos usar batch search si Scryfall lo permite (Scryfall soporta búsquedas con `q=name:{nombre1} OR name:{nombre2} OR ...` pero tiene límites de longitud de query).

## Prioridad: Media

Dependiendo del interés del usuario en usar otros servicios de construcción de mazos.

---

*Ver también: fase 4.1 (Mazos Commander).*