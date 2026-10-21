---
type: api-contract
title: "API Contract: Videojuegos (Games)"
description: "Endpoints CRUD, búsqueda, contenido adicional (DLC), plataformas y logros de videojuegos"
tags: [api, games, contrato]
---

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
| GET | `/games/genres` | Catálogo de géneros (Fase 27) | Público | ✅ |
| GET | `/games/platforms` | Catálogo de plataformas desde RAWG | Público | ✅ |
| GET | `/games/{id}` | Obtener juego por ID | Público | ✅ |
| POST | `/games` | Crear juego | ✅ Auth | ✅ |
| PUT | `/games/{id}` | Actualizar juego | ✅ Auth | ✅ |
| DELETE | `/games/{id}` | Eliminar juego (borra en cascada sus DLC si es base) | ✅ Auth | ✅ (204 No Content) |

### Contenido adicional (DLC)

| Método | Endpoint | Descripción | Auth | Estados |
|--------|----------|-------------|------|---------|
| GET | `/games/{id}/additions` | Catálogo RAWG de adiciones del juego base, con `inCollection` y `alreadyLinked` del espectador | Público | ✅ |
| POST | `/games/{id}/additions` | Dar de alta contenido adicional enlazado al base (crea, enlaza o salta) | ✅ Auth | ✅ (201 Created) |

### Búsqueda Externa

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/games/search?name={query}` | Buscar en RAWG (fallback a FreeToGame). RAWG recibe `exclude_additions=true`: la búsqueda no ofrece adiciones | Público |

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
| `platform` | String (repetible) | Filtrar por plataforma (texto libre, catálogo `GET /games/platforms`); repetir el parámetro = «cualquiera de» | `?platform=pc&platform=playstation-5` |
| `status` | Enum | Filtrar por estado: PLAYING, COMPLETED, WISHLIST, ABANDONED | `?status=PLAYING` |
| `genre` | String (repetible) | Filtrar por género (coincide con cualquiera de la lista) | `?genre=RPG` |
| `owner` | String | Filtro de visibilidad (Fase 9+): `mine`, `other`, `all` | `?owner=other` |
| `type` | Enum | Filtro por tipo: `base`, `dlc`, `all` (por defecto `all`; valor desconocido ⇒ 400) | `?type=dlc` |

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
  "genres": ["string"],
  "platform": "string (obligatorio, texto libre del catálogo RAWG)",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer (1-5)",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string (RAWG | FreeToGame)",
  "steamAppId": "string",
  "acquisitionDate": "date",
  "acquisitionPrice": "number (min 0, máx 2 decimales)"
}
```

> `GameRequest` **no** acepta `parentId` ni `parentTitle` (campos de solo lectura del flujo de adiciones); el `PUT` tampoco los modifica.

### GameResponse (GET /games, GET /games/{id})

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "genres": ["string"],
  "platform": "string",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "steamAppId": "string",
  "acquisitionDate": "date",
  "acquisitionPrice": "number",
  "parentId": "string (id del juego base, null si es base)",
  "parentTitle": "string",
  "userOwned": { "ownerId": "string", "ownerName": "string", "username": "string" }
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

### GameAdditionsListResponse (GET /games/{id}/additions)

```json
{
  "baseGameId": "string (id local del juego base)",
  "baseExternalId": "string (externalId del base en RAWG)",
  "additions": [
    {
      "externalId": "string",
      "title": "string",
      "platform": "string",
      "genres": ["string"],
      "thumbnailUrl": "string",
      "releaseDate": "date",
      "externalSource": "string",
      "inCollection": "boolean (del espectador)",
      "alreadyLinked": "boolean (del espectador; ya colgaba de este base)"
    }
  ],
  "totalAvailable": "integer (total que RAWG conoce, más allá de esta página)"
}
```

### AddGameAdditionsRequest (POST /games/{id}/additions)

Exactamente **una** de las dos formas (las dos o ninguna ⇒ 400; máx. 40 entradas):

```json
{
  "externalIds": ["string (no en blanco)"],
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED (obligatorio)"
}
```
```json
{
  "additions": [
    {
      "externalId": "string (obligatorio, no en blanco)",
      "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED (obligatorio)",
      "link": "boolean — true = solo enlazar uno que ya tengo (no exige datos)",
      "acquisitionDate": "date — obligatorio fuera de WISHLIST y de los ítems link",
      "acquisitionPrice": "number (min 0, 8 dígitos + 2 decimales) — obligatorio fuera de WISHLIST y de los ítems link",
      "dateAdded": "date — obligatorio si status = PLAYING o COMPLETED",
      "dateCompleted": "date — obligatorio si status = COMPLETED"
    }
  ]
}
```

Forma legada (`externalIds` + `status`): sin datos de adquisición; en `PLAYING`/`COMPLETED` la fecha de inicio (`dateAdded`) queda en hoy.

### GameAdditionsAddResponse (POST → 201 Created)

```json
{
  "created": [GameResponse, ...],
  "linked": [GameResponse, ...],
  "skipped": [ { "externalId": "string", "reason": "ALREADY_LINKED" }, ... ]
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
| 200 | OK — lista obtenida, juego encontrado, actualizado, catálogo de adiciones |
| 201 | Created — juego creado o adiciones creadas/enlazadas |
| 204 | No Content — juego eliminado (con sus DLC en cascada si era base) |
| 400 | Bad Request — datos inválidos (incluye: sin ids o con las dos formas, ids en blanco, más de 40 adiciones, sin estado, datos de adquisición incompletos, `type` desconocido) |
| 404 | Not Found — juego no encontrado |
| 401 | No autenticado (para POST/PUT/DELETE) |
| 403 | Forbidden — el juego base es de otro usuario |
| 422 | Unprocessable Entity — regla de negocio del contenido adicional (base en WISHLIST, base sin `externalId` o no RAWG, `externalId` que no es una adición del base, ítem `link` ya no coleccionado) |
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

### ForbiddenException (403)
```json
{
  "timestamp": "...",
  "status": 403,
  "error": "Forbidden",
  "message": "No tienes permiso sobre este recurso",
  "path": "/api/v1/games/{id}/additions"
}
```

### GameAdditionsException (422)
```json
{
  "timestamp": "...",
  "status": 422,
  "error": "Unprocessable Entity",
  "message": "El juego base no viene de RAWG: no se pueden consultar sus adiciones",
  "path": "/api/v1/games/{id}/additions"
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
- **platform es texto libre** — viene del catálogo dinámico RAWG (`GET /api/v1/games/platforms`); el enum `GamePlatform` fue eliminado
- **Búsqueda con fallback** — si RAWG no devuelve resultados, se consulta FreeToGame automáticamente
- **Búsqueda externa sin auth** — los endpoints `/search` son públicos
- **Logros Steam:** requieren `steamAppId` en el juego y `steamId` del jugador como query param
- **`GET /{id}/additions` es público** — `inCollection`/`alreadyLinked` se calculan con los registros del espectador **fuera** de la caché; solo el catálogo puro de RAWG va en la caché `gameAdditions` (clave `rawg-additions:{externalId}` del base). Si el base no viene de RAWG o no tiene `externalId` ⇒ `422` (no hay «catálogo vacío»)
- **`POST /{id}/additions` → `201` con tres listas** — `created` (no la tenías), `linked` (ya la tenías suelta: solo se escribe `parentId`/`parentTitle` y conserva estado, nota, valoración, precio y fechas) y `skipped` (`ALREADY_LINKED`). Máximo 40 (una página de RAWG), ids duplicados se deduplican y los ids se verifican contra el catálogo de RAWG del base antes de guardar nada
- **Alta con evicción de caché** — `gameList`, `gameDetail` y `stats` se vacían al crear adiciones
- **Campos de vínculo inmutables** — `parentId` y `parentTitle` no se aceptan en `POST`/`PUT` ni se modifican en `PUT`
- **Búsqueda sin adiciones** — `GET /games/search` envía `exclude_additions=true` a RAWG
- **Borrado en cascada** — `DELETE /{id}` de un registro con `parentId == null` borra primero sus DLC (`ownerId` + `parentId`) y responde igual `204`
