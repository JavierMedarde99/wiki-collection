# HobbyQuantHQ — API de Funko Pop

**Investigación para:** futura colección de Funkos (sin fase asignada)
**Estado:** ✅ Activa — **API en beta** (contrato verificado en vivo el 2026-10-11)

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://hobbyquanthq.com/api/v1` |
| **Auth** | Header `x-api-key` |
| **Rate limit** | No documentado ni expuesto en cabeceras; la API está en beta |
| **Gratis** | Sí, durante la beta |
| **Registro** | No necesario en beta (clave de desarrollo `dev-key-change-me`) |
| **Formato** | JSON |
| **Catálogo** | 15.458 pops · 3.776 exclusividades · 3.805 variantes · 303 franquicias · 133 categorías |
| **Documentación** | https://hobbyquanthq.com/api-docs |
| **Estado** | Beta — el catálogo y los campos pueden evolucionar |

## Autenticación

```http
x-api-key: TU_API_KEY
```

- En beta funciona la clave pública `dev-key-change-me`.
- Sin cabecera: **422** con `{"detail":[{"type":"missing","loc":["header","x-api-key"]}]}`.
- **No hardcodear la clave en el frontend** — el patrón del proyecto es clave en backend + variable de entorno.

## Endpoints Verificados

| Uso | Endpoint | Parámetros | Notas |
|-----|----------|------------|-------|
| Listar pops | `GET /pops` | `page`, `limit`, `series`, `category`, `vaulted`, `exclusive`, `sort`, `order` | `limit` máximo **100** (101 → 422) |
| Obtener un pop | `GET /pops/{id}` | — | 200 con el objeto del pop |
| Buscar | `GET /search?q={q}` | `q` (mínimo 2 caracteres), `limit` | Busca por nombre, serie y número |
| Categorías | `GET /categories` | — | Devuelve `[{"name","count"}]` |

La raíz `/api/v1/` responde **308** (redirección); usar los endpoints concretos. No existe `api.hobbyquanthq.com`.

## Forma Real de las Respuestas

```json
{
  "items": [ { "…": "Pop" } ],
  "total": 15458,
  "page": 1,
  "limit": 2,
  "pages": 7729
}
```

```json
{
  "results": [ { "…": "Pop" } ],
  "query": "Batman",
  "total": 2
}
```

> ⚠️ **Corrige documentación de terceros:** algunas notas circulan `{"data": [...]}` como sobre de respuesta. **No existe**: `/pops` usa `items` y `/search` usa `results`.

## Campos de un Pop

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | int | Identificador externo (PK de la fuente) |
| `name` | string | Nombre del Pop |
| `series` | string\|null | Serie (p. ej. `Marvel`) |
| `number` | string\|null | Número del Pop (p. ej. `"01"`) |
| `category` | string\|null | Categoría (p. ej. `Movies`, `Animation`) |
| `exclusive` | string\|null | Exclusividad (p. ej. `Walmart`) |
| `image_url` | string\|null | Imagen alojada en `images.hobbydb.com` |
| `vaulted` | bool | Retirado del catálogo (Vaulted) |
| `release_year` | int\|null | Año de lanzamiento |
| `franchise` | string\|null | Franquicia |
| `pop_type` | string\|null | Tipo (p. ej. `Diamond`) |
| `rarity_score` | int | Índice de rareza de la propia API |
| `image_valid` | bool | Si la URL de imagen es válida |

**Ojo:** muchos campos vienen a `null` en el catálogo real; los habitualmente poblados son `id`, `name`, `category`, `rarity_score` e `image_valid`.

## Mapeo Propuesto → Modelo Interno (Funko)

| Campo HobbyQuantHQ | Campo interno | Notas |
|--------------------|---------------|-------|
| `id` | `externalId` (+ `source: hobbyquanthq`) | Clave de la fuente |
| `name` | `name` | |
| `series` | `series` | |
| `number` | `number` | |
| `category` | `category` | |
| `exclusive` | `exclusive` | |
| `vaulted` | `vaulted` | |
| `release_year` | `releaseYear` | |
| `franchise` | `franchise` | |
| `pop_type` | `popType` | |
| `rarity_score` | `rarityScore` | Índice de la API, no una valoración de mercado |
| `image_url` | `imageUrl` | Ver decisión de imágenes |

El resto de campos son los comunes a toda colección del proyecto (propietario, estado, notas, fecha de adquisición, imagen propia).

## Limitaciones

- ❌ **Beta**: el contrato no tiene garantía de estabilidad ni de campos.
- ❌ Sin cabeceras de rate limit: no se puede saber cuánto queda de cuota.
- ❌ Muchos campos nulos (`series`, `number`, `release_year`, `franchise`).
- ❌ **Precios "en evolución"** → **no se documentan** (decisión del 2026-10-11).
- ⚠️ Imágenes alojadas en `images.hobbydb.com` (CDN de HobbyDB): el catálogo parece originado en HobbyDB. Decisión de imágenes **pendiente**.
- ⚠️ No hay endpoint de búsqueda por código de barras.

## Configuración Propuesta

| Propiedad | Valor |
|-----------|-------|
| `hobbyquanthq.api.base-url` | `https://hobbyquanthq.com` |
| `hobbyquanthq.api.key` | `${HOBBYQUANT_API_KEY}` |

## Referencias

- [HobbyQuantHQ API Docs](https://hobbyquanthq.com/api-docs)
- [HobbyQuantHQ](https://hobbyquanthq.com/)
