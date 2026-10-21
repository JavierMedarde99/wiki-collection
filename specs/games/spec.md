---
type: spec
title: "Spec: Videojuegos (Games)"
description: "Reglas de negocio de videojuegos: estados, plataformas, DLC y logros"
tags: [api, games, spec]
---

# Spec: Videojuegos (Games)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** Game (domain/model/Game.java)

---

## Dominio

### Entidad Game

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| externalId | String | ❌ | ID en RAWG o FreeToGame |
| title | String | ✅ | Título del juego |
| platform | String (texto libre) | ✅ | Plataforma desde el catálogo RAWG (`GET /games/platforms`) — antes era enum GamePlatform |
| genres | List<String> | ❌ | Géneros (Fase 26) |
| thumbnailUrl | String | ❌ | URL al thumbnail |
| status | Enum (GameStatus) | ✅ | PLAYING, COMPLETED, WISHLIST, ABANDONED |
| userRating | Integer (1-5) | ❌ | Valoración personal |
| comment | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió |
| dateCompleted | LocalDate | ❌ | Cuándo se completó |
| externalSource | String | ❌ | RAWG, FreeToGame |
| steamAppId | String | ❌ | ID de juego en Steam (para logros) |
| acquisitionDate | LocalDate | ❌ | Fecha de adquisición |
| acquisitionPrice | BigDecimal | ❌ | Precio de adquisición |
| parentId | String | ❌ | id local del juego base del que cuelga el contenido adicional (`null` = base) |
| parentTitle | String | ❌ | Título del juego base, denormalizado para listar sin lookup |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |
| userOwned | Object | Auto | owner + visibilidad: ownerId, ownerName, username |

> **Campos de vínculo inmutables:** `parentId` y `parentTitle` **no** se aceptan en `POST`/`PUT` (`GameRequest` no los incluye y `GameService.copyUpdatableFields` no los copia). Solo los escribe el alta de contenido adicional (`GameAdditionsService`), ya sea creando el DLC o enlazar un juego que ya tenías.

### Enums

**GameStatus:** `PLAYING`, `COMPLETED`, `WISHLIST`, `ABANDONED`

### Reglas de Negocio

- **externalId es único** — no se pueden duplicar juegos por externalId
- **Platform es texto libre** — viene del catálogo dinámico RAWG (`GET /api/v1/games/platforms`); el enum `GamePlatform` fue eliminado
- **userRating:** rango 1-5, opcional
- **Fallback RAWG → FreeToGame:** si RAWG no devuelve resultados, se consulta FreeToGame

#### Contenido adicional (DLC)

- **El juego base debe venir de RAWG y tener `externalId`** para leer o recibir adiciones; si no ⇒ `422` (`GameAdditionsException`). Aquí **no** hay «catálogo vacío» como alternativa.
- **El base no puede estar en `WISHLIST`** para recibir adiciones ⇒ `422`.
- **La adición pedida tiene que ser una adición real del base en RAWG**; si no ⇒ `422`. Esa comprobación se hace contra el catálogo completo del base **antes de guardar nada**.
- **Tres desenlaces en el alta (201):** `created` (no la tenías), `linked` (ya la tenías suelta: se le escribe `parentId`/`parentTitle` y **conserva** su estado, nota, valoración, precio y fechas) y `skipped` (`reason: "ALREADY_LINKED"`, ya colgaba de este base).
- **Máximo 40 adiciones por petición** (`MAX_ADDITIONS`), que coincide con el `page_size` de la página de adiciones de RAWG. Ids duplicados se deduplican (se conserva la primera aparición).
- **Los flags dependen del espectador**: `inCollection` y `alreadyLinked` se calculan con los registros del usuario **fuera** de la caché; solo el catálogo puro de RAWG va cacheado (caché `gameAdditions`).
- **La búsqueda externa excluye adiciones**: `GET /games/search` llama a RAWG con `exclude_additions=true`, así que el listado de resultados no ofrece DLC ni ediciones.
- **Borrado en cascada**: al `DELETE` de un registro con `parentId == null` se borran primero sus DLC (`deleteByOwnerIdAndParentId` con el `ownerId` del dueño) y después el propio registro; la respuesta sigue siendo `204`.
- **Datos por estado en el alta detallada:** `WISHLIST` nace sin adquisición; el resto exige `acquisitionDate` + `acquisitionPrice`; `PLAYING`/`COMPLETED` exigen `dateAdded` (fecha de inicio) y `COMPLETED`, además, `dateCompleted`. Un ítem con `link: true` no exige datos (solo enlaza los suyos).
- **Plataforma de la adición:** la que RAWG asigne; si no viene, se hereda la del juego base.

---

## Puertos (Interfaces de Dominio)

### GameUseCase (in)
```java
public interface GameUseCase {
    Page<Game> search(GameSearchCriteria criteria, Pageable pageable);
    List<String> distinctGenres();
    Page<Game> search(GameSearchCriteria criteria, Pageable pageable, String owner, String viewerId);
    Game findById(String id);
    Game save(Game game, boolean obtainPlatinum, String ownerId);
    Game update(String id, Game updates, boolean obtainPlatinum, String userId);
    void delete(String id, String userId);
}
```

### GameSearchUseCase (in)
```java
public interface GameSearchUseCase {
    Page<GameSearchResult> search(String query, Pageable pageable);
}
```

### GameAchievementsUseCase (in)
```java
public interface GameAchievementsUseCase {
    AchievementsSummary getAchievements(String gameId, String steamId);
}
```

### GameAdditionsUseCase (in)
```java
public interface GameAdditionsUseCase {
    // Catálogo RAWG del base con inCollection/alreadyLinked del espectador.
    // 404 si el base no existe, 422 si no viene de RAWG o no tiene externalId, 502 si RAWG falla.
    GameAdditionsCatalog additionsOf(String baseId, String viewerId);

    // Alta de adiciones enlazadas al base (400 body, 404 base, 403 ajeno,
    // 422 regla de negocio, 502 RAWG): created + linked + skipped.
    GameAdditionsAddResult addAdditions(String baseId, List<GameAdditionSeed> additions, String userId);
}
```

Tipos de dominio nuevos: `GameAdditionsCatalog` (`baseGameId`, `baseExternalId`, `additions`, `totalAvailable`), `GameAddition` (entrada del catálogo con `inCollection` y `alreadyLinked`), `GameAdditionsPage` (primera página RAWG + total), `GameAdditionSeed` (externalId, estado, datos de adquisición/seguimiento, flag `link`), `GameAdditionsAddResult` (`created`, `linked`, `skipped`), `GameAdditionSkip` (`externalId` + `reason`). Puerto de salida: `ExternalGameAdditionsCatalogClient`.

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `GameService` | CRUD de juegos + validaciones |
| `GameSearchService` | Búsqueda externa RAWG + fallback FreeToGame + mapeo |
| `GameAchievementsService` | Obtención de logros Steam para juegos vinculados |
| `GameAdditionsService` | Catálogo y alta de contenido adicional (DLC) enlazado al juego base |

---

## APIs Externas

### RAWG (PRINCIPAL)

- **Base URL:** `https://api.rawg.io/api`
- **Auth:** API Key (gratuita con registro)
- **Rate limit:** 100,000 requests/mes (free tier)
- **Total juegos:** 500,000+
- **Estado:** Activa y mantenida

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /api/games?key={key}&search={query}&exclude_additions=true` |
| Detalle de juego | `GET /api/games/{id}?key={key}` |
| Adiciones (DLC) de un juego base | `GET /api/games/{id}/additions?page_size=40` |

> El contrato completo de RAWG (auth por `key`, `page_size`, qué mezcla `/additions` — DLC, ediciones GOTY, apps — y los filtros `exclude_*`) está en `research/external-apis/rawg.md`. `exclude_additions=true` es el filtro limpio de «solo juegos base»: por eso la búsqueda ya no ofrece adiciones.

### FreeToGame (FALLBACK)

- **Base URL:** `https://www.freetogame.com/api`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Total juegos:** ~415 (PC + Web Browser)
- **Estado:** Activa

| Uso | Endpoint |
|-----|----------|
| Todos los juegos | `GET /api/games` |
| Filtrar por plataforma | `GET /api/games?platform=windows` |
| Detalle de juego | `GET /api/game?id={game_id}` |

### Steam Achievements

- **Base URL:** `https://api.steampowered.com/ISteamUserStats/`
- **Auth:** API Key (gratuita)
- **Rate limit:** ~200 requests/5 minutos (sin key), ~100,000/día (con key)
- **Estado:** Activa

| Uso | Endpoint |
|-----|----------|
| Buscar juego por nombre (AppID) | `GET https://store.steampowered.com/api/storesearch/?term={nombre}` |
| Esquema de logros | `GET /GetSchemaForGame/v2/?key={key}&appid={appid}` |
| Logros de jugador | `GET /GetPlayerAchievements/v1/?key={key}&steamid={steamid}&appid={appid}` |

---

## Controlador REST

**Base:** `/api/v1/games`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/games` | Listar juegos (pagina, filtra por name/platform/status/genre/owner/type) |
| GET | `/api/v1/games/genres` | Catálogo de géneros (Fase 27) |
| GET | `/api/v1/games/platforms` | Catálogo de plataformas desde RAWG (texto libre) |
| GET | `/api/v1/games/{id}` | Obtener juego por ID |
| POST | `/api/v1/games` | Crear juego (auth requerido) |
| PUT | `/api/v1/games/{id}` | Actualizar juego (auth requerido, ownership verificado) |
| DELETE | `/api/v1/games/{id}` | Eliminar juego (204 No Content, auth requerido; si es base, borra antes sus DLC en cascada) |
| GET | `/api/v1/games/{id}/additions` | Catálogo RAWG de contenido adicional del juego base (público) |
| POST | `/api/v1/games/{id}/additions` | Dar de alta contenido adicional enlazado al base (201, auth requerido) |
| GET | `/api/v1/games/search?name={query}` | Buscar en RAWG (fallback a FreeToGame; RAWG recibe `exclude_additions=true`) |
| GET | `/api/v1/games/{gameId}/achievements?steamId={id}` | Logros de un jugador (Steam) |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Buscar por título |
| `platform` | String (repetible) | Filtrar por plataforma (texto libre, catálogo `GET /games/platforms`); varias plataformas = «cualquiera de» |
| `status` | Enum | Filtrar por estado (PLAYING, COMPLETED, WISHLIST, ABANDONED) |
| `genre` | String (repetible) | Filtrar por género (coincide con cualquiera de la lista) |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |
| `type` | Enum | Filtrar por tipo: `base`, `dlc`, `all` (por defecto `all`). `base` = `parentId` `null` (incluye los registros antiguos); `dlc` = `parentId` con valor. Valor desconocido ⇒ `400` |

---

## DTOs

### GameRequest (POST/PUT)

```json
{
  "externalId": "string",
  "title": "string (obligatorio)",
  "genres": ["string"],
  "platform": "string (obligatorio, texto libre del catálogo RAWG)",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer (1-5)",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "steamAppId": "string",
  "acquisitionDate": "date",
  "acquisitionPrice": "number (min 0, máx 2 decimales)"
}
```

> `GameRequest` **no** acepta `parentId` ni `parentTitle` (campos de solo lectura del flujo de adiciones); el `PUT` tampoco los modifica.

### GameResponse (GET)

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "genres": ["string"],
  "platform": "string",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "steamAppId": "string",
  "acquisitionDate": "date",
  "acquisitionPrice": "number",
  "parentId": "string (id del juego base, null si es base)",
  "parentTitle": "string",
  "userOwned": { "ownerId": "string", "ownerName": "string", "username": "string" }
}
```

### GameSearchResult (resultado de búsqueda externa)

```json
{
  "id": "string",
  "title": "string",
  "description": "string",
  "genre": "string",
  "platform": "string",
  "publisher": "string",
  "developer": "string",
  "releaseDate": "string",
  "thumbnailUrl": "string",
  "externalSource": "string"
}
```

### AchievementsResponse

```json
{
  "achievements": [
    {
      "name": "string",
      "description": "string",
      "achieved": "boolean",
      "iconUrl": "string"
    }
  ],
  "totalAchievements": "integer",
  "totalAchieved": "integer",
  "percentage": "number"
}
```

### GameAdditionsListResponse (GET /games/{id}/additions)

```json
{
  "baseGameId": "string (id local del juego base)",
  "baseExternalId": "string (externalId del base en RAWG)",
  "additions": [
    {
      "externalId": "string",
      "title": "string",
      "platform": "string",
      "genres": ["string"],
      "thumbnailUrl": "string",
      "releaseDate": "date",
      "externalSource": "string",
      "inCollection": "boolean (ya lo tienes, según el espectador)",
      "alreadyLinked": "boolean (ya colgaba de este base)"
    }
  ],
  "totalAvailable": "integer (adiciones que conoce RAWG en total, no solo las de la página)"
}
```

### AddGameAdditionsRequest (POST /games/{id}/additions)

Dos formas mutuamente excluyentes — hay que enviar **exactamente una** (ambas o ninguna ⇒ `400`), máximo 40 entradas:

```json
{
  "externalIds": ["string (no en blanco)"],
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED (obligatorio)"
}
```
```json
{
  "additions": [
    {
      "externalId": "string (obligatorio, no en blanco)",
      "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED (obligatorio)",
      "link": "boolean — true = solo enlazar uno que ya tengo; no exige datos",
      "acquisitionDate": "date — obligatorio fuera de WISHLIST y de los ítems link",
      "acquisitionPrice": "number (min 0, máx 8+2 dígitos) — obligatorio fuera de WISHLIST y de los ítems link",
      "dateAdded": "date (inicio) — obligatorio si status = PLAYING o COMPLETED",
      "dateCompleted": "date (fin) — obligatorio si status = COMPLETED"
    }
  ]
}
```

### GameAdditionsAddResponse (POST → 201 Created)

```json
{
  "created": [GameResponse, ...],
  "linked": [GameResponse, ...],
  "skipped": [ { "externalId": "string", "reason": "ALREADY_LINKED" }, ... ]
}
```

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `GameNotFoundException` | 404 | Juego no encontrado |
| `ForbiddenException` | 403 | No se tiene permiso sobre el recurso (ownership en escrituras) |
| `GameAdditionsException` | 422 | Regla de negocio del contenido adicional: base en `WISHLIST`, base sin `externalId` o que no viene de RAWG, `externalId` que no es una adición del base, o ítem `link` que ya no está en la colección |
| `ExternalCatalogException` | 502 | RAWG no responde o devuelve error al pedir las adiciones (el fallo no se enmascara como catálogo vacío) |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `GameServiceTest` | Unitario | CRUD de juegos |
| `GameControllerTest` | Integración | Endpoints de juegos (incluye `GET`/`POST /{id}/additions` y filtro `?type=`) |
| `GameSearchServiceTest` | Unitario | Búsqueda con fallback |
| `GameAchievementsServiceTest` | Unitario | Logros de Steam |
| `GameAdditionsServiceTest` | Unitario | Catálogo y alta de adiciones: 404/403/422/400/502, `created`/`linked`/`skipped`, dedupe, datos por estado |
| `RAWGClientTest` | Unitario | Cliente RAWG con mock |
| `FreeToGameClientTest` | Unitario | Cliente FreeToGame con mock |
| `GamePersistenceAdapterTest` | Integración | Persistencia de juegos |
| `GameDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
| `GameEntityMapperTest` | Unitario | Mapeo Domain ↔ Entity |
| `GameTest` | Unitario | Modelo de dominio Game |
| `GameServiceOwnerFilterTest` | Unitario | Filtro owner en juegos (Fase 9) |
| `StringToGameStatusConverterTest` | Unitario | Conversor String → GameStatus |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/games/search?name=overwatch` devuelve resultados de RAWG
- [x] Si RAWG no devuelve resultados, el endpoint usa FreeToGame como fallback
- [x] Los resultados incluyen: título, género, plataforma, publisher, developer, thumbnail, descripción
- [x] El endpoint `GET /api/v1/games` devuelve lista vacía al inicio
- [x] Se puede crear un juego vía `POST /api/v1/games` (auth requerido)
- [x] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/v1/games/{id}` (auth + ownership)
- [x] `GET /api/v1/games/{id}/additions` devuelve el catálogo RAWG del base con `inCollection` y `alreadyLinked` calculados para el espectador (fuera de la caché) y `totalAvailable`; `422` si el base no viene de RAWG o no tiene `externalId`; `404` si no existe; `502` si RAWG falla
- [x] `POST /api/v1/games/{id}/additions` responde `201` con `created`, `linked` y `skipped`; acepta exactamente una de las dos formas, máximo 40, y verifica los ids contra RAWG antes de guardar (`400`/`403`/`404`/`422`/`502`)
- [x] Al enlazar un juego que ya se tenía suelto se conservan estado, nota, valoración, precio y fechas (solo se escriben `parentId`/`parentTitle`)
- [x] `parentId` y `parentTitle` no se pueden fijar ni modificar vía `POST`/`PUT`
- [x] `GET /api/v1/games?type=base|dlc|all` filtra por tipo (por defecto `all`; los registros antiguos son `base`)
- [x] `DELETE /api/v1/games/{id}` de un base borra en cascada sus DLC y responde `204`
- [x] La búsqueda externa en RAWG envía `exclude_additions=true` (no ofrece adiciones)
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)

---

## Estado de Implementación

Fase 2 completada. Todos los componentes implementados:
- ✅ Domain: Game.java, GameStatus.java, GameSearchCriteria.java, GameSearchResult.java, PlatformInfo.java, SteamAchievement.java, AchievementsSummary.java, GameAddition.java, GameAdditionSeed.java, GameAdditionSkip.java, GameAdditionsAddResult.java, GameAdditionsCatalog.java, GameAdditionsPage.java, GameTypeFilter.java
- ✅ Ports: GameUseCase.java, GameSearchUseCase.java, GameAchievementsUseCase.java, GameAdditionsUseCase.java, PlatformCatalogUseCase.java, GameRepository.java, ExternalGameCatalogClient.java, ExternalGameAdditionsCatalogClient.java, SteamCatalogueClient.java
- ✅ Application: GameService.java, GameSearchService.java, GameAchievementsService.java, GameAdditionsService.java, GameAdditionsException.java
- ✅ Infrastructure: RAWGClient.java, FreeToGameClient.java, SteamAchievementsClient.java, GameController.java, GameEntity.java, GamePersistenceAdapter.java, SpringDataGameRepository.java, GameDtoMapper.java, GameAchievementMapper.java, AddGameAdditionsRequest.java, GameAdditionsAddResponse.java, GameAdditionResponse.java, GameAdditionSkipResponse.java, GameAdditionsListResponse.java
- ✅ Config: StringToGameStatusConverter.java, CacheConfig.java (caché `gameAdditions`)
- ✅ Tests: 13 archivos de test
