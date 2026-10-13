# Proposal — Colección de Amiibo

## Why

El producto gestiona colecciones de libros, juegos, juegos de mesa, cartas Magic, mazos y películas/series, pero no tiene ningún tipo de coleccionable físico. El usuario quiere añadir sus **Amiibo** (figuras y cartas con chip NFC de Nintendo) a la colección personal con el mismo patrón que el resto de capacidades: catálogo externo para buscar, añadir a la colección, listar/filtrar, editar y eliminar.

La API externa ya está investigada y verificada en vivo el 2026-10-13 (`research/external-apis/amiiboapi.md`): **AmiiboAPI** es pública, gratuita y sin auth ni API key, con un catálogo estable de 950 amiibo. Es la integración más barata del proyecto — no añade secretos ni configuración sensible — y encaja en la arquitectura actual (cliente HTTP + caché Caffeine + mapeo a dominio).

Este change crea la **especificación** de la colección de Amiibo siguiendo el flujo Spec-Driven (OpenSpec): primero el contrato, después la implementación en `backend-collection` y `frontend-collection`.

## What Changes

- **Nueva capability `amiibos`**: entidad `Amiibo`, enums `AmiiboStatus` (OWNED, WISHLIST) y `AmiiboType` (CARD, FIGURE, BAND, YARN, BLOCK), resultados de búsqueda, puertos, servicios, controller y cliente externo `AmiiboApiClient`.
- **Nueva colección Mongo `AMIIBOS`**: campos de catálogo (`externalId` = head+tail, `name`, `character`, `amiiboSeries`, `gameSeries`, `amiiboType`, `releaseDates`, `imageUrl`, `compatibleGames`) + campos comunes de colección (`status`, `notes`, `dateAdded`, `ownerId`, `userOwned`).
- **Endpoints `/api/v1/amiibos`**: CRUD + búsqueda externa en AmiiboAPI, con el patrón de autorización/visibilidad del resto (GET público, escritura con auth + ownership, filtro `owner` con `mine`/`other`/`all`).
- **Cachés nuevas**: `amiiboSearch`, `amiiboList`, `amiiboDetail`.
- **Imágenes por hotlink**: `imageUrl` apunta directamente a la URL externa (repo `8bitDream/AmiiboAPI` en GitHub raw). No se descargan ni se suben a Catbox.
- **Frontend** (al implementar): páginas de listado, detalle, búsqueda y formulario, con navegación nueva.

## Capabilities

| Capability | Impacto |
|------------|---------|
| `amiibos` (nueva) | `specs/amiibos/spec.md` + `specs/amiibos/api-contract.md` |

## Impact

- **Backend** (código, cuando se implemente): ~15 archivos nuevos (domain, ports, application, infrastructure, config) y 3 cachés nuevas (40 → 43). Controllers: 11 → 12.
- **Frontend** (código): rutas/páginas nuevas y API client.
- **Docs** (al aplicar este change): `specs/amiibos/spec.md`, `specs/amiibos/api-contract.md`, `data-model/schema.md`, `architecture/backend.md`, `architecture/frontend-*.md`, `requirements.md`, `README.md`, `research/future/index.md`.
- **Riesgos**: el catálogo de AmiiboAPI es comunitario (mantenido con donaciones); las imágenes viven en un repo GitHub de un tercero (riesgo de enlace muerto, asumido — ver `decisions.md` DA3). Sin rate limit documentado, no se puede medir cuota.

## Decisiones Tomadas

Resumen — el detalle está en `decisions.md`:

- Imágenes mediante **hotlink** de la URL externa (sin Catbox).
- Estados de colección: **OWNED, WISHLIST** (mismo patrón que board-games).
- Colección Mongo **separada** `AMIIBOS` (no una colección genérica de «collectibles»).
- **Sin precios** y **sin código de barras** (AmiiboAPI no los expone).
- Amiibo va en un change OpenSpec **separado** de Funko y **antes** que Funko.
- **Sin endpoints de catálogo de series** (`/amiiboseries/`, `/gameseries/`) por ahora: el formulario usa los datos del resultado de búsqueda.

## Próximos pasos

1. Revisar `design.md` y `tasks.md` de este change.
2. Revisión humana del change (PR).
3. Aplicar: crear `specs/amiibos/` y actualizar `data-model/schema.md`, `architecture/backend.md`, `architecture/frontend-*.md`, `requirements.md`, `README.md` y `research/future/index.md`.
4. Implementar en backend + frontend (TDD) en una fase distinta — propuesta: Fase 31 (sin asignar todavía).