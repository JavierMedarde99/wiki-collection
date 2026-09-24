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
| **Total cartas** | 70,000+ |
| **Total sets** | 1,049+ |
| **Documentación** | https://scryfall.com/docs/api |
| **Estado** | Activa y mantenida (2026) |

## Endpoints Disponibles

### Cartas

| Uso | Endpoint |
|-----|----------|
| Buscar por nombre (fuzzy) | `GET /cards/named?fuzzy={name}` |
| Buscar por nombre (exacto) | `GET /cards/named?exact={name}` |
| Búsqueda avanzada | `GET /cards/search?q={query}` |
| Carta por ID | `GET /cards/{id}` |
| Carta por colección | `GET /cards/collector/{set}/{number}` |
| Cartas aleatorias | `GET /cards/random` |
| Lista de cartas | `GET /cards?page={n}` |

### Sets

| Uso | Endpoint |
|-----|----------|
| Listar sets | `GET /sets` |
| Set por código | `GET /sets/{code}` |

### Catálogos

| Uso | Endpoint |
|-----|----------|
| Lista de catálogos | `GET /catalogs` |
| Catálogo específico | `GET /catalogs/{name}` |

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

| Parámetro | Descripción | Ejemplo |
|-----------|-------------|---------|
| `q` | Query de búsqueda | `q=t:creature+c:red` |
| `unique` | Eliminar duplicados | `unique=prints` |
| `order` | Ordenar por campo | `order=name`, `order=released`, `order=usd` |
| `dir` | Dirección | `dir=asc`, `dir=desc` |
| `page` | Página | `page=2` |
| `include_extras` | Incluir extras | `include_extras=true` |

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
| `type` | `type` | String | Tipo de carta (ej: "Instant", "Creature") |
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
| `condition` | `condition` | Enum | Estado: MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR |
| `promo` / `eggborne` | — | — | Detectado para determinar foil |
| `quantity` | `quantity` | Integer | Cantidad de copias (solo management) |
| `notes` | `notes` | String | Notas personales |

## Estrategia de Implementación

1. **Scryfall como API primaria** — Búsqueda por nombre, 70k+ cartas, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a DTOs de MagicCard
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 1s
5. **Rate limit** — Delay de 100ms entre peticiones para respetar límites

## Notas Importantes

- **El backend NO expone POST/PUT genéricos para Magic.** Las cartas solo se pueden añadir desde Scryfall con `POST /scryfall/{scryfallId}`
- MagicCardUseCase solo tiene: `search`, `findById`, `addFromScryfall`, `delete`
- Scryfall soporta búsqueda fuzzy tolerante a errores
- Los precios se actualizan diariamente
- Las imágenes están en alta resolución

## Referencias

- [Scryfall API Documentation](https://scryfall.com/docs/api)
- [Scryfall Search Syntax](https://scryfall.com/docs/syntax)
- [Scryfall API Rate Limits](https://scryfall.com/docs/api#rate-limits)
