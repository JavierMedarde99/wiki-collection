# API Reference

## Base URL

```
http://localhost:8080/api
```

## Estructura

| Sección | Descripción |
|---------|-------------|
| [Endpoints](./05.1-usuarios.md) | Endpoints de usuarios |
| [Autenticación](./05.2-autenticacion.md) | Endpoints de auth |
| [APIs Externas](./externas/) | Integración con APIs externas |

## APIs Externas

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [externas-books](./externas/externas-books.md) | Google Books API | ✅ Fase 1 |
| [externas-videogames](./externas/externas-videogames.md) | RAWG + FreeToGame | ✅ Fase 2 |
| [externas-boardgames](./externas/externas-boardgames.md) | BoardGameGeek (planificado) | 📋 Fase 3 |
| [externas-magic](./externas/externas-magic.md) | Scryfall (planificado) | 📋 Fase 4 |
| [externas-movies](./externas/externas-movies.md) | TMDB (planificado) | 📋 Fase 5 |

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 404 | Not Found |
| 500 | Server Error |

## Paginación

Los endpoints que devuelven listas soportan paginación Spring Data:

```
GET /api/books?page=0&size=20&sort=title,asc
GET /api/games?page=0&size=20&sort=title,asc
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

## Filtros de Libros

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título (LIKE case-insensitive) | `?name=harry` |
| `author` | String | Buscar por autor (LIKE case-insensitive) | `?author=rowling` |
| `type` | Enum | Filtrar por tipo (MANGA, NOVEL, GRAPHIC_NOVEL) | `?type=MANGA` |
| `state` | Enum | Filtrar por estado (TO_READ, READING, COMPLETED) | `?state=READING` |

## Filtros de Juegos

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=witcher` |
| `platform` | Enum | Filtrar por plataforma (PC, PS2, PS3, WII_U, SWITCH) | `?platform=PC` |
| `status` | Enum | Filtrar por estado (PLAYING, COMPLETED, WISHLIST, ABANDONED) | `?status=PLAYING` |

## Endpoints de Libros

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/books` | Listar libros con paginación y filtros |
| GET | `/api/books/{id}` | Obtener libro por ID |
| POST | `/api/books` | Crear libro |
| PUT | `/api/books/{id}` | Actualizar libro |
| DELETE | `/api/books/{id}` | Eliminar libro (204 No Content) |
| GET | `/api/books/search?name={query}` | Buscar en Google Books API |

## Endpoints de Juegos

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/games` | Listar juegos con paginación y filtros |
| GET | `/api/games/{id}` | Obtener juego por ID |
| POST | `/api/games` | Crear juego |
| PUT | `/api/games/{id}` | Actualizar juego |
| DELETE | `/api/games/{id}` | Eliminar juego (204 No Content) |
| GET | `/api/games/search?name={query}` | Buscar en RAWG (fallback a FreeToGame) |
