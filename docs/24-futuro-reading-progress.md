# Futuro — Progreso de lectura en libros

## Estado: Investigación / No implementado

## 1. Concepto

El usuario puede registrar cuántas páginas ha leído de cada libro que está en estado `READING`, y la aplicación calcula automáticamente el porcentaje completado y las páginas que faltan.

**Objetivo:** poder decir "estoy en el capítulo 8 de un libro de 320 páginas" y que el sistema muestre "25% completado, 240 páginas restantes".

---

## 2. Modelo de datos

### 2.1 Campo nuevo en `Book`

```
pagesRead | Integer | ❌ | Páginas leídas hasta el momento (0 si no ha empezado)
```

**Colección MongoDB:** `books`, campo `pagesRead`.

**Derivados (calculados, no persistidos):**

| Valor | Fórmula |
|-------|---------|
| Porcentaje | `pagesRead * 100 / pages` (redondeado) |
| Páginas restantes | `pages - pagesRead` |
| ¿Progreso visible? | `state === READING && pages && pages > 0` |

### 2.2 ¿Por qué solo `pagesRead` y no `readingPercentage`?

`pagesRead` es la fuente de verdad. El porcentaje se calcula siempre a partir de `pagesRead / pages`. Guardar ambos introduciría inconsistencia: si el usuario edita `pages` del libro, el porcentaje guardado quedaría obsoleto.

### 2.3 Restricciones de validación

- `pagesRead >= 0`
- `pagesRead <= pages` (si `pages` está definido). Si `pages` es null/0, no se puede calcular porcentaje y se ignora la validación de máximo.
- `pagesRead` solo tiene sentido cuando `state === READING`. Se recomienda (no obligatorio) que el frontend reseteé `pagesRead = 0` al pasar a `TO_READ` y lo ignore al estar en `COMPLETED`.

---

## 3. Backend

### 3.1 Estado actual

El modelo `Book` ya tiene `pages` (número total de páginas) y `state` (TO_READ / READING / COMPLETED). No hay campo de progreso.

### 3.2 Cambios en el dominio

**Archivos afectados:**
- `Book.java` (domain model) → añadir `private Integer pagesRead;`
- `BookEntity.java` (MongoDB entity) → añadir `@Field("pagesRead") private Integer pagesRead;`
- `BookRequest.java` (DTO input) → añadir `private Integer pagesRead;` (opcional, sin `@NotNull`)
- `BookResponse.java` (DTO output) → añadir `private Integer pagesRead;`
- `BookDtoMapper.java` → mapear `pagesRead` en ambas direcciones

**Interfaz `BookUseCase`:** No cambia. El `update()` ya recibe un `Book` con los nuevos valores y los persiste.

**Interfaz `BookRepository`:** No cambia (Spring Data deriva la query automáticamente).

### 3.3 Endpoints

**No es necesario un endpoint nuevo si se usa el PUT existente:**

```
PUT /api/v1/books/{id}
Body:
{
  "pagesRead": 120,
  "comment": "..."   // opcional: otros campos que también quieras actualizar
}
```

**Opcional — Endpoint dedicado para progreso:**

Si se prefiere actualizar solo el progreso sin tocar el resto del documento:

```
PATCH /api/v1/books/{id}/progress
Content-Type: application/json

{ "pagesRead": 120 }
```

Response: `200 OK` con el `BookResponse` actualizado.

**Implementación del endpoint PATCH (si se decide):**

```java
@PatchMapping("/{id}/progress")
public ResponseEntity<BookResponse> updateProgress(
    @PathVariable String id,
    @RequestBody ProgressUpdateRequest request) {
    Book book = bookService.findById(id);
    book.setPagesRead(request.getPagesRead());
    Book updated = bookService.update(id, book, ownerId);
    return ResponseEntity.ok(bookDtoMapper.toResponse(updated));
}
```

```java
// DTO mínimo
public class ProgressUpdateRequest {
    @Min(0)
    private Integer pagesRead;
    public Integer getPagesRead() { return pagesRead; }
    public void setPagesRead(Integer pagesRead) { this.pagesRead = pagesRead; }
}
```

### 3.4 Lógica de negocio

Se recomienda validar en `BookService.update()` o en un validador dedicado:

```java
// Validar que pagesRead no excede pages (si pages está definido)
if (book.getPagesRead() != null && book.getPages() != null && book.getPages() > 0) {
    if (book.getPagesRead() > book.getPages()) {
        throw new InvalidProgressException("Las páginas leídas no pueden exceder el total de páginas");
    }
}
```

**Excepción nueva (si se decide):**
- `InvalidProgressException` → HTTP 400 Bad Request, mensaje "Las páginas leídas (X) no pueden exceder el total (Y)"

### 3.5 Tests necesarios

- `BookServiceTest` → test que `pagesRead` se persiste correctamente en update
- `BookServiceTest` → test que lanza excepción si `pagesRead > pages`
- `BookControllerTest` → test PUT con campo `pagesRead` (MockMvc)
- `BookControllerTest` → test PATCH `/progress` (si se implementa el endpoint dedicado)
- `BookDtoMapperTest` → test de mapeo `pagesRead` domain ↔ response/request

---

## 4. Frontend

### 4.1 Tipos TypeScript

**Archivo:** `src/types/Book.ts`

```typescript
export interface Book {
  id: string;
  externalId?: string;
  title: string;
  descripcion?: string;
  author: string;
  pages?: number;
  pagesRead?: number;   // ← NUEVO
  type: BookType;
  state: BookState;
  comment?: string;
  start?: number;
  startDate?: string;
  endDate?: string;
  frontpage?: string;
  ownerId?: string;
  username?: string;
}
```

### 4.2 Nuevo componente: `ReadingProgressBar`

**Archivo:** `src/components/ReadingProgressBar.tsx`

**Props:**
```typescript
interface ReadingProgressBarProps {
  pages: number | undefined;      // total de páginas
  pagesRead: number | undefined;  // páginas leídas
  state: BookState;               // TO_READ | READING | COMPLETED
  showInput?: boolean;            // si true, muestra input editable
  onChange?: (pagesRead: number) => void;  // callback al cambiar (si editable)
  className?: string;
}
```

**Comportamiento:**

| Condición | Qué muestra |
|-----------|-------------|
| `state !== READING` | Nada (o badge "Sin progreso" / "Finalizado") |
| `pages` undefined o 0 | "Sin información de páginas" |
| `pagesRead === 0` | "No comenzado" + barra vacía al 0% |
| `pagesRead === pages` | "Completado" + barra al 100% (verde) |
| Entre 0 y 100% | Barra con porcentaje + "X de Y páginas (Z% restantes)" |
| `pagesRead > pages` (defensivo) | Mostrar al 100% y stricker el valor |

**Visual (ejemplo Tailwind):**
```tsx
<div className="w-full bg-gray-200 rounded-full h-2.5">
  <div
    className="bg-blue-600 h-2.5 rounded-full transition-all duration-300"
    style={{ width: `${percent}%` }}
  />
</div>
<p className="text-sm mt-1">
  {pagesRead} / {pages} páginas — {percent}% completado
  {pagesLeft > 0 ? ` (${pagesLeft} restantes)` : ''}
</p>
```

### 4.3 Actualización de componentes existentes

**`BookCard.tsx` (listado):**
- Añadir barrita de progreso minimalista cuando `state === READING` y hay `pages` y `pagesRead`.
- Diseño: barra pequeña (h-1.5) debajo del título o dentro del área del badge de estado.
- Texto opcional: "43%" discretamente.

**`BookDetailPage.tsx`:**
- Mostrar `ReadingProgressBar` con `showInput={true}`.
- Input numérico "Páginas leídas" que actualiza el estado local y, al guardar, llama a `PUT /api/v1/books/{id}` con el nuevo `pagesRead`.
- Alternativamente, input con `onChange` inmediato + auto-guardado (debounce 1s).

**`BookForm.tsx` (crear/editar):**
- Añadir campo "Páginas leídas" que:
  - Solo se muestra cuando `state === 'READING'`
  - Es numérico, mínimo 0
  - Al cambiar el `state` a `TO_READ`, resetea `pagesRead` a 0 automáticamente (opcional, decisorio)
- Validación: si `pagesRead > pages`, mostrar error inline.

**`BookCreatePage.tsx`:**
- No mostrar campo `pagesRead` (nuevo libro empieza en 0 páginas leídas).

### 4.4 Hoja de cálculo de progreso (hook opcional)

**Archivo:** `src/hooks/useReadingProgress.ts`

```typescript
export function useReadingProgress(pages?: number, pagesRead?: number, state?: BookState) {
  const total = pages ?? 0;
  const read = pagesRead ?? 0;

  const percent = total > 0 ? Math.round((read / total) * 100) : 0;
  const pagesLeft = total > 0 ? Math.max(0, total - read) : 0;
  const isVisible = state === 'READING' && total > 0;

  return { percent, pagesLeft, isVisible, read, total };
}
```

---

## 5. Flujo de usuario

### 5.1 Escenario principal

```
1. Usuario tiene "El Señor de los Anillos" en estado READING
   - pages: 1178
   - pagesRead: 0

2. Abre BookDetailPage → ve barra al 0% ("0 de 1178 páginas")

3. Edita "Páginas leídas" → pon 350 → guarda

4. Backend: PUT /api/v1/books/{id} con pagesRead=350
   → Valida: 350 <= 1178 ✓
   → Persiste en MongoDB

5. Frontend refresca → barra ahora muestra:
   - ██████████░░░░░░░░░░ 29%
   - "350 de 1178 páginas (828 restantes)"
```

### 5.2 Pasar libro a COMPLETED

```
1. User cambia state de READING → COMPLETED
2. Frontend pregunta: "¿Marcar como completado (100%)?"
   - Opción A: sí → pagesRead = pages automáticamente
   - Opción B: no → conserva pagesRead tal cual (el usuario puede haber
     llegado a 95% y ya no quiera seguir)
3. State pasa a COMPLETED → la barra de progreso desaparece (o muestra
   "Finalizado")
```

### 5.3 Volver a leer

```
1. User tiene libro en COMPLETED con pagesRead=1178
2. Lo pasa de nuevo a READING
3. pagesRead conserva el valor (o se pone a 0, según decisión de UX)
4. Continúa editando desde donde lo dejó
```

---

## 6. Casos límite

### 6.1 Libro sin `pages` definido

Algunos libros (ej: audiolibros, ebooks sin numeración clara, poesía) no tienen número de páginas conocido.

**Comportamiento:**
- No se muestra barra de porcentaje.
- Se muestra únicamente "X páginas leídas" (si `pagesRead > 0`).
- El input de `pagesRead` sigue disponible si el usuario quiere llevar cuenta absoluta.

**Alternativa:** pedir que `pages` sea requerido si `state === READING`. Esto es más estricto pero simplifica la lógica. No se recomienda por defecto.

### 6.2 `pagesRead` mayor que `pages`

**Validación en backend:** rechazar con 400 si `pagesRead > pages` (y `pages > 0`).

**Frontend:** input numérico con `max={pages}` para prevenir el error antes de enviarlo.

### 6.3 `pages = 0`

Libro con 0 páginas (datos corruptos o no rellenados).

- `percent = 0` (división por cero evitada).
- Se recomienda mostrar "Sin páginas registradas" en lugar de barra.
- El usuario debe editar `pages` antes de poder usar el progreso.

### 6.4 Cambiar `pages` después de haber leído

Si el usuario corrige `pages` de 300 a 350 después de haber puesto `pagesRead = 200`:
- `200 <= 350` → válido, progreso ahora es 57%.
- Si el usuario corrige `pages` de 300 a 150:
  - `200 > 150` → invalido. Backend rechaza o auto-acorta `pagesRead` a 150.
  - **Decisión recomendada:** auto-acortar `pagesRead` al nuevo `pages` con aviso al usuario ("El total de páginas se redujo, se ajustó tu progreso a 150").

---

## 7. Perspectiva de fases futuras

### 7.1 Relación con otras mejoras

| Feature | Relación |
|---------|----------|
| **Barcode scanner (fase 22)** | Independiente. El scanner añade libros; el progreso los acompaña después. |
| **Import de mazos (fase 23)** | Independiente. Otra colección (Magic). |
| **Estadísticas** | El progreso de lectura habilita estadísticas como "páginas leídas este mes", "libros completados", "promedio de páginas/día". Podría ser una futura fase de estadísticas. |
| **Objetivos de lectura** | Podría extenderse a "metas": "quiero leer 500 páginas este mes" con tracking. |

### 7.2 Si se quieren estadísticas en el futuro

Con `pagesRead` persistido por libro, se pueda calcular:

- Total de páginas leídas en el año
- Libros completados por mes
- Promedio de páginas/día (si se añade `updatedAt` o `pagesReadUpdatedAt`)
- Tasa de lectura por tipo (manga vs novela)

Esto queda fuera del alcance de esta propuesta, pero el campo `pagesRead` es el prerequisites.

---

## 8. Resumen de cambios

### Backend

| Componente | Cambio |
|------------|--------|
| `Book.java` | Añadir `Integer pagesRead` |
| `BookEntity.java` | Añadir campo `pagesRead` con @Field |
| `BookRequest.java` | Añadir `Integer pagesRead` (opcional) |
| `BookResponse.java` | Añadir `Integer pagesRead` |
| `BookDtoMapper.java` | Mapear `pagesRead` ida/vuelta |
| `BookService.java` | Validar `pagesRead <= pages` (opcional) |
| `BookController.java` | Aceptar `pagesRead` en PUT (ya funciona si el DTO lo tiene) |
| (Opcional) `ProgressUpdateRequest.java` | Nuevo DTO para PATCH `/progress` |
| (Opcional) `BookController.java` | Nuevo endpoint `PATCH /{id}/progress` |
| Tests | Añadir tests de `pagesRead` en servicio, controller, mapper |

### Frontend

| Componente | Cambio |
|------------|--------|
| `src/types/Book.ts` | Añadir `pagesRead?: number` |
| `src/components/ReadingProgressBar.tsx` | **NUEVO** — barra de progreso |
| `src/components/BookCard.tsx` | Añadir mini barra de progreso en estado READING |
| `src/components/BookForm.tsx` | Añadir campo "Páginas leídas" (solo si state=READING) |
| `src/pages/BookDetailPage.tsx` | Integrar `ReadingProgressBar` con input editable |
| `src/pages/BookCreatePage.tsx` | No cambia (ocultar campo) |
| `src/hooks/useReadingProgress.ts` | **NUEVO** (opcional) — hook de cálculo derivado |
| `src/components/BookBarcodeScanner.tsx` (existente) | No cambia |

### Documentation

| Archivo | Cambio |
|---------|--------|
| `docs/03-base-de-datos/03.1-tablas.md` | Añadir columna `pagesRead` a tabla Books |
| `docs/05-api/README.md` | Actualizar `BookRequest` y `BookResponse` con `pagesRead` |
| `docs/06-frontend/06.1-componentes.md` | Añadir documentación de `ReadingProgressBar` |
| `docs/00-home.md` | Añadir entrada en sección "Futuro" |

---

## 9. Prioridad y esfuerzo estimado

| Item | Esfuerzo |
|------|----------|
| Backend (modelo + DTO + mapper + 1 endpoint PATCH opcional) | ~2-4 horas |
| Tests backend | ~1-2 horas |
| Frontend (componente barra + BookCard + BookForm + BookDetailPage) | ~3-5 horas |
| Tests frontend | ~1 hora |
| Documentación | ~30 min |

**Total estimado:** 7-12 horas para implementación completa.

**Dependencia crítica:** Ninguna. El feature es totalmente aislado en la colección de libros.
