# Futuro: Series y agrupación de libros (Book Series)

**Fase:** 24 (sub-feature de "Historial de sesiones, objetivos y series")
**Estado:** 📋 Investigación completada · sin implementar
**Issues:** backend #425 · frontend #488
**Fecha de investigación:** 2026-10-02

## Descripción

Agrupar libros en series para poder ver el progreso agregado de una serie completa. Caso canónico: "Harry Potter" → 7 libros, cada uno con su posición, y una vista que indique cuántos se han leído.

El modelo es de cardinalidad **0..1**: cada libro pertenece como mucho a una serie.

### ¿Manual o automático?

**Conclusión de la investigación: manual.** El autocompletado desde la API externa no es viable en la práctica. Ver "APIs externas necesarias".

## APIs externas necesarias

Ninguna nueva. Se investigaron Google Books (ya integrada) y Open Library (no integrada, descartada).

### Google Books — `volumeInfo.seriesInfo` existe, pero no sirve

El esquema oficial de la API **sí incluye** `volumeInfo.seriesInfo`:

```json
"seriesInfo": {
  "bookDisplayNumber": "A String",   // solo para display; el orden real está en orderNumber
  "shortSeriesBookTitle": "A String",
  "volumeSeries": [
    {
      "issue": [ { "issueDisplayNumber": "...", "issueOrderNumber": 42 } ],
      "orderNumber": 42,              // posición en la serie
      "seriesBookType": "Single Issue, Collection Edition, etc.",
      "seriesId": "A String"          // id opaco
    }
  ]
}
```

Tres bloqueos verificados:

1. **Solo está en `volumes.get`, no en `volumes.list`.** El parámetro `includeNonComicsSeries` existe únicamente en `volumes.get`. `GoogleBooksClient.java:68` ya llama a `/v1/volumes/{id}`, así que es alcanzable; la ruta de búsqueda (`volumes.list`, línea 117) no lo obtiene.
2. **Requiere `includeNonComicsSeries=true`** (por defecto `false`). Sin ese parámetro solo se obtienen series de cómics, no de novelas.
3. **No hay nombre de serie.** Los campos disponibles son `seriesId` (opaco), `shortSeriesBookTitle`, `bookDisplayNumber` y `orderNumber`. Nada devuelve el texto "Harry Potter".

> No se pudo verificar la cobertura real en vivo: la máquina devolvió **HTTP 429 "Quota exceeded"** en el proyecto anónimo compartido. Además `google.books.api-key` está vacío por defecto (`GoogleBooksClient.java:34`), por lo que el cliente actual corre sin autenticar.

### Open Library — tiene nombres, cobertura insuficiente

El endpoint de edición (`/books/{key}.json`) sí expone `series` como texto. Muestra medida sobre 20 ediciones de Harry Potter:

| Métrica | Resultado |
|---------|-----------|
| Ediciones con `series` | **5 de 20** (25%) |
| Ediciones en inglés | `null` |
| Único nombre en español/árabe | `"Silsilat Hārī Būtir -- 5"` (corrupto) |

Cobertura del 25% y nombres corruptos descartan Open Library como fuente automática. Además implicaría añadir una API externa nueva sin clave ni cuota gestionada.

## Capacidades afectadas

- `books` — dos campos nuevos en `Book`.
- `specs/books/spec.md` y `specs/books/api-contract.md` — documentar campos, filtros y endpoints.
- Frontend — filtro por serie, vista de detalle de serie y progreso agregado.

### Diseño de datos sugerido (Opción A — recomendada)

```json
// Book (campos nuevos embebidos, sin entidad nueva)
{
  "series": "Harry Potter",
  "seriesKey": "harry potter",   // clave normalizada para agrupar
  "seriesOrder": 1
}
```

**Por qué embebido y no entidad `Series`:**

- MongoDB resuelve el grupo con un `group-by` sobre `seriesKey`, sin join ni colección nueva.
- La cardinalidad es 0..1, no N:M — no justifica una entidad separada.
- No requiere migración: los documentos existentes leen `null`.
- El progreso agregado (`COUNT` de leídos / total) se calcula al vuelo.

**Riesgo principal:** nombres de serie como texto libre producen duplicados (`"Harry Potter"` vs `"harry potter"` vs `"Harry potter "`). Se mitiga con `seriesKey` normalizado para agrupar + `series` como nombre visible. El backend ya tiene el patrón `StringTo*Converter` para conversiones de este tipo.

### Opción B — entidad `Series` separada

```json
{ "id": "...", "name": "Harry Potter", "ownerId": "..." }
```
Book referencia `seriesId`.

**Cuándo migrar:** solo si las series necesitan metadatos propios (portada, descripción, total de tomos, orden canónico) o si se generaliza a colecciones libres tipo estantería (Goodreads), donde un item puede estar en varios grupos.

## Generalización a las demás colecciones

Investigado para responder si conviene preparar el modelo ahora. **El resultado invierte la prioridad: movies es lo más barato, no libros.**

| Colección | ¿Agrupación automática disponible? | Fuente |
|-----------|-----------------------------------|--------|
| **movies/shows** | **Sí — con nombre, fiable** | TMDB `belongs_to_collection` → `{id, name}` (ej: `"The Avengers Collection"`, o `null`) |
| magic-cards | Parcial | Scryfall: los sets tienen `set_type` y `released_at`, pero **no existe campo `series`** (verificado sobre las 17 claves del set) |
| books | Débil | Google Books `seriesInfo`: id + orden, sin nombre |
| games | No | RAWG no expone campo de serie/franquicia |
| board-games | No | BGG XML no expone serie |
| decks | Ya agrupado | Por comandante / identidad de color |

**Consecuencia de diseño:** el modelo de referencia debería ser el de TMDB (`{id, name}`), no el de Google Books. Si se generaliza, conviene guardar `externalGroupId` + `name` + `order` para que `movie-shows` pueda completarlo automáticamente y `books` solo manualmente.

## Consideraciones de diseño

- **Visibilidad (decisión pendiente):** una serie es un grupo de libros, pero cada libro tiene su propia visibilidad vía `owner=mine|other|all` y perfil público. Hay que decidir si una serie aparece en el perfil público de otro usuario y qué se muestra cuando solo algunos de sus libros son públicos.
- **Costes de API:** aunque hoy es manual, si en el futuro se autocompleta desde Google Books, hará falta `volumes.get` por volumen con `includeNonComicsSeries=true` — una llamada extra por libro. Debe pasar por Caffeine (ya existen `bookDetail` y `bookList`).
- **Credenciales:** configurar `google.books.api-key` antes de depender de cualquier enriquecimiento de Google Books; sin clave el proyecto comparte cuota anónima y ya devuelve 429.
- **Filtro vs endpoint:** filtrar por `?series=` reutiliza el `BookSearchCriteria` existente; una vista de detalle de serie puede ser un `group-by` paginado, sin endpoint dedicado.
- **Consistencia de texto:** normalizar `seriesKey` en el punto de escritura (converter), no en lectura.

## Prioridad estimada

**Media.** El coste de implementación es bajo (dos campos), pero la densidad de uso depende de cuántos libros de serie tenga el usuario. Si al final se quiere el modelo general, **empezar por `movie-shows`** tiene mejor retorno: la fuente externa ya entrega el nombre y no requiere entrada manual.

## Alternativa más simple (sin modelo de serie)

Si el objetivo real es solo "ver cuántos libros he leído de esta saga", una opción sin ningún campo nuevo es filtrar por autor (`?author=`) o por `endDate` en un rango. Cubre el caso de un único autor por saga a coste cero. Descartada si se quiere soportar sagas con varios autores o tomos sin autor compartido.

---

*Ver también: fase 1 (Libros), fase 24 (`future/reading-progress.md`), fase 29 (`future/books-wishlist.md`).*

*Esta documentación es informativa y no representa un compromiso de implementación. Las decisiones finales se documentan en `research/decisions/` cuando se priorizan.*