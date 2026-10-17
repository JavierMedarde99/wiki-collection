---
type: external-api
title: "RAWG Video Games Database API"
description: "Contrato de RAWG: búsqueda de videojuegos y plataformas"
tags: [api-externa, videojuegos]
---

# RAWG Video Games Database API

**Investigación para:** Fase 2 — Colección de Videojuegos. Backend: búsqueda externa `GET /api/v1/games/search` (`RAWGClient`) y catálogo de plataformas `GET /api/v1/games/platforms` (`RawgPlatformsClient`).

**Verificado en vivo:** 2026-10-15 con `curl --max-time 20` contra `https://api.rawg.io/api`, y contraste del contrato con el OpenAPI público en `https://api.rawg.io/docs/?format=openapi`. **No se pudo probar una petición autenticada correcta** (ni el contenido real de una respuesta): en esta máquina no hay `RAWG_API_KEY` en el entorno ni en la configuración local. Todo lo relativo a campos de respuesta se apoya en el OpenAPI publicado y en el código del backend, no en una respuesta real.

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.rawg.io/api` |
| **Auth** | API key obligatoria en **todas** las peticiones, como query param `key`. Sin ella: `401` `{"error": "The key parameter is not provided"}`. Con clave inválida: `401` `{"error": "The API key is not found"}`. |
| **Rate limit** | Plan Free: **20.000 peticiones/mes** (plan Business 50.000, Enterprise 1.000.000). No se han observado cabeceras de rate limit en la respuesta `401`. |
| **Gratis** | Sí, plan Free solo para proyectos personales/no comerciales, con backlink activo a RAWG obligatorio en cada página donde se usen los datos. Sin redistribución de datos. |
| **Formato** | JSON (`application/json`), incluido el cuerpo de error. |
| **Volumen** | 500.000+ juegos en 50 plataformas (incl. móviles); 2.100.000 capturas; 220.000 desarrolladores; 45.000 publicadores. |
| **Documentación oficial** | https://api.rawg.io/docs/ · registro: https://rawg.io/apidocs · Términos: https://rawg.io/tos_api · OpenAPI: `https://api.rawg.io/docs/?format=openapi` |
| **Estado** | Activa (2026-10-15): `/games` y `/platforms` responden `401` (no 5xx) y el portal de docs responde `200`. |

## Endpoints

Tabla de endpoints que el backend **consume realmente** y del resto del catálogo RAWG disponible pero no usado.

| Uso | Endpoint | Parámetros | Notas |
|-----|----------|------------|-------|
| Búsqueda por título (**usado**) | `GET /games` | `search` (el código no envía nada más salvo `key`) | `RAWGClient.search`. No envía `page`/`page_size`, así que solo obtiene la primera página de RAWG (su `page_size` por defecto). Devuelve `results[]`. |
| Géneros de un juego (**usado**) | `GET /games/{id}` | `key` | `RAWGClient.getGenres(externalId)`; lee `genres[].name`. `{id}` es el `externalId` guardado en el `Game`. |
| Catálogo de plataformas (**usado**) | `GET /platforms` | `key` | `RawgPlatformsClient.getPlatforms`; lee `results[]` → `PlatformInfo`. Tampoco envía paginación (ver Limitaciones). |
| Detalle de juego | `GET /games/{id}` | `key` | Disponible; el backend solo lo usa para géneros. |
| Capturas / trailers | `GET /games/{id}/screenshots`, `GET /games/{id}/movies` | `key` | Existen; **no usados**. |
| Relacionados | `GET /games/{id}/suggested`, `/game-series`, `/parent-games`, `/additions` | `key` | Existen; **no usados**. |
| Filtros de catálogo | `GET /games` | `genres`, `tags`, `platforms`, `publishers`, `developers`, `dates`, `metacritic`, `ordering`, `exclude_*`, … | Disponibles; el backend solo usa `search`. |
| Catálogos auxiliares | `GET /genres`, `/tags`, `/developers`, `/publishers`, `/platforms/{id}` | `key` | Existen; **no usados** (los géneros del proyecto son texto libre, ver mapeo). |

## Mapeo: RAWG → modelo interno

`RAWGClient` transforma cada resultado de `/games` en un `GameSearchResult`. La plataforma que solo se guarda en `Game` es texto libre.

**Búsqueda (`/games` → `GameSearchResult`):**

| Campo RAWG | Campo interno | Notas |
|------------|---------------|-------|
| `id` (integer) | `id` | Se convierte a String con `toString()`. |
| `name` | `title` | |
| `background_image` | `thumbnailUrl` | Leído con `@JsonProperty("background_image")`. |
| `genres[0].name` | `genre` | Solo el **primero** de la lista. |
| `platforms[0].platform.name` | `platform` | Solo la **primera** plataforma con nombre. |
| `publishers[0].name` | `publisher` | Solo el primero. |
| `developers[0].name` | `developer` | Solo el primero. |
| `released` (`YYYY-MM-DD`) | `releaseDate` (`LocalDate`) | `LocalDate.parse`; si el formato no encaja → `null`. |
| — | `description` | No se rellena: siempre `null` en la búsqueda. |
| — | `externalSource` | Constante `"RAWG"`. |
| `rating` | — | **No** se usa como `userRating`. |

**Enriquecimiento de géneros (`GameService.fillEmptyGenres`):** si el `Game` guardado no tiene géneros, se llama a `RAWGClient.getGenres(externalId)` → `GET /games/{id}` → `genres[].name` → `Game.genres` (`List<String>`). Si algo falla, se deja como estaba.

**Catálogo de plataformas (`GET /platforms` → `PlatformInfo`):**

| Campo RAWG | Campo interno | Notas |
|------------|---------------|-------|
| `results[].id` (integer) | `PlatformInfo.id` (`Long`) | Convertido con `longValue()`. |
| `results[].name` | `PlatformInfo.name` | Se descartan las plataformas sin nombre. |
| `results[].slug` | `PlatformInfo.slug` | |

**Aviso sobre `platform`:** en el modelo interno, `Game.platform` es **texto libre** (`String`) y el enum `GamePlatform` **ya no existe**. No hay tabla de equivalencias fija: la UI obtiene las opciones en tiempo de ejecución desde `GET /api/v1/games/platforms`, que devuelve `PlatformInfo(id, name, slug)` servido por `RawgPlatformsClient` sobre RAWG `/platforms`. El filtro `platform` de `GET /api/v1/games` compara contra ese texto libre.

## Configuración en el proyecto

Propiedades reales en `src/main/resources/application.properties`:

| Propiedad | Valor | Notas |
|-----------|-------|-------|
| `rawg.api-key` | `${RAWG_API_KEY:}` | **Vacía por defecto.** Sin clave, `RAWGClient`/`RawgPlatformsClient` omiten el parámetro `key` y RAWG responde `401`. |
| `http.client.connect-timeout` | `5s` | Compartido por todos los clientes externos (`HttpClientProperties`). |
| `http.client.read-timeout` | `10s` | Compartido. |

Fijado en código, no en propiedades:
- Base URL del `RestClient`: `https://api.rawg.io/api` (`RestClientConfig.rawgRestClient()`).
- No existe ninguna propiedad `rawg.api.retry-*` ni `rawg.api.base-url`.

**Comportamiento sin clave:** la petición sin `key` da `401` → `RestClientResponseException` → se registra un `warn` y `RAWGClient.search` devuelve lista vacía → `GameSearchService` cae en el fallback **FreeToGame**. Es decir, sin `RAWG_API_KEY` la búsqueda no falla con 5xx: degrada en silencio al catálogo de FreeToGame.

## Retry y caché

- **Reintentos:** RAWG **no tiene reintentos**. `HttpClientProperties` indica que los reintentos son por cliente (`retry-attempts`/`retry-delay-ms`) y RAWG no define esas propiedades (a diferencia de Scryfall, BGG o TMDB). `RAWGClient` y `RawgPlatformsClient` sólo capturan `RestClientResponseException` (401/4xx/5xx) y `ResourceAccessException` (timeout/red), registran un `warn` y devuelven lista vacía.
- **Timeouts:** conexión 5 s, lectura 10 s (propiedades compartidas de arriba).
- **Caché** (Caffeine en memoria, sin Redis):
  - `gameSearch` (`CacheConfig.GAME_SEARCH`) con claves `'rawg:' + query` y `'rawg-genres:' + externalId`. TTL = `app.cache.ttl.games` = `3600s` (1 h). Una búsqueda fallida se cachea igualmente como lista vacía (no hay `unless`), así que un fallo puntual de RAWG bloquea esa consulta durante 1 h.
  - `platformSearch` (`CacheConfig.PLATFORM_SEARCH`) con clave `'all'` y `unless = "#result.isEmpty()"` → **los resultados vacíos no se cachean**, de modo que cada llamada al endpoint de plataformas reintenta contra RAWG.

## Limitaciones

- **Clave obligatoria.** Sin `RAWG_API_KEY` todo cae a FreeToGame de forma silenciosa, sin error visible para el cliente.
- **Sin paginación.** Ni la búsqueda ni el catálogo de plataformas envían `page`/`page_size`. `RAWGClient` solo recibe la primera página de `/games` (limitando cuántos resultados externos hay disponibles aunque `GET /api/v1/games/search` acepte `page`/`size`, porque `PagedResults.slice` pagina en memoria sobre lo que devolvió RAWG). Igualmente, `GET /api/v1/games/platforms` devuelve solo la primera página de `/platforms`, **no las ~50 plataformas completas**.
- **Errores silenciosos.** `401`, `5xx` y timeouts se traducen todos a lista vacía; no se distinguen ni se propagan, aunque el contrato del controlador documente `502`/`503` para el catálogo externo.
- **Modelo OpenAPI incompleto.** El OpenAPI público declara `genres`, `publishers` y `developers` **solo como parámetros de filtro de `/games`**, nunca como propiedades de los modelos `Game` (lista) ni `GameSingle` (detalle). Sin embargo, `RAWGClient` lee esos tres campos de las respuestas. La biblioteca de RAWG es conocida por estar incompleta, pero **no se ha podido verificar en vivo** (requiere clave) si `/games` devuelve realmente `publishers`/`developers`; si no lo hace, `publisher` y `developer` llegarían `null` en la búsqueda. `platforms` sí está declarado en el modelo de lista.
- **Solo se toma el primer elemento** de `genres`, `platforms`, `publishers` y `developers`.
- **`rating` de RAWG no se usa** como valoración del usuario; `metacritic`, `added`, capturas, trailers, tiendas y juegos relacionados tampoco se consumen.
- **Licencia Free no comercial**, con backlink obligatorio y sin redistribución de datos.

## Referencias

- [RAWG API Documentation](https://api.rawg.io/docs/)
- [RAWG API — planes y registro](https://rawg.io/apidocs)
- [RAWG API Terms of Service](https://rawg.io/tos_api)
- [OpenAPI público](https://api.rawg.io/docs/?format=openapi)
