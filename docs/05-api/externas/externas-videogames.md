# APIs Externas — Videojuegos

## RAWG Video Games Database API (PRINCIPAL)

- **Base URL:** `https://api.rawg.io/api`
- **Auth:** API Key (gratuita con registro en https://rawg.io/apidocs)
- **Rate limit:** 100,000 requests/mes (free tier)
- **Gratis:** Sí, con registro gratuito
- **Total juegos:** 500,000+
- **Documentación:** https://api.rawg.io/docs/
- **Cobertura:** 50 plataformas incluyendo móviles

### Endpoints útiles

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /api/games?key={key}&search={query}` |
| Detalle de juego | `GET /api/games/{id}?key={key}` |
| Juegos por plataforma | `GET /api/games?key={key}&platforms={ids}` |
| Juegos por género | `GET /api/games?key={key}&genres={slug}` |
| Juegos por fecha | `GET /api/games?key={key}&dates={from,to}` |
| Juegos populares | `GET /api/games?key={key}&ordering=-rating` |
| Screenshots de juego | `GET /api/games/{id}/screenshots?key={key}` |
| Trailers de juego | `GET /api/games/{id}/movies?key={key}` |
| Plataformas | `GET /api/platforms?key={key}` |
| Géneros | `GET /api/genres?key={key}` |
| Tags | `GET /api/tags?key={key}` |
| Desarrolladores | `GET /api/developers?key={key}` |
| Publicadores | `GET /api/publishers?key={key}` |
| Tiendas | `GET /api/stores?key={key}` |

### Ejemplo de Request

```http
GET https://api.rawg.io/api/games?key=YOUR_API_KEY&search=overwatch&page_size=5
```

### Ejemplo de Respuesta

```json
{
  "count": 1234,
  "next": "https://api.rawg.io/api/games?key=YOUR_API_KEY&page=2&search=overwatch",
  "previous": null,
  "results": [
    {
      "id": 459414,
      "slug": "overwatch",
      "name": "Overwatch",
      "released": "2016-05-24",
      "background_image": "https://media.rawg.io/media/games/6c5/6c55dfa72c7317eab3a8e8a8c7e8f8c7.jpg",
      "rating": 4.05,
      "ratings_count": 1234,
      "playtime": 6,
      "platforms": [
        {
          "platform": {
            "id": 4,
            "name": "PC",
            "slug": "pc"
          }
        },
        {
          "platform": {
            "id": 18,
            "name": "PlayStation 4",
            "slug": "playstation4"
          }
        }
      ],
      "genres": [
        {
          "id": 4,
          "name": "Action",
          "slug": "action"
        },
        {
          "id": 5,
          "name": "Shooter",
          "slug": "shooter"
        }
      ],
      "stores": [
        {
          "store": {
            "id": 1,
            "name": "Steam",
            "slug": "steam"
          }
        }
      ],
      "tags": [
        {
          "id": 31,
          "name": "Singleplayer",
          "slug": "singleplayer"
        }
      ],
      "short_screenshots": [
        {
          "id": 123,
          "image": "https://media.rawg.io/media/screenshots/..."
        }
      ]
    }
  ]
}
```

### Campos de respuesta relevantes

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `count` | Integer | Total de resultados |
| `next` | String | URL de la siguiente página |
| `previous` | String | URL de la página anterior |
| `results[].id` | Integer | ID único del juego |
| `results[].slug` | String | Slug URL-friendly |
| `results[].name` | String | Título del juego |
| `results[].released` | String | Fecha de lanzamiento (YYYY-MM-DD) |
| `results[].background_image` | String | URL de imagen de fondo |
| `results[].rating` | Float | Rating (0-5) |
| `results[].ratings_count` | Integer | Número de valoraciones |
| `results[].playtime` | Integer | Horas de juego estimadas |
| `results[].platforms[]` | List | Plataformas disponibles |
| `results[].platforms[].platform.id` | Integer | ID de plataforma |
| `results[].platforms[].platform.name` | String | Nombre de plataforma |
| `results[].platforms[].platform.slug` | String | Slug de plataforma |
| `results[].genres[]` | List | Géneros del juego |
| `results[].genres[].id` | Integer | ID de género |
| `results[].genres[].name` | String | Nombre de género |
| `results[].genres[].slug` | String | Slug de género |
| `results[].stores[]` | List | Tiendas disponibles |
| `results[].stores[].store.id` | Integer | ID de tienda |
| `results[].stores[].store.name` | String | Nombre de tienda |
| `results[].tags[]` | List | Tags del juego |
| `results[].short_screenshots[]` | List | Screenshots del juego |

### IDs de Plataformas Comunes

| ID | Plataforma |
|----|------------|
| 1 | Xbox One |
| 2 | PlayStation 4 |
| 3 | iOS |
| 4 | PC |
| 5 | macOS |
| 6 | Linux |
| 7 | Nintendo Switch |
| 8 | Android |
| 9 | Nintendo 3DS |
| 10 | Nintendo DS |
| 11 | Wii U |
| 12 | Wii |
| 13 | PlayStation 3 |
| 14 | Xbox 360 |
| 15 | PlayStation 2 |
| 16 | PlayStation |
| 17 | GameCube |
| 18 | Nintendo 64 |
| 19 | Game Boy Advance |
| 20 | Game Boy Color |
| 21 | Game Boy |
| 22 | SNES |
| 23 | NES |
| 24 | Sega Genesis |
| 25 | Sega Saturn |
| 26 | Sega Dreamcast |
| 27 | PlayStation Portable |
| 28 | PlayStation Vita |
| 29 | PlayStation 5 |
| 30 | Xbox Series S/X |

### IDs de Géneros Comunes

| ID | Género |
|----|--------|
| 4 | Action |
| 5 | Shooter |
| 3 | Adventure |
| 51 | Indie |
| 7 | RPG |
| 2 | Strategy |
| 10 | Racing |
| 14 | Simulation |
| 15 | Sports |
| 6 | Fighting |
| 1 | Racing |
| 17 | Card |
| 19 | Family |
| 83 | Platformer |
| 11 | Arcade |
| 59 | Massively Multiplayer |
| 83 | Board Games |

### Mapeo a Modelo Interno

| RAWG | Interno (Game) |
|------|----------------|
| `id` | `externalId` |
| `name` | `title` |
| `released` | `releaseDate` |
| `background_image` | `thumbnailUrl` |
| `genres[].name` | `genre` |
| `platforms[].platform.name` | `platform` |
| `rating` | `userRating` (mapeado a 1-5) |

### Ordenación

| Valor | Descripción |
|-------|-------------|
| `name` | Alfabético ascendente |
| `-name` | Alfabético descendente |
| `released` | Fecha ascendente |
| `-released` | Fecha descendente |
| `rating` | Rating ascendente |
| `-rating` | Rating descendente |
| `metacritic` | Metacritic ascendente |
| `-metacritic` | Metacritic descendente |
| `added` | Añadido ascendente |
| `-added` | Añadido descendente |

### Filtros Avanzados

```
# Juegos de 2023 con rating > 4
GET /api/games?key={key}&dates=2023-01-01,2023-12-31&ordering=-rating&metacritic=80,100

# Juegos de acción para PC
GET /api/games?key={key}&genres=action&platforms=4

# Juegos populares de Nintendo Switch
GET /api/games?key={key}&platforms=7&ordering=-rating&page_size=20
```

---

## FreeToGame API (SECUNDARIA)

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

## ⚠️ Limitación Importante: Búsqueda por Nombre en FreeToGame

**FreeToGame NO soporta búsqueda por nombre.** Los parámetros `?title=`, `?name=` o `?search=` no funcionan y devuelven todos los juegos.

### Solución: RAWG como Principal

RAWG sí soporta búsqueda por nombre (`?search={query}`), por lo que se usa como API principal. FreeToGame se usa como fallback para juegos específicos o cuando RAWG no tiene cobertura.

---

## Mapeo de Plataformas (FreeToGame)

| Valor API | Valor Interno (GamePlatform) |
|-----------|------------------------------|
| `PC (Windows)` | `PC` |
| `Web Browser` | `WEB_BROWSER` |
| `PC (Windows), Web Browser` | `BOTH` |

---

## Mapeo a Modelo Interno (FreeToGame)

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

## Implementación en el Backend

### RAWGClient (Spring Boot)

```java
@Component
public class RAWGClient implements ExternalGameCatalogClient {
    
    private final RestClient restClient;
    private final String apiKey;
    
    public RAWGClient(RestClient.Builder builder, 
                      @Value("${rawg.api.key}") String apiKey) {
        this.restClient = builder
            .baseUrl("https://api.rawg.io/api")
            .build();
        this.apiKey = apiKey;
    }
    
    @Override
    public List<GameSearchResult> search(String query) {
        try {
            var response = restClient.get()
                .uri(uriBuilder -> uriBuilder
                    .path("/games")
                    .queryParam("key", apiKey)
                    .queryParam("search", query)
                    .queryParam("page_size", 10)
                    .build())
                .retrieve()
                .body(RAWGResponse.class);
            
            return response.results().stream()
                .map(this::toSearchResult)
                .toList();
                
        } catch (Exception e) {
            // Graceful degradation
            return List.of();
        }
    }
    
    @Override
    public List<GameSearchResult> getAllGames() {
        // RAWG no tiene endpoint para todos los juegos
        // Se usa paginación si es necesario
        return List.of();
    }
    
    private GameSearchResult toSearchResult(RAWGGame dto) {
        return new GameSearchResult(
            String.valueOf(dto.id()),
            dto.name(),
            dto.released(),
            dto.backgroundImage(),
            dto.genres().stream().map(RAWGGenre::name).findFirst().orElse(""),
            dto.platforms().stream().map(p -> p.platform().name()).toList(),
            dto.rating()
        );
    }
}

public record RAWGResponse(
    int count,
    String next,
    String previous,
    List<RAWGGame> results
) {}

public record RAWGGame(
    int id,
    String slug,
    String name,
    String released,
    String backgroundImage,
    double rating,
    List<RAWGGenre> genres,
    List<RAWGPlatformWrapper> platforms
) {}

public record RAWGGenre(int id, String name, String slug) {}

public record RAWGPlatformWrapper(RAWGPlatform platform) {}

public record RAWGPlatform(int id, String name, String slug) {}
```

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

### GameSearchService (Spring Boot)

```java
@Service
public class GameSearchService implements GameSearchUseCase {
    
    private final RAWGClient rawgClient;
    private final FreeToGameClient freeToGameClient;
    
    public GameSearchService(RAWGClient rawgClient, 
                             FreeToGameClient freeToGameClient) {
        this.rawgClient = rawgClient;
        this.freeToGameClient = freeToGameClient;
    }
    
    @Override
    public List<GameSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            return List.of();
        }
        
        // 1. Intentar RAWG (principal)
        List<GameSearchResult> results = rawgClient.search(query);
        
        // 2. Si no hay resultados, intentar FreeToGame (secundaria)
        if (results.isEmpty()) {
            results = freeToGameClient.search(query);
        }
        
        return results;
    }
}
```

---

## Comparativa

| Característica | RAWG | FreeToGame |
|----------------|------|------------|
| Auth | API Key (gratis) | No |
| Búsqueda por nombre | ✅ | ❌ (filtrado en backend) |
| Total juegos | 500,000+ | ~415 |
| Rate limit | 100k/mes | ~10/s |
| Datos | Muy completos | Completos |
| Plataformas | 50+ | 2 (PC, Browser) |
| Screenshots | ✅ | ❌ |
| Trailers | ✅ | ❌ |
| Géneros | ✅ | ✅ |
| Tags | ✅ | ✅ |
| Gratis | ✅ (con registro) | ✅ (sin registro) |
| API Key | Necesaria | No necesaria |

---

## Errores Comunes

### RAWG

| Error | Causa | Solución |
|-------|-------|----------|
| 401 | API key inválida | Verificar key en rawg.io |
| 404 | Juego no encontrado | Verificar ID |
| 429 | Rate limit exponencial | Backoff retry |
| 500 | Error del servidor | Reintentar |

### FreeToGame

| Error | Causa | Solución |
|-------|-------|----------|
| 404 | ID de juego no existe | Verificar ID |
| 429 | Rate limit (10/s) | Añadir delay entre requests |
| Timeout | Red lenta | Configurar timeout de 5s |
| Vacío | Juego eliminado | Verificar que el juego sigue existiendo |

---

## Configuración en application.properties

```properties
# RAWG API Key (obligatorio para búsqueda por nombre)
rawg.api.key=YOUR_RAWG_API_KEY

# FreeToGame no requiere configuración
```

---

## Futuras Alternativas

Si en el futuro se necesita más capacidad o diferentes datos:

### IGDB (requiere cuenta Twitch)

- **Base URL:** `https://api.igdb.com/v4`
- **Auth:** Client ID + Access Token
- **Búsqueda por nombre:** ✅ Sí
- **Total juegos:** 200,000+
- **Documentación:** https://api-docs.igdb.com

### Epic Games Store

- **Base URL:** `https://store-site-backend-static.ak.epicgames.com`
- **Auth:** No requerida
- **Búsqueda por nombre:** ❌ No (solo free games)
- **Gratis:** Sí
- **Documentación:** No oficial
