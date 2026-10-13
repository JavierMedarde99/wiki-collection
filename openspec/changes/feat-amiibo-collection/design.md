# Design — Colección de Amiibo

## Contexto

La capability sigue el patrón consolidado de las colecciones existentes (books, games, board-games, magic-cards, movie-shows): cliente HTTP contra un catálogo externo con caché Caffeine, mapeo a entidad de dominio, CRUD con chequeo de ownership y filtro `owner` (`mine`/`other`/`all`), y paginación Spring Data.

**Referencia de patrón:** `specs/board-games/` es la capability más parecida: dos estados (OWNED, WISHLIST) + búsqueda externa pública. Amiibo se modela igual, con la salvedad de que el catálogo externo (AmiiboAPI) **no pagina** ni requiere auth.

**Fuente del contrato externo:** `research/external-apis/amiiboapi.md` (verificada en vivo el 2026-10-13).

---

## Modelo de dominio

### Entidad `Amiibo` (colección Mongo `AMIIBOS`)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador MongoDB |
| externalId | String | ✅ | Identificador único de catálogo: `head` + `tail` de AmiiboAPI (16 hex) |
| name | String | ✅ | Nombre del amiibo |
| character | String | ❌ | Personaje |
| amiiboSeries | String | ❌ | Serie del amiibo (p. ej. `Super Smash Bros.`) |
| gameSeries | String | ❌ | Serie de juego (p. ej. `Super Mario`) |
| amiiboType | Enum (AmiiboType) | ❌ | CARD, FIGURE, BAND, YARN, BLOCK |
| releaseDates | Object | ❌ | Fechas de lanzamiento por región `{au, eu, jp, na}` (LocalDate, opcionales) |
| imageUrl | String | ❌ | URL de imagen (**hotlink**, PNG de `raw.githubusercontent.com`) |
| compatibleGames | List<String> | ❌ | Juegos compatibles (nombres aplanados), de `showusage` — opcional |
| status | Enum (AmiiboStatus) | ✅ | OWNED, WISHLIST |
| notes | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió a la colección |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |
| userOwned | Object | Auto | owner + visibilidad: ownerId, ownerName, username |

> El campo `externalId` es la clave natural del catálogo (head+tail). Un amiibo es único por catálogo; la unicidad de colección se aplica igual que `bggId` en board-games.

### Enums

**AmiiboStatus:** `OWNED`, `WISHLIST`

**AmiiboType:** `CARD`, `FIGURE`, `BAND`, `YARN`, `BLOCK` — mapeado directo de los tipos de AmiiboAPI (`Card`, `Figure`, `Band`, `Yarn`, `Block`). `FIGURE` es el tipo «coleccionable» principal (258 en el catálogo), pero el resto son amiibo reales y se permiten.

### Reglas de Negocio

- **`externalId` único** — no se pueden duplicar amiibo del catálogo en la colección.
- **`status` tiene 2 valores** — OWNED y WISHLIST (patrón board-games, AD-012).
- **Mínimo para guardar** — se puede añadir a la colección cualquier resultado de la búsqueda; solo `name` y `externalId` vienen siempre poblados del catálogo.
- **Sin precios ni barcode** — fuera de alcance (decisión 2026-10-11, ratificada).

---

## Puertos (Interfaces de Dominio)

### AmiiboUseCase (in)

```java
public interface AmiiboUseCase {
    Amiibo save(Amiibo amiibo, String ownerId);
    Amiibo update(String id, Amiibo updates, String ownerId);
    void delete(String id, String ownerId);
    Amiibo findById(String id);
    Page<Amiibo> search(AmiiboSearchCriteria criteria, Pageable pageable);
}
```

### AmiiboSearchUseCase (in)

```java
public interface AmiiboSearchUseCase {
    List<AmiiboSearchResult> searchExternal(String query);
    AmiiboSearchResult findById(String externalId);
}
```

### Ports out

- **`AmiiboRepository`** — persistencia: save, update, delete, findById, search (filtros + paginación).
- **`ExternalAmiiboCatalogClient`** — búsqueda contra AmiiboAPI.

---

## API Externa: AmiiboAPI

- **Base URL:** `https://amiiboapi.org/api`
- **Auth:** ninguna (sin API key)
- **Rate limit:** no documentado ni expuesto en cabeceras
- **Formato:** JSON
- **CORS:** `Access-Control-Allow-Origin: *`
- **Catálogo:** 950 amiibo · 32 series amiibo · 48 series de juego · 660 personajes
- **Imágenes:** PNG (`image`) y WebP (`imgwebp`), en `raw.githubusercontent.com` (repo `8bitDream/AmiiboAPI`)

### Endpoints usados

| Uso | Endpoint | Notas |
|-----|----------|-------|
| Búsqueda externa | `GET /amiibo/?name={query}` | Coincidencia **parcial**; añadir `&showusage` para juegos compatibles |
| Detalle por id externo | `GET /amiibo/?id={externalId}` | ⚠️ devuelve **un objeto**, no una lista |

> La raíz `/api/` responde 404; solo funcionan los endpoints concretos (`/api/amiibo/`).

### Trampas verificadas del contrato (obligatorias para el cliente)

1. **Los nombres de parámetro distinguen mayúsculas/minúsculas**: `name`, `character`, `amiiboSeries`, `type`, `head`, `id` funcionan; **`gameseries` solo en minúscula** (camelCase `gameSeries` se ignora en silencio → devuelve el catálogo entero). El cliente construye los parámetros exactos; la búsqueda por nombre no expone filtros libres al usuario.
2. **Parámetros desconocidos se ignoran en silencio**: un typo devuelve 950 resultados, no un error. No hay filtro por nombre que valide parámetros — es el cliente quien los controla.
3. **`?id=` rompe el patrón de forma**: `GET /amiibo/` devuelve `{"amiibo": [...]}`, pero `?id=` devuelve `{"amiibo": {…}}` (objeto único). El mapper debe aceptar ambos contenedores — o el cliente usa un DTO distinto para el detalle.
4. **Sin resultados → HTTP 200 con `{"amiibo": []}`**, no 404. Lista vacía es un resultado válido.
5. **Sin paginación en el catálogo**: AmiiboAPI devuelve todos los que coinciden. La paginación es competencia del repositorio local (Spring Data), nunca del cliente.

### Forma de las respuestas

```json
{ "amiibo": [ { "...": "amiibo" } ] }
```

Con `?id=`:

```json
{ "amiibo": { "...": "amiibo" } }
```

### Mapeo AmiiboAPI → AmiiboSearchResult

| Campo AmiiboAPI | Campo interno | Notas |
|-----------------|---------------|-------|
| `head` + `tail` | `externalId` | Identificador único |
| `name` | `name` | |
| `character` | `character` | |
| `amiiboSeries` | `amiiboSeries` | |
| `gameSeries` | `gameSeries` | |
| `type` | `amiiboType` | `Figure` → `FIGURE`, etc. |
| `release` | `releaseDates` | `{au, eu, jp, na}` |
| `image` | `imageUrl` | PNG (hotlink) |
| `games3DS`/`gamesWiiU`/`gamesSwitch`/`gamesSwitch2` | `compatibleGames` | Con `showusage`; nombres aplanados, opcional |

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `AmiiboService` | CRUD de amiibo + validaciones (ownership, unicidad externalId) |
| `AmiiboSearchService` | Búsqueda externa AmiiboAPI + mapeo + caché |

---

## Infraestructura

### Cliente: `AmiiboApiClient`

- `RestClient` con base URL `amiiboapi.base-url`.
- Métodos: `searchByName(query)` (con `showusage`) y `findById(externalId)`.
- Maneja el doble contenedor de respuesta (lista vs objeto único).
- Sin reintentos de autenticación (no hay auth); en 5xx/timeout lanza el error al servicio (el patrón del proyecto reintenta en `GoogleBooksClient`/Scryfall correspondientemente — AmiiboAPI sin rate limit y con catálogo estable se trata como RAWG/BGG: **fallo de red = error**, no lista vacía).
- La coincidencia por nombre de AmiiboAPI es parcial (`?name=ario` → 41 resultados), así que la búsqueda no requiere `intitle:`.

### Persistencia

- `AmiiboEntity` (Mongo `AMIIBOS`), `SpringDataAmiiboRepository`, `AmiiboPersistenceAdapter` (implementa `AmiiboRepository`), `AmiiboEntityMapper`.
- Índice único/consulta por `externalId` (patrón `bggId`).

### Controlador y DTOs

- `AmiiboController` bajo `/api/v1/amiibos` con `AmiiboRequest`, `AmiiboResponse` y `AmiiboSearchResponse`.
- `GET /api/v1/**` público; escritura con auth + ownership (patrón global).

---

## Configuración

| Propiedad | Valor |
|-----------|-------|
| `amiiboapi.base-url` | `https://amiiboapi.org/api` |

Sin variable de entorno ni secreto (AmiiboAPI no requiere key).

**Cachés** (Caffeine, en `CacheConfig`): `amiiboSearch`, `amiiboList`, `amiiboDetail` — misma política TTL/grupo que las cachés del resto de colecciones (claves de list/detail incluyen el id del viewer).

**Converter:** `StringToAmiiboStatusConverter` para `?status=OWNED`/`WISHLIST`.

---

## API REST

**Base:** `/api/v1/amiibos`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/amiibos` | Listar amiibo (paginado; filtros name/status/owner) |
| GET | `/api/v1/amiibos/{id}` | Obtener amiibo por id interno |
| POST | `/api/v1/amiibos` | Añadir amiibo a la colección (auth) |
| PUT | `/api/v1/amiibos/{id}` | Actualizar (auth + ownership) |
| DELETE | `/api/v1/amiibos/{id}` | Eliminar (204, auth + ownership) |
| GET | `/api/v1/amiibos/search?name={query}` | Buscar en AmiiboAPI |

### Parámetros de listado

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `page` | Integer | Página (0-based) |
| `size` | Integer | Tamaño de página |
| `sort` | String | Ordenación `field,asc/desc` |
| `name` | String | Búsqueda local por nombre (LIKE) |
| `status` | Enum | Filtrar por estado: OWNED, WISHLIST |
| `owner` | String | `mine` (default) / `other` / `all` |

### DTOs

**AmiiboRequest (POST/PUT)**

```json
{
  "externalId": "string (obligatorio, head+tail)",
  "name": "string (obligatorio)",
  "character": "string",
  "amiiboSeries": "string",
  "gameSeries": "string",
  "amiiboType": "CARD | FIGURE | BAND | YARN | BLOCK",
  "releaseDates": { "au": "date", "eu": "date", "jp": "date", "na": "date" },
  "imageUrl": "string",
  "compatibleGames": ["string"],
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date"
}
```

**AmiiboResponse (GET)**

```json
{
  "id": "string",
  "externalId": "string",
  "name": "string",
  "character": "string",
  "amiiboSeries": "string",
  "gameSeries": "string",
  "amiiboType": "CARD | FIGURE | BAND | YARN | BLOCK",
  "releaseDates": { "au": "date", "eu": "date", "jp": "date", "na": "date" },
  "imageUrl": "string",
  "compatibleGames": ["string"],
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date",
  "userOwned": { "ownerId": "string", "ownerName": "string", "username": "string" }
}
```

**AmiiboSearchResponse (resultado externo)**

```json
{
  "externalId": "string",
  "name": "string",
  "character": "string",
  "amiiboSeries": "string",
  "gameSeries": "string",
  "amiiboType": "CARD | FIGURE | BAND | YARN | BLOCK",
  "releaseDates": { "au": "date", "eu": "date", "jp": "date", "na": "date" },
  "imageUrl": "string",
  "compatibleGames": ["string"],
  "externalSource": "amiiboapi"
}
```

---

## Frontend (al implementar)

- **Rutas**: `/amiibos` (listado), `/amiibos/:id` (detalle), búsqueda y formulario (modal o página, patrón de las colecciones existentes).
- **Nav**: entrada nueva en la navegación principal.
- **API client + tipos TS**: `Amiibo`, `AmiiboStatus` (OWNED | WISHLIST), `AmiiboType`.
- **Imágenes**: render directo de `imageUrl` (hotlink).

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `AmiiboNotFoundException` | 404 | Amiibo no encontrado |
| Error de catálogo externo (5xx/timeout) | 502/503 | Patrón del proyecto para APIs externas |

---

## Tests (al implementar)

| Test | Tipo | Descripción |
|------|------|-------------|
| `AmiiboServiceTest` | Unitario | CRUD + validaciones |
| `AmiiboSearchServiceTest` | Unitario | Búsqueda externa |
| `AmiiboApiClientTest` | Unitario | Cliente con mock — trampas de caso, contenedor único/lista, 200 vacío |
| `AmiiboControllerTest` | Integración | Endpoints + CORS + owner filter |
| `AmiiboPersistenceAdapterTest` | Integración | Persistencia Mongo (mock) |
| `AmiiboEntityMapperTest` | Unitario | Mapeo dominio ↔ entidad |

---

## Criterios de Aceptación

- [ ] `GET /api/v1/amiibos/search?name=mario` devuelve resultados de AmiiboAPI (coincidencia parcial permitida)
- [ ] Los resultados incluyen: externalId, name, character, amiiboSeries, gameSeries, amiiboType, imageUrl; y compatibleGames cuando aplica
- [ ] `GET /api/v1/amiibos` devuelve lista paginada vacía al inicio; filtra por `name`, `status` y `owner`
- [ ] `POST /api/v1/amiibos` añade un amiibo (auth requerido; externalId no duplicable)
- [ ] `PUT`/`DELETE /api/v1/amiibos/{id}` con auth + ownership verificados
- [ ] El cliente tolera `{"amiibo": []}` (200) como resultado vacío válido y el contenedor de objeto único con `?id=`
- [ ] Error de red del catálogo (5xx/timeout) → 502/503, no una lista vacía
- [ ] `mvn verify` verde (JaCoCo ≥ 0.80)

---

## Impacto en Documentación (tras aprobación)

- `specs/amiibos/spec.md` — nuevo (esta planificación convertida en spec).
- `specs/amiibos/api-contract.md` — nuevo.
- `data-model/schema.md` — colección `AMIIBOS` + índices.
- `architecture/backend.md` — controllers 11 → 12, cachés 40 → 43, paquetes nuevos (`amiibo`).
- `architecture/frontend-*.md` — rutas/nav.
- `requirements.md` — Fase 31 (propuesta, **sin marcar**).
- `README.md` — tabla de fases.
- `research/future/index.md` + `research/future/collectibles-funko-amiibo.md` — estado: Amiibo especificado.