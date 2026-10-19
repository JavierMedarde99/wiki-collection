---
type: changelog
title: "Changelog — Wiki-Collection"
description: "Historial de cambios de la documentación"
tags: [changelog]
---

# Changelog — Wiki-Collection

## [Unreleased]

### Added
- **Investigación de las 11 APIs externas**: fichas de `research/external-apis/` actualizadas y verificadas en vivo el 2026-10-15 (Google Books, Open Library, RAWG, FreeToGame, Steam, BGG XML, Scryfall, TMDB, Catbox, HobbyQuantHQ, AmiiboAPI)
- Ficha nueva **`openlibrary.md`** (el enriquecedor de libros por ISBN, que no estaba documentado)
- Tabla de APIs externas en `README.md` con enlaces a cada ficha y nota de que las 4 APIs con clave solo se pudieron verificar sin credenciales

- **Diseño de expansiones de juegos de mesa**: documento nuevo `research/future/boardgame-expansions.md` con el diseño cerrado (expansión como entrada propia enlazada a su juego base, que debe estar en `OWNED`; campos `bggType`/`parentId`/`parentTitle`; endpoints `GET` y `POST /boardgames/{id}/expansions`; filtro `?type=`; `502` explícito cuando BGG falla)
- Ficha `research/external-apis/boardgamegeek.md`: documentados el atributo `item/@type`, los *thing types* de BGG y la semántica de `inbound` en los links de expansión

- **Diseño de DLC y contenido adicional de videojuegos**: documento nuevo `research/future/game-dlcs.md` (el DLC es una entrada propia enlazada a su juego base, que no puede estar en `WISHLIST`; campos `parentId`/`parentTitle`; endpoints `GET` y `POST /games/{id}/additions`; filtro `?type=`; y vinculación de los DLC que ya estuvieran sueltos en la colección)
- Ficha `research/external-apis/rawg.md`: documentados los filtros `exclude_additions`/`exclude_parents`, el endpoint `/games/{game_pk}/additions` y los campos `parents_count`/`additions_count`

### Fixed
- **La wiki se leía contra los checkouts locales, que están obsoletos**: el backend está en un `HEAD` suelto y el frontend en una rama ya borrada del remoto. Eso dio por no implementado lo que `origin/main` ya había publicado (las expansiones de juegos de mesa) y falseó el recuento de cachés. `AGENTS.md` ahora lo advierte y documenta cómo leer `origin/main` con `git show`
- **Controllers**: `AGENTS.md` decía 11, son **13** (`amiibos` y `funkos` ya existen)
- **Número de cachés Caffeine**: la wiki decía **40** y una comprobación inicial contra el checkout local dijo **33**; las dos cifras eran erróneas. Sobre `origin/main` son **43** (`CacheConfig.CACHE_NAMES`, 2026-10-19). Corregido en `AGENTS.md` (con el recuento completo y el aviso de verificar siempre sobre `origin/main`), en `research/future/boardgame-expansions.md`, en `research/future/game-dlcs.md` y en los issues backend-collection#495 y #505
- **`research/future/boardgame-expansions.md` e `research/future/index.md`**: la feature estaba implementada pero la wiki la daba por «Investigación / no implementado». Estado corregido (backend PR #496 del 2026-10-18, frontend PR #563 del 2026-10-19). **Pendiente**: reflejarla en `specs/board-games/spec.md`, `specs/board-games/api-contract.md`, `data-model/schema.md` y `architecture/backend.md`, que hoy no la mencionan
- **`research/future/game-dlcs.md`**: la sección «trampa de implementación» era falsa. `GameService.update` carga el registro existente y le aplica una lista blanca (`copyUpdatableFields`), así que `parentId` no se pierde en el `PUT` con solo no añadirlo a esa lista. Corregido también en el issue backend-collection#505

### Changed
- **Wiki convertida a bundle Open Knowledge Format (OKF v0.1)**: los 70 documentos concepto (`specs/`, `architecture/`, `research/`, `data-model/`, `requirements.md`, `changelog.md`) llevan frontmatter YAML con `type`, `title`, `description` y `tags`; los cuerpos no se han tocado
- Añadidos `index.md` (listado de directorio OKF) en raíz, `specs/`, `architecture/`, `research/`, `research/external-apis/` y `data-model/`
- `AGENTS.md`: nueva sección **OKF bundle** con las convenciones y el vocabulario controlado de `type`
- `README.md`: nota de formato OKF en Convenciones
- `research/future/index.md`: Funko y Amiibo pasan de 📋 Investigación a ✅ **Implementado** (el código de `main` ya los consume)
- `research/future/collectibles-funko-amiibo.md`: estado actualizado a implementado (sin número de fase en la wiki)
- `README.md`: árbol de documentación y tabla de APIs externas actualizados
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
- Campo publishedDate (ya no se usa)

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

## [1.0.0] — Fase 1: Libros

### Added
- Configuración inicial Spring Boot 4 + Java 25
- Modelo de datos Book
- Repositorio Spring Data MongoDB
- Documentación inicial
