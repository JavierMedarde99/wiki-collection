# API Reference — Backend

## Base URL

```
http://localhost:3000/api
```

## Books Endpoints

### GET /api/books
Obtener todos los libros de la colección.

**Query params:**
- `status` — Filtrar por estado (`reading`, `completed`, `wishlist`, `abandoned`)
- `tag` — Filtrar por tag
- `sort` — Campo de ordenación (default: `-dateAdded`)
- `page` — Página (default: 1)
- `limit` — Items por página (default: 20)

**Response:**
```json
{
  "data": [ { ...book }, ... ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 42,
    "pages": 3
  }
}
```

### GET /api/books/:id
Obtener un libro por ID.

### POST /api/books
Añadir un libro a la colección.

**Body:**
```json
{
  "title": "string (required)",
  "authors": ["string"],
  "isbn": "string",
  "publisher": "string",
  "publishedDate": "date",
  "description": "string",
  "pageCount": 0,
  "categories": ["string"],
  "coverImage": "string (url)",
  "language": "string",
  "status": "wishlist | reading | completed | abandoned",
  "userRating": 1-5,
  "notes": "string",
  "tags": ["string"],
  "externalSource": "google_books | open_library",
  "externalId": "string"
}
```

### PUT /api/books/:id
Actualizar un libro existente.

### DELETE /api/books/:id
Eliminar un libro de la colección.

### GET /api/books/search?q={query}
Buscar libros en Google Books API (proxy del backend).

**Response:**
```json
{
  "source": "google_books",
  "results": [
    {
      "externalId": "string",
      "title": "string",
      "authors": ["string"],
      "isbn": "string",
      "coverImage": "string",
      "description": "string",
      "publishedDate": "string",
      "pageCount": 0,
      "publisher": "string"
    }
  ]
}
```

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 400 | Bad Request |
| 404 | Not Found |
| 500 | Server Error |
