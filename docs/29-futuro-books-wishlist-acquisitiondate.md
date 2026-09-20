# Futuro — Wishlist + fecha de adquisición en libros

## Estado: Investigación / No implementado

## 1. Resumen

Dos cambios independientes pero relacionados para la colección de Books:

| Cambio | Tipo | Descripción |
|--------|------|-------------|
| **Nuevo estado `WISHLIST`** | Enum `BookState` | Añadir estado de "lista de deseado" similar al que ya tienen Games y BoardGames |
| **Campo `acquisitionDate`** | `LocalDate` (opcional) | Fecha en que el usuario adquirió el libro (compra, regalo, biblioteca...) |

**Qué NO cambia:** `TO_READ`, `READING`, `COMPLETED` se mantienen igual. `acquisitionDate` es opcional para cualquier estado, no solo TO_READ.

---

## 2. Estado actual

### 2.1 BookState enum (actual)

**`domain/model/BookState.java`**:
```java
public enum BookState {
    TO_READ,
    READING,
    COMPLETED
}
```

### 2.2 Book model (actual)

**`domain/model/Book.java`**:
```java
private String id;
private String externalId;
private String title;
private String descripcion;
private String author;
private Integer pages;
private BookType type;
private BookState state;       // TO_READ | READING | COMPLETED
private String comment;
private Integer start;          // 0-5, valoración personal
private LocalDate startDate;    // fecha de inicio de lectura
private LocalDate endDate;      // fecha de fin de lectura
private String frontpage;
```

### 2.3 Estado de otras colecciones para comparar

| Colección | Estados disponibles |
|-----------|---------------------|
| **Books** | TO_READ, READING, COMPLETED |
| **Games** | PLAYING, COMPLETED, WISHLIST, ABANDONED |
| **BoardGames** | OWNED, WISHLIST |
| **MovieShows** | WATCHING, WATCHED, PLAN_TO_WATCH |

Games y BoardGames tienen `WISHLIST`. MovieShows tiene `PLAN_TO_WATCH`. Books es la única colección sin un estado de "quiero leerlo pero no lo tengo todavía".

### 2.4 Comparación de campos de fecha

| Campo | Books | Games | BoardGames |
|-------|-------|-------|------------|
| Fecha de adición a la colección | ❌ No existe | `dateAdded` | `dateAdded` |
| Fecha de inicio (cuando empiezas a leer/jugar/ver) | `startDate` | `dateAdded` (cuando empiezas a jugar) | ❌ No existe |
| Fecha de fin (cuando terminas) | `endDate` | `dateCompleted` | ❌ No existe |
| **Fecha de adquisición** (compra/regalo) | ❌ No existe | ❌ No existe | ❌ No existe |

`acquisitionDate` sería un nuevo campo que ninguna colección tiene actualmente. Tendría sentido añadirlo a todas, pero este documento se centra solo en Books (si se hace en otras colecciones, sería un cambio separado).

---

## 3. BookState — añadir WISHLIST

### 3.1 Nuevo enum

**`domain/model/BookState.java`** — cambiar a:
```java
public enum BookState {
    WISHLIST,     // Quiero leerlo pero no lo tengo todavía
    TO_READ,      // Lo tengo pero no he empezado a leer
    READING,      // Actualmente leyéndolo
    COMPLETED     // Terminé de leer
}
```

**Orden semántico:** WISHLIST → TO_READ → READING → COMPLETED.

### 3.2 Impacto en conversores

**`infrastructure/config/StringToBookStateConverter.java`** — no requiere cambios. El converter usa `BookState.valueOf(source.trim().toUpperCase())`, así que cualquier nuevo valor del enum se convierte automáticamente.

### 3.3 Frontend

**`src/types/BookState.ts`** (o donde esté definido el enum de TypeScript) — añadir `WISHLIST`.

**`src/constants/books.ts`** — añadir entrada para WISHLIST con label y color.

**`src/components/StatusBadge.tsx`** — no requiere cambios si usa el enum directamente.

**`src/components/BookStateBadge.tsx`** (o el badge específico de libros) — añadir opción de WISHLIST con su estilo (diferente a TO_READ, probablemente un color más "pendiente", como gris o naranja suave).

### 3.4 Filtros y búsqueda

El endpoint `GET /api/v1/books?state=WISHLLIST` ya funciona si el backend usa el enum en la query. El `BookSearchCriteria` y el repositorio tendrán que soportar el nuevo valor, pero no requieren cambios estructurales — el filtro por `state` ya existe y funciona con cualquier valor del enum.

### 3.5 Semántica deWISHLIST vs TO_READ

| Estado | Qué significa | ¿Tengo el libro físico/digital? |
|--------|--------------|--------------------------------|
| WISHLIST | Quiero leerlo, no lo tengo todavía | No |
| TO_READ | Lo tengo, pero no he empezado | Sí |
| READING | Estoy leyéndolo | Sí |
| COMPLETED | Terminé de leerlo | Sí (ya lo tengo, lo terminé) |

La transición típica: WISHLIST → (compra) → TO_READ → (empiezas a leer) → READING → (terminas) → COMPLETED.

---

## 4. Campo `acquisitionDate`

### 4.1 Definición

**`domain/model/Book.java`** — añadir:
```java
private LocalDate acquisitionDate;   // fecha en que se adquirió el libro (compra, regalo, préstamo...)
```

**Semántica:**
- Fecha en que el usuario consiguió el libro (compra online, tienda, regalo, préstamo de biblioteca, descarga legal...).
- Es independiente de `startDate` (cuando empiezas a leer) — puedes adquirir un libro y no leerlo durante meses.
- Es opcional — no todos los libros tienen fecha de adquisición conocida (ej: libros heredados, libros que siempre estuvieron ahí).
- No cambia al cambiar de estado — si compras un libro, lo putting en WISHLIST o TO_READ con la `acquisitionDate`. Si luego pasas a READING, la `acquisitionDate` no cambia.

### 4.2 Diferencia con `dateAdded`

| Campo | Qué representa |
|-------|----------------|
| `dateAdded` | Cuándo el usuario añadió el libro a la aplicación (cuando lo creó en la DB) |
| `acquisitionDate` | Cuándo el usuario adquirió físicamente/el digitalmente el libro |

Pueden ser la misma fecha si añades el libro a la app justo cuando lo compras. Pero pueden ser diferentes si añades un libro que compraste hace años y no lo habías registrado.

### 4.3 DTOs

**`infrastructure/adapter/in/web/dto/BookRequest.java`** — añadir:
```java
LocalDate acquisitionDate;
```

**`infrastructure/adapter/in/web/dto/BookResponse.java`** — añadir:
```java
LocalDate acquisitionDate;
```

### 4.4 Mapper

**`infrastructure/adapter/in/web/dto/BookDtoMapper.java`** — añadir mapeo de `acquisitionDate` en `toDomain()` y `toResponse()`.

**`infrastructure/adapter/out/persistence/BookEntityMapper.java`** — añadir mapeo de `acquisitionDate` en `toEntity()` y `toDomain()`.

### 4.5 Entity

**`infrastructure/adapter/out/persistence/BookEntity.java`** — añadir:
```java
private LocalDate acquisitionDate;
```

### 4.6 Service

**`application/service/BookService.java`** — `copyUpdatableFields()` añadir:
```java
target.setAcquisitionDate(source.getAcquisitionDate());
```

No requiere lógica adicional — el campo es opcional y se copia directamente.

---

## 5. Frontend

### 5.1 Tipos TypeScript

**`src/types/Book.ts`** — añadir:
```typescript
export interface Book {
  // ... campos existentes ...
  state: BookState;           // ya existe, pero el enum cambia
  acquisitionDate?: string;   // ← NUEVO (ISO date string)
}
```

**`src/types/BookState.ts`** (o donde esté definido) — añadir `WISHLIST`.

**`src/types/BookFormData.ts`** — añadir:
```typescript
state: BookState;
acquisitionDate?: string;
```

### 5.2 Componente: `BookForm.tsx`

**Cambios:**

1. **Nuevo estado WISHLIST en el select de estado:** añadir opción "📚 Lista de deseados" (o similar).

2. **Campo de fecha de adquisición:** añadir input `type="date"` para `acquisitionDate`.

3. **Visibilidad condicional:** la fecha de adquisición puede ser visible siempre (como campo opcional) o mostrar solo cuando `state !== WISHLIST` (porque si está en WISHLIST todavía no lo tienes). Recomendación: mostrar siempre como campo opcional, que el usuario decida cuándo rellenarlo.

Ejemplo de campos del formulario:
```
Estado: [WISHLIST ▼]
Título: ...
Autor: ...
Fecha de adquisición: [2024-03-15]  ← campo opcional, siempre visible
...
```

### 5.3 Componente: `BookCard.tsx`

Mostrar el estado con el badge correspondiente. Si el estado es WISHLIST, el badge tiene estilo diferente (más "pendiente"). No mostrar `acquisitionDate` en el card (solo en el detail page).

### 5.4 Página: `BookDetailPage.tsx`

Mostrar:
- Estado con badge ( WISHLIST tiene su propio estilo).
- Fecha de adquisición (si existe): "Adquirido: 15 Mar 2024".
- Si no hay `acquisitionDate`: no mostrar el campo (o mostrar "Sin fecha de adquisición registrada").

### 5.5 Estadísticas visuales (opcional, futuro)

Con WISHLIST + `acquisitionDate`, estadísticas futuras podrían incluir:
- "Libros en wishlist: 12"
- "Tiempo promedio entre adquisición y lectura: 3 meses"
- "Libros adquiridos este año: 24"

Esto es fuera del alcance de este documento pero es el tipo de cosas que el campo habilita.

---

## 6. Tests

### Backend

- **`BookDtoMapperTest`**: verificar mapeo de `acquisitionDate` domain ↔ request/response.
- **`BookEntityMapperTest`**: verificar mapeo entity ↔ domain con `acquisitionDate`.
- **`BookServiceTest`**: verificar que `copyUpdatableFields` copia `acquisitionDate`; verificar que los estados WISHLIST/TO_READ/READING/COMPLETED funcionan en CRUD.
- **`BookControllerTest`**: verificar que PUT/POST aceptan `acquisitionDate` y el nuevo estado WISHLIST sin error 400.
- **`StringToBookStateConverterTest`**: verificar que WISHLIST se convierte correctamente (si hay tests existentes del converter).

### Frontend

- **`BookForm.test.tsx`**: verificar que WISHLIST aparece en el select de estado; verificar que el campo de `acquisitionDate` se muestra y envía correctamente.
- **`BookCard.test.tsx`**: verificar que el badge de WISHLIST tiene el estilo correcto.
- **`BookDetailPage.test.tsx`**: verificar que `acquisitionDate` se muestra en la sección de detalles.

---

## 7. Esquema de cambios — resumen por archivo

### Backend

| Archivo | Cambio |
|---------|--------|
| `domain/model/BookState.java` | Añadir `WISHLIST` al enum |
| `domain/model/Book.java` | Añadir `LocalDate acquisitionDate` |
| `infrastructure/adapter/out/persistence/BookEntity.java` | Añadir `LocalDate acquisitionDate` |
| `infrastructure/adapter/in/web/dto/BookRequest.java` | Añadir `LocalDate acquisitionDate` |
| `infrastructure/adapter/in/web/dto/BookResponse.java` | Añadir `LocalDate acquisitionDate` |
| `infrastructure/adapter/in/web/dto/BookDtoMapper.java` | Mapear `acquisitionDate` en `toDomain()` y `toResponse()` |
| `infrastructure/adapter/out/persistence/BookEntityMapper.java` | Mapear `acquisitionDate` en `toEntity()` y `toDomain()` |
| `application/service/BookService.java` | Añadir `setAcquisitionDate()` en `copyUpdatableFields()` |
| `infrastructure/config/StringToBookStateConverter.java` | Sin cambios (el converter ya maneja cualquier valor del enum) |

### Frontend

| Archivo | Cambio |
|---------|--------|
| `src/types/BookState.ts` | Añadir `WISHLIST` al enum |
| `src/types/Book.ts` | Añadir `state` (enum actualizado) y `acquisitionDate?: string` |
| `src/types/BookFormData.ts` | Añadir `state: BookState` y `acquisitionDate?: string` |
| `src/constants/books.ts` | Añadir WISHLIST con label y color al mapa de estados |
| `src/components/BookForm.tsx` | Añadir WISHLIST al select de estado; añadir campo de fecha de adquisición |
| `src/components/BookStateBadge.tsx` (o `StatusBadge.tsx`) | Añadir estilo para WISHLIST (si usa un mapa de colores por estado) |
| `src/pages/BookDetailPage.tsx` | Mostrar `acquisitionDate` en detalles |
| `src/pages/BookCreatePage.tsx` / `BookEditPage.tsx` | Sin cambios directos (usan BookForm) |
| `src/api/booksApi.ts` | Sin cambios directos (los nuevos campos vienen en BookResponse) |

---

## 8. Flujo de usuario

### 8.1 Añadir libro a wishlist

```
BookCreatePage
├── Título: "El nombre del viento"
├── Autor: Patrick Rothfuss
├── Estado: [WISHLIST ▼]  ← nuevo estado
├── Fecha de adquisición: [ ]  ← vacío (no lo tengo todavía)
├── ...
└── [Crear libro]

→ El libro aparece en la colección con estado WISHLIST
→ No se puede pasar a READING directamente desde WISHLIST (el frontend puede ocultar el botón de "empezar a leer" para WISHLIST)
```

### 8.2 Comprar el libro y pasar a TO_READ

```
BookEditPage (juego en WISHLIST)
├── Estado: [WISHLIST → TO_READ ▼]
├── Fecha de adquisición: [2024-06-15]  ← ahora rellenable
├── ...
└── [Guardar]

→ El libro pasa a TO_READ con acquisitionDate = 2024-06-15
```

### 8.3 Ver libro en wishlist

```
BookDetailPage (libro en WISHLIST)
├── Título: "El nombre del viento"
├── Autor: Patrick Rothfuss
├── Estado: 📚 Lista de deseados
├── Fecha de adquisición: —
└── ...
```

### 8.4 Ver libro adquirido pero no leído

```
BookDetailPage (libro en TO_READ)
├── Título: "El nombre del viento"
├── Estado: 📖 Por leer
├── Fecha de adquisición: 15 Jun 2024
└── ...
```

---

## 9. Riesgos y consideraciones

### 9.1 Migración del enum

Añadir `WISHLIST` al enum `BookState` es un cambio backwards-compatible para el backend — los documentos existentes con `TO_READ`, `READING`, `COMPLETED` siguen siendo válidos. No hay datos que migrar.

Para el frontend, si el usuario tiene libros creados antes de este cambio, el frontend los renderiza con los estados existentes. WISHLIST solo aparece cuando el usuario lo selecciona explícitamente.

### 9.2 `acquisitionDate` vs `dateAdded`

Si un usuario crea un libro hoy y pone `acquisitionDate = hoy`, el campo `dateAdded` (que pone Spring Data automáticamente) también será hoy. No hay conflicto — son campos con semántica diferente:

- `dateAdded` = cuándo se añadió a la app (auto-generado, no editable).
- `acquisitionDate` = cuándo se adquirió el libro físicamente (manual, opcional).

El frontend puede mostrar ambos si es útil: "Añadido a la colección: 20 Ene 2026" / "Adquirido: 15 Mar 2024".

### 9.3 Validación de fecha

No hay validación necesaria para `acquisitionDate` — es opcional y puede ser cualquier fecha pasada o futura (si compras un libro prestado para futuro, la fecha de adquisición puede ser la de la reserva). No se recomienda validar que no sea futura, porque hay casos legítimos.

### 9.4 Transiciones de estado

No hay reglas automatizadas de transición — el usuario puede cambiar de cualquier estado a cualquier otro manualmente. La app no valida que la transición sea "lógica" (ej: el usuario podría poner un libro de WISHLIST directamente a COMPLETED sin pasar por TO_READ y READING). Eso es aceptable — el usuario puede saber lo que hace.

Si en el futuro se quieren reglas de transición, se pueden añadir en `BookService` sin cambiar el modelo.

### 9.5 Filtrado por WISHLIST

El endpoint `GET /api/v1/books?state=WISHLIST` debe funcionar automaticamente con el nuevo valor del enum. Si el repositorio usa una consulta con `@Query` que filtra por `state`, solo necesita que el valor esté en el enum. No se requieren cambios en el repositorio.

---

## 10. Esfuerzo estimado

| Área | Tiempo |
|-------|--------|
| Backend: enum + modelo + entity + DTOs + mappers + service | 1-2h |
| Backend tests | 1h |
| Frontend: tipos + constantes + BookForm + BookStateBadge + BookDetailPage | 2-3h |
| Frontend tests | 1h |
| **Total** | **5-7h** |

---

## 12. Documentos relacionados

- `docs/03-base-de-datos/03.1-tablas.md` — esquema actual de Books (sin WISHLIST ni acquisitionDate)
- `docs/05-api/README.md` — BookRequest y BookResponse actuales
- `docs/13-fase-1-libros.md` — fase 1 de libros (estado actual)
- `docs/27-futuro-generos-books-games-boardgames.md` — añade `categories` a Books (compatible)
- `docs/24-futuro-reading-progress.md` — añade `pagesRead` a Books (compatible)
- `docs/28-futuro-boardgame-rating.md` — añade rating a BoardGames (idea similar de valoración personal)

---

## 13. Orden de implementación

Este cambio es independiente de `24-futuro-reading-progress.md` (pagesRead) y `27-futuro-generos-books-games-boardgames.md` (categories). Se pueden implementar en cualquier orden o juntos. Si se hacen los tres juntos, el cambio en `Book` sería: añadir `WISHLIST` al enum, `acquisitionDate`, `categories` y `pagesRead` de una vez.
