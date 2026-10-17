# Proposal — Colección de Funko Pop

## Why

El producto ya gestiona libros, juegos, juegos de mesa, cartas Magic, mazos, películas/series y (especificado) Amiibo, pero no **Funko Pop**. El usuario quiere añadir sus **Pop!** (figuras vinílicas) a la colección personal con el mismo patrón que el resto de capacidades: catálogo externo para buscar, añadir a la colección, listar/filtrar, editar y eliminar.

La API externa ya está investigada y **verificada en vivo el 2026-10-11 y de nuevo el 2026-10-13** (`research/external-apis/hobbyquanthq.md`): **HobbyQuantHQ** es una API REST con header `x-api-key` (en beta la clave pública `dev-key-change-me` basta), con un catálogo de 15.458 pops, 303 franquicias y 133 categorías. Encaja en la arquitectura actual (cliente HTTP + caché Caffeine + mapeo a dominio), como RAWG o Google Books.

Este change crea la **especificación** de la colección de Funko siguiendo el flujo Spec-Driven (OpenSpec): primero el contrato, después la implementación en `backend-collection` y `frontend-collection`.

## What Changes

- **Nueva capability `funkos`**: entidad `Funko`, enum `FunkoStatus` (OWNED, WISHLIST), `FunkoSearchResult`, `FunkoCategory`, puertos, servicios, controller y cliente externo `FunkoApiClient`.
- **Nueva colección Mongo `FUNKOS`**: campos de catálogo (`externalId`, `name`, `series`, `number`, `category`, `exclusive`, `vaulted`, `releaseYear`, `franchise`, `popType`, `rarityScore`, `imageUrl`) + campos comunes de colección (`status`, `notes`, `dateAdded`, `ownerId`, `userOwned`).
- **Endpoints `/api/v1/funkos`**: CRUD + búsqueda externa en HobbyQuantHQ + listado de categorías, con el patrón de autorización/visibilidad del resto (GET público, escritura con auth + ownership, filtro `owner` con `mine`/`other`/`all`).
- **Cachés nuevas**: `funkoSearch`, `funkoList`, `funkoDetail`, `funkoCategories` (36 → 40).
- **Imágenes por hotlink**: `imageUrl` apunta directamente a la URL externa (`images.hobbydb.com`). No se descargan ni se suben a Catbox.
- **Frontend** (al implementar): páginas de listado, detalle, búsqueda y formulario, con navegación nueva.

## Capabilities

| Capability | Impacto |
|------------|---------|
| `funkos` (nueva) | `specs/funkos/spec.md` + `specs/funkos/api-contract.md` |

## Impact

- **Backend** (código, cuando se implemente): ~15 archivos nuevos (domain, ports, application, infrastructure, config) y 4 cachés nuevas (36 → 40). Controllers: 12 → 13.
- **Frontend** (código): rutas/páginas nuevas y API client.
- **Docs** (al aplicar este change): `specs/funkos/spec.md`, `specs/funkos/api-contract.md`, `data-model/schema.md`, `architecture/backend.md`, `architecture/frontend-*.md`, `requirements.md`, `README.md`, `research/future/index.md` y el estado de `research/future/collectibles-funko-amiibo.md`.
- **Riesgos**: la API está **en beta** — el contrato y los campos pueden cambiar sin aviso, y no hay garantía de estabilidad; **sin cabeceras de rate limit** no se puede medir la cuota; **muchos campos vienen a `null`** en el catálogo real; las imágenes viven en `images.hobbydb.com` (CDN de HobbyDB, un tercero).

## Decisiones Tomadas

Resumen — el detalle está en `decisions.md`:

- Imágenes mediante **hotlink** de la URL externa (sin Catbox).
- Estados de colección: **OWNED, WISHLIST** (mismo patrón que board-games y Amiibo).
- Colección Mongo **separada** `FUNKOS` (no una colección genérica de «collectibles»).
- **Sin precios** y **sin código de barras** (HobbyQuantHQ no expone barcode y sus precios están «en evolución»).
- Funko va en un change OpenSpec **separado** de Amiibo y **después** de Amiibo.
- Se expone `GET /funkos/categories` (proxy de `/categories`); **no** se expone navegación del catálogo completo (`/pops`).
- `externalId` se guarda como **String** aunque la fuente use `int`.

## Próximos pasos

1. Revisar `design.md` y `tasks.md` de este change.
2. Revisión humana del change (PR).
3. Aplicar: crear `specs/funkos/` y actualizar `data-model/schema.md`, `architecture/backend.md`, `architecture/frontend-*.md`, `requirements.md`, `README.md`, `research/future/index.md` y `research/future/collectibles-funko-amiibo.md`.
4. Implementar en backend + frontend (TDD) en una fase distinta — propuesta: Fase 30 (sin asignar todavía).
