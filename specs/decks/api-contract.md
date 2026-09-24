# API Contract: Mazos Commander (Decks)

**Capability:** decks
**Spec:** `specs/decks/spec.md`
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
| GET | `/decks` | Listar mazos del usuario autenticado | ✅ Auth | ✅ |
| GET | `/decks/{id}` | Obtener mazo por ID | ✅ Auth | ✅ |
| POST | `/decks` | Crear mazo | ✅ Auth | ✅ |
| PUT | `/decks/{id}` | Actualizar mazo | ✅ Auth | ✅ |
| DELETE | `/decks/{id}` | Eliminar mazo | ✅ Auth | ✅ (204 No Content) |

### Códigos de Comandante

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/decks/colors/{colorString}/commanders` | Buscar comandantes por identidad de color | Público |
| GET | `/decks/colors/{colorString}` | Obtener códigos de colores para una identidad | Público |

### Cartas del Mazo

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| POST | `/decks/{deckId}/cards?scryfallId={id}` | Añadir carta al mazo desde Scryfall | ✅ Auth |
| DELETE | `/decks/{deckId}/cards/{cardId}` | Eliminar carta del mazo | ✅ Auth |
| GET | `/decks/{deckId}/cards` | Listar cartas del mazo | ✅ Auth |
| GET | `/decks/{deckId}/cards/others` | Lista de cartas que podrían añadirse (Scryfall, excluye las del mazo) | ✅ Auth |

### Generar PDF

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/decks/{id}/pdf` | Generar PDF del mazo (layout 16x10) | ✅ Auth |

### Validación

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/decks/{id}/status` | Obtener el estado (DRAFT/INVALID/COMPLETE) del mazo | ✅ Auth |

---

## Parámetros de Búsqueda

### Filtros Locales (GET /decks)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `page` | Integer | Página (0-based) | `?page=0` |
| `size` | Integer | Tamaño de página | `?size=20` |
| `sort` | String | Ordenación por nombre | `?sort=name,asc` |

### Peticiones de carta (POST /decks/{deckId}/cards)

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `scryfallId` | String | ID de la carta en Scryfall (query param) |

---

## DTOs

### DeckRequest (POST /decks, PUT /decks/{id})

```json
{
  "name": "string (obligatorio, min 1 char)",
  "description": "string",
  "commander": "string (requerido si colors no está vacío)",
  "commanderColors": ["BLACK", "BLUE", "GREEN", "RED", "WHITE"]
}
```

### DeckResponse (GET /decks, GET /decks/{id})

```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "commander": "string",
  "commanderColors": ["BLACK", "BLUE", "GREEN", "RED", "WHITE"],
  "cards": [DeckCardBrief, ...],
  "status": "DRAFT | INVALID | COMPLETE",
  "deckCount": "integer",
  "createdAt": "datetime",
  "updatedAt": "datetime",
  "ownerId": "string"
}
```

### DeckCardBrief

```json
{
  "id": "string",
  "name": "string",
  "manaCost": "string",
  "type": "string",
  "colors": ["string"],
  "rarity": "string",
  "isFoil": "boolean",
  "quantity": "integer",
  "imageUrl": "string",
  "artCropUrl": "string",
  "scryfallId": "string"
}
```

### CardAddRequest

```json
{
  "scryfallId": "string (obligatorio)"
}
```

### CardAddResponse

```json
{
  "id": "string",
  "name": "string",
  "imageUrl": "string",
  "scryfallId": "string"
}
```

### CommanderListResponse

```json
{
  "commanders": [
    {
      "name": "string",
      "manaCost": "string",
      "type": "string",
      "colors": ["string"],
      "rarity": "string",
      "imageUrl": "string"
    }
  ]
}
```

### ColorCodesResponse

```json
{
  "codes": {
    "BLACK": "B",
    "BLUE": "U",
    "GREEN": "G",
    "RED": "R",
    "WHITE": "W"
  }
}
```

### CardTreeResponse (GET /decks/{deckId}/cards/others)

```json
{
  "otherCards": [
    {
      "name": "string",
      "manaCost": "string",
      "type": "string",
      "rarity": "string",
      "imageUrl": "string",
      "scryfallId": "string"
    }
  ],
  "totalCards": "integer"
}
```

### DeckStatusResponse (GET /decks/{id}/status)

```json
{
  "status": "DRAFT | INVALID | COMPLETE",
  "reason": "string (si invalido)"
}
```

### PagedResponse<DeckResponse>

```json
{
  "content": [DeckResponse, ...],
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
| 200 | OK — lista obtenida, mazo encontrado, actualizado, propiedades devueltas |
| 201 | Created — mazo creado, carta añadida |
| 204 | No Content — mazo/carta eliminados |
| 400 | Bad Request — datos inválidos (ver ValidationService) |
| 404 | Not Found — mazo/carta no encontrada |
| 401 | No autenticado |
| 403 | Forbidden — intento de acceder a mazo de otro usuario |
| 409 | Conflict — conflicto detectado (ej: intentar añadir 202 cartas cuando el límite es 200) |
| 502 | Bad Gateway — error en API Scryfall |

---

## Errores

### DeckNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Mazo no encontrado",
  "path": "/api/v1/decks/{id}"
}
```

### DeckHasMaxCardsException (409)
```json
{
  "timestamp": "...",
  "status": 409,
  "error": "Conflict",
  "message": "El mazo ya tiene el máximo de cartas permitidas (202)",
  "path": "/api/v1/decks/{id}/cards"
}
```

### DeckHasThisCardException (409)
```json
{
  "timestamp": "...",
  "status": 409,
  "error": "Conflict",
  "message": "Esta carta ya está en el mazo (o es el comandante)",
  "path": "/api/v1/decks/{id}/cards"
}
```

### TooManyExceptionsForList (400)
```json
{
  "timestamp": "...",
  "status": 400,
  "error": "Bad Request",
  "message": "El mazo no puede tener más de 1 excepción del mismo tipo de carta",
  "path": "/api/v1/decks/{id}/cards"
}
```

### UnauthenticatedException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "No autenticado",
  "path": "/api/v1/decks"
}
```

### UnauthorizedException (403)
```json
{
  "timestamp": "...",
  "status": 403,
  "error": "Forbidden",
  "message": "No tienes permisos para acceder a este mazo",
  "path": "/api/v1/decks/{id}"
}
```

---

## Notas

- **Ownership:** Los mazos pertenecen a un usuario. GET/{id} existe SOLO si el usuario autenticado es el propietario. GET /decks lista SOLO los mazos del usuario autenticado.
- **Estado del mazo:** El endpoint GET /decks/{id}/status evalúa el estado usando DeckValidator.
  - **DRAFT:** < 200 cartas, o condiciones de crear una mazo no cumplidas
  - **INVALID:** >= 200 cartas pero alguna regla no cumplida (ej: no hay comandante, identidad de color incorrecta, excepción duplicada)
  - **COMPLETE:** >= 200 cartas con todas las reglas cumplidas
- **Comandantes:** endpoint GET /decks/colors/{colorString}/commanders busca en Scryfall cartas con `is:commander` y `coloridentity:{colors}`.
- **PDF:** se genera el layout 16x10 del mazo + comandante. Ver ADR.
- **Scryfall:** todas las operaciones de carta usan Scryfall para datos de carta. No hay endpoint para crear/modificar carta directamente.
- **Excepciones de mazo:** las reglas de excepción son específicas del formato Commander. Ver Dessarrollo de DeckValidator.
