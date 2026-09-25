# Futuro: Seguimiento de Lectura (Reading Progress)

**Fase:** 24
**Estado:** ✅ Implementado (parcial) · 📋 Pendiente (sesiones, objetivos, series)

## Lo que ya está implementado

### Seguimiento de lectura básico — ✅ Completo

**Backend — `Book.java`:**
- `Integer pagesRead` — páginas leídas
- `LocalDate startDate` — fecha de inicio de lectura
- `LocalDate endDate` — fecha de finalización
- `Integer start` (0-5) — valoración personal (1-5 estrellas)
- `String comment` — notas al finalizar

**Backend — Endpoints:**
- `PATCH /api/v1/books/{id}/progress` — actualiza solo `pagesRead` sin tocar el resto del libro
- `BookController.java`: `updateProgress()` → `book.setPagesRead(request.pagesRead())` + `bookUseCase.update()`
- `ProgressUpdateRequest.java` — DTO con único campo `pagesRead`

**Backend — Validación:**
- `DateRangeValidator.java`: valida que `endDate` no sea anterior a `startDate`

**Frontend — Componentes:**
- `components/ReadingProgressBar.tsx`: barra visual de progreso. Solo visible para `state === READING` y cuando hay `pages` definidos. Muestra porcentaje, páginas restantes. Editable (botón que abre modal para actualizar `pagesRead`).
- `hooks/useReadingProgress.ts`: cálculo derivado — `percent`, `pagesLeft`, `isVisible`, `read`, `total`. Visible solo en estado READING con pages > 0.
- `components/StarRating.tsx`: componente reutilizable de 1-5 estrellas usado en `BookIsbnScan` y detalles.

**Frontend — Flujo de creación:**
- `components/BookIsbnScan.tsx`: al seleccionar un libro de Google Books, el formulario captura `startDate`, `endDate`, `start` (rating), `comment`, `pagesRead` según el estado elegido:
  - `TO_READ`: no muestra startDate/endDate/rating
  - `READING`: muestra startDate (obligatorio), oculta endDate
  - `COMPLETED`: muestra startDate + endDate + rating (1-5) + comment

**Tests:** `ReadingProgressBar.test.tsx`, `useReadingProgress.test.tsx`, `BookDetailProgress.test.tsx`

### Lo que queda por implementar

#### Historial de sesiones de lectura (`ReadingSession`) — 📋 Pendiente

El documento original proponía registrar sesiones individuales de lectura:
```json
{
  "date": "2026-01-20",
  "pagesRead": 30,
  "durationMinutes": 45
}
```
Esto requeriría una nueva entidad `ReadingSession` con su repositorio y endpoint CRUD. Actualmente no existe.

#### Objetivos de lectura — 📋 Pendiente

El usuario se propondría leer X libros en un año y la app mostraría el progreso hacia esa meta. Requiere una nueva entidad `ReadingGoal` (año, objetivo, progreso actual).

#### Series y colecciones de libros — 📋 Pendiente

Agrupar libros por serie (ej: "Harry Potter" → 7 libros) y ver el progreso de la serie completa. Requeriría un campo `series` en `Book` (nombre + posición) o una entidad `Series` separada.

#### "Libros leídos en 2026" — 📋 Pendiente

Vista/filtro que muestre todos los libros con `state = COMPLETED` y `endDate` en un año dado. No es una entidad nueva, sino un filtro/página adicional que sí se podría implementar con los campos existentes (`endDate`).

## Estado actual de investigación

El documento original asumía que el seguimiento de lectura no estaba implementado. En realidad, los campos `pagesRead`, `startDate`, `endDate` y la barra de progreso visual **sí están implementados**. El documento queda pendiente solo para: historial de sesiones, objetivos anuales, series, y vista de "libros leídos en período".

## Consideraciones de diseño

- `pagesRead`, `startDate`, `endDate` ya existen en `Book`. Añadir `ReadingSession` es una entidad complementaria, no un cambio en `Book`.
- `endDate` ya permite filtrar "libros leídos en 2026" sin cambios adicionales — solo falta la UI/filtro.
- Los objetivos de lectura y las series son features independientes que no bloquean el resto.

## Prioridad

- **Historial de sesiones:** Baja
- **Objetivos anuales:** Baja
- **Series:** Media (si el usuario tiene muchos libros de series)
- **Vista "libros leídos en período":** Baja (solo filtro, sin nueva entidad)

---

*Ver también: fase 1 (Libros), fase 29 (Wishlist + fecha adquisición).*