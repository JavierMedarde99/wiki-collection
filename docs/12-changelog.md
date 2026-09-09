# Changelog

## [Unreleased]

### Added
- Fase 3: Juegos de mesa con BoardGameGeek (JSON + XML)
- Fase 4: Cartas Magic con Scryfall API
- GameAchievementsService para logros de Steam
- SteamAchievementsClient con soporte para Store search, Player achievements y Game schema
- BoardGameSearchService con fallback BGG JSON → BGG XML
- MagicCardSearchService para búsqueda en Scryfall
- Nuevos componentes frontend: GameCard, GameForm, GameSearch, GamePlatformBadge, GameStatusBadge, ErrorBanner
- Nuevas páginas frontend: GameListPage, GameCreatePage, GameEditPage, GameAchievementsPage
- Tests completos para BoardGame, MagicCard y GameAchievements
- Jacoco configurado con 80% cobertura mínima
- springdoc-openapi para documentación Swagger

### Changed
- Modelo de datos de juegos actualizado con nuevos campos (externalSource, steamAppId)
- Wiki dividida en secciones navegables
- Actualización a Spring Boot 4.1.1 y Java 25
- RAWG como API principal para juegos (reemplaza FreeToGame)
- BGG JSON como API principal para juegos de mesa (reemplaza XML)

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
