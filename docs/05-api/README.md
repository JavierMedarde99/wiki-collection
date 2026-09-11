# API Reference

## Base URL

```
http://localhost:8080/api
```

## Estructura

| Sección | Descripción |
|---------|-------------|
| [Endpoints de Libros](#endpoints-de-libros) | CRUD + búsqueda en Google Books |
| [Endpoints de Juegos](#endpoints-de-juegos) | CRUD + búsqueda en RAWG/FreeToGame + logros Steam |
| [Endpoints de Juegos de Mesa](#endpoints-de-juegos-de-mesa) | CRUD + búsqueda en BGG |
| [Endpoints de Magic](#endpoints-de-magic) | Listado, detalle, eliminar + búsqueda en Scryfall |
| [Endpoints de Mazos](#endpoints-de-mazos) | CRUD + gestión cartas + status Commander |
| [Endpoints de Películas/Series](#endpoints-de-películas-series) | CRUD + búsqueda en TMDB |
| [APIs Externas](./externas/) | Integración con APIs externas |

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 404 | Not Found |
| 500 | Server Error |
| 502 | Bad Gateway (API externa) |
| 503 | Service Unavailable (API externa) |

## Paginación

Los endpoints que devuelven listas soportan paginación Spring Data:

```
GET /api/books?page=0&size=20&sort=title,asc
GET /api/games?page=0&size=20&sort=title,asc
GET /api/boardgames?page=0&size=20&sort=title,asc
GET /api/magic?page=0&size=20&sort=name,asc
GET /api/movieshows?page=0&size=20&sort=title,asc
```

**Response:**
```json
{
  "content": [...],
  "totalPages": 5,
  "totalElements": 100,
  "number": 0,
  "size": 20
}
```

---

## Endpoints de Libros

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/books` | Listar libros con paginación y filtros | ✅ |
| GET | `/api/books/{id}` | Obtener libro por ID | ✅ |
| POST | `/api/books` | Crear libro | ✅ |
| PUT | `/api/books/{id}` | Actualizar libro | ✅ |
| DELETE | `/api/books/{id}` | Eliminar libro (204 No Content) | ✅ |
| GET | `/api/books/search?name={query}` | Buscar en Google Books API | ✅ |

### Filtros de Libros

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título (LIKE case-insensitive) | `?name=harry` |
| `author` | String | Buscar por autor (LIKE case-insensitive) | `?author=rowling` |
| `type` | Enum | Filtrar por tipo (MANGA, NOVEL, GRAPHIC_NOVEL) | `?type=MANGA` |
| `state` | Enum | Filtrar por estado (TO_READ, READING, COMPLETED) | `?state=READING` |

---

## Endpoints de Juegos

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/games` | Listar juegos con paginación y filtros | ✅ |
| GET | `/api/games/{id}` | Obtener juego por ID | ✅ |
| POST | `/api/games` | Crear juego | ✅ |
| PUT | `/api/games/{id}` | Actualizar juego | ✅ |
| DELETE | `/api/games/{id}` | Eliminar juego (204 No Content) | ✅ |
| GET | `/api/games/search?name={query}` | Buscar en RAWG/FreeToGame | ✅ |
| GET | `/api/games/{id}/achievements?steamId={id}` | Obtener logros de Steam | ✅ |

### Filtros de Juegos

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=witcher` |
| `platform` | Enum | Filtrar por plataforma (PC, PS2, PS3, WII_U, SWITCH) | `?platform=PC` |
| `status` | Enum | Filtrar por estado (PLAYING, COMPLETED, WISHLIST, ABANDONED) | `?status=PLAYING` |

---

## Endpoints de Juegos de Mesa

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/boardgames` | Listar juegos de mesa con paginación y filtros | ✅ |
| GET | `/api/boardgames/{id}` | Obtener juego de mesa por ID | ✅ |
| POST | `/api/boardgames` | Crear juego de mesa | ✅ |
| PUT | `/api/boardgames/{id}` | Actualizar juego de mesa | ✅ |
| DELETE | `/api/boardgames/{id}` | Eliminar juego de mesa (204 No Content) | ✅ |
| GET | `/api/boardgames/search?name={query}` | Buscar en BoardGameGeek (XML API) | ✅ |

### Filtros de Juegos de Mesa

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=catan` |
| `status` | Enum | Filtrar por estado (OWNED, WISHLIST) | `?status=OWNED` |

---

## Endpoints de Magic: The Gathering

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/magic` | Listar cartas con paginación y filtros | ✅ |
| GET | `/api/magic/{id}` | Obtener carta por ID | ✅ |
| DELETE | `/api/magic/{id}` | Eliminar carta (204 No Content) | ✅ |
| GET | `/api/magic/search?name={query}` | Buscar en Scryfall | ✅ |

**Nota:** El backend no expone POST/PUT para Magic. El frontend crea cartas directamente contra Scryfall y las persiste en local storage (no en MongoDB a través del backend). El backend solo gestiona listado, detalle, eliminación y búsqueda.

### Filtros de Magic

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por nombre (LIKE case-insensitive) | `?name=lightning` |
| `rarity` | String | Filtrar por rareza | `?rarity=rare` |
| `color` | String | Filtrar por color | `?color=R` |
| `type` | String | Filtrar por tipo | `?type=Creature` |
| `convertedManaCost` | Double | Filtrar por coste de maná convertido | `?convertedManaCost=3.0` |

---

## Endpoints de Mazos (Commander)

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/decks` | Listar mazos (filtro por nombre opcional) | ✅ |
| GET | `/api/decks/{id}` | Obtener mazo por ID | ✅ |
| POST | `/api/decks` | Crear mazo | ✅ |
| PUT | `/api/decks/{id}` | Actualizar mazo | ✅ |
| DELETE | `/api/decks/{id}` | Eliminar mazo (204 No Content) | ✅ |
| POST | `/api/decks/{id}/cards` | Añadir carta desde Scryfall | ✅ |
| DELETE | `/api/decks/{id}/cards/{scryfallId}` | Quitar carta del mazo | ✅ |
| GET | `/api/decks/{id}/status` | Estado del mazo (DRAFT, COMPLETE, INVALID) | ✅ |

### Filtros de Mazos

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Filtrar por nombre exacto | `?name=Mi Mazo` |

### DeckCardRequest (body para añadir carta)

```json
{
  "scryfallId": "abc123",
  "quantity": 1
}
```

### DeckStatusResponse

```json
{
  "status": "COMPLETE",
  "message": "El mazo cumple las reglas Commander"
}
```

---

## Endpoints de Películas/Series

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/movieshows` | Listar películas/series con paginación y filtros | ✅ |
| GET | `/api/movieshows/{id}` | Obtener película/serie por ID | ✅ |
| POST | `/api/movieshows` | Crear película/serie | ✅ |
| PUT | `/api/movieshows/{id}` | Actualizar película/serie | ✅ |
| DELETE | `/api/movieshows/{id}` | Eliminar película/serie (204 No Content) | ✅ |
| GET | `/api/movieshows/search?name={query}` | Buscar en TMDB | ✅ |

### Filtros de Películas/Series

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=matrix` |
| `status` | Enum | Filtrar por estado (WATCHING, WATCHED, PLAN_TO_WATCH) | `?status=WATCHING` |
| `mediaType` | Enum | Filtrar por tipo (MOVIE, TV) | `?mediaType=MOVIE` |

---

## DTOs de Request/Response

### BookRequest
```json
{
  "externalId": "string",
  "title": "string (obligatorio)",
  "descripcion": "string",
  "author": "string (obligatorio)",
  "pages": "integer (min 0)",
  "type": "MANGA | NOVEL | GRAPHIC_NOVEL",
  "state": "TO_READ | READING | COMPLETED",
  "comment": "string",
  "start": "integer (0-5)",
  "startDate": "date",
  "endDate": "date",
  "frontpage": "string (URL)"
}
```

### GameRequest
```json
{
  "externalId": "string",
  "title": "string (obligatorio)",
  "platform": "PC | PS2 | PS3 | WII_U | SWITCH",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer (1-5)",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "steamAppId": "string",
  "obtainPlatinum": "boolean"
}
```

### BoardGameRequest
```json
{
  "title": "string (obligatorio)",
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer (min 1)",
  "maxPlayers": "integer (min 1)",
  "minPlaytime": "integer (min 1)",
  "maxPlaytime": "integer (min 1)",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number (0-10)",
  "bggId": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date"
}
```

### MovieShowRequest
```json
{
  "externalId": "string (obligatorio)",
  "title": "string (obligatorio)",
  "overview": "string",
  "releaseDate": "date",
  "posterUrl": "string",
  "backdropUrl": "string",
  "voteAverage": "number",
  "mediaType": "MOVIE | TV",
  "status": "WATCHING | WATCHED | PLAN_TO_WATCH",
  "userRating": "integer (1-5)",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string"
}
```

### DeckRequest
```json
{
  "name": "string (obligatorio)",
  "description": "string",
  "commander": "string",
  "commanderColors": ["string"]
}
```

### AchievementsResponse
```json
{
  "achievements": [
    {
      "name": "string",
      "description": "string",
      "achieved": "boolean",
      "iconUrl": "string"
    }
  ],
  "totalAchievements": "integer",
  "totalAchieved": "integer",
  "percentage": "number"
}
```
