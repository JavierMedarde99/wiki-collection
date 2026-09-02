# Backend

## Stack

- **Lenguaje:** Java 25
- **Framework:** Spring Boot 4
- **Base de datos:** MongoDB + Spring Data
- **Build:** Maven
- **Documentación:** springdoc-openapi (Swagger)

## Estructura de Carpetas

```
src/main/java/com/wikicollection/
├── WikiCollectionApplication.java
├── domain/
│   ├── model/
│   │   ├── Book.java
│   │   ├── BookStatus.java
│   │   ├── BookSearchResult.java
│   │   ├── Game.java
│   │   ├── GameStatus.java
│   │   ├── GamePlatform.java
│   │   └── GameSearchResult.java
│   └── port/
│       ├── in/
│       │   ├── BookUseCase.java
│       │   ├── BookSearchUseCase.java
│       │   ├── GameUseCase.java
│       │   └── GameSearchUseCase.java
│       └── out/
│           ├── BookRepository.java
│           ├── ExternalBookCatalogClient.java
│           ├── GameRepository.java
│           └── ExternalGameCatalogClient.java
├── application/
│   └── service/
│       ├── BookService.java
│       ├── BookSearchService.java
│       ├── GameService.java
│       └── GameSearchService.java
└── infrastructure/
    ├── adapter/
    │   ├── in/web/
    │   │   ├── BookController.java
    │   │   ├── GameController.java
    │   │   ├── GlobalExceptionHandler.java
    │   │   └── dto/
    │   │       ├── BookRequest.java
    │   │       ├── BookResponse.java
    │   │       ├── BookDtoMapper.java
    │   │       ├── GameRequest.java
    │   │       ├── GameResponse.java
    │   │       └── GameDtoMapper.java
    │   └── out/
    │       ├── freetogame/
    │       │   └── FreeToGameClient.java
    │       ├── google/
    │       │   └── GoogleBooksClient.java
    │       └── persistence/
    │           ├── BookEntity.java
    │           ├── SpringDataBookRepository.java
    │           ├── BookEntityMapper.java
    │           ├── BookPersistenceAdapter.java
    │           ├── GameEntity.java
    │           ├── SpringDataGameRepository.java
    │           ├── GameEntityMapper.java
    │           └── GamePersistenceAdapter.java
    └── config/
        ├── WebConfig.java
        ├── MongoAuditConfig.java
        ├── RestClientConfig.java
        ├── StringToBookStatusConverter.java
        └── StringToGameStatusConverter.java
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
| GET | /api/games | Listar juegos | ⏳ |
| GET | /api/games/{id} | Obtener juego | ⏳ |
| POST | /api/games | Crear juego | ⏳ |
| PUT | /api/games/{id} | Actualizar juego | ⏳ |
| DELETE | /api/games/{id} | Eliminar juego | ⏳ |
| GET | /api/games/search | Buscar en FreeToGame | ⏳ |
