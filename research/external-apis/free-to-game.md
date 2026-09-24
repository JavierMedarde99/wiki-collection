# FreeToGame API

**Investigación para:** Fase 2 — Colección de Videojuegos (fallback)

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://www.freetogame.com/api` |
| **Auth** | No requerida |
| **Rate limit** | ~10 requests/segundo |
| **Gratis** | Sí, completamente |
| **Total juegos** | ~415 (PC + Web Browser) |
| **Documentación** | https://www.freetogame.com/api-doc |
| **Estado** | Activa (2026) |

## Endpoints

| Uso | Endpoint |
|-----|----------|
| Todos los juegos | `GET /api/games` |
| Filtrar por plataforma | `GET /api/games?platform=windows` |
| Filtrar por categoría | `GET /api/games?category=shooter` |
| Ordenar | `GET /api/games?sort-by=alphabetical` |
| Detalle de juego | `GET /api/game?id={game_id}` |
| Filtrar por tags | `GET /api/filter?tag=3d.mmorpg.fantasy` |

## Limitaciones

- ❌ No soporta búsqueda por nombre
- ❌ Catálogo limitado (~415 juegos)
- ❌ Solo detecta plataforma PC

## Rol en el Sistema

FreeToGame es el **fallback** cuando RAWG no devuelve resultados. No es la fuente principal.

## Referencias

- [FreeToGame API Documentation](https://www.freetogame.com/api-doc)
