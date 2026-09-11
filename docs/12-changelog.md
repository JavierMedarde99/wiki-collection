# Changelog

## [Unreleased]

### Added
- Fase 5: Películas/Series con TMDB API
- Fase 4.1: Mazos Commander con Scryfall API
- MovieShowService y MovieSearchService para gestión de películas/series
- TmdbClient para búsqueda en TMDB API
- MovieShowController con endpoints CRUD y búsqueda
- MovieShow DTOs y mapper
- MovieShowEntity y MovieShowPersistenceAdapter
- SpringDataMovieShowRepository
- MovieShowUseCase y MovieSearchUseCase ports
- MovieShow, MovieStatus, MovieMediaType, MovieSearchCriteria, MovieSearchResult modelos
- StringToMovieMediaTypeConverter y StringToMovieStatusConverter
- TmdbClientConfig para RestClient de TMDB
- DeckService, DeckSearchService y DeckValidator para gestión de mazos
- DeckController con endpoints CRUD, gestión de cartas y status
- DeckDtoMapper, DeckEntityMapper, DeckPersistenceAdapter
- DeckEntity, DeckCardEntity, SpringDataDeckRepository
- DeckUseCase, DeckSearchUseCase ports
- Deck, DeckCard, DeckStatus, DeckStatusReport modelos
- ScryfallClient.searchCommanders() para búsqueda de comandantes
- Nuevos componentes frontend: MovieShowCard, MovieShowForm, MovieShowSearch, MovieShowStatusBadge, DeckCommanderImage, MagicCard
- Nuevas páginas frontend: MovieShowListPage, MovieShowCreatePage, MovieShowEditPage, MovieShowDetailPage, DeckListPage, DeckCreatePage, DeckEditPage, DeckDetailPage
- Nuevos tipos TypeScript: MovieShow.ts, MovieShowStatus.ts, MovieType.ts, Deck.ts
- Nuevas constantes: movieshows.ts, decks.ts
- Nuevas APIs: movieshowsApi.ts, deckApi.ts
- Tests completos para MovieShow, Deck, TmdbClient
- Jacoco configurado con 80% cobertura mínima
- springdoc-openapi para documentación Swagger

### Changed
- Modelo de datos de películas/series (MovieShow) con campos: externalId, title, overview, releaseDate, posterUrl, backdropUrl, voteAverage, mediaType, status, userRating, comment, dateAdded, dateCompleted, externalSource
- Modelo de datos de mazos (Deck) con campos: name, description, commander, commanderColors, cards, createdAt, updatedAt
- Wiki dividida en secciones navegables
- Actualización a Spring Boot 4.1.1 y Java 25
- RAWG como API principal para juegos (reemplaza FreeToGame)
- BGG XML como API para juegos de mesa
- Colecciones MongoDB renombradas a minúsculas
- Endpoint base de películas/series renombrado a /api/movieshows
- MovieStatus reducido a WATCHING, WATCHED, PLAN_TO_WATCH (sin WISHLIST)

## [2.0.0] — Fase 2: Juegos

### Added
- Plan de implementación de Fase 2 (videojuegos)
- Integración con RAWG API (principal) y FreeToGame (secundaria)
- Documentación de APIs externas
- GameService, GameSearchService, RAWGClient, FreeToGameClient
- GameController con endpoints CRUD y búsqueda
- GameRequest, GameResponse, GameDtoMapper

## [1.5.0] — Refactor Modelo BOOKS

### Changed
- Modelo de datos de libros actualizado a esquema BOOKS
- Campos renombrados: status→state, description→descripcion, etc.
- Nuevos campos: type, startDate, endDate

### Removed
- Campo isbn (ya no se usa)
- Campo publisher (ya no se usa)
- Campo publishedDate (ya no se usado)

## [1.4.0] — Arquitectura Hexagonal

### Added
- Migración completa a arquitectura hexagonal
- Paquetes domain/, application/, infrastructure/
- Puertos y adaptadores para libros

## [1.3.0] — Frontend CRUD

### Added
- BookListPage: listado con filtros y paginación
- BookCreatePage: formulario de creación
- BookEditPage: formulario de edición
- BookSearchPage: búsqueda vía Google Books
- Componentes reutilizables: BookCard, BookForm, StatusBadge, StarRating, ConfirmDialog

## [1.2.0] — Tests

### Added
- Tests unitarios de BookService
- Tests de integración de BookController
- Tests de GoogleBooksClient con MockWebServer

## [1.1.0] — Backend CRUD

### Added
- BookController con endpoints CRUD
- BookService con lógica de negocio
- GoogleBooksClient para búsqueda externa
- BookPersistenceAdapter para MongoDB

## [1.0.0] — Fase 1: Libros

### Added
- Configuración inicial Spring Boot 4 + Java 25
- Modelo de datos Book
- Repositorio Spring Data MongoDB
- Documentación inicial
