# Base de Datos

## MongoDB

Se usa MongoDB como base de datos principal por su flexibilidad de esquemas, ideal para colecciones con atributos variables por entidad.

### Ventajas

- **Esquemas flexibles:** Cada entidad puede tener campos diferentes
- **Escalabilidad horizontal:** Sharding y replicación
- **Integración con Spring Data:** Repositorios listos para usar
- **JSON nativo:** Alineado con el formato de APIs REST

## Colecciones

| Colección | Descripción |
|-----------|-------------|
| `books` | Libros de la colección |
| `games` | Videojuegos de la colección |
| `board_games` | Juegos de mesa (fase 3) |
| `magic_cards` | Cartas Magic (fase 4) |
| `movies_shows` | Películas y series (fase 5) |
| `users` | Usuarios (fase auth) |

## Índices

### Books
- `id` (PK)
- `externalId` (único, para evitar duplicados)
- `state` (para filtros)
- `title` (para búsquedas de texto)

### Games
- `id` (PK)
- `externalId` (único)
- `status` (para filtros)
- `title` (para búsquedas de texto)
