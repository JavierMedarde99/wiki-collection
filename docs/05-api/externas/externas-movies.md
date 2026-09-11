# APIs Externas — Películas y Series

## Estado: Implementado (Fase 5)

---

## API Seleccionada

### TMDB (The Movie Database) API

- **Base URL:** `https://api.themoviedb.org/3`
- **Auth:** API Key (gratuita con registro)
- **Rate limit:** ~40 requests/segundo
- **Gratis:** Sí, con registro
- **Formato:** JSON
- **Documentación:** https://developer.themoviedb.org/docs
- **Estado:** Activa y mantenida (2026-09)

---

## Endpoints Utilizados

| Uso | Endpoint |
|-----|----------|
| Buscar película | `GET /search/movie?query={query}` |
| Buscar serie | `GET /search/tv?query={query}` |
| Detalle película | `GET /movie/{id}` |
| Detalle serie | `GET /tv/{id}` |
| Créditos | `GET /movie/{id}/credits` |
| Imágenes | `GET /movie/{id}/images` |

---

## Mapeo de Campos TMDB → MovieShow

| Campo TMDB | Campo MovieShow | Tipo | Notas |
|------------|-----------------|------|-------|
| `id` | `externalId` | String | ID único de TMDB |
| `title` / `name` | `title` | String | title para películas, name para series |
| `overview` | `overview` | String | Sinopsis |
| `release_date` / `first_air_date` | `releaseDate` | LocalDate | Fecha de estreno |
| `poster_path` | `posterUrl` | String | URL del póster (image.tmdb.org/t/p/w500) |
| `backdrop_path` | `backdropUrl` | String | URL de imagen de fondo (image.tmdb.org/t/p/original) |
| `vote_average` | `voteAverage` | Double | Puntuación media |
| - | `mediaType` | Enum | MOVIE o TV (según endpoint) |
| - | `externalSource` | String | "TMDB" |

---

## Estrategia de Implementación

1. **TMDB como API primaria** — Búsqueda por título, películas y series, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a MovieShow
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 1s
5. **Filtrado por tipo** — Parámetro `mediaType` opcional (MOVIE, TV, o ambos)

### Flujo de Búsqueda

```
1. Cliente → GET /api/movieshows/search?name=matrix&mediaType=MOVIE
2. Backend → TMDB API (/search/movie?query=matrix)
3. Mapear a MovieSearchResult → Convertir a JSON → Devolver
4. Si no hay resultados → Lista vacía
```

### Flujo de Detalle

```
1. Cliente → GET /api/movieshows/{id}
2. Backend → MongoDB (película/serie ya persistida localmente)
3. Mapear a MovieShow → Convertir a JSON → Devolver
```

---

## Implementación en el Backend

### TmdbClient (Spring Boot)

```java
@Component("tmdbClient")
public class TmdbClient implements ExternalMovieCatalogClient {

    private final RestClient tmdbRestClient;
    private final String apiKey;
    private final int retryAttempts;
    private final long retryDelayMs;

    @Override
    public List<MovieSearchResult> search(String query, MovieMediaType mediaType) {
        // Busca en /search/movie y/o /search/tv según mediaType
        // Usa RestClient para llamadas HTTP
        // Reintentos automáticos (3 intentos, delay 1s)
    }
}
```

### TmdbClientConfig

```java
@Configuration
public class TmdbClientConfig {
    @Bean
    public RestClient tmdbRestClient(
        @Value("${tmdb.api.base-url:https://api.themoviedb.org/3}") String baseUrl) {
        return RestClient.builder().baseUrl(baseUrl).build();
    }
}
```

---

## Decisiones Técnicas

- **TMDB elegido sobre OMDb y TVMaze** — Mayor catálogo, API oficial, búsqueda por tipo
- **Idioma: es-ES** — Todas las búsquedas se hacen en español
- **Imágenes con base URL** — `https://image.tmdb.org/t/p/w500` (póster) y `/t/p/original` (fondo)
- **externalId único** — Índice unique en MongoDB para evitar duplicados
- **WISHLIST eliminado** — El enum MovieStatus solo tiene WATCHING, WATCHED, PLAN_TO_WATCH

---

## Referencias

- [TMDB API Documentation](https://developer.themoviedb.org/docs)
- [TMDB API Key](https://www.themoviedb.org/settings/api)
- [TMDB Image Base URL](https://developer.themoviedb.org/docs/images)
