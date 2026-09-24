# Futuro: Géneros para Colecciones (Genres for Collections)

**Fase:** 26 (no implementada)

## Descripción

Añadir soporte para géneros en las colecciones, permitiendo al usuario filtrar y organizar sus items por género.

### Ideas
- Para cada item (libro, juego, juego de mesa, película/serie), el usuario puede asignar uno o más géneros.
- Los géneros serían una lista cerrada por tipo de item:
  - Libros: Ficción, No Ficción, Fantasía, Ciencia Ficción, Misterio, Romance, Terror, Histórico, Biografía, etc.
  - Videojuegos: Acción, Aventura, RPG, Estrategia, Simulación, Deportes, Carreras, Puzzle, Horror, Survival, etc.
  - Juegos de mesa: Eurogame, Ameritrash, Wargame, Worker Placement, Deck Building, 4X, Party Game, etc.
  - Películas/Series: Acción, Comedia, Drama, Terror, Ciencia Ficción, Fantasía, Thriller, Documental, etc.
- El usuario puede filtrar su colección por género (ej: "Mostrar solo libros de Fantasía").
- Posible página de estadísticas: "Mis libros por género" (gráfico de pastel).

## Capacidades afectadas

- `books`, `games`, `board-games`, `movie-shows` — Nuevos campos de género en cada entidad.
- Posible nueva entidad `Genre` si queremos normalizar los géneros (evitar duplicados, poder añadir géneros en el futuro sin migrar la BD).

## Diseño de datos sugerido (normalizado)

```json
// Entity Genre
{
  "id": "...",
  "name": "Fantasía",
  "type": "BOOK"  // BOOK, GAME, BOARD_GAME, MOVIE_SHOW
}
```

```json
// Book con géneros
{
  "id": "...",
  "title": "...",
  "genres": [
    { "id": "...", "name": "Fantasía", "type": "BOOK" },
    { "id": "...", "name": "Fantasía Juvenil", "type": "BOOK" }
  ]
}
```

## Estrategia de implementación

1. **Fase 1:** añadir campos de género como arrays de strings en cada entidad (sin normalización). Más simple, pero con posibles duplicados ("Fantasía" vs "fantasía" vs "Fantasía ").
2. **Fase 2:** normalizar géneros en entidad separada `Genre`, con previsualización de géneros existentes al añadir/editar un item.

## APIs necesarias

N/A — no requiere APIs externas. Los géneros se asignan manualmente por el usuario.

## Consideraciones de diseño

- ¿Permitimos géneros personalizados por el usuario, o solo una lista cerrada?
- ¿Los géneros son por idioma? (ej: "Science Fiction" vs "Ciencia Ficción")
- ¿Cómo afecta a la búsqueda y paginación?

## Prioridad: Media

Los géneros mejoran la organización de la colección, pero no son esenciales para el uso básico.

---

*Ver también: fase 27 (Géneros específicos por tipo de colección).*
