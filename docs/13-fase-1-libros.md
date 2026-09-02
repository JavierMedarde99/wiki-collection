# Fase 1: Colección de Libros

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 (arquitectura hexagonal) para:

1. Hacer llamadas a la API externa Google Books para buscar libros
2. Devolver la información de libros en el endpoint `GET /api/books/search`
3. CRUD completo de libros en la colección local (MongoDB)

---

## API Externa: Google Books

**URL Base:** `https://www.googleapis.com/books/v1/volumes`

**Endpoints:**

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/volumes?q=intitle:{title}` | Buscar por título |
| GET | `/volumes?q=inauthor:{author}` | Buscar por autor |
| GET | `/volumes?q=isbn:{isbn}` | Buscar por ISBN |
| GET | `/volumes/{volumeId}` | Obtener por ID |

**Ventajas:**
- Gratuita con API key (opcional para desarrollo)
- Búsqueda por nombre, autor, ISBN
- Datos completos: título, autores, descripción, portada, etc.

**Estructura de respuesta (un libro):**

```json
{
  "volumeInfo": {
    "title": "Cien años de soledad",
    "authors": ["Gabriel García Márquez"],
    "publisher": "Editorial Sudamericana",
    "publishedDate": "1967",
    "description": "La novela que revolucionó la literatura...",
    "industryIdentifiers": [
      {"type": "ISBN_13", "identifier": "9780307474728"}
    ],
    "pageCount": 471,
    "categories": ["Fiction"],
    "imageLinks": {
      "thumbnail": "https://books.google.com/books/content?id=..."
    },
    "language": "es"
  }
}
```

---

## Paso 1: Modelo de Datos — Book (Dominio)

- [x] Crear enum `BookState`: TO_READ, READING, COMPLETED
- [x] Crear enum `BookType`: MANGA, NOVEL, GRAPHIC_NOVEL
- [x] Crear documento `Book.java` en `domain/model/`:
  - Anotación `@Document(collection = "BOOKS")`
  - Campos: id, externalId, title, author, description, pages, type, state, comment, start, startDate, endDate, frontpage, externalSource

## Paso 2: Repositorio — Puerto y Adaptador

- [x] Crear interface `BookRepository.java` en `domain/port/out/`:
  - `findAll(Pageable)`
  - `findById(String)`
  - `findByState(BookState, Pageable)`
  - `save(Book)`
  - `deleteById(String)`
- [x] Crear interface `ExternalBookCatalogClient.java` en `domain/port/out/`:
  - `search(String query) → List<BookSearchResult>`
- [x] Crear `SpringDataBookRepository.java` en `infrastructure/adapter/out/persistence/`:
  - `findByState(BookState, Pageable)`
  - `findByExternalId(String)`
- [x] Crear `BookEntity.java` en `infrastructure/adapter/out/persistence/`:
  - `@Document(collection = "BOOKS")`
  - Mismos campos que dominio
- [x] Crear `BookEntityMapper.java` en `infrastructure/adapter/out/persistence/`:
  - `toDomain(BookEntity)`
  - `toEntity(Book)`
- [x] Crear `BookPersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`:
  - Implementa `BookRepository`
  - Usa `SpringDataBookRepository` + `BookEntityMapper`

## Paso 3: Servicio — Caso de Uso y Aplicación

- [x] Crear interface `BookUseCase.java` en `domain/port/in/`:
  - `findAll(Pageable)`
  - `findByState(BookState, Pageable)`
  - `findById(String)`
  - `save(Book)`
  - `update(String, Book)`
  - `delete(String)`
- [x] Crear interface `BookSearchUseCase.java` en `domain/port/in/`:
  - `search(String query)`
- [x] Crear `BookService.java` en `application/service/`:
  - Implementa `BookUseCase`
  - Lógica de CRUD
- [x] Crear `BookSearchService.java` en `application/service/`:
  - Implementa `BookSearchUseCase`
  - Valida query no vacío
  - Delega a `ExternalBookCatalogClient`
- [x] Crear `GoogleBooksClient.java` en `infrastructure/adapter/out/google/`:
  - Implementa `ExternalBookCatalogClient`
  - Usa `RestClient` para llamar a Google Books
  - Mapea respuesta al modelo `BookSearchResult`
  - Manejo de errores (graceful degradation: lista vacía si falla)

## Paso 4: Controller — Endpoints REST

- [x] Crear `BookController.java` en `infrastructure/adapter/in/web/`:
  - `GET /api/books` — listar con paginación y filtro por state
  - `GET /api/books/{id}` — obtener libro por ID
  - `POST /api/books` — añadir libro a la colección
  - `PUT /api/books/{id}` — actualizar libro
  - `DELETE /api/books/{id}` — eliminar libro
  - `GET /api/books/search?name={query}` — buscar en Google Books
- [x] Crear `BookRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`:
  - Record con campos de entrada + validaciones
- [x] Crear `BookResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`:
  - Record con campos de salida
- [x] Crear `BookDtoMapper.java` en `infrastructure/adapter/in/web/dto/`:
  - `toDomain(BookRequest)`
  - `toResponse(Book)`
- [x] Crear `StringToBookStateConverter.java` en `infrastructure/config/`:
  - Convierte String → BookState

## Paso 5: Tests

- [x] `BookServiceTest.java` — Tests unitarios de CRUD
- [x] `BookControllerTest.java` — Tests de integración MockMvc
- [x] `GoogleBooksServiceTest.java` — Tests del cliente externo con mock server
- [x] `BookPersistenceAdapterTest.java` — Tests del adaptador de persistencia

## Criterios de Aceptación

- [x] El endpoint `GET /api/books/search?name=harry` devuelve resultados de Google Books
- [x] Los resultados incluyen: título, autor, descripción, portada, páginas, editorial
- [x] El endpoint `GET /api/books` devuelve lista vacía al inicio
- [x] Se puede crear un libro vía `POST /api/books`
- [x] Se puede actualizar/eliminar un libro vía `PUT`/`DELETE /api/books/{id}`
- [x] Los tests pasan (`mvn verify`)
- [x] El respeta arquitectura hexagonal (dependencias hacia dentro)

## Estructura de Paquetes Final

```
com.wikicollection/
├── domain/
│   └── model/
│       ├── Book
│       ├── BookState
│       ├── BookType
│       └── BookSearchResult
│   └── port/
│       ├── in/
│       │   ├── BookUseCase
│       │   └── BookSearchUseCase
│       └── out/
│           ├── BookRepository
│           └── ExternalBookCatalogClient
├── application/
│   └── service/
│       ├── BookService
│       └── BookSearchService
├── infrastructure/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   ├── BookController
│   │   │   └── dto/
│   │   │       ├── BookRequest
│   │   │       ├── BookResponse
│   │   │       └── BookDtoMapper
│   │   └── out/
│   │       ├── google/
│   │       │   └── GoogleBooksClient
│   │       └── persistence/
│   │           ├── BookEntity
│   │           ├── SpringDataBookRepository
│   │           ├── BookEntityMapper
│   │           └── BookPersistenceAdapter
│   └── config/
│       ├── StringToBookStateConverter
│       └── RestClientConfig
```

## Notas

- Google Books no requiere API key para desarrollo, pero es recomendado para producción
- La búsqueda se hace por título con el query param `name` (consistente con Books)
- ~10 millones de libros disponibles, suficiente para cualquier demo
- Si en el futuro se quiere una API más completa, se puede añadir Open Library como segundo `ExternalBookCatalogClient`
