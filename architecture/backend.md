---
type: architecture
title: "Arquitectura Backend — Wiki-Collection"
description: "Arquitectura hexagonal del backend: puertos, adaptadores y caché"
tags: [backend, arquitectura]
---

# Arquitectura Backend — Wiki-Collection

**stack:** Java 25 + Spring Boot 4.1.1 + MongoDB + Spring Data + Maven

## Arquitectura Hexagonal

El backend sigue el patrón de puertos y adaptadores para mantener el dominio aislado de frameworks y tecnologías externas.

### Estructura de Paquetes

```
com.wikicollection/
├── WikiCollectionApplication.java          # @SpringBootApplication - punto de entrada
├── domain/                                 # Núcleo del negocio
│   ├── model/                              # Entidades y value objects (79 archivos)
│   │   ├── Book.java                       # id, externalId, title, descripcion, author, pages, type, state, comment, start, startDate, endDate, frontpage, ownerId
│   │   ├── BookState.java                  # enum: TO_READ, READING, COMPLETED
│   │   ├── BookType.java                   # enum: MANGA, NOVEL, GRAPHIC_NOVEL
│   │   ├── BookSearchCriteria.java         # record: name, author, type, state
│   │   ├── BookSearchResult.java           # record: id, title, authors, isbn, coverImage, description, pageCount, publisher, publishedDate, language, categories
│   │   ├── Game.java                       # id, externalId, title, genres, platform (free text), thumbnailUrl, status, userRating, comment, dateAdded, dateCompleted, externalSource, steamAppId, acquisitionDate, acquisitionPrice, parentId, parentTitle, ownerId
│   │   ├── GameStatus.java                 # enum: PLAYING, COMPLETED, WISHLIST, ABANDONED
│   │   ├── GameSearchCriteria.java         # record: name, platforms (lista), status, genres, ownerId, excludeOwnerIds, type (base|dlc|all)
│   │   ├── GameSearchResult.java           # record: id, title, description, genre, platform, publisher, developer, releaseDate, thumbnailUrl, externalSource
│   │   ├── GameTypeFilter.java             # enum: BASE, DLC, ALL (filtro GET /games?type=)
│   │   ├── GameAddition.java               # record: externalId, title, platform, genres, thumbnailUrl, releaseDate, externalSource, inCollection, alreadyLinked
│   │   ├── GameAdditionsCatalog.java       # record: baseGameId, baseExternalId, additions, totalAvailable
│   │   ├── GameAdditionsPage.java          # record: additions, totalAvailable (página RAWG, vacía = empty())
│   │   ├── GameAdditionsAddResult.java     # record: created, linked, skipped
│   │   ├── GameAdditionSeed.java           # record: externalId, status, acquisitionDate, acquisitionPrice, dateAdded, dateCompleted, link
│   │   ├── GameAdditionSkip.java           # record: externalId, reason (REASON_ALREADY_LINKED)
│   │   ├── BoardGame.java                  # id, title, description, yearPublished, minPlayers, maxPlayers, minPlaytime, maxPlaytime, publisher, designers, categories, mechanics, imageUrl, thumbnailUrl, bggRating, bggId, status, notes, dateAdded, personalRating, playCount, lastPlayedDate, difficulty, acquisitionPrice, bggType, parentId, parentTitle, ownerId
│   │   ├── BoardGameStatus.java            # enum: OWNED, WISHLIST
│   │   ├── BoardGameSearchCriteria.java    # record: name, status, genres, ownerId, excludeOwnerIds, type (base|expansion|all)
│   │   ├── BoardGameSearchResult.java      # record: bggId, title, description, yearPublished, minPlayers, maxPlayers, minPlaytime, maxPlaytime, publisher, designers, categories, mechanics, imageUrl, thumbnailUrl, bggRating, externalSource
│   │   ├── BoardGameType.java              # enum: BOARDGAME("boardgame"), BOARDGAME_EXPANSION("boardgameexpansion") — claves BGG (@JsonValue/@JsonCreator)
│   │   ├── BoardGameTypeFilter.java        # enum: BASE, EXPANSION, ALL (filtro GET /boardgames?type=)
│   │   ├── BoardGameExpansion.java         # record: bggId, title, yearPublished, imageUrl, thumbnailUrl, inCollection
│   │   ├── BoardGameExpansionCatalog.java  # record: baseGameId, baseBggId, expansions
│   │   ├── BoardGameExpansionInfo.java     # record del catálogo BGG: bggId, title, description, datos de ficha, type, baseBggIds
│   │   ├── BoardGameExpansionAddResult.java # record: created, skipped
│   │   ├── BoardGameExpansionSeed.java     # record: bggId, status, acquisitionPrice, dateAdded
│   │   ├── BoardGameExpansionSkip.java     # record: bggId, reason (REASON_ALREADY_IN_COLLECTION)
│   │   ├── Deck.java                       # id, name, description, commander, commanderColors, cards (List<DeckCard>), createdAt, updatedAt, ownerId
│   │   ├── DeckCard.java                   # cardName, quantity, inCollection, isProxy, manaCost, typeLine, colorIdentity, imageUrl, scryfallId
│   │   ├── DeckStatus.java                 # enum: DRAFT, COMPLETE, INVALID
│   │   ├── DeckStatusReport.java           # record: status, reasons
│   │   ├── DeckImportFormat.java           # enum: TXT, JSON, CSV (Fase 23)
│   │   ├── DeckImportMode.java             # enum: REPLACE, MERGE (Fase 23)
│   │   ├── DeckImportStatus.java           # enum: PENDING, RUNNING, COMPLETED, FAILED (Fase 23)
│   │   ├── MagicCard.java                  # id, scryfallId, oracleId, name, language, releaseDate, manaCost, convertedManaCost, type, text, power, toughness, loyalty, colors, colorIdentity, keywords, rarity, setCode, setName, artist, frame, borderColor, layout, legalities, priceUsd, priceEur, imageUrl, imageLargeUrl, artCropUrl, condition, isFoil, quantity, notes, dateAdded, ownerId
│   │   ├── MagicCardCondition.java         # enum: MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR
│   │   ├── MagicCardLanguage.java          # enum: ENGLISH, SPANISH, FRENCH, GERMAN, ITALIAN, PORTUGUESE, JAPANESE, CHINESE
│   │   ├── MagicCardSearchCriteria.java    # record: name, rarity, color, type
│   │   ├── MagicCardSearchResult.java      # record: scryfallId, name, manaCost, type, rarity, setCode, setName, imageUrl, priceUsd, colors, colorIdentity, text
│   │   ├── MovieShow.java                  # id, externalId, title, overview, releaseDate, posterUrl, backdropUrl, voteAverage, mediaType, status, userRating, comment, dateAdded, dateCompleted, externalSource, createdAt, updatedAt, ownerId
│   │   ├── MovieMediaType.java             # enum: MOVIE, TV
│   │   ├── MovieStatus.java                # enum: WATCHING, WATCHED, PLAN_TO_WATCH
│   │   ├── MovieSearchCriteria.java        # record: name, status, mediaType
│   │   ├── MovieSearchResult.java          # record: externalId, title, overview, releaseDate, posterUrl, backdropUrl, voteAverage, mediaType, externalSource
│   │   ├── SteamAchievement.java           # record: apiname, achieved, name, description, iconUrl
│   │   ├── AchievementsSummary.java        # record: achievements, totalAchievements, totalAchieved, percentage
│   │   ├── User.java                       # id, username, email, password, displayName, avatarUrl, bio, createdAt, updatedAt
│   │   ├── UserOwned.java                  # record: userId, username, displayName
│   │   ├── UserPreferences.java            # id, userId, activeCollections, collectionVisibility, createdAt, updatedAt
│   │   ├── CollectionType.java             # enum: BOOKS, GAMES, BOARD_GAMES, MAGIC, DECKS, MOVIE_SHOWS
│   │   ├── CollectionVisibility.java       # enum: PUBLIC, PRIVATE
│   │   ├── AuthSession.java                # record: accessToken, refreshToken, user
│   │   ├── AuthTokens.java                 # record: accessToken, refreshToken
│   │   └── AchievementsSummary.java        # record: achievements, totalAchievements, totalAchieved, percentage
│   └── port/
│       ├── in/                             # Casos de uso (puertos de entrada) — 25 interfaces
│       │   ├── BookUseCase.java
│       │   ├── BookSearchUseCase.java
│       │   ├── GameUseCase.java
│       │   ├── GameSearchUseCase.java
│       │   ├── GameAchievementsUseCase.java
│       │   ├── GameAdditionsUseCase.java   # additionsOf (catálogo RAWG), addAdditions (alta/vínculo de DLC)
│       │   ├── BoardGameUseCase.java
│       │   ├── BoardGameSearchUseCase.java
│       │   ├── BoardGameExpansionUseCase.java # expansionsOf (catálogo BGG), addExpansions (alta de expansiones)
│       │   ├── MagicCardUseCase.java
│       │   ├── MagicCardSearchUseCase.java
│       │   ├── DeckUseCase.java
│       │   ├── DeckSearchUseCase.java
│       │   ├── DeckImportUseCase.java      # Fase 23: startImport, findJob
│       │   ├── MovieShowUseCase.java
│       │   ├── MovieSearchUseCase.java
│       │   ├── UserUseCase.java              # CRUD de usuarios (buscar por username/email, existe)
│       │   ├── UserProfileUseCase.java       # Perfiles públicos y colecciones públicas con paginación
│       │   ├── UserPreferencesUseCase.java   # CRUD de preferencias de colección (activas + visibilidad)
│       │   └── StatsUseCase.java             # Estadísticas globales del sistema
│       └── out/                            # Puertos de salida — 27 interfaces
│           ├── BookRepository.java
│           ├── ExternalBookCatalogClient.java
│           ├── GameRepository.java
│           ├── ExternalGameCatalogClient.java
│           ├── ExternalGameAdditionsCatalogClient.java # getAdditions(externalId) → GameAdditionsPage (RAWG, separado de la búsqueda)
│           ├── SteamCatalogueClient.java
│           ├── BoardGameRepository.java
│           ├── ExternalBoardGameCatalogClient.java # search + findByBggIds + expansionsOf (BGG)
│           ├── MagicCardRepository.java
│           ├── ExternalMagicCardCatalogClient.java
│           ├── DeckRepository.java
│           ├── MovieShowRepository.java
│           ├── ExternalMovieCatalogClient.java
│           ├── ImageHostingClient.java
│           ├── UserRepository.java           # CRUD de usuarios (findByUsername, findByEmail, exists)
│           ├── UserPreferencesRepository.java # CRUD de preferencias (findByUserId)
│           └── UserProfilePort.java          # Consulta de perfiles públicos y colecciones
├── application/                            # Capa de aplicación
│   ├── exception/                          # Excepciones de aplicación — 24 archivos
│   │   ├── BookConflictException.java      # HTTP 409
│   │   ├── BookNotFoundException.java      # HTTP 404
│   │   ├── GameNotFoundException.java       # HTTP 404
│   │   ├── GameAdditionsException.java     # HTTP 422 (regla de contenido adicional: base en WISHLIST, no viene de RAWG o el id no es una adición suya)
│   │   ├── BoardGameNotFoundException.java # HTTP 404
│   │   ├── BoardGameExpansionException.java # HTTP 422 (regla de expansiones: base no OWNED o el bggId no cuelga de ese base)
│   │   ├── MagicCardNotFoundException.java # HTTP 404
│   │   ├── DeckNotFoundException.java      # HTTP 404
│   │   ├── MovieShowNotFoundException.java # HTTP 404
│   │   ├── MovieShowConflictException.java # HTTP 409
│   │   ├── CatboxUploadException.java      # HTTP 502
│   │   ├── UserAlreadyExistsException.java # HTTP 409 (registro: username ocupado)
│   │   ├── EmailAlreadyExistsException.java # HTTP 409 (registro: email ocupado)
│   │   ├── UserNotFoundException.java      # HTTP 404
│   │   ├── ForbiddenException.java         # HTTP 403 (operación no permitida)
│   │   └── UnauthenticatedException.java   # HTTP 401 (token inválido/expirado)
│   └── service/                            # Implementación de casos de uso — 43 archivos
│       ├── BookService.java
│       ├── BookSearchService.java
│       ├── GameService.java
│       ├── GameSearchService.java
│       ├── GameAchievementsService.java
│       ├── GameAdditionsService.java      # DLC/adiciones de RAWG enlazados al base (gemelo del de expansiones)
│       ├── BoardGameService.java
│       ├── BoardGameSearchService.java
│       ├── BoardGameExpansionService.java # Expansiones de BGG enlazadas al base + catálogo
│       ├── MagicCardService.java
│       ├── MagicCardSearchService.java
│       ├── DeckService.java
│       ├── DeckSearchService.java
│       ├── DeckValidator.java
│       ├── DeckImportService.java          # Fase 23: parseo, resolución y guardado
│       ├── DeckImportWorker.java           # Fase 23: jobs en segundo plano
│       ├── DeckImportJobStore.java         # Fase 23: registro de jobs en memoria
│       ├── MovieShowService.java
│       ├── MovieSearchService.java
│       ├── ImageStorageService.java
│       ├── DateRangeValidator.java
│       ├── PagedResults.java                # Utilidad de paginación manual
│       ├── AuthService.java                 # Login, register, refresh token
│       ├── JwtService.java                  # Generación y validación de JWT (access + refresh)
│       ├── UserDetailsServiceImpl.java      # UserDetailsService para Spring Security (carga por userId)
│       ├── UserPreferencesService.java      # CRUD preferencias de colección (activas + visibilidad)
│       ├── UserProfileService.java          # Perfiles públicos y listados de colecciones públicas
│       ├── StatsService.java                # Estadísticas globales (totales por colección)
│       ├── OwnerResolver.java               # Resuelve ownerId del usuario autenticado
│       ├── OwnerScopeResolver.java          # Resuelve scope de visibilidad (mine/other/all)
│       ├── OwnershipValidator.java          # Valida ownership para operaciones de escritura
│       └── UserPrincipal.java               # Principal personalizado que expone username
└── infrastructure/                         # Capa de infraestructura
    ├── adapter/
    │   ├── in/web/                         # Controladores REST — 16 archivos
    │   │   ├── BookController.java         # /api/v1/books
    │   │   ├── GameController.java         # /api/v1/games (+ /{id}/additions, contenido adicional RAWG)
    │   │   ├── BoardGameController.java    # /api/v1/boardgames (+ /{id}/expansions, expansiones BGG)
    │   │   ├── MagicCardController.java    # /api/v1/magic
    │   │   ├── DeckController.java         # /api/v1/decks (CRUD + /{id}/imports, Fase 23)
    │   │   ├── MovieShowController.java    # /api/v1/movieshows
    │   │   ├── ImageStorageController.java # /api/v1/images
    │   │   ├── AuthController.java         # /api/v1/auth (register, login, refresh, me)
    │   │   ├── UserProfileController.java  # /api/v1/users/{username}
    │   │   ├── UserPreferencesController.java # /api/v1/preferences
    │   │   ├── StatsController.java        # /api/v1/stats
    │   │   ├── GlobalExceptionHandler.java
    │   │   └── dto/                        # DTOs de request/response — 70 archivos
    │   │       ├── BookRequest.java
    │   │       ├── BookResponse.java
    │   │       ├── BookDtoMapper.java
    │   │       ├── GameRequest.java
    │   │       ├── GameResponse.java       # incluye parentId, parentTitle (vínculo DLC)
    │   │       ├── GameDtoMapper.java
    │   │       ├── GameAchievementMapper.java
    │   │       ├── GameAchievementResponse.java
    │   │       ├── AchievementsResponse.java
    │   │       ├── AddGameAdditionsRequest.java # externalIds+status legado o additions con estado (máx 40), toSeeds()
    │   │       ├── GameAdditionsListResponse.java # baseGameId, baseExternalId, additions, totalAvailable
    │   │       ├── GameAdditionResponse.java # externalId, title, platform, genres, thumbnailUrl, releaseDate, externalSource, inCollection, alreadyLinked
    │   │       ├── GameAdditionsAddResponse.java # created + linked + skipped
    │   │       ├── GameAdditionSkipResponse.java # externalId, reason (ALREADY_LINKED)
    │   │       ├── BoardGameRequest.java
    │   │       ├── BoardGameResponse.java  # incluye bggType, parentId, parentTitle (vínculo expansión)
    │   │       ├── BoardGameSearchResponse.java
    │   │       ├── BoardGameDtoMapper.java
    │   │       ├── AddExpansionsRequest.java # bggIds legado o expansions con estado (máx 20), toSeeds()
    │   │       ├── BoardGameExpansionListResponse.java # baseGameId, baseBggId, expansions
    │   │       ├── BoardGameExpansionResponse.java # bggId, title, yearPublished, imageUrl, thumbnailUrl, inCollection
    │   │       ├── AddExpansionsResponse.java # created + skipped
    │   │       ├── SkippedExpansionResponse.java # bggId, reason (ALREADY_IN_COLLECTION)
    │   │       ├── MagicCardResponse.java
    │   │       ├── MagicCardSearchResponse.java
    │   │       ├── MagicCardDtoMapper.java
    │   │       ├── DeckRequest.java
    │   │       ├── DeckResponse.java
    │   │       ├── DeckCardRequest.java
    │   │       ├── DeckCardResponse.java
    │   │       ├── DeckStatusResponse.java
    │   │       ├── DeckImportAcceptedResponse.java   # Fase 23: 202 {jobId, status, statusUrl}
    │   │       ├── DeckImportJobResponse.java        # Fase 23: estado del job
    │   │       ├── DeckImportDtoMapper.java
    │   │       ├── DeckDtoMapper.java
    │   │       ├── MovieShowRequest.java
    │   │       ├── MovieShowResponse.java
    │   │       ├── MovieShowDtoMapper.java
    │   │       ├── ImageResponse.java
    │   │       ├── ErrorResponse.java
    │   │       ├── AuthResponse.java         # accessToken, refreshToken, user
    │   │       ├── RegisterRequest.java      # username, email, password, displayName
    │   │       ├── LoginRequest.java         # username, password
    │   │       ├── RefreshTokenRequest.java  # refreshToken
    │   │       ├── UserResponse.java         # id, username, displayName, avatarUrl, bio, createdAt
    │   │       ├── PublicProfileResponse.java # username, displayName, avatarUrl, bio, publicCollections
    │   │       ├── UserPreferencesResponse.java # id, userId, activeCollections, collectionVisibility
    │   │       ├── UserPreferencesRequest.java  # activeCollections, collectionVisibility
    │   │       ├── CollectionVisibilityRequest.java # visibility map
    │   │       ├── ActiveCollectionsRequest.java   # collections map
    │   │       ├── GlobalStatsResponse.java        # totalUsers, totalBooks, totalGames, totalBoardGames, totalMagicCards, totalDecks, totalMovieShows
    │   │       └── UserOwnedResponse.java         # userId, username, displayName
    │   └── out/
    │       ├── persistence/                 # Adaptadores de base de datos — 50 archivos
    │       │   ├── BookEntity.java          # @Document("books"), @Indexed en externalId
    │       │   ├── BookEntityMapper.java
    │       │   ├── BookPersistenceAdapter.java
    │       │   ├── SpringDataBookRepository.java
    │       │   ├── GameEntity.java          # @Document("games"), @Indexed en ownerId; parentId/parentTitle (vínculo DLC)
    │       │   ├── GameEntityMapper.java
    │       │   ├── GamePersistenceAdapter.java
    │       │   ├── SpringDataGameRepository.java
    │       │   ├── BoardGameEntity.java     # @Document("board_games"), @Indexed en ownerId; bggType/parentId/parentTitle (expansiones)
    │       │   ├── BoardGameEntityMapper.java
    │       │   ├── BoardGamePersistenceAdapter.java
    │       │   ├── SpringDataBoardGameRepository.java
    │       │   ├── MagicCardEntity.java     # @Document("magic_cards")
    │       │   ├── MagicCardEntityMapper.java
    │       │   ├── MagicCardPersistenceAdapter.java
    │       │   ├── SpringDataMagicCardRepository.java
    │       │   ├── DeckEntity.java          # @Document("decks"), contiene List<DeckCardEntity>
    │       │   ├── DeckCardEntity.java      # Subdocumento embebido
    │       │   ├── DeckEntityMapper.java
    │       │   ├── DeckPersistenceAdapter.java
    │       │   ├── SpringDataDeckRepository.java
    │       │   ├── MovieShowEntity.java     # @Document("movie_shows"), @CreatedDate, @LastModifiedDate
    │       │   ├── MovieShowEntityMapper.java
    │       │   ├── MovieShowPersistenceAdapter.java
    │       │   ├── SpringDataMovieShowRepository.java
    │       │   ├── BoardGameStatusMigration.java  # CommandLineRunner para migración de estados
    │       │   ├── UserEntity.java           # @Document("users"), @Indexed en username + email
    │       │   ├── UserEntityMapper.java
    │       │   ├── UserPersistenceAdapter.java
    │       │   ├── SpringDataUserRepository.java
    │       │   ├── UserPreferencesEntity.java # @Document("user_preferences")
    │       │   ├── UserPreferencesEntityMapper.java
    │       │   ├── UserPreferencesPersistenceAdapter.java
    │       │   ├── SpringDataUserPreferencesRepository.java
    │       │   ├── UserOwnedEntity.java      # @Document("user_owned")
    │       │   ├── UserOwnedMapping.java     # @DocumentReference / mapping
    │       │   └── UserProfilePersistenceAdapter.java
    │       ├── google/                      # Cliente Google Books — 1 archivo
    │       │   └── GoogleBooksClient.java   # RestClient, search con intitle: + langRestrict=es, @Cacheable("bookSearch"), 4 reintentos
    │       ├── rawg/                        # Cliente RAWG — 1 archivo
    │       │   └── RAWGClient.java          # RestClient, @Cacheable("gameSearch"), mapeo de plataformas limitado, 3 reintentos
    │       ├── freetogame/                  # Cliente FreeToGame — 1 archivo
    │       │   └── FreeToGameClient.java    # RestClient, @Cacheable("gameSearch"), solo detecta PC, 3 reintentos
    │       ├── steam/                       # Cliente Steam — 1 archivo
    │       │   └── SteamAchievementsClient.java  # RestClient, 2 clientes (Web API + Store search), GetSchemaForGame + GetPlayerAchievements, 3 reintentos
    │       ├── bgg/                         # Cliente BoardGameGeek — 2 archivos
    │       │   ├── xml/
    │       │   │   └── BggXmlClient.java   # RestTemplate, Jackson XML, @Cacheable("boardgameSearch"), reintentos con 2s delay para 202 Accepted
    │       │   └── mapper/
    │       │       └── BoardGameXmlMapper.java  # Parsea XML de BGG a BoardGameSearchResult
    │       ├── scryfall/                    # Cliente Scryfall — 2 archivos
    │       │   ├── ScryfallClient.java      # RestTemplate, búsqueda fuzzy + por ID + comandantes, @Cacheable("magicSearch") + @Cacheable("commanderSearch"), rate limit 100ms, 3 reintentos
    │       │   └── MagicCardMapper.java     # Mapea respuestas Scryfall a MagicCardSearchResult/MagicCard
    │       ├── tmdb/                        # Cliente TMDB — 1 archivo
    │       │   └── TmdbClient.java          # RestClient, busca en /search/movie y /search/tv, idioma es-ES, @Cacheable("movieSearch"), 3 reintentos
    │       └── catbox/                      # Cliente Catbox — 1 archivo
    │           └── CatboxClient.java        # RestClient, upload/delete a /user/api.php
    └── config/                             # Configuración Spring — 28 archivos
        ├── AuthDataMigration.java          # CommandLineRunner: crear usuario admin por defecto
        ├── BggClientConfig.java
        ├── CacheConfig.java                # @EnableCaching + Caffeine (44 cachés en CACHE_NAMES)
        ├── CacheProperties.java            # @ConfigurationProperties(\"app.cache\"): defaultTtl, maxSize, TTL por caché
        ├── CurrentUser.java                # @interface para inyectar userId en parámetros de controller
        ├── CurrentUserHandlerMethodArgumentResolver.java # Resolver para @CurrentUser (extrae de SecurityContext)
        ├── HttpClientProperties.java       # @ConfigurationProperties(\"http.client\"): connectTimeout, readTimeout
        ├── JwtAuthenticationFilter.java    # OncePerRequestFilter: extrae Bearer token, valida, setea SecurityContext
        ├── MongoAuditConfig.java           # @EnableMongoAuditing (única clase con esta anotación)
        ├── MongoIndexMigration.java        # CommandLineRunner: crear índices (ownerId, externalId parcial, {ownerId, parentId} en games/board_games)
        ├── OpenApiConfig.java              # Configuración springdoc-openapi (Swagger 3.1.0)
        ├── RateLimitFilter.java            # Filtro de rate limiting por IP (mapa en memoria)
        ├── RestClientConfig.java           # Beans RestClient: google, rawg, freetogame, steam, catbox, tmdb
        ├── ScryfallClientConfig.java       # Bean scryfallRestClient
        ├── SecurityConfig.java             # @EnableWebSecurity + @EnableMethodSecurity: JWT, CORS, reglas de acceso (GET público, POST/PUT/DELETE autenticado)
        ├── TmdbClientConfig.java           # Bean tmdbRestClient
        ├── StringToBoardGameStatusConverter.java
        ├── StringToBoardGameTypeFilterConverter.java # ?type=base|expansion|all
        ├── StringToBookStateConverter.java
        ├── StringToGameStatusConverter.java
        ├── StringToGameTypeFilterConverter.java # ?type=base|dlc|all
        ├── StringToMovieMediaTypeConverter.java
        ├── StringToMovieStatusConverter.java
        ├── UserPreferencesMigration.java   # CommandLineRunner: crear UserPreferences por defecto al registrar usuario
        └── WebConfig.java                  # CORS: /api/** → orígenes configurables
```

### Base de Datos — 9 Colecciones MongoDB

| Colección | Entidad | Índices |
|-----------|---------|---------|
| `books` | BookEntity | externalId |
| `games` | GameEntity | ownerId, {ownerId, parentId} |
| `board_games` | BoardGameEntity | ownerId, {ownerId, parentId} |
| `magic_cards` | MagicCardEntity | — |
| `decks` | DeckEntity | — |
| `movie_shows` | MovieShowEntity | externalId |
| `users` | UserEntity | username, email |
| `user_preferences` | UserPreferencesEntity | userId |
| `user_owned` | UserOwnedEntity | userId |

> **Nota sobre índices:** Todos los índices de la tabla los crea `MongoIndexMigration` al arrancar (`spring.data.mongodb.auto-index-creation=false`, las anotaciones `@Indexed` no crean nada por sí solas). Detalle por colección en [data-model/schema.md](../data-model/schema.md).

> **Nota sobre campos de auditoría:** `createdAt`/`updatedAt` están presentes en `DeckEntity` y `MovieShowEntity` (con anotaciones `@CreatedDate`/`@LastModifiedDate`). El resto de entidades no tienen campos de auditoría explícitos. Verificar en el código real de `backend-collection` si es intencional o un caso de inconsistencia documental.

### Controladores REST — 12 Controladores

| Controlador | Ruta Base | Endpoints |
|-------------|-----------|-----------|
| BookController | `/api/v1/books` | GET, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search |
| GameController | `/api/v1/games` | GET, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /{id}/achievements, GET /{id}/additions, POST /{id}/additions, GET /search |
| BoardGameController | `/api/v1/boardgames` | GET, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /{id}/expansions, POST /{id}/expansions, GET /search |
| MagicCardController | `/api/v1/magic` | GET, GET /{id}, POST /scryfall/{scryfallId}, DELETE /{id}, GET /search, GET /commanders |
| DeckController | `/api/v1/decks` | GET, GET /{id}, POST, PUT /{id}, DELETE /{id}, POST /{id}/cards, DELETE /{id}/cards/{scryfallId}, GET /{id}/status |
| MovieShowController | `/api/v1/movieshows` | GET, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search |
| ImageStorageController | `/api/v1/images` | POST /upload, DELETE /{filename} |
| AuthController | `/api/v1/auth` | POST /register, POST /login, POST /refresh, GET /me |
| UserProfileController | `/api/v1/users/{username}` | GET (perfil público), GET /books, GET /games, GET /boardgames, GET /magic, GET /decks, GET /movieshows |
| UserPreferencesController | `/api/v1/preferences` | GET, PUT, PATCH /active-collections, PATCH /collection-visibility, GET /active-collections |
| StatsController | `/api/v1/stats` | GET /global (totales del sistema) |
| GlobalExceptionHandler | — | Manejo centralizado de excepciones → ErrorResponse |

### Cache Caffeine — 44 Cachés

Todas declaradas en `CacheConfig.CACHE_NAMES` (44 nombres) y montadas por `CaffeineCacheManager`. Los parámetros salen de `app.cache` en `application.properties`: `default-ttl=3600s` y `max-size=1000` para todas; el TTL de cada caché se resuelve por grupo de colección con `CacheProperties.ttlFor` (`users` 300s, `preferences` 120s, `stats` 300s, el resto 3600s).

| Grupo | Cachés | Nº |
|-------|--------|----|
| Catálogo externo y búsqueda | `bookSearch`, `bookEnrich`, `gameSearch`, `boardgameSearch`, `boardgameExpansions`, `gameAdditions`, `magicSearch`, `magicPrintings`, `commanderSearch`, `movieSearch`, `platformSearch`, `amiiboSearch`, `funkoSearch`, `funkoCategories` | 14 |
| Detalle por id | `bookDetail`, `gameDetail`, `boardgameDetail`, `movieDetail`, `magicDetail`, `deckDetail`, `amiiboDetail`, `funkoDetail`, `userProfileDetail` | 9 |
| Listados paginados | `bookList`, `gameList`, `boardgameList`, `movieList`, `magicList`, `deckList`, `amiiboList`, `funkoList` | 8 |
| Listados por usuario (la clave incluye al espectador) | `userBooksList`, `userGamesList`, `userBoardGamesList`, `userMagicCardsList`, `userDecksList`, `userMovieShowsList`, `userStreamsList`, `userAmiibosList`, `userFunkosList` | 9 |
| Preferencias | `userPreferencesDetail`, `userActiveCollections`, `userPreferenceFlags` | 3 |
| Estadísticas | `stats` | 1 |
| **Total** | `CacheConfig.CACHE_NAMES` | **44** |

> `boardgameExpansions` (clave = `bggId` del base) y `gameAdditions` (clave = `externalId` del base) son cachés de **catálogo externo puro**: los flags de colección (`inCollection`, `alreadyLinked`) se calculan en el servicio, fuera de la caché, porque dependen del espectador.

### Expansiones de juegos de mesa y contenido adicional de videojuegos (PRs #496/#507/#508/#509)

Dos features gemelas que enlazan registros de la colección con un juego base mediante `parentId`/`parentTitle` (campos y reglas de datos en [data-model/schema.md](../data-model/schema.md)). Comparten estructura hexagonal:

| Pieza | Juegos de mesa — expansiones BGG | Videojuegos — contenido adicional RAWG |
|-------|----------------------------------|----------------------------------------|
| Puerto de entrada | `BoardGameExpansionUseCase` | `GameAdditionsUseCase` |
| Servicio | `BoardGameExpansionService` | `GameAdditionsService` |
| Puerto de salida (catálogo) | `ExternalBoardGameCatalogClient` → `BggXmlClient` | `ExternalGameAdditionsCatalogClient` → `RAWGClient` |
| Caché de catálogo | `boardgameExpansions` | `gameAdditions` |
| Excepción de regla → HTTP | `BoardGameExpansionException` → 422 | `GameAdditionsException` → 422 |
| Endpoints | `GET/POST /api/v1/boardgames/{id}/expansions` | `GET/POST /api/v1/games/{id}/additions` |
| Filtro de listado | `?type=base\|expansion\|all` (`BoardGameTypeFilter`) | `?type=base\|dlc\|all` (`GameTypeFilter`) |
| Límite por petición | 20 (máximo de BGG por llamada; `findByBggIds` consulta en lotes de 20) | 40 (tamaño de la página de adiciones de RAWG) |

Ambos servicios usan `OwnershipValidator` (escrituras → 403) y `OwnerResolver` (snapshot `userOwned` en el alta).

**Catálogo (`GET`, público):** el servicio resuelve el base (404 si no existe) y pide el catálogo externo ya cacheado; con el `viewerId` (anónimo ⇒ sin flags, todo `false`) calcula los flags con una sola consulta a la colección. Divergencia de diseño: un base sin `bggId` devuelve **lista vacía** en expansiones, mientras que un base sin `externalId` o sin `externalSource=RAWG` devuelve **422** en adiciones.

**Alta (`POST`, auth → 201):** `AddExpansionsRequest`/`AddGameAdditionsRequest` normalizan el body con `toSeeds()`: hay que enviar exactamente una de las dos formas (ids legados o items con estado) y los datos de adquisición se validan antes de mirar el base (400 → `IllegalArgumentException`). Después: existencia del base (404), ownership (403) y regla de negocio (422) — en expansiones, el base debe estar `OWNED` y BGG debe confirmar que el bggId es expansión de ese base; en adiciones, el base no puede estar en `WISHLIST` y el externalId debe ser adición suya en RAWG. Todo se valida **antes** de guardar nada (una expansión inválida tira la petición entera). Idempotencia: lo ya presente se devuelve en `skipped` (`ALREADY_IN_COLLECTION` / `ALREADY_LINKED`); en videojuegos, un juego suelto ya existente se **enlaza** (`linked`) conservando su estado, nota y precio.

**Invalidación:** `addExpansions` hace `@CacheEvict(allEntries)` de `boardgameList`, `boardgameDetail` y `stats`; `addAdditions` de `gameList`, `gameDetail` y `stats`. Las cachés de catálogo no se invalidan. El **borrado en cascada** vive en los servicios CRUD: `BoardGameService.delete` borra las expansiones del base si el registro no es `BOARDGAME_EXPANSION`, y `GameService.delete` borra los DLC si `parentId == null` (ambos vía `deleteByOwnerIdAndParentId`).

### Configuración de Proyecto

| Archivo | Propósito |
|---------|-----------|
| `pom.xml` | Maven: Spring Boot 4.1.1, Java 25, Lombok 1.18.46, JaCoCo ≥80% línea, Springdoc OpenAPI 3.1.0, Spring Security, JJWT 0.12.6 |
| `application.properties` | Server (8080), multipart, MongoDB (spring.mongodb.uri), APIs externas, cache Caffeine, CORS, OpenAPI, JWT |
| `.github/workflows/checks.yml` | CI: `./mvnw clean package` con Java 25 |
| `AGENTS.md` | Guía para contribuyentes: comandos, gotchas, flujo hexagonal |
| `ARCHITECTURE.md` | Documentación detallada de arquitectura hexagonal |

### Flujo de una Petición

1. **Controller** recibe HTTP request y la convierte a DTO
2. **Service** (application) procesa la lógica de negocio
3. **Domain** valida reglas de negocio
4. **Persistence Adapter** guarda/consulta en MongoDB
5. **External Client** consulta APIs externas si es necesario (con caché Caffeine)
6. La respuesta fluye de vuelta por las mismas capas

### Testing — 56 archivos de test

| Paquete | Tests | Cobertura |
|---------|-------|-----------|
| `application/service/` | 20 | Todos los servicios con puertos mockeados (incluye OwnerFilter tests) |
| `domain/model/` | 3 | DeckModelTest, GameTest, MovieShowModelTest, UserModelTest, UserPreferencesTest |
| `infrastructure/adapter/in/web/` | 10 | Controller tests + GlobalExceptionHandlerTest + AuthJwtFlowTest + SecurityMatrixTest + UserProfileControllerTest + UserPreferencesControllerTest + StatsControllerTest |
| `infrastructure/adapter/in/web/dto/` | 4 | Mapper tests (Deck, Game, MagicCard, MovieShow) |
| `infrastructure/adapter/out/persistence/` | 8 | Adapter tests (Book, Game, BoardGame, MagicCard, Deck, MovieShow, User, UserPreferences) + BoardGameStatusMigrationTest + UserPreferencesMigrationTest + UserOwnedBackfillMigrationTest + MongoIndexMigrationTest |
| `infrastructure/adapter/out/{api}/` | 8 | Client tests (Google, RAWG, FreeToGame, BGG, Scryfall, Steam, TMDB, Catbox) + MagicCardMapperTest + BoardGameXmlMapperTest |
| `infrastructure/config/` | 10 | HttpClientPropertiesTest, WebConfigTest, OpenApiConfigTest, SecurityConfigTest, JwtAuthenticationFilterTest, CurrentUserHandlerMethodArgumentResolverTest, StringToEnumConvertersNullSafeTest + 3 converter tests |
| `domain/port/` y otros | 3 | OwnershipValidatorTest, OwnerResolverTest, OwnerScopeResolverTest |

### Notas Importantes

- **Imágenes**: El backend usa **Catbox.moe** (hosting externo), NO filesystem local. `ImageStorageController`, `CatboxClient`, `ImageResponse`, `CatboxUploadException`.
- **MagicCardUseCase**: Solo soporta `search`, `findById`, `addFromScryfall`, `delete`. NO tiene `save` ni `update`.
- **Caché**: Implementada con Spring Cache + Caffeine (en memoria, sin Redis). 44 cachés con TTL por grupo de colección (ver sección «Cache Caffeine»).
- **Expansiones y contenido adicional**: `GET/POST /api/v1/boardgames/{id}/expansions` (BGG) y `GET/POST /api/v1/games/{id}/additions` (RAWG) enlazan registros a un juego base con `parentId`/`parentTitle` inmutables vía PUT y borrado en cascada — ver sección «Expansiones de juegos de mesa y contenido adicional de videojuegos».
- **MongoDB URI**: La propiedad es `spring.mongodb.uri` (NO `spring.data.mongodb.uri`).
- **Lombok + JDK 25**: Requiere configuración específica (ver AGENTS.md del backend).
- **JaCoCo**: Gate mínimo 80% cobertura de línea.
- **Seguridad**: Spring Security + JWT (JJWT 0.12.6). GET endpoints son públicos, POST/PUT/DELETE requieren autenticación.
- **AuthController**: `/api/v1/auth/register`, `/login`, `/refresh`, `/me`. Devuelve `AuthResponse` con accessToken + refreshToken + UserResponse.
- **UserPreferences**: `/api/v1/preferences` con sub-endpoints para active-collections y collection-visibility.
- **Perfiles públicos**: `/api/v1/users/{username}` + sub-recursos por colección. Visibles por anónimos.
- **StatsController**: `/api/v1/stats/global` devuelve totales del sistema.
- **Filtros de visibilidad**: Todos los endpoints de colección soportan parámetro `owner` con valores `mine`/`other`/`all`.
- **Cada entidad de colección tiene `ownerId`** para ownership y visibilidad.
- **UserOwnedBackfillMigration**: Población automática de user_owned para evitar joins.
