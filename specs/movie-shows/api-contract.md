# API Contract: Películas y Series (Movie Shows)

**Capability:** movie-shows
**Spec:** `specs/movie-shows/spec.md`
**Estado:** ✅ Completada

---

## Base URL

```
http://localhost:8080/api/v1
```

---

## Endpoints

### CRUD

| Método | Endpoint | Descripción | Auth | Estados |
|--------|----------|-------------|------|---------|
| GET | `/movieshows` | Listar películas/series con paginación y filtros | Público | ✅ |
| GET | `/movieshows/{id}` | Obtener película/serie por ID | Público | ✅ |
| POST | `/movieshows` | Crear película/serie | ✅ Auth | ✅ |
| PUT | `/movieshows/{id}` | Actualizar película/serie | ✅ Auth | ✅ |
| DELETE | `/movieshows/{id}` | Eliminar película/serie | ✅ Auth | ✅ (204 No Content) |

### Búsqueda Externa (TMDB)

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/movieshows/search?name={query}&type={type}` | Buscar en TMDB (por nombre + tipo) | Público |
| GET | `/movieshows/images/{id}` | Imágenes de película/serie desde TMDB | Público |

### Logro de visualización

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/movieshows/{msId}/completed?userId={userId}` | Obtener el logro de visualización de una película/serie para un usuario | Público |

---

## Parámetros de Búsqueda

### Filtros Locales (GET /movieshows)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `page` | Integer | Página (0-based) | `?page=0` |
| `size` | Integer | Tamaño de página | `?size=20` |
| `sort` | String | Ordenación (field,asc/desc) | `?sort=title,asc` |
| `name` | String | Búsqueda por título (LIKE) | `?name=inception` |
| `mediaType` | Enum | Filtrar por tipo: MOVIE, TV | `?mediaType=MOVIE` |
| `status` | Enum | Filtrar por estado: WATCHING, WATCHED, PLAN_TO_WATCH | `?status=WATCHED` |
| `owner` | String | Filtro de visibilidad (Fase 9+): `mine`, `other`, `all` | `?owner=other` |

### Búsqueda Externa (GET /movieshows/search)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Título a buscar en TMDB | `?name=inception` |
| `type` | Enum | Tipo: MOVIE o TV (por defecto MOVIE) | `?name=breaking+bad&type=TV` |

---

## DTOs

### MovieShowRequest (POST /movieshows, PUT /movieshows/{id})

```json
{
  "externalId": "string",
  "title": "string (obligatorio)",
  "overview": "string",
  "releaseDate": "date",
  "posterUrl": "string",
  "backdropUrl": "string",
  "voteAverage": "number",
  "mediaType": "MOVIE | TV",
  "status": "WATCHING | WATCHED | PLAN_TO_WATCH",
  "userRating": "integer (1-5 nullable)",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string (debe ser TMDB)"
}
```

### MovieShowResponse (GET /movieshows, GET /movieshows/{id})

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "overview": "string",
  "releaseDate": "date",
  "posterUrl": "string",
  "backdropUrl": "string",
  "voteAverage": "number",
  "mediaType": "MOVIE | TV",
  "status": "WATCHING | WATCHED | PLAN_TO_WATCH",
  "userRating": "integer",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "ownerId": "string"
}
```

### MovieShowSearchResponse (resultado de búsqueda externa — TMDB)

```json
{
  "id": "string",
  "title": "string",
  "overview": "string",
  "posterPath": "string",
  "backdropPath": "string",
  "releaseDate": "string",
  "voteAverage": "number",
  "mediaType": "movie | tv",
  "externalSource": "TMDB"
}
```

### MovieShowImagesResponse

```json
{
  "backdrops": [
    {
      "imageUrl": "string",
      "aspectRatio": "string",
      "voteAverage": "number"
    }
  ],
  "posters": [
    {
      "imageUrl": "string",
      "aspectRatio": "string",
      "voteAverage": "number"
    }
  ]
}
```

### PagedResponse<MovieShowResponse>

```json
{
  "content": [MovieShowResponse, ...],
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
| 200 | OK — lista obtenida, película/serie encontrada, actualizada |
| 201 | Created — película/serie creada |
| 204 | No Content — película/serie eliminada |
| 400 | Bad Request — datos inválidos |
| 404 | Not Found — película/serie no encontrada |
| 401 | No autenticado (para POST/PUT/DELETE) |
| 502 | Bad Gateway — error en API externa (TMDB) |

---

## Errores

### MovieShowNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Película/Serie no encontrada",
  "path": "/api/v1/movieshows/{id}"
}
```

### UnauthenticatedException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "No autenticado",
  "path": "/api/v1/movieshows"
}
```

---

## Notas

- **externalId es único** — no se pueden duplicar películas/series por externalId de TMDB
- **mediaType es requerido** — MOVIE o TV
- **status solo tiene 3 valores** — WATCHING, WATCHED, PLAN_TO_WATCH (no WISHLIST, ver ADR-015)
- **externalSource debe ser "TMDB"** — validado en MovieShowDtoValidator
- **Búsqueda externa sin auth** — los endpoints `/search` y `/images` son públicos
- **TMDB:** requiere API key para producción (secreto en `/api/v1/secrets/tmdb`) y permite ~40 req/segundo
- **Endpoint /completed:** devuelve el logro de visualización de un usuario específico para una película/serie (si existe). Version específico para Fase 9.
