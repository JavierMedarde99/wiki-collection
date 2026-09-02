# Fase 2: Colección de Videojuegos

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 (arquitectura hexagonal) para:

1. Hacer llamadas a las APIs externas **RAWG** (principal) y **FreeToGame** (secundaria) para buscar videojuegos
2. Devolver la información de videojuegos en el endpoint `GET /api/games/search`
3. CRUD completo de videojuegos en la colección local (MongoDB)

---

## APIs Externas

### RAWG Video Games Database API (PRINCIPAL)

- **Base URL:** `https://api.rawg.io/api`
- **Auth:** API Key (gratuita con registro en https://rawg.io/apidocs)
- **Rate limit:** 100,000 requests/mes (free tier)
- **Gratis:** Sí, con registro gratuito
- **Total juegos:** 500,000+
- **Documentación:** https://api.rawg.io/docs/

**Endpoints útiles:**

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /api/games?key={key}&search={query}` |
| Detalle de juego | `GET /api/games/{id}?key={key}` |
| Juegos por plataforma | `GET /api/games?key={key}&platforms={ids}` |
| Juegos por género | `GET /api/games?key={key}&genres={slug}` |
| Juegos populares | `GET /api/games?key={key}&ordering=-rating` |

**Ventajas:**
- ✅ Búsqueda por nombre nativa
- ✅ 500,000+ juegos
- ✅ Datos muy completos (ratings, screenshots, trailers, géneros, tags)
- ✅ 50 plataformas incluyendo móviles
- ✅ 100,000 requests/mes gratis

**Estructura de respuesta (un juego):**

```json
{
  "id": 459414,
  "slug": "overwatch",
  "name": "Overwatch",
  "released": "2016-05-24",
  "background_image": "https://media.rawg.io/media/games/6c5/...",
  "rating": 4.05,
  "ratings_count": 1234,
  "playtime": 6,
  "platforms": [
    {"platform": {"id": 4, "name": "PC", "slug": "pc"}},
    {"platform": {"id": 18, "name": "PlayStation 4", "slug": "playstation4"}}
  ],
  "genres": [
    {"id": 4, "name": "Action", "slug": "action"},
    {"id": 5, "name": "Shooter", "slug": "shooter"}
  ],
  "stores": [
    {"store": {"id": 1, "name": "Steam", "slug": "steam"}}
  ],
  "tags": [
    {"id": 31, "name": "Singleplayer", "slug": "singleplayer"}
  ],
  "short_screenshots": [
    {"id": 123, "image": "https://media.rawg.io/media/screenshots/..."}
  ]
}
```

---

### FreeToGame API (SECUNDARIA)

- **Base URL:** `https://www.freetogame.com/api`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí, completamente
- **Total juegos:** ~415 (PC + Web Browser)
- **Documentación:** https://www.freetogame.com/api-doc

**Endpoints:**

| Uso | Endpoint |
|-----|----------|
| Todos los juegos | `GET /api/games` |
| Filtrar por plataforma | `GET /api/games?platform=windows` |
| Filtrar por categoría | `GET /api/games?category=shooter` |
| Ordenar | `GET /api/games?sort-by=alphabetical` |
| Detalle de juego | `GET /api/game?id={game_id}` |
| Filtrar por tags | `GET /api/filter?tag=3d.mmorpg.fantasy` |

**Ventajas:**
- ✅ Sin autenticación
- ✅ Sin API key
- ✅ 100% gratis
- ❌ No soporta búsqueda por nombre
- ❌ Catálogo limitado (~415 juegos)

**Estructura de respuesta (un juego):**

```json
{
  "id": 540,
  "title": "Overwatch",
  "thumbnail": "https://www.freetogame.com/g/540/thumbnail.jpg",
  "short_description": "A hero-focused first-person team shooter...",
  "game_url": "https://www.freetogame.com/open/overwatch",
  "genre": "Shooter",
  "platform": "PC (Windows)",
  "publisher": "Activision Blizzard",
  "developer": "Blizzard Entertainment",
  "release_date": "2022-10-04",
  "freetogame_profile_url": "https://www.freetogame.com/overwatch"
}
```

---

## Estrategia de Implementación

1. **RAWG como primaria** — Búsqueda por nombre, 500k+ juegos, datos completos
2. **FreeToGame como secundaria** — Fallback para juegos específicos, sin auth
3. **Combinación** — Usar RAWG para búsqueda general y FreeToGame para juegos gratuitos específicos

### Flujo de Búsqueda

```
1. Cliente → GET /api/games/search?name=overwatch
2. Backend → RAWG API (search)
3. Si hay resultados → Mapear y devolver
4. Si no hay resultados → FreeToGame API (getAll + filter)
5. Mapear y devolver
```

---

## Paso 1: Modelo de Datos — Game (Dominio)

- [ ] Crear enum `GameStatus`: PLAYING, COMPLETED, WISHLIST, ABANDONED
- [ ] Crear enum `GamePlatform`: PC, WEB_BROWSER, BOTH (mapeo de plataformas FreeToGame)
- [ ] Crear documento `Game.java` en `domain/model/`:
  - Anotación `@Document(collection = "GAMES")`
  - Campos: id, externalId (RAWG/FreeToGame ID), title, description, genre, platform, publisher, developer, releaseDate, thumbnailUrl, status, userRating (1-5), notes, dateAdded, dateCompleted, externalSource

## Paso 2: Repositorio — Puerto y Adaptador

- [ ] Crear interface `GameRepository.java` en `domain/port/out/`:
  - `findAll(Pageable)`
  - `findById(String)`
  - `findByStatus(GameStatus, Pageable)`
  - `save(Game)`
  - `deleteById(String)`
- [ ] Crear interface `ExternalGameCatalogClient.java` en `domain/port/out/`:
  - `search(String query) → List<GameSearchResult>`
  - `getAllGames() → List<GameSearchResult>`
- [ ] Crear `SpringDataGameRepository.java` en `infrastructure/adapter/out/persistence/`:
  - `findByStatus(GameStatus, Pageable)`
  - `findByExternalId(String)`
- [ ] Crear `GameEntity.java` en `infrastructure/adapter/out/persistence/`:
  - `@Document(collection = "GAMES")`
  - Mismos campos que dominio
- [ ] Crear `GameEntityMapper.java` en `infrastructure/adapter/out/persistence/`:
  - `toDomain(GameEntity)`
  - `toEntity(Game)`
- [ ] Crear `GamePersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`:
  - Implementa `GameRepository`
  - Usa `SpringDataGameRepository` + `GameEntityMapper`

## Paso 3: Servicio — Caso de Uso y Aplicación

- [ ] Crear interface `GameUseCase.java` en `domain/port/in/`:
  - `findAll(Pageable)`
  - `findByStatus(GameStatus, Pageable)`
  - `findById(String)`
  - `save(Game)`
  - `update(String, Game)`
  - `delete(String)`
- [ ] Crear interface `GameSearchUseCase.java` en `domain/port/in/`:
  - `search(String query)`
- [ ] Crear `GameService.java` en `application/service/`:
  - Implementa `GameUseCase`
  - Lógica de CRUD
- [ ] Crear `GameSearchService.java` en `application/service/`:
  - Implementa `GameSearchUseCase`
  - Valida query no vacío
  - **Intenta primero RAWG** (búsqueda por nombre nativa)
  - **Fallback a FreeToGame** si RAWG no devuelve resultados
  - Cachea resultados (TTL 1 hora)
- [ ] Crear `RAWGClient.java` en `infrastructure/adapter/out/rawg/`:
  - Implementa `ExternalGameCatalogClient`
  - Usa `RestClient` para llamar a RAWG
  - Mapea respuesta al modelo `GameSearchResult`
  - Manejo de errores (graceful degradation: lista vacía si falla)
- [ ] Crear `FreeToGameClient.java` en `infrastructure/adapter/out/freetogame/`:
  - Implementa `ExternalGameCatalogClient`
  - Usa `RestClient` para llamar a FreeToGame
  - Mapea respuesta al modelo `GameSearchResult`
  - Manejo de errores (graceful degradation: lista vacía si falla)

## Paso 4: Controller — Endpoints REST

- [ ] Crear `GameController.java` en `infrastructure/adapter/in/web/`:
  - `GET /api/games` — listar con paginación y filtro por status
  - `GET /api/games/{id}` — obtener juego por ID
  - `POST /api/games` — añadir juego a la colección
  - `PUT /api/games/{id}` — actualizar juego
  - `DELETE /api/games/{id}` — eliminar juego
  - `GET /api/games/search?name={query}` — buscar en RAWG (principal) + FreeToGame (secundaria)
- [ ] Crear `GameRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`:
  - Record con campos de entrada + validaciones
- [ ] Crear `GameResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`:
  - Record con campos de salida
- [ ] Crear `GameDtoMapper.java` en `infrastructure/adapter/in/web/dto/`:
  - `toDomain(GameRequest)`
  - `toResponse(Game)`
- [ ] Crear `StringToGameStatusConverter.java` en `infrastructure/config/`:
  - Convierte String → GameStatus

## Paso 5: Tests

- [ ] `GameServiceTest.java` — Tests unitarios de CRUD
- [ ] `GameControllerTest.java` — Tests de integración MockMvc
- [ ] `RAWGClientTest.java` — Tests del cliente RAWG con mock server
- [ ] `FreeToGameClientTest.java` — Tests del cliente FreeToGame con mock server
- [ ] `GamePersistenceAdapterTest.java` — Tests del adaptador de persistencia
- [ ] `GameSearchServiceTest.java` — Tests del flujo de búsqueda con fallback

## Criterios de Aceptación

- [ ] El endpoint `GET /api/games/search?name=overwatch` devuelve resultados de RAWG
- [ ] Si RAWG no devuelve resultados, el endpoint usa FreeToGame como fallback
- [ ] Los resultados incluyen: título, género, plataforma, publisher, developer, thumbnail, descripción
- [ ] El endpoint `GET /api/games` devuelve lista vacía al inicio
- [ ] Se puede crear un juego vía `POST /api/games`
- [ ] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/games/{id}`
- [ ] Los tests pasan (`mvn verify`)
- [ ] El respeta arquitectura hexagonal (dependencias hacia dentro)
- [ ] El flujo de búsqueda con fallback funciona correctamente

## Estructura de Paquetes Final

```
com.wikicollection/
├── domain/
│   └── model/
│       ├── Game
│       ├── GameStatus
│       ├── GamePlatform
│       └── GameSearchResult
│   └── port/
│       ├── in/
│       │   ├── GameUseCase
│       │   └── GameSearchUseCase
│       └── out/
│           ├── GameRepository
│           └── ExternalGameCatalogClient
├── application/
│   └── service/
│       ├── GameService
│       └── GameSearchService
├── infrastructure/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   ├── GameController
│   │   │   └── dto/
│   │   │       ├── GameRequest
│   │   │       ├── GameResponse
│   │   │       └── GameDtoMapper
│   │   └── out/
│   │       ├── rawg/
│   │       │   └── RAWGClient
│   │       ├── freetogame/
│   │       │   └── FreeToGameClient
│   │       └── persistence/
│   │           ├── GameEntity
│   │           ├── SpringDataGameRepository
│   │           ├── GameEntityMapper
│   │           └── GamePersistenceAdapter
│   └── config/
│       ├── StringToGameStatusConverter
│       └── RestClientConfig (ya existente)
```

## Configuración en application.properties

```properties
# RAWG API Key (obligatorio para búsqueda por nombre)
rawg.api.key=YOUR_RAWG_API_KEY

# FreeToGame no requiere configuración
```

## Notas

- **RAWG es la API principal** — Búsqueda por nombre nativa, 500k+ juegos, 100k req/mes gratis
- **FreeToGame es la API secundaria** — Fallback sin auth, ~415 juegos, sin búsqueda por nombre
- La búsqueda se hace por título con el query param `name` (consistente con Books)
- Se recomienda cachear la lista de juegos (TTL 1 hora) para evitar llamadas repetidas
- Si en el futuro se quiere una API más completa, se puede añadir IGDB como tercer `ExternalGameCatalogClient`
- Documentación RAWG: https://api.rawg.io/docs/
- Documentación FreeToGame: https://www.freetogame.com/api-doc
