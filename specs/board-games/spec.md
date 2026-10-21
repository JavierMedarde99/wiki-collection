---
type: spec
title: "Spec: Juegos de Mesa (Board Games)"
description: "Reglas de negocio de juegos de mesa: estados, BGG, expansiones y valoración"
tags: [api, board-games, spec]
---

# Spec: Juegos de Mesa (Board Games)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** BoardGame (domain/model/BoardGame.java)

---

## Dominio

### Entidad BoardGame

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| title | String | ✅ | Nombre del juego de mesa |
| genres | List<String> | ❌ | Géneros (Fase 26) |
| description | String | ❌ | Descripción del juego |
| yearPublished | Integer | ❌ | Año de publicación |
| minPlayers | Integer | ❌ | Mínimo de jugadores |
| maxPlayers | Integer | ❌ | Máximo de jugadores |
| minPlaytime | Integer | ❌ | Duración mínima (minutos) |
| maxPlaytime | Integer | ❌ | Duración máxima (minutos) |
| publisher | String | ❌ | Editorial/publicador |
| designers | List<String> | ❌ | Lista de diseñadores |
| categories | List<String> | ❌ | Categorías del juego |
| mechanics | List<String> | ❌ | Mecánicas de juego |
| imageUrl | String | ❌ | URL de imagen completa |
| thumbnailUrl | String | ❌ | URL de miniatura |
| bggRating | BigDecimal | ❌ | Rating promedio BGG (0-10) |
| bggId | String | ❌ | ID externo de BoardGameGeek |
| status | Enum (BoardGameStatus) | ✅ | OWNED, WISHLIST |
| notes | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió a la colección |
| personalRating | Integer (1-5) | ❌ | Valoración personal (Fase 28) |
| playCount | Integer | ❌ | Número de jugadas (Fase 28) |
| lastPlayedDate | LocalDate | ❌ | Fecha de última partida (Fase 28) |
| difficulty | Enum (Difficulty) | ❌ | VERY_EASY, EASY, MEDIUM, HARD, VERY_HARD (Fase 28) |
| acquisitionPrice | BigDecimal | ❌ | Precio de adquisición (Fase 28) |
| bggType | Enum (BoardGameType) | ❌ | Tipo BGG: `boardgame` o `boardgameexpansion`. `null` = base (todos los registros que se crean hoy: alta manual, búsqueda y API) |
| parentId | String | ❌ | id local del juego base del que cuelga (`null` si es base) |
| parentTitle | String | ❌ | Título del juego base, denormalizado para listar sin lookup |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |
| userOwned | Object | Auto | owner + visibilidad: ownerId, ownerName, username |

> **Campos de expansión inmutables:** `bggType`, `parentId` y `parentTitle` **no** se aceptan en `POST`/`PUT` (`BoardGameRequest` no los incluye y `BoardGameService.copyUpdatableFields` no los copia). Solo los escribe el alta de expansiones (`BoardGameExpansionService`).

### Enum

**BoardGameStatus:** `OWNED`, `WISHLIST`

**Difficulty:** `VERY_EASY`, `EASY`, `MEDIUM`, `HARD`, `VERY_HARD`

**BoardGameType:** `boardgame`, `boardgameexpansion` — vocabulario JSON de BGG (`@JsonValue`/`@JsonCreator`, acepta mayúsculas y espacios; clave desconocida ⇒ `IllegalArgumentException`). `null` en los registros que no han entrado por el flujo de expansiones.

**BoardGameTypeFilter** (filtro de listado, no del modelo): `BASE`, `EXPANSION`, `ALL`. Es independiente de `BoardGameType`: ese enum mapea el vocabulario de BGG y no contempla «todo», que es lo que pide el listado por defecto.

> **Nota:** El enum original tenía 4 valores (OWNED, WISHLIST, PREVIOUSLY_OWNED, FOR_TRADE) pero la implementación real solo usa 2. AD-012: se redujo a OWNED y WISHLIST.

### Reglas de Negocio

- **BggId es único** — no se pueden duplicar juegos por ID de BGG
- **Status only has 2 values** — OWNED y WISHLIST (no PREVIOUSLY_OWNED ni FOR_TRADE)

#### Expansiones (base ↔ expansión)

- **El juego base debe estar en `OWNED`** para añadirle expansiones; en `WISHLIST` ⇒ `422` (`BoardGameExpansionException`).
- **El alta valida todo antes de guardar nada** — una expansión inválida tira la petición entera: no hay altas parciales.
- **La relación se verifica contra BGG**: cada `bggId` tiene que existir, ser de tipo `boardgameexpansion` y colgar de ese base en BGG (links `boardgameexpansion` con `inbound="true"`). Si no ⇒ `422`. Si el base no tiene `bggId` no se puede verificar ⇒ `422`.
- **Alta idempotente**: lo que ya está en la colección del usuario (por `bggId`) no se crea y se devuelve en `skipped` con `reason: "ALREADY_IN_COLLECTION"`.
- **Máximo 20 ids por petición** (`MAX_BGG_IDS`), el mismo límite que `/thing` de BGG por llamada; las consultas a BGG se trocean en lotes de 20.
- **Una expansión nace con los datos de su semilla**: `OWNED` con precio y fecha de adición (`dateAdded` opcional ⇒ hoy); `WISHLIST` sin precio ni fecha. La forma legada (`bggIds` simple) crea todo en `OWNED`, **sin precio** y con fecha de hoy.
- **Los catálogos no se enriquecen con flags dentro de la caché**: `GET /{id}/expansions` cachea solo el catálogo puro de BGG (caché `boardgameExpansions`, clave = `bggId` del base) y `inCollection` se calcula después con los registros del espectador (anónimo ⇒ todo `false`).
- **Borrado en cascada**: al `DELETE` de un registro que **no** es expansión se borran primero sus expansiones (`deleteByOwnerIdAndParentId` con el `ownerId` del dueño) y después el propio registro; la respuesta sigue siendo `204`.

---

## Puertos (Interfaces de Dominio)

### BoardGameUseCase (in)
```java
public interface BoardGameUseCase {
    Page<BoardGame> search(BoardGameSearchCriteria criteria, Pageable pageable);
    List<String> distinctGenres();
    Page<BoardGame> search(BoardGameSearchCriteria criteria, Pageable pageable, String owner, String viewerId);
    BoardGame findById(String id);
    BoardGame save(BoardGame boardGame, String ownerId);
    BoardGame update(String id, BoardGame updates, String userId);
    void delete(String id, String userId);
}
```

### BoardGameSearchUseCase (in)
```java
public interface BoardGameSearchUseCase {
    Page<BoardGameSearchResult> search(String query, Pageable pageable);
}
```

### BoardGameExpansionUseCase (in)
```java
public interface BoardGameExpansionUseCase {
    // Catálogo BGG del base con inCollection del espectador (anónimo ⇒ false).
    // 404 si el base no existe, 502 si BGG falla; catálogo vacío si el base no tiene bggId.
    BoardGameExpansionCatalog expansionsOf(String baseId, String viewerId);

    // Alta de expansiones enlazadas al base (400 body, 404 base, 403 ajeno,
    // 422 regla de negocio, 502 BGG). Idempotente: lo ya coleccionado va a skipped.
    BoardGameExpansionAddResult addExpansions(String baseId, List<BoardGameExpansionSeed> seeds, String userId);
}
```

Tipos de dominio nuevos: `BoardGameExpansionCatalog` (`baseGameId`, `baseBggId`, `expansions`), `BoardGameExpansion` (`bggId`, `title`, `yearPublished`, `imageUrl`, `thumbnailUrl`, `inCollection`), `BoardGameExpansionInfo` (entrada del catálogo BGG con `type` y `baseBggIds`), `BoardGameExpansionSeed` (bggId + estado + precio + fecha), `BoardGameExpansionAddResult` (`created`, `skipped`), `BoardGameExpansionSkip` (`bggId` + `reason`).

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `BoardGameService` | CRUD de juegos de mesa + validaciones |
| `BoardGameSearchService` | Búsqueda externa BGG XML + parseo + mapeo |
| `BoardGameExpansionService` | Catálogo y alta de expansiones enlazadas a su base (idempotente, todo-validado-antes-de-guardar) |

---

## API Externa: BoardGameGeek XML API 2

- **Base URL:** `https://boardgamegeek.com/xmlapi2`
- **Auth:** No requerida
- **Rate limit:** Variable (~10 requests/segundo recomendado)
- **Gratis:** Sí
- **Formato:** XML (parseado con Jackson XML)
- **Total juegos:** 100,000+
- **Estado:** Activa y mantenida por BGG

### Endpoints usados

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /search?query={query}&type=boardgame` |
| Obtener detalle | `GET /thing?id={id}&stats=1` |
| Expansiones de un base y sus fichas | `GET /thing?id={ids}` (hasta 20 ids por llamada, `id` separado por comas) |
| Colección usuario | `GET /collection/{username}?own=1` |

> El detalle del contrato (tipos de `item`, semántica de `inbound` en los links de expansión, `exclude_*`, 202/cookie, auth) está en `research/external-apis/boardgamegeek.md`. En particular: **un link `boardgameexpansion` sin `inbound` es una expansión *de* este base; con `inbound` apunta *hacia* su base** — de ahí que el alta verifique `baseBggIds` contra el `bggId` del base.

### Mapeo BGG XML → BoardGameSearchResult

| Campo BGG | Campo SearchResult | Notas |
|-----------|--------------------|-------|
| `id` | `bggId` | ID externo de BGG |
| `name` | `title` | Nombre del juego |
| `yearpublished` | `yearPublished` | Año de publicación |
| `minplayers` | `minPlayers` | Mínimo de jugadores |
| `maxplayers` | `maxPlayers` | Máximo de jugadores |
| `minplaytime` | `minPlaytime` | Duración mínima |
| `maxplaytime` | `maxPlaytime` | Duración máxima |
| `description` | `description` | Descripción |
| `thumbnail` | `thumbnailUrl` | Miniatura |
| `image` | `imageUrl` | Imagen completa |
| `publisher` | `publisher` | Editorial |
| `designers` | `designers` | Lista de diseñadores |
| `categories` | `categories` | Categorías |
| `mechanics` | `mechanics` | Mecánicas |
| `rating` | `bggRating` | Rating promedio |

---

## Controlador REST

**Base:** `/api/v1/boardgames`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/boardgames` | Listar juegos de mesa (pagina, filtra por name/status/genre/owner/type) |
| GET | `/api/v1/boardgames/genres` | Catálogo de géneros en uso (excluye colecciones privadas) |
| GET | `/api/v1/boardgames/{id}` | Obtener juego de mesa por ID |
| POST | `/api/v1/boardgames` | Crear juego de mesa (auth requerido) |
| PUT | `/api/v1/boardgames/{id}` | Actualizar juego de mesa (auth requerido, ownership verificado) |
| DELETE | `/api/v1/boardgames/{id}` | Eliminar juego de mesa (204 No Content, auth requerido; si no es expansión, borra antes sus expansiones en cascada) |
| GET | `/api/v1/boardgames/{id}/expansions` | Catálogo BGG de las expansiones de un juego base (público) |
| POST | `/api/v1/boardgames/{id}/expansions` | Dar de alta expansiones enlazadas al base (201, auth requerido) |
| GET | `/api/v1/boardgames/search?name={query}` | Buscar en BoardGameGeek (XML API) |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Buscar por título |
| `status` | Enum | Filtrar por estado (OWNED, WISHLIST) |
| `genre` | String (repetible) | Filtrar por género (coincide con cualquiera de la lista) |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |
| `type` | Enum | Filtrar por tipo: `base`, `expansion`, `all` (por defecto `all`). `base` = `bggType` `null` o `boardgame` (incluye los registros antiguos); `expansion` = `boardgameexpansion`. Valor desconocido ⇒ `400` |

---

## DTOs

### BoardGameRequest (POST/PUT)

```json
{
  "title": "string (obligatorio)",
  "genres": ["string"],
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer (min 1)",
  "maxPlayers": "integer (min 1)",
  "minPlaytime": "integer (min 1)",
  "maxPlaytime": "integer (min 1)",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number (0-10)",
  "bggId": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date",
  "personalRating": "integer (1-5)",
  "playCount": "integer (min 0)",
  "lastPlayedDate": "date",
  "difficulty": "VERY_EASY | EASY | MEDIUM | HARD | VERY_HARD",
  "acquisitionPrice": "number (min 0, máx 2 decimales)"
}
```

> `BoardGameRequest` **no** acepta `bggType`, `parentId` ni `parentTitle`: los envíes o no, la API los ignora y `copyUpdatableFields` no los pisa en el `PUT` (son de solo lectura, los escribe el alta de expansiones).

### BoardGameResponse (GET)

```json
{
  "id": "string",
  "title": "string",
  "genres": ["string"],
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer",
  "maxPlayers": "integer",
  "minPlaytime": "integer",
  "maxPlaytime": "integer",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number",
  "bggId": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date",
  "personalRating": "integer (1-5)",
  "playCount": "integer",
  "lastPlayedDate": "date",
  "difficulty": "VERY_EASY | EASY | MEDIUM | HARD | VERY_HARD",
  "acquisitionPrice": "number",
  "userOwned": { "ownerId": "string", "ownerName": "string", "username": "string" },
  "bggType": "boardgame | boardgameexpansion | null",
  "parentId": "string (id del juego base, null si es base)",
  "parentTitle": "string"
}
```

### BoardGameSearchResponse (resultado de búsqueda externa)

```json
{
  "bggId": "string",
  "title": "string",
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer",
  "maxPlayers": "integer",
  "minPlaytime": "integer",
  "maxPlaytime": "integer",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number",
  "externalSource": "string"
}
```

### BoardGameExpansionListResponse (GET /boardgames/{id}/expansions)

```json
{
  "baseGameId": "string (id local del juego base)",
  "baseBggId": "string (null si el base no tiene bggId ⇒ catálogo vacío)",
  "expansions": [
    {
      "bggId": "string",
      "title": "string",
      "yearPublished": "integer",
      "imageUrl": "string",
      "thumbnailUrl": "string",
      "inCollection": "boolean (según el espectador; anónimo ⇒ false)"
    }
  ]
}
```

### AddExpansionsRequest (POST /boardgames/{id}/expansions)

Dos formas mutuamente excluyentes — hay que enviar **exactamente una** (ambas o ninguna ⇒ `400`), máximo 20 entradas por petición:

```json
{
  "bggIds": ["string (obligatorio, no en blanco)"]
}
```
```json
{
  "expansions": [
    {
      "bggId": "string (obligatorio)",
      "status": "OWNED | WISHLIST (obligatorio)",
      "acquisitionPrice": "number (min 0) — obligatorio si status = OWNED",
      "dateAdded": "date — obligatorio si status = OWNED"
    }
  ]
}
```

### AddExpansionsResponse (201 Created)

```json
{
  "created": [BoardGameResponse, ...],
  "skipped": [ { "bggId": "string", "reason": "ALREADY_IN_COLLECTION" }, ... ]
}
```

### SkippedExpansionResponse

```json
{
  "bggId": "string",
  "reason": "ALREADY_IN_COLLECTION"
}
```

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `BoardGameNotFoundException` | 404 | Juego de mesa no encontrado |
| `ForbiddenException` | 403 | No se tiene permiso sobre el recurso (ownership en escrituras) |
| `BoardGameExpansionException` | 422 | Regla de negocio de expansiones: base no en `OWNED`, base sin `bggId`, `bggId` desconocido o que no es una expansión de ese base |
| `ExternalCatalogException` | 502 | BGG no responde o devuelve error en la lectura estricta de `/thing` (las expansiones no confunden «BGG caído» con «catálogo vacío») |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `BoardGameServiceTest` | Unitario | CRUD de juegos de mesa |
| `BoardGameControllerTest` | Integración | Endpoints de juegos de mesa (incluye `GET`/`POST /{id}/expansions` y filtro `?type=`) |
| `BoardGameSearchServiceTest` | Unitario | Búsqueda BGG |
| `BggXmlClientTest` | Unitario | Cliente BGG XML con mock |
| `BoardGamePersistenceAdapterTest` | Integración | Persistencia de juegos de mesa |
| `BoardGameXmlMapperTest` | Unitario | Mapeo XML → BoardGame |
| `BoardGameDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
| `BoardGameEntityMapperTest` | Unitario | Mapeo Domain ↔ Entity |
| `BoardGameStatusMigrationTest` | Unitario | Migración de estado |
| `BoardGameServiceOwnerFilterTest` | Unitario | Filtro owner en board games (Fase 9) |
| `BoardGameExpansionServiceTest` | Unitario | Catálogo y alta de expansiones: 404/403/422/400/502, idempotencia, OWNED con precio y fecha, WISHLIST sin datos |
| `BoardGameTypeTest` | Unitario | Vocabulario JSON del enum `BoardGameType` (`boardgame` / `boardgameexpansion`) |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/boardgames/search?name=catan` devuelve resultados de BGG XML
- [x] Los resultados incluyen: título, año, jugadores (min/max), duración, publisher, diseñadores, categorías, mecánicas, imagen, rating
- [x] El endpoint `GET /api/v1/boardgames/{id}` devuelve el detalle completo de un juego
- [x] El endpoint `GET /api/v1/boardgames` devuelve lista vacía al inicio
- [x] Se puede crear un juego de mesa vía `POST /api/v1/boardgames` (auth requerido)
- [x] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/v1/boardgames/{id}` (auth + ownership)
- [x] Se puede buscar por nombre parcial (búsqueda flexible)
- [x] Los errores de API externa se manejan correctamente (503, 429, timeout)
- [x] `GET /api/v1/boardgames/{id}/expansions` devuelve el catálogo de BGG del base con `inCollection` del espectador (anónimo ⇒ `false`); catálogo vacío con `baseBggId: null` si el base no tiene `bggId`; `404` si no existe; `502` si BGG falla
- [x] `POST /api/v1/boardgames/{id}/expansions` responde `201` con `created` y `skipped`; acepta exactamente una de las dos formas (`bggIds` o `expansions`), máximo 20, y valida todo antes de guardar (`400`/`403`/`404`/`422`/`502`)
- [x] El alta es idempotente: lo que ya está en la colección aparece en `skipped` con `reason: ALREADY_IN_COLLECTION`
- [x] `bggType`, `parentId` y `parentTitle` no se pueden fijar ni modificar vía `POST`/`PUT`
- [x] `GET /api/v1/boardgames?type=base|expansion|all` filtra por tipo (por defecto `all`; los registros antiguos son `base`)
- [x] `DELETE /api/v1/boardgames/{id}` borra en cascada las expansiones de un base y responde `204`
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)

---

## Estado de Implementación

Fase 3 completada. Todos los componentes implementados:
- ✅ Domain: BoardGame.java, BoardGameStatus.java, BoardGameSearchCriteria.java, BoardGameSearchResult.java, BoardGameType.java, BoardGameTypeFilter.java, BoardGameExpansion.java, BoardGameExpansionInfo.java, BoardGameExpansionCatalog.java, BoardGameExpansionSeed.java, BoardGameExpansionAddResult.java, BoardGameExpansionSkip.java
- ✅ Ports: BoardGameUseCase.java, BoardGameSearchUseCase.java, BoardGameExpansionUseCase.java, BoardGameRepository.java, ExternalBoardGameCatalogClient.java
- ✅ Application: BoardGameService.java, BoardGameSearchService.java, BoardGameExpansionService.java, BoardGameExpansionException.java
- ✅ Infrastructure: BggXmlClient.java, BoardGameXmlMapper.java, BoardGameController.java, BoardGameEntity.java, BoardGamePersistenceAdapter.java, SpringDataBoardGameRepository.java, BoardGameDtoMapper.java, BoardGameSearchResponse.java, AddExpansionsRequest.java, AddExpansionsResponse.java, SkippedExpansionResponse.java, BoardGameExpansionListResponse.java, BoardGameExpansionResponse.java
- ✅ Config: BggClientConfig.java, StringToBoardGameStatusConverter.java, CacheConfig.java (caché `boardgameExpansions`)
- ✅ Tests: 12 archivos de test
