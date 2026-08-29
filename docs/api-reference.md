# API Reference — Backend

## Base URL

```
http://localhost:8080/api
```

## Books Endpoints

### GET /api/books
Obtener todos los libros de la colección.

**Query params:**
- `status` — Filtrar por estado (reading, completed, wishlist, abandoned)
- `tag` — Filtrar por tag
- `sort` — Campo de ordenación (default: dateAdded,desc)
- `page` — Página (default: 0)
- `size` — Items por página (default: 20)

**Response:** Página de libros con metadata de paginación.

### GET /api/books/{id}
Obtener un libro por ID.

### POST /api/books
Añadir un libro a la colección.

**Body:** JSON con los campos del modelo Book.

### PUT /api/books/{id}
Actualizar un libro existente.

### DELETE /api/books/{id}
Eliminar un libro de la colección.

### GET /api/books/search?q={query}
Buscar libros en Google Books API (proxy del backend).

**Response:** Lista de resultados de la API externa con campos normalizados.

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 404 | Not Found |
| 500 | Server Error |
