# Fase 7: Caché con Caffeine para APIs de Búsqueda

## Objetivo

Implementar **Caffeine** como caché en memoria para las APIs de búsqueda externas, aplicando **Spring Cache** como abstracción.

El objetivo es cachear temporalmente las respuestas de las APIs externas para:
- Evitar peticiones repetidas a APIs externas
- Mejorar el tiempo de respuesta
- Reducir el consumo de las APIs
- Evitar alcanzar límites de peticiones (rate limits)
- Disminuir la carga de servicios externos

---

## Arquitectura

```text
Cliente
   |
   v
Spring Boot
   |
   v
Service
   |
   +------> Caffeine Cache
   |              |
   |         ¿Existe?
   |          /       \
   |        Sí         No
   |        |           |
   |        v           v
   |     Respuesta   API externa
   |                    |
   |                    v
   |                 Caffeine
   |                    |
   +--------------------+
```

La aplicación utilizará **Spring Cache** como abstracción y **Caffeine** como implementación.

---

## APIs de Búsqueda a Cachear

| Servicio | API externa | Endpoint search | Caché | Clave |
|----------|-------------|-----------------|-------|-------|
| BookSearchService | Google Books | `GET /api/books/search?name={query}` | `bookSearch` | `#query` |
| GameSearchService | RAWG + FreeToGame | `GET /api/games/search?name={query}` | `gameSearch` | `#query` |
| BoardGameSearchService | BGG XML | `GET /api/boardgames/search?name={query}` | `boardGameSearch` | `#query` |
| MagicCardSearchService | Scryfall | `GET /api/magic/search?name={query}` | `magicCardSearch` | `#query` |
| DeckSearchService | Scryfall | búsqueda de comandantes | `deckSearch` | `#colors` |
| MovieSearchService | TMDB | `GET /api/movieshows/search?name={query}` | `movieSearch` | `#query` |

---

## Pasos de Implementación

### 1. Dependencias (pom.xml)

Añadir al `pom.xml` del backend:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

**Nota:** No es necesario utilizar Redis ni otro servidor externo. Caffeine funciona directamente en memoria.

---

### 2. Activar Spring Cache

Crear clase de configuración:

```java
package com.wikicollection.config;

import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableCaching
public class CacheConfig {
}
```

La anotación `@EnableCaching` permite utilizar `@Cacheable`, `@CacheEvict` y `@CachePut` en los servicios.

---

### 3. Configurar Caffeine en application.yml

```yaml
spring:
  cache:
    type: caffeine
    cache-names:
      - bookSearch
      - gameSearch
      - boardGameSearch
      - magicCardSearch
      - deckSearch
      - movieSearch
    caffeine:
      spec: maximumSize=500,expireAfterWrite=2h
```

**Política inicial:**
- `maximumSize=500` — máximo 500 entradas por caché
- `expireAfterWrite=2h` — entradas expiran 2 horas después de escribirse

**Justificación del TTL de 2 horas:**
- Los datos de libros, videojuegos, películas y cartas son relativamente estables
- Un TTL de 2 horas evita peticiones repetidas sin datos obsoletos prolongados
- Ajustable según necesidades (ver sección de TTL)

---

### 4. Añadir @Cacheable a BookSearchService

```java
@Service
public class BookSearchService implements BookSearchUseCase {

    private final ExternalBookCatalogClient externalBookCatalogClient;

    public BookSearchService(ExternalBookCatalogClient externalBookCatalogClient) {
        this.externalBookCatalogClient = externalBookCatalogClient;
    }

    @Cacheable(
        value = "bookSearch",
        key = "#query",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<BookSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            throw new IllegalArgumentException("El parámetro de búsqueda 'query' es obligatorio");
        }
        return externalBookCatalogClient.search(query);
    }
}
```

**Clave:** `#query` — la búsqueda por texto completo como clave de caché.

**Condición `unless`:** No cachear resultados nulos ni listas vacías para no llenar la caché con entradas inútiles.

---

### 5. Añadir @Cacheable a GameSearchService

```java
@Service
public class GameSearchService implements GameSearchUseCase {

    private final ExternalGameCatalogClient rawgClient;
    private final ExternalGameCatalogClient freeToGameClient;

    public GameSearchService(
            @Qualifier("rawgClient") ExternalGameCatalogClient rawgClient,
            @Qualifier("freeToGameClient") ExternalGameCatalogClient freeToGameClient) {
        this.rawgClient = rawgClient;
        this.freeToGameClient = freeToGameClient;
    }

    @Cacheable(
        value = "gameSearch",
        key = "#query",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<GameSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            throw new IllegalArgumentException("El parámetro de búsqueda 'query' es obligatorio");
        }
        List<GameSearchResult> results = rawgClient.search(query);
        if (results.isEmpty()) {
            return freeToGameClient.search(query);
        }
        return results;
    }
}
```

**Nota:** El flujo de fallback RAWG → FreeToGame se mantiene. La caché almacena el resultado final del fallback completo.

---

### 6. Añadir @Cacheable a BoardGameSearchService

```java
@Service
public class BoardGameSearchService implements BoardGameSearchUseCase {

    private final ExternalBoardGameCatalogClient bggClient;

    public BoardGameSearchService(ExternalBoardGameCatalogClient bggClient) {
        this.bggClient = bggClient;
    }

    @Cacheable(
        value = "boardGameSearch",
        key = "#query",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<BoardGameSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            throw new IllegalArgumentException("El parámetro de búsqueda 'query' es obligatorio");
        }
        return bggClient.search(query);
    }
}
```

---

### 7. Añadir @Cacheable a MagicCardSearchService

```java
@Service
public class MagicCardSearchService implements MagicCardSearchUseCase {

    private final ExternalMagicCardCatalogClient magicCardCatalogClient;

    public MagicCardSearchService(ExternalMagicCardCatalogClient magicCardCatalogClient) {
        this.magicCardCatalogClient = magicCardCatalogClient;
    }

    @Cacheable(
        value = "magicCardSearch",
        key = "#query",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<MagicCardSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            throw new IllegalArgumentException("El parámetro de búsqueda 'query' es obligatorio");
        }
        return magicCardCatalogClient.search(query);
    }
}
```

---

### 8. Añadir @Cacheable a DeckSearchService

```java
@Service
public class DeckSearchService implements DeckSearchUseCase {

    private final ExternalMagicCardCatalogClient magicCardCatalogClient;

    public DeckSearchService(ExternalMagicCardCatalogClient magicCardCatalogClient) {
        this.magicCardCatalogClient = magicCardCatalogClient;
    }

    @Cacheable(
        value = "deckSearch",
        key = "#colors",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<MagicCardSearchResult> searchCommanders(String colors) {
        if (colors == null || colors.isBlank()) {
            throw new IllegalArgumentException("El parámetro 'colors' es obligatorio");
        }
        return magicCardCatalogClient.searchCommanders(colors);
    }
}
```

---

### 9. Añadir @Cacheable a MovieSearchService

```java
@Service
public class MovieSearchService implements MovieSearchUseCase {

    private final ExternalMovieCatalogClient movieCatalogClient;

    public MovieSearchService(ExternalMovieCatalogClient movieCatalogClient) {
        this.movieCatalogClient = movieCatalogClient;
    }

    @Cacheable(
        value = "movieSearch",
        key = "#query",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<MovieSearchResult> search(String query) {
        if (query == null || query.isBlank()) {
            throw new IllegalArgumentException("El parámetro de búsqueda 'query' es obligatorio");
        }
        return movieCatalogClient.search(query);
    }

    @Cacheable(
        value = "movieSearch",
        key = "#query + ':' + #mediaType",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<MovieSearchResult> search(String query, MovieMediaType mediaType) {
        if (query == null || query.isBlank()) {
            throw new IllegalArgumentException("El parámetro de búsqueda 'query' es obligatorio");
        }
        return movieCatalogClient.search(query, mediaType);
    }
}
```

**Nota:** Para el método con `mediaType`, la clave incluye ambos parámetros para diferenciar búsquedas de películas vs series.

---

### 10. Añadir logging para verificar caché

En cada API client, añadir log para verificar que la caché funciona:

```java
@Service
public class GoogleBooksClient implements ExternalBookCatalogClient {

    public List<BookSearchResult> search(String query) {
        log.info("Consultando Google Books API para query: {}", query);
        // ... llamada HTTP
    }
}
```

**Verificación:** Si se hace la misma búsqueda 3 veces, el log debe aparecer solo 1 vez mientras la entrada esté en caché.

---

### 11. Tests de caché

Crear test para verificar que la caché funciona:

```java
@SpringBootTest
class BookSearchServiceCacheTest {

    @Autowired
    private BookSearchService bookSearchService;

    @MockBean
    private ExternalBookCatalogClient externalBookCatalogClient;

    @Test
    void search_cachesResults() {
        when(externalBookCatalogClient.search("tolkien"))
            .thenReturn(List.of(new BookSearchResult(...)));

        // Primera llamada — debe invocar al cliente externo
        bookSearchService.search("tolkien");
        verify(externalBookCatalogClient, times(1)).search("tolkien");

        // Segunda llamada — debe usar caché, NO invocar al cliente externo
        bookSearchService.search("tolkien");
        verify(externalBookCatalogClient, times(1)).search("tolkien");
    }
}
```

---

## Configuración Avanzada (Opcional)

### Configuración por Java (alternativa a YAML)

Si se necesita más control, configurar Caffeine mediante Java:

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();

        CaffeineCache bookSearch = new CaffeineCache(
            "bookSearch",
            Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(Duration.ofHours(24))
                .build()
        );

        CaffeineCache gameSearch = new CaffeineCache(
            "gameSearch",
            Caffeine.newBuilder()
                .maximumSize(500)
                .expireAfterWrite(Duration.ofHours(6))
                .build()
        );

        cacheManager.setCaches(List.of(bookSearch, gameSearch));
        return cacheManager;
    }
}
```

### TTL recomendados por tipo de dato

| Caché | TTL recomendado | Razón |
|-------|-----------------|-------|
| `bookSearch` | 24h | Datos de libros muy estables |
| `gameSearch` | 6h | Datos de videojuegos relativamente estables |
| `boardGameSearch` | 24h | Datos de juegos de mesa muy estables |
| `magicCardSearch` | 12h | Datos de cartas estables, precios pueden variar |
| `deckSearch` | 12h | Lista de comandantes estable |
| `movieSearch` | 6h | Películas/series pueden añadirse nuevas |

---

## Invalidación de Caché

### Invalidar una entrada específica

```java
@CacheEvict(value = "bookSearch", key = "#query")
public void evictBookSearch(String query) {
}
```

### Limpiar toda una caché

```java
@CacheEvict(value = "bookSearch", allEntries = true)
public void clearBookSearchCache() {
}
```

### Forzar actualización de una entrada

```java
@CachePut(value = "bookSearch", key = "#query")
public List<BookSearchResult> refreshBookSearch(String query) {
    return externalBookCatalogClient.search(query);
}
```

---

## Consideraciones Importantes

### 1. No cachear datos propios de usuario

Los datos de la colección del usuario (valoraciones, notas, fechas, estado) deben estar en MongoDB, NO en Caffeine.

**Correcto cachear:**
- Título, autor, descripción, portada de libros
- Título, género, plataforma de videojuegos
- Metadatos de películas/series

**NO cachear:**
- Valoraciones del usuario
- Notas personales
- Estado de lectura/juego
- Fechas de adquisición

### 2. No cachear respuestas erróneas

Las excepciones de la API no deben convertirse en valores de caché. Los métodos deben lanzar excepción cuando la API falle.

### 3. No cachear valores nulos

Usar `unless = "#result == null"` para evitar llenar la caché con resultados inexistentes.

### 4. Spring AOP y llamadas internas

`@Cacheable` funciona mediante proxies de Spring. Una llamada interna dentro del mismo bean no activa la caché. Los métodos cacheados deben ser llamados desde otros beans.

### 5. Clave de caché

La clave debe representar de forma inequívoca los parámetros que afectan al resultado:
- Búsqueda simple: `#query`
- Búsqueda con filtro: `#query + ':' + #mediaType`

---

## Estructura de Paquetes Final

```
com.wikicollection/
├── config/
│   └── CacheConfig.java          ← NUEVO
├── application/
│   └── service/
│       ├── BookSearchService.java      ← MODIFICADO (@Cacheable)
│       ├── GameSearchService.java      ← MODIFICADO (@Cacheable)
│       ├── BoardGameSearchService.java ← MODIFICADO (@Cacheable)
│       ├── MagicCardSearchService.java ← MODIFICADO (@Cacheable)
│       ├── DeckSearchService.java      ← MODIFICADO (@Cacheable)
│       └── MovieSearchService.java     ← MODIFICADO (@Cacheable)
```

---

## Estado: 📋 PLANIFICADA

- [ ] Añadir dependencias `spring-boot-starter-cache` y `caffeine` al pom.xml
- [ ] Crear `CacheConfig.java` con `@EnableCaching`
- [ ] Configurar cachés en `application.yml`
- [ ] Añadir `@Cacheable` a `BookSearchService`
- [ ] Añadir `@Cacheable` a `GameSearchService`
- [ ] Añadir `@Cacheable` a `BoardGameSearchService`
- [ ] Añadir `@Cacheable` a `MagicCardSearchService`
- [ ] Añadir `@Cacheable` a `DeckSearchService`
- [ ] Añadir `@Cacheable` a `MovieSearchService`
- [ ] Añadir logging en clientes externos para verificación
- [ ] Crear tests de caché para cada servicio
- [ ] Verificar que los tests pasan (`mvn verify`)

---

## Criterios de Aceptación

- [ ] Las búsquedas repetidas en `GET /api/books/search?name=X` usan caché
- [ ] Las búsquedas repetidas en `GET /api/games/search?name=X` usan caché
- [ ] Las búsquedas repetidas en `GET /api/boardgames/search?name=X` usan caché
- [ ] Las búsquedas repetidas en `GET /api/magic/search?name=X` usan caché
- [ ] Las búsquedas repetidas en `GET /api/movieshows/search?name=X` usan caché
- [ ] Los resultados nulos o vacíos no se cachean
- [ ] Las excepciones de API no se cachean
- [ ] Los datos de colección del usuario NO se cachean (solo MongoDB)
- [ ] Los tests de caché pasan (`mvn verify`)
- [ ] El TTL es configurable sin recompilar
- [ ] La caché funciona en memoria sin servidor externo (sin Redis)

---

## Notas

- Caffeine funciona en memoria, no requiere servidor externo
- Spring Cache permite cambiar a Redis en el futuro si es necesario
- El TTL debe ajustarse según la frecuencia de cambio de los datos de cada API
- La configuración por Java permite diferentes TTL por caché
- La configuración por YAML es más sencilla y recomendada para empezar
- Documentación Caffeine: https://github.com/ben-manes/caffeine
- Documentación Spring Cache: https://docs.spring.io/spring-framework/reference/integration/cache.html
