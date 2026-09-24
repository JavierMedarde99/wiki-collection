# Spec: Películas y Series (Movie Shows)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** MovieShow (domain/model/MovieShow.java)

---

## Dominio

### Entidad MovieShow

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| externalId | String | ❌ | ID en TMDB (unique) |
| title | String | ✅ | Título de la película/serie |
| overview | String | ❌ | Sinopsis |
| releaseDate | LocalDate | ❌ | Fecha de estreno |
| posterUrl | String | ❌ | URL del póster |
| backdropUrl | String | ❌ | URL de la imagen de fondo |
| voteAverage | Double | ❌ | Puntuación media TMDB |
| mediaType | Enum (MovieMediaType) | ✅ | MOVIE, TV |
| status | Enum (MovieStatus) | ✅ | WATCHING, WATCHED, PLAN_TO_WATCH |
| userRating | Integer (1-5) | ❌ | Valoración personal |
| comment | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió a la colección |
| dateCompleted | LocalDate | ❌ | Cuándo se terminó de ver |
| externalSource | String | ❌ | "TMDB" |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Enums

**MovieMediaType:** `MOVIE`, `TV`

**MovieStatus:** `WATCHING`, `WATCHED`, `PLAN_TO_WATCH`

> **Nota:** El enum MovieStatus NO incluye WISHLIST (AD-015: MovieStatus sin WISHLIST). Solo 3 valores alineados con el dominio de visualización.

### Reglas de Negocio

- **externalId es único** — no se pueden duplicar películas/series por externalId de TMDB
- **mediaType es requerido** — debe ser MOVIE o TV
- **dateCompleted no puede ser anterior a dateAdded** (validado en DateRangeValidator)
- **El endpoint base es `/api/movieshows`** (no `/api/movies`) — AD-017: endpoints base renombrados a /api/movieshows

---

## Puertos (Interfaces de Dominio)

### MovieShowUseCase (in)
```java
public interface MovieShowUseCase {
    MovieShow save(MovieShow movieShow, String ownerId);
    MovieShow update(String id, MovieShow updates, String ownerId);
    void delete(String id, String ownerId);
    MovieShow findById(String id);
    Page<MovieShow> search(MovieSearchCriteria criteria, Pageable pageable);
}
```

### MovieSearchUseCase (in)
```java
public interface MovieSearchUseCase {
    List<MovieSearchResult> searchExternal(String query, MovieMediaType mediaType);
    MovieSearchResult findById(String externalId);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `MovieShowService` | CRUD de películas/series + validaciones |
| `MovieSearchService` | Búsqueda externa TMDB + mapeo a MovieSearchResult |
| `DateRangeValidator` | Validación de fechas (dateAdded vs dateCompleted) |

---

## API Externa: TMDB

- **Base URL:** `https://api.themoviedb.org/3`
- **Auth:** API Key (gratuita con registro)
- **Rate limit:** ~40 requests/segundo
- **Gratis:** Sí, con registro
- **Formato:** JSON
- **Estado:** Activa y mantenida (2026)

### Endpoints usados

| Uso | Endpoint |
|-----|----------|
| Buscar película | `GET /search/movie?query={query}` |
| Buscar serie | `GET /search/tv?query={query}` |
| Detalle película | `GET /movie/{id}` |
| Detalle serie | `GET /tv/{id}` |

### Mapeo TMDB → MovieSearchResult

| Campo TMDB | Campo SearchResult | Notas |
|------------|--------------------|-------|
| `id` | `externalId` | ID único de TMDB |
| `title` / `name` | `title` | `title` para películas, `name` para series |
| `overview` | `overview` | Sinopsis |
| `release_date` / `first_air_date` | `releaseDate` | Fecha de estreno |
| `poster_path` | `posterUrl` | Póster (image.tmdb.org/t/p/w500) |
| `backdrop_path` | `backdropUrl` | Fondo (image.tmdb.org/t/p/original) |
| `vote_average` | `voteAverage` | Puntuación media |

### URLs de Imágenes TMDB

| Tipo | Base URL |
|------|----------|
| Póster (w500) | `https://image.tmdb.org/t/p/w500` |
| Backdrop (original) | `https://image.tmdb.org/t/p/original` |

---

## Controlador REST

**Base:** `/api/v1/movieshows`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/movieshows` | Listar películas/series (pagina, filtra por name/status/mediaType/owner) |
| GET | `/api/v1/movieshows/{id}` | Obtener película/serie por ID |
| POST | `/api/v1/movieshows` | Crear película/serie (auth requerido) |
| PUT | `/api/v1/movieshows/{id}` | Actualizar película/serie (auth requerido, ownership verificado) |
| DELETE | `/api/v1/movieshows/{id}` | Eliminar película/serie (204 No Content, auth requerido) |
| GET | `/api/v1/movieshows/search?name={query}&mediaType={type}` | Buscar en TMDB |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Buscar por título |
| `status` | Enum | Filtrar por estado (WATCHING, WATCHED, PLAN_TO_WATCH) |
| `mediaType` | Enum | Filtrar por tipo (MOVIE, TV) |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |

---

## DTOs

### MovieShowRequest (POST/PUT)

```json
{
  "externalId": "string (obligatorio)",
  "title": "string (obligatorio)",
  "overview": "string",
  "releaseDate": "date",
  "posterUrl": "string",
  "backdropUrl": "string",
  "voteAverage": "number",
  "mediaType": "MOVIE | TV",
  "status": "WATCHING | WATCHED | PLAN_TO_WATCH",
  "userRating": "integer (1-5)",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string"
}
```

### MovieShowResponse (GET)

```json
{
  "id": "string",
  "externalId": "string",
  "title": "string",
  "overview": "string",
  "releaseDate": "date",
  "posterUrl": "string",
  "backdropUrl": "string",
  "voteAverage": "number",
  "mediaType": "MOVIE | TV",
  "status": "WATCHING | WATCHED | PLAN_TO_WATCH",
  "userRating": "integer",
  "comment": "string",
  "dateAdded": "date",
  "dateCompleted": "date",
  "externalSource": "string",
  "createdAt": "datetime",
  "updatedAt": "datetime",
  "ownerId": "string"
}
```

### MovieShowDtoMapper

Mapea entre MovieShow domain entity ↔ MovieShowRequest/MovieShowResponse DTOs.

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `MovieShowNotFoundException` | 404 | Película/serie no encontrada |
| `MovieShowConflictException` | 409 | Conflicto al guardar (externalId duplicado) |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `MovieShowServiceTest` | Unitario | CRUD de películas/series |
| `MovieShowControllerTest` | Integración | Endpoints de películas/series |
| `MovieSearchServiceTest` | Unitario | Búsqueda en TMDB |
| `TmdbClientTest` | Unitario | Cliente TMDB con mock |
| `MovieShowPersistenceAdapterTest` | Integración | Persistencia de películas/series |
| `MovieShowDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
| `MovieShowModelTest` | Unitario | Modelo de dominio MovieShow |
| `MovieShowServiceOwnerFilterTest` | Unitario | Filtro owner en movieshows (Fase 9) |
| `DateRangeValidatorTest` | Unitario | Validación de fechas |
| `StringToMovieMediaTypeConverterTest` | Unitario | Conversor String → MovieMediaType |
| `StringToMovieStatusConverterTest` | Unitario | Conversor String → MovieStatus |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/movieshows/search?name=matrix` devuelve resultados de TMDB
- [x] Los resultados incluyen: título, sinopsis, fecha de estreno, póster, puntuación, tipo
- [x] Se puede filtrar por tipo (MOVIE, TV) con el parámetro `mediaType`
- [x] El endpoint `GET /api/v1/movieshows` devuelve lista vacía al inicio
- [x] Se puede crear una película/serie vía `POST /api/v1/movieshows` (auth requerido)
- [x] Se puede actualizar vía `PUT /api/v1/movieshows/{id}` (auth + ownership)
- [x] Se puede eliminar vía `DELETE /api/v1/movieshows/{id}` (204, auth requerido)
- [x] Los filtros funcionan: por nombre, estado y tipo
- [x] Se evitan duplicados por externalId
- [x] Las fechas se validan correctamente (dateAdded vs dateCompleted)
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)

---

## Estado de Implementación

Fase 5 completada. Todos los componentes implementados:
- ✅ Domain: MovieShow.java, MovieStatus.java, MovieMediaType.java, MovieSearchCriteria.java, MovieSearchResult.java
- ✅ Ports: MovieShowUseCase.java, MovieSearchUseCase.java, MovieShowRepository.java, ExternalMovieCatalogClient.java
- ✅ Application: MovieShowService.java, MovieSearchService.java, DateRangeValidator.java
- ✅ Infrastructure: TmdbClient.java, MovieShowController.java, MovieShowEntity.java, MovieShowPersistenceAdapter.java, SpringDataMovieShowRepository.java, MovieShowDtoMapper.java, MovieShowRequest.java, MovieShowResponse.java
- ✅ Config: TmdbClientConfig.java, StringToMovieMediaTypeConverter.java, StringToMovieStatusConverter.java
- ✅ Tests: 11 archivos de test
