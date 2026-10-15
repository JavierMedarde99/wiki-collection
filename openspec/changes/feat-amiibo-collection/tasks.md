# Tasks — Colección de Amiibo

## Contexto

Change de **especificación**: las tareas describen lo que se aplica en el wiki tras aprobación y lo que se implementará después en los repos de código. **Nada de esto se ejecuta en esta iteración de SDD** (scope: solo spec). Estado global: sin empezar.

## 1. Aplicar la spec en el wiki (tras aprobación del change)

- [ ] Crear `specs/amiibos/spec.md` (dominio, puertos, API externa, servicios, controller, DTOs, excepciones, tests, criterios de aceptación) — siguiendo el template de `board-games`.
- [ ] Crear `specs/amiibos/api-contract.md` (endpoints, parámetros, DTOs, códigos de estado, errores).
- [ ] Añadir colección `AMIIBOS` a `data-model/schema.md` (campos + índice `externalId`).
- [ ] Actualizar `architecture/backend.md` (controllers 11 → 12, cachés 40 → 43, paquetes nuevos `amiibo`).
- [ ] Actualizar `architecture/frontend-*.md` (rutas/nav de Amiibo).
- [ ] Añadir Fase 31 a `requirements.md` (**sin marcar**).
- [ ] Actualizar `README.md` (tabla de fases).
- [ ] Actualizar `research/future/index.md` y el estado de `research/future/collectibles-funko-amiibo.md` (Amiibo: especificado).

## 2. Implementar backend (`backend-collection`, TDD — posterior)

- [ ] Domain: `Amiibo`, `AmiiboStatus`, `AmiiboType`, `AmiiboSearchResult`, `AmiiboSearchCriteria`
- [ ] Ports: `AmiiboUseCase`, `AmiiboSearchUseCase` (in); `AmiiboRepository`, `ExternalAmiiboCatalogClient` (out)
- [ ] Application: `AmiiboService`, `AmiiboSearchService`
- [ ] Infrastructure: `AmiiboApiClient` (trampas: case-sensitive, contenedor único/lista, 200 vacío), `AmiiboController` (+DTOs), `AmiiboEntity`, `SpringDataAmiiboRepository`, `AmiiboPersistenceAdapter`, `AmiiboEntityMapper`
- [ ] Config: `amiiboapi.base-url` en properties (sin secreto), cachés `amiiboSearch`/`amiiboList`/`amiiboDetail` en `CacheConfig`, `StringToAmiiboStatusConverter`
- [ ] Tests: `AmiiboServiceTest`, `AmiiboSearchServiceTest`, `AmiiboApiClientTest`, `AmiiboControllerTest`, `AmiiboPersistenceAdapterTest`, `AmiiboEntityMapperTest` — `mvn verify` verde (JaCoCo ≥ 0.80)
- [ ] PR contra `main` con TDD (rojo → verde)

## 3. Implementar frontend (`frontend-collection` — posterior)

- [ ] Tipos TS: `Amiibo`, `AmiiboStatus`, `AmiiboType`
- [ ] API client (`/api/v1/amiibos`, búsqueda por `name`)
- [ ] Rutas + páginas: listado, detalle, búsqueda y formulario
- [ ] Entrada en la navegación
- [ ] Tests de componente (vitest)

## 4. Cierre

- [ ] Verificar consistencia con `specs/amiibos/` y `data-model/schema.md`
- [ ] Archivar el change en `openspec/changes/archive/`