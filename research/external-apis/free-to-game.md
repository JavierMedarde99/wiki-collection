# FreeToGame (Free-To-Play Games Database) API

**Investigación para:** Fase 2 — Colección de Videojuegos. FreeToGame es la **API secundaria / fallback** de búsqueda de videojuegos: `GameSearchService` solo la consulta si RAWG devuelve una lista vacía.

**Verificado en vivo:** 2026-10-15 (todas las llamadas con `curl --max-time 20`, sin clave). Ver el detalle de códigos HTTP en cada sección. El código del `FreeToGameClient` se ha re-verificado contra el repo `JavierMedarde99/backend-collection`.

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://www.freetogame.com/api` |
| **Auth** | No requerida (ni clave ni cuenta). El servicio pide atribución a FreeToGame.com con hipervínculo activo |
| **Rate limit** | Documentado como "no más de 10 requests/segundo". No verificado empíricamente en esta investigación |
| **Gratis** | Sí, personal y comercial, con atribución |
| **Formato** | JSON (`application/json`). Listados = array; detalle = objeto |
| **Catálogo** | 419 juegos el 2026-10-15. Solo juegos free-to-play de PC (Windows) y navegador |
| **Documentación oficial** | https://www.freetogame.com/api-doc |
| **Estado** | Operativa el 2026-10-15 (HTTP 200). Sigue sirviendo sobre Cloudflare, con caché de borde de 24 h |

Notas de la verificación:

- `GET /api/games` → **HTTP 200**, `content-type: application/json`, ~195 KB.
- El endpoint **no exige clave**. La variante RapidAPI (`free-to-play-games-database.p.rapidapi.com`) sí la exige: sin cabecera devuelve **HTTP 401**.
- CORS: las respuestas (`200` y `404`) incluyen `access-control-allow-origin: *`. Sin embargo, el *preflight* `OPTIONS` → **HTTP 403** (Cloudflare). La propia documentación afirma que, si se necesita CORS, debe usarse la versión de RapidAPI. Es decir: **CORS no está soportado de forma fiable** por la API directa, pese a la cabecera anterior.
- La página `/api-doc` responde **HTTP 403** a `curl` (bloqueo Cloudflare), pero su contenido es legible desde un navegador/`webfetch`; de ahí se han tomado los endpoints y el rate limit documentados.
- Cabeceras de caché del servidor: `cache-control: max-age=86400` y `cf-cache-status: HIT`.

## Endpoints

Endpoints documentados oficialmente y verificados en vivo:

| Uso | Endpoint | Parámetros | Notas |
|-----|----------|------------|-------|
| Todos los juegos | `GET /api/games` | — | HTTP 200. 419 elementos el 2026-10-15 |
| Filtrar por plataforma | `GET /api/games?platform={valor}` | `platform`: `windows`, `browser`, `all` | HTTP 200. `platform=browser` devuelve también las entradas mixtas |
| Filtrar por categoría/tag | `GET /api/games?category={valor}` | `category`: `shooter`, `mmorpg`, `pvp`... | HTTP 200. Filtra; reduce el tamaño de respuesta |
| Ordenar | `GET /api/games?sort-by={valor}` | `sort-by`: `release-date`, `popularity`, `alphabetical`, `relevance` | HTTP 200 |
| Combinar plataforma + categoría + orden | `GET /api/games?platform={p}&category={c}&sort-by={s}` | los tres | HTTP 200 |
| Filtrar por varios tags | `GET /api/filter?tag={tags}` | `tag` separados por punto; opcional `platform` y `sort` | HTTP 200 |
| Detalle de un juego | `GET /api/game?id={game_id}` | `id` numérico | HTTP 200. Devuelve objeto con descripción larga, capturas y requisitos |
| Detalle sin `id` o con `id` inexistente | `GET /api/game` / `GET /api/game?id=999999` | — | **HTTP 404** con cuerpo `{"status":0,"status_message":"..."}` |
| Recomendaciones | `/api/...` | — | Documentado como "Coming Soon"; **no existe** |

**Lo que consume realmente el backend** (`FreeToGameClient`):

| Método del cliente | Endpoint real | Parámetros que envía | Notas |
|--------------------|---------------|----------------------|-------|
| `search(query)` | `GET /games` | `title={query}` | **`title` NO es un parámetro documentado y la API lo ignora.** Verificado: `GET /api/games?title=overwatch` devuelve exactamente el mismo cuerpo (md5 idéntico) que `GET /api/games` — el catálogo completo. El test unitario `FreeToGameClientTest` solo comprueba que el path sea `/games?title=...`, no que la API filtre |
| `getGenres(externalId)` | `GET /game` | `id={externalId}` | Devuelve el campo `genre` del detalle como lista de un solo elemento. Si no hay `genre` o el id es vacío → lista vacía |

No hay endpoint propio en `GameController` para `getGenres`: se usa internamente desde `GameService.fillEmptyGenres()` cuando el juego tiene `externalSource = FreeToGame`.

## Mapeo: FreeToGame → modelo interno

Campos realmente leídos del JSON y su destino en `GameSearchResult` (record de dominio):

| Campo FreeToGame | Campo interno (`GameSearchResult`) | Notas |
|------------------|------------------------------------|-------|
| `id` (número) | `id` | Se convierte a `String` con `toString()`; `null` si falta |
| `title` | `title` | — |
| `short_description` | `description` | Anotado `@JsonProperty("short_description")` |
| `genre` | `genre` | String único, no lista (el modelo guarda aparte `genres`) |
| `platform` | `platform` | Se guarda **tal cual** viene (`mapPlatform` solo hace `trim`). Valores vistos: `PC (Windows)`, `Web Browser`, `PC (Windows), Web Browser`. En el detalle `id=452` la API devuelve `Windows` (sin "PC"), así que hay inconsistencia entre listado y detalle |
| `publisher` | `publisher` | — |
| `developer` | `developer` | — |
| `release_date` | `releaseDate` | `LocalDate.parse`; si el formato es inválido o vacío → `null` |
| `thumbnail` | `thumbnailUrl` | URL de la miniatura en `freetogame.com` |
| — | `externalSource` | El cliente fija el literal `"FreeToGame"` |

Campos de la API que el backend **ignora**: `game_url`, `freetogame_profile_url`, y en el detalle `description`, `status`, `screenshots`, `minimum_system_requirements`.

## Configuración en el proyecto

- **Bean RestClient:** `RestClientConfig.freeToGameRestClient()`.
- **Base URL fija en código**, no configurable: `httpClientProperties.restClientBuilder("https://www.freetogame.com/api")`. **No existe ninguna propiedad `freetogame.*` en `application.properties`** (es el único `application*.properties` del repo). A diferencia de `amiiboapi.base-url` o `hobbyquanthq.api.base-url`, la URL de FreeToGame no se puede sobrescribir por entorno.
- **Cliente:** `infrastructure/adapter/out/freetogame/FreeToGameClient.java` (único archivo del paquete), bean `"freeToGameClient"`, implementa `ExternalGameCatalogClient`.
- **Timeouts:** heredados de la política HTTP global `http.client.connect-timeout=5s` y `http.client.read-timeout=10s`.
- **Manejo de errores:** captura `RestClientResponseException` y `ResourceAccessException` y devuelve `List.of()` (nunca propaga error al servicio). Un fallo de FreeToGame es indistinguible de "sin resultados".
- **Encadenado de búsqueda:** `GameSearchService.search()` llama primero a `rawgClient.search(query)` y solo si devuelve lista vacía llama a `freeToGameClient.search(query)`. El resultado se pagina en memoria con `PagedResults.slice`.

## Retry y caché

- **Reintentos: no hay.** `FreeToGameClient` no implementa reintentos y `application.properties` no define `freetogame.api.retry-attempts` ni `retry-delay-ms` (sí existen para Scryfall, BGG y TMDB). La política central `HttpClientProperties` solo aplica timeouts, no reintentos. **Discrepancia detectada:** `architecture/backend.md` afirma que el cliente FreeToGame tiene "3 reintentos" — no es cierto según el código actual.
- **Caché de aplicación (Caffeine, en memoria):** anotación `@Cacheable(cacheNames = CacheConfig.GAME_SEARCH, key = "'ftg:' + #query")` sobre `search()`. El nombre de caché es `gameSearch`, TTL `app.cache.ttl.games=3600s` (1 hora) y tamaño máximo global `app.cache.max-size=1000`. La clave incluye el prefijo `'ftg:'` y la query tal cual, de modo que dos búsquedas con la misma cadena comparten entrada durante una hora. **Ojo:** cualquier texto de búsqueda produce el catálogo completo (419 resultados) en la primera llamada, y ese resultado se cachea bajo la clave de esa query.
- **Caché de red:** el CDN de FreeToGame sirve con `cache-control: max-age=86400` (24 h) y Cloudflare marca `cf-cache-status: HIT`.
- `getGenres()` **no** está anotado con `@Cacheable`; se llama en cada `fillEmptyGenres()`.

## Limitaciones

- **No soporta búsqueda por nombre real.** El cliente envía `?title={query}`, pero la API **ignora** ese parámetro (no está documentado) y devuelve el catálogo completo. Verificado con md5 idéntico entre `?title=overwatch` y sin parámetro. Consecuencia práctica: cuando se activa el fallback, el usuario recibe los ~419 juegos del catálogo (paginados) en lugar de coincidencias con su búsqueda; es la causa de los "falsos positivos" del fallback.
- **Catálogo pequeño y sesgado:** 419 juegos el 2026-10-15, únicamente free-to-play.
- **Solo PC (Windows) y navegador.** No hay consolas ni móvil; el campo `platform` se guarda verbatim (`PC (Windows)`, `Web Browser` o la combinación de ambos) sin normalizar al enum interno.
- **Inconsistencia de `platform`** entre listado (`PC (Windows)`) y detalle (`Windows`).
- **El género es un único string**, no una lista; `getGenres` envuelve ese único valor en una lista.
- **Sin búsqueda por ID en el listado** más allá de `/game?id=`.
- **CORS no fiable:** cabecera `access-control-allow-origin: *` en GET, pero `OPTIONS` preflight → 403 y la doc remite a RapidAPI para CORS.
- **Códigos de error:** 200, 404 (no encontrado) y 500. El cliente los traga y devuelve lista vacía.
- **La API está viva y ha cambiado:** confirmado HTTP 200 en `/api/games`, `/api/games?platform=`, `?category=`, `?sort-by=`, `/api/filter` y `/api/game?id=`. Se ha movido a Cloudflare y el `access-control-allow-methods` viene duplicado; históricamente el servicio ha cambiado de comportamiento. No se ha podido verificar empíricamente el rate limit (10 req/s) ni la respuesta 500.
- **No se pudo comprobar** `/api-doc` con `curl` (403 por Cloudflare); se leyó con `webfetch`.
- **Atribución obligatoria** a FreeToGame.com (personal y comercial).

## Referencias

- [FreeToGame API Documentation](https://www.freetogame.com/api-doc) (leída el 2026-10-15 vía fetch; `curl` recibe 403)
- Variante RapidAPI de la misma base de datos: https://rapidapi.com/digiwalls/api/free-to-play-games-database (requiere clave; 401 sin ella)
- Cliente en el código: `src/main/java/com/wikicollection/infrastructure/adapter/out/freetogame/FreeToGameClient.java`
- Bean RestClient: `src/main/java/com/wikicollection/infrastructure/config/RestClientConfig.java` (`freeToGameRestClient`)
- Uso del fallback: `src/main/java/com/wikicollection/application/service/GameSearchService.java`
- Uso de `getGenres`: `src/main/java/com/wikicollection/application/service/GameService.java` (`fillEmptyGenres`)
- API principal relacionada: `research/external-apis/rawg.md`
