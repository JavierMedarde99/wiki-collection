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
| platform | Enum (GamePlatform) | ✅ | PC, PS2, PS3, WII_U, SWITCH |
| thumbnailUrl | String | ❌ | URL al thumbnail |
| status | Enum (GameStatus) | ✅ | PLAYING, COMPLETED, WISHLIST, ABANDONED |
| userRating | Integer (1-5) | ❌ | Valoración personal |
| comment | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió |
| dateCompleted | LocalDate | ❌ | Cuándo se completó |
| externalSource | String | ❌ | RAWG, FreeToGame |
| steamAppId | String | ❌ | ID de juego en Steam (para logros) |
| obtainPlatinum | Boolean | ❌ | Si obtuvo trophy platinum |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Enums

**GamePlatform:** `PC`, `PS2`, `PS3`, `WII_U`, `SWITCH`

**GameStatus:** `PLAYING`, `COMPLETED`, `WISHLIST`, `ABANDONED`

### Reglas de Negocio

- **externalId es único** — no se pueden duplicar juegos por externalId
- **Platform es requerido** — debe ser uno de los valores del enum
- **userRating:** rango 1-5, opcional
- **Fallback RAWG → FreeToGame:** si RAWG no devuelve resultados, se consulta FreeToGame

---

## Puertos (Interfaces de Dominio)

### GameUseCase (in)
```java
public interface GameUseCase {
    Game save(Game game, String ownerId);
    Game update(String id, Game updates, String ownerId);
    void delete(String id, String ownerId);
    Game findById(String id);
    Page<Game> search(GameSearchCriteria criteria, Pageable pageable);
}
```

### GameSearchUseCase (in)
```java
public interface GameSearchUseCase {
    List<GameSearchResult> searchExternal(String query);
    GameSearchResult findById(String externalId);
}
```

### GameAchievementsUseCase (in)
```java
public interface GameAchievementsUseCase {
    AchievementsSummary getAchievements(String steamAppId, String steamId);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `GameService` | CRUD de juegos + validaciones |
| `GameSearchService` | Búsqueda externa RAWG + fallback FreeToGame + mapeo |
| `GameAchievementsService` | Obtención de logros Steam para juegos vinculados |

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
| Buscar juegos | `GET /api/games?key={key}&search={query}` |
| Detalle de juego | `GET /api/games/{id}?key={key}` |

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
| GET | `/api/v1/games` | Listar juegos (pagina, filtra por name/platform/status/owner) |
| GET | `/api/v1/games/{id}` | Obtener juego por ID |
| POST | `/api/v1/games` | Crear juego (auth requerido) |
| PUT | `/api/v1/games/{id}` | Actualizar juego (auth requerido, ownership verificado) |
| DELETE | `/api/v1/games/{id}` | Eliminar juego (204 No Content, auth requerido) |
| GET | `/api/v1/games/search?name={query}` | Buscar en RAWG (fallback a FreeToGame) |
| GET | `/api/v1/games/{gameId}/achievements?steamId={id}` | Logros de un jugador (Steam) |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Buscar por título |
| `platform` | Enum | Filtrar por plataforma (PC, PS2, PS3, WII_U, SWITCH) |
| `status` | Enum | Filtrar por estado (PLAYING, COMPLETED, WISHLIST, ABANDONED) |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |

---

## DTOs

### GameRequest (POST/PUT)

```json
{
  "externalId": "string",
  "title": "string (obligatorio)",
  "platform": "PC | PS2 | PS3 | WII_U | SWITCH",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer (1-5)",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "steamAppId": "string",
  "obtainPlatinum": "boolean"
}
```

### GameResponse (GET)

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "platform": "PC | PS2 | PS3 | WII_U | SWITCH",
  "thumbnailUrl": "string",
  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
  "userRating": "integer",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "steamAppId": "string",
  "obtainPlatinum": "boolean",
  "ownerId": "string"
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

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `GameNotFoundException` | 404 | Juego no encontrado |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `GameServiceTest` | Unitario | CRUD de juegos |
| `GameControllerTest` | Integración | Endpoints de juegos |
| `GameSearchServiceTest` | Unitario | Búsqueda con fallback |
| `GameAchievementsServiceTest` | Unitario | Logros de Steam |
| `RAWGClientTest` | Unitario | Cliente RAWG con mock |
| `FreeToGameClientTest` | Unitario | Cliente FreeToGame con mock |
| `GamePersistenceAdapterTest` | Integración | Persistencia de juegos |
| `GameDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
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
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)

---

## Estado de Implementación

Fase 2 completada. Todos los componentes implementados:
- ✅ Domain: Game.java, GameStatus.java, GamePlatform.java, GameSearchCriteria.java, GameSearchResult.java, SteamAchievement.java, AchievementsSummary.java
- ✅ Ports: GameUseCase.java, GameSearchUseCase.java, GameAchievementsUseCase.java, GameRepository.java, ExternalGameCatalogClient.java, SteamCatalogueClient.java
- ✅ Application: GameService.java, GameSearchService.java, GameAchievementsService.java
- ✅ Infrastructure: RAWGClient.java, FreeToGameClient.java, SteamAchievementsClient.java, GameController.java, GameEntity.java, GamePersistenceAdapter.java, SpringDataGameRepository.java, GameDtoMapper.java, GameAchievementMapper.java
- ✅ Config: StringToGameStatusConverter.java
- ✅ Tests: 11 archivos de test
