# API Contract: Videojuegos (Games)

**Capability:** games
**Spec:** `specs/games/spec.md`
**Estado:** ✅ Completada

---

## Base URL

```
http://localhost:8080/api/v1
```

## Endpoints

### CRUD

| Método | Endpoint | Descripción | Auth | Estados |
|--------|----------|-------------|------|---------|
| GET | `/games` | Listar juegos con paginación y filtros | Público | ✅ |
| GET | `/games/{id}` | Obtener juego por ID | Público | ✅ |
| POST | `/games` | Crear juego | ✅ Auth | ✅ |
| PUT | `/games/{id}` | Actualizar juego | ✅ Auth | ✅ |
| DELETE | `/games/{id}` | Eliminar juego | ✅ Auth | ✅ (204 No Content) |

### Búsqueda Externa

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/games/search?name={query}` | Buscar en RAWG (fallback a FreeToGame) | Público |

### Logros Steam

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/games/{gameId}/achievements?steamId={id}` | Logros de un jugador (Steam) | Público |

---

## Parámetros de Búsqueda

### Filtros Locales (GET /games)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `page` | Integer | Página (0-based) | `?page=0` |
| `size` | Integer | Tamaño de página | `?size=20` |
| `sort` | String | Ordenación (field,asc/desc) | `?sort=title,asc` |
| `name` | String | Búsqueda por título (LIKE) | `?name=witcher` |
| `platform` | Enum | Filtrar por plataforma: PC, PS2, PS3, WII_U, SWITCH | `?platform=PC` |
| `status` | Enum | Filtrar por estado: PLAYING, COMPLETED, WISHLIST, ABANDONED | `?status=PLAYING` |
| `owner` | String | Filtro de visibilidad (Fase 9+): `mine`, `other`, `all` | `?owner=other` |

### Búsqueda Externa (GET /games/search)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Título a buscar en RAWG | `?name=witcher+3` |

---

## DTOs

### GameRequest (POST /games, PUT /games/{id})

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
  "externalSource": "string (RAWG | FreeToGame)",
  "steamAppId": "string",
  "obtainPlatinum": "boolean"
}
```

### GameResponse (GET /games, GET /games/{id})

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "platform": "PC | PS2 | PS3 | WII_U | SWITCH",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "steamAppId": "string",
  "obtainPlatinum": "boolean",
  "ownerId": "string"
}
```

### GameSearchResult (resultado de búsqueda externa — RAWG)

```json
{
  "id": "string",
  "title": "string",
  "description": "string",
  "genre": "string",
  "platform": "string",
  "publisher": "string",
  "developer": "string",
  "releaseDate": "string",
  "thumbnailUrl": "string",
  "externalSource": "RAWG"
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

### PagedResponse<GameResponse>

```json
{
  "content": [GameResponse, ...],
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
| 200 | OK — lista obtenida, juego encontrado, actualizado |
| 201 | Created — juego creado |
| 204 | No Content — juego eliminado |
| 400 | Bad Request — datos inválidos |
| 404 | Not Found — juego no encontrado |
| 401 | No autenticado (para POST/PUT/DELETE) |
| 502 | Bad Gateway — error en API externa (RAWG/FreeToGame/Steam) |

---

## Errores

### GameNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Juego no encontrado",
  "path": "/api/v1/games/{id}"
}
```

### UnauthenticatedException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "No autenticado",
  "path": "/api/v1/games"
}
```

---

## Notas

- **externalId es único** — no se pueden duplicar juegos por externalId
- **platform es requerido** — debe ser uno de los valores del enum (PC, PS2, PS3, WII_U, SWITCH)
- **Búsqueda con fallback** — si RAWG no devuelve resultados, se consulta FreeToGame automáticamente
- **Búsqueda externa sin auth** — los endpoints `/search` son públicos
- **Logros Steam:** requieren `steamAppId` en el juego y `steamId` del jugador como query param
