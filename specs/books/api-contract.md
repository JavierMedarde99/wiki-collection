# API Contract: Libros (Books)

**Capability:** books
**Spec:** `specs/books/spec.md`
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
| GET | `/books` | Listar libros con paginación y filtros | Público | ✅ |
| GET | `/books/{id}` | Obtener libro por ID | Público | ✅ |
| POST | `/books` | Crear libro | ✅ Auth | ✅ |
| PUT | `/books/{id}` | Actualizar libro | ✅ Auth | ✅ |
| DELETE | `/books/{id}` | Eliminar libro | ✅ Auth | ✅ (204 No Content) |

### Búsqueda Externa

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/books/search?name={query}` | Buscar en Google Books API por título | Público |
| GET | `/books/search?isbn={isbn}` | Buscar en Google Books API por ISBN-13 | Público |
| GET | `/books/search?author={author}` | Buscar en Google Books API por autor | Público |

---

## Parámetros de Búsqueda

### Filtros Locales (GET /books)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `page` | Integer | Página (0-based) | `?page=0` |
| `size` | Integer | Tamaño de página | `?size=20` |
| `sort` | String | Ordenación (field,asc/desc) | `?sort=title,asc` |
| `name` | String | Búsqueda por título (LIKE) | `?name=harry` |
| `author` | String | Búsqueda por autor | `?author=rowling` |
| `type` | Enum | Filtrar por tipo: MANGA, NOVEL, GRAPHIC_NOVEL | `?type=MANGA` |
| `state` | Enum | Filtrar por estado: TO_READ, READING, COMPLETED | `?state=READING` |
| `owner` | String | Filtro de visibilidad (Fase 9+): `mine`, `other`, `all` | `?owner=other` |

### Búsqueda Externa (GET /books/search)

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `name` | String | Título a buscar en Google Books | `?name=harry+potter` |
| `isbn` | String | ISBN-13 a buscar | `?isbn=9788498382671` |
| `author` | String | Autor a buscar | `?author=j+k+rowling` |

---

## DTOs

### BookRequest (POST /books, PUT /books/{id})

```json
{
  "externalId": "string",
  "title": "string (obligatorio)",
  "descripcion": "string",
  "author": "string (obligatorio)",
  "pages": "integer (min 0)",
  "type": "MANGA | NOVEL | GRAPHIC_NOVEL",
  "state": "TO_READ | READING | COMPLETED",
  "comment": "string",
  "start": "integer (0-5)",
  "startDate": "date",
  "endDate": "date",
  "frontpage": "string (URL)"
}
```

### BookResponse (GET /books, GET /books/{id})

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "descripcion": "string",
  "author": "string",
  "pages": "integer",
  "type": "MANGA | NOVEL | GRAPHIC_NOVEL",
  "state": "TO_READ | READING | COMPLETED",
  "comment": "string",
  "start": "integer",
  "startDate": "date",
  "endDate": "date",
  "frontpage": "string",
  "ownerId": "string"
}
```

### PagedResponse<BookResponse>

```json
{
  "content": [BookResponse, ...],
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
| 200 | OK — lista obtenida, libro encontrado, actualizado |
| 201 | Created — libro creado |
| 204 | No Content — libro eliminado |
| 400 | Bad Request — datos inválidos |
| 404 | Not Found — libro no encontrado |
| 409 | Conflicto — externalId duplicado (BookConflictException) |
| 401 | No autenticado (para POST/PUT/DELETE) |

---

## Errores

### BookNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Libro no encontrado",
  "path": "/api/v1/books/{id}"
}
```

### BookConflictException (409)
```json
{
  "timestamp": "...",
  "status": 409,
  "error": "Conflict",
  "message": "Ya existe un libro con este externalId",
  "path": "/api/v1/books"
}
```

### UnauthenticatedException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "No autenticado",
  "path": "/api/v1/books"
}
```

---

## Notas

- **author es string singular** — los autores de Google Books vuelven como lista; el backend normaliza a `authors[0]`
- **externalId es único** — no se pueden duplicar libros por externalId de Google Books
- **Búsqueda externa sin auth** — los endpoints `/search` son públicos, no requieren autenticación
