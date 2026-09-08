# APIs Externas — Cartas Magic: The Gathering

## Estado: Planificado (Fase 4)

---

## API Seleccionada

### Scryfall API

- **Base URL:** `https://api.scryfall.com`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo (recomendado)
- **Gratis:** Sí, totalmente gratuita
- **Formato:** JSON
- **Total cartas:** 70,000+
- **Total sets:** 1,049+
- **Documentación:** https://scryfall.com/docs/api
- **Estado:** Activa y mantenida (2026-09)

---

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

---

## Búsqueda por Nombre

### Fuzzy (tolerante a errores)

```
GET /cards/named?fuzzy=lightning+bolt
```

**Respuesta:**
```json
{
  "object": "card",
  "id": "7673784e-db4b-43a1-8d55-1bb9fc1e284f",
  "oracle_id": "4457ed35-7c10-48c8-9776-456485fdf070",
  "name": "Lightning Bolt",
  "lang": "en",
  "released_at": "2026-06-26",
  "uri": "https://api.scryfall.com/cards/7673784e-db4b-43a1-8d55-1bb9fc1e284f",
  "scryfall_uri": "https://scryfall.com/card/msc/806/lightning-bolt",
  "layout": "normal",
  "highres_image": true,
  "image_status": "highres_scan",
  "mana_cost": "{R}",
  "cmc": 1.0,
  "type_line": "Instant",
  "oracle_text": "Lightning Bolt deals 3 damage to any target.",
  "colors": ["R"],
  "color_identity": ["R"],
  "keywords": [],
  "rarity": "uncommon",
  "set": "msc",
  "set_name": "Marvel Super Heroes Commander",
  "artist": "Milivoj Ćeran",
  "frame": "2015",
  "border_color": "black",
  "legalities": {
    "standard": "not_legal",
    "future": "not_legal",
    "historic": "legal",
    "timeless": "legal",
    "gladiator": "legal",
    "pioneer": "legal",
    "modern": "legal",
    "legacy": "legal",
    "pauper": "legal",
    "vintage": "legal",
    "penny": "legal",
    "commander": "legal",
    "oathbreaker": "legal",
    "brawl": "legal",
    "historicbrawl": "legal",
    "alchemy": "legal",
    "paupercommander": "legal",
    "duel": "legal",
    "oldschool": "not_legal",
    "premodern": "legal",
    "predh": "legal"
  },
  "prices": {
    "usd": "0.65",
    "usd_foil": null,
    "usd_etched": null,
    "eur": "2.02",
    "eur_foil": null,
    "tix": null
  },
  "image_uris": {
    "small": "https://cards.scryfall.io/small/front/7/6/7673784e...jpg",
    "normal": "https://cards.scryfall.io/normal/front/7/6/7673784e...jpg",
    "large": "https://cards.scryfall.io/large/front/7/6/7673784e...jpg",
    "png": "https://cards.scryfall.io/png/front/7/6/7673784e...png",
    "art_crop": "https://cards.scryfall.io/art_crop/front/7/6/7673784e...jpg",
    "border_crop": "https://cards.scryfall.io/border_crop/front/7/6/7673784e...jpg"
  }
}
```

### Exacta

```
GET /cards/named?exact=Black+Lotus
```

---

## Búsqueda Avanzada

Permite filtrar con el lenguaje de búsqueda de Scryfall.

```
GET /cards/search?q=t:creature+c:red+r:mythic
```

**Parámetros comunes:**

| Parámetro | Descripción | Ejemplo |
|-----------|-------------|---------|
| `q` | Query de búsqueda | `q=t:creature+c:red` |
| `unique` | Eliminar duplicados | `unique=prints` |
| `order` | Ordenar por campo | `order=name`, `order=released`, `order=usd` |
| `dir` | Dirección | `dir=asc`, `dir=desc` |
| `page` | Página | `page=2` |
| `include_extras` | Incluir extras | `include_extras=true` |

**Respuesta:**
```json
{
  "object": "list",
  "total_cards": 568,
  "has_more": false,
  "data": [
    {
      "object": "card",
      "name": "Abaddon the Despoiler",
      "mana_cost": "{2}{U}{B}{R}",
      "type_line": "Legendary Creature — Astartes Warrior",
      "set_name": "Warhammer 40,000 Commander"
    }
  ]
}
```

---

## Mapeo de Campos Scryfall → MagicCard

| Campo Scryfall | Campo MagicCard | Tipo | Notas |
|----------------|-----------------|------|-------|
| `id` | `scryfallId` | String | ID único de Scryfall |
| `oracle_id` | `oracleId` | String | ID de oracle (único por carta) |
| `name` | `name` | String | Nombre de la carta |
| `lang` | `language` | String | Idioma (en, es, fr, etc.) |
| `released_at` | `releaseDate` | String | Fecha de lanzamiento |
| `mana_cost` | `manaCost` | String | Coste de maná |
| `cmc` | `convertedManaCost` | Double | Coste de maná convertido |
| `type_line` | `type` | String | Tipo de carta |
| `oracle_text` | `text` | String | Texto/rules de la carta |
| `power` | `power` | String | Fuerza (criaturas) |
| `toughness` | `toughness` | String | Resistencia (criaturas) |
| `loyalty` | `loyalty` | String | Lealtad (planeswalkers) |
| `colors` | `colors` | List<String> | Colores de la carta |
| `color_identity` | `colorIdentity` | List<String> | Identidad de color |
| `keywords` | `keywords` | List<String> | Palabras clave |
| `rarity` | `rarity` | String | Rareza |
| `set` | `setCode` | String | Código del set |
| `set_name` | `setName` | String | Nombre del set |
| `artist` | `artist` | String | Artista |
| `frame` | `frame` | String | Año del frame |
| `border_color` | `borderColor` | String | Color del borde |
| `layout` | `layout` | String | Layout de la carta |
| `legalities` | `legalities` | Map<String,String> | Legalidades por formato |
| `prices.usd` | `priceUsd` | String | Precio en USD |
| `prices.eur` | `priceEur` | String | Precio en EUR |
| `image_uris.normal` | `imageUrl` | String | URL de imagen normal |
| `image_uris.large` | `imageLargeUrl` | String | URL de imagen grande |
| `image_uris.art_crop` | `artCropUrl` | String | URL de arte recortado |

---

## Estrategia de Implementación

1. **Scryfall como API primaria** — Búsqueda por nombre, 70k+ cartas, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a DTOs de MagicCard
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Cache** — Caché de resultados (TTL 1 hora) para reducir llamadas

### Flujo de Búsqueda

```
1. Cliente → GET /api/magic/search?name=lightning+bolt
2. Backend → Scryfall API (/cards/named?fuzzy=lightning+bolt)
3. Mapear a DTO → Convertir a JSON → Devolver
4. Si no hay resultados → 404 Not Found
```

### Flujo de Detalle

```
1. Cliente → GET /api/magic/{id}
2. Backend → Scryfall API (/cards/{id})
3. Mapear a MagicCard detallado → Convertir a JSON → Devolver
```

---

## Endpoints Planificados (Backend)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/magic/search?name={query}` | Buscar cartas por nombre |
| GET | `/api/magic/{id}` | Obtener detalle de una carta |
| POST | `/api/magic` | Crear carta en colección local |
| GET | `/api/magic` | Listar colección local |
| PUT | `/api/magic/{id}` | Actualizar carta en colección |
| DELETE | `/api/magic/{id}` | Eliminar carta de colección |

---

## Decisiones Pendientes

- [ ] Definir estrategia de cache (Redis vs caché en memoria)
- [ ] Implementar cliente Scryfall en backend
- [ ] Tests con mock server

---

## Referencias

- [Scryfall API Documentation](https://scryfall.com/docs/api)
- [Scryfall Search Syntax](https://scryfall.com/docs/syntax)
- [Scryfall API Rate Limits](https://scryfall.com/docs/api#rate-limits)
