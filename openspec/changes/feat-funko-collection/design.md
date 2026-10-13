# Design — Colección de Funko Pop

## Contexto

La capability sigue el patrón consolidado de las colecciones existentes (books, games, board-games, magic-cards, movie-shows, amiibos): cliente HTTP contra un catálogo externo con caché Caffeine, mapeo a entidad de dominio, CRUD con chequeo de ownership y filtro `owner` (`mine`/`other`/`all`), y paginación Spring Data.

**Referencia de patrón:** `specs/amiibos/` es la capability más parecida (coleccionable físico, dos estados, búsqueda externa pública). Funko se modela igual, con dos diferencias: el catálogo externo (HobbyQuantHQ) **requiere API key** y **sí tiene paginación en `/pops`**, pero la búsqueda por nombre (`/search`) **no pagina** y está limitada a 100 resultados.

**Fuente del contrato externo:** `research/external-apis/hobbyquanthq.md` (verificada en vivo el 2026-10-11 y **re-verificada el 2026-10-13**).

---

## Modelo de dominio

### Entidad `Funko` (colección Mongo `FUNKOS`)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador MongoDB |
| externalId | String | ✅ | Identificador único de catálogo (el `id` entero de HobbyQuantHQ, guardado como String) |
| name | String | ✅ | Nombre del Pop (p. ej. `Batman`) |
| series | String | ❌ | Serie (p. ej. `DC Comics`); a menudo `null` |
| number | String | ❌ | Número del Pop (p. ej. `"01"`); a menudo `null` |
| category | String | ❌ | Categoría (p. ej. `Comics`, `Movies`, `Animation`) |
| exclusive | String | ❌ | Exclusividad (p. ej. `Walmart`) |
| vaulted | Boolean | ❌ | Retirado del catálogo (Vaulted) |
| releaseYear | Integer | ❌ | Año de lanzamiento; a menudo `null` |
| franchise | String | ❌ | Franquicia; a menudo `null` |
| popType | String | ❌ | Tipo (p. ej. `Diamond`) |
| rarityScore | Integer | ❌ | Índice de rareza de la propia API — **no** es una valoración de mercado |
| imageUrl | String | ❌ | URL de imagen (**hotlink**, `images.hobbydb.com`) |
| status | Enum (FunkoStatus) | ✅ | OWNED, WISHLIST |
| notes | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió a la colección |
| ownerId | String | ✅ | ID del usuario propietario |
| userOwned | Object | Auto | owner + visibilidad: ownerId, ownerName, username |

> El campo `externalId` es la clave natural del catálogo (`id` de HobbyQuantHQ). Un pop es único por catálogo; la unicidad de colección se aplica igual que `bggId` en board-games. El campo `image_valid` de la API **no se modela**: se infiere de `imageUrl != null`.

### Enums

**FunkoStatus:** `OWNED`, `WISHLIST`

### Value objects / resultados

- **`FunkoSearchResult`** — subconjunto del catálogo devuelto por la búsqueda externa: `externalId`, `name`, `series`, `number`, `category`, `exclusive`, `vaulted`, `releaseYear`, `franchise`, `popType`, `rarityScore`, `imageUrl`, `externalSource` (`"hobbyquanthq"`).
- **`FunkoCategory`** — `name`, `count`.
- **`FunkoSearchCriteria`** — `name`, `status`, `ownerId`, `excludeOwnerIds` (mismo patrón que el resto).

### Reglas de Negocio

- **`externalId` único** — no se pueden duplicar pops del catálogo en la colección.
- **`status` tiene 2 valores** — OWNED y WISHLIST (patrón board-games/Amiibo).
- **Mínimo para guardar** — se puede añadir a la colección cualquier resultado de la búsqueda; solo `name` y `externalId` vienen siempre poblados del catálogo.
- **Sin precios ni barcode** — fuera de alcance (HobbyQuantHQ no expone barcode; sus precios están «en evolución»).
- **`rarityScore` es un índice de la fuente** — nunca se presenta como precio ni como valor de mercado.

---

## Puertos (Interfaces de Dominio)

### FunkoUseCase (in)

```java
public interface FunkoUseCase {
    Funko save(Funko funko, String ownerId);
    Funko update(String id, Funko updates, String ownerId);
    void delete(String id, String ownerId);
    Funko findById(String id);
    Page<Funko> search(FunkoSearchCriteria criteria, Pageable pageable, String owner, String viewerId);
}
```

### FunkoSearchUseCase (in)

```java
public interface FunkoSearchUseCase {
    Page<FunkoSearchResult> search(String name, Pageable pageable);
    List<FunkoCategory> listCategories();
}
```

### Ports out

- **`FunkoRepository`** — persistencia: save, update, delete, findById, search (filtros + paginación).
- **`ExternalFunkoCatalogClient`** — `searchByName(name)`, `findById(externalId)`, `listCategories()` contra HobbyQuantHQ.

---

## API Externa: HobbyQuantHQ

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://hobbyquanthq.com/api/v1` |
| **Auth** | Header `x-api-key` |
| **Rate limit** | No documentado ni expuesto en cabeceras (API en beta) |
| **Formato** | JSON |
| **Catálogo** | 15.458 pops · 3.776 exclusividades · 3.805 variantes · 303 franquicias · 133 categorías |
| **Imágenes** | URLs en `images.hobbydb.com` (CDN de HobbyDB) |
| **Estado** | Beta — contrato y campos sin garantía de estabilidad |

### Endpoints usados

| Uso | Endpoint | Notas |
|-----|----------|-------|
| Búsqueda externa | `GET /search?q={query}&limit=100` | `q` **mínimo 2 caracteres**; **no pagina**; `limit` por defecto 50, máximo 100 |
| Detalle por id externo | `GET /pops/{externalId}` | 200 con el objeto; **404** si no existe |
| Categorías | `GET /categories` | `[{"name","count"}]` |

> La raíz `/api/v1` responde **308** (redirección); usar endpoints concretos. No existe `api.hobbyquanthq.com`. El endpoint `GET /pops` (paginado, `limit` máx 100) **existe pero no lo usa este cliente** (ver decisiones: sin navegación de catálogo completo).

### Trampas verificadas del contrato (obligatorias para el cliente)

1. **Auth por header `x-api-key`.** Sin cabecera → **422** con `{"detail":[{"type":"missing","loc":["header","x-api-key"]}]}`. En beta funciona la clave pública `dev-key-change-me`. El cliente envía la cabecera siempre.
2. **Sobre de respuesta: `items`/`results`, nunca `data`.** `/pops` devuelve `{items, total, page, limit, pages}` y `/search` devuelve `{results, query, total}`. Documentación de terceros que muestre `{"data": […]}` es incorrecta.
3. **`/search` no pagina.** El parámetro `page` se **ignora** en silencio (devuelve siempre el mismo primer bloque). Para paginar en nuestra API solo se puede **cortar en local** los resultados que devuelve `/search`.
4. **`total` de `/search` no es el total real.** Es el número de resultados devueltos, ya recortado por `limit` (sin `limit` → 50; con `limit=100` → 100 aunque haya más coincidencias). No sirve como `totalElements` global.
5. **`q` mínimo 2 caracteres.** Con 1 carácter → **422** `string_too_short` (`min_length: 2`). La validación debe hacerse en nuestro servicio (→ 400) para no propagar un 422 de la fuente.
6. **`limit` máximo 100.** `limit=101` → **422**. El cliente pide siempre 100 (el máximo) para maximizar resultados antes del corte local.
7. **404 es «no existe», no un fallo.** `/pops/{id}` desconocido → **404** `{"detail":"Pop not found."}`. El cliente lo traduce a `Optional.empty()` y **no** reintenta en 4xx.
8. **Sin resultados de búsqueda → HTTP 200 con `{"results": [], "total": 0}`.** Lista vacía es un resultado válido, no un error.
9. **Muchos campos vienen a `null`** (`series`, `number`, `release_year`, `franchise`): el modelo y los DTOs los aceptan opcionales.
10. **Parámetros desconocidos / filtros libres**: el cliente construye los parámetros exactos que necesita; no expone filtros libres al usuario.

### Forma de las respuestas

`GET /search?q=Batman&limit=2`:

```json
{ "results": [ { "...": "Pop" } ], "query": "Batman", "total": 2 }
```

`GET /pops/{id}`:

```json
{ "id": 44, "name": "Batman", "series": "DC Comics", "number": "01", "category": "Comics", "exclusive": null, "image_url": null, "vaulted": true, "release_year": 2012, "franchise": null, "pop_type": null, "rarity_score": 30, "image_valid": true }
```

`GET /categories`:

```json
[ { "name": "Television", "count": 1385 }, { "name": "Movies", "count": 1238 } ]
```

### Mapeo HobbyQuantHQ → FunkoSearchResult

| Campo HobbyQuantHQ | Campo interno | Notas |
|--------------------|---------------|-------|
| `id` (int) | `externalId` (String) | Clave de la fuente |
| `name` | `name` | |
| `series` | `series` | Puede ser `null` |
| `number` | `number` | String (p. ej. `"01"`) |
| `category` | `category` | |
| `exclusive` | `exclusive` | |
| `vaulted` | `vaulted` | |
| `release_year` | `releaseYear` | |
| `franchise` | `franchise` | |
| `pop_type` | `popType` | |
| `rarity_score` | `rarityScore` | Índice de la API, no valor de mercado |
| `image_url` | `imageUrl` | Hotlink |
| — | `externalSource` | Constante `"hobbyquanthq"` |
| `image_valid` | *(no se guarda)* | Se infiere de `imageUrl != null` |

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `FunkoService` | CRUD de funkos + validaciones (ownership, unicidad externalId) + listado con owner scope |
| `FunkoSearchService` | Búsqueda externa HobbyQuantHQ + corte local + categorías |

---

## Infraestructura

### Cliente: `FunkoApiClient`

- `RestClient` con base URL `hobbyquanthq.api.base-url` y cabecera `x-api-key: ${hobbyquanthq.api.key}`.
- Métodos: `searchByName(name)` (pide `limit=100`), `findById(externalId)` (404 → `Optional.empty()`), `listCategories()`.
- Deserialización con records propios (`@JsonIgnoreProperties(ignoreUnknown = true)` en **todos** y **un solo constructor** — lección de Scryfall) y nombres de campo Jackson (`image_url` → `imageUrl`, `release_year`, `pop_type`, `rarity_score`).
- **Reintentos:** 4 intentos totales (3 reintentos a 1 s; intervalo inyectable, 0 ms en tests), **solo en 5xx/timeout**. Al agotar, relanza (`RestClientResponseException` → 502, `ResourceAccessException` → 503). Un 4xx (401/403/422/404) **no** se reintenta.
- **`@Autowired` obligatorio en el constructor público** (dos constructores: el público de un argumento y el package-private `(RestClient, long)` para tests) — sin la anotación el contexto falla con `No default constructor found` (patrón `GoogleBooksClient`/`AmiiboApiClient`).

### Persistencia

- `FunkoEntity` (Mongo `FUNKOS`), `SpringDataFunkoRepository`, `FunkoPersistenceAdapter` (implementa `FunkoRepository`), `FunkoEntityMapper`.
- Índice por `externalId` (patrón `bggId`) y por `ownerId` (filtro de visibilidad).

### Controlador y DTOs

- `FunkoController` bajo `/api/v1/funkos` con `FunkoRequest`, `FunkoResponse`, `FunkoSearchResponse` y `FunkoCategoryResponse`.
- `GET /api/v1/**` público; escritura con auth + ownership (patrón global); `ResponseVisibility` recorta `notes` cuando el visitante no puede ver campos privados.

---

## Configuración

| Propiedad | Valor |
|-----------|-------|
| `hobbyquanthq.api.base-url` | `https://hobbyquanthq.com/api/v1` |
| `hobbyquanthq.api.key` | `${HOBBYQUANT_API_KEY:dev-key-change-me}` |

La clave beta `dev-key-change-me` es **pública** (no es un secreto real); en producción se inyecta `HOBBYQUANT_API_KEY`. Nunca se hardcodea una clave real en `application.properties` ni se expone al frontend.

**Cachés** (Caffeine, en `CacheConfig`): `funkoSearch`, `funkoList`, `funkoDetail`, `funkoCategories` (36 → 40). `funkoCategories` es de cambio lento: TTL larga. Misma política de grupos/TTL que el resto; las claves de list/detail incluyen el id del viewer.

**Converter:** `StringToFunkoStatusConverter` para `?status=OWNED`/`WISHLIST`.

**Tipos de colección:** `CollectionType.FUNKOS("funkos")`; `MongoIndexMigration` incluye `funkos` en las colecciones con índice de `ownerId`.

---

## API REST

**Base:** `/api/v1/funkos`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/funkos` | Listar funkos (paginado; filtros name/status/owner) |
| GET | `/api/v1/funkos/{id}` | Obtener funko por id interno |
| POST | `/api/v1/funkos` | Añadir funko a la colección (auth) |
| PUT | `/api/v1/funkos/{id}` | Actualizar (auth + ownership) |
| DELETE | `/api/v1/funkos/{id}` | Eliminar (204, auth + ownership) |
| GET | `/api/v1/funkos/search?name={query}` | Buscar en HobbyQuantHQ (mínimo 2 caracteres) |
| GET | `/api/v1/funkos/categories` | Listar categorías del catálogo externo |

### Parámetros de listado

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `page` | Integer | Página (0-based) |
| `size` | Integer | Tamaño de página |
| `sort` | String | Ordenación `field,asc/desc` (default `name,asc`) |
| `name` | String | Búsqueda local por nombre (LIKE) |
| `status` | Enum | Filtrar por estado: OWNED, WISHLIST |
| `owner` | String | `mine` (default) / `other` / `all` |

### DTOs

**FunkoRequest (POST/PUT)**

```json
{
  "externalId": "string (obligatorio)",
  "name": "string (obligatorio)",
  "series": "string",
  "number": "string",
  "category": "string",
  "exclusive": "string",
  "vaulted": true,
  "releaseYear": 2012,
  "franchise": "string",
  "popType": "string",
  "rarityScore": 30,
  "imageUrl": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date"
}
```

**FunkoResponse (GET)**

```json
{
  "id": "string",
  "externalId": "string",
  "name": "string",
  "series": "string",
  "number": "string",
  "category": "string",
  "exclusive": "string",
  "vaulted": true,
  "releaseYear": 2012,
  "franchise": "string",
  "popType": "string",
  "rarityScore": 30,
  "imageUrl": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date",
  "userOwned": { "ownerId": "string", "ownerName": "string", "username": "string" }
}
```

**FunkoSearchResponse (resultado externo)**

```json
{
  "externalId": "string",
  "name": "string",
  "series": "string",
  "number": "string",
  "category": "string",
  "exclusive": "string",
  "vaulted": true,
  "releaseYear": 2012,
  "franchise": "string",
  "popType": "string",
  "rarityScore": 30,
  "imageUrl": "string",
  "externalSource": "hobbyquanthq"
}
```

**FunkoCategoryResponse (categorías)**

```json
{ "name": "Television", "count": 1385 }
```

---

## Frontend (al implementar)

- **Rutas**: `/funkos` (listado), `/funkos/:id` (detalle), búsqueda y formulario (modal o página, patrón de las colecciones existentes).
- **Nav**: entrada nueva en la navegación principal.
- **API client + tipos TS**: `Funko`, `FunkoStatus` (OWNED | WISHLIST), `FunkoCategory`.
- **Imágenes**: render directo de `imageUrl` (hotlink); placeholder si es `null`.

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `FunkoNotFoundException` | 404 | Funko no encontrado |
| `FunkoConflictException` | 409 | `externalId` ya existe en la colección |
| `IllegalArgumentException` | 400 | Filtros inválidos (p. ej. búsqueda con menos de 2 caracteres) |
| Error de catálogo externo (5xx/timeout) | 502/503 | Patrón del proyecto para APIs externas |

---

## Tests (al implementar)

| Test | Tipo | Descripción |
|------|------|-------------|
| `FunkoServiceTest` | Unitario | CRUD + validaciones (ownership, unicidad externalId) |
| `FunkoSearchServiceTest` | Unitario | Búsqueda externa + corte local + categorías |
| `FunkoApiClientTest` | Unitario | Cliente con mock — `x-api-key`, sobres `results`/`items`, 404→vacío, 422 sin cabecera, 5xx con reintentos |
| `FunkoControllerTest` | Integración | Endpoints + CORS + owner filter + recorte de `notes` |
| `FunkoPersistenceAdapterTest` | Integración | Persistencia Mongo (mock) |
| `FunkoEntityMapperTest` | Unitario | Mapeo dominio ↔ entidad |

---

## Criterios de Aceptación

- [ ] `GET /api/v1/funkos/search?name=batman` devuelve resultados de HobbyQuantHQ (con `limit=100`)
- [ ] `name` de menos de 2 caracteres (o vacío) → **400**, sin llamar a la fuente
- [ ] Los resultados incluyen `externalId`, `name`, `series`, `number`, `category`, `exclusive`, `vaulted`, `releaseYear`, `franchise`, `popType`, `rarityScore`, `imageUrl` y `externalSource`
- [ ] `GET /api/v1/funkos/categories` devuelve las categorías de HobbyQuantHQ (`name`, `count`)
- [ ] `GET /api/v1/funkos` devuelve lista paginada vacía al inicio; filtra por `name`, `status` y `owner`
- [ ] `POST /api/v1/funkos` añade un funko (auth requerido; `externalId` no duplicable → 409)
- [ ] `PUT`/`DELETE /api/v1/funkos/{id}` con auth + ownership verificados
- [ ] El cliente traduce `404 Pop not found` a «no existe» (sin reintento) y tolera `results: []` (200) como vacío válido
- [ ] Error de red del catálogo (5xx/timeout) → 502/503 tras 4 intentos, no una lista vacía
- [ ] `mvn verify` verde (JaCoCo ≥ 0.80)

---

## Impacto en Documentación (tras aprobación)

- `specs/funkos/spec.md` — nuevo (esta planificación convertida en spec).
- `specs/funkos/api-contract.md` — nuevo.
- `data-model/schema.md` — colección `FUNKOS` + índices.
- `architecture/backend.md` — controllers 12 → 13, cachés 36 → 40, paquetes nuevos (`funko`).
- `architecture/frontend-*.md` — rutas/nav.
- `requirements.md` — Fase 30 (propuesta, **sin marcar**).
- `README.md` — tabla de fases.
- `research/future/index.md` + `research/future/collectibles-funko-amiibo.md` — estado: Funko especificado.
