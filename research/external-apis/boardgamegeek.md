# BoardGameGeek XML API 2

**Investigación para:** Fase 3 — Búsqueda de juegos de mesa en el catálogo externo
(`GET /boardgames/search?name=…` → `BoardGameSearchService` → `BggXmlClient`)

**Verificado en vivo:** 2026-10-15 con `curl --max-time 25` (fecha confirmada con `date -u`).
La **auth ya no es opcional**: todas las llamadas a endpoints sin token devolvieron **401**,
y las páginas de documentación oficial devolvieron **403** a `curl` (Cloudflare), así que el
contenido XML de las respuestas (y el 202) **no se pudo observar** en esta sesión. Ver
"Verificación en vivo" más abajo.

> Corrección importante respecto a la versión anterior de esta ficha: decía "Auth: no requerida".
> Desde 2025 BGG exige registro y cabecera `Authorization: Bearer <token>`; sin ella la API
> responde **401** y el cliente del backend devuelve lista vacía (ver Limitaciones).

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://boardgamegeek.com/xmlapi2` |
| **Auth** | **Sí, obligatoria** — `Authorization: Bearer <token>` |
| **Rate limit** | No verificado en vivo (documentación inaccesible desde aquí). Un hilo de BGG de 2026 afirma que la descarga de colecciones sin auth está "severamente" limitada; no hay cifra confirmada. **No asumir ninguna cifra** |
| **Gratis** | Sin verificar directamente (docs en 403). Los hilos consultados indican registro gratuito de la aplicación y sujeción a los términos de uso de BGG |
| **Formato** | XML siempre, incluso en errores. Errores de auth: `text/plain` con `Unauthorized. See https://boardgamegeek.com/using_the_xml_api` |
| **Documentación oficial** | https://boardgamegeek.com/using_the_xml_api y https://boardgamegeek.com/wiki/page/BGG_XML_API2 (ambas **403** para `curl` el 2026-10-15) |
| **Registro de aplicación** | Página de registro de BGG (probé `https://boardgamegeek.com/applications`: **403**; enlace referenciado en los hilos de BGG) |
| **Estado** | Activa, con auth obligatoria. El backend asume que puede devolver **202 Accepted** mientras prepara la respuesta; **no observado en vivo** (requiere token) |

### Verificación en vivo (2026-10-15)

| Prueba | HTTP | Respuesta |
|--------|------|-----------|
| `GET /xmlapi2/search?query=catan&type=boardgame` (sin cabecera) | **401** | `text/plain`, 61 B: `Unauthorized. See https://boardgamegeek.com/using_the_xml_api` |
| `GET /xmlapi2/thing?id=13&stats=1` (sin cabecera) | **401** | idem |
| `GET /xmlapi2/thing?id=13&stats=1` con `Authorization: Bearer <token inválido>` | **401** | `text/html`, 0 B |
| `GET /xmlapi2/thing?id=999999999999` (id inexistente, sin cabecera) | **401** | idem (la auth se comprueba antes que el id) |
| `GET /xmlapi2/collection?username=tomvasel&own=1` (sin cabecera) | **401** | `text/html`, 0 B |
| `GET /xmlapi2/collection?username=rahdo&own=1` (sin cabecera) | **401** | `text/html`, 0 B |
| `GET /xmlapi2/collection?username=JavierMedarde&own=1` (usuario inexistente, sin cabecera) | **200** | `<errors><error><message>Invalid username specified</message></error></errors>` |
| `GET /xmlapi2` (raíz), `/hot?type=boardgame`, `/user?name=tomvasel` (sin cabecera) | **401** | `text/plain` |
| `https://www.boardgamegeek.com/xmlapi2/thing?id=13` | **301** | redirección de Cloudflare → usar el host sin `www` |
| `https://api.geekdo.com/xmlapi2/thing?id=13` | **401** | el host alternativo también exige token |
| `GET /using_the_xml_api`, `/wiki/page/BGG_XML_API2`, `/applications` | **403** | bloqueo de Cloudflare (ni con `User-Agent` de navegador) |

Detalle curioso: con un **usuario inexistente** en `/collection` la respuesta llega sin auth
(200 con XML de error). Para usuarios reales, sin token, es 401. No sirve como atajo.

Consecuencia práctica: **son los 401, y no el 202, lo que se observa hoy sin token**. El
`202 Accepted` solo se puede comprobar con un token válido.

## Endpoints

Solo los dos primeros los consume el backend. El resto se listan para situarlos.

| Uso | Endpoint | Parámetros | Notas |
|-----|----------|-----------|-------|
| **Buscar juegos** (usado) | `GET /search` | `query` (texto), `type=boardgame` | Devuelve `<items>` con un `<item>` por coincidencia: `id`, `name`, `yearpublished`, `image`, `thumbnail`. **No** trae descripción ni estadísticas. El `total` de `<items>` no se lee |
| **Detalle** (usado) | `GET /thing` | `id` (lista separada por comas), `stats=1` | Aporta `description`, `minplayers`, `maxplayers`, `minplaytime`, `maxplaytime`, todos los `link` (publisher, designer, category, mechanic) y `statistics/ratings`. El backend lo llama **siempre después de `/search`**, en lotes de **20 ids** (`MAX_THING_IDS`) |
| Colección de usuario | `GET /collection` | `username`, `own=1`, … | **El backend no lo usa.** En 2026-10-15 devuelve 401 sin token |
| Otros (`/hot`, `/user`, `/plays`, `/forumlist`, `/family`…) | — | — | No usados por el backend. `/hot` y `/user` comprobados: 401 sin token |

El cliente no envía ningún parámetro de paginación: `/search` devuelve el bloque de
coincidencias que BGG considere y el backend las pagina en memoria (`PagedResults.slice`).
No pude comprobar en vivo si BGG admite algún parámetro de página en `/search`
(habría hecho falta un token); lo que sí es seguro es que el backend no lo usa.

## Mapeo: BGG XML → modelo interno

El parseo vive en `BoardGameXmlMapper` (Jackson `XmlMapper`, clases anidadas
`BggXmlItem`, `BggXmlName`, `BggXmlValue`, `BggXmlIntValue`, `BggThingStats`,
`BggThingRatings`, `BggXmlRatingValue`, `BggLink`), todas con
`@JsonIgnoreProperties(ignoreUnknown = true)`. El destino es el record
`BoardGameSearchResult` (respuesta de `GET /boardgames/search`).

Dos rutas de mapeo:

1. `map(xml)` — parsea los `<item>` de **`/search`**: solo id, nombre, año, imagen y miniatura.
2. `mapThing(xml)` — indexa por `id` los `<item>` de **`/thing`**; `enrich(...)` fusiona ambos.

| Campo BGG (XML) | Campo interno | Tipo | Origen | Notas |
|-----------------|---------------|------|--------|-------|
| `item/@id` | `bggId` | String | search + thing | Se guarda como texto, no como número |
| `name/@value` | `title` | String | search | No se filtra por `name/@type="primary"`: se toma el primer `name` que expone el XML |
| `description` | `description` | String | thing | En `/search` no viene; queda `null` si falla el `/thing` |
| `yearpublished/@value` | `yearPublished` | Integer | search + thing | Prevalece el de `/search`; si es `null` se usa el de `/thing` |
| `minplayers/@value` | `minPlayers` | Integer | thing | Solo de `/thing` (`@Min(1)` en `BoardGameRequest`) |
| `maxplayers/@value` | `maxPlayers` | Integer | thing | idem |
| `minplaytime/@value` | `minPlaytime` | Integer | thing | Tiempo **mínimo** declarado por BGG |
| `maxplaytime/@value` | `maxPlaytime` | Integer | thing | idem (máximo) |
| `link[@type="boardgamepublisher"]/@value` | `publisher` | String | thing | **Solo el primero** de los publicadores |
| `link[@type="boardgamedesigner"]/@value` | `designers` | List\<String> | thing | Todos los diseñadores |
| `link[@type="boardgamecategory"]/@value` | `categories` | List\<String> | thing | Todas las categorías |
| `link[@type="boardgamemechanic"]/@value` | `mechanics` | List\<String> | thing | Todas las mecánicas |
| `image` | `imageUrl` | String | search + thing | Prevalece el de `/search`; si es `null`, el de `/thing` |
| `thumbnail` | `thumbnailUrl` | String | search + thing | idem |
| `statistics/ratings/average/@value` | `bggRating` | BigDecimal | thing | Nota media de BGG (0-10). **Solo con `stats=1`** |
| `statistics/ratings/bayesaverage/@value` | — | BigDecimal | thing | **Se parsea pero no se usa** (`bayesAverage`) |
| — | `externalSource` | String | constante | Siempre `"BGG"`, no viene de BGG |
| `item/@type`, `items/@total`, `items/@termsofuse`, `link/@id`, `name/@sortindex`, `name/@type` | — | — | — | Ignorados |
| `statistics/ratings/usersrated`, `ranks`, `stddev`, `median`… | — | — | — | Ignorados |

Continuidad hacia persistencia: `GET /boardgames/search` devuelve `BoardGameSearchResult`
(15 campos). No crea nada. El frontend elige un resultado y crea el juego con
`POST /boardgames`; `BoardGameRequest` acepta los mismos 15 campos (incluido `bggId`)
más los propios de la colección (`status`, `notes`, `personalRating`, `playCount`,
`lastPlayedDate`, `difficulty`, `acquisitionPrice`). `externalSource` no se persiste.
Los campos en `BoardGame` con el mismo nombre son: `bggId`, `title`, `description`,
`yearPublished`, `minPlayers`, `maxPlayers`, `minPlaytime`, `maxPlaytime`, `publisher`,
`designers`, `categories`, `mechanics`, `imageUrl`, `thumbnailUrl`, `bggRating`.
`publisher → genres` es cosa del usuario, no de BGG (los `genres` del modelo interno no
se rellenan desde el catálogo externo).

## Configuración en el proyecto

`src/main/resources/application.properties`:

```properties
bgg.api.xml-url=https://boardgamegeek.com/xmlapi2
bgg.api.retry-attempts=3
bgg.api.retry-delay-ms=2000
bgg.auth.token=${BGG_AUTH_TOKEN:}
```

| Elemento | Detalle |
|----------|---------|
| `BggClientConfig` | Bean `bggXmlRestClient`. Si `bgg.auth.token` no está vacío, añade un `ClientHttpRequestInterceptor` que pone `Authorization: Bearer <token>` en **todas** las peticiones. Si está vacío, no añade cabecera alguna → 401 |
| `HttpClientProperties` | `http.client.connect-timeout` 5 s, `read-timeout` 10 s (`SimpleClientHttpRequestFactory`). Son **por intento**, no del ciclo completo de reintentos |
| Gotcha de configuración | `BggClientConfig` fija el `baseUrl` del `RestClient` a `https://boardgamegeek.com/xmlapi2` **en duro**, mientras `BggXmlClient` construye URIs absolutas con `bgg.api.xml-url`. Cambiar la propiedad apunta las peticiones a otro sitio (la URI absoluta manda), pero el `baseUrl` del bean queda descolgado |
| Cabeceras | `Accept: application/xml` en cada petición. **No** se envía `User-Agent` propio |
| Caché | `@Cacheable(cacheNames = "boardgameSearch", key = "#query")` sobre `BggXmlClient.search`, con TTL `app.cache.ttl.boardgames=3600s` y tamaño máx. 1000. La clave es solo el texto buscado |
| Timeouts + reintentos | Peor caso por llamada: 4 peticiones × 10 s de lectura + 3 × 2 s de espera ≈ 46 s |
| Variable de entorno | `BGG_AUTH_TOKEN` (documentada en `architecture/deployment.md` como "opcional": **de facto es obligatoria**) |

## Reintentos (por qué: 202 Accepted de BGG)

BGG encola las peticiones pesadas y responde **202 Accepted** mientras las prepara; la respuesta
buena llega en un reintento posterior (comportamiento documentado y asumido por el código;
**no observado en vivo**, haría falta un token). `BggXmlClient.callWithRetry` implementa esto:

1. `for (attempt = 1; attempt <= retryAttempts + 1; attempt++)` → con
   `retry-attempts=3` se hacen **4 peticiones HTTP** (3 reintentos + el intento inicial),
   con **3 esperas** de `retry-delay-ms` (2 s) entre ellas. La documentación que dice
   "3 intentos" es imprecisa: son 4 llamadas.
2. Solo se reintenta el **202**. Un 200 se devuelve tal cual. Un 4xx/5xx **no se reintenta**:
   `RestClientResponseException` se captura arriba.
3. Si la respuesta 202 trae `Set-Cookie`, se guarda **el primer** `Set-Cookie` (recortado en el
   primer `;`) y se reenvía como cabecera `Cookie` en los reintentos siguientes. Es la cookie de
   sesión que BGG usa para asociar el trabajo en curso. El `cookie` es local a cada
   `callWithRetry`, así que la cookie obtenida en `/search` **no** viaja a las llamadas de
   `/thing`.
4. Si se agotan los intentos y sigue el 202, devuelve un `ResponseEntity` 202 sin cuerpo y
   registra un `warn`: `BGG XML siguió devolviendo 202 tras N reintentos`.
5. `Thread.sleep` bloqueante: los reintentos ocurren dentro de la petición del cliente HTTP
   (el hilo de Tomcat queda ocupado durante el ciclo completo).

Comportamiento ante fallo, resumido (todo verificado en `BggXmlClientTest`):

| Situación | Resultado |
|-----------|-----------|
| `/search` OK + `/thing` OK | Lista con todos los campos |
| `/search` OK + `/thing` 500 o `<items total="0">` | Lista con los campos de `/search`; `description` y `bggRating` a `null` |
| `/search` vacío | Lista vacía y **no** llama a `/thing` |
| 401, 429, 500, timeout de conexión | `List.of()` (lista vacía). Se registra un `warn`, no se propaga error. Los tests cubren 500 y fallo de conexión; el 401 y el 429 caen en el mismo `catch` por ser subclases de `RestClientResponseException` |
| 202 agotado | Lista vacía |

## Limitaciones

| Limitación | Detalle |
|------------|---------|
| **Sin token no funciona nada** | `BGG_AUTH_TOKEN` vacío ⇒ 401 ⇒ el cliente devuelve lista vacía. La ficha anterior decía "Auth: no requerida": ya no es cierto |
| **El fallo se cachea 1 hora** | La lista vacía (401, 429, 202 agotado) se guarda en la caché `boardgameSearch` con TTL 3600 s. Un problema puntual de auth o de rate limit deja la búsqueda vacía para esa query durante una hora |
| **El error no llega al cliente** | `GET /boardgames/search` responde **200** con `results` vacío; el usuario no distingue "no hay resultados" de "BGG rechazó la petición" |
| **No se reintenta el 429** | Solo se reintenta el 202. Si BGG limita por rate limit, la búsqueda devuelve vacío |
| **Formato XML** | Requiere Jackson XML (`jackson-dataformat-xml`) y DTOs anidados; no hay JSON oficial |
| **Respuesta asíncrona** | 202 Accepted: la primera respuesta útil puede no ser la buena |
| **`/thing` en lotes de 20** | BGG admite como máximo 20 ids por llamada (lo asume `MAX_THING_IDS = 20`; **no verificado en vivo** por falta de token). Una búsqueda de 100 resultados = 5 llamadas extra |
| **Sin paginación real** | El backend pagina en memoria el bloque que devuelva `/search`; nunca pide una "página 2" a BGG |
| **Sin `bayesaverage`** | Se parsea pero no se expone; el rating que se guarda es `average`, sensible a juegos con pocos votos |
| **Sin filtros de BGG** | No se usan `type` (más allá de `boardgame`), ni orden, ni expansiones, ni `playingtime` |
| **`/collection` sin usar** | El backend no importa la colección del usuario de BGG (y hoy requeriría token) |
| **Documentación no verificable desde aquí** | Las páginas oficiales devuelven 403 a `curl`; no se pudieron confirmar los límites de rate ni el límite exacto de ids de `/thing` |
| **Bloqueo anti-bot** | Evaluar en despliegue: BGG está detrás de Cloudflare y puede responder 403/401 según origen y cabeceras |

## Referencias

- [Using the XML API](https://boardgamegeek.com/using_the_xml_api) (403 a `curl` el 2026-10-15)
- [BGG XML API2 (wiki)](https://boardgamegeek.com/wiki/page/BGG_XML_API2)
- [XML API Terms of Use](https://boardgamegeek.com/xmlapi/termsofuse)
- [Hilo: BGG now requiring authorization tokens](https://boardgamegeek.com/thread/3600185/heads-up-bgg-now-requiring-authorization-tokens-fo)
- [Hilo: Collection API does not require auth token?](https://boardgamegeek.com/thread/3656850/collection-api-does-not-require-auth-token) (contradicho por la prueba del 2026-10-15)
- [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml)
