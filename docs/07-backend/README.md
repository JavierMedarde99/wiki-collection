# Backend

## Stack

- **Lenguaje:** Java 25
- **Framework:** Spring Boot 4.1.1
- **Base de datos:** MongoDB + Spring Data
- **Build:** Maven
- **Documentación:** springdoc-openapi (Swagger 3.1.0)
- **Testing:** JUnit 5 + Mockito + MockWebServer + Jacoco (80% cobertura mínima)
- **HTTP Client:** RestClient (Spring) + RestTemplate (para BGG y Scryfall)
- **XML Parsing:** Jackson XML
- **Seguridad:** Spring Security + JJWT 0.12.6
- **Caché:** Spring Cache + Caffeine
- **Lombok:** Sí (1.18.46)

## Estructura de Carpetas

```
src/main/java/com/wikicollection/
├── WikiCollectionApplication.java
├── domain/
│   ├── model/
│   │   ├── Book.java
│   │   ├── BookState.java
│   │   ├── BookType.java
│   │   ├── BookSearchCriteria.java
│   │   ├── BookSearchResult.java
│   │   ├── Game.java
│   │   ├── GameStatus.java
│   │   ├── GamePlatform.java
│   │   ├── GameSearchCriteria.java
│   │   ├── GameSearchResult.java
│   │   ├── BoardGame.java
│   │   ├── BoardGameStatus.java
│   │   ├── BoardGameSearchCriteria.java
│   │   ├── BoardGameSearchResult.java
│   │   ├── MagicCard.java
│   │   ├── MagicCardCondition.java
│   │   ├── MagicCardLanguage.java
│   │   ├── MagicCardSearchCriteria.java
│   │   ├── MagicCardSearchResult.java
│   │   ├── MovieShow.java
│   │   ├── MovieStatus.java
│   │   ├── MovieMediaType.java
│   │   ├── MovieSearchCriteria.java
│   │   ├── MovieSearchResult.java
│   │   ├── Deck.java
│   │   ├── DeckCard.java
│   │   ├── DeckStatus.java
│   │   ├── DeckStatusReport.java
│   │   ├── AchievementsSummary.java
│   │   ├── SteamAchievement.java
│   │   ├── User.java
│   │   ├── UserOwned.java
│   │   ├── UserPreferences.java
│   │   ├── CollectionType.java
│   │   ├── CollectionVisibility.java
│   │   ├── AuthSession.java
│   │   └── AuthTokens.java
│   └── port/
│       ├── in/
│       │   ├── BookUseCase.java
│       │   ├── BookSearchUseCase.java
│       │   ├── GameUseCase.java
│       │   ├── GameSearchUseCase.java
│       │   ├── GameAchievementsUseCase.java
│       │   ├── BoardGameUseCase.java
│       │   ├── BoardGameSearchUseCase.java
│       │   ├── MagicCardUseCase.java
│       │   ├── MagicCardSearchUseCase.java
│       │   ├── MovieShowUseCase.java
│       │   ├── MovieSearchUseCase.java
│       │   ├── DeckUseCase.java
│       │   ├── DeckSearchUseCase.java
│       │   ├── AuthUseCase.java
│       │   ├── UserUseCase.java
│       │   ├── UserPreferencesUseCase.java
│       │   ├── UserProfileUseCase.java
│       │   └── StatsUseCase.java
│       └── out/
│           ├── BookRepository.java
│           ├── ExternalBookCatalogClient.java
│           ├── GameRepository.java
│           ├── ExternalGameCatalogClient.java
│           ├── BoardGameRepository.java
│           ├── ExternalBoardGameCatalogClient.java
│           ├── MagicCardRepository.java
│           ├── ExternalMagicCardCatalogClient.java
│           ├── MovieShowRepository.java
│           ├── ExternalMovieCatalogClient.java
│           ├── DeckRepository.java
│           ├── ImageHostingClient.java
│           ├── SteamCatalogueClient.java
│           ├── UserRepository.java
│           ├── UserPreferencesRepository.java
│           ├── UserProfilePort.java
│           └── SteamClient.java
├── application/
│   ├── exception/
│   │   ├── BookConflictException.java
│   │   ├── BookNotFoundException.java
│   │   ├── GameNotFoundException.java
│   │   ├── BoardGameNotFoundException.java
│   │   ├── MagicCardNotFoundException.java
│   │   ├── MovieShowConflictException.java
│   │   ├── MovieShowNotFoundException.java
│   │   ├── DeckNotFoundException.java
│   │   ├── CatboxUploadException.java
│   │   ├── UserAlreadyExistsException.java
│   │   ├── EmailAlreadyExistsException.java
│   │   ├── UserNotFoundException.java
│   │   ├── ForbiddenException.java
│   │   ├── UnauthenticatedException.java
│   │   └── InvalidTokenException.java
│   └── service/
│       ├── BookService.java
│       ├── BookSearchService.java
│       ├── GameService.java
│       ├── GameSearchService.java
│       ├── BoardGameService.java
│       ├── BoardGameSearchService.java
│       ├── MagicCardService.java
│       ├── MovieShowService.java
│       ├── MovieSearchService.java
│       ├── DeckService.java
│       ├── DeckSearchService.java
│       ├── DeckValidator.java
│       ├── DateRangeValidator.java
│       ├── ImageStorageService.java
│       ├── AuthService.java
│       ├── JwtService.java
│       ├── UserDetailsServiceImpl.java
│       ├── UserPreferencesService.java
│       ├── UserProfileService.java
│       ├── StatsService.java
│       ├── OwnerResolver.java
│       ├── OwnerScopeResolver.java
│       ├── OwnershipValidator.java
│       └── UserPrincipal.java
└── infrastructure/
    ├── adapter/
    │   ├── in/web/
    │   │   ├── BookController.java
    │   │   ├── GameController.java
    │   │   ├── BoardGameController.java
    │   │   ├── MagicCardController.java
    │   │   ├── MovieShowController.java
    │   │   ├── DeckController.java
    │   │   ├── ImageStorageController.java
    │   │   ├── AuthController.java
    │   │   ├── UserProfileController.java
    │   │   ├── UserPreferencesController.java
    │   │   ├── StatsController.java
    │   │   ├── ResponseVisibility.java
    │   │   ├── GlobalExceptionHandler.java
    │   │   └── dto/
    │   │       ├── BookRequest.java, BookResponse.java, BookDtoMapper.java
    │   │       ├── GameRequest.java, GameResponse.java, GameDtoMapper.java
    │   │       ├── GameAchievementMapper.java, GameAchievementResponse.java
    │   │       ├── AchievementsResponse.java
    │   │       ├── BoardGameRequest.java, BoardGameResponse.java, BoardGameDtoMapper.java
    │   │       ├── BoardGameSearchResponse.java
    │   │       ├── MagicCardResponse.java, MagicCardSearchResponse.java, MagicCardDtoMapper.java
    │   │       ├── MovieShowRequest.java, MovieShowResponse.java, MovieShowDtoMapper.java
    │   │       ├── DeckRequest.java, DeckResponse.java, DeckCardRequest.java
    │   │       ├── DeckCardResponse.java, DeckDtoMapper.java, DeckStatusResponse.java
    │   │       ├── ImageRequest.java, ImageResponse.java
    │   │       ├── AuthRequest.java, AuthResponse.java, RegisterRequest.java
    │   │       ├── LoginRequest.java, RefreshTokenRequest.java
    │   │       ├── UserResponse.java, PublicProfileResponse.java
    │   │       ├── UserPreferencesRequest.java, UserPreferencesResponse.java
    │   │       ├── CollectionVisibilityRequest.java, ActiveCollectionsRequest.java
    │   │       └── GlobalStatsResponse.java
    │   └── out/
    │       ├── persistence/
    │       │   ├── BookEntity.java, BookEntityMapper.java, BookPersistenceAdapter.java
    │       │   ├── SpringDataBookRepository.java
    │       │   ├── GameEntity.java, GameEntityMapper.java, GamePersistenceAdapter.java
    │       │   ├── SpringDataGameRepository.java
    │       │   ├── BoardGameEntity.java, BoardGameEntityMapper.java, BoardGamePersistenceAdapter.java
    │       │   ├── SpringDataBoardGameRepository.java
    │       │   ├── MagicCardEntity.java, MagicCardEntityMapper.java, MagicCardPersistenceAdapter.java
    │       │   ├── SpringDataMagicCardRepository.java
    │       │   ├── MovieShowEntity.java, MovieShowEntityMapper.java, MovieShowPersistenceAdapter.java
    │       │   ├── SpringDataMovieShowRepository.java
    │       │   ├── DeckEntity.java, DeckEntityMapper.java, DeckPersistenceAdapter.java
    │       │   ├── DeckCardEntity.java
    │       │   ├── SpringDataDeckRepository.java
    │       │   ├── UserEntity.java, UserEntityMapper.java, UserPersistenceAdapter.java
    │       │   ├── SpringDataUserRepository.java
    │       │   ├── UserPreferencesEntity.java, UserPreferencesEntityMapper.java
    │       │   ├── UserPreferencesPersistenceAdapter.java
    │       │   ├── SpringDataUserPreferencesRepository.java
    │       │   ├── UserOwnedEntity.java, UserOwnedMapping.java
    │       │   └── UserProfilePersistenceAdapter.java
    │       ├── google/
    │       │   └── GoogleBooksClient.java
    │       ├── rawg/
    │       │   └── RAWGClient.java
    │       ├── freetogame/
    │       │   └── FreeToGameClient.java
    │       ├── steam/
    │       │   └── SteamAchievementsClient.java
    │       ├── bgg/
    │       │   ├── xml/
    │       │   │   └── BggXmlClient.java
    │       │   └── mapper/
    │       │       └── BoardGameXmlMapper.java
    │       ├── scryfall/
    │       │   ├── ScryfallClient.java
    │       │   └── MagicCardMapper.java
    │       ├── tmdb/
    │       │   └── TmdbClient.java
    │       └── catbox/
    │           └── CatboxClient.java
    └── config/
        ├── AuthDataMigration.java
        ├── BggClientConfig.java
        ├── CacheConfig.java
        ├── CacheProperties.java
        ├── CurrentUser.java
        ├── CurrentUserHandlerMethodArgumentResolver.java
        ├── HttpClientProperties.java
        ├── JwtAuthenticationFilter.java
        ├── MongoAuditConfig.java
        ├── MongoIndexMigration.java
        ├── OpenApiConfig.java
        ├── RateLimitFilter.java
        ├── RestClientConfig.java
        ├── ScryfallClientConfig.java
        ├── SecurityConfig.java
        ├── TmdbClientConfig.java
        ├── StringToBoardGameStatusConverter.java
        ├── StringToBookStateConverter.java
        ├── StringToGameStatusConverter.java
        ├── StringToMovieMediaTypeConverter.java
        ├── StringToMovieStatusConverter.java
        ├── UserOwnedBackfillMigration.java
        ├── UserPreferencesMigration.java
        └── WebConfig.java
```

## Endpoints Implementados (34 endpoints + Auth)

### Libros (BookController — /api/v1/books)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/books | Listar libros (pagina, filtra por name/author/type/state/owner) ||
|| GET | /api/v1/books/{id} | Obtener libro ||
|| POST | /api/v1/books | Crear libro ||
|| PUT | /api/v1/books/{id} | Actualizar libro ||
|| DELETE | /api/v1/books/{id} | Eliminar libro ||
|| GET | /api/v1/books/search?name= | Buscar en Google Books ||

### Juegos (GameController — /api/v1/games)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/games | Listar juegos (pagina, filtra por name/platform/status/owner) ||
|| GET | /api/v1/games/{id} | Obtener juego ||
|| POST | /api/v1/games | Crear juego ||
|| PUT | /api/v1/games/{id} | Actualizar juego ||
|| DELETE | /api/v1/games/{id} | Eliminar juego ||
|| GET | /api/v1/games/search?name= | Buscar en RAWG (+ fallback FreeToGame) ||
|| GET | /api/v1/games/{id}/achievements?steamId= | Logros Steam ||

### Juegos de Mesa (BoardGameController — /api/v1/boardgames)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/boardgames | Listar juegos de mesa ||
|| GET | /api/v1/boardgames/{id} | Obtener juego de mesa ||
|| POST | /api/v1/boardgames | Crear juego de mesa ||
|| PUT | /api/v1/boardgames/{id} | Actualizar juego de mesa ||
|| DELETE | /api/v1/boardgames/{id} | Eliminar juego de mesa ||
|| GET | /api/v1/boardgames/search?name= | Buscar en BGG ||

### Magic (MagicCardController — /api/v1/magic)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/magic | Listar cartas Magic ||
|| GET | /api/v1/magic/{id} | Obtener carta ||
|| POST | /api/v1/magic/scryfall/{scryfallId} | Añadir carta desde Scryfall ||
|| DELETE | /api/v1/magic/{id} | Eliminar carta ||
|| GET | /api/v1/magic/search?name= | Buscar en Scryfall ||
|| GET | /api/v1/magic/commanders?colors= | Buscar comandantes por colores ||

### Mazos (DeckController — /api/v1/decks)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/decks | Listar mazos ||
|| GET | /api/v1/decks/{id} | Obtener mazo ||
|| POST | /api/v1/decks | Crear mazo ||
|| PUT | /api/v1/decks/{id} | Actualizar mazo ||
|| DELETE | /api/v1/decks/{id} | Eliminar mazo ||
|| POST | /api/v1/decks/{id}/cards | Añadir carta al mazo ||
|| DELETE | /api/v1/decks/{id}/cards/{scryfallId} | Quitar carta del mazo ||
|| GET | /api/v1/decks/{id}/status | Estado del mazo (DRAFT/COMPLETE/INVALID) ||

### Películas/Series (MovieShowController — /api/v1/movieshows)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/movieshows | Listar películas/series ||
|| GET | /api/v1/movieshows/{id} | Obtener película/serie ||
|| POST | /api/v1/movieshows | Crear película/serie ||
|| PUT | /api/v1/movieshows/{id} | Actualizar película/serie ||
|| DELETE | /api/v1/movieshows/{id} | Eliminar película/serie ||
|| GET | /api/v1/movieshows/search?name= | Buscar en TMDB ||

### Imágenes (ImageStorageController — /api/v1/images)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| POST | /api/v1/images/upload | Subir imagen a Catbox ||
|| DELETE | /api/v1/images/{filename} | Eliminar imagen de Catbox ||

### Autenticación (AuthController — /api/v1/auth)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| POST | /api/v1/auth/register | Registrar usuario ||
|| POST | /api/v1/auth/login | Login (devuelve accessToken + refreshToken) ||
|| POST | /api/v1/auth/refresh | Refrescar accessToken ||
|| GET | /api/v1/auth/me | Obtener usuario actual ||

### Perfil Público (UserProfileController — /api/v1/users)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/users/{username} | Perfil público de usuario ||
|| GET | /api/v1/users/{username}/books | Libros públicos ||
|| GET | /api/v1/users/{username}/games | Juegos públicos ||
|| GET | /api/v1/users/{username}/boardgames | Juegos de mesa públicos ||
|| GET | /api/v1/users/{username}/magic | Cartas Magic públicas ||
|| GET | /api/v1/users/{username}/decks | Mazos públicos ||
|| GET | /api/v1/users/{username}/movieshows | Películas/series públicas ||

### Preferencias (UserPreferencesController — /api/v1/preferences)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/preferences | Obtener preferencias del usuario ||
|| PUT | /api/v1/preferences | Actualizar preferencias ||
|| PATCH | /api/v1/preferences/active-collections | Activar/desactivar colecciones ||
|| PATCH | /api/v1/preferences/collection-visibility | Cambiar visibilidad ||
|| GET | /api/v1/preferences/active-collections | Colecciones activas ||

### Estadísticas (StatsController — /api/v1/stats)
|| Método | Endpoint | Descripción ||
||--------|----------|-------------||
|| GET | /api/v1/stats/global | Estadísticas globales ||

## Excepciones (14 excepciones)

|| Excepción | HTTP | Descripción ||
||-----------|------|-----||
|| BookNotFoundException | 404 | Libro no encontrado ||
|| BookConflictException | 409 | Conflicto al guardar libro ||
|| GameNotFoundException | 404 | Juego no encontrado ||
|| BoardGameNotFoundException | 404 | Juego de mesa no encontrado ||
|| MagicCardNotFoundException | 404 | Carta Magic no encontrada ||
|| MovieShowNotFoundException | 404 | Película/serie no encontrada ||
|| MovieShowConflictException | 409 | Conflicto al guardar película/serie ||
|| DeckNotFoundException | 404 | Mazo no encontrado ||
|| CatboxUploadException | 502 | Error al subir imagen a Catbox ||
|| UserAlreadyExistsException | 409 | El username ya existe ||
|| EmailAlreadyExistsException | 409 | El email ya está registrado ||
|| UserNotFoundException | 404 | Usuario no encontrado ||
|| ForbiddenException | 403 | Acceso denegado (ownership) ||
|| UnauthenticatedException | 401 | Token inválido o expirado ||
|| InvalidTokenException | 401 | Token JWT inválido ||

## Configuración (22 clases de config)

|| Clase | Responsabilidad ||
||-------|----------------||
|| BggClientConfig | Configuración RestTemplate para BGG XML API ||
|| CacheConfig | @EnableCaching, beans de Caffeine (6 cachés) ||
|| CacheProperties | @ConfigurationProperties para TTL y maxSize de cachés ||
|| CurrentUser | @interface para inyectar userId en controladores ||
|| CurrentUserHandlerMethodArgumentResolver | Resolver para @CurrentUser ||
|| HttpClientProperties | Timeouts de conexión y lectura ||
|| JwtAuthenticationFilter | Filtro OncePerRequestFilter para Bearer tokens ||
|| MongoAuditConfig | @EnableMongoAuditing (única instancia) ||
|| MongoIndexMigration | CommandLineRunner para índices ||
|| OpenApiConfig | springdoc-openapi: título, versión, seguridad ||
|| RateLimitFilter | Filtro de rate limiting por IP (mapa en memoria) ||
|| RestClientConfig | Beans RestClient para Google, RAWG, FreeToGame, Steam, Catbox, TMDB ||
|| ScryfallClientConfig | Bean RestTemplate para Scryfall ||
|| SecurityConfig | @EnableWebSecurity + @EnableMethodSecurity, JWT, CORS ||
|| TmdbClientConfig | Bean RestClient para TMDB ||
|| StringToBoardGameStatusConverter | Conversor String→BoardGameStatus ||
|| StringToBookStateConverter | Conversor String→BookState ||
|| StringToGameStatusConverter | Conversor String→GameStatus ||
|| StringToMovieMediaTypeConverter | Conversor String→MovieMediaType ||
|| StringToMovieStatusConverter | Conversor String→MovieStatus ||
|| UserOwnedBackfillMigration | CommandLineRunner para poblar user_owned ||
|| UserPreferencesMigration | CommandLineRunner para crear preferencias por defecto ||
|| WebConfig | CORS: /api/** → orígenes configurables ||
|| AuthDataMigration | CommandLineRunner: crear admin por defecto ||

## Caché Caffeine (6 cachés)

|| Caché | TTL | Max Size | Uso ||
||-------|-----|----------|-----||
|| bookSearch | Configurable | 500 | Google Books resultados ||
|| gameSearch | Configurable | 500 | RAWG + FreeToGame resultados ||
|| boardgameSearch | Configurable | 500 | BGG resultados ||
|| magicSearch | Configurable | 500 | Scryfall resultados ||
|| commanderSearch | Configurable | 500 | Scryfall comandantes ||
|| movieSearch | Configurable | 500 | TMDB resultados ||
