# Futuro — Géneros y categorías en Books, Games y BoardGames

## Estado: Investigación / No implementado

## 1. Resumen

Añadir campos de género/categoría a tres colecciones que ya los reciben de sus APIs externas pero no los persisten, o solo persisten parcialmente:

| Colección | API | Campo API | Estado actual | Cambio |
|-----------|-----|-----------|---------------|--------|
| **Books** | Google Books | `volumeInfo.categories[]` (List\<String\>) | Mapeado en `BookSearchResult`, **NO persistido** en `Book` | Añadir `List<String> categories` |
| **Games** | RAWG | `genres[].name` (array de objects con id+name) | Solo **primer género** persistido como `String genre` | Cambiar a `List<String> genres` |
| **BoardGames** | BGG XML API | `categories[]` + `mechanics[]` | `categories` persistido. `mechanics` **NO persistido** (está en search result) | Añadir `List<String> mechanics` |

**No incluye:** MovieShows (TMDB no tiene género nativo, requiere petición extra o segunda API) y Magic (Scryfall no tiene género — el campo `type` es tipo de carta).

---

## 2. Books — añadir `categories` de Google Books

### 2.1 Estado actual

**GoogleBooksClient.java** (línea 119): mapea `info.categories()` → `BookSearchResult.categories`. El dato de entrada está disponible y llega al search result.

**BookSearchResult.java** (línea 16): ya tiene `List<String> categories`.

**Book.java** (`docs/03-base-de-datos/03.1-tablas.md:3-19`): **NO tiene `categories`**. Campos actuales: id, externalId, title, descripcion, author, pages, type, state, comment, start, startDate, endDate, frontpage.

**BookEntity.java**: igual, sin `categories`.

### 2.2 Qué devuelve Google Books

```json
{
  "volumeInfo": {
    "categories": ["Fiction", "Juvenile Fiction", "Magic", "School stories"]
  }
}
```

- Array de strings libres (no normalizados, no hay IDs ni slugs).
- Un libro puede tener entre 1 y 5 categorías.
- Ejemplos: "Fiction", "Juvenile Fiction", "Biography", "History", "Science Fiction", "Comics & Graphic Novels", "Fantasy".
- La búsqueda por categoría existe en Google Books (`?q=subject:fiction`) pero no se usa actualmente en la app.

### 2.3 Casos donde `categories` está vacío

- Algunos libros no tienen categorías asignadas por Google Books.
- El campo es opcional — si está vacío, se muestra "Sin categoría" o no se muestra la sección.

### 2.4 Cambios backend

**`domain/model/Book.java`**:
```java
private List<String> categories;
```

**`infrastructure/adapter/out/persistence/BookEntity.java`**:
```java
@Field("categories")
private List<String> categories;
```

**`infrastructure/adapter/in/web/dto/BookRequest.java`**:
```java
List<String> categories;
```

**`infrastructure/adapter/in/web/dto/BookResponse.java`**:
```java
List<String> categories;
```

**`infrastructure/adapter/in/web/dto/BookDtoMapper.java`**:
```java
// toDomain: añadir .categories(request.categories())
// toResponse: añadir .categories(movieShow.getCategories())
```

**`infrastructure/adapter/out/persistence/BookEntityMapper.java`**:
```java
// toEntity: añadir .categories(movieShow.getCategories())
// toDomain: añadir .categories(entity.getCategories())
```

No se requieren cambios en `GoogleBooksClient` — el mapeo de `categories` ya existe en `toResult()`.

### 2.5 Schema MongoDB

El campo se añade a la colección `books` como array de strings. No requiere índice nuevo — las queries de búsqueda por categoría no están implementadas ni planeadas actualmente.

---

## 3. Games — migrar `genre` → `List<String> genres`

### 3.1 Estado actual

**`Game.java`** (`docs/03-base-de-datos/03.1-tablas.md` — campo `genre: String`): solo un género persistido. El wiki no documenta explícitamente este campo pero el código existe.

**`RAWGClient.java`** (línea 121): `firstGenre()` toma `genres.get(0).name()` — solo el primer género de la lista de RAWG.

**`GameSearchResult.java`**: tiene `String genre`.

**`GameResponse.java` / `GameRequest.java`**: persisten `String genre`.

### 3.2 Qué devuelve RAWG

```json
{
  "genres": [
    { "id": 4,  "name": "Action" },
    { "id": 3,  "name": "Adventure" },
    { "id": 7,  "name": "RPG" }
  ]
}
```

- Array de objects con `id` (numérico) y `name` (string).
- RAWG tiene lista maestra de géneros en `GET /genres?key=...` — IDs estables: 1=Action, 2=Strategy, 3=Adventure, 4=Action, 5=Shooter, 7=RPG, 10=Racing, 14=Simulation, 15=Sports, 17=Card, 19=Family, 51=Indie...
- La app actual solo usa el `name` del primer género.

### 3.3 Cambios backend

**`domain/model/Game.java`**: cambiar `String genre` por `List<String> genres`.

**`domain/model/GameSearchResult.java`**: cambiar `String genre` por `List<String> genres`.

**`infrastructure/adapter/out/rawg/RAWGClient.java`**:
```java
private List<String> allGenres(List<RawgGenre> genres) {
    if (genres == null || genres.isEmpty()) {
        return List.of();
    }
    return genres.stream()
        .map(RawgGenre::name)
        .toList();
}
```
Y en `toResult()`: usar `allGenres(game.genres())` en lugar de `firstGenre(game.genres())`.

**`infrastructure/adapter/in/web/dto/GameRequest.java`**: cambiar `String genre` por `List<String> genres`.

**`infrastructure/adapter/in/web/dto/GameResponse.java`**: cambiar `String genre` por `List<String> genres`.

**`infrastructure/adapter/in/web/dto/GameDtoMapper.java`**: ajustar mapeo de `genre` → `genres` en ambas direcciones.

**`infrastructure/adapter/out/persistence/GameEntity.java`**: cambiar `String genre` por `List<String> genres`.

**`infrastructure/adapter/out/persistence/GameEntityMapper.java`**: ajustar mapeo.

**`application/service/GameService.java`**: `copyUpdatableFields()` debe copiar `genres` en lugar de `genre`.

### 3.4 Qué pasa con FreeToGame

FreeToGame devuelve `genre` como un **string único** (ej: "Shooter"). En el momento de mapear FreeToGame a `GameSearchResult`, se puede envolver en una lista: `List.of(game.genre())`. No requiere cambios estructurales.

### 3.5 Migración de datos existentes

Los documentos existentes en MongoDB con `genre: "Action"` (string) tendrán que ser migrados. Opciones:

- **Migración manual en código:** al leer un `Game` con `genre` string y `genres` null, convertir automáticamente: si `genres == null && genre != null` → `genres = List.of(genre)`.
- **Script de migración MongoDB:** actualizar todos los documentos en una operación.
- **Compatibilidad:** el entity puede tener ambos campos temporalmente (`genre: String` como campo legacy + `genres: List<String>` nuevo), pero esto añade complejidad innecesaria. Mejor migración limpia.

**Recomendación:** añadir lógica de migración automática en `GameEntityMapper.toDomain()`:
```java
// Si el documento tiene genre string (versión antigua) pero no genres lista,
// convertir automáticamente
if (entity.getGenres() == null && entity.getGenre() != null) {
    entity.setGenres(List.of(entity.getGenre()));
    entity.setGenre(null); // limpiar campo legacy
}
```

---

## 4. BoardGames — añadir `mechanics` de BGG

### 4.1 Estado actual

**`BoardGameXmlMapper.java`**: mapea `categories` y `mechanics` del XML de BGG. Ambos están disponibles en el DTO de búsqueda.

**`BoardGameSearchResult`**: tiene `List<String> categories` mapeado. `mechanics` — no 확인ado explícitamente, pero estimable que el mapper lo tiene mapeado ya que el XML lo incluye.

**`BoardGame.java`** (`docs/03-base-de-datos/03.1-tablas.md:38-60`): tiene `categories: List<String>` (persistido). **NO tiene `mechanics`.**

**`BoardGameEntity.java`**: tiene `categories`, no tiene `mechanics`.

### 4.2 Qué devuelve BGG XML API

```xml
<item type="boardgame" id="131972">
  <name type="primary" value="Catan" />
  <categories>
    <!!category>Economic</!!category>
    <!!category>Industry / Manufacturing</!!category>
  </categories>
  <mechanics>
    <!!mechanic>Dice Rolling</!!mechanic>
    <!!mechanic>Modular Board</!!mechanic>
    <!!mechanic>Resource Management</!!mechanic>
    <!!mechanic>Trading</!!mechanic>
  </mechanics>
</item>
```

**Dos conceptos distintos que es importante no mezclar:**

| Campo | Qué representa | Ejemplos |
|-------|----------------|----------|
| `categories` | Clasificación temática de BGG | Economic, Fantasy, War Games, Party Games, Trivia, Adventure... |
| `mechanics` | Mecánicas de juego (cómo se juega) | Dice Rolling, Worker Placement, Hand Management, Area Control, Trading, Deck Building, Auction... |

BGG tiene listas maestras para ambos en su wiki, pero la API devuelve strings libres (no IDs ni slugs normalizados).

### 4.3 Cambios backend

**`domain/model/BoardGame.java`**: añadir `List<String> mechanics`.

**`infrastructure/adapter/out/persistence/BoardGameEntity.java`**: añadir `@Field("mechanics") private List<String> mechanics`.

**`infrastructure/adapter/in/web/dto/BoardGameRequest.java`**: añadir `List<String> mechanics`.

**`infrastructure/adapter/in/web/dto/BoardGameResponse.java`**: añadir `List<String> mechanics`.

**`infrastructure/adapter/in/web/dto/BoardGameDtoMapper.java`**: añadir mapeo de `mechanics` en `toDomain()` y `toResponse()`.

**`infrastructure/adapter/out/persistence/BoardGameEntityMapper.java`**: añadir mapeo de `mechanics` en `toEntity()` y `toDomain()`.

**`BoardGameXmlMapper.java`** (si `mechanics` no está mapeado todavía): añadir parseo del XML `<mechanics>` al DTO de búsqueda.

### 4.4 Frontend BoardGames

- `BoardGame.ts`: añadir `mechanics?: string[]`
- `BoardGameCard.tsx`: mostrar badges de mecánicas (estilo tags pequeños, color diferente a categorías)
- `BoardGameDetailPage.tsx`: sección de mecánicas separada de categorías (son conceptos diferentes)

---

## 5. Frontend — cambios por colección

### 5.1 Books

**`src/types/Book.ts`**:
```typescript
export interface Book {
  // ... campos existentes ...
  categories?: string[];   // ← NUEVO
}
```

**`src/types/BookFormData.ts`** (o donde esté definido):
```typescript
categories?: string[];
```

**`src/components/BookCard.tsx`**:
- Añadir fila de pills de categorías debajo del título (o junto al badge de estado).
- Ejemplo: `[Fiction] [Juvenile Fiction] [Magic]`
- Usar componente existente `FilterPill` o `StatusBadge` como plantilla visual.

**`src/pages/BookDetailPage.tsx`**:
- Mostrar categorías en sección de detalles del libro (junto con autor, páginas, etc.).
- Si `categories` está vacío → mostrar "Sin clasificación".

**`src/pages/BookCreatePage.tsx` / `BookEditPage.tsx`**:
- No mostrar campo de categorías editable (viene de Google Books, no se edita manualmente).
- Al crear desde búsqueda de Google Books, las categorías se auto-populan desde `BookSearchResult.categories`.

**`src/api/booksApi.ts`**: no requiere cambios — los datos vienen en `BookResponse` y el fetch genérico los maneja.

### 5.2 Games

**`src/types/Game.ts`**:
```typescript
export interface Game {
  // ... campos existentes ...
  genres: string[];   // ← cambio: era String genre, ahora array
}
```

**`src/types/GameFormData.ts`**: cambiar `genre?: string` por `genres?: string[]`.

**`src/components/GameCard.tsx`**:
- Mostrar todos los géneros como pills (no solo uno).
- Ejemplo: `[Action] [Adventure] [RPG]`
- Si la lista es larga, mostrar los 2-3 primeros y " +N más" o truncar.

**`src/pages/GameCreatePage.tsx` / `GameEditPage.tsx`**:
- El campo de género en el formulario: si es string unario, cambiarlo a un multi-select o tags input.
- Opcional: usar un componente de tags (ej: `<input type="text" />` con separación por comas, o un tag input con clicks).
- El género viene de RAWG al crear desde búsqueda — no es editable manualmente (o sí, si el usuario quiere sobreescribirlo).

**Migración frontend:** si el backend devuelve `genres: ["Action"]` en lugar de `genre: "Action"`, el frontend tiene que adaptarse. Si se hace la migración del backend con el campo legacy, el frontend puede seguir usando `genre` hasta que se completa la migración.

### 5.3 BoardGames

**`src/types/BoardGame.ts`**:
```typescript
export interface BoardGame {
  // ... campos existentes ...
  categories: string[];   // ya existe
  mechanics: string[];    // ← NUEVO
}
```

**`src/components/BoardGameCard.tsx`**:
- Mostrar mecánicas como tags pequeños (diferente estilo a categorías).
- Ejemplo: `[Worker Placement] [Dice Rolling] [Trading]`
- Podría haber dos grupos: "Categorías" y "Mecánicas" o mostrar solo las más relevantes en el card.

**`src/pages/BoardGameDetailPage.tsx`**:
- Sección "Categorías" con badges temáticos.
- Sección "Mecánicas" con badges de mecánicas.
- Ambas secciones con listas de pills/badges.

**`src/pages/BoardGameCreatePage.tsx` / `BoardGameEditPage.tsx`**:
- Las mecánicas vienen de BGG al crear desde búsqueda — no es editable manualmente por defecto.
- Si el usuario quiere sobreescribirlas, el formulario puede tener un campo de texto con etiquetas separadas por comas.

---

## 6. Tests necesarios

### Books

- `BookDtoMapperTest`: verificar que `categories` se mapea domain ↔ response/request.
- `BookEntityMapperTest`: verificar que `categories` se mapea entity ↔ domain.
- `BookServiceTest`: si se añaden validaciones sobre categories.
- `GoogleBooksClientTest`: no requiere cambios — el mapeo de categories ya existe.

### Games

- `GameDtoMapperTest`: verificar mapeo de `genres` (array) en ambas direcciones.
- `GameEntityMapperTest`: verificar mapeo entity ↔ domain con `genres`.
- `GameServiceTest`: verificar `copyUpdatableFields` copia `genres`.
- `RAWGClientTest`: verificar que `allGenres()` devuelve todos los géneros (no solo el primero). Test con 0, 1, 2+ géneros.
- `FreeToGameClientTest`: verificar que el string único se convierte a lista de un elemento.

### BoardGames

- `BoardGameDtoMapperTest`: verificar mapeo de `mechanics` en ambas direcciones.
- `BoardGameEntityMapperTest`: verificar mapeo entity ↔ domain con `mechanics`.
- `BoardGameXmlMapperTest`: verificar que `mechanics` del XML se mapea al search result (o confirmar que ya está mapeado).
- `BoardGameServiceTest`: si se añaden validaciones sobre mechanics.

### Frontend

- `BookCard.test.tsx`: verificar que las categorías se renderizan cuando existen.
- `BookDetailPage.test.tsx`: verificar sección de categorías.
- `GameCard.test.tsx`: verificar que los géneros se renderizan como array (múltiples pills).
- `GameForm.test.tsx`: verificar que el campo de género acepta múltiples valores.
- `BoardGameCard.test.tsx`: verificar que las mecánicas se renderizan.
- `BoardGameDetailPage.test.tsx`: verificar secciones de categorías y mecánicas.

---

## 7. Esquema de cambios — resumen por archivo

### Books

| Archivo | Cambio |
|---------|--------|
| `domain/model/Book.java` | Añadir `List<String> categories` |
| `infrastructure/adapter/out/persistence/BookEntity.java` | Añadir `@Field("categories") List<String> categories` |
| `infrastructure/adapter/in/web/dto/BookRequest.java` | Añadir `List<String> categories` |
| `infrastructure/adapter/in/web/dto/BookResponse.java` | Añadir `List<String> categories` |
| `infrastructure/adapter/in/web/dto/BookDtoMapper.java` | Mapear `categories` en `toDomain()` y `toResponse()` |
| `infrastructure/adapter/out/persistence/BookEntityMapper.java` | Mapear `categories` en `toEntity()` y `toDomain()` |
| `GoogleBooksClient.java` | Sin cambios (ya mapea categories) |
| `BookSearchResult.java` | Sin cambios (ya tiene categories) |

### Games

| Archivo | Cambio |
|---------|--------|
| `domain/model/Game.java` | Cambiar `String genre` → `List<String> genres` |
| `domain/model/GameSearchResult.java` | Cambiar `String genre` → `List<String> genres` |
| `infrastructure/adapter/out/rawg/RAWGClient.java` | Reemplazar `firstGenre()` por `allGenres()` que devuelve lista |
| `infrastructure/adapter/in/web/dto/GameRequest.java` | Cambiar `String genre` → `List<String> genres` |
| `infrastructure/adapter/in/web/dto/GameResponse.java` | Cambiar `String genre` → `List<String> genres` |
| `infrastructure/adapter/in/web/dto/GameDtoMapper.java` | Ajustar mapeo de `genres` |
| `infrastructure/adapter/out/persistence/GameEntity.java` | Cambiar `String genre` → `List<String> genres` (o añadir ambos temporalmente) |
| `infrastructure/adapter/out/persistence/GameEntityMapper.java` | Ajustar mapeo + lógica de migración de `genre` → `genres` |
| `application/service/GameService.java` | Ajustar `copyUpdatableFields()` para `genres` |
| `FreeToGameClient.java` | Envolver `genre` string en `List.of(genre)` al mapear |

### BoardGames

| Archivo | Cambio |
|---------|--------|
| `domain/model/BoardGame.java` | Añadir `List<String> mechanics` |
| `infrastructure/adapter/out/persistence/BoardGameEntity.java` | Añadir `@Field("mechanics") List<String> mechanics` |
| `infrastructure/adapter/in/web/dto/BoardGameRequest.java` | Añadir `List<String> mechanics` |
| `infrastructure/adapter/in/web/dto/BoardGameResponse.java` | Añadir `List<String> mechanics` |
| `infrastructure/adapter/in/web/dto/BoardGameDtoMapper.java` | Mapear `mechanics` en `toDomain()` y `toResponse()` |
| `infrastructure/adapter/out/persistence/BoardGameEntityMapper.java` | Mapear `mechanics` en `toEntity()` y `toDomain()` |
| `BoardGameXmlMapper.java` | Confirmar que `mechanics` del XML está mapeado (o añadirlo) |

### Frontend (tipos)

| Archivo | Cambio |
|---------|--------|
| `src/types/Book.ts` | Añadir `categories?: string[]` |
| `src/types/Game.ts` | Cambiar `genre?: string` → `genres?: string[]` |
| `src/types/BoardGame.ts` | Añadir `mechanics?: string[]` |

---

## 8. Flujo de usuario

### 8.1 Books — ver categorías de un libro

```
BookListPage
├── BookCard
│   ├── Título: "Harry Potter y la piedra filosofal"
│   ├── Autor: J.K. Rowling
│   └── Categorías: [Fiction] [Juvenile Fiction] [Magic]

BookDetailPage
├── ...
├── Categorías: Fiction, Juvenile Fiction, Magic, School stories
└── ...
```

Al crear el libro desde búsqueda de Google Books, las categorías se auto-llenan desde `BookSearchResult.categories`. El usuario no edita las categorías manualmente.

### 8.2 Games — ver múltiples géneros

```
GameListPage
├── GameCard
│   ├── Título: "The Witcher 3"
│   └── Géneros: [Action] [Adventure] [RPG]

GameDetailPage
├── ...
├── Géneros: Action, Adventure, RPG, Open World
└── ...
```

Antes: solo se veía "Action" (el primer género). Ahora se ven todos los que devuelve RAWG.

### 8.3 BoardGames — ver mecánicas

```
BoardGameListPage
├── BoardGameCard
│   ├── Título: "Catan"
│   ├── Categoría: Economic
│   └── Mecánicas: [Dice Rolling] [Modular Board] [Trading]

BoardGameDetailPage
├── ...
├── Categorías: Economic, Industry / Manufacturing
├── Mecánicas: Dice Rolling, Modular Board, Resource Management, Trading, Hex-and-Counter
└── ...
```

---

## 9. Riesgos y consideraciones

### 9.1 Migration de datos existentes (Games)

Los documentos de `games` en MongoDB con el campo `genre: String` deben migrarse a `genres: List<String>`. Sin migración, los documentos antiguos tendrán `genres = null` y el frontend tendría que manejar ambos formatos.

**Estrategia recomendada:** migrar en el `GameEntityMapper.toDomain()` — si `genres` es null pero `genre` tiene valor, convertir automáticamente. Esto hace la migración transparente sin script de MongoDB.

### 9.2 Datos inconsistentes en Google Books

Las categorías de Google Books son strings libres — un libro puede tener "Fiction" y otro "Ficción" para lo mismo. No hay normalización posible sin una lista maestra propia.

**Comportamiento aceptable:** mostrar las categorías tal como llegan de Google Books. Si el usuario quiere consistentes, tendría que editarlas manualmente en el futuro (out of scope para esta feature).

### 9.3 Cadenas largas de mecánicas en BoardGames

Algunos juegos de mesa tienen 8-10 mecánicas. Mostrarlas todas en el card puede hacer el card muy alto. Estrategia: mostrar las 3-4 primeras en el card y todas en el detalle.

### 9.4 Campos opcionales

Todos los nuevos campos son opcionales (`List` puede ser null/empty). No rompen la creación manual de libros/juegos/boardgames sin usar la búsqueda externa.

### 9.5 Frontend con datos antiguos

Si el backend se despliega con los nuevos campos pero el frontend no se actualiza, el frontend ignorará los campos nuevos (TypeScript los tiene como opcionales o no los lee). No hay breaking change visible para el usuario hasta que se actualice el frontend.

---

## 10. Esfuerzo estimado

| Área | Tiempo |
|-------|--------|
| Backend Books (modelo + entity + DTOs + mapper) | 1-2h |
| Backend Games (modelo + entity + DTOs + mapper + servicio + RAWGClient + FreeToGame) | 2-3h |
| Backend BoardGames (modelo + entity + DTOs + mapper + XML mapper) | 1-2h |
| Backend tests (Books + Games + BoardGames) | 2-3h |
| Frontend tipos (Book, Game, BoardGame) | 30min |
| Frontend componentes (BookCard, GameCard, BoardGameCard, detail pages) | 2-3h |
| Frontend tests | 1-2h |
| **Total** | **10-16h** |

---

## 11. Orden de implementación recomendado

1. **Books — `categories`** (cambio más simple, solo añadir campo, sin migración de datos existentes).
2. **BoardGames — `mechanics`** (cambio simple, solo añadir campo).
3. **Games — `genres[]`** (requiere migración de datos existentes, más complejo).

---

## 12. Documentos relacionados

- `docs/03-base-de-datos/03.1-tablas.md` — esquema actual de cada colección
- `docs/05-api/externas/externas-books.md` — Google Books API, campos categories
- `docs/05-api/externas/externas-videogames.md` — RAWG genres, FreeToGame genre
- `docs/05-api/externas/externas-boardgames.md` — BGG categories + mechanics
- `docs/25-futuro-streaming-platforms.md` — otra feature futura para MovieShows (no relacionada con género)
- `docs/24-futuro-reading-progress.md` — feature futura para Books (páginas leídas, no relacionado con género)
