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

---

## Estado: ✅ COMPLETADA

Todos los pasos fueron implementados y verificados:

- [x] Crear enum `BookState`: TO_READ, READING, COMPLETED
- [x] Crear enum `BookType`: MANGA, NOVEL, GRAPHIC_NOVEL
- [x] Crear documento `Book.java` en `domain/model/`
- [x] Crear interface `BookRepository.java` en `domain/port/out/`
- [x] Crear interface `ExternalBookCatalogClient.java` en `domain/port/out/`
- [x] Crear `SpringDataBookRepository.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `BookEntity.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `BookEntityMapper.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `BookPersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear interface `BookUseCase.java` en `domain/port/in/`
- [x] Crear interface `BookSearchUseCase.java` en `domain/port/in/`
- [x] Crear `BookService.java` en `application/service/`
- [x] Crear `BookSearchService.java` en `application/service/`
- [x] Crear `GoogleBooksClient.java` en `infrastructure/adapter/out/google/`
- [x] Crear `BookController.java` en `infrastructure/adapter/in/web/`
- [x] Crear `BookRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `BookResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `BookDtoMapper.java` en `infrastructure/adapter/in/web/dto/`
- [x] Crear `StringToBookStateConverter.java` en `infrastructure/config/`
- [x] `BookServiceTest.java` — Tests unitarios de CRUD
- [x] `BookControllerTest.java` — Tests de integración MockMvc
- [x] `GoogleBooksClientTest.java` — Tests del cliente externo con mock server
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
