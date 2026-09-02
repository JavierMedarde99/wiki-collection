# Fase 2: Colección de Videojuegos

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 (arquitectura hexagonal) para:

1. Hacer llamadas a la API externa FreeToGame para buscar videojuegos
2. Devolver la información de videojuegos en el endpoint `GET /api/games/search`
3. CRUD completo de videojuegos en la colección local (MongoDB)

---

## API Externa: FreeToGame

**URL Base:** `https://www.freetogame.com/api`

**Documentación:** https://www.freetogame.com/api-doc

**Endpoints:**

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/games` | Listar todos los juegos |
| GET | `/api/games?platform=windows` | Filtrar por plataforma |
| GET | `/api/games?category=shooter` | Filtrar por categoría/género |
| GET | `/api/games?sort-by=alphabetical` | Ordenar alfabéticamente |
| GET | `/api/game?id=540` | Detalle de un juego |
| GET | `/api/filter?tag=3d.mmorpg.fantasy` | Filtrar por tags |

**Ventajas:**
- 100% gratuita, sin API key
- Sin autenticación
- ~415 juegos (PC + Web Browser)
- Datos: título, thumbnail, descripción, género, plataforma, publisher, developer, fecha lanzamiento

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

## ⚠️ Limitación Importante: Búsqueda por Nombre

**FreeToGame NO soporta búsqueda por nombre.** Los parámetros `?title=`, `?name=` o `?search=` no funcionan y devuelven todos los juegos.

**Solución implementada:** El backend descarga todos los juegos y filtra por título en memoria usando Java Streams.

### Implementación en el Backend

```java
@Service
public class GameSearchService implements GameSearchUseCase {
    
    private final ExternalGameCatalogClient externalClient;
    private List<GameSearchResult> cachedGames;
    
    @Override
    public List<GameSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            return List.of();
        }
        
        // Obtener todos los juegos (cacheados)
        List<GameSearchResult> allGames = cachedGames != null 
            ? cachedGames 
            : externalClient.getAllGames();
        cachedGames = allGames;
        
        // Filtrar por título en memoria
        String lowerQuery = query.toLowerCase();
        return allGames.stream()
            .filter(g -> g.title().toLowerCase().contains(lowerQuery))
            .toList();
    }
}
```

### Alternativa con Cacheo (Recomendado)

```java
@Service
public class GameSearchService implements GameSearchUseCase {
    
    private final ExternalGameCatalogClient externalClient;
    private List<GameSearchResult> cachedGames;
    private Instant lastFetch;
    private static final Duration CACHE_TTL = Duration.ofHours(1);
    
    @Override
    public List<GameSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            return List.of();
        }
        
        if (cachedGames == null || isCacheExpired()) {
            cachedGames = externalClient.getAllGames();
            lastFetch = Instant.now();
        }
        
        String lowerQuery = query.toLowerCase();
        return cachedGames.stream()
            .filter(g -> g.title().toLowerCase().contains(lowerQuery))
            .toList();
    }
    
    private boolean isCacheExpired() {
        return lastFetch == null || 
               Duration.between(lastFetch, Instant.now()).compareTo(CACHE_TTL) > 0;
    }
}
```

---

## Paso 1: Modelo de Datos — Game (Dominio)

- [ ] Crear enum `GameStatus`: PLAYING, COMPLETED, WISHLIST, ABANDONED
- [ ] Crear enum `GamePlatform`: PC, WEB_BROWSER, BOTH (mapeo de plataformas FreeToGame)
- [ ] Crear documento `Game.java` en `domain/model/`:
  - Anotación `@Document(collection = "GAMES")`
  - Campos: id, externalId (FreeToGame ID), title, description, genre, platform, publisher, developer, releaseDate, thumbnailUrl, status, userRating (1-5), notes, dateAdded, dateCompleted, externalSource

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
  - Obtiene todos los juegos de FreeToGame
  - **Filtra por título en memoria** (Java Streams)
  - Cachea resultados (TTL 1 hora)
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
  - `GET /api/games/search?name={query}` — buscar en FreeToGame (filtrado en backend)
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
- [ ] `FreeToGameClientTest.java` — Tests del cliente externo con mock server
- [ ] `GamePersistenceAdapterTest.java` — Tests del adaptador de persistencia
- [ ] `GameSearchServiceTest.java` — Tests del filtrado por nombre

## Criterios de Aceptación

- [ ] El endpoint `GET /api/games/search?name=overwatch` devuelve resultados de FreeToGame filtrados por nombre
- [ ] Los resultados incluyen: título, género, plataforma, publisher, developer, thumbnail, descripción
- [ ] El endpoint `GET /api/games` devuelve lista vacía al inicio
- [ ] Se puede crear un juego vía `POST /api/games`
- [ ] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/games/{id}`
- [ ] Los tests pasan (`mvn verify`)
- [ ] El respeta arquitectura hexagonal (dependencias hacia dentro)
- [ ] El filtrado por nombre funciona correctamente (case-insensitive, partial match)

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

## Notas

- FreeToGame no requiere API key ni auth, ideal para desarrollo
- La búsqueda se hace por título con el query param `name` (consistente con Books)
- ~400 juegos en PC + Web Browser, suficiente para demo
- **FreeToGame NO soporta búsqueda por nombre** — el filtrado se hace en el backend
- Se recomienda cachear la lista de juegos (TTL 1 hora) para evitar llamadas repetidas
- Si en el futuro se quiere una API más completa, se puede añadir RAWG/IGDB como segundo `ExternalGameCatalogClient`
