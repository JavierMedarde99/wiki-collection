# APIs Externas — Steam (Logros de Videojuegos)

## Steam Web API (ISteamUserStats)

- **Base URL:** `https://api.steampowered.com/ISteamUserStats/`
- **Auth:** API Key (gratuita en https://steamcommunity.com/dev/apikey)
- **Rate limit:** ~200 requests/5 minutos (sin key), ~100,000/día (con key)
- **Gratis:** Sí, con registro gratuito
- **Documentación:** https://developer.valvesoftware.com/wiki/Steam_Web_API
- **Cobertura:** Todos los juegos de Steam con logros

### Endpoints de Logros

| Uso | Endpoint | Auth |
|-----|----------|------|
| Buscar juego por nombre (obtener AppID) | `GET https://store.steampowered.com/api/storesearch/?term={nombre}&l={lang}&cc={country}` | No |
| Porcentajes globales de logros | `GET /GetGlobalAchievementPercentagesForApp/v0002/?gameid={appid}` | No |
| Esquema de logros (nombres, descripciones, iconos) | `GET /GetSchemaForGame/v2/?key={key}&appid={appid}` | Sí |
| Logros de un jugador específico | `GET /GetPlayerAchievements/v1/?key={key}&steamid={steamid}&appid={appid}` | Sí |

### Ejemplo: Buscar Juego por Nombre (sin API key)

```http
GET https://store.steampowered.com/api/storesearch/?term=witcher&l=english&cc=US
```

**Respuesta:**
```json
{
  "total": 10,
  "items": [
    {
      "type": "app",
      "name": "The Witcher 3: Wild Hunt — Songs of the Past",
      "id": 5006530,
      "tiny_image": "https://shared.akamai.steamstatic.com/store_item_assets/steam/apps/5006530/...",
      "metascore": "",
      "platforms": {
        "windows": true,
        "mac": false,
        "linux": false
      },
      "streamingvideo": false,
      "controller_support": "full"
    },
    {
      "type": "app",
      "name": "The Witcher 3: Wild Hunt - Complete Edition",
      "id": 292030,
      "price": {
        "currency": "USD",
        "initial": 4999,
        "final": 4999
      },
      "tiny_image": "https://shared.akamai.steamstatic.com/store_item_assets/steam/apps/292030/...",
      "metascore": "93",
      "platforms": {
        "windows": true,
        "mac": false,
        "linux": false
      },
      "streamingvideo": false
    }
  ]
}
```

### Ejemplo: Porcentajes Globales (sin API key)

```http
GET https://api.steampowered.com/ISteamUserStats/GetGlobalAchievementPercentagesForApp/v0002/?gameid=292030&format=json
```

**Respuesta:**
```json
{
  "achievementpercentages": {
    "achievements": [
      {
        "name": "LILAC",
        "percent": "60.1"
      },
      {
        "name": "BUTCHER_OF_BLAVIKEN",
        "percent": "50.9"
      },
      {
        "name": "FIST_OF_THE_SOUTH_STAR",
        "percent": "49.3"
      }
    ]
  }
}
```

### Ejemplo: Esquema de Logros (con API key)

```http
GET https://api.steampowered.com/ISteamUserStats/GetSchemaForGame/v2/?key=YOUR_STEAM_KEY&appid=292030
```

**Respuesta (parcial):**
```json
{
  "game": {
    "gameName": "The Witcher 3: Wild Hunt",
    "gameVersion": "1",
    "availableGameStats": {
      "achievements": [
        {
          "name": "LILAC",
          "defaultvalue": 0,
          "displayName": "Lilac and Gooseberries",
          "hidden": 0,
          "description": "Visit the Baron's family cemetery.",
          "icon": "https://steamcdn-a.akamaihd.net/steamcommunity/public/images/apps/292030/...",
          "icongray": "https://steamcdn-a.akamaihd.net/steamcommunity/public/images/apps/292030/..."
        }
      ]
    }
  }
}
```

### Ejemplo: Logros de Jugador (con API key)

```http
GET https://api.steampowered.com/ISteamUserStats/GetPlayerAchievements/v1/?key=YOUR_STEAM_KEY&steamid=76561198000000001&appid=292030
```

**Respuesta (parcial):**
```json
{
  "playerstats": {
    "steamID": "76561198000000001",
    "gameName": "The Witcher 3: Wild Hunt",
    "achievements": [
      {
        "apiname": "LILAC",
        "achieved": 1,
        "unlocktime": 1458316800
      },
      {
        "apiname": "BUTCHER_OF_BLAVIKEN",
        "achieved": 0,
        "unlocktime": 0
      }
    ],
    "success": true
  }
}
```

### Campos de Respuesta

#### storesearch (Buscar Juego por Nombre)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `total` | Integer | Total de resultados encontrados |
| `items[].type` | String | Tipo de resultado (app, dlc, bundle, etc.) |
| `items[].id` | Integer | AppID del juego (este es el `gameid` para otros endpoints) |
| `items[].name` | String | Nombre del juego |
| `items[].tiny_image` | String | URL de la imagen miniatura |
| `items[].metascore` | String | Puntuación Metascore (puede estar vacía) |
| `items[].platforms` | Object | Disponibilidad por plataforma (windows, mac, linux) |
| `items[].price` | Object | Información de precio (currency, initial, final) |
| `items[].controller_support` | String | Soporte de controller (full, partial, none) |

#### GetGlobalAchievementPercentagesForApp

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `achievementpercentages.achievements[].name` | String | Nombre interno del logro (API name) |
| `achievementpercentages.achievements[].percent` | String | Porcentaje de jugadores que lo desbloquearon (0-100) |

#### GetSchemaForGame

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `game.gameName` | String | Nombre del juego |
| `game.gameVersion` | String | Versión del esquema |
| `game.availableGameStats.achievements[].name` | String | Nombre interno del logro |
| `game.availableGameStats.achievements[].displayName` | String | Nombre visible del logro |
| `game.availableGameStats.achievements[].description` | String | Descripción del logro |
| `game.availableGameStats.achievements[].hidden` | Integer | 1 si está oculto, 0 si no |
| `game.availableGameStats.achievements[].icon` | String | URL del icono (desbloqueado) |
| `game.availableGameStats.achievements[].icongray` | String | URL del icono (bloqueado) |

#### GetPlayerAchievements

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `playerstats.steamID` | String | SteamID64 del jugador |
| `playerstats.gameName` | String | Nombre del juego |
| `playerstats.achievements[].apiname` | String | Nombre interno del logro |
| `playerstats.achievements[].achieved` | Integer | 1 si está desbloqueado, 0 si no |
| `playerstats.achievements[].unlocktime` | Integer | Unix timestamp del desbloqueo (0 si no) |
| `playerstats.success` | Boolean | true si la consulta fue exitosa |

### IDs de Juegos Steam (AppIDs) Comunes

| AppID | Juego |
|-------|-------|
| 730 | Counter-Strike 2 |
| 570 | Dota 2 |
| 440 | Team Fortress 2 |
| 220 | Half-Life 2 |
| 292030 | The Witcher 3: Wild Hunt |
| 1086940 | Baldur's Gate 3 |
| 1245620 | ELDEN RING |
| 1091500 | Cyberpunk 2077 |
| 374320 | DARK SOULS III |
| 582010 | MONSTER HUNTER: WORLD |

### Mapeo a Modelo Interno

| Steam | Interno (GameAchievement) |
|-------|---------------------------|
| `name` / `apiname` | `achievementId` |
| `displayName` | `title` |
| `description` | `description` |
| `icon` | `iconUrl` |
| `percent` (global) | `globalCompletionRate` |
| `achieved` (jugador) | `unlocked` |
| `unlocktime` | `unlockedAt` |

### Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| 401 | API key inválida | Verificar key en steamcommunity.com/dev/apikey |
| 403 | Acceso denegado | Verificar key o permisos |
| 404 | Juego no encontrado | Verificar AppID |
| 429 | Rate limit excedido | Implementar backoff retry |
| 500 | Error del servidor | Reintentar |
| Vacío | Juego sin logros | Verificar que el juego tenga logros |

### Configuración en application.properties

```properties
# Steam API Key (obtener en https://steamcommunity.com/dev/apikey)
# Necesario para GetSchemaForGame y GetPlayerAchievements
# No necesario para GetGlobalAchievementPercentagesForApp ni storesearch
steam.api.key=YOUR_STEAM_API_KEY
```

### Estrategia de Implementación

1. **storesearch** — Sin auth, buscar juego por nombre para obtener AppID
2. **GetGlobalAchievementPercentagesForApp** — Sin auth, ideal para mostrar estadísticas globales de logros
3. **GetSchemaForGame** — Con auth, para obtener nombres, descripciones e iconos de logros
4. **GetPlayerAchievements** — Con auth, para mostrar el progreso de un jugador específico

### Flujo de Consulta

```
1. Cliente → GET /api/games/search?name=witcher
2. Backend → Steam storesearch (buscar AppID)
3. Devolver lista de juegos con su AppID

1. Cliente → GET /api/games/{id}/achievements?type=global
2. Backend → Steam API (GetGlobalAchievementPercentagesForApp)
3. Devolver porcentajes globales

1. Cliente → GET /api/games/{id}/achievements?type=schema
2. Backend → Steam API (GetSchemaForGame)
3. Devolver esquema completo de logros

1. Cliente → GET /api/games/{id}/achievements?steamid={id}&type=player
2. Backend → Steam API (GetPlayerAchievements)
3. Devolver logros del jugador
```

---

## Implementación en el Backend

### SteamClient (Spring Boot)

```java
@Component
public class SteamClient {

    private final RestClient restClient;
    private final String apiKey;

    public SteamClient(RestClient.Builder builder,
                       @Value("${steam.api.key:}") String apiKey) {
        this.restClient = builder
            .baseUrl("https://api.steampowered.com/ISteamUserStats")
            .build();
        this.apiKey = apiKey;
    }

    /**
     * Busca un juego por nombre y devuelve los resultados con su AppID
     * (NO requiere API key — usa el store search público)
     */
    public List<SteamSearchResult> searchGameByName(String name) {
        try {
            var response = restClient.get()
                .uri(uriBuilder -> uriBuilder
                    .scheme("https")
                    .host("store.steampowered.com")
                    .path("/api/storesearch/")
                    .queryParam("term", name)
                    .queryParam("l", "english")
                    .queryParam("cc", "US")
                    .build())
                .retrieve()
                .body(SteamStoreSearchResponse.class);

            return response.items().stream()
                .filter(item -> "app".equals(item.type()))
                .map(this::toSearchResult)
                .toList();

        } catch (Exception e) {
            return List.of();
        }
    }

    /**
     * Obtiene porcentajes globales de logros (NO requiere API key)
     */
    public List<AchievementPercentage> getGlobalAchievementPercentages(int appId) {
        try {
            var response = restClient.get()
                .uri(uriBuilder -> uriBuilder
                    .path("/GetGlobalAchievementPercentagesForApp/v0002/")
                    .queryParam("gameid", appId)
                    .queryParam("format", "json")
                    .build())
                .retrieve()
                .body(SteamAchievementPercentagesResponse.class);

            return response.achievementpercentages().achievements().stream()
                .map(this::toAchievementPercentage)
                .toList();

        } catch (Exception e) {
            return List.of();
        }
    }

    /**
     * Obtiene el esquema completo de logros (REQUIERE API key)
     */
    public List<AchievementSchema> getSchemaForGame(int appId) {
        if (apiKey == null || apiKey.isBlank()) {
            return List.of();
        }

        try {
            var response = restClient.get()
                .uri(uriBuilder -> uriBuilder
                    .path("/GetSchemaForGame/v2/")
                    .queryParam("key", apiKey)
                    .queryParam("appid", appId)
                    .build())
                .retrieve()
                .body(SteamSchemaResponse.class);

            if (response.game() == null || response.game().availableGameStats() == null) {
                return List.of();
            }

            return response.game().availableGameStats().achievements().stream()
                .map(this::toAchievementSchema)
                .toList();

        } catch (Exception e) {
            return List.of();
        }
    }

    /**
     * Obtiene los logros de un jugador específico (REQUIERE API key)
     */
    public List<PlayerAchievement> getPlayerAchievements(long steamId, int appId) {
        if (apiKey == null || apiKey.isBlank()) {
            return List.of();
        }

        try {
            var response = restClient.get()
                .uri(uriBuilder -> uriBuilder
                    .path("/GetPlayerAchievements/v1/")
                    .queryParam("key", apiKey)
                    .queryParam("steamid", steamId)
                    .queryParam("appid", appId)
                    .build())
                .retrieve()
                .body(SteamPlayerAchievementsResponse.class);

            if (response.playerstats() == null || !response.playerstats().success()) {
                return List.of();
            }

            return response.playerstats().achievements().stream()
                .map(this::toPlayerAchievement)
                .toList();

        } catch (Exception e) {
            return List.of();
        }
    }

    private SteamSearchResult toSearchResult(SteamStoreSearchItem dto) {
        return new SteamSearchResult(
            dto.id(),
            dto.name(),
            dto.tinyImage(),
            dto.metascore(),
            dto.platforms() != null && dto.platforms().windows(),
            dto.platforms() != null && dto.platforms().mac(),
            dto.platforms() != null && dto.platforms().linux()
        );
    }

    private AchievementPercentage toAchievementPercentage(SteamAchievementPercentage dto) {
        return new AchievementPercentage(
            dto.name(),
            Double.parseDouble(dto.percent())
        );
    }

    private AchievementSchema toAchievementSchema(SteamAchievementSchema dto) {
        return new AchievementSchema(
            dto.name(),
            dto.displayName(),
            dto.description(),
            dto.hidden() == 1,
            dto.icon(),
            dto.icongray()
        );
    }

    private PlayerAchievement toPlayerAchievement(SteamPlayerAchievement dto) {
        return new PlayerAchievement(
            dto.apiname(),
            dto.achieved() == 1,
            dto.unlocktime() != 0 ? Instant.ofEpochSecond(dto.unlocktime()) : null
        );
    }
}

// Records de respuesta de Steam API

// storesearch
public record SteamStoreSearchResponse(
    int total,
    List<SteamStoreSearchItem> items
) {}

public record SteamStoreSearchItem(
    String type,
    int id,
    String name,
    String tinyImage,
    String metascore,
    SteamPlatforms platforms,
    SteamPrice price,
    boolean streamingvideo,
    String controllerSupport
) {}

public record SteamPlatforms(
    boolean windows,
    boolean mac,
    boolean linux
) {}

public record SteamPrice(
    String currency,
    int initial,
    int final_
) {}

// GetGlobalAchievementPercentagesForApp
public record SteamAchievementPercentagesResponse(
    SteamAchievementPercentages achievementpercentages
) {}

public record SteamAchievementPercentages(
    List<SteamAchievementPercentage> achievements
) {}

public record SteamAchievementPercentage(
    String name,
    String percent
) {}

// GetSchemaForGame
public record SteamSchemaResponse(
    SteamGameSchema game
) {}

public record SteamGameSchema(
    String gameName,
    String gameVersion,
    SteamAvailableGameStats availableGameStats
) {}

public record SteamAvailableGameStats(
    List<SteamAchievementSchema> achievements
) {}

public record SteamAchievementSchema(
    String name,
    int defaultvalue,
    String displayName,
    int hidden,
    String description,
    String icon,
    String icongray
) {}

// GetPlayerAchievements
public record SteamPlayerAchievementsResponse(
    SteamPlayerStats playerstats
) {}

public record SteamPlayerStats(
    String steamID,
    String gameName,
    List<SteamPlayerAchievement> achievements,
    boolean success
) {}

public record SteamPlayerAchievement(
    String apiname,
    int achieved,
    long unlocktime
) {}

// DTOs internos
public record SteamSearchResult(
    int appId,
    String name,
    String thumbnailUrl,
    String metascore,
    boolean windows,
    boolean mac,
    boolean linux
) {}

public record AchievementPercentage(
    String achievementId,
    double globalCompletionRate
) {}

public record AchievementSchema(
    String achievementId,
    String title,
    String description,
    boolean hidden,
    String iconUrl,
    String iconGrayUrl
) {}

public record PlayerAchievement(
    String achievementId,
    boolean unlocked,
    Instant unlockedAt
) {}
```

### GameAchievementService (Spring Boot)

```java
@Service
public class GameAchievementService {

    private final SteamClient steamClient;

    public GameAchievementService(SteamClient steamClient) {
        this.steamClient = steamClient;
    }

    /**
     * Busca un juego por nombre y devuelve los resultados con su AppID
     */
    public List<SteamSearchResult> searchGameByName(String name) {
        return steamClient.searchGameByName(name);
    }

    /**
     * Obtiene porcentajes globales de logros para un juego
     */
    public List<AchievementPercentage> getGlobalAchievements(int appId) {
        return steamClient.getGlobalAchievementPercentages(appId);
    }

    /**
     * Obtiene el esquema completo de logros (nombres, descripciones, iconos)
     */
    public List<AchievementSchema> getAchievementSchema(int appId) {
        return steamClient.getSchemaForGame(appId);
    }

    /**
     * Obtiene los logros de un jugador específico
     */
    public List<PlayerAchievement> getPlayerAchievements(long steamId, int appId) {
        return steamClient.getPlayerAchievements(steamId, appId);
    }

    /**
     * Combina esquema con porcentajes globales para enriquecer datos
     */
    public List<AchievementDetail> getAchievementsWithPercentages(int appId) {
        var schema = steamClient.getSchemaForGame(appId);
        var percentages = steamClient.getGlobalAchievementPercentages(appId);

        var percentageMap = percentages.stream()
            .collect(Collectors.toMap(AchievementPercentage::achievementId, Function.identity()));

        return schema.stream()
            .map(s -> {
                var pct = percentageMap.get(s.achievementId());
                return new AchievementDetail(
                    s.achievementId(),
                    s.title(),
                    s.description(),
                    s.hidden(),
                    s.iconUrl(),
                    pct != null ? pct.globalCompletionRate() : 0.0
                );
            })
            .toList();
    }
}

public record AchievementDetail(
    String achievementId,
    String title,
    String description,
    boolean hidden,
    String iconUrl,
    double globalCompletionRate
) {}
```

### GameAchievementController (Spring Boot)

```java
@RestController
@RequestMapping("/api/games")
public class GameAchievementController {

    private final GameAchievementService achievementService;

    public GameAchievementController(GameAchievementService achievementService) {
        this.achievementService = achievementService;
    }

    /**
     * GET /api/games/search?name={nombre}
     * Busca un juego por nombre y devuelve los resultados con su AppID
     */
    @GetMapping("/search")
    public ResponseEntity<List<SteamSearchResult>> searchGames(
            @RequestParam String name) {
        return ResponseEntity.ok(achievementService.searchGameByName(name));
    }

    /**
     * GET /api/games/{gameId}/achievements?type=global
     * Obtiene porcentajes globales de logros (sin auth)
     */
    @GetMapping(value = "/{gameId}/achievements", params = "type=global")
    public ResponseEntity<List<AchievementPercentage>> getGlobalAchievements(
            @PathVariable int gameId) {
        return ResponseEntity.ok(achievementService.getGlobalAchievements(gameId));
    }

    /**
     * GET /api/games/{gameId}/achievements?type=schema
     * Obtiene esquema completo de logros (nombres, descripciones, iconos)
     */
    @GetMapping(value = "/{gameId}/achievements", params = "type=schema")
    public ResponseEntity<List<AchievementSchema>> getSchema(
            @PathVariable int gameId) {
        return ResponseEntity.ok(achievementService.getAchievementSchema(gameId));
    }

    /**
     * GET /api/games/{gameId}/achievements?type=player&steamid={steamId}
     * Obtiene logros de un jugador específico
     */
    @GetMapping(value = "/{gameId}/achievements", params = "type=player")
    public ResponseEntity<List<PlayerAchievement>> getPlayerAchievements(
            @PathVariable int gameId,
            @RequestParam long steamid) {
        return ResponseEntity.ok(achievementService.getPlayerAchievements(steamid, gameId));
    }

    /**
     * GET /api/games/{gameId}/achievements?type=detailed
     * Obtiene esquema combinado con porcentajes globales
     */
    @GetMapping(value = "/{gameId}/achievements", params = "type=detailed")
    public ResponseEntity<List<AchievementDetail>> getDetailedAchievements(
            @PathVariable int gameId) {
        return ResponseEntity.ok(achievementService.getAchievementsWithPercentages(gameId));
    }
}
```

---

## Endpoints del Backend

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/games/search?name={nombre}` | Buscar juego por nombre (Steam storesearch) |
| GET | `/api/games/{gameId}/achievements?type=global` | Porcentajes globales de logros |
| GET | `/api/games/{gameId}/achievements?type=schema` | Esquema completo (nombres, descripciones, iconos) |
| GET | `/api/games/{gameId}/achievements?type=player&steamid={id}` | Logros de un jugador |
| GET | `/api/games/{gameId}/achievements?type=detailed` | Esquema + porcentajes combinados |

---

## Comparativa: Steam vs RAWG/FreeToGame

| Característica | Steam | RAWG | FreeToGame |
|----------------|-------|------|------------|
| Logros | ✅ | ❌ | ❌ |
| Porcentajes globales | ✅ (sin key) | ❌ | ❌ |
| Esquema de logros | ✅ (con key) | ❌ | ❌ |
| Progreso de jugador | ✅ (con key) | ❌ | ❌ |
| Búsqueda de juegos | ✅ (storesearch) | ✅ | ✅ |
| Catálogo de juegos | ❌ | ✅ | ✅ |
| Screenshots | ❌ | ✅ | ❌ |
| Gratis | ✅ (con registro) | ✅ (con registro) | ✅ (sin registro) |

---

## Notas Importantes

1. **storesearch** no requiere API key, ideal para buscar juegos por nombre y obtener su AppID
2. **GetGlobalAchievementPercentagesForApp** no requiere API key, ideal para mostrar estadísticas básicas
3. **GetSchemaForGame** y **GetPlayerAchievements** requieren API key
4. El `gameid` en Steam es el AppID, diferente del ID en RAWG/FreeToGame
5. Los nombres de logros en Steam son internos (ej: "LILAC"), no legibles por humanos
6. Para obtener nombres legibles, usar **GetSchemaForGame** que devuelve `displayName`
7. El `steamid` debe ser SteamID64 (17 dígitos)
8. Algunos juegos no tienen logros (devuelve array vacío)
9. La API de Steam tiene rate limit de ~200 requests/5 min sin key
