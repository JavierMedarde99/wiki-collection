---
type: external-api
title: "Scryfall API"
description: "Contrato de Scryfall: cartas Magic, impresiones y rate limits"
tags: [api-externa, magic]
---

# Scryfall API

**Investigación para:** Fase 4 — Colección de Cartas Magic: The Gathering · usada además por la importación de mazos (Fase 23) y por el listado de impresiones (`GET /api/v1/magic/scryfall/{scryfallId}/printings`)

**Verificado en vivo:** 2026-10-15 (peticiones `curl` sin clave contra `https://api.scryfall.com`)

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.scryfall.com` |
| **Auth** | No requerida |
| **Rate limit** | **Por endpoint**: `/cards/search` y `/cards/named` a **2 req/s (500 ms)**; el resto a 10 req/s (100 ms). Ver "Rate limit y caché" |
| **Cabeceras obligatorias** | `User-Agent` (identificando la app) y `Accept` |
| **Gratis** | Sí, totalmente gratuita |
| **Formato** | JSON, codificación UTF-8 |
| **Total impresiones** | 118.740 (suma de `card_count` de los sets) |
| **Total sets** | 1.059 |
| **Tamaño de página** | **Fijo 175** en `/cards/search`. Ignora `page_size`, `per_page` y `limit` |
| **Campo de conteo** | `total_cards` (**no** `total_found`) |
| **`unique` por defecto** | `cards` (si se omite el parámetro) |
| **Búsqueda sin resultados** | **HTTP 404** con `{"object":"error","code":"not_found"}`, no 200 con lista vacía |
| **Documentación** | https://scryfall.com/docs/api |
| **Estado** | Activa y mantenida (2026) |

> Verificado en vivo el 2026-10-15. Respecto a la verificación del 2026-10-02: los sets han
> pasado de 1.053 a 1.059 y el catálogo de nombres vuelve a existir (`/catalog/{name}`).
> Ver "Cambios respecto a fichas anteriores" al final.

## Endpoints

| Uso | Endpoint | Parámetros | Notas |
|-----|----------|-----------|-------|
| Buscar por nombre (fuzzy) | `GET /cards/named` | `fuzzy={name}` | Tolerante a errores. HTTP 200 |
| Buscar por nombre (exacto) | `GET /cards/named` | `exact={name}` | HTTP 404 si no existe |
| Autocompletado | `GET /cards/autocomplete` | **`q={prefijo}`** | `?f=` se ignora en silencio: devuelve `total_values: 0` |
| Búsqueda avanzada | `GET /cards/search` | `q`, `unique`, `order`, `dir`, `page`, `include_extras` | Página fija de 175. **404 si `q` no coincide con nada** |
| Carta por ID de impresión | `GET /cards/{id}` | — | Es el que usa el backend para `addFromScryfall` y para resolver el `oracle_id` |
| Impresiones de una carta | `GET /cards/search` | `q=oracleid:{oracleId}`, `unique=prints`, `order=released`, `page={n}` | **No existe endpoint dedicado**; ver abajo |
| Listar sets | `GET /sets` | — | Lista completa en una respuesta |
| Set por código | `GET /sets/{code}` | — | — |
| Catálogos | `GET /catalog/{name}` | p. ej. `card-names` | **Singular**. `/catalog` y `/catalogs` dan 404 |

### Endpoints que NO existen (verificados 404 el 2026-10-15)

| Endpoint | Estado | Sustituto |
|----------|--------|-----------|
| `GET /cards/{id}/prints` | 404 | `GET /cards/search?q=oracleid:{oracleId}&unique=prints` |
| `GET /cards/{id}/printings` | 404 | Igual que el anterior |
| `GET /cards/collector/{set}/{number}` | 404 | `GET /cards/{set}/{number}` |
| `GET /cards?page={n}` | 404 | `GET /cards/search?q=…&page={n}` |
| `GET /catalogs` y `GET /catalogs/{name}` | 404 | `GET /catalog/{name}` |
| `GET /catalog` a secas | 404 | Necesita el nombre del catálogo |

> Ojo: `/cards` a secas no es un endpoint. La búsqueda vive en `/cards/search`.

### Parámetros de `/cards/search`

| Parámetro | Descripción | Ejemplo | Notas |
|-----------|-------------|---------|-------|
| `q` | Query de búsqueda | `q=t:creature+c:red` | Los filtros (idioma, tipo…) van **dentro** de `q`, no como parámetros sueltos |
| `unique` | Granularidad del resultado | `unique=prints` | Ver tabla de abajo |
| `order` | Ordenar por campo | `order=name`, `order=released`, `order=usd` | `set` y `rarity` también funcionan |
| `dir` | Dirección | `dir=asc`, `dir=desc` | **`dir`**, no `direction`. Por defecto `desc` |
| `page` | Página (base 1) | `page=2` | Siempre 175 resultados, sin excepción |
| `include_extras` | Incluir extras | `include_extras=true` | Aditivos, tokens, etc. |

> `direction=` **no existe**: se ignora en silencio. El parámetro correcto es `dir`.
> `page_size`, `per_page` y `limit` **no existen** aquí: se ignoran y la página sigue siendo de 175.

#### Valores de `unique`

Medido sobre Llanuras (`oracle_id: bc71ebf6-2056-41f7-be35-b2e5c34afa99`) el 2026-10-15:

| Valor | Devuelve | Total Llanuras |
|-------|----------|----------------|
| `unique=prints` | Una fila por **impresión física** (set + número + finish) | 917 |
| `unique=art` | Una fila por **arte distinto** | 392 |
| `unique=set` | Una fila por expansión | 1 |
| `unique=none` | Una fila por coincidencia de nombre | 1 |
| `unique=oracle` | Una fila por **carta** (agrupa todas sus reimpresiones) | 1 |
| `unique=cards` | Valor por defecto si se omite `unique` | — |

## Búsqueda por nombre

### Fuzzy (tolerante a errores)

```
GET /cards/named?fuzzy=lightnin+bolt     → 200, "Lightning Bolt"
GET /cards/named?fuzzy=Atraxa Praetors Voice → 200, "Atraxa, Praetors' Voice"
```

### Exacta

```
GET /cards/named?exact=Lightning+Bolt    → 200
GET /cards/named?exact=Zzzz+Not+A+Card   → 404 {"code":"not_found"}
```

`GET /cards/named?exact=Lightning+Bolt&lang=es` devolvió **200 con `lang: "en"`**: en esta
prueba el parámetro `lang` no cambió el idioma del resultado. No se ha verificado en qué casos
Scryfall lo respeta.

## Impresiones y arte alternativo

Una carta de Scryfall tiene dos identificadores, y la diferencia es la clave de todo esto:

| ID | Identifica | Ejemplo |
|----|-----------|---------|
| `id` | **Una impresión concreta** | `7673784e-…` = Lightning Bolt de MSC #806 |
| `oracle_id` | **La carta**, a través de todas sus reimpresiones | `4457ed35-…` |

### Obtener todas las impresiones de una carta

El objeto carta trae ya la URL construida en `prints_search_uri` (verificado):

```
GET /cards/{id}
  → "prints_search_uri": "https://api.scryfall.com/cards/search?order=released&q=oracleid%3A4457ed35-…&unique=prints"

GET /cards/search?q=oracleid:{oracle_id}&unique=prints&order=released&page={n}
```

`order=released` sale **descendente** sin necesidad de `dir`. El backend lo consume así para
`GET /api/v1/magic/scryfall/{scryfallId}/printings`.

### Filtrar por idioma

El idioma va dentro de `q`. Como parámetro suelto se ignora (medido sobre Llanuras, 2026-10-15):

```
GET /cards/search?q=oracleid:{oracle_id}+lang:es&unique=prints   ✅ 432 de 917
GET /cards/search?q=oracleid:{oracle_id}&unique=prints&lang=es   ❌ 917, se ignora
```

### `variation` y `variation_of`

Scryfall tiene un sistema de arte alternativo, pero **es un caso legado y no sirve para el caso
de uso normal** (datos de la ficha anterior, no reverificados el 2026-10-15):

- Solo **83 cartas** de toda la base tienen `variation: true`
- La carta base **ya no tiene** el array `variations` → no hay forma de listar sus variantes
- `q=variation_of:{id}` no es una clave de búsqueda válida (404)
- Las variaciones marcan el `collector_number` con un dagger: `#52†`

Para el arte alternativo real hay que usar `unique=art` junto con los campos descriptivos:

| Campo | Indica |
|-------|--------|
| `frame_effects` | Inusual, eclipse, etc. |
| `full_art` | Arte a sangre completa |
| `border_color` | `black`, `borderless`, `textless`… |
| `promo_types` | `showcase`, `extendedart`, `galaxyfoil`, `serialized`… |
| `finishes` | `nonfoil`, `foil` |
| `illustrated` | Impresión ilustrada |
| `security_stamp` | `oval`, `triangle`, `acorn`, `circle`, `arena`, `heart` |

### `image_uris` tiene 12 variantes

Verificado en vivo el 2026-10-15: `small`, `normal`, `large`, `png`, `jpeg`, `art_crop`,
`border_crop`, `thumb`, `grid`, `display`, `art`, `crop`. El noveno, `jpeg`, es nuevo respecto a
la ficha anterior. El objeto carta trae además `highres_image`, `image_status` e
`image_updated_at`.

## Mapeo: Scryfall → modelo interno

### `MagicCard` (`MagicCardMapper.map()`)

Los campos que el backend copia realmente al añadir una carta con
`POST /api/v1/magic/scryfall/{scryfallId}`:

| Campo Scryfall | Campo interno (`MagicCard`) | Tipo | Notas |
|----------------|-----------------------------|------|-------|
| `id` | `scryfallId` | String | ID de **impresión concreta** |
| `oracle_id` | `oracleId` | String | ID de carta a través de reimpresiones |
| `name` | `name` | String | Nombre (siempre en inglés desde `/cards/named`) |
| `released_at` | `releaseDate` | String | Fecha del set, `AAAA-MM-DD` |
| `mana_cost` | `manaCost` | String | Ej. `{2}{R}{R}` |
| `cmc` | `convertedManaCost` | Double | Coste convertido |
| `type_line` | `type` | String | **Renombrado**: el campo origen es `type_line` |
| `oracle_text` | `text` | String | Texto/rules |
| `power` | `power` | String | Criaturas |
| `toughness` | `toughness` | String | Criaturas |
| `loyalty` | `loyalty` | String | Planeswalkers |
| `colors` | `colors` | List de String | |
| `color_identity` | `colorIdentity` | List de String | |
| `keywords` | `keywords` | List de String | |
| `rarity` | `rarity` | String | common, uncommon, rare, mythic |
| `set` | `setCode` | String | Código del set |
| `set_name` | `setName` | String | |
| `artist` | `artist` | String | |
| `frame` | `frame` | String | |
| `border_color` | `borderColor` | String | |
| `layout` | `layout` | String | |
| `legalities` | `legalities` | Map de String a String | |
| `prices.usd` | `priceUsd` | String | |
| `prices.eur` | `priceEur` | String | |
| `image_uris.normal` | `imageUrl` | String | |
| `image_uris.large` | `imageLargeUrl` | String | |
| `image_uris.art_crop` | `artCropUrl` | String | |
| `lang` | — | — | **No se mapea**: `MagicCard.language` queda a `null`, aunque el mapper declara el campo `lang` |
| `full_art`, `padding`, `promo_types`, `frame_effects` | — | — | Describen el arte; no se persisten en `MagicCard` |
| `collector_number` | — | — | Número dentro del set. Las variaciones llevan dagger (`52†`) |
| `finishes` | — | — | `nonfoil` / `foil`; no se persiste en `MagicCard` |
| `promo_prices` | — | — | El record lo declara, pero no se expone ni se usa (comentario explícito en el mapper) |

Campos que **no vienen de Scryfall** y rellena el backend:

| Campo interno | Origen |
|---------------|--------|
| `id`, `dateAdded` | Mongo al guardar |
| `ownerId`, `userOwned` | Usuario autenticado |
| `quantity` | Parámetro `quantity` de `POST /scryfall/{scryfallId}` (o `POST /{id}/copies`) |
| `condition`, `notes`, `isFoil` | Dato del usuario; Scryfall no los aporta |

### `MagicCardPrinting` (`MagicCardMapper.toPrinting()`)

Cada fila de `GET /api/v1/magic/scryfall/{scryfallId}/printings`:

| Campo Scryfall | Campo interno (`MagicCardPrinting`) |
|----------------|-------------------------------------|
| `id` | `scryfallId` (es el que acepta `POST /api/v1/magic/scryfall/{scryfallId}`) |
| `name` | `name` |
| `set` | `setCode` |
| `set_name` | `setName` |
| `collector_number` | `collectorNumber` |
| `rarity` | `rarity` |
| `artist` | `artist` |
| `released_at` | `releasedAt` |
| `lang` | `lang` |
| `image_uris.normal` | `imageUrl` |
| `image_uris.art_crop` | `artCropUrl` |
| `finishes` | `finishes` |
| `full_art` | `fullArt` (primitivo `boolean`; Scryfall omite el campo en cartas antiguas) |
| `promo_types` | `promoTypes` |
| `frame_effects` | `frameEffects` |
| `border_color` | `borderColor` |
| `prices.usd` | `priceUsd` |
| `prices.eur` | `priceEur` |

`MagicCardPrinting` **no** expone `oracleId` ni `quantity`/`ownerId` (hay test que lo comprueba).

### Campos renombrados en la API de Scryfall

| Campo antiguo | Nombre actual |
|---------------|---------------|
| `type` | `type_line` |
| `set_code` | `set` |
| `text` | `oracle_text` |
| `variations` (array) | `variation` (boolean) + `variation_of` (UUID) |
| `total_found` | `total_cards` |

## Configuración en el proyecto

`src/main/resources/application.properties`:

```properties
# Scryfall API (Magic: The Gathering)
# URL base de la API publica de Scryfall.
scryfall.api.base-url=https://api.scryfall.com
# Reintentos ante 5xx/timeout (ademas del intento inicial).
scryfall.api.retry-attempts=3
# Espera entre reintentos, en milisegundos.
scryfall.api.retry-delay-ms=1000
# Pausa minima entre peticiones, en milisegundos. Scryfall pide no superar
# ~10 peticiones/segundo, asi que 100 ms es el maximo seguro por defecto.
scryfall.api.rate-limit-delay-ms=100
```

> El comentario de `rate-limit-delay-ms` está desactualizado: Scryfall limita `/cards/search` y
> `/cards/named` a **2 req/s (500 ms)**, no a 10 req/s. Ver "Limitaciones".

Clientes y timeouts:

- `ScryfallClientConfig` crea un `RestClient` con `HttpClientProperties`:
  `http.client.connect-timeout=5s`, `http.client.read-timeout=10s`.
- `HttpClientProperties` **no añade cabecera `User-Agent`** propia; el `RestClient` usa la
  que ponga la librería por defecto.
- El puerto de salida es `ExternalMagicCardCatalogClient`, implementado por
  `infrastructure/adapter/out/scryfall/ScryfallClient`.

Endpoints que consume el backend, con el resto de la app:

| Método del puerto | Llamada real a Scryfall | Origen |
|-------------------|-------------------------|--------|
| `search(query)` | `GET /cards/search?q={query}` | `GET /api/v1/magic/search` |
| `searchCommanders(colors)` | `GET /cards/search?q=is:commander id<={colors}` | `GET /api/v1/magic/commanders` |
| `findById(id)` | `GET /cards/{id}` | `POST /api/v1/magic/scryfall/{scryfallId}` y `POST /api/v1/magic/scryfall/{scryfallId}/printings` (resolución de `oracle_id`) |
| `findByName(name)` | `GET /cards/named?fuzzy={name}` | — |
| `searchByNameExact(name)` | `GET /cards/search?q=name:"{name}"&unique=oracle` | Importación de mazos (Fase 23) |
| `searchSuggestions(name)` | `GET /cards/search?q={name}&unique=oracle` | Sugerencias de la importación; recorta a 5 en cliente |
| `findPrintings(oracleId, page)` | `GET /cards/search?q=oracleid:{id}&unique=prints&order=released&page={page+1}` | `GET /api/v1/magic/scryfall/{scryfallId}/printings` |

Detalles del comportamiento de error (código real):

- Los `4xx` **no se reintentan** (`executeWithRetry` los relanza tal cual); `5xx` y timeouts se
  reintentan hasta 3 veces más con 1 s de espera.
- `search()` y `searchCommanders()` **tragan** cualquier error y devuelven lista vacía.
- `searchByNameExact()` / `searchSuggestions()` distinguen el **404 como "no existe"**
  (lista vacía, sin reintentar) del resto de errores, que se propagan para no guardar un mazo
  incompleto.
- `findById()` y `findPrintings()` propagan los errores.
- `findPrintings()` pagina en base 1 hacia Scryfall (`page + 1`); nuestra API es base 0.

## Rate limit y caché

### Límites reales publicados por Scryfall (verificados en su documentación el 2026-10-15)

| Endpoint | Límite |
|----------|--------|
| `/cards/search` | 2 req/s (500 ms) |
| `/cards/named` | 2 req/s (500 ms) |
| `/cards/random`, `/cards/collection`, `/cards/manifest` | 2 req/s / 10 req/min |
| Todos los demás | 10 req/s (100 ms) |
| `*.scryfall.io` (imágenes) | Sin límite |

Un **429** limita el acceso durante **30 segundos**; ignorarlo repetidamente puede provocar un
bloqueo temporal o permanente. Scryfall pide cachear al menos 24 h los datos descargados.

Cabeceras observadas en las respuestas 200 el 2026-10-15: **ninguna `x-ratelimit-*` ni
`Retry-After`**. La respuesta incluye `cache-control: public, max-age=…` (172800 s en
`/cards/named`, 57600 s en `/cards/search`) y `etag`.

### Caché del backend

Caffeine en memoria (`CacheConfig`), sin Redis. Claves relevantes para Scryfall:

| Caché | Método | Clave | Notas |
|-------|--------|-------|-------|
| `magicSearch` | `search(query)` | `#query` | TTL de `app.cache.ttl.magic=3600s` |
| `commanderSearch` | `searchCommanders(colors)` | `#colors` | Igual TTL |
| `magicPrintings` | `findPrintings(oracleId, page)` | `#oracleId + ':' + #page` | `unless = "#result.content.isEmpty()"`: las páginas vacías **no** se cachean |

`app.cache.default-ttl=3600s` y `app.cache.max-size=1000`. Las escrituras (`addFromScryfall`,
`delete`, `addCopies`) invalidan `magicList`/`magicDetail` y los mazos cacheados.

## Limitaciones

| Limitación | Detalle |
|------------|---------|
| **Ritmo mal ajustado** | `rate-limit-delay-ms=100` deja el ritmo en 10 req/s, pero `/cards/search` y `/cards/named` solo admiten **2 req/s**. El backend puede recibir 429 con facilidad; el comentario del `.properties` dice "~10 peticiones/segundo" |
| **429 silencioso** | Al ser 4xx no se reintenta, y `search()` lo convierte en lista vacía: una sobrecarga de Scryfall se ve como "sin resultados" en `/magic/search` |
| **`search()` no pagina** | No envía `page` a Scryfall, así que `/magic/search` está topado a los **175** primeros resultados |
| **Página fija** | Scryfall ignora `page_size`, `per_page` y `limit` en `/cards/search`: no hay forma de pedir menos de 175 (verificado con `page_size=1` → 175) |
| **`language` no se rellena** | `MagicCardMapper.map()` no copia `lang` a `MagicCard.language`; las cartas guardadas desde Scryfall quedan sin idioma |
| **Sin filtro de idioma** | Scryfall lo soporta (`q=… lang:es`), el backend no lo expone |
| **Nombre solo en inglés** | `/cards/named?exact=` busca por el nombre en inglés; `&lang=es` no cambió el resultado en la prueba |
| **Sin `User-Agent` propio** | Scryfall exige una cabecera `User-Agent` que identifique la app; el `RestClient` no la fija explícitamente |
| **Sin endpoint de impresiones en Scryfall** | `/cards/{id}/prints` da 404; hay que buscar por `oracleid` + `unique=prints` |
| **Cambios de API sin aviso** | Varios endpoints/campos de fichas antiguas ya no existen; conviene reverificar antes de citar campos |

## Referencias

- [Scryfall API Documentation](https://scryfall.com/docs/api)
- [Scryfall API Rate Limits](https://scryfall.com/docs/api/rate-limits)
- [Scryfall Search Syntax](https://scryfall.com/docs/syntax)

## Cambios respecto a fichas anteriores

Corregido el 2026-10-15 tras leer el código y llamar a la API:

- **`/cards/autocomplete` usa `q=`, no `f=`.** `f=` responde 200 con `total_values: 0`.
- **`/catalog/{name}` sí existe** (singular; p. ej. `/catalog/card-names` → 200). La ficha
  anterior lo daba por eliminado sin sustituto.
- **`/cards/search` responde 404 cuando no hay coincidencias**, no 200 con lista vacía.
- **`/cards/{id}/prints` y `/cards/{id}/printings` dan 404**; las impresiones se obtienen con
  `q=oracleid:…&unique=prints`.
- **`unique=oracle` existe** y es lo que usa la importación de mazos (Fase 23); si se omite
  `unique`, Scryfall usa `cards`.
- **Sí hay endpoint de impresiones en nuestro backend**:
  `GET /api/v1/magic/scryfall/{scryfallId}/printings` (la ficha anterior decía que no existía).
- **Los sets son 1.059 y las impresiones 118.740** (antes 1.053 y 118.467+).
- **Llanuras tiene 917 impresiones** y 432 en español (antes 955 y 433).
- **`image_uris` tiene 12 claves**: se añade `jpeg`.
- **`lang` no se mapea** a `MagicCard.language`.
- El rate limit de `/cards/search` y `/cards/named` es **2 req/s**, no 10.
