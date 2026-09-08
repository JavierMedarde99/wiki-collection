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

## Endpoints de Juegos de Mesa

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/boardgames` | Listar juegos de mesa con paginación y filtros |
| GET | `/api/boardgames/{id}` | Obtener juego de mesa por ID |
| POST | `/api/boardgames` | Crear juego de mesa |
| PUT | `/api/boardgames/{id}` | Actualizar juego de mesa |
| DELETE | `/api/boardgames/{id}` | Eliminar juego de mesa (204 No Content) |
| GET | `/api/boardgames/search?name={query}` | Buscar en BoardGameGeek JSON |

## Endpoints de Magic: The Gathering

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/magic` | Listar cartas con paginación y filtros |
| GET | `/api/magic/{id}` | Obtener carta por ID |
| POST | `/api/magic` | Crear carta |
| PUT | `/api/magic/{id}` | Actualizar carta |
| DELETE | `/api/magic/{id}` | Eliminar carta (204 No Content) |
| GET | `/api/magic/search?name={query}` | Buscar en Scryfall |

## Filtros de Juegos de Mesa

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Buscar por título | `?name=catan` |
| `status` | Enum | Filtrar por estado (OWNED, WISHLIST, PREVIOUSLY_OWNED, FOR_TRADE) | `?status=OWNED` |
| `minPlayers` | Integer | Filtrar por mínimo de jugadores | `?minPlayers=2` |
| `maxPlayers` | Integer | Filtrar por máximo de jugadores | `?maxPlayers=4` |
