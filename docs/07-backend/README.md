# Backend

## Stack

- **Lenguaje:** Java 25
- **Framework:** Spring Boot 4.1.1
- **Base de datos:** MongoDB + Spring Data
- **Build:** Maven
- **Documentación:** springdoc-openapi (Swagger 3.1.0)
- **Testing:** JUnit 5 + Mockito + MockWebServer + Jacoco (80% cobertura mínima)
- **HTTP Client:** RestClient (Spring) + RestTemplate (para APIs externas)
- **XML Parsing:** Jackson XML
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
│   │   └── SteamAchievement.java
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
│       │   └── DeckSearchUseCase.java
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
│           ├── SteamCatalogueClient.java
│           ├── DeckRepository.java
│           └── (alias de ExternalMagicCardCatalogClient)
├── application/
│   ├── exception/
│   │   ├── BookConflictException.java
│   │   ├── BookNotFoundException.java
│   │   ├── GameNotFoundException.java
│   │   ├── BoardGameNotFoundException.java
│   │   ├── MagicCardNotFoundException.java
│   │   ├── MovieShowConflictException.java
│   │   ├── MovieShowNotFoundException.java
│   │   └── DeckNotFoundException.java
│   └── service/
│       ├── BookService.java
│       ├── BookSearchService.java
│       ├── GameService.java
│       ├── GameSearchService.java
│       ├── GameAchievementsService.java
│       ├── BoardGameService.java
│       ├── BoardGameSearchService.java
│       ├── MagicCardService.java
│       ├── MagicCardSearchService.java
│       ├── MovieShowService.java
│       ├── MovieSearchService.java
│       ├── DeckService.java
│       ├── DeckSearchService.java
│       ├── DeckValidator.java
│       └── DateRangeValidator.java
└── infrastructure/
    ├── adapter/
    │   ├── in/web/
    │   │   ├── BookController.java
    │   │   ├── GameController.java
    │   │   ├── BoardGameController.java
    │   │   ├── MagicCardController.java
    │   │   ├── MovieShowController.java
    │   │   ├── DeckController.java
    │   │   ├── GlobalExceptionHandler.java
    │   │   └── dto/
    │   │       ├── BookRequest.java
    │   │       ├── BookResponse.java
    │   │       ├── BookDtoMapper.java
    │   │       ├── GameRequest.java
    │   │       ├── GameResponse.java
    │   │       ├── GameDtoMapper.java
    │   │       ├── GameAchievementMapper.java
    │   │       ├── GameAchievementResponse.java
    │   │       ├── AchievementsResponse.java
    │   │       ├── BoardGameRequest.java
    │   │       ├── BoardGameResponse.java
    │   │       ├── BoardGameSearchResponse.java
    │   │       ├── BoardGameDtoMapper.java
    │   │       ├── MagicCardResponse.java
    │   │       ├── MagicCardSearchResponse.java
    │   │       ├── MagicCardDtoMapper.java
    │   │       ├── MovieShowRequest.java
    │   │       ├── MovieShowResponse.java
    │   │       ├── MovieShowDtoMapper.java
    │   │       ├── DeckRequest.java
    │   │       ├── DeckResponse.java
    │   │       ├── DeckCardRequest.java
    │   │       ├── DeckCardResponse.java
    │   │       ├── DeckDtoMapper.java
    │   │       ├── DeckStatusResponse.java
    │   │       └── ErrorResponse.java
    │   └── out/
    │       ├── persistence/
    │       │   ├── BookEntity.java
    │       │   ├── BookEntityMapper.java
    │       │   ├── BookPersistenceAdapter.java
    │       │   ├── SpringDataBookRepository.java
    │       │   ├── GameEntity.java
    │       │   ├── GameEntityMapper.java
    │       │   ├── GamePersistenceAdapter.java
    │       │   ├── SpringDataGameRepository.java
    │       │   ├── BoardGameEntity.java
    │       │   ├── BoardGameEntityMapper.java
    │       │   ├── BoardGamePersistenceAdapter.java
    │       │   ├── SpringDataBoardGameRepository.java
    │       │   ├── MagicCardEntity.java
    │       │   ├── MagicCardEntityMapper.java
    │       │   ├── MagicCardPersistenceAdapter.java
    │       │   ├── SpringDataMagicCardRepository.java
    │       │   ├── MovieShowEntity.java
    │       │   ├── MovieShowEntityMapper.java
    │       │   ├── MovieShowPersistenceAdapter.java
    │       │   ├── SpringDataMovieShowRepository.java
    │       │   ├── DeckEntity.java
    │       │   ├── DeckEntityMapper.java
    │       │   ├── DeckPersistenceAdapter.java
    │       │   ├── SpringDataDeckRepository.java
    │       │   └── BoardGameStatusMigration.java
    │       ├── google/
    │       │   └── GoogleBooksClient.java
    │       ├── rawg/
    │       │   └── RAWGClient.java
    │       ├── freetogame/
    │       │   └── FreeToGameClient.java
    │       ├── bgg/
    │       │   ├── xml/
    │       │   │   └── BggXmlClient.java
    │       │   └── mapper/
    │       │       └── BoardGameXmlMapper.java
    │       ├── scryfall/
    │       │   ├── ScryfallClient.java
    │       │   └── MagicCardMapper.java
    │       ├── steam/
    │       │   └── SteamAchievementsClient.java
    │       └── tmdb/
    │           └── TmdbClient.java
    └── config/
        ├── BggClientConfig.java
        ├── MongoAuditConfig.java
        ├── RestClientConfig.java
        ├── ScryfallClientConfig.java
        ├── StringToBoardGameStatusConverter.java
        ├── StringToBookStateConverter.java
        ├── StringToGameStatusConverter.java
        ├── StringToMovieMediaTypeConverter.java
        ├── StringToMovieStatusConverter.java
        ├── TmdbClientConfig.java
        └── WebConfig.java
```

## Endpoints Implementados

| Método | Endpoint | Descripción | Estado |
|--------|----------|-------------|--------|
| GET | /api/books | Listar libros | ✅ |
| GET | /api/books/{id} | Obtener libro | ✅ |
| POST | /api/books | Crear libro | ✅ |
| PUT | /api/books/{id} | Actualizar libro | ✅ |
| DELETE | /api/books/{id} | Eliminar libro | ✅ |
| GET | /api/books/search | Buscar en Google Books | ✅ |
| GET | /api/games | Listar juegos | ✅ |
| GET | /api/games/{id} | Obtener juego | ✅ |
| POST | /api/games | Crear juego | ✅ |
| PUT | /api/games/{id} | Actualizar juego | ✅ |
| DELETE | /api/games/{id} | Eliminar juego | ✅ |
| GET | /api/games/search | Buscar en RAWG/FreeToGame | ✅ |
| GET | /api/games/{id}/achievements | Logros de Steam | ✅ |
| GET | /api/boardgames | Listar juegos de mesa | ✅ |
| GET | /api/boardgames/{id} | Obtener juego de mesa | ✅ |
| POST | /api/boardgames | Crear juego de mesa | ✅ |
| PUT | /api/boardgames/{id} | Actualizar juego de mesa | ✅ |
| DELETE | /api/boardgames/{id} | Eliminar juego de mesa | ✅ |
| GET | /api/boardgames/search | Buscar en BGG | ✅ |
| GET | /api/magic | Listar cartas Magic | ✅ |
| GET | /api/magic/{id} | Obtener carta | ✅ |
| DELETE | /api/magic/{id} | Eliminar carta | ✅ |
| GET | /api/magic/search | Buscar en Scryfall | ✅ |
| GET | /api/decks | Listar mazos | ✅ |
| GET | /api/decks/{id} | Obtener mazo | ✅ |
| POST | /api/decks | Crear mazo | ✅ |
| PUT | /api/decks/{id} | Actualizar mazo | ✅ |
| DELETE | /api/decks/{id} | Eliminar mazo | ✅ |
| POST | /api/decks/{id}/cards | Añadir carta | ✅ |
| DELETE | /api/decks/{id}/cards/{scryfallId} | Quitar carta | ✅ |
| GET | /api/decks/{id}/status | Estado del mazo | ✅ |
| GET | /api/movieshows | Listar películas/series | ✅ |
| GET | /api/movieshows/{id} | Obtener película/serie | ✅ |
| POST | /api/movieshows | Crear película/serie | ✅ |
| PUT | /api/movieshows/{id} | Actualizar película/serie | ✅ |
| DELETE | /api/movieshows/{id} | Eliminar película/serie | ✅ |
| GET | /api/movieshows/search | Buscar en TMDB | ✅ |
