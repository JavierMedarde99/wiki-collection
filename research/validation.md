# Validación de Datos — Wiki-Collection

Los DTOs de entrada se validan usando `jakarta.validation` (annotations + implementación `spring-boot-starter-validation`). La validación ocurre en los DTOs y en los services cuando es necesario.

## Validaciones por Entity

### BookRequest

| Campo | Regla | Mensaje de error |
|-------|-------|-----------------|
| `title` | @NotBlank, min 1, max 300 | "El título es obligatorio (1-300 caracteres)" |
| `descripcion` | max 2000 | "La descripción no puede superar 2000 caracteres" |
| `author` | @NotBlank, min 1, max 200 | "El autor es obligatorio (1-200 caracteres)" |
| `pages` | @Min(0) | "Las páginas deben ser >= 0" |
| `start` | @Min(0), @Max(5) | "La puntuación debe ser entre 0 y 5" |

### GameRequest

| Campo | Regla | Mensaje de error |
|-------|-------|-----------------|
| `title` | @NotBlank, min 1, max 300 | "El título es obligatorio (1-300 caracteres)" |
| `userRating` | @Min(1), @Max(5) | "La puntuación debe ser entre 1 y 5" |
| `start` | @Min(0), @Max(5) | "La puntuación debe ser entre 0 y 5" |

### BoardGameRequest

| Campo | Regla | Mensaje de error |
|-------|-------|-----------------|
| `title` | @NotBlank, min 1, max 300 | "El título es obligatorio (1-300 caracteres)" |
| `minPlayers` | @Min(1) | "Mínimo de jugadores debe ser >= 1" |
| `maxPlayers` | @Min(1) | "Máximo de jugadores debe ser >= 1" |
| `minPlaytime` | @Min(1) | "Mínimo de tiempo debe ser >= 1 minuto" |
| `maxPlaytime` | @Min(1) | "Máximo de tiempo debe ser >= 1 minuto" |

### MagicCardRequest

| Campo | Regla | MensajE de error |
|-------|-------|-----------------|
| `name` | @NotBlank, min 1, max 200 | "El nombre es obligatorio (1-200 caracteres)" |

### DeckRequest

| Campo | Regla | Mensaje de error |
|-------|-------|-----------------|
| `name` | @NotBlank, min 1, max 100 | "El nombre es obligatorio (1-100 caracteres)" |
| `commander` | @NotBlank cuando colors no está vacío | "El comandante es obligatorio si se especifican colores" |

### MovieShowRequest

| Campo | Regla | Mensaje de error |
|-------|-------|-----------------|
| `title` | @NotBlank, min 1, max 300 | "El título es obligatorio (1-300 caracteres)" |
| `userRating` | @Min(1), @Max(5) | "La puntuación debe ser entre 1 y 5" |
| `start` | @Min(0), @Max(5) | "La puntuación debe ser entre 0 y 5" |

### Authorization

| Contexto | Regla |
|----------|-------|
| Acciones protegidas (POST/PUT/DELETE en /books, /games, /magic, /decks, /movieShows, /me/visibility) | Requieren token válido (Bearer) |
| GET endpoints públicos | No requieren token (libres) |
| GET endpoints de colecciones de otro usuario | Devuelve 403 Forbidden |

---

## Respuestas de error de validación

Cuando un DTO no pasa la validación, el backend devuelve 400 Bad Request con los errores por campo:

```json
{
  "timestamp": "...",
  "status": 400,
  "error": "Bad Request",
  "message": "Datos inválidos",
  "errors": {
    "title": ["El título es obligatorio (1-300 caracteres)"],
    "author": ["El autor es obligatorio (1-200 caracteres)"],
    "pages": ["Las páginas deben ser >= 0"]
  },
  "path": "/api/v1/books"
}
```

Si el error es por una sola regla violada en un campo, el mensaje de error se incluye en el array `errors[field]`.

Si hay un error global (no por campo, ej: conflicto de externalId), se devuelve con `errors` vacío y el mensaje en `message`.

---

## Notas

- `startDate` y `endDate` son campos opcionales (no anotados con @NotNull) y se validan manualmente en el service si ambos están presentes (endDate no puede ser anterior a startDate).
- El DTO de response no tiene validaciones (solo lectura).
- Los servicios que requieren validación adicional (ej: que startDate < endDate) lo hacen manualmente en el service, lanzando IllegalArgumentException con el mensaje adecuado.
