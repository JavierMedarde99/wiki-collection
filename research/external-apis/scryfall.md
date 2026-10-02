# Scryfall API

**Investigación para:** Fase 4 — Colección de Cartas Magic: The Gathering

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.scryfall.com` |
| **Auth** | No requerida |
| **Rate limit** | ~10 requests/segundo (recomendado) |
| **Gratis** | Sí, totalmente gratuita |
| **Formato** | JSON |
| **Total impresiones** | 118.467+ (suma de `card_count` de los 1.053 sets) |
| **Total sets** | 1.053 |
| **Tamaño de página** | **Fijo 175** en `/cards/search`. Ignora `page_size`, `per_page` y `limit` |
| **Campo de conteo** | `total_cards` (**no** `total_found`) |
| **Documentación** | https://scryfall.com/docs/api |
| **Estado** | Activa y mantenida (2026) |

> Verificado en vivo el 2026-10-02. Scryfall cambió su API: varios endpoints y campos que
> se documentaban antes ya no existen. Ver "Cambios de la API" más abajo.

## Endpoints Disponibles

### Cartas

| Uso | Endpoint | Estado |
|-----|----------|--------|
| Buscar por nombre (fuzzy) | `GET /cards/named?fuzzy={name}` | ✅ |
| Buscar por nombre (exacto) | `GET /cards/named?exact={name}` | ✅ |
| Autocompletado | `GET /cards/autocomplete?f={prefix}` | ✅ |
| Búsqueda avanzada | `GET /cards/search?q={query}` | ✅ |
| Carta por ID | `GET /cards/{id}` | ✅ |
| Carta por set y número | `GET /cards/{set}/{number}` | ✅ |
| Cartas aleatorias | `GET /cards/random` | ✅ |
| **Todas las impresiones** | `GET /cards/search?order=released&q=oracleid:{oracleId}&unique=prints` | ✅ |

### Sets

| Uso | Endpoint | Estado |
|-----|----------|--------|
| Listar sets | `GET /sets` | ✅ |
| Set por código | `GET /sets/{code}` | ✅ |

### Endpoints eliminados

| Endpoint anterior | Estado | Sustituto |
|------------------|--------|-----------|
| `GET /cards/collector/{set}/{number}` | ❌ 404 | `GET /cards/{set}/{number}` |
| `GET /cards?page={n}` | ❌ 404 | `GET /cards/search?q=…&page={n}` |
| `GET /catalogs` y `GET /catalogs/{name}` | ❌ 404 | — (sin sustituto) |
| `GET /cards/{id}/printings` | ❌ 404 | `prints_search_uri` en el objeto carta |
| `GET /catalog` | ❌ 404 | — |

> Ojo: `/cards` a secas no es un endpoint. La búsqueda vive en `/cards/search`.

## Búsqueda por Nombre

### Fuzzy (tolerante a errores)
```
GET /cards/named?fuzzy=lightning+bolt
```

### Exacta
```
GET /cards/named?exact=Black+Lotus
```

## Búsqueda Avanzada

Permite filtrar con el lenguaje de búsqueda de Scryfall:
```
GET /cards/search?q=t:creature+c:red+r:mythic
```

### Parámetros comunes

| Parámetro | Descripción | Ejemplo | Notas |
|-----------|-------------|---------|-------|
| `q` | Query de búsqueda | `q=t:creature+c:red` | Los filtros (idioma, tipo…) van **dentro** de `q`, no como parámetros sueltos |
| `unique` | Granularidad del resultado | `unique=prints` | Ver tabla de abajo |
| `order` | Ordenar por campo | `order=name`, `order=released`, `order=usd` | `set` y `rarity` también funcionan |
| `dir` | Dirección | `dir=asc`, `dir=desc` | **`dir`**, no `direction`. Por defecto `desc` |
| `page` | Página (base 1) | `page=2` | Siempre 175 resultados, sin excepción |
| `include_extras` | Incluir extras | `include_extras=true` | ✅ funciona |

> `direction=` **no existe**: se ignora en silencio. El parámetro correcto es `dir`.

#### Valores de `unique`

| Valor | Devuelve | Llanuras |
|-------|----------|----------|
| `unique=prints` | Una fila por **impresión física** (set + número + finish) | 955 |
| `unique=art` | Una fila por **arte distinto** | 389 |
| `unique=set` | Una fila por expansión | 1 |
| `unique=none` | Una fila por coincidencia de nombre | 1 |

## Impresiones y arte alternativo

Una carta de Scryfall tiene dos identificadores, y la diferencia es la clave de todo esto:

| ID | Identifica | Ejemplo (Llanuras) |
|----|-----------|--------------------|
| `id` | **Una impresión concreta** | `dce15387-…` = Star Trek #325 |
| `oracle_id` | **La carta**, a través de todas sus reimpresiones | `b34bb2dc-…` |

### Obtener todas las impresiones de una carta

El objeto carta trae ya la URL construida en `prints_search_uri`:

```
GET /cards/{id}
  → "prints_search_uri": "https://api.scryfall.com/cards/search?order=released&q=oracleid%3Ab34bb2dc-…&unique=prints"

GET /cards/search?order=released&q=oracleid:{oracle_id}&unique=prints&page={n}
```

`order=released` sale **descendente** sin necesidad de `dir`.

### Filtrar por idioma

El idioma va dentro de `q`. Como parámetro suelto se ignora:

```
GET /cards/search?q=oracleid:{oracle_id}+lang:es&unique=prints   ✅ 433 de 955
GET /cards/search?q=oracleid:{oracle_id}&unique=prints&lang=es   ❌ 955, se ignora
```

### `variation` y `variation_of`

Scryfall tiene un sistema de arte alternativo, pero **es un caso legado y no sirve para el caso de uso normal**:

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

## Mapeo de Campos: Scryfall → MagicCard (Modelo Interno)

| Campo Scryfall | Campo Interno (MagicCard) | Tipo | Notas |
|----------------|---------------------------|------|-------|
| `id` | `scryfallId` | String | ID único de Scryfall |
| `oracle_id` | `oracleId` | String | ID de oracle (único por carta) |
| `name` | `name` | String | Nombre de la carta |
| `lang` | `language` | Enum | Idioma (en, es, fr, etc.) |
| `released_at` | `releaseDate` | String | Fecha de lanzamiento del set |
| `mana_cost` | `manaCost` | String | Coste de maná (ej: `{2}{R}{R}`) |
| `cmc` | `convertedManaCost` | Double | Coste de maná convertido |
| `type_line` | `type` | String | Tipo de carta. **Renombrado**: ya no existe `type` |
| `oracle_text` | `text` | String | Texto/rules de la carta |
| `power` | `power` | String | Fuerza (criaturas) |
| `toughness` | `toughness` | String | Resistencia (criaturas) |
| `loyalty` | `loyalty` | String | Lealtad (planeswalkers) |
| `colors` | `colors` | List<String> | Colores de la carta |
| `color_identity` | `colorIdentity` | List<String> | Identidad de color |
| `keywords` | `keywords` | List<String> | Palabras clave |
| `rarity` | `rarity` | String | Rareza (common, uncommon, rare, mythic) |
| `set` | `setCode` | String | Código del set |
| `set_name` | `setName` | String | Nombre del set |
| `artist` | `artist` | String | Artista de la ilustración |
| `frame` | `frame` | String | Año del frame |
| `border_color` | `borderColor` | String | Color del borde |
| `layout` | `layout` | String | Layout de la carta |
| `legalities` | `legalities` | Map<String,String> | Legalidades por formato |
| `prices.usd` | `priceUsd` | String | Precio en USD |
| `prices.eur` | `priceEur` | String | Precio en EUR |
| `image_uris.normal` | `imageUrl` | String | URL de imagen normal |
| `image_uris.large` | `imageLargeUrl` | String | URL de imagen grande |
| `image_uris.art_crop` | `artCropUrl` | String | URL de arte recortado |
| `collector_number` | — | String | Número dentro del set. Las variaciones llevan dagger (`52†`) |
| `finishes` | `isFoil` (derivado) | List<String> | `nonfoil` / `foil` |
| `full_art`, `frame_effects`, `border_color`, `promo_types` | — | — | Describen el arte; hoy no se persisten |
| `condition` | `condition` | Enum | **No viene de Scryfall.** Es dato del usuario: MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR |
| `quantity` | `quantity` | Integer | Cantidad de copias (solo management) |
| `notes` | `notes` | String | Notas personales |

### `image_uris` tiene 11 variantes

`small`, `normal`, `large`, `png`, `art_crop`, `border_crop`, `thumb`, `grid`, `display`,
`art`, `crop`. Además el objeto carta trae `highres_image`, `image_status` e
`image_updated_at`.

### Campos que ya no existen

| Campo antiguo | Sustituto |
|---------------|-----------|
| `type` | `type_line` |
| `set_code` | `set` |
| `variations` (array) | `variation` (boolean) + `variation_of` (UUID) |
| `total_found` (en la respuesta) | `total_cards` |
| `text` | `oracle_text` |

## Estrategia de Implementación

1. **Scryfall como API primaria** — Búsqueda por nombre, 118k+ impresiones, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a DTOs de MagicCard
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 1s
5. **Rate limit** — Delay de 100ms entre peticiones para respetar límites

## Notas Importantes

- **El backend NO expone POST/PUT genéricos para Magic.** Las cartas solo se pueden añadir desde Scryfall con `POST /scryfall/{scryfallId}`
- `MagicCardUseCase` tiene: `search` (2 sobrecargas), `findById`, `addFromScryfall`, `delete`
- **No hay endpoint para obtener las impresiones de una carta.** `MagicCard` guarda una sola impresión (`setCode`, `setName`, `rarity`, `artist`, `frame`, `borderColor`), pero sí persiste `oracleId`, que es la clave para agruparlas
- Scryfall soporta búsqueda fuzzy tolerante a errores
- Los precios se actualizan diariamente
- Las imágenes están en alta resolución

## Limitaciones conocidas

| Limitación | Detalle |
|------------|---------|
| `ScryfallClient.search()` no pagina | No envía `page` a Scryfall, así que `/magic/search` está topado a los **175** primeros resultados |
| Página fija | Scryfall ignora `page_size`, `per_page` y `limit`. No hay forma de pedir menos de 175 |
| Sin filtro de idioma implementado | Scryfall lo soporta (`q=… lang:es`), el backend no lo expone |
| Idioma de la carta | `/cards/named?exact=` solo busca por el nombre inglés salvo que se pase `lang` |

## Referencias

- [Scryfall API Documentation](https://scryfall.com/docs/api)
- [Scryfall Search Syntax](https://scryfall.com/docs/syntax)
- [Scryfall API Rate Limits](https://scryfall.com/docs/api#rate-limits)
