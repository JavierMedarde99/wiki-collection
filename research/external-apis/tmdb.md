---
type: external-api
title: "TMDB (The Movie Database) API"
description: "Contrato de TMDB: películas, series y proveedores de streaming"
tags: [api-externa, peliculas-series]
---

# TMDB (The Movie Database) API

**Investigación para:** Fase 5 — Películas y Series (búsqueda de catálogo) y Fase 25 — Plataformas de Streaming (watch providers)

**Verificado en vivo:** 2026-10-15 (parcial: sin clave de API; solo se pudo comprobar el comportamiento no autenticado)

**Código que la consume:**
- `src/main/java/com/wikicollection/infrastructure/adapter/out/tmdb/TmdbClient.java` — único cliente; implementa `ExternalMovieCatalogClient`, `MovieDetailsClient` y `WatchProvidersClient`
- `src/main/java/com/wikicollection/infrastructure/adapter/out/tmdb/ProviderUrlMapper.java` — construye la URL del logo del proveedor
- `src/main/java/com/wikicollection/infrastructure/config/TmdbClientConfig.java` — bean `tmdbRestClient` sobre `HttpClientProperties`
- Consumidores: `MovieSearchService` (búsqueda) y `MovieShowService` (géneros y proveedores al guardar/actualizar)

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.themoviedb.org/3` |
| **Auth** | Clave de API. TMDB acepta dos formas en v3: query param `api_key` o cabecera `Authorization: Bearer <API Read Access Token>`. **Este proyecto usa `api_key` como query param** |
| **Rate limit** | El límite antiguo (40 peticiones / 10 s) se desactivó el 16-12-2019. Hoy hay un techo difuso "en torno a 40 peticiones/segundo" y TMDB pide respetar el `429` si aparece |
| **Gratis** | Sí, con registro en https://www.themoviedb.org/settings/api |
| **Formato** | JSON (UTF-8) |
| **Sin clave** | Toda la API `/3` responde **401**; el CDN de imágenes no requiere clave |
| **Documentación oficial** | https://developer.themoviedb.org/docs · https://developer.themoviedb.org/reference/intro/getting-started |
| **Estado** | Activa y mantenida (2026) |

### Comprobaciones en vivo (sin clave)

| Petición | HTTP | Cuerpo / observación |
|----------|------|----------------------|
| `GET /3/search/movie?query=matrix` | `401` | `{"status_code":7,"status_message":"Invalid API key: You must be granted a valid key.","success":false}` |
| `GET /3/search/tv?query=matrix` | `401` | mismo cuerpo (`status_code: 7`) |
| `GET /3/movie/603?language=es-ES` | `401` | mismo cuerpo |
| `GET /3/movie/603/watch/providers` | `401` | mismo cuerpo |
| `GET /3/configuration` | `401` | mismo cuerpo |
| `GET /4/list/1` (v4, sin token) | `401` | mismo cuerpo |
| `GET /3/search/movie?query=matrix&api_key=INVALID_KEY_TEST` | `401` | mismo cuerpo; una clave inválida da exactamente la misma respuesta que no enviar clave |
| `GET https://image.tmdb.org/t/p/w500/<ruta-inexistente>` | `404` | El CDN de imágenes es público: responde 404 por recurso inexistente, no 401 |
| `GET https://image.tmdb.org/t/p/original/<ruta-inexistente>` | `404` | Igual que arriba |

Las respuestas llegan servidas por CloudFront (`server: openresty`, `x-cache: Error from cloudfront`) con `cache-control: public, max-age=300`.

**No fue posible verificar con clave:** no hay `TMDB_API_KEY` disponible en el entorno de esta investigación, así que no se pudo comprobar ninguna respuesta 200 real (forma exacta del JSON de `search`, `genres` ni `watch/providers`), ni el código de error de un identificador inexistente con clave válida, ni el `429` por rate limit. Lo que sigue sobre campos y endpoints está derivado del código del backend y de los tests con `MockWebServer`, no de una llamada real autenticada.

## Endpoints usados

Todos se invocan contra la base `https://api.themoviedb.org/3`, con `api_key` añadido solo si `tmdb.api-key` no está vacío.

| Uso | Endpoint | Parámetros que envía el backend | Notas |
|-----|----------|--------------------------------|-------|
| Buscar películas | `GET /search/movie` | `query`, `language=es-ES`, `page=1`, `api_key` | Devuelve el array `results` |
| Buscar series | `GET /search/tv` | `query`, `language=es-ES`, `page=1`, `api_key` | Mismo contrato que la búsqueda de películas |
| Géneros de una película | `GET /movie/{id}` | `language=es-ES`, `api_key` | Del detalle solo se lee `genres[].name`; el resto de campos se ignora |
| Géneros de una serie | `GET /tv/{id}` | `language=es-ES`, `api_key` | Igual que el anterior |
| Proveedores de streaming (película) | `GET /movie/{id}/watch/providers` | `api_key` | Se lee `results["ES"]` |
| Proveedores de streaming (serie) | `GET /tv/{id}/watch/providers` | `api_key` | Se lee `results["ES"]` |

**Endpoints que NO se consumen** (aparecen en documentación antigua o en el contrato REST del wiki, pero no en el código):

- `GET /movie/{id}/credits` y `GET /movie/{id}/images` — `TmdbClient` no los llama. El endpoint `GET /movieshows/images/{id}` que figura en `specs/movie-shows/api-contract.md` no existe en `MovieShowController`.
- `GET /configuration` — la construcción de URLs de imágenes está hardcodeada, no se negocia con TMDB.
- Cualquier endpoint `person`, `trending`, `discover`, `find`, `collection` o de múltiples.

**Búsqueda:** el backend no usa la paginación de TMDB. Fija `page=1` y pagina en memoria con `PagedResults.slice(...)`, de modo que solo ve los primeros ~20 resultados por tipo. Cuando `mediaType` es `null` hace **dos** llamadas (movie + tv) y concatena: primero películas, luego series.

**Ruta del endpoint interno:** `GET /api/v1/movieshows/search?name={query}&mediaType={MOVIE|TV}&page=&size=`. El parámetro real es `mediaType` (no `type`). El 400 por `name` vacío lo genera el backend, no TMDB.

## Mapeo: TMDB → modelo interno

`TmdbClient` deserializa con records, todos anotados `@JsonIgnoreProperties(ignoreUnknown = true)`, así que los campos no listados se descartan silenciosamente.

### Búsqueda → `MovieSearchResult`

| Campo TMDB | Campo interno | Notas |
|------------|---------------|-------|
| `id` | `externalId` | Se convierte a String; `null` si TMDB no lo manda |
| `title` (película) / `name` (serie) | `title` | El campo elegido depende del tipo, no del contenido |
| `overview` | `overview` | Sinopsis |
| `release_date` (película) / `first_air_date` (serie) | `releaseDate` | Se hace `LocalDate.parse`; si el formato falla o está vacío → `null` |
| `poster_path` | `posterUrl` | Se le antepone `https://image.tmdb.org/t/p/w500`; `null` si falta |
| `backdrop_path` | `backdropUrl` | Se le antepone `https://image.tmdb.org/t/p/original`; `null` si falta |
| `vote_average` | `voteAverage` | `Double`; es la nota de la comunidad TMDB, no la del usuario |
| — (según el endpoint llamado) | `mediaType` | `MOVIE` si vino de `/search/movie`, `TV` si vino de `/search/tv` |
| — | `externalSource` | Cadena literal `"TMDB"` |

### Detalle → géneros de `MovieShow`

| Campo TMDB | Campo interno | Notas |
|------------|---------------|-------|
| `genres[].name` | `genres` (`List<String>`) | Solo se rellena si el usuario no envió géneros. Con `language=es-ES` los nombres llegan en español |

### Watch providers → `streamingProviders` de `MovieShow`

| Campo TMDB | Campo interno | Notas |
|------------|---------------|-------|
| `results["ES"]` | — | El país está hardcodeado a `ES` en `MovieShowService` (`WATCH_COUNTRY`); otros países se ignoran. Si la clave no existe → listas vacías |
| bloques `flatrate` / `buy` / `rent` | `streamingProviders[].type` | Enum `ProviderAccessType`: `FLATRATE`, `BUY`, `RENT`. No se leen `ads`, `free` ni `rent` con otras variantes |
| `provider_id` | `providerId` | Se mapea a `Integer` |
| `provider_name` | `providerName` | Tal cual lo devuelve TMDB (en inglés: "Netflix", "Google Play") |
| `logo_path` | `logoUrl` | `ProviderUrlMapper` antepone `https://image.tmdb.org/t/p/original`; `null` si el path falta o está en blanco |
| — | `watchCountry` | Cadena `"ES"` |
| `id` (raíz) | — | Se deserializa pero no se usa |

El resultado de `getWatchProviders` es siempre un mapa con las tres claves, aunque alguna quede vacía. Los proveedores se aplanan a una lista única en `MovieShowService.enrichStreamingProviders`: primero los de tipo `FLATRATE`, después `BUY` y `RENT` (orden de iteración de un `EnumMap`, es decir, orden de declaración del enum). Si la lista aplanada queda vacía, **no** se sobrescriben `streamingProviders` ni `watchCountry`.

## Configuración en el proyecto

En `src/main/resources/application.properties`:

```
tmdb.api-key=${TMDB_API_KEY:}
tmdb.api.base-url=https://api.themoviedb.org/3
tmdb.api.retry-attempts=3
tmdb.api.retry-delay-ms=1000
```

| Propiedad | Variable de entorno | Default en código | Uso |
|-----------|--------------------|-------------------|-----|
| `tmdb.api-key` | `TMDB_API_KEY` | vacío | Si está presente y no en blanco, se añade `api_key` a cada petición |
| `tmdb.api.base-url` | — | `https://api.themoviedb.org/3` | Base del bean `tmdbRestClient` |
| `tmdb.api.retry-attempts` | — | `3` | Reintentos **adicionales** al intento inicial (4 peticiones en total) |
| `tmdb.api.retry-delay-ms` | — | `1000` | Espera entre reintentos, en milisegundos |

La clave es opcional en la configuración (el `@Value` tiene default vacío y `TmdbClient` omite el parámetro si no hay clave), pero **en la práctica no lo es**: sin clave TMDB responde 401 y la búsqueda devuelve una lista vacía. `architecture/deployment.md` ya la marca como opcional para desarrollo.

`TmdbClientConfig` construye el `RestClient` con `HttpClientProperties.restClientBuilder(baseUrl)`, que aplica la política HTTP común:

```
http.client.connect-timeout=5s
http.client.read-timeout=10s
```

## Retry y caché

**Reintentos.** `executeWithRetry` envuelve cada llamada y hace hasta `retry-attempts + 1` intentos. Detalle importante: solo captura `RestClientResponseException`, es decir **reintenta cualquier respuesta de error, incluidos 401, 403 y 404**, no solo 5xx. Un timeout o error de conexión (`ResourceAccessException`) **no** se reintenta y sale disparado hacia el `catch` del llamante. El test `TmdbClientTest.retriesAndReturnsEmpty_whenUnavailable` confirma 4 peticiones ante un 503 persistente.

**Degradación.** Ningún fallo de TMDB llega al cliente como error: `search` captura `RestClientResponseException`, `ResourceAccessException` y `RestClientException` y devuelve lista vacía; `getWatchProviders` y `getGenres` devuelven vacío. El backend prefiere una colección sin proveedores a un 5xx.

**Caché (Caffeine, en memoria).** `TmdbClient` cachea dos cosas bajo el nombre `movieSearch`:

| Método | Nombre de caché | Clave |
|--------|-----------------|-------|
| `search(query, mediaType)` | `movieSearch` | `"{query}|{mediaType}"` |
| `getGenres(tmdbId, mediaType)` | `movieSearch` | `"genres:{tmdbId}|{mediaType}"` |

`getWatchProviders` **no** está cacheado: cada guardado, actualización o `POST /movieshows/{id}/refresh-providers` golpea TMDB.

Datos de la caché `movieSearch` según `CacheConfig`: TTL `app.cache.ttl.movieshows=3600s` (1 hora), tamaño máximo `app.cache.max-size=1000`. Además, `MovieShowService` cachea `movieList` y `movieDetail`, que se invalidan (`@CacheEvict`) al guardar, actualizar, refrescar proveedores o borrar.

## Limitaciones

- **Sin clave no hay nada.** Verificado: `/3/search/movie`, `/3/search/tv`, `/3/movie/{id}`, `/3/movie/{id}/watch/providers`, `/3/configuration` y `/4/list/1` devuelven los seis `401` con `status_code: 7`. Una clave inválida produce exactamente la misma respuesta, así que un 401 no distingue "falta la clave" de "clave revocada".
- **Verificación autenticada pendiente.** Los 200 con JSON real quedan sin comprobar en este entorno (no hay clave). Los campos documentados arriba provienen del código y de los fixtures de `TmdbClientTest`/`ProviderUrlMapperTest`.
- **Búsqueda recortada.** `page=1` fijo + paginación en memoria: el backend nunca consulta la página 2 de TMDB. Por eso el total de resultados es como máximo ~20 por tipo y sin `mediaType` son dos llamadas.
- **Géneros en español y no normalizados.** Con `language=es-ES` los géneros llegan traducidos ("Ciencia ficción"), se guardan como texto libre en `genres` y son el resultado de `GET /movieshows/genres`. Mezclar idiomas o cambiar de idioma rompería el filtro por género.
- **Solo proveedores de España.** `WATCH_COUNTRY = "ES"` está hardcodeado en `MovieShowService`: no hay parámetro de país ni lectura de otros mercados, y no se procesan los bloques `free` ni `ads`.
- **Paginación, `credits` e `images` de TMDB sin usar.** El `TmdbClient` no llama a `/credits` ni a `/images`; el contrato `GET /movieshows/images/{id}` del wiki no tiene implementación en `MovieShowController`.
- **La clave viaja en la URL.** Al usar `api_key` como query param (y no `Authorization: Bearer`), la clave queda expuesta en logs de proxy, historiales y trazas. TMDB ofrece el token Bearer como alternativa.
- **Reintento indiscriminado de 4xx.** Un 401 por clave mal configurada se reintenta 3 veces con 1 s de espera antes de degradar a lista vacía: latencia y tráfico inútiles.
- **Sin caché en watch providers.** Cada escritura de una película o serie dispara una llamada a TMDB; el refresco manual está pensado precisamente para cuando el dato cacheado en Mongo se queda obsoleto.
- **`executeWithRetry` puede lanzar `NullPointerException` silenciosa** si `retryAttempts` fuese negativo (`last` quedaría `null` y se lanzaría); con los valores configurados (3) no ocurre.
- **Estado del registro en el wiki.** `research/architecture.md` dice "TMDB | 2 endpoints (search + images) | 40 req/seg"; a fecha de hoy son 6 endpoints reales y ninguno de imágenes, y el rate limit de 40/10 s lleva desactivado desde 2019.

## Referencias

- [TMDB API — documentación](https://developer.themoviedb.org/docs)
- [TMDB — guía de autenticación](https://developer.themoviedb.org/docs/authentication-application)
- [TMDB — rate limiting](https://developer.themoviedb.org/docs/rate-limiting)
- [TMDB — claves de API](https://www.themoviedb.org/settings/api)
- [TMDB — imágenes](https://developer.themoviedb.org/docs/image-basics)
- Código: `TmdbClient.java`, `ProviderUrlMapper.java`, `TmdbClientConfig.java`, `MovieShowService.java`, `MovieSearchService.java`, `MovieShowController.java` en `JavierMedarde99/backend-collection`
