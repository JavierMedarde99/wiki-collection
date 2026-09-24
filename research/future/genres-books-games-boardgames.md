# Futuro: Géneros Específicos por Tipo de Colección (Genres per Collection Type)

**Fase:** 27 (no implementada)

## Descripción

Esta es una evolución de la fase 26 (Géneros para Colecciones). Mientras que la fase 26 añade géneros genéricos a todas las colecciones, esta fase define listas de géneros específicas para cada tipo de colección, con posiblemente una taxonomía más rica.

## Motivación

Los géneros de un libro no son los mismos que los géneros de un videojuego. Algunos géneros son específicos de un medio:

- **Libros:** subgéneros literarios (hard sci-fi, space opera, cyberpunk, high fantasy, grimdark, cozy mystery, etc.)
- **Videojuegos:** géneros de jugabilidad (metroidvania, roguelike, deckbuilder, action-rpg, tactical rpg, etc.)
- **Juegos de mesa:** mecanismos (worker placement, deck building, area control, route building, etc.)
- **Películas/Series:** géneros cinematográficos (film noir, western, musical, mockumentary, etc.)

## Implementación

Esta fase es esencialmente una extensión de la fase 26 con:

- Listas de géneros por tipo que el usuario puede seleccionar (checkbox multiselect o autocomplete).
- Posible importación de géneros desde APIs externas (ej: RAWG devuelve genres para juegos; TMDB devuelve genres para películas/series).

## APIs necesarias

- **RAWG API:** devuelve una lista de géneros por juego (endpoint `/games/{id}/details`).
- **TMDB API:** devuelve una lista de géneros por película/serie (endpoint `/movie/{id}` o `/tv/{id}` devuelve `genres` array).
- **Google Books API:** devuelve `categories` para libros (no son géneros literarios, pero se puede usar como aproximación).

## Consideraciones de diseño

- Si importamos géneros desde APIs externas, ¿cómo manejar los géneros personalizados del usuario? Necesitaríamos una lista de géneros propios + géneros importados.
- Las APIs devuelven géneros en inglés. ¿Los traducimos automáticamente o los dejamos en inglés?

## Prioridad: Baja

Más una mejoraincremental de la fase 26 que una feature standalone.

---

*Ver también: fase 26 (Géneros para Colecciones).*
