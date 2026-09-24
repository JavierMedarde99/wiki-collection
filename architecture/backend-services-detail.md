# Servicios Backend — Wiki-Collection

## 17 Services + 9 External Clients

### Services (Application Layer)

| Service | Capa | Responsabilidad |
|---------|------|-----------------|
| `BookService` | application/service | CRUD de libros + validaciones de dominio |
| `BookSearchService` | application/service | Búsqueda externa Google Books + mapeo a BookSearchResult |
| `GameService` | application/service | CRUD de juegos + validaciones |
| `GameSearchService` | application/service | Búsqueda externa RAWG + fallback FreeToGame + mapeo |
| `GameAchievementsService` | application/service | Obtención de logros Steam para juegos vinculados |
| `BoardGameService` | application/service | CRUD de juegos de mesa + validaciones |
| `BoardGameSearchService` | application/service | Búsqueda externa BGG XML + parseo + mapeo |
| `MagicCardService` | application/service | Listado, detalle, eliminación + addFromScryfall |
| `MagicCardSearchService` | application/service | Búsqueda externa Scryfall + mapeo a MagicCardSearchResult |
| `DeckService` | application/service | CRUD de mazos + gestión de cartas (add/remove) |
| `DeckSearchService` | application/service | Búsqueda de comandantes en Scryfall |
| `DeckValidator` | application/service | Validación de reglas Commander (DRAFT/COMPLETE/INVALID) |
| `MovieShowService` | application/service | CRUD de películas/series + validaciones |
| `MovieSearchService` | application/service | Búsqueda externa TMDB + mapeo a MovieSearchResult |
| `ImageStorageService` | application/service | Validación + upload de imágenes a Catbox |
| `DateRangeValidator` | application/service | Validación de fechas (dateAdded vs dateCompleted) |
| `AuthService` | application/service | Login, register, refresh token |
| `JwtService` | application/service | Generación y validación de JWT (access + refresh) |
| `UserDetailsServiceImpl` | application/service | UserDetailsService para Spring Security (carga por userId) |
| `UserPreferencesService` | application/service | CRUD preferencias de colección (activas + visibilidad) |
| `UserProfileService` | application/service | Perfiles públicos y listados de colecciones públicas |
| `StatsService` | application/service | Estadísticas globales (totales por colección) |
| `OwnerResolver` | application/service | Resuelve ownerId del usuario autenticado |
| `OwnerScopeResolver` | application/service | Resuelve scope de visibilidad (mine/other/all) |
| `OwnershipValidator` | application/service | Valida ownership para operaciones de escritura |
| `PagedResults` | application/service | Utilidad de paginación manual |
| `UserPrincipal` | application/service | Principal personalizado que expone username |

### External Clients (Infrastructure Layer — Out)

| Cliente | API | Cache | Reintentos |
|---------|-----|-------|------------|
| `GoogleBooksClient` | Google Books API | `bookSearch` | 4 |
| `RAWGClient` | RAWG Video Games Database | `gameSearch` | 3 |
| `FreeToGameClient` | FreeToGame API | `gameSearch` | 3 |
| `SteamAchievementsClient` | Steam Web API | — | 3 |
| `BggXmlClient` | BoardGameGeek XML API 2 | `boardgameSearch` | 3 (delay 2s para 202 Accepted) |
| `ScryfallClient` | Scryfall API | `magicSearch` + `commanderSearch` | 3 (rate limit 100ms) |
| `TmdbClient` | TMDB API | `movieSearch` | 3 |
| `CatboxClient` | Catbox.moe | — | — |
| `BoardGameXmlMapper` | — (parseo XML) | — | — |
| `MagicCardMapper` | — (mapeo JSON) | — | — |

### Validators

| Validator | Responsabilidad |
|-----------|-----------------|
| `DateRangeValidator` | Valida que dateCompleted no sea anterior a dateAdded |
| `DeckValidator` | Evalúa estado del mazo Commander (DRAFT/COMPLETE/INVALID) |

### PagedResults

Utilidad de paginación manual para construir respuestas pageadas sin depender de Spring Data Page.

## Flujo de Búsqueda Externa

```
1. Cliente → GET /api/v1/{entity}/search?name={query}
2. Service → External Client (con @Cacheable)
3. Si hay caché → devolver caché
4. Si no hay caché → llamada API externa
5. Mapeo de respuesta externa → SearchResult DTO
6. Devolver al cliente
```

## Excepciones Manejadas

Ver `architecture/backend.md` → sección "Excepciones de aplicación".
