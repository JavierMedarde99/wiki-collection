# Tasks — Colección de Funko Pop

## Contexto

Change de **especificación**: las tareas describen lo que se aplica en el wiki tras aprobación y lo que se implementará después en los repos de código. **Nada de esto se ejecuta en esta iteración de SDD** (scope: solo spec). Estado global: sin empezar.

## 1. Aplicar la spec en el wiki (tras aprobación del change)

- [ ] Crear `specs/funkos/spec.md` (dominio, puertos, API externa, servicios, controller, DTOs, excepciones, tests, criterios de aceptación) — siguiendo el template de `board-games`/`amiibos`.
- [ ] Crear `specs/funkos/api-contract.md` (endpoints, parámetros, DTOs, códigos de estado, errores).
- [ ] Añadir colección `FUNKOS` a `data-model/schema.md` (campos + índices `externalId`/`ownerId`).
- [ ] Actualizar `architecture/backend.md` (controllers 12 → 13, cachés 36 → 40, paquetes nuevos `funko`).
- [ ] Actualizar `architecture/frontend-*.md` (rutas/nav de Funko).
- [ ] Añadir Fase 30 a `requirements.md` (**sin marcar**).
- [ ] Actualizar `README.md` (tabla de fases).
- [ ] Actualizar `research/future/index.md` y el estado de `research/future/collectibles-funko-amiibo.md` (Funko: especificado).

## 2. Implementar backend (`backend-collection`, TDD — posterior)

- [ ] Domain: `Funko`, `FunkoStatus`, `FunkoSearchResult`, `FunkoCategory`, `FunkoSearchCriteria`
- [ ] Ports: `FunkoUseCase`, `FunkoSearchUseCase` (in); `FunkoRepository`, `ExternalFunkoCatalogClient` (out)
- [ ] Application: `FunkoService`, `FunkoSearchService`
- [ ] Infrastructure: `FunkoApiClient` (trampas: `x-api-key`, sobres `results`/`items`, `q` mín. 2, 404→vacío, 5xx con reintentos), `FunkoController` (+DTOs), `FunkoEntity`, `SpringDataFunkoRepository`, `FunkoPersistenceAdapter`, `FunkoEntityMapper`
- [ ] Config: `hobbyquanthq.api.base-url`/`hobbyquanthq.api.key` en properties, cachés `funkoSearch`/`funkoList`/`funkoDetail`/`funkoCategories` en `CacheConfig`, `StringToFunkoStatusConverter`, `CollectionType.FUNKOS`, `MongoIndexMigration`
- [ ] Tests: `FunkoServiceTest`, `FunkoSearchServiceTest`, `FunkoApiClientTest`, `FunkoControllerTest`, `FunkoPersistenceAdapterTest`, `FunkoEntityMapperTest` — `mvn verify` verde (JaCoCo ≥ 0.80)
- [ ] PR contra `main` con TDD (rojo → verde)

## 3. Implementar frontend (`frontend-collection` — posterior)

- [ ] Tipos TS: `Funko`, `FunkoStatus`, `FunkoCategory`
- [ ] API client (`/api/v1/funkos`, búsqueda por `name`, categorías)
- [ ] Rutas + páginas: listado, detalle, búsqueda y formulario
- [ ] Entrada en la navegación
- [ ] Tests de componente (vitest)

## 4. Cierre

- [ ] Verificar consistencia con `specs/funkos/` y `data-model/schema.md`
- [ ] Archivar el change en `openspec/changes/archive/`
