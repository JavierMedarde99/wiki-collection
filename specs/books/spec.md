# Spec: Libros (Books)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** Book (domain/model/Book.java)

---

## Dominio

### Entidad Book

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| externalId | String | ❌ | ID en Google Books API |
| title | String | ✅ | Título del libro |
| descripcion | String | ❌ | Sinopsis |
| author | String | ✅ | Autor (string singular, no lista) |
| pages | Integer | ❌ | Número de páginas |
| type | Enum (BookType) | ✅ | MANGA, NOVEL, GRAPHIC_NOVEL |
| state | Enum (BookState) | ✅ | TO_READ, READING, COMPLETED |
| comment | String | ❌ | Notas personales |
| start | Integer (0-5) | ❌ | Valoración personal |
| startDate | LocalDate | ❌ | Fecha de inicio de lectura |
| endDate | LocalDate | ❌ | Fecha de fin de lectura |
| frontpage | String | ❌ | URL a la portada |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Enums

**BookType:** `MANGA`, `NOVEL`, `GRAPHIC_NOVEL`

**BookState:** `TO_READ`, `READING`, `COMPLETED`

### Reglas de Negocio

- **externalId es único** — no se pueden duplicar libros por externalId de Google Books
- **author es string singular** — los autores de Google Books vuelven como lista; el backend normaliza a `authors[0]`
- **Valoración start:** rango 0-5, opcional
- **Fecha range:** endDate no puede ser anterior a startDate (validado en DateRangeValidator)

---

## Puertos (Interfaces de Dominio)

### BookUseCase (in)
```java
public interface BookUseCase {
    Book save(Book book, String ownerId);
    Book update(String id, Book updates, String ownerId);
    void delete(String id, String ownerId);
    Book findById(String id);
    Page<Book> search(BookSearchCriteria criteria, Pageable pageable);
}
```

### BookSearchUseCase (in)
```java
public interface BookSearchUseCase {
    List<BookSearchResult> searchExternal(String query, BookSearchCriteria criteria);
    List<BookSearchResult> searchByIsbn(String isbn);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `BookService` | CRUD de libros + validaciones de dominio |
| `BookSearchService` | Búsqueda externa Google Books + mapeo a BookSearchResult |

---

## API External: Google Books

- **Base URL:** `https://www.googleapis.com/books/v1/volumes`
- **Auth:** API Key (opcional para desarrollo, recomendado para producción)
- **Rate limit:** 100 requests/100 segundos (con API Key), 10/sin key
- **Cobertura:** ~10 millones de libros
- **Estado:** Activa y mantenida

### Endpoints usados

| Uso | Endpoint |
|-----|----------|
| Buscar por título | `GET /volumes?q=intitle:{title}` |
| Buscar por autor | `GET /volumes?q=inauthor:{author}` |
| Buscar por ISBN | `GET /volumes?q=isbn:{isbn}` |

### Mapeo Google Books → BookSearchResult

| Campo Google Books | Campo SearchResult | Notas |
|-------------------|--------------------|-------|
| `id` | `externalId` | ID del volumen |
| `volumeInfo.title` | `title` | Título |
| `volumeInfo.authors[0]` | `author` | Primer autor |
| `volumeInfo.description` | `description` | Sinopsis |
| `volumeInfo.pageCount` | `pageCount` | Páginas |
| `volumeInfo.imageLinks.thumbnail` | `coverImage` | Portada |
| `volumeInfo.publisher` | `publisher` | Editorial |
| `volumeInfo.publishedDate` | `publishedDate` | Fecha publicación |
| `volumeInfo.language` | `language` | Idioma |

---

## Controlador REST

**Base:** `/api/v1/books`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/books` | Listar libros (pagina, filtra por name/author/type/state/owner) |
| GET | `/api/v1/books/{id}` | Obtener libro por ID |
| POST | `/api/v1/books` | Crear libro (auth requerido) |
| PUT | `/api/v1/books/{id}` | Actualizar libro (auth requerido, ownership verificado) |
| DELETE | `/api/v1/books/{id}` | Eliminar libro (204 No Content, auth requerido) |
| GET | `/api/v1/books/search?name={query}` | Buscar en Google Books API por título |
| GET | `/api/v1/books/search?isbn={isbn}` | Buscar en Google Books API por ISBN-13 |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Buscar por título (LIKE case-insensitive) |
| `author` | String | Buscar por autor |
| `type` | Enum | Filtrar por tipo (MANGA, NOVEL, GRAPHIC_NOVEL) |
| `state` | Enum | Filtrar por estado (TO_READ, READING, COMPLETED) |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |

### Filtro `owner` (Fase 9+)

| Valor | Descripción | Auth requerido |
|-------|-------------|----------------|
| `mine` | Solo elementos del usuario autenticado | Sí |
| `other` | Solo elementos públicos de otros usuarios | Sí |
| `all` | Todos (propios + públicos de otros) | Sí |

---

## DTOs

### BookRequest (POST/PUT)

```json
{
  "externalId": "string",
  "title": "string (obligatorio)",
  "descripcion": "string",
  "author": "string (obligatorio)",
  "pages": "integer (min 0)",
  "type": "MANGA | NOVEL | GRAPHIC_NOVEL",
  "state": "TO_READ | READING | COMPLETED",
  "comment": "string",
  "start": "integer (0-5)",
  "startDate": "date",
  "endDate": "date",
  "frontpage": "string (URL)"
}
```

### BookResponse (GET)

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "descripcion": "string",
  "author": "string",
  "pages": "integer",
  "type": "MANGA | NOVEL | GRAPHIC_NOVEL",
  "state": "TO_READ | READING | COMPLETED",
  "comment": "string",
  "start": "integer",
  "startDate": "date",
  "endDate": "date",
  "frontpage": "string",
  "ownerId": "string"
}
```

### BookSearchResult (resultado de búsqueda externa)

```json
{
  "id": "string",
  "title": "string",
  "authors": "string",
  "isbn": "string",
  "coverImage": "string",
  "description": "string",
  "pageCount": "integer",
  "publisher": "string",
  "publishedDate": "string",
  "language": "string",
  "categories": ["string"]
}
```

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `BookNotFoundException` | 404 | Libro no encontrado |
| `BookConflictException` | 409 | Conflicto al guardar (externalId duplicado) |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `BookServiceTest` | Unitario | CRUD de libros |
| `BookControllerTest` | Integración | Endpoints de libros |
| `BookSearchServiceTest` | Unitario | Búsqueda en Google Books |
| `GoogleBooksClientTest` | Unitario | Cliente Google Books con mock server |
| `BookPersistenceAdapterTest` | Integración | Persistencia de libros |
| `BookDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
| `BookServiceOwnerFilterTest` | Unitario | Filtro owner=mine/other/all (Fase 9) |
| `StringToBookStateConverterTest` | Unitario | Conversor String → BookState |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/books/search?name=harry` devuelve resultados de Google Books
- [x] Los resultados incluyen: título, autor, descripción, portada, páginas, editorial
- [x] El endpoint `GET /api/v1/books` devuelve lista vacía al inicio
- [x] Se puede crear un libro vía `POST /api/v1/books` (auth requerido)
- [x] Se puede actualizar/eliminar un libro vía `PUT`/`DELETE /api/v1/books/{id}` (auth + ownership)
- [x] El endpoint `GET /api/v1/books/{id}` devuelve 404 si no existe
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)
- [x] El backend no expone save/update para magic cards — solo search, findById, addFromScryfall, delete

---

## Estado de Implementación

Fase 1 completada. Todos los componentes implementados:
- ✅ Domain: Book.java, BookState.java, BookType.java, BookSearchCriteria.java, BookSearchResult.java
- ✅ Ports: BookUseCase.java, BookSearchUseCase.java, BookRepository.java, ExternalBookCatalogClient.java
- ✅ Application: BookService.java, BookSearchService.java
- ✅ Infrastructure: GoogleBooksClient.java, BookController.java, BookEntity.java, BookPersistenceAdapter.java, SpringDataBookRepository.java, BookDtoMapper.java
- ✅ Config: StringToBookStateConverter.java
- ✅ Tests: 8 archivos de test
