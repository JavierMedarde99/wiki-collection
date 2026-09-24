# API Reference

## Base URL

```
http://localhost:8080/api/v1
```

## Estructura de Specs

Esta página es el índice de los contratos de API. Cada capability tiene su propio `api-contract.md` en `specs/{capability}/api-contract.md`.

| Capability | Spec | API Contract |
|------------|------|-------------|
| Libros | `specs/books/spec.md` | `specs/books/api-contract.md` |
| Videojuegos | `specs/games/spec.md` | `specs/games/api-contract.md` |
| Juegos de Mesa | `specs/board-games/spec.md` | `specs/board-games/api-contract.md` |
| Magic: The Gathering | `specs/magic-cards/spec.md` | `specs/magic-cards/api-contract.md` |
| Mazos Commander | `specs/decks/spec.md` | `specs/decks/api-contract.md` |
| Películas/Series | `specs/movie-shows/spec.md` | `specs/movie-shows/api-contract.md` |
| Autenticación | `specs/auth/spec.md` | `specs/auth/api-contract.md` |
| Colecciones Personales | `specs/user-preferences/spec.md` | `specs/user-preferences/api-contract.md` |

## APIs Externas

Cada API externa tiene su documentación en `research/external-apis/`:

| Archivo | API | Estado |
|---------|-----|--------|
| `research/external-apis/google-books.md` | Google Books API | ✅ Fase 1 |
| `research/external-apis/rawg.md` | RAWG Video Games Database | ✅ Fase 2 |
| `research/external-apis/free-to-game.md` | FreeToGame API | ✅ Fase 2 (fallback) |
| `research/external-apis/steam.md` | Steam Web API (logros) | ✅ Fase 2 |
| `research/external-apis/boardgamegeek.md` | BoardGameGeek XML API 2 | ✅ Fase 3 |
| `research/external-apis/scryfall.md` | Scryfall | ✅ Fase 4 |
| `research/external-apis/tmdb.md` | TMDB | ✅ Fase 5 |
| `research/external-apis/catbox.md` | Catbox.moe | ✅ Fase 6 |
| `research/external-apis/steam-api-key-guide.md` | Cómo obtener Steam API Key | ✅ Fase 2 |

## Códigos de Estado Comunes

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | No autenticado |
| 403 | Forbidden (operación no permitida) |
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
GET /api/v1/decks?page=0&size=20&sort=name,asc
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

## Filtro `owner` (Fase 9+)

Todos los endpoints GET de listado aceptan parámetro `owner`:

| Valor | Descripción | Auth requerido |
|-------|-------------|----------------|
| `mine` | Solo elementos del usuario autenticado (default) | Sí |
| `other` | Solo elementos públicos de otros usuarios | Sí |
| `all` | Todos (propios + públicos de otros) | Sí |

## Códigos de Error

Todas las respuestas de error siguen el formato:

```json
{
  "timestamp": "2024-01-01T12:00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Libro no encontrado",
  "path": "/api/v1/books/123"
}
```
