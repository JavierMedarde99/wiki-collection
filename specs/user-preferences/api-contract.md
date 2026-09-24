# API Contract: Colecciones Personales y Visibilidad (User Preferences)

**Capability:** user-preferences
**Spec:** `specs/user-preferences/spec.md`
**Estado:** ✅ Completada

---

## Base URL

```
http://localhost:8080/api/v1
```

---

## Endpoints (Fase 9)

### Visibilidad de colecciones

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/me/visibility` | Obtener la configuración de visibilidad del usuario autenticado | ✅ Auth |
| PUT | `/me/visibility` | Actualizar la configuración de visibilidad | ✅ Auth |

### Perfiles públicos

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/users/{username}/profile` | Obtener el perfil público de un usuario | Público |
| GET | `/users/{username}/collections/public` | Colecciones públicas del usuario (con paginación) | Público |

---

## Parámetros de Búsqueda

### Visibilidad (GET/PUT /me/visibility)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `booksVisibility` | Enum | `PUBLIC`, `PRIVATE` |
| `gamesVisibility` | Enum | `PUBLIC`, `PRIVATE` |
| `boardGamesVisibility` | Enum | `PUBLIC`, `PRIVATE` |
| `magicCardsVisibility` | Enum | `PUBLIC`, `PRIVATE` |
| `decksVisibility` | Enum | `PUBLIC`, `PRIVATE` |
| `movieShowsVisibility` | Enum | `PUBLIC`, `PRIVATE` |

### Colecciones públicas (GET /users/{username}/collections/public)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `page` | Integer | Página (0-based) | `?page=0` |
| `size` | Integer | Tamaño de página | `?size=20` |
| `sort` | String | Ordenación (field,asc/desc) | `?sort=title,asc` |
| `capability` | String | Filtrar por capability: `books`, `games`, `boardgames`, `magic`, `decks`, `movieshows` | `?capability=books` |
| `type` | String | Para books: `MANGA`, `NOVEL`, `GRAPHIC_NOVEL` | `?capability=books&type=MANGA` |
| `state` | String | Para books: `TO_READ`, `READING`, `COMPLETED` | `?capability=books&state=COMPLETED` |

---

## DTOs

### VisibilityRequest (PUT /me/visibility)

```json
{
  "booksVisibility": "PUBLIC | PRIVATE",
  "gamesVisibility": "PUBLIC | PRIVATE",
  "boardGamesVisibility": "PUBLIC | PRIVATE",
  "magicCardsVisibility": "PUBLIC | PRIVATE",
  "decksVisibility": "PUBLIC | PRIVATE",
  "movieShowsVisibility": "PUBLIC | PRIVATE"
}
```

### UserVisibilityResponse (GET /me/visibility)

```json
{
  "booksVisibility": "PUBLIC | PRIVATE",
  "gamesVisibility": "PUBLIC | PRIVATE",
  "boardGamesVisibility": "PUBLIC | PRIVATE",
  "magicCardsVisibility": "PUBLIC | PRIVATE",
  "decksVisibility": "PUBLIC | PRIVATE",
  "movieShowsVisibility": "PUBLIC | PRIVATE"
}
```

### PublicUserProfileResponse (GET /users/{username}/profile)

```json
{
  "username": "string",
  "createdAt": "datetime"
}
```

### PublicCollectionsPageResponse

```json
{
  "username": "string",
  "books": [
    {
      "id": "string",
      "title": "string",
      "author": "string",
      "pages": "integer",
      "type": "MANGA | NOVEL | GRAPHIC_NOVEL",
      "state": "TO_READ | READING | COMPLETED",
      "userRating": "integer",
      "startDate": "date",
      "endDate": "date",
      "thumbnailUrl": "string"
    }
  ],
  "games": [
    {
      "id": "string",
      "title": "string",
      "platform": "PC | PS2 | PS3 | WII_U | SWITCH",
      "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
      "userRating": "integer",
      "thumbnailUrl": "string"
    }
  ],
  "boardGames": [
    {
      "id": "string",
      "title": "string",
      "status": "OWNED | WISHLIST",
      "bggRating": "number",
      "thumbnailUrl": "string"
    }
  ],
  "magicCards": [],
  "decks": [
    {
      "id": "string",
      "name": "string",
      "commander": "string",
      "commanderColors": ["string"],
      "deckCount": "integer",
      "status": "DRAFT | INVALID | COMPLETE"
    }
  ],
  "movieShows": [
    {
      "id": "string",
      "title": "string",
      "mediaType": "MOVIE | TV",
      "status": "WATCHING | WATCHED | PLAN_TO_WATCH",
      "userRating": "integer",
      "posterUrl": "string"
    }
  ],
  "totalPages": 5,
  "totalElements": 100,
  "number": 0,
  "size": 20
}
```

---

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK — visibilidad obtenida/actualizada, perfil obtenido, colecciones obtenidas |
| 400 | Bad Request — datos inválidos |
| 401 | Unauthorized — no autenticado |
| 403 | Forbidden — intento de actualizar visibilidad de otro usuario |
| 404 | Not Found — usuario no encontrado |

---

## Errores

### UnauthenticatedException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "No autenticado",
  "path": "/api/v1/me/visibility"
}
```

### UserNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Usuario no encontrado",
  "path": "/api/v1/users/{username}/profile"
}
```

---

## Notas

- **Solo el propio usuario puede ver/editar su visibilidad** (GET/PUT /me/visibility). Los perfiles públicos son accesibles por cualquiera.
- **Colecciones públicas:** solo se incluyen items de capabilities que el usuario ha marcado como PUBLIC. Para PRIVATE, el item no aparece en la respuesta.
- **Cero items de un tipo:** si un usuario no tiene items de una capability, el array de ese tipo es `[]` (no se omite).
- **Perfiles públicos:** GET /users/{username}/profile devuelve solo username y createdAt (info mínima).
- **Colecciones públicas:** GET /users/{username}/collections/public devuelve colecciones paginadas.
- **Paginación:** usa PageableSpecification para construir query cuando hay filtros (por defecto no hay filtros específicos, pero puede haber filtros por capability/type/state en el futuro).
- **Frontend necesita:** para cada capability, una lista de items seguros (connected component o similar) que renderice películas de forma declarativa (ej: <MovieShowList>).
