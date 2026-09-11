# Fase 5: Películas y Series (TMDB)

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 (arquitectura hexagonal) para:

1. Hacer llamadas a la API externa **TMDB** para buscar películas y series
2. Devolver la información de películas/series en el endpoint `GET /api/movieshows/search`
3. CRUD completo de películas/series en la colección local (MongoDB)
4. Filtrar por tipo (MOVIE, TV) y estado (WATCHING, WATCHED, PLAN_TO_WATCH)

---

## API Externa

### TMDB (The Movie Database) API

- **Base URL:** `https://api.themoviedb.org/3`
- **Auth:** API Key (gratuita con registro)
- **Rate limit:** ~40 requests/segundo
- **Gratis:** Sí, con registro
- **Formato:** JSON
- **Documentación:** https://developer.themoviedb.org/docs

**Endpoints utilizados:**

| Uso | Endpoint |
|-----|----------|
| Buscar película | `GET /search/movie?query={query}` |
| Buscar serie | `GET /search/tv?query={query}` |
| Detalle película | `GET /movie/{id}` |
| Detalle serie | `GET /tv/{id}` |

**Ventajas:**
- ✅ API oficial con catálogo masivo
- ✅ Búsqueda por nombre nativa
- ✅ Soporte para películas Y series
- ✅ Imágenes incluidas (póster, backdrop)
- ✅ Puntuaciones y votos
- ✅ Idioma configurable (es-ES)

**Desventajas:**
- ⚠️ Requiere API key para producción
- ⚠️ Rate limit de ~40 req/segundo

---

## Estrategia de Implementación

1. **TMDB como API primaria** — Búsqueda por título, películas y series, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a MovieShow
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 1s
5. **Filtrado por tipo** — Parámetro `mediaType` opcional (MOVIE, TV, o ambos)

### Flujo de Búsqueda

```
1. Cliente → GET /api/movieshows/search?name=matrix&mediaType=MOVIE
2. Backend → TMDB API (/search/movie?query=matrix&language=es-ES)
3. Mapear a MovieSearchResult → Convertir a JSON → Devolver
4. Si no hay resultados → Lista vacía
```

### Flujo de Detalle

```
1. Cliente → GET /api/movieshows/{id}
2. Backend → MongoDB (película/serie ya persistida localmente)
3. Mapear a MovieShow → Convertir a JSON → Devolver
```

### Flujo de Creación

```
1. Cliente → POST /api/movieshows (body con datos de película/serie)
2. Backend → Validar fechas, verificar duplicados por externalId
3. Guardar en MongoDB → Devolver 201 Created
```

---

## Estado: ✅ COMPLETADA

Todos los pasos fueron implementados y verificados:

- [x] Crear enum `MovieStatus`: WATCHING, WATCHED, PLAN_TO_WATCH
- [x] Crear enum `MovieMediaType`: MOVIE, TV
- [x] Crear documento `MovieShow.java` en `domain/model/`
- [x] Crear documento `MovieSearchCriteria.java` en `domain/model/`
- [x] Crear documento `MovieSearchResult.java` en `domain/model/`
- [x] Crear interface `MovieShowUseCase.java` en `domain/port/in/`
- [x] Crear interface `MovieSearchUseCase.java` en `domain/port/in/`
- [x] Crear interface `MovieShowRepository.java` en `domain/port/out/`
- [x] Crear interface `ExternalMovieCatalogClient.java` en `domain/port/out/`
- [x] Crear `SpringDataMovieShowRepository.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `MovieShowEntity.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `MovieShowEntityMapper.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `MovieShowPersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `MovieShowService.java` en `application/service/`
- [x] Crear `MovieSearchService.java` en `application/service/`
- [x] Crear `TmdbClient.java` en `infrastructure/adapter/out/tmdb/`
- [x] Crear `MovieShowController.java` en `infrastructure/adapter/in/web/`
- [x] Crear `MovieShowRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `MovieShowResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `MovieShowDtoMapper.java` en `infrastructure/adapter/in/web/dto/`
- [x] Crear `MovieShowConflictException.java` en `application/exception/`
- [x] Crear `MovieShowNotFoundException.java` en `application/exception/`
- [x] Crear `TmdbClientConfig.java` en `infrastructure/config/`
- [x] Crear `StringToMovieStatusConverter.java` en `infrastructure/config/`
- [x] Crear `StringToMovieMediaTypeConverter.java` en `infrastructure/config/`
- [x] `MovieShowServiceTest.java` — Tests unitarios de CRUD
- [x] `MovieShowControllerTest.java` — Tests de integración MockMvc
- [x] `MovieSearchServiceTest.java` — Tests del flujo de búsqueda
- [x] `TmdbClientTest.java` — Tests del cliente TMDB con mock server
- [x] `MovieShowPersistenceAdapterTest.java` — Tests del adaptador de persistencia
- [x] `MovieShowDtoMapperTest.java` — Tests de mapeo DTO ↔ Domain
- [x] `MovieShowModelTest.java` — Tests del modelo de dominio

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/movieshows/search?name=matrix` devuelve resultados de TMDB
- [x] Los resultados incluyen: título, sinopsis, fecha de estreno, póster, puntuación, tipo
- [x] Se puede filtrar por tipo (MOVIE, TV) con el parámetro `mediaType`
- [x] El endpoint `GET /api/movieshows` devuelve lista vacía al inicio
- [x] Se puede crear una película/serie vía `POST /api/movieshows`
- [x] Se puede actualizar vía `PUT /api/movieshows/{id}`
- [x] Se puede eliminar vía `DELETE /api/movieshows/{id}`
- [x] Los filtros funcionan: por nombre, estado y tipo
- [x] Los tests pasan (`mvn verify`)
- [x] El respeta arquitectura hexagonal (dependencias hacia dentro)
- [x] Se evitan duplicados por externalId
- [x] Las fechas se validan correctamente (dateAdded vs dateCompleted)

---

## Estructura de Paquetes Final

```
com.wikicollection/
├── domain/
│   └── model/
│       ├── MovieShow
│       ├── MovieStatus
│       ├── MovieMediaType
│       ├── MovieSearchCriteria
│       └── MovieSearchResult
│   └── port/
│       ├── in/
│       │   ├── MovieShowUseCase
│       │   └── MovieSearchUseCase
│       └── out/
│           ├── MovieShowRepository
│           └── ExternalMovieCatalogClient
├── application/
│   ├── exception/
│   │   ├── MovieShowConflictException
│   │   └── MovieShowNotFoundException
│   └── service/
│       ├── MovieShowService
│       ├── MovieSearchService
│       └── DateRangeValidator
├── infrastructure/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   ├── MovieShowController
│   │   │   └── dto/
│   │   │       ├── MovieShowRequest
│   │   │       ├── MovieShowResponse
│   │   │       └── MovieShowDtoMapper
│   │   └── out/
│   │       ├── tmdb/
│   │       │   └── TmdbClient
│   │       └── persistence/
│   │           ├── MovieShowEntity
│   │           ├── MovieShowEntityMapper
│   │           ├── MovieShowPersistenceAdapter
│   │           └── SpringDataMovieShowRepository
│   └── config/
│       ├── TmdbClientConfig
│       ├── StringToMovieStatusConverter
│       └── StringToMovieMediaTypeConverter
```

---

## Configuración en application.properties

```properties
# TMDB API
tmdb.api-key=${TMDB_API_KEY:}
tmdb.api.base-url=https://api.themoviedb.org/3
tmdb.api.retry-attempts=3
tmdb.api.retry-delay-ms=1000
```

---

## Notas

- **TMDB es la API elegida** — Oficial, catálogo masivo, búsqueda por nombre, soporte para películas y series
- La búsqueda se hace por nombre con el query param `name` (consistente con el resto de entidades)
- Se busca en español (`language=es-ES`) pero los resultados pueden incluir contenido en otros idiomas
- Las imágenes usan la base URL `https://image.tmdb.org/t/p/w500` (póster) y `/t/p/original` (backdrop)
- El campo `externalId` es único en MongoDB para evitar duplicados
- El enum `MovieStatus` tiene 3 valores: WATCHING, WATCHED, PLAN_TO_WATCH (sin WISHLIST)
- El endpoint base es `/api/movieshows` (no `/api/movies`)
- Documentación TMDB: https://developer.themoviedb.org/docs

---

## Referencias

- [TMDB API Documentation](https://developer.themoviedb.org/docs)
- [TMDB API Key](https://www.themoviedb.org/settings/api)
- [TMDB Image Base URL](https://developer.themoviedb.org/docs/images)
- [TMDB Search Movies](https://developer.themoviedb.org/reference/search-movie)
- [TMDB Search TV](https://developer.themoviedb.org/reference/search-tv)
