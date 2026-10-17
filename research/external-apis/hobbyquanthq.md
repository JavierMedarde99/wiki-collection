---
type: external-api
title: "HobbyQuantHQ — API de Funko Pop"
description: "Contrato de HobbyQuantHQ (beta): catálogo de Funko Pop"
tags: [api-externa, funko]
---

# HobbyQuantHQ — API de Funko Pop

**Investigación para:** catálogo de Funkos del backend (`FunkoApiClient`)
**Estado:** ✅ Activa — **API en beta**
**Verificado en vivo:** 2026-10-15 (con `x-api-key: dev-key-change-me`)
**Consumidor:** `FunkoApiClient` en `src/main/java/com/wikicollection/infrastructure/adapter/out/funko/` (rama `main`)

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://hobbyquanthq.com/api/v1` |
| **Auth** | Header `x-api-key` |
| **Rate limit** | No documentado ni expuesto en cabeceras; la API está en beta. Verificado el 2026-10-15: solo devuelve cabeceras de Vercel y de seguridad, ninguna de cuota |
| **Gratis** | Sí, durante la beta |
| **Registro** | No necesario en beta (clave de desarrollo `dev-key-change-me`) |
| **Formato** | JSON |
| **Catálogo** | 15.458 pops · 133 categorías · 303 franquicias · 3.776 pops con exclusividad (37 exclusividades distintas) |
| **Documentación** | https://hobbyquanthq.com/api-docs |
| **Estado** | Beta — el catálogo y los campos pueden evolucionar |

> El número de categorías (133) es el de entradas que devuelve `/categories`; el de franquicias (303), el de entradas de `/franchises`; y el de 3.776 exclusividades, la **suma de los `count`** de `/exclusives` (pops con alguna exclusividad), no el número de exclusividades distintas (37). La cifra de "3.805 variantes" que circula **no se pudo verificar** el 2026-10-15: no hay endpoint de variantes (`/variants` responde 404).

## Autenticación

```http
x-api-key: TU_API_KEY
```

- En beta funciona la clave pública `dev-key-change-me`.
- Sin cabecera: **422** con `{"detail":[{"type":"missing","loc":["header","x-api-key"],"msg":"Field required"}]}`.
- **No hardcodear la clave en el frontend** — el patrón del proyecto es clave en backend + variable de entorno.

## Endpoints Verificados

| Uso | Endpoint | Parámetros | ¿Lo consume el backend? | Notas |
|-----|----------|------------|-------------------------|-------|
| Listar pops | `GET /pops` | `page` (def. 1), `limit` (def. 20, máx **100**), `series` (coincidencia parcial), `category` (coincidencia exacta), `vaulted`, `exclusive`, `sort` (`name`\|`series`\|`number`\|`category`\|`release_year`, def. `name`), `order` (`asc`\|`desc`, def. `asc`) | No | 200 con el sobre `items`. `limit=101` → **422**; `sort` inválido → **422**. Filtros comprobados: `category=Comics` (43), `vaulted=true` (8), `exclusive=Walmart` (216), `series=DC Comics` (5) |
| Obtener un pop | `GET /pops/{id}` | — | Sí (`findById`) | 200 con el objeto suelto (sin sobre). Id inexistente → **404** `{"detail":"Pop not found."}` |
| Buscar | `GET /search?q={q}` | `q` (mínimo **2** caracteres), `limit` (def. 50, máx **100**) | Sí (`searchByName`, fija `limit=100`) | 200 con el sobre `results`. `q` de 1 carácter → **422**; `limit=101` → **422**. Busca por nombre, serie y número. **No pagina**: el parámetro `page` se ignora |
| Categorías | `GET /categories` | — | Sí (`listCategories`) | 200 con `[{"name","count"}]` (133 entradas) |
| Franquicias | `GET /franchises` | — (`limit` se ignora) | No | 200 con `[{"name","count"}]` (303 entradas). **No aparece en `/api-docs`** |
| Exclusividades | `GET /exclusives` | — (`limit` se ignora) | No | 200 con `[{"name","count"}]` (37 entradas). **No aparece en `/api-docs`** |
| Tendencias | `GET /trends/mock/{pop_id}` | — | No | 200 con `price_history`; está en `/api-docs`, pero queda **fuera de alcance** (no se documentan precios) |

La raíz `/api/v1/` responde **308** (redirección); usar los endpoints concretos. No existe `api.hobbyquanthq.com`.

> ⚠️ `/franchises` y `/exclusives` responden pero **no están en la documentación oficial**, así que su contrato puede cambiar sin aviso. El backend no los usa.

## Forma Real de las Respuestas

`/pops` devuelve un sobre paginado:

```json
{
  "items": [ { "…": "Pop" } ],
  "total": 15458,
  "page": 1,
  "limit": 2,
  "pages": 7729
}
```

`/search` devuelve un sobre distinto, **sin paginación**:

```json
{
  "results": [ { "…": "Pop" } ],
  "query": "Batman",
  "total": 2
}
```

`/categories`, `/franchises` y `/exclusives` devuelven un array plano sin sobre: `[{"name": "…", "count": 0}]`.

> ⚠️ **Corrige documentación de terceros:** algunas notas circulan `{"data": [...]}` como sobre de respuesta. **No existe**: `/pops` usa `items` y `/search` usa `results`.

> ⚠️ En `/search`, `total` es **el número de resultados devueltos** (recortado por `limit`), no el total de coincidencias del catálogo. Verificado: `q=Batman` da 50 sin `limit`, 10 con `limit=10` y 100 con `limit=100`. Por eso el backend pide `limit=100` y corta la página en el servicio.

## Campos de un Pop

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | int | Identificador externo (PK de la fuente) |
| `name` | string | Nombre del Pop |
| `series` | string\|null | Serie (p. ej. `DC Comics`) |
| `number` | string\|null | Número del Pop (p. ej. `"01"`) |
| `category` | string\|null | Categoría (p. ej. `Movies`, `Comics`) |
| `exclusive` | string\|null | Exclusividad (p. ej. `Walmart`) |
| `image_url` | string\|null | Imagen alojada en `images.hobbydb.com` |
| `vaulted` | bool | Retirado del catálogo (Vaulted) |
| `release_year` | int\|null | Año de lanzamiento |
| `franchise` | string\|null | Franquicia |
| `pop_type` | string\|null | Tipo (p. ej. `Diamond`) |
| `rarity_score` | int | Índice de rareza de la propia API |
| `image_valid` | bool | Si la URL de imagen es válida |

**Ojo:** muchos campos vienen a `null` en el catálogo real; los habitualmente poblados son `id`, `name`, `category`, `rarity_score` e `image_valid`.

**`image_valid` no lo lee el backend.** La API lo devuelve en cada pop, pero el `record FunkoApiItem` del cliente **no declara ese campo** y lo descarta con `@JsonIgnoreProperties(ignoreUnknown = true)`. El cliente solo lee: `id`, `name`, `series`, `number`, `category`, `exclusive`, `vaulted`, `release_year`, `franchise`, `pop_type`, `rarity_score` e `image_url`.

## Mapeo Real → Modelo Interno (Funko)

Traducido por `FunkoApiClient.toSearchResult` hacia `FunkoSearchResult` (y de ahí a la entidad `Funko`):

| Campo HobbyQuantHQ | Campo interno | Notas |
|--------------------|---------------|-------|
| `id` | `externalId` (+ `externalSource: hobbyquanthq`) | Entero convertido a String; clave natural de la colección |
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
| `image_valid` | — | **No se mapea**: el cliente no lo lee |

El resto de campos son los comunes a toda colección del proyecto (propietario, estado, notas, fecha de adquisición, imagen propia).

## Limitaciones

- ❌ **Beta**: el contrato no tiene garantía de estabilidad ni de campos.
- ❌ Sin cabeceras de rate limit: no se puede saber cuánto queda de cuota.
- ❌ Muchos campos nulos (`series`, `number`, `release_year`, `franchise`).
- ❌ `/search` no pagina y su `total` no es el total real de coincidencias, sino el de la página devuelta.
- ❌ **Precios "en evolución"** → **no se documentan** (decisión del 2026-10-11). El endpoint documentado `/trends/mock/{pop_id}` queda fuera de alcance por el mismo motivo.
- ❌ `image_valid` existe en la respuesta pero el backend lo ignora.
- ⚠️ `/franchises` y `/exclusives` no están en la documentación oficial (no consumidos por el backend).
- ⚠️ Imágenes alojadas en `images.hobbydb.com` (CDN de HobbyDB): el catálogo parece originado en HobbyDB. Decisión de imágenes **pendiente**.
- ⚠️ No hay endpoint de búsqueda por código de barras.
- ⚠️ No se pudo verificar la cifra de "3.805 variantes" (no hay endpoint de variantes).

## Configuración

Valores reales en `src/main/resources/application.properties`:

| Propiedad | Valor |
|-----------|-------|
| `hobbyquanthq.api.base-url` | `https://hobbyquanthq.com/api/v1` |
| `hobbyquanthq.api.key` | `${HOBBYQUANT_API_KEY:dev-key-change-me}` |

El cliente usa `RestClient`, reintenta 5xx/timeouts hasta 4 intentos (1 s entre ellos) y no reintenta 4xx. Cachea la búsqueda (`funkoSearch`, clave = consulta) y las categorías (`funkoCategories`, clave `'all'`); un 404 del catálogo se traduce a `Optional.empty()`.

## Referencias

- [HobbyQuantHQ API Docs](https://hobbyquanthq.com/api-docs)
- [HobbyQuantHQ](https://hobbyquanthq.com/)
