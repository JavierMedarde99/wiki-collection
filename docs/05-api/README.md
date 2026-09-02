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
| [APIs Externas](./05.3-externas.md) | Integración con APIs externas |

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Entity |
| 500 | Server Error |

## Paginación

Los endpoints que devuelven listas soportan paginación:

```
GET /api/books?page=0&size=20&sort=dateAdded,desc
```

**Response:**
```json
{
  "content": [...],
  "page": 0,
  "size": 20,
  "totalElements": 150,
  "totalPages": 8
}
```

## Filtros

| Parámetro | Aplicación | Ejemplo |
|-----------|-----------|---------|
| `state` | Filtrar por estado | `?state=READING` |
| `status` | Filtrar por estado (juegos) | `?status=PLAYING` |
| `tag` | Filtrar por tag | `?tag=fantasy` |
| `sort` | Ordenar | `?sort=title,asc` |
