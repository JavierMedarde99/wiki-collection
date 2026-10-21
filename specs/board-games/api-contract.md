---
type: api-contract
title: "API Contract: Juegos de Mesa (Board Games)"
description: "Endpoints CRUD, búsqueda, expansiones y estadísticas de juegos de mesa"
tags: [api, board-games, contrato]
---

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
| GET | `/boardgames/genres` | Catálogo de géneros en uso (excluye colecciones privadas) | Público | ✅ |
| GET | `/boardgames/{id}` | Obtener juego de mesa por ID | Público | ✅ |
| POST | `/boardgames` | Crear juego de mesa | ✅ Auth | ✅ |
| PUT | `/boardgames/{id}` | Actualizar juego de mesa | ✅ Auth | ✅ |
| DELETE | `/boardgames/{id}` | Eliminar juego de mesa (borra en cascada sus expansiones si no es expansión) | ✅ Auth | ✅ (204 No Content) |

### Expansiones

| Método | Endpoint | Descripción | Auth | Estados |
|--------|----------|-------------|------|---------|
| GET | `/boardgames/{id}/expansions` | Catálogo BGG de las expansiones del juego base, con `inCollection` del espectador | Público | ✅ |
| POST | `/boardgames/{id}/expansions` | Dar de alta expansiones enlazadas al base de tu colección | ✅ Auth | ✅ (201 Created) |

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
| `genre` | String (repetible) | Filtrar por género (coincide con cualquiera de la lista) | `?genre=Estrategia&genre=Familiar` |
| `owner` | String | Filtro de visibilidad (Fase 9+): `mine`, `other`, `all` | `?owner=other` |
| `type` | Enum | Filtro por tipo: `base`, `expansion`, `all` (por defecto `all`; valor desconocido ⇒ 400) | `?type=expansion` |

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
  "genres": ["string"],
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
  "dateAdded": "date",
  "personalRating": "integer (1-5)",
  "playCount": "integer (min 0)",
  "lastPlayedDate": "date",
  "difficulty": "VERY_EASY | EASY | MEDIUM | HARD | VERY_HARD",
  "acquisitionPrice": "number (min 0, máx 2 decimales)"
}
```

> `BoardGameRequest` **no** acepta `bggType`, `parentId` ni `parentTitle` (campos de solo lectura del flujo de expansiones); el `PUT` tampoco los modifica.

### BoardGameResponse (GET /boardgames, GET /boardgames/{id})

```json
{
  "id": "string",
  "title": "string",
  "genres": ["string"],
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
  "personalRating": "integer (1-5)",
  "playCount": "integer",
  "lastPlayedDate": "date",
  "difficulty": "VERY_EASY | EASY | MEDIUM | HARD | VERY_HARD",
  "acquisitionPrice": "number",
  "userOwned": { "ownerId": "string", "ownerName": "string", "username": "string" },
  "bggType": "boardgame | boardgameexpansion | null",
  "parentId": "string (id del juego base, null si es base)",
  "parentTitle": "string"
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

### BoardGameExpansionListResponse (GET /boardgames/{id}/expansions)

```json
{
  "baseGameId": "string (id local del juego base)",
  "baseBggId": "string (null si el base no tiene bggId ⇒ lista vacía)",
  "expansions": [
    {
      "bggId": "string",
      "title": "string",
      "yearPublished": "integer",
      "imageUrl": "string",
      "thumbnailUrl": "string",
      "inCollection": "boolean (del espectador; anónimo ⇒ false)"
    }
  ]
}
```

### AddExpansionsRequest (POST /boardgames/{id}/expansions)

Exactamente **una** de las dos formas (las dos o ninguna ⇒ 400; máx. 20 entradas por petición):

```json
{ "bggIds": ["string (no en blanco)"] }
```
```json
{
  "expansions": [
    {
      "bggId": "string (obligatorio, no en blanco)",
      "status": "OWNED | WISHLIST (obligatorio)",
      "acquisitionPrice": "number (min 0) — obligatorio si status = OWNED",
      "dateAdded": "date — obligatorio si status = OWNED"
    }
  ]
}
```

### AddExpansionsResponse (POST → 201 Created)

```json
{
  "created": [BoardGameResponse, ...],
  "skipped": [ { "bggId": "string", "reason": "ALREADY_IN_COLLECTION" }, ... ]
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
| 200 | OK — lista obtenida, juego encontrado, actualizado, catálogo de expansiones |
| 201 | Created — juego creado o expansiones creadas |
| 204 | No Content — juego eliminado (con sus expansiones en cascada si era base) |
| 400 | Bad Request — datos inválidos (incluye: sin altas —lista vacía— o con las dos formas, más de 20 expansiones, `bggId` en blanco, datos de adquisición incompletos, `type` desconocido) |
| 404 | Not Found — juego no encontrado |
| 401 | No autenticado (para POST/PUT/DELETE) |
| 403 | Forbidden — el juego base es de otro usuario |
| 422 | Unprocessable Entity — regla de negocio de expansiones (base no en `OWNED`, base sin `bggId`, `bggId` desconocido o que no es una expansión de ese base) |
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

### ForbiddenException (403)
```json
{
  "timestamp": "...",
  "status": 403,
  "error": "Forbidden",
  "message": "No tienes permiso sobre este recurso",
  "path": "/api/v1/boardgames/{id}/expansions"
}
```

### BoardGameExpansionException (422)
```json
{
  "timestamp": "...",
  "status": 422,
  "error": "Unprocessable Entity",
  "message": "El juego base debe estar en OWNED para añadirle expansiones; está en WISHLIST",
  "path": "/api/v1/boardgames/{id}/expansions"
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
- **`GET /{id}/expansions` es público** — `inCollection` se calcula con los registros del espectador (anónimo ⇒ todo `false`); el catálogo puro de BGG va en la caché `boardgameExpansions` (clave = `bggId` del base), los flags se resuelven fuera de la caché
- **Base sin `bggId`** ⇒ `200` con `baseBggId: null` y lista de expansiones vacía (no se llama a BGG)
- **`POST /{id}/expansions` idempotente** — lo que ya está en la colección del usuario aparece en `skipped` con `reason: "ALREADY_IN_COLLECTION"`; todo el body se valida antes de guardar nada (una expansión inválida tira la petición entera) y máximo 20 ids por petición (límite de `/thing` de BGG)
- **Alta con evicción de caché** — `boardgameList`, `boardgameDetail` y `stats` se vacían al crear expansiones
- **Campos de expansión inmutables** — `bggType`, `parentId` y `parentTitle` no se aceptan en `POST`/`PUT` ni se modifican en `PUT`
- **Borrado en cascada** — `DELETE /{id}` de un registro que no es expansión borra primero sus expansiones (`ownerId` + `parentId`) y responde igual `204`
