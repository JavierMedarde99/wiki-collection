# Fase 1: Colección de Libros — Backend

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 para:

1. Hacer llamadas a las APIs externas (Google Books / OpenLibrary)
2. Devolver la información de libros en el endpoint `GET /api/books/search`
3. Crear la estructura del proyecto y conexión con MongoDB

---

## Paso 1: Configuración del Proyecto Spring Boot

- [ ] Crear proyecto Spring Boot 4 con Java 25 y dependencias:
  - `spring-boot-starter-web`
  - `spring-boot-starter-data-mongodb`
  - `spring-boot-starter-validation`
  - `spring-boot-starter-test`
- [ ] Configurar `application.properties`:
  - Puerto: 8080
  - MongoDB URI: `mongodb://localhost:27017/wiki-collection`
  - Google Books API Key (variable de entorno)
- [ ] Configurar CORS para permitir peticiones del frontend (localhost:5173)
- [ ] Configurar auditoría MongoDB (`@EnableMongoAuditing`)

## Paso 2: Modelo de Datos

- [ ] Crear documento `Book.java`:
  - Anotación `@Document(collection = "books")`
  - Campos: title, authors, isbn, publisher, publishedDate, description, pageCount, categories, coverImage, language, status, userRating, notes, tags, dateAdded, dateCompleted, externalSource, externalId
  - Enum `BookStatus`: WISHLIST, READING, COMPLETED, ABANDONED
  - Anotaciones de validación (`@NotBlank`, `@NotEmpty`, etc.)
  - Timestamps automáticos (`@CreatedDate`, `@LastModifiedDate`)

## Paso 3: Repositorio

- [ ] Crear `BookRepository extends MongoRepository<Book, String>`:
  - `findByStatus(BookStatus, Pageable)`
  - `findByTagsContaining(String, Pageable)`
  - `findByIsbn(String)`
  - `existsByIsbn(String)`
  - Búsqueda por título o autor con `@Query`

## Paso 4: Servicio

- [ ] Crear `BookService`:
  - Lógica de CRUD (findAll, findById, save, update, delete)
  - `save()`: asignar `dateAdded` automáticamente, `dateCompleted` si status=COMPLETED
  - `update()`: verificar existencia, actualizar campos, manejar dateCompleted
- [ ] Crear `GoogleBooksService`:
  - Llamada a `https://www.googleapis.com/books/v1/volumes?q={query}&maxResults=20`
  - Incluir API Key en la petición (si está configurada)
  - Devolver lista de resultados crudos
- [ ] (Futuro) `OpenLibraryService` como fallback

## Paso 5: Controller

- [ ] Crear `BookController`:
  - `GET /api/books` — listar con paginación y filtros (status, tag)
  - `GET /api/books/{id}` — obtener libro por ID
  - `POST /api/books` — añadir libro a la colección
  - `PUT /api/books/{id}` — actualizar libro
  - `DELETE /api/books/{id}` — eliminar libro
  - **`GET /api/books/search?q={query}`** — buscar en Google Books y devolver resultados

## Paso 6: Tests

- [ ] Tests unitarios para `BookService`
- [ ] Tests de integración para `BookController`
- [ ] Mock de GoogleBooksService

## Criterios de Aceptación

- [ ] El endpoint `GET /api/books/search?q=cien` devuelve resultados de Google Books
- [ ] Los resultados incluyen: título, autores, isbn, portada, descripción, página, editorial
- [ ] El endpoint `GET /api/books` devuelve lista vacía al inicio
- [ ] Se puede crear un libro vía `POST /api/books`
- [ ] El proyecto compila sin errores (`mvn compile`)
- [ ] Los tests pasan (`mvn test`)
