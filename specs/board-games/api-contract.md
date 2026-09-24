# API Contract: Juegos de Mesa (Board Games)

**Capability:** board-games
**Spec:** `specs/board-games/spec.md`
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
| GET | `/boardgames` | Listar juegos de mesa con paginación y filtros | Público | ✅ |
| GET | `/boardgames/{id}` | Obtener juego de mesa por ID | Público | ✅ |
| POST | `/boardgames` | Crear juego de mesa | ✅ Auth | ✅ |
| PUT | `/boardgames/{id}` | Actualizar juego de mesa | ✅ Auth | ✅ |
| DELETE | `/boardgames/{id}` | Eliminar juego de mesa | ✅ Auth | ✅ (204 No Content) |

### Búsqueda Externa

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/boardgames/search?name={query}` | Buscar en BoardGameGeek (XML API) | Público |

---

## Parámetros de Búsqueda

### Filtros Locales (GET /boardgames)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `page` | Integer | Página (0-based) | `?page=0` |
| `size` | Integer | Tamaño de página | `?size=20` |
| `sort` | String | Ordenación (field,asc/desc) | `?sort=title,asc` |
| `name` | String | Búsqueda por título (LIKE) | `?name=catan` |
| `status` | Enum | Filtrar por estado: OWNED, WISHLIST | `?status=OWNED` |
| `owner` | String | Filtro de visibilidad (Fase 9+): `mine`, `other`, `all` | `?owner=other` |

### Búsqueda Externa (GET /boardgames/search)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Título a buscar en BGG | `?name=catan` |

---

## DTOs

### BoardGameRequest (POST /boardgames, PUT /boardgames/{id})

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

### BoardGameResponse (GET /boardgames, GET /boardgames/{id})

```json
{
  "id": "string",
  "title": "string",
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer",
  "maxPlayers": "integer",
  "minPlaytime": "integer",
  "maxPlaytime": "integer",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number",
  "bggId": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date",
  "ownerId": "string"
}
```

### BoardGameSearchResponse (resultado de búsqueda externa — BGG XML)

```json
{
  "bggId": "string",
  "title": "string",
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer",
  "maxPlayers": "integer",
  "minPlaytime": "integer",
  "maxPlaytime": "integer",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number",
  "externalSource": "BGG"
}
```

### PagedResponse<BoardGameResponse>

```json
{
  "content": [BoardGameResponse, ...],
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
| 502 | Bad Gateway — error en API externa (BGG XML) |
| 503 | Service Unavailable — BGG devuelve 202 Accepted (procesando) |

---

## Errores

### BoardGameNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Juego de mesa no encontrado",
  "path": "/api/v1/boardgames/{id}"
}
```

### UnauthenticatedException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "No autenticado",
  "path": "/api/v1/boardgames"
}
```

---

## Notas

- **BggId es único** — no se pueden duplicar juegos por ID de BGG
- **status solo tiene 2 valores** — OWNED y WISHLIST (no PREVIOUSLY_OWNED ni FOR_TRADE, ver ADR-012)
- **Búsqueda externa sin auth** — el endpoint `/search` es público
- **BGG devuelve 202 Accepted** — el cliente hace reintentos con delay de 2s mientras BGG procesa la búsqueda
- **Parseo XML** — BGG usa XML, parseado con Jackson XML en `BoardGameXmlMapper`
