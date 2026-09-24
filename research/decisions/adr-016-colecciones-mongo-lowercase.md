# ADR-016: Colecciones MongoDB en minúsculas

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

MongoDB distingue mayúsculas de minúsculas en nombres de colección. El código original usaba mayúsculas (BOOKS, GAMES, etc.).

## Decisión

Renombrar todas las colecciones a minúsculas (books, games, board_games, magic_cards, movie_shows, decks).

## Consecuencias

- ✅ Consistencia con convenciones MongoDB
- ✅ Evita errores de case-sensitivity
- ❌ Requiere migración de datos existentes
