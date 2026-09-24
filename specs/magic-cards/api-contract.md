# API Contract: Cartas Magic: The Gathering (Magic Cards)

**Capability:** magic-cards
**Spec:** `specs/magic-cards/spec.md`
**Estado:** ✅ Completada

---

## Base URL

```
http://localhost:8080/api/v1
```

## Endpoints

### CRUD (limitado — ver Notas)

| Método | Endpoint | Descripción | Auth | Estados |
|--------|----------|-------------|------|---------|
| GET | `/magic` | Listar cartas con paginación y filtros | Público | ✅ |
| GET | `/magic/{id}` | Obtener carta por ID | Público | ✅ |
| DELETE | `/magic/{id}` | Eliminar carta | ✅ Auth | ✅ (204 No Content) |
| POST | `/magic/scryfall/{scryfallId}` | Añadir carta desde Scryfall | ✅ Auth | ✅ |

### Búsqueda

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/magic/search?name={query}` | Buscar en Scryfall | Público |
| GET | `/magic/commanders?colors={colors}` | Buscar comandantes por colores | Público |

---

## Parámetros de Búsqueda

### Filtros Locales (GET /magic)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `page` | Integer | Página (0-based) | `?page=0` |
| `size` | Integer | Tamaño de página | `?size=20` |
| `sort` | String | Ordenación (field,asc/desc) | `?sort=name,asc` |
| `name` | String | Búsqueda por nombre (LIKE) | `?name=lightning` |
| `rarity` | String | Filtrar por rareza | `?rarity=rare` |
| `color` | String | Filtrar por color | `?color=R` |
| `type` | String | Filtrar por tipo | `?type=Creature` |
| `owner` | String | Filtro de visibilidad (Fase 9+): `mine`, `other`, `all` | `?owner=other` |

### Búsqueda Externa (GET /magic/search, GET /magic/commanders)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Título a buscar en Scryfall | `?name=lightning+bolt` |
| `colors` | String | Colores para comandantes (comma-separated) | `?colors=RW` |

---

## DTOs

### MagicCardResponse (GET /magic, GET /magic/{id})

```json
{
  "id": "string",
  "scryfallId": "string",
  "oracleId": "string",
  "name": "string",
  "language": "ENGLISH | SPANISH | FRENCH | GERMAN | ITALIAN | PORTUGUESE | JAPANESE | CHINESE",
  "releaseDate": "string",
  "manaCost": "string",
  "convertedManaCost": "number",
  "type": "string",
  "text": "string",
  "power": "string",
  "toughness": "string",
  "loyalty": "string",
  "colors": ["string"],
  "colorIdentity": ["string"],
  "keywords": ["string"],
  "rarity": "string",
  "setCode": "string",
  "setName": "string",
  "artist": "string",
  "frame": "string",
  "borderColor": "string",
  "layout": "string",
  "legalities": {},
  "priceUsd": "string",
  "priceEur": "string",
  "imageUrl": "string",
  "imageLargeUrl": "string",
  "artCropUrl": "string",
  "condition": "MINT | NEAR_MINT | EXCELLENT | GOOD | PLAYED | POOR",
  "isFoil": "boolean",
  "quantity": "integer",
  "notes": "string",
  "dateAdded": "datetime",
  "ownerId": "string"
}
```

### MagicCardSearchResponse (resultado de búsqueda externa — Scryfall)

```json
{
  "scryfallId": "string",
  "name": "string",
  "manaCost": "string",
  "type": "string",
  "rarity": "string",
  "setCode": "string",
  "setName": "string",
  "imageUrl": "string",
  "priceUsd": "string",
  "colors": ["string"],
  "colorIdentity": ["string"],
  "text": "string"
}
```

### PagedResponse<MagicCardResponse>

```json
{
  "content": [MagicCardResponse, ...],
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
| 200 | OK — lista obtenida, carta encontrada |
| 201 | Created — carta añadida desde Scryfall |
| 204 | No Content — carta eliminada |
| 400 | Bad Request — datos inválidos |
| 404 | Not Found — carta no encontrada |
| 401 | No autenticado (para POST/DELETE) |
| 502 | Bad Gateway — error en API externa (Scryfall) |

---

## Errores

### MagicCardNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Carta Magic no encontrada",
  "path": "/api/v1/magic/{id}"
}
```

### UnauthenticatedException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "No autenticado",
  "path": "/api/v1/magic"
}
```

---

## Notas Críticas

- **NO hay POST/PUT genéricos para Magic** — las cartas solo se pueden añadir desde Scryfall con `POST /scryfall/{scryfallId}`
- **MagicCardUseCase solo tiene:** `search`, `findById`, `addFromScryfall`, `delete` — NO `save` ni `update`
- **externalId no se usa** — la identificación única es `scryfallId` + `oracleId`
- **Búsqueda externa sin auth** — los endpoints `/search` y `/commanders` son públicos
- **Scryfall es gratuita y sin auth** — no requiere API key, pero tiene rate limit de ~10 req/segundo
- **Fuzzy search:** Scryfall soporta búsqueda fuzzy tolerante a errores (`?fuzzy=name`)
