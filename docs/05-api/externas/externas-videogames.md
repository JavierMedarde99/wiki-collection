# APIs Externas — Videojuegos

## FreeToGame API

- **Base URL:** `https://www.freetogame.com/api`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí, completamente
- **Total juegos:** ~415 (PC + Web Browser)
- **Documentación:** https://www.freetogame.com/api-doc

### Endpoints

| Uso | Endpoint |
|-----|----------|
| Todos los juegos | `GET /api/games` |
| Filtrar por plataforma | `GET /api/games?platform=windows` |
| Filtrar por categoría | `GET /api/games?category=shooter` |
| Ordenar por fecha | `GET /api/games?sort-by=release-date` |
| Ordenar por popularidad | `GET /api/games?sort-by=popularity` |
| Ordenar alfabéticamente | `GET /api/games?sort-by=alphabetical` |
| Detalle de juego | `GET /api/game?id={game_id}` |
| Filtrar por tags | `GET /api/filter?tag=3d.mmorpg.fantasy` |
| Filtrar + plataforma + orden | `GET /api/filter?tag=mmorpg&platform=windows&sort-by=release-date` |

### Valores de plataforma

| Valor | Descripción |
|-------|-------------|
| `windows` | PC (Windows) |
| `browser` | Navegador web |
| `all` | Todas las plataformas |

### Valores de categoría

| Valor | Descripción |
|-------|-------------|
| `mmorpg` | MMORPG |
| `shooter` | Shooter |
| `strategy` | Estrategia |
| `moba` | MOBA |
| `racing` | Carreras |
| `sports` | Deportes |
| `social` | Social |
| `sandbox` | Sandbox |

### Valores de tag

```
mmorpg, shooter, strategy, moba, racing, sports, social, sandbox,
open-world, survival, pvp, pve, pixel, voxel, zombie, turn-based,
first-person, third-person, top-down, tank, space, sailing,
side-scroller, superhero, permadeath, card, battle-royale, mmo,
mmofps, mmotps, 3d, 2d, anime, fantasy, sci-fi, fighting,
action-rpg, action, military, martial-arts, flight, low-spec,
tower-defense, horror, mmorts
```

### Ejemplo de Request

```http
GET https://www.freetogame.com/api/games?platform=windows&category=shooter&sort-by=alphabetical
```

### Ejemplo de Respuesta

```json
[
  {
    "id": 540,
    "title": "Overwatch",
    "thumbnail": "https://www.freetogame.com/g/540/thumbnail.jpg",
    "short_description": "A hero-focused first-person team shooter from Blizzard Entertainment.",
    "game_url": "https://www.freetogame.com/open/overwatch",
    "genre": "Shooter",
    "platform": "PC (Windows)",
    "publisher": "Activision Blizzard",
    "developer": "Blizzard Entertainment",
    "release_date": "2022-10-04",
    "freetogame_profile_url": "https://www.freetogame.com/overwatch"
  },
  {
    "id": 516,
    "title": "PUBG: BATTLEGROUNDS",
    "thumbnail": "https://www.freetogame.com/g/516/thumbnail.jpg",
    "short_description": "Get into the action in one of the longest running battle royale games.",
    "game_url": "https://www.freetogame.com/open/pubg",
    "genre": "Shooter",
    "platform": "PC (Windows)",
    "publisher": "KRAFTON, Inc.",
    "developer": "KRAFTON, Inc.",
    "release_date": "2022-01-12",
    "freetogame_profile_url": "https://www.freetogame.com/pubg"
  }
]
```

### Campos de respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Integer | ID único del juego |
| `title` | String | Título |
| `thumbnail` | String | URL del thumbnail |
| `short_description` | String | Descripción breve |
| `game_url` | String | URL para jugar en FreeToGame |
| `genre` | String | Género |
| `platform` | String | Plataforma |
| `publisher` | String | Publicador |
| `developer` | String | Desarrollador |
| `release_date` | String | Fecha de lanzamiento (YYYY-MM-DD) |
| `freetogame_profile_url` | String | URL del perfil en FreeToGame |

---

## ⚠️ Limitación Importante: Búsqueda por Nombre

**FreeToGame NO soporta búsqueda por nombre.** Los parámetros `?title=`, `?name=` o `?search=` no funcionan y devuelven todos los juegos.

### Solución: Filtrado en Backend

El backend descarga todos los juegos y filtra por título en memoria usando Java Streams.

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
        
        // Obtener todos los juegos (cacheados)
        if (cachedGames == null || isCacheExpired()) {
            cachedGames = externalClient.getAllGames();
            lastFetch = Instant.now();
        }
        
        // Filtrar por título en memoria (case-insensitive, partial match)
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

## Mapeo de Plataformas

| Valor API | Valor Interno (GamePlatform) |
|-----------|------------------------------|
| `PC (Windows)` | `PC` |
| `Web Browser` | `WEB_BROWSER` |
| `PC (Windows), Web Browser` | `BOTH` |

---

## Mapeo a Modelo Interno

| FreeToGame | Interno (Game) |
|------------|----------------|
| `id` | `externalId` |
| `title` | `title` |
| `short_description` | `description` |
| `genre` | `genre` |
| `platform` | `platform` (mapeado) |
| `publisher` | `publisher` |
| `developer` | `developer` |
| `release_date` | `releaseDate` |
| `thumbnail` | `thumbnailUrl` |

---

## Implementación en el Backend

### FreeToGameClient (Spring Boot)

```java
@Component
public class FreeToGameClient implements ExternalGameCatalogClient {
    
    private final RestClient restClient;
    
    public FreeToGameClient(RestClient.Builder builder) {
        this.restClient = builder
            .baseUrl("https://www.freetogame.com/api")
            .build();
    }
    
    @Override
    public List<GameSearchResult> search(String query) {
        // FreeToGame no soporta búsqueda por nombre
        // Se obtiene todo y se filtra en memoria
        return getAllGames().stream()
            .filter(g -> g.title().toLowerCase().contains(query.toLowerCase()))
            .toList();
    }
    
    @Override
    public List<GameSearchResult> getAllGames() {
        try {
            var games = restClient.get()
                .uri("/games")
                .retrieve()
                .body(FreeToGameDTO[].class);
            
            return Arrays.stream(games)
                .map(this::toSearchResult)
                .toList();
                
        } catch (Exception e) {
            // Graceful degradation
            return List.of();
        }
    }
    
    private GameSearchResult toSearchResult(FreeToGameDTO dto) {
        return new GameSearchResult(
            String.valueOf(dto.id()),
            dto.title(),
            dto.shortDescription(),
            dto.genre(),
            mapPlatform(dto.platform()),
            dto.publisher(),
            dto.developer(),
            dto.releaseDate(),
            dto.thumbnail()
        );
    }
    
    private GamePlatform mapPlatform(String platform) {
        return switch (platform) {
            case "PC (Windows)" -> GamePlatform.PC;
            case "Web Browser" -> GamePlatform.WEB_BROWSER;
            default -> GamePlatform.BOTH;
        };
    }
}

public record FreeToGameDTO(
    int id,
    String title,
    String thumbnail,
    String shortDescription,
    String gameUrl,
    String genre,
    String platform,
    String publisher,
    String developer,
    String releaseDate,
    String freetogameProfileUrl
) {}
```

---

## Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| 404 | ID de juego no existe | Verificar ID |
| 429 | Rate limit (10/s) | Añadir delay entre requests |
| Timeout | Red lenta | Configurar timeout de 5s |
| Vacío | Juego eliminado | Verificar que el juego sigue existiendo |

---

## Futuras Alternativas

Si en el futuro se necesita más capacidad de búsqueda o un catálogo más amplio:

### RAWG (requiere registro gratuito)

- **Base URL:** `https://api.rawg.io/api`
- **Auth:** API Key (gratuita con registro)
- **Búsqueda por nombre:** ✅ Sí
- **Total juegos:** 500,000+
- **Documentación:** https://rawg.io/apidocs

### IGDB (requiere cuenta Twitch)

- **Base URL:** `https://api.igdb.com/v4`
- **Auth:** Client ID + Access Token
- **Búsqueda por nombre:** ✅ Sí
- **Total juegos:** 200,000+
- **Documentación:** https://api-docs.igdb.com
