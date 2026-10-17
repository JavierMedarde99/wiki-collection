---
type: external-api
title: "Steam Web API — Logros de Videojuegos"
description: "Contrato de la Steam Web API: logros de juegos"
tags: [api-externa, steam]
---

# Steam Web API — Logros de Videojuegos

**Investigación para:** Fase 2 — Colección de Videojuegos (logros de Steam) y resolución automática del `steamAppId` a partir del título.

**Verificado en vivo:** 2026-10-15 (parcial: sin clave de API; solo se pudo comprobar el comportamiento no autenticado y los endpoints públicos)

**Código que la consume:**
- `src/main/java/com/wikicollection/infrastructure/adapter/out/steam/SteamAchievementsClient.java` — único cliente; implementa `SteamCatalogueClient`
- `src/main/java/com/wikicollection/domain/port/out/SteamCatalogueClient.java` — puerto de salida (`searchGameByName`, `getGameSchema`, `getPlayerAchievements`)
- `src/main/java/com/wikicollection/application/service/GameAchievementsService.java` — combina esquema + progreso del jugador
- `src/main/java/com/wikicollection/application/service/GameService.java` — usa `searchGameByName` para rellenar `steamAppId`
- `src/main/java/com/wikicollection/infrastructure/config/RestClientConfig.java` — beans `steamRestClient` y `steamStoreRestClient`

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.steampowered.com` (Web API). El buscador de tienda vive en otro host: `https://store.steampowered.com/api` |
| **Auth** | Clave de API. La documentación oficial permite enviarla como query param `key` **o** como cabecera `x-webapi-key`. **Este proyecto usa siempre `key` como query param** |
| **Rate limit** | Tope contractual de **100.000 llamadas/día** por clave (Steam Web API Terms of Use, §2). No hay cifra oficial por minuto. Al superarlo responde `429 Too Many Requests` |
| **Gratis** | Sí, con cuenta de Steam y aceptación de los términos |
| **Formato** | JSON por defecto; la Web API admite además `format=xml` y `format=vdf`. **Este proyecto no envía `format`**, así que siempre consume JSON |
| **Documentación oficial** | https://partner.steamgames.com/doc/webapi_overview · https://partner.steamgames.com/doc/webapi/ISteamUserStats |
| **Estado** | Activa (2026) |

### Comprobaciones en vivo (sin clave)

| Petición | HTTP | Cuerpo / observación |
|----------|------|----------------------|
| `GET /ISteamUserStats/GetSchemaForGame/v2/?appid=440` | `400` | `Required parameter 'key' is missing` (HTML, `text/html`) |
| `GET /ISteamUserStats/GetSchemaForGame/v2/?appid=1` | `400` | Igual |
| `GET /ISteamUserStats/GetPlayerAchievements/v1/?steamid=76561197960287930&appid=440` | `400` | Igual |
| `GET /ISteamUserStats/GetSchemaForGame/v2/?appid=440&key=00000000000000000000000000000000` | `403` | `Access is denied. Retrying will not help. Please verify your key= parameter.` (HTML) |
| `GET https://store.steampowered.com/api/storesearch/?term=half-life&l=spanish&cc=ES` | `200` | JSON real: `{"total":10,"items":[{"type":"app","name":"Half-Life 2","id":220,...},...]}` |
| `GET /ISteamUserStats/GetGlobalAchievementPercentagesForApp/v0002/?gameid=440` | `200` | JSON real, **sin clave**. Endpoint **no usado** por el backend |
| `GET https://steamcommunity.com/dev/apikey` | `200` | Redirige a `https://steamcommunity.com/login/home/?goto=%2Fdev%2Fapikey` sin sesión: el formulario de alta no es visible ni verificable sin login |

Dos detalles observados que importan para el manejo de errores:

- Los cuerpos de error de la Web API son **HTML**, no JSON. El cliente solo llama a `getResponseBodyAsString()` para el log, así que no intenta deserializarlos.
- El `403` por clave inválida es el que documenta Valve como "Access is denied. Retrying will not help.": reintentar con la misma clave no arregla nada.

### No verificable sin clave

No hay `STEAM_API_KEY` en el entorno de esta investigación, así que **no se pudo comprobar ninguna respuesta `200` autenticada** de `GetSchemaForGame` ni de `GetPlayerAchievements`: ni la forma exacta del JSON real, ni el `429` por rate limit, ni el comportamiento con un `steamid` inexistente o un perfil privado. Lo que sigue sobre parámetros y campos se deriva del código del backend, de `SteamAchievementsClientTest` (MockWebServer) y de la documentación oficial de `ISteamUserStats`.

## Endpoints usados

`SteamAchievementsClient` inyecta **dos** `RestClient` con bases distintas:

- `steamRestClient` → `https://api.steampowered.com` (logros)
- `steamStoreRestClient` → `https://store.steampowered.com/api` (búsqueda de AppID)

| Uso | Endpoint | Parámetros que envía el backend | Notas |
|-----|----------|--------------------------------|-------|
| Buscar AppID por título | `GET https://store.steampowered.com/api/storesearch` | `term={título}`, `l=spanish`, `cc=ES` | **Sin clave.** Devuelve `items[]`; el backend toma `items[0].id` y descarta el resto |
| Esquema de logros del juego | `GET /ISteamUserStats/GetSchemaForGame/v2` | `appid`, `key` | Requiere clave. No envía `l` (idioma) |
| Logros del jugador | `GET /ISteamUserStats/GetPlayerAchievements/v1` | `steamid`, `appid`, `key` | Requiere clave. No envía `l` (idioma) |

**Endpoints que NO se consumen** (aparecen en documentación antigua del wiki o en la API, pero no en el código):

- `GET /ISteamUserStats/GetGlobalAchievementPercentagesForApp/v0002` — figuraba en la ficha anterior como usado; **es falso**: responde `200` sin clave, pero `SteamAchievementsClient` nunca lo llama. El modelo no guarda porcentajes globales.
- `GetUserStatsForGame`, `GetNumberOfCurrentPlayers`, `ISteamUserStats/SetUserStatsForGame` y el resto de interfaces (`ISteamUser`, `ISteamApps`, `IPlayerService`, `ISteamNews`, etc.).
- `partner.steam-api.com` — el host para publishers que aparece en los ejemplos de la documentación oficial; el proyecto usa el host público `api.steampowered.com`.
- `?format=xml` / `?format=vdf`.

**Ruta del endpoint interno:** `GET /api/v1/games/{id}/achievements?steamId={steamid}`. Si no se envía `steamId`, el backend usa el `steamId` guardado del dueño del juego (`GameAchievementsService.ownerSteamId`). Si no hay ninguno de los dos, o si el juego no tiene `steamAppId`, lanza `IllegalArgumentException` → **400** generado por el backend, no por Steam. Detalle en `specs/games/api-contract.md`.

**Resolución automática del `steamAppId`:** al crear o actualizar un juego con `obtainPlatinum=true` y `steamAppId` vacío, `GameService.resolveSteamAppId` llama a `searchGameByName(title)` y guarda `items[0].id` como String en `Game.steamAppId`. No hay validación de que el primer resultado se corresponda con el título.

## Mapeo: Steam → modelo interno

`SteamAchievementsClient` deserializa con records anidados **sin** `@JsonIgnoreProperties(ignoreUnknown = true)`, pero Jackson solo falla con propiedades desconocidas si `FAIL_ON_UNKNOWN_PROPERTIES` estuviera activo (en Spring Boot el default es tolerarlas). Los campos no listados simplemente se ignoran.

### Búsqueda de tienda → `steamAppId` (`Game`)

| Campo Steam | Campo interno | Notas |
|-------------|---------------|-------|
| `items[0].id` | `Game.steamAppId` | Se convierte a `String`. Solo se lee el **primer** resultado |
| `items[]` (resto) | — | Se descartan |
| `name`, `tiny_image`, `price`, `metascore`, `platforms`, `total` | — | Se deserializan parcialmente o se ignoran; no se mapean |

### Esquema del juego → `SteamAchievement` (`getGameSchema`)

Fuente: `game.availableGameStats.achievements[]`.

| Campo Steam | Campo interno | Notas |
|-------------|---------------|-------|
| `name` | `apiname` | Identificador interno del logro |
| `displayName` | `name` | Nombre visible |
| `description` | `description` | Descripción |
| `icon` | `iconUrl` | Icono desbloqueado. **No se lee `icongray`** (icono bloqueado) |
| — (fijo) | `achieved` | Siempre `false`: el esquema no dice si el jugador lo logró |
| `hidden`, `icongray` | — | Ignorados |
| `game.gameName` | — | Ignorado; el backend solo lee `availableGameStats.achievements` |
| `availableGameStats.stats` | — | Las estadísticas del juego se ignoran por completo |

### Logros del jugador → `SteamAchievement` (`getPlayerAchievements`)

Fuente: `playerstats.achievements[]`.

| Campo Steam | Campo interno | Notas |
|-------------|---------------|-------|
| `apiname` | `apiname` | Clave de cruce con el esquema |
| `achieved` | `achieved` | Entero `0`/`1` de Steam → booleano (`== 1`). `null` se trata como no logrado |
| `name` | `name` | Solo se usa si **no** hay esquema (ver combinación) |
| `description` | `description` | Igual que arriba |
| — (fijo) | `iconUrl` | `null`: este endpoint no aporta icono |
| `unlocktime` | — | **Descartado**: el modelo no guarda la fecha de desbloqueo |

### Combinación (`GameAchievementsService.getAchievements`)

1. Pide primero el **esquema**. Si vuelve vacío (lista vacía), usa **tal cual** los logros del jugador: la respuesta contendrá solo los desbloqueados.
2. Si hay esquema, indexa los logros del jugador por `apiname` y recorre el esquema, marcando `achieved` según ese índice. Los logros del esquema sin entrada en el progreso del jugador quedan en `false`.
3. Los logros del jugador cuyo `apiname` **no** aparece en el esquema se descartan.

Resumen calculado a partir de la lista combinada: `totalAchievements = size`, `totalAchieved = nº de achieved`, `percentage = redondeo a 1 decimal de totalAchieved*100/total` (`0.0` si la lista está vacía).

### Respuesta REST → DTOs

`GET /api/v1/games/{id}/achievements` devuelve `AchievementsResponse`:

| Campo REST | Origen |
|------------|--------|
| `achievements[].name` | `SteamAchievement.name` |
| `achievements[].description` | `SteamAchievement.description` |
| `achievements[].achieved` | `SteamAchievement.achieved` |
| `achievements[].iconUrl` | `SteamAchievement.iconUrl` |
| `totalAchievements` | tamaño de la lista combinada |
| `totalAchieved` | logros desbloqueados |
| `percentage` | porcentaje con 1 decimal |

**El `apiname` no se expone** en la respuesta REST: vive solo en el dominio y se usa como clave de cruce interna.

## Configuración en el proyecto

En `src/main/resources/application.properties`:

```
# Steam API
steam.api-key=${STEAM_API_KEY:}
```

Y la política HTTP común, en las líneas de `http.client`:

```
http.client.connect-timeout=5s
http.client.read-timeout=10s
```

| Propiedad | Variable de entorno | Default en código | Uso |
|-----------|--------------------|-------------------|-----|
| `steam.api-key` | `STEAM_API_KEY` | vacío | Se inyecta con `@Value("${steam.api-key:}")` y se añade como query param `key` en las dos llamadas autenticadas |
| `http.client.connect-timeout` | — | `5s` | Timeout de conexión de **todos** los clientes externos (`HttpClientProperties`) |
| `http.client.read-timeout` | — | `10s` | Timeout de lectura de todos los clientes externos |

`RestClientConfig` crea los beans con `HttpClientProperties.restClientBuilder(baseUrl)`:

- `steamRestClient` → `https://api.steampowered.com`
- `steamStoreRestClient` → `https://store.steampowered.com/api`

**El nombre correcto es `steam.api-key` con guion**, no `steam.api.key`. La guía de la clave usaba la forma con puntos; se corrige allí.

`architecture/deployment.md` marca `STEAM_API_KEY` como opcional. En la práctica **sin clave los logros no funcionan**: `GetSchemaForGame` y `GetPlayerAchievements` responden `400` (`Required parameter 'key' is missing`), el cliente degrada a lista vacía y el endpoint devuelve `200` con `achievements: []`, `totalAchievements: 0`, `percentage: 0.0`. La búsqueda de AppID por tienda sí funciona sin clave.

## Retry y caché

**Reintentos: ninguno.** Es la diferencia más llamativa respecto a otros clientes externos del proyecto:

- `application.properties` define `retry-attempts` / `retry-delay-ms` para Scryfall, BGG y TMDB, pero **no para Steam**.
- No hay `@Retryable`, ni bucle de reintentos, ni envío de `x-webapi-key`.
- `SteamAchievementsClient` captura `RestClientResponseException` (cualquier 4xx/5xx) y `ResourceAccessException` (timeout/conexión), registra un `log.warn` y **degrada**: `searchGameByName` devuelve `null`; `getGameSchema` y `getPlayerAchievements` devuelven `List.of()`.
- Consecuencia: un fallo transitorio de Steam (500/503) o un timeout se convierte en "sin logros" en la primera y única tentativa. El `403` por clave inválida tampoco se reintenta, lo cual es correcto.
- **`architecture/backend.md` dice que `SteamAchievementsClient` hace "3 reintentos"**; el código no tiene ninguno. Hay que corregir esa ficha (fuera del alcance de esta investigación, que solo escribe el presente archivo).

**Caché: ninguna para logros.** `GameAchievementsService` no lleva `@Cacheable` y no existe un nombre de caché de Steam/achievements en `CacheConfig` (los 40 nombres de caché actuales no incluyen ninguno de logros). Cada petición a `GET /api/v1/games/{id}/achievements` golpea Steam **1 o 2 veces** (esquema + progreso del jugador). Las cachés `gameDetail` / `gameList` solo guardan la entidad `Game`, nunca los logros.

## Limitaciones

- **Sin verificación autenticada.** No hay clave en el entorno: los `200` reales de esquema y logros del jugador están sin comprobar, igual que el `429` y el caso de perfil privado. Lo documentado viene del código, de los fixtures de `SteamAchievementsClientTest` y de la documentación oficial.
- **Sin reintentos.** Un fallo puntual de Steam deja la respuesta sin logros; no hay backoff ni segundo intento. Contradice lo que afirma `architecture/backend.md`.
- **Sin caché y sin rate limit propio.** Cada visita a la pantalla de logros implica 1–2 llamadas. Como `GET /games/{id}/achievements` es un endpoint **público**, cualquiera puede consumir la cuota de 100.000 llamadas/día de la única clave del servidor. Con el rate limit de entrada del proyecto (`RateLimitFilter`) aplicando solo a los endpoints de auth, no hay ninguna protección aquí.
- **La clave viaja en la URL.** Al usar `key` como query param en vez de la cabecera `x-webapi-key`, queda expuesta en logs de proxy y trazas. Valve documenta el uso de HTTPS precisamente por esto.
- **Idioma sin fijar en los logros.** La búsqueda de tienda fuerza `l=spanish` y `cc=ES`, pero `GetSchemaForGame` y `GetPlayerAchievements` **no envían `l`**, así que nombres y descripciones de logros llegan en el idioma por defecto (inglés) en la mayoría de juegos.
- **El `apiname` no llega al frontend.** `GameAchievementResponse` solo expone `name`, `description`, `achieved`, `iconUrl`. No se puede distinguir de forma estable dos logros con el mismo nombre visible.
- **La fecha de desbloqueo se pierde.** El campo `unlocktime` no se mapea.
- **Sin icono en el fallback.** Si el esquema vuelve vacío y se usan los logros del jugador, `iconUrl` es `null` para todos.
- **`storesearch` no es la Web API.** Es el endpoint JSON de la tienda, no documentado como API oficial; puede cambiar sin aviso. Además el backend elige `items[0]` a ciegas: buscar "Half-Life" devuelve "Half-Life 2" como primer resultado (verificado en vivo), y no hay comprobación de que coincida con el título guardado.
- **Un `steamid` inválido o un perfil privado se ven como "0 %".** `GetPlayerAchievements` fallará y devolverá lista vacía; como el esquema sí responde, todos los logros se combinan como no desbloqueados. El usuario no recibe ningún error, solo un progreso vacío. (Comportamiento deducido del código; el fallo real no se pudo verificar sin clave.)
- **Correcciones respecto a la ficha anterior:**
  - Decía "rate limit ~200 requests/5 minutos (sin key)". **No está documentado por Valve.** El único tope oficial es 100.000 llamadas/día por clave.
  - Listaba `GetGlobalAchievementPercentagesForApp` como endpoint en uso. **No se usa.**
  - Mapeaba `iconclosed` / `iconlarge` → `iconUrl`. Los campos reales que consume el cliente son `icon` (esquema); el campo `icongray` ni se lee.
  - Hablaba de `achieved` como booleano de Steam; en el JSON es un entero `0`/`1`.

## Referencias

- [Steam Web API — visión general](https://partner.steamgames.com/doc/webapi_overview)
- [Autenticación con Web API Keys](https://partner.steamgames.com/doc/webapi_overview/auth)
- [Códigos de error y respuestas](https://partner.steamgames.com/doc/webapi_overview/responses)
- [Interfaz ISteamUserStats](https://partner.steamgames.com/doc/webapi/ISteamUserStats)
- [Steam Web API Terms of Use](https://steamcommunity.com/dev/apiterms) — §2: límite de 100.000 llamadas/día
- [Registro de clave de API](https://steamcommunity.com/dev/apikey) — ver `steam-api-key-guide.md`
- Código: `SteamAchievementsClient.java`, `SteamCatalogueClient.java`, `GameAchievementsService.java`, `GameService.java`, `RestClientConfig.java`, `CacheConfig.java`, `application.properties` en `JavierMedarde99/backend-collection`
