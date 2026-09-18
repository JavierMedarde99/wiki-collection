# Changelog

## [Unreleased]

### Added
- Documentación de despliegue: guía completa para MongoDB Atlas + Render (backend) + Vercel (frontend)
- Variables de entorno requeridas para producción (JWT_SECRET, MONGODB_URI, API keys externas, CORS)
- Configuración de CORS para conectar frontend Vercel con backend Render
- Pipeline de despliegue inicial: orden recomendado (Atlas → Backend → Frontend)
- Checklist de despliegue con 20 verificacións
- Costes reales: planes gratis ($0/mes) vs upgrade (Render ~$7/mes, Atlas M10 ~$57/mes)
- Migración de datos locales a Atlas (mongodump/mongorestore + Compass + script Java)
- Rollback y troubleshooting (backend no arranca, CORS, JWT, MongoDB connection)
- Notas importantes: Render free tier idle (cold start 30-60s), JDK 25 vía Dockerfile, JWT_SECRET estable entre despliegues
- Documentación de Catbox como sistema de almacenamiento de imágenes (reemplaza filesystem local)
- Documentación de ImageStorageController, CatboxClient, ImageResponse, CatboxUploadException
- Documentación de DeckController, DeckService, DeckSearchService, DeckValidator
- Documentación de MovieShowController, MovieShowService, MovieSearchService, TmdbClient
- Documentación de todos los modelos de dominio: Deck, DeckCard, DeckStatus, DeckStatusReport, MovieShow, MovieMediaType, MovieStatus
- Documentación de todos los puertos: DeckUseCase, DeckSearchUseCase, MovieShowUseCase, MovieSearchUseCase, DeckRepository, MovieShowRepository, ImageHostingClient
- Documentación de CacheConfig, CacheProperties, WebConfig, HttpClientProperties
- Documentación de BoardGameStatusMigration, PagedResults, DateRangeValidator
- Documentación de todos los DTOs: DeckRequest, DeckResponse, DeckCardRequest, DeckCardResponse, DeckStatusResponse, DeckDtoMapper, MovieShowRequest, MovieShowResponse, MovieShowDtoMapper, ImageResponse, ErrorResponse
- Documentación de persistencia completa: entidades, mappers, adaptadores para Deck y MovieShow
- Documentación de repositorios Spring Data para Deck y MovieShow
- Documentación de tests nuevos: CatboxClientTest, HttpClientPropertiesTest, WebConfigTest, GlobalExceptionHandlerTest, ImageStorageServiceTest, ImageStorageControllerTest
- Documentación de rutas frontend completas (25 rutas)
- Documentación de componentes frontend nuevos: BoardGameCard, BoardGameForm, BoardGameSearch, BoardGameStatusBadge, MagicCard, ManaColorDots, DeckCommanderImage, Breadcrumbs, GlobalSearch, HelpModal, ImageUpload, ThemeToggle, Toast, SearchField, SortSelect, FilterPill, Pagination, ActionLink, CardMenu, ExportButton, FormSection
- Documentación de hooks frontend: useBackFallback, useInfiniteScroll, useListQuery, usePagedList, usePageTitle, useSearchShortcut, useUnsavedGuard
- Documentación de páginas frontend: todas las de BoardGame, Magic, Deck, MovieShow

### Changed
- Wiki sincronizada con el backend real (ya no dice filesystem local para imágenes, usa Catbox)
- MagicCardUseCase corregido: solo tiene search, findById, addFromScryfall, delete (NO save/update)
- Endpoints REST documentados con ruta base /api/v1
- Todos los endpoints de Deck, MovieShow, BoardGame, Magic e Imágenes añadidos
- 56 archivos de test backend documentados (antes ~40)
- 5 archivos de test frontend documentados
- Estructura de paquetes backend completa y actualizada (30 modelos, 13 puertos, 17 servicios, 9 excepciones, 14 configs)
- DTOs documentados de todas las entidades
- Excepciones documentadas con códigos HTTP

## [3.0.0] — Fase 5: Películas/Series + Fase 6: Imágenes + Fase 4.1: Mazos

### Added
- Fase 5: Películas/Series con TMDB API
- Fase 4.1: Mazos Commander con Scryfall API
- Fase 6: Imágenes con Catbox.moe
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
- ImageStorageService, CatboxClient, ImageStorageController
- Nuevos componentes frontend: MovieShowCard, MovieShowForm, MovieShowSearch, MovieShowStatusBadge, DeckCommanderImage, MagicCard
- Nuevas páginas frontend: MovieShowListPage, MovieShowCreatePage, MovieShowEditPage, MovieShowDetailPage, DeckListPage, DeckCreatePage, DeckEditPage, DeckDetailPage
- Nuevos tipos TypeScript: MovieShow.ts, MovieShowStatus.ts, MovieType.ts, Deck.ts
- Nuevas constantes: movieshows.ts, decks.ts
- Nuevas APIs: movieshowsApi.ts, deckApi.ts, imagesApi.ts
- Tests completos para MovieShow, Deck, TmdbClient, CatboxClient
- Jacoco configurado con 80% cobertura mínima
- springdoc-openapi para documentación Swagger

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
