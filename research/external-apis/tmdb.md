# TMDB (The Movie Database) API

**Investigación para:** Fase 5 — Películas y Series

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.themoviedb.org/3` |
| **Auth** | API Key (gratuita con registro) |
| **Rate limit** | ~40 requests/segundo |
| **Gratis** | Sí, con registro |
| **Formato** | JSON |
| **Documentación** | https://developer.themoviedb.org/docs |
| **Estado** | Activa y mantenida (2026) |

## Endpoints Utilizados

| Uso | Endpoint |
|-----|----------|
| Buscar película | `GET /search/movie?query={query}` |
| Buscar serie | `GET /search/tv?query={query}` |
| Detalle película | `GET /movie/{id}` |
| Detalle serie | `GET /tv/{id}` |
| Créditos | `GET /movie/{id}/credits` |
| Imágenes | `GET /movie/{id}/images` |

## Mapeo: TMDB → MovieShow (Modelo Interno)

| Campo TMDB | Campo Interno (MovieShow) | Tipo | Notas |
|------------|---------------------------|------|-------|
| `id` | `externalId` | String | ID único de TMDB (unique index) |
| `title` / `name` | `title` | String | `title` para películas, `name` para series |
| `overview` | `overview` | String | Sinopsis |
| `release_date` / `first_air_date` | `releaseDate` | LocalDate | Fecha de estreno |
| `poster_path` | `posterUrl` | String | URL del póster (image.tmdb.org/t/p/w500) |
| `backdrop_path` | `backdropUrl` | String | URL de imagen de fondo (image.tmdb.org/t/p/original) |
| `vote_average` | `voteAverage` | Double | Puntuación media |
| — | `mediaType` | Enum | `MOVIE` o `TV` (según endpoint usado) |
| — | `externalSource` | String | `"TMDB"` |

## Estrategia de Implementación

1. **TMDB como API primaria** — Búsqueda por título, películas y series, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a MovieShow
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 1s
5. **Filtrado por tipo** — Parámetro `mediaType` opcional (MOVIE, TV, o ambos)
6. **Idioma** — Todas las búsquedas se hacen en español (`language=es-ES`)

## URLs de Imágenes

| Tipo | Base URL |
|------|----------|
| Póster (w500) | `https://image.tmdb.org/t/p/w500` |
| Backdrop (original) | `https://image.tmdb.org/t/p/original` |

## Decisión de Diseño: MovieStatus sin WISHLIST

El enum `MovieStatus` tiene solo 3 valores:
- `WATCHING` — Viendo actualmente
- `WATCHED` — Ya visto
- `PLAN_TO_WATCH` — Planea ver

Se eliminó `WISHLIST` para alinear con el dominio de visualización (una película no se "queuea" como deseo, se planea ver).

## Decisión de Diseño: Endpoint `/api/movieshows`

El endpoint base es `/api/movieshows` (no `/api/movies`) para mayor claridad semántica, ya que gestiona películas Y series.

## Referencias

- [TMDB API Documentation](https://developer.themoviedb.org/docs)
- [TMDB API Key](https://www.themoviedb.org/settings/api)
- [TMDB Image Base URL](https://developer.themoviedb.org/docs/images)
