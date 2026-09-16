# API Reference

## Base URL

```
http://localhost:8080/api/v1
```

## Estructura

| Sección | Descripción |
|---------|-------------|
| [Endpoints de Libros](#endpoints-de-libros) | CRUD + búsqueda en Google Books |
| [Endpoints de Juegos](#endpoints-de-juegos) | CRUD + búsqueda en RAWG/FreeToGame + logros Steam |
| [Endpoints de Juegos de Mesa](#endpoints-de-juegos-de-mesa) | CRUD + búsqueda en BGG |
| [Endpoints de Magic](#endpoints-de-magic) | Listado, detalle, añadir desde Scryfall, eliminar + búsqueda |
| [Endpoints de Mazos](#endpoints-de-mazos) | CRUD + gestión cartas + status Commander |
| [Endpoints de Películas/Series](#endpoints-de-películas-series) | CRUD + búsqueda en TMDB |
| [Endpoints de Imágenes](#endpoints-de-imágenes) | Subida y eliminación de imágenes en Catbox |
| [Endpoints de Preferencias](#endpoints-de-preferencias) | Configuración de colecciones activas y visibilidad |
| [Endpoints de Perfiles Públicos](#endpoints-de-perfiles-públicos) | Ver perfiles y colecciones públicas de usuarios |
| [APIs Externas](./externas/) | Integración con APIs externas |

## APIs Externas

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [externas-books](./externas/externas-books.md) | Google Books API | ✅ Fase 1 |
| [externas-videogames](./externas/externas-videogames.md) | RAWG + FreeToGame | ✅ Fase 2 |
| [externas-steam](./externas/externas-steam.md) | Steam Web API (logros) | ✅ Fase 2 |
| [Guía: Steam API Key](./externas/steam-api-key-guide.md) | Cómo obtener tu API Key | ✅ Fase 2 |
| [externas-boardgames](./externas/externas-boardgames.md) | BoardGameGeek XML | ✅ Fase 3 |
| [externas-magic](./externas/externas-magic.md) | Scryfall | ✅ Fase 4 |
| [externas-movies](./externas/externas-movies.md) | TMDB | ✅ Fase 5 |
| [Image Hosting](./externas/externas-image-hosting.md) | Catbox.moe para subir imágenes | ✅ Fase 6 |

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 404 | Not Found |
| 409 | Conflicto (duplicado) |
| 500 | Server Error |
| 502 | Bad Gateway (API externa) |

## Paginación

Los endpoints que devuelven listas soportan paginación Spring Data:

```
GET /api/v1/books?page=0&size=20&sort=title,asc
GET /api/v1/games?page=0&size=20&sort=title,asc
GET /api/v1/boardgames?page=0&size=20&sort=title,asc
GET /api/v1/magic?page=0&size=20&sort=name,asc
GET /api/v1/movieshows?page=0&size=20&sort=title,asc
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
| GET | `/api/v1/books` | Listar libros con paginación y filtros | ✅ |
| GET | `/api/v1/books/{id}` | Obtener libro por ID | ✅ |
| POST | `/api/v1/books` | Crear libro | ✅ |
| PUT | `/api/v1/books/{id}` | Actualizar libro | ✅ |
| DELETE | `/api/v1/books/{id}` | Eliminar libro (204 No Content) | ✅ |
| GET | `/api/v1/books/search?name={query}` | Buscar en Google Books API | ✅ |

### Filtros de Libros

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título (LIKE case-insensitive) | `?name=harry` |
| `author` | String | Buscar por autor (LIKE case-insensitive) | `?author=rowling` |
| `type` | Enum | Filtrar por tipo (MANGA, NOVEL, GRAPHIC_NOVEL) | `?type=MANGA` |
| `state` | Enum | Filtrar por estado (TO_READ, READING, COMPLETED) | `?state=READING` |

### Parámetros de Visibilidad (Fase 9)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` | `?owner=other` |

---

## Endpoints de Juegos

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/v1/games` | Listar juegos con paginación y filtros | ✅ |
| GET | `/api/v1/games/{id}` | Obtener juego por ID | ✅ |
| POST | `/api/v1/games` | Crear juego | ✅ |
| PUT | `/api/v1/games/{id}` | Actualizar juego | ✅ |
| DELETE | `/api/v1/games/{id}` | Eliminar juego (204 No Content) | ✅ |
| GET | `/api/v1/games/search?name={query}` | Buscar en RAWG (fallback a FreeToGame) | ✅ |
| GET | `/api/v1/games/{gameId}/achievements?steamId={id}` | Logros de un jugador (Steam) | ✅ |

### Filtros de Juegos

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=witcher` |
| `platform` | Enum | Filtrar por plataforma (PC, PS2, PS3, WII_U, SWITCH) | `?platform=PC` |
| `status` | Enum | Filtrar por estado (PLAYING, COMPLETED, WISHLIST, ABANDONED) | `?status=PLAYING` |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` | `?owner=other` |

---

## Endpoints de Juegos de Mesa

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/v1/boardgames` | Listar juegos de mesa con paginación y filtros | ✅ |
| GET | `/api/v1/boardgames/{id}` | Obtener juego de mesa por ID | ✅ |
| POST | `/api/v1/boardgames` | Crear juego de mesa | ✅ |
| PUT | `/api/v1/boardgames/{id}` | Actualizar juego de mesa | ✅ |
| DELETE | `/api/v1/boardgames/{id}` | Eliminar juego de mesa (204 No Content) | ✅ |
| GET | `/api/v1/boardgames/search?name={query}` | Buscar en BoardGameGeek (XML API) | ✅ |

### Filtros de Juegos de Mesa

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=catan` |
| `status` | Enum | Filtrar por estado (OWNED, WISHLIST) | `?status=OWNED` |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` | `?owner=other` |

---

## Endpoints de Magic: The Gathering

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/v1/magic` | Listar cartas con paginación y filtros | ✅ |
| GET | `/api/v1/magic/{id}` | Obtener carta por ID | ✅ |
| POST | `/api/v1/magic/scryfall/{scryfallId}` | Añadir carta desde Scryfall | ✅ |
| DELETE | `/api/v1/magic/{id}` | Eliminar carta (204 No Content) | ✅ |
| GET | `/api/v1/magic/search?name={query}` | Buscar en Scryfall | ✅ |
| GET | `/api/v1/magic/commanders?colors={colors}` | Buscar comandantes por colores | ✅ |

**Nota:** El backend NO expone POST/PUT genéricos para Magic. Las cartas solo se pueden añadir desde Scryfall con `POST /scryfall/{scryfallId}`.

### Filtros de Magic

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por nombre (LIKE case-insensitive) | `?name=lightning` |
| `rarity` | String | Filtrar por rareza | `?rarity=rare` |
| `color` | String | Filtrar por color | `?color=R` |
| `type` | String | Filtrar por tipo | `?type=Creature` |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` | `?owner=other` |

---

## Endpoints de Mazos (Commander)

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | `/api/v1/decks` | Listar mazos (filtro por nombre opcional) | ✅ |
| GET | `/api/v1/decks/{id}` | Obtener mazo por ID | ✅ |
| POST | `/api/v1/decks` | Crear mazo | ✅ |
| PUT | `/api/v1/decks/{id}` | Actualizar mazo | ✅ |
| DELETE | `/api/v1/decks/{id}` | Eliminar mazo (204 No Content) | ✅ |
| POST | `/api/v1/decks/{id}/cards` | Añadir carta desde Scryfall | ✅ |
| DELETE | `/api/v1/decks/{id}/cards/{scryfallId}` | Quitar carta del mazo | ✅ |
| GET | `/api/v1/decks/{id}/status` | Estado del mazo (DRAFT, COMPLETE, INVALID) | ✅ |

### Filtros de Mazos

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Filtrar por nombre exacto | `?name=Mi Mazo` |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` | `?owner=other` |

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
| GET | `/api/v1/movieshows` | Listar películas/series con paginación y filtros | ✅ |
| GET | `/api/v1/movieshows/{id}` | Obtener película/serie por ID | ✅ |
| POST | `/api/v1/movieshows` | Crear película/serie | ✅ |
| PUT | `/api/v1/movieshows/{id}` | Actualizar película/serie | ✅ |
| DELETE | `/api/v1/movieshows/{id}` | Eliminar película/serie (204 No Content) | ✅ |
| GET | `/api/v1/movieshows/search?name={query}&mediaType={type}` | Buscar en TMDB | ✅ |

### Filtros de Películas/Series

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=matrix` |
| `status` | Enum | Filtrar por estado (WATCHING, WATCHED, PLAN_TO_WATCH) | `?status=WATCHING` |
| `mediaType` | Enum | Filtrar por tipo (MOVIE, TV) | `?mediaType=MOVIE` |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` | `?owner=other` |

---

## Endpoints de Imágenes

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| POST | `/api/v1/images/upload` | Subir imagen a Catbox (multipart, campo `file`) → 201 `{url, filename}` | ✅ |
| DELETE | `/api/v1/images/{filename}` | Eliminar en Catbox (204; requiere userhash) | ✅ |

### Upload Image

```http
POST /api/v1/images/upload
Content-Type: multipart/form-data

file: <archivo>
```

**Response:**

```json
{
  "url": "https://files.catbox.moe/abc123.jpg",
  "filename": "abc123.jpg"
}
```

**Flujo:** `POST /api/v1/images/upload` → Catbox devuelve `url` → usarla en el campo de imagen de cada entidad (`frontpage`, `thumbnailUrl`, `posterUrl`, etc.).

```bash
curl -X POST -F "file=@foto.jpg" http://localhost:8080/api/v1/images/upload
# {"url":"https://files.catbox.moe/abc123.jpg","filename":"abc123.jpg"}
```

**Validaciones:**
- Tamaño máximo: 5 MB
- MIME permitido: image/jpeg, image/png, image/gif, image/webp
- Verificación de magic bytes

**Configuración:** `catbox.api.base-url`, `catbox.userhash` (necesario solo para borrar).

---

## Endpoints de Preferencias

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/v1/preferences` | Obtener preferencias del usuario | ✅ |
| PUT | `/api/v1/preferences` | Actualizar preferencias completas | ✅ |
| PATCH | `/api/v1/preferences/active-collections` | Activar/desactivar colecciones | ✅ |
| PATCH | `/api/v1/preferences/collection-visibility` | Cambiar visibilidad de colecciones | ✅ |
| GET | `/api/v1/preferences/active-collections` | Obtener solo colecciones activas | ✅ |

### UserPreferencesResponse

```json
{
  "id": "string",
  "userId": "string",
  "activeCollections": {
    "books": true,
    "games": true,
    "boardgames": false,
    "magic": true,
    "decks": true,
    "movieshows": false
  },
  "collectionVisibility": {
    "books": "PUBLIC",
    "games": "PUBLIC",
    "boardgames": "PUBLIC",
    "magic": "PUBLIC",
    "decks": "PUBLIC",
    "movieshows": "PUBLIC"
  }
}
```

### ActiveCollectionsRequest

```json
{
  "collections": {
    "books": true,
    "games": false,
    "boardgames": true,
    "magic": true,
    "decks": true,
    "movieshows": false
  }
}
```

### CollectionVisibilityRequest

```json
{
  "visibility": {
    "books": "PUBLIC",
    "games": "PRIVATE",
    "boardgames": "PUBLIC",
    "magic": "PUBLIC",
    "decks": "PUBLIC",
    "movieshows": "PUBLIC"
  }
}
```

### Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | Preferencias obtenidas/actualizadas |
| 400 | Datos inválidos |
| 401 | No autenticado |
| 404 | Preferencias no encontradas |

---

## Endpoints de Perfiles Públicos

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/v1/users/{username}` | Perfil público de un usuario | Público |
| GET | `/api/v1/users/{username}/books` | Libros públicos de un usuario | Público |
| GET | `/api/v1/users/{username}/games` | Juegos públicos de un usuario | Público |
| GET | `/api/v1/users/{username}/boardgames` | Juegos de mesa públicos | Público |
| GET | `/api/v1/users/{username}/magic` | Cartas Magic públicas | Público |
| GET | `/api/v1/users/{username}/decks` | Mazos públicos | Público |
| GET | `/api/v1/users/{username}/movieshows` | Películas/series públicas | Público |

### PublicProfileResponse

```json
{
  "username": "string",
  "displayName": "string",
  "avatarUrl": "string",
  "bio": "string",
  "publicCollections": [
    {
      "type": "books",
      "count": 25,
      "visibility": "PUBLIC"
    }
  ]
}
```

### PublicCollectionSummary

```json
{
  "type": "books",
  "count": 25,
  "visibility": "PUBLIC"
}
```

### Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | Perfil obtenido |
| 404 | Usuario no encontrado |

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

### DeckCardRequest

```json
{
  "scryfallId": "string (obligatorio)",
  "quantity": "integer (min 1)"
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

### ImageResponse

```json
{
  "url": "https://files.catbox.moe/abc123.jpg",
  "filename": "abc123.jpg"
}
```

### ErrorResponse

```json
{
  "timestamp": "2024-01-01T12:00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Libro no encontrado",
  "path": "/api/v1/books/123"
}
```
