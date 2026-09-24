# Wiki-Collection — Arquitectura General

## Arquitectura Hexagonal (Backend)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CAPA INTERFAZ (API)                          │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                  Controladores REST (7)                      │  │
│  │  BookController / GameController / BoardGameController       │  │
│  │  MagicController / DeckController / MovieShowController      │  │
│  │  ImageStorageController                                      │  │
│  └─────────────────────────┬────────────────────────────────────┘  │
└────────────────────────────┼─────────────────────────────────────────┘
                             │ (usa)
┌────────────────────────────┼─────────────────────────────────────────┐
│                    CAPA APLICACIÓN (Domain + Application)           │
│  ┌─────────────────────────┼──────────────────────────────────────┐  │
│  │                    Domain (Modelos)                           │  │
│  │  Book · Game · BoardGame · MagicCard · Deck · MovieShow      │  │
│  │  User · UserPreferences · UserOwned                         │  │
│  └─────────────────────────┼──────────────────────────────────────┘  │
│  ┌─────────────────────────┼──────────────────────────────────────┐  │
│  │              Puertos (Interfaces ← Domain)                    │  │
│  │  CrudPort, SearchPort, AuthProviderPort, ImageStoragePort,   │  │
│  │  OAuth2UserServicePort                                      │  │
│  └─────────────────────────┼──────────────────────────────────────┘  │
│  ┌─────────────────────────┼──────────────────────────────────────┐  │
│  │             Use Cases (Application Services)                  │  │
│  │  17 servicios: BookService, GameService, ...                 │  │
│  │  17 *validators*: BookValidator, GameValidator, ...         │  │
│  └─────────────────────────┼──────────────────────────────────────┘  │
└────────────────────────────┼─────────────────────────────────────────┘
                             │ (implementa)
┌────────────────────────────┼─────────────────────────────────────────┐
│                    CAPA INFRASTRUCTURA (Adaptadores)                │
│  ┌─────────────────────────┼──────────────────────────────────────┐  │
│  │         Adaptadores de Puerto (← Application/Domain)          │  │
│  │                                                              │  │
│  │  ┌─ MongoDB ──────────────────────────────────────────────┐  │  │
│  │  │  Repositories: BookRepository, GameRepository, ...    │  │  │
│  │  │  Entity mappers: BookEntityMapper, GameEntityMapper.. │  │  │
│  │  └────────────────────────────────────────────────────────┘  │  │
│  │                                                              │  │
│  │  ┌─ External APIs ─────────────────────────────────────────┐ │  │
│  │  │  Clients (9): GoogleBooksClient, RawgClient,           │ │  │
│  │  │  FreeToGameClient, BoardgameClient, ScryfallClient,   │ │  │
│  │  │  SteamClient, CatboxClient, TmdClient, OAuth2UserService│ │  │
│  │  │  Cache: Caffeine (6 caches: bookSearch, gameSearch...) │ │  │
│  │  └────────────────────────────────────────────────────────┘ │  │
│  │                                                              │  │
│  │  ┌─ JWT Auth ───────────────────────────────────────────────┐│  │
│  │  │  JwtTokenProvider, JwtAuthenticationFilter              ││  │
│  │  └──────────────────────────────────────────────────────────┘│  │
│  │                                                              │  │
│  │  ┌─ File Storage ────────────────────────────────────────────┐│  │
│  │  │  ImageStorageService + CatboxClient                      ││  │
│  │  └───────────────────────────────────────────────────────────┘│  │
│  └───────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

## Stack Técnico

### Backend
| Componente | Versión | Propósito |
|-----------|---------|-----------|
| Java | 25 (Liberica JDK 25) | Lenguaje principal |
| Spring Boot | 4.1.1 | Framework web + DI |
| Spring Data MongoDB | — | Repositorios + mapping |
| Spring Security | 6.x | Auth (futuro) |
| JJWT | 0.12.6 | JWT tokens |
| Caffeine | 3.1.8 | In-memory cache |
| Jackson XML | 2.17.2 | Parseo BGG XML |
| MongoDB Driver | 5.x | Driver oficial |
| Lombok | 1.18.32 | Boilerplate reduction |
| JUnit 5 + Mockito | — | Testing |
| Jacoco | — | Cobertura (umbral 80%) |

### Frontend
| Componente | Versión | Propósito |
|-----------|---------|-----------|
| React | 18.3 | UI library |
| Vite | 5 | Build tool |
| React Router | v6 | Routing |
| Tailwind CSS | 3 | Styling |
| TypeScript | 7.0.2 | Type checking |
| Vitest | — | Testing |
| React Testing Library | — | Component tests |

### Servicios Externos
| Servicio | Tipo | Uso |
|----------|------|-----|
| Google Books API | Books search | Búsqueda de libros |
| RAWG API | Games search | Búsqueda de juegos |
| FreeToGame API | Games fallback | Búsqueda alternativa |
| BoardGameGeek XML API | Board games search | Búsqueda BGG |
| Scryfall API | Magic cards search | Búsqueda cartas + comandantes |
| Steam Web API | Achievements | Logros de juegos |
| TMDB API | Movies/shows search | Búsqueda + imágenes |
| Catbox.moe | Image hosting | Upload imágenes |

### Despliegue
| Servicio | Plan | Uso |
|----------|------|-----|
| MongoDB Atlas | Free (512MB) | Base de datos |
| Render | Free (750 horas/mes) | Backend Java |
| Vercel | Hobby (gratis) | Frontend React |

## Caché Caffeine (Fase 7)

Se implementó caché en memoria para 6 operaciones de búsqueda externa:

| Cache | Backend | TTL | Tamaño máximo |
|-------|---------|-----|---------------|
| `bookSearch` | Google Books | INFINITE (inmutable) | 1000 resultados |
| `gameSearch` | RAWG (con fallback FreeToGame) | INFINITE | 1000 resultados |
| `boardgameSearch` | BGG XML | INFINITE | 1000 resultados |
| `magicSearch` | Scryfall | INFINITE | 1000 resultados |
| `commanderSearch` | Scryfall (comandantes por color) | 1 día | 100 resultados |
| `movieSearch` | TMDB | 7 días | 1000 resultados |

**Nota:** El caché de búsqueda es inmutable (no se invalida), ya que los datos externos no cambian con frecuencia. El caché de comandantes expira en 1 día porque los datos de Scryfall pueden actualizarse.

## Contratos de API Externa (sin modificar)

El backend NO modifica los contratos de las APIs externas. Los DTOs internos son independientes y se mapean desde los DTOs externos.

| API Externa | # Endpoints | Total métodos | Rate limit |
|-------------|------------|---------------|------------|
| Google Books | 2 | ~9 internos | 100 req/seg (con key) |
| RAWG | 3 | 1 (search) | 100k req/mes (free tier) |
| FreeToGame | 1 | 1 (list_all_games) | ~415 juegos totales |
| BGG XML | 1 (search) | 1 | Sin límite explícito |
| Scryfall | ~8 | 2 (search + commanders) | ~10 req/seg |
| Steam Web | 1 (achievements) | 1 | Sin límite documentado |
| TMDB | 2 (search + images) | 2 | 40 req/seg |
| Catbox.moe | 1 (upload) | 1 | Cookies limitadas |

## Flujo de Búsqueda (Fase 1-5)

```
Usuario busca → Controlador → SearchPort → UseCase → ExternalClient
     ↓                                                    ↓
   UI ← Controlador ← UseCase ← ResponseDTO ← ExternalResponseDTO
                     ↑
              Cache hit? → Caffeine (sin cache → API externa → cache)
```

## Dominio (Entity Models)

Las entity models definen solo los campos que el backend necesita. **No replican los DTOs externos.**

| Entity | Campos clave | Colección MongoDB |
|--------|-------------|------------------|
| Book | id, externalId, title, descripcion, author, pages, type, state, comment, start, startDate, endDate, frontpage | `books` |
| Game | id, externalId, title, platform, thumbnailUrl, status, userRating, comment, dateAdded, dateCompleted, externalSource, steamAppId, obtainPlatinum | `games` |
| BoardGame | id, title, description, yearPublished, minPlayers, maxPlayers, minPlaytime, maxPlaytime, publisher, designers, categories, mechanics, imageUrl, thumbnailUrl, bggRating, bggId, category, notes, dateAdded | `board_games` |
| MagicCard | id, scryfallId, oracleId, name, language, releaseDate, manaCost, convertedManaCost, type, text, power, toughness, loyalty, colors, colorIdentity, keywords, rarity, setCode, setName, artist, frame, borderColor, layout, legalities, priceUsd, priceEur, imageUrl, imageLargeUrl, artCropUrl, condition, isFoil, quantity, notes, dateAdded | `magic_cards` |
| MovieShow | id, externalId, title, overview, releaseDate, posterUrl, backdropUrl, voteAverage, mediaType, STATUS, userRating, comment, dateAdded, dateCompleted, externalSource, createdAt, updatedAt | `movie_shows` |
| Deck | id, name, description, commander, commanderColors, cards (List<DeckCard>), createdAt, updatedAt | `decks` |
| User | id, username, email, passwordEncrypted, enabled, accountNonLocked, createdAt, updatedAt | `users` |
| UserPreferences | id, userId, publicProfile, booksVisibility, gamesVisibility, boardGamesVisibility, magicCardsVisibility, decksVisibility, movieShowsVisibility | `user_preferences` |
| UserOwned | id, userId, ownerUsername, followeeId | `user_owned` |

**Nota:** El campo `description` en BookEntity usa minúscula (no `descripcion`), pero el DTO usa `descripcion` (ver ADR-003 + documentación de railway check).

---

## Relaciones entre colecciones

- `books` → `UserPreferences`: N/A (no hay FK, visibilidad se basa en preferences del owner)
- `games` → `UserPreferences`: N/A
- `magic_cards` → `decks`: N/A (las cartas de mazo vienen de Scryfall, no se persisten en `magic_cards`)
- `decks` → `magic_cards`: N/A
- `movie_shows` → `UserPreferences`: N/A
- `users` → `user_preferences`: 1:1 (userId)
- `users` → `user_owned`: 1:N (followeeId → id)
