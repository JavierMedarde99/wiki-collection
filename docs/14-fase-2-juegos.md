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
4. Si no hay resultados → FreeToGame API (search)
5. Mapear y devolver
```

---

## Estado: ✅ COMPLETADA

Todos los pasos fueron implementados y verificados:

- [x] Crear enum `GameStatus`: PLAYING, COMPLETED, WISHLIST, ABANDONED
- [x] Crear enum `GamePlatform`: PC, PS2, PS3, WII_U, SWITCH
- [x] Crear documento `Game.java` en `domain/model/`
- [x] Crear interface `GameRepository.java` en `domain/port/out/`
- [x] Crear interface `ExternalGameCatalogClient.java` en `domain/port/out/`
- [x] Crear `SpringDataGameRepository.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `GameEntity.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `GameEntityMapper.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `GamePersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear interface `GameUseCase.java` en `domain/port/in/`
- [x] Crear interface `GameSearchUseCase.java` en `domain/port/in/`
- [x] Crear `GameService.java` en `application/service/`
- [x] Crear `GameSearchService.java` en `application/service/`
- [x] Crear `RAWGClient.java` en `infrastructure/adapter/out/rawg/`
- [x] Crear `FreeToGameClient.java` en `infrastructure/adapter/out/freetogame/`
- [x] Crear `GameController.java` en `infrastructure/adapter/in/web/`
- [x] Crear `GameRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `GameResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `GameDtoMapper.java` en `infrastructure/adapter/in/web/dto/`
- [x] Crear `StringToGameStatusConverter.java` en `infrastructure/config/`
- [x] `GameServiceTest.java` — Tests unitarios de CRUD
- [x] `GameControllerTest.java` — Tests de integración MockMvc
- [x] `RAWGClientTest.java` — Tests del cliente RAWG con mock server
- [x] `FreeToGameClientTest.java` — Tests del cliente FreeToGame con mock server
- [x] `GamePersistenceAdapterTest.java` — Tests del adaptador de persistencia
- [x] `GameSearchServiceTest.java` — Tests del flujo de búsqueda con fallback

## Criterios de Aceptación

- [x] El endpoint `GET /api/games/search?name=overwatch` devuelve resultados de RAWG
- [x] Si RAWG no devuelve resultados, el endpoint usa FreeToGame como fallback
- [x] Los resultados incluyen: título, género, plataforma, publisher, developer, thumbnail, descripción
- [x] El endpoint `GET /api/games` devuelve lista vacía al inicio
- [x] Se puede crear un juego vía `POST /api/games`
- [x] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/games/{id}`
- [x] Los tests pasan (`mvn verify`)
- [x] El respeta arquitectura hexagonal (dependencias hacia dentro)
- [x] El flujo de búsqueda con fallback funciona correctamente

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
rawg.api-key=${RAWG_API_KEY:}

# Google Books API Key
google.books.api-key=${GOOGLE_BOOKS_API_KEY:}

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
