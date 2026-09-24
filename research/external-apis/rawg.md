# RAWG Video Games Database API

**Investigación para:** Fase 2 — Colección de Videojuegos

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.rawg.io/api` |
| **Auth** | API Key (gratuita con registro en https://rawg.io/apidocs) |
| **Rate limit** | 100,000 requests/mes (free tier) |
| **Gratis** | Sí, con registro gratuito |
| **Total juegos** | 500,000+ |
| **Cobertura** | 50 plataformas incluyendo móviles |
| **Documentación** | https://api.rawg.io/docs/ |
| **Estado** | Activa y mantenida (2026) |

## Endpoints Útiles

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /api/games?key={key}&search={query}` |
| Detalle de juego | `GET /api/games/{id}?key={key}` |
| Juegos por plataforma | `GET /api/games?key={key}&platforms={ids}` |
| Juegos por género | `GET /api/games?key={key}&genres={slug}` |
| Juegos por fecha | `GET /api/games?key={key}&dates={from,to}` |
| Juegos populares | `GET /api/games?key={key}&ordering=-rating` |
| Screenshots | `GET /api/games/{id}/screenshots?key={key}` |
| Trailers | `GET /api/games/{id}/movies?key={key}` |
| Plataformas | `GET /api/platforms?key={key}` |
| Géneros | `GET /api/genres?key={key}` |
| Tags | `GET /api/tags?key={key}` |
| Desarrolladores | `GET /api/developers?key={key}` |
| Publicadores | `GET /api/publishers?key={key}` |

## IDs de Plataformas Relevantes

| ID | Plataforma | Equivalente interno |
|----|------------|---------------------|
| 4 | PC | `PC` |
| 12 | Wii | — |
| 11 | Wii U | `WII_U` |
| 7 | Nintendo Switch | `SWITCH` |
| 15 | PlayStation 2 | `PS2` |
| 13 | PlayStation 3 | `PS3` |
| 29 | PlayStation 5 | — |
| 30 | Xbox Series S/X | — |

## IDs de Géneros Relevantes

| ID | Género |
|----|--------|
| 4 | Action |
| 5 | Shooter |
| 3 | Adventure |
| 51 | Indie |
| 7 | RPG |
| 2 | Strategy |
| 10 | Racing |
| 14 | Simulation |
| 15 | Sports |
| 6 | Fighting |
| 17 | Card |
| 83 | Board Games |

## Mapeo: RAWG → Game (Modelo Interno)

| Campo RAWG | Campo Interno (Game) | Notas |
|------------|----------------------|-------|
| `id` | `externalId` | ID único de RAWG |
| `name` | `title` | Título |
| `released` | — | No se usa directamente |
| `background_image` | `thumbnailUrl` | URL del thumbnail |
| `genres[].name` | `genre` | Género principal |
| `platforms[].platform.name` | `platform` | Plataforma (mapeado a enum) |
| `rating` | — | Rating global, no usado como userRating |

## Estrategia de Uso

- **RAWG es la API principal** para búsqueda por nombre
- **FreeToGame es el fallback** cuando RAWG no devuelve resultados
- FreeToGame no soporta búsqueda por nombre; su catálogo es limitado (~415 juegos)

## Referencias

- [RAWG API Documentation](https://api.rawg.io/docs/)
