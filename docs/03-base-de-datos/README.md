# Base de Datos

## MongoDB

Se usa MongoDB como base de datos principal por su flexibilidad de esquemas, ideal para colecciones con atributos variables por entidad.

### Ventajas

- **Esquemas flexibles:** Cada entidad puede tener campos diferentes
- **Escalabilidad horizontal:** Sharding y replicación
- **Integración con Spring Data:** Repositorios listos para usar
- **JSON nativo:** Alineado con el formato de APIs REST

## Colecciones

| Colección | Descripción | Documento Entity |
|-----------|-------------|------------------|
| `books` | Libros de la colección | BookEntity |
| `games` | Videojuegos de la colección | GameEntity |
| `board_games` | Juegos de mesa | BoardGameEntity |
| `magic_cards` | Cartas Magic | MagicCardEntity |
| `movie_shows` | Películas y series | MovieShowEntity |
| `decks` | Mazos Commander | DeckEntity |

## Índices

### Books
- `id` (PK, automático)
- `externalId` (búsqueda de duplicados)
- `state` (para filtros)
- `title` (para búsquedas de texto)

### Games
- `id` (PK, automático)
- `externalId` (búsqueda de duplicados)
- `status` (para filtros)
- `title` (para búsquedas de texto)

### Board Games
- `id` (PK, automático)
- `bggId` (búsqueda por ID externo BGG)
- `status` (para filtros)
- `title` (para búsquedas de texto)

### Magic Cards
- `id` (PK, automático)
- `name` (para búsquedas de texto)

### Movie Shows
- `id` (PK, automático)
- `externalId` (búsqueda de duplicados, unique)
- `title` (para búsquedas de texto)
- `status` (para filtros)

### Decks
- `id` (PK, automático)
