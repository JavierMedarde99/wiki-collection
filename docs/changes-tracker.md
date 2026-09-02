# Cambios en Repositorios Backend y Frontend

Documento de seguimiento de todos los issues y pull requests implementados en los repositorios del proyecto Wiki-Collection.

---

## Backend — `JavierMedarde99/backend-collection`

### Issues

| # | Título | Estado | Descripción |
|---|--------|--------|-------------|
| 17 | Error de autenticación de MongoDB en arranque/guardado | OPEN | Credenciales hardcodeadas en `application.properties` causan fallo de autenticación. Solución: parametrizar con variable de entorno `SPRING_MONGODB_URI`. |
| 15 | Refactor del modelo de datos a nuevo esquema BOOKS | CLOSED | Migración del modelo Book a nuevo esquema con 12 campos (externalId, title, descripcion, author, pages, type, state, comment, start, startDate, endDate, frontpage). |
| 6 | Paso 6: Tests - Unitarios e Integración | CLOSED | Tests unitarios y de integración para todas las capas. |
| 5 | Paso 5: Controller - BookController | CLOSED | Implementación del controlador REST de libros. |
| 4 | Paso 4: Servicio - BookService y GoogleBooksService | CLOSED | Servicios de aplicación y cliente Google Books. |
| 3 | Paso 3: Repositorio - BookRepository | CLOSED | Repositorio Spring Data MongoDB. |
| 2 | Paso 2: Modelo de Datos - Documento Book | CLOSED | Modelo de dominio Book inicial. |
| 1 | Paso 1: Configuración del Proyecto Spring Boot | CLOSED | Setup inicial del proyecto. |

### Pull Requests

| # | Título | Rama | Estado | Cambios Principales |
|---|--------|------|--------|---------------------|
| 19 | docs: add AGENTS.md | docs/agents-md | MERGED | Documentación de arquitectura hexagonal, comandos, gotchas (Lombok JDK25, MongoDB property key, tests sin Mongo). |
| 18 | Fix: parametrizar URI de MongoDB con variable de entorno | fix/issue-17-mongodb-uri | OPEN | Parametriza URI MongoDB; cambia collection `BOOKS`→`books`; añade springdoc-openapi. |
| 16 | Refactor del modelo de datos al nuevo esquema BOOKS | feat/issue-15-books-model | MERGED | Migración completa al esquema BOOKS. Ver detalle abajo. |
| 14 | Refactor: arquitectura hexagonal sobre la implementación completa | refactor/hexagonal-architecture | MERGED | Migración a arquitectura hexagonal. Ver detalle abajo. |
| 13 | Feat/issue 1 spring boot setup | feat/issue-1-spring-boot-setup | MERGED | Setup inicial: modelo Book, BookStatus, Lombok, repositorio, servicio, controller, Google Books. |
| 12 | Paso 6: Tests - Unitarios e Integración | feat/issue-6-tests | MERGED | Tests: BookServiceTest, BookControllerTest, GoogleBooksServiceTest. |
| 11 | Paso 5: Controller - BookController | feat/issue-5-book-controller | MERGED | Controller REST con CRUD + search. |
| 10 | Paso 4: Servicio - BookService y GoogleBooksService | feat/issue-4-services | MERGED | BookService (CRUD, validaciones ISBN, fechas) + GoogleBooksService. |
| 9 | Paso 3: Repositorio - BookRepository | feat/issue-3-book-repository | MERGED | SpringDataBookRepository con queries personalizadas. |
| 8 | Paso 2: Modelo de Datos - Documento Book | feat/issue-2-book-model | MERGED | Entidad Book con anotaciones MongoDB + enum BookStatus. |
| 7 | Paso 1: Configuración del Proyecto Spring Boot | feat/issue-1-spring-boot-setup | MERGED | Proyecto base: pom.xml, application.properties. |

---

### Detalle PR #14 — Arquitectura Hexagonal

**Estado:** MERGED

**Cambios:**
- Reorganización de paquetes en `domain/`, `application/`, `infrastructure/`
- **Domain:** modelos `Book`, `BookStatus`, `BookSearchResult` + puertos `BookUseCase`, `BookSearchUseCase`, `BookRepository`, `ExternalBookCatalogClient`
- **Application:** servicios `BookService`, `BookSearchService` + excepciones `BookNotFoundException`, `DuplicateIsbnException`
- **Infrastructure:**
  - `adapter/in/web`: `BookController`, `GlobalExceptionHandler`, DTOs `BookRequest`, `BookResponse`, `BookDtoMapper`
  - `adapter/out/persistence`: `BookEntity`, `SpringDataBookRepository`, `BookEntityMapper`, `BookPersistenceAdapter`
  - `adapter/out/google`: `GoogleBooksClient`
  - `config`: `WebConfig`, `MongoAuditConfig`, `StringToBookStatusConverter`, `RestClientConfig`
- Dependencias apuntan solo hacia el dominio
- `@EnableMongoAuditing` movido a `MongoAuditConfig` (clase separada para evitar duplicados)

---

### Detalle PR #16 — Refactor Modelo BOOKS

**Estado:** MERGED

**Cambios en modelo de datos:**

| Campo Antiguo | Campo Nuevo | Cambio |
|---------------|-------------|--------|
| `status` (WISHLIST/READING/COMPLETED/ABANDONED) | `state` (TO_READ/READING/COMPLETED) | Renombrado + reducido |
| — | `type` (MANGA/NOVEL/GRAPHIC_NOVEL) | Nuevo campo |
| `authors` (List<String>) | `author` (String) | Simplificado a string único |
| `description` | `descripcion` | Renombrado |
| `pageCount` | `pages` | Renombrado |
| `notes` | `comment` | Renombrado |
| `userRating` (1-5) | `start` (0-5) | Renombrado + rango ampliado |
| — | `startDate` (LocalDate) | Nuevo campo |
| — | `endDate` (LocalDate) | Nuevo campo |
| `coverImage` | `frontpage` | Renombrado |
| `isbn` | — | Eliminado |
| `publisher` | — | Eliminado |
| `publishedDate` | — | Eliminado |
| `categories` | — | Eliminado |
| `language` | — | Eliminado |
| `tags` | — | Eliminado |
| `dateAdded` | — | Eliminado |
| `dateCompleted` | — | Eliminado |
| `dateUpdated` | — | Eliminado |
| `externalSource` | — | Eliminado |

**Otros cambios:**
- Eliminados: `DuplicateIsbnException`, `StringToBookStatusConverter`
- Creados: `BookState`, `BookType`, `StringToBookStateConverter`
- Collection MongoDB: `books` → `BOOKS`
- API: filtro `tag` eliminado; filtro `status` → `state`; búsqueda usa `name` (no `q`); maxResults 5→10

---

## Frontend — `JavierMedarde99/frontend-collection`

### Issues

| # | Título | Estado | Descripción |
|---|--------|--------|-------------|
| 5 | [Feature] Página: Editar libro | CLOSED | Formulario de edición de libros con datos precargados. |
| 4 | [Feature] Página: Lista de libros | CLOSED | Listado con filtros por estado y paginación. |
| 3 | [Feature] Página: Borrar libro | OPEN | Eliminar libro desde lista y detalle con confirmación. |
| 2 | [Feature] Página: Buscar libros | CLOSED | Búsqueda vía Google Books API con adición a colección. |
| 1 | [Feature] Página: Insertar libro | CLOSED | Formulario de creación de libros. |

### Pull Requests

| # | Título | Rama | Estado | Cambios Principales |
|---|--------|------|--------|---------------------|
| 10 | Borrar libro (desde la lista) | feature/borrar-libro | MERGED | Añade botón eliminar en BookCard + diálogo de confirmación en lista. |
| 9 | Página: Editar libro | feature/editar-libro | MERGED | BookEditPage: formulario edición + botón eliminar + ConfirmDialog. |
| 8 | Página: Buscar libros | feature/buscar-libros | MERGED | BookSearchPage: búsqueda Google Books + añadir a colección. |
| 7 | Página: Insertar libro | feature/insertar-libro | MERGED | BookCreatePage: formulario creación + BookForm reutilizable. |
| 6 | Página: Lista de libros | feature/lista-libros | MERGED | BookListPage: listado con filtros por estado, paginación, BookCard. |

---

### Detalle PR #6 — Página Lista de Libros

**Estado:** MERGED

**Componentes creados:**
- `pages/BookListPage.jsx` — Página principal con listado paginado
- `components/BookCard.jsx` — Tarjeta de libro (portada, título, autor, estado, tipo, puntuación, acciones)
- `components/StatusBadge.jsx` — Badge de estado del libro
- `components/StarRating.jsx` — Componente de puntuación por estrellas
- `constants/books.js` — Constantes de estados de libros

**Funcionalidades:**
- Listado con paginación (12 items por página)
- Filtro por estado (TO_READ, READING, COMPLETED)
- Ordenación por título ascendente
- Estados vacío y de carga
- Navegación a edición desde cada tarjeta

---

### Detalle PR #7 — Página Insertar Libro

**Estado:** MERGED

**Componentes creados:**
- `pages/BookCreatePage.jsx` — Página de creación
- `components/BookForm.jsx` — Formulario reutilizable (create/edit)

**Funcionalidades:**
- Formulario con campos del esquema BOOKS
- Validación de campos requeridos
- Navegación a edición tras creación exitosa

---

### Detalle PR #8 — Página Buscar Libros

**Estado:** MERGED

**Componentes creados:**
- `pages/BookSearchPage.jsx` — Búsqueda vía Google Books API

**Funcionalidades:**
- Campo de búsqueda con formulario
- Resultados en grid con portada, título, autor, ISBN, editorial
- Botón "Añadir a mi colección" en cada resultado
- Mapeo de resultado Google Books → esquema BOOKS
- Estados: carga, sin resultados, error

---

### Detalle PR #9 — Página Editar Libro

**Estado:** MERGED

**Componentes creados:**
- `pages/BookEditPage.jsx` — Página de edición
- `components/ConfirmDialog.jsx` — Diálogo de confirmación reutilizable

**Funcionalidades:**
- Formulario precargado con datos del libro
- Guardado de cambios (PUT al backend)
- Botón eliminar con confirmación
- Manejo de errores (libro no encontrado)

---

### Detalle PR #10 — Borrar Libro desde Lista

**Estado:** MERGED

**Cambios:**
- `BookCard`: prop `onDelete` opcional para mostrar botón eliminar
- `BookListPage`: integración de ConfirmDialog para eliminación
- Eliminación reactiva (UI se actualiza sin recargar)

---

## Resumen de Estado del Proyecto

### Backend
- ✅ Arquitectura hexagonal implementada
- ✅ Modelo de datos BOOKS activo
- ✅ CRUD completo de libros
- ✅ Búsqueda Google Books integrada
- ✅ Tests unitarios y de integración
- ✅ Spring Boot 4 + Java 25 + MongoDB
- ⏳ Issue #17 abierto: parametrizar URI MongoDB (PR #18 pendiente mergear)

### Frontend
- ✅ CRUD completo (listar, crear, editar, buscar)
- ⏳ Issue #3 abierto: eliminar desde lista (PR #10 mergeado; falta confirmar si cierra issue)
- ✅ React + Vite + React Router
- ✅ Componentes reutilizables (BookForm, ConfirmDialog, StatusBadge, StarRating)
- ✅ Diseño responsive con Tailwind CSS
