# Futuro — Géneros en colecciones (Books, Games, BoardGames, MovieShows)

## Estado: Investigación / No implementado

## 1. Resumen ejecutivo

| Colección | API externa | ¿Tiene género? | Campo API | Estado actual |
|-----------|-------------|----------------|-----------|---------------|
| Books (Libros) | Google Books | ✅ Sí | `volumeInfo.categories[]` | Mapeado en `BookSearchResult`, NO persistido en `Book` |
| Games (Videojuegos) | RAWG + FreeToGame | ✅ Sí | `genres[].name` (RAWG), `genre` (FreeToGame) | Mapeado en `GameSearchResult`, YA persistido en `Game` |
| Board Games (Juegos de Mesa) | BGG XML API 2 | ✅ Sí (doble) | `categories[]` + `mechanics[]` | `categories` mapeado+persistido. `mechanics` mapeado en search result, NO persistido |
| MovieShows (Películas/Series) | TMDB | ❌ No (pelis) / ⚠ Parcial (TV) | Sin equivalente directo | No mapeado ni persistido |
| Magic Cards | Scryfall | N/A | `type` (tipo de carta, no género) | No aplica — excluido |

**Conclusión:** Books y BoardGames tienen un gap claro (géneros disponibles en la API pero no persistidos). Games ya tiene género persistido. MovieShows requiere investigación adicional (TVDB o alternativas) porque TMDB no tiene género nativo para películas.

---

## 2. Books — Google Books categories

### 2.1 Estado actual del código

**`GoogleBooksClient.java`** (119): mapea `info.categories()` directamente a `BookSearchResult.categories` (List<String>). **El dato llega al search result pero se pierde al persistir.**

**`BookSearchResult.java`** (16): ya tiene `List<String> categories`.

**`Book.java`** (domain model, `docs/03-base-de-datos/03.1-tablas.md:3-19`): NO tiene campo `categories` ni `genre`. Solo: id, externalId, title, descripcion, author, pages, type, state, comment, start, startDate, endDate, frontpage.

**`BookEntity.java`**: igual, sin campos de categoría.

### 2.2 Lo que devuelve Google Books

```json
{
  "volumeInfo": {
    "categories": ["Fiction", "Juvenile Fiction", "Magic", "School stories"],
    ...
  }
}
```

- `categories` es un **array de strings** con los géneros/classificaciones del libro.
- No es un campo normalizado — es texto libre (puede ser "Fiction", "Ficción", "Juvenile Fiction", "Biography", etc.).
- Un libro puede tener 1-5 categorías.
- La API no devuelve un ID ni slug de género, ni una lista maestra de géneros válidos.
- La búsqueda por categoría existe en Google Books: `?q=subject:fiction` pero no se usa actualmente.

### 2.3 Lo que persistiría

Opción recomendada: persistir **todas las categorías** como lista de strings, igual que llegan de Google Books.

```java
// Book.java — campo nuevo
private List<String> categories;   // ej: ["Fiction", "Juvenile Fiction"]
```

No es necesario un enum porque Google Books usa strings libres y no hay una lista cerrada de géneros oficiales.

### 2.4 Frontend

- `Book.ts`: añadir `categories?: string[]`
- `BookCard.tsx`: mostrar pills de categoría (ej: `<CategoryPill label="Fiction" />`)
- `BookDetailPage.tsx`: mostrar todas las categorías
- `BookForm.tsx`: no editable manualmente (viene de Google Books, no se edita)

---

## 3. Games — RAWG genres (ya implementado)

### 3.1 Estado actual del código

**YA IMPLEMENTADO.** `Game.java` (`docs/03-base-de-datos/03.1-tablas.md`) ya tiene campo `genre: String` (el wiki no lo documenta explícitamente en la tabla, pero el código existe).

**`RAWGClient.java`** (121): `firstGenre()` mapea el primer género de la lista `genres[].name` de RAWG a `GameSearchResult.genre`.

**`GameSearchResult.java`** (9): tiene `String genre`.

**`GameResponse.java` / `GameRequest.java`**: persisten `genre`.

### 3.2 Lo que devuelve RAWG

```json
{
  "genres": [
    { "id": 4, "name": "Action" },
    { "id": 3, "name": "Adventure" }
  ]
}
```

- `genres` es un **array de objects** con `id` y `name`.
- RAWG tiene un endpoint `GET /genres` que devuelve la lista maestra de géneros (con ID y slug).
- IDs conocidos: 1=Action, 2=Strategy, 3=Adventure, 4=Action, 5=Shooter, 7=RPG, 10=Racing, 14=Simulation, 15=Sports...
- La categoría se guarda como el `name` del primer género (solo uno).

### 3.3 Limitación actual

**Solo se guarda el primer género** (`genres.get(0).name()` en RAWGClient.java:121). RAWG devuelve múltiples géneros pero el código de mapeo solo toma el primero.

**Mejora posible:** persistir todos los géneros como lista, no solo el primero.

```java
// Game.java — mejora propuesta
private List<String> genres;   // en vez de String genre
```

### 3.4 FreeToGame

FreeToGame devuelve `genre` como un **string único** (ej: "Shooter", "Strategy"). También tiene `tags[]` con valores más granulares (mmorpg, open-world, survival, etc.).

### 3.5 Frontend

Game ya tiene `genre` (o podría tener `genres[]`). No requiere cambios mayores. Sustantivo: pasar de `String genre` a `List<String> genres` requiere cambiar `Game.ts`, `GameCard.tsx`, `GameForm.tsx`.

---

## 4. BoardGames — BGG categories + mechanics

### 4.1 Estado actual del código

**`BoardGameXmlMapper.java`**: mapea `categories` (categorías de BGG) y `mechanics` (mecánicas de juego) del XML de BGG.

**`BoardGameSearchResult`**: tiene `List<String> categories` (confirmado por el mapeo XML).

**`BoardGame.java`** (`docs/03-base-de-datos/03.1-tablas.md:38-60`): tiene `categories: List<String>` (persistido). **NO tiene `mechanics`.**

**`BoardGameEntity.java`**: tiene `categories`, no tiene `mechanics`.

### 4.2 Lo que devuelve BGG XML API

```xml
<item type="boardgame" id="131972">
  <name type="primary" value="Catan" />
  <yearpublished value="1995" />
  <categories>
    <!!category>Economic</!!category>
    <!!category>Industry / Manufacturing</!!category>
    <!!category>Nuclear</!!category>
  </categories>
  <mechanics>
    <!!mechanic>Dice Rolling</!!mechanic>
    <!!mechanic>Modular Board</!!mechanic>
    <!!mechanic>Resource Management</!!mechanic>
    <!!mechanic>Routing</!!mechanic>
    <!!mechanic>Trading</!!mechanic>
  </mechanics>
  ...
</item>
```

**Dos conceptos distintos:**

| Campo BGG | Tipos de valores | Ejemplo |
|-----------|-----------------|---------|
| `categories` | Clasificación temática | Economic, Fantasy, Party Games, War Games, Trivia... |
| `mechanics` | Mecánicas de juego | Dice Rolling, Worker Placement, Hand Management, Area Control... |

BGG tiene lists maestras para ambos en su wiki, pero la API no devuelve IDs ni slugs normalizados — son strings libres.

### 4.3 Lo que hay que añadir

**`mechanics` ya está mapeado en el search result pero NO persistido.** Hay que añadir `List<String> mechanics` a:

- `BoardGame.java`
- `BoardGameEntity.java`
- `BoardGameRequest.java` / `BoardGameResponse.java`
- `BoardGameDtoMapper.java`

Las `categories` ya están persistidas — no hay cambio necesario allí.

### 4.4 Frontend

- `BoardGame.ts`: añadir `mechanics?: string[]`
- `BoardGameCard.tsx`: mostrar tags de mecánicas (ej: pequeños badges "Dice Rolling", "Worker Placement")
- `BoardGameDetailPage.tsx`: mostrar sección de mecánicas y categorías separadas (son conceptos diferentes)

---

## 5. MovieShows — ausencia de género en TMDB

### 5.1 Estado actual

**TMDB NO tiene un campo de género para películas.** La API de TMDB para películas (`/movie/{id}`) y series (`/tv/{id}`) no incluye un campo `genre` o `genres` en sus responses principales.

**`TmdbClient.java`** (`docs/05-api/externas/externas-movies.md:79-110`): mapea título, sinopsis, fecha, póster, backdrop, voto. No hay mapeo de género porque TMDB no lo devuelve en estos endpoints.

**`MovieShow.java`**: no tiene campo de género.

### 5.2 Alternativa: TVDB (TheTVDB) o TMDB con endpoint adicional

**Opción A: TMDB /genre/movie/list y /genre/tv/list**

TMDB sí tiene endpoints de género:
```
GET /genre/movie/list?language=es-ES
GET /genre/tv/list?language=es-ES
```

Y se puede añadir `?with_genres=28,12` a la búsqueda, pero solo funciona como filtro de búsqueda, no como campo del response de detalle. Para obtener los géneros de una película ya encontrada, habría que hacer una petición extra al endpoint `/movie/{id}` con `append_to_response=genres`.

**Opción B: TVDB (TheTVDB)**

TheTVDB es una alternativa a TMDB con un esquema más enriquecido para series, incluyendo género de forma nativa. Pero sería una segunda API para maintenance.

### 5.3 Recomendación

**No implementar géneros en MovieShows ahora.** Razones:

1. TMDB no lo ofrece de forma nativa y sencilla.
2. Obtenerlo requiere petición extra o segunda API (TVDB).
3. El beneficio es bajo comparado con el esfuerzo.

Si en el futuro se implementa, la vía más limpia sería usar TMDB con `append_to_response=genres` en el detalle, o añadir TheTVDB para series.

### 5.4 Frontend

No aplica hasta que el backend tenga el campo.

---

## 6. Comparativa de esfuerzo

| Colección | Estado actual | Cambio necesario | Esfuerzo estimado |
|-----------|--------------|------------------|-------------------|
| Books | Género disponible en API, NO persistido | Añadir `List<String> categories` a Book.java, BookEntity.java, DTOs, mapper | Baja (1-2h) |
| Games | Género YA persistido (un solo genre) | Cambiar `String genre` → `List<String> genres` para aprovechar todos los géneros de RAWG | Baja-Media (2-3h) |
| BoardGames | Categ. persistidas, mecánicas NO persistidas | Añadir `List<String> mechanics` a BoardGame.java, entity, DTOs, mapper | Baja (1-2h) |
| MovieShows | No hay género disponible | Requiere integración TMDB genres con petición extra O segunda API (TVDB) | Media-Alta (4-6h) |
| Magic | N/A (no tiene género) | Sin cambio | — |

---

## 7. Implementación recomendada por orden

1. **BoardGames — añadir `mechanics`** (el dato ya está en BGG, solo falta persistirlo, es el cambio más limpio y con mayor valor añadido para usuarios de juegos de mesa).

2. **Books — añadir `categories`** (Google Books ya lo devuelve, está mapeado en search result, falta persistirlo).

3. **Games — migrar a `List<String> genres`** (cambio menor, permite mostrar múltiples géneros en lugar de uno solo).

4. **MovieShows — posponer** (requiere más investigación/integración).

---

## 8. Documentos relacionados

- `docs/05-api/externas/externas-books.md` — Google Books API, campos categories
- `docs/05-api/externas/externas-videogames.md` — RAWG genres, FreeToGame genre
- `docs/05-api/externas/externas-boardgames.md` — BGG categories + mechanics
- `docs/05-api/externas/externas-movies.md` — TMDB, sin género nativo
- `docs/03-base-de-datos/03.1-tablas.md` — esquema actual de cada colección
- `docs/00-home.md` — índice del proyecto
