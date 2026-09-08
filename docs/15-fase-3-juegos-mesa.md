# Fase 3: Colección de Juegos de Mesa

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 (arquitectura hexagonal) para:

1. Hacer llamadas a las APIs externas **BoardGameGeek XML2** (principal) y **BoardGameGeek JSON** (secundaria) para buscar juegos de mesa
2. Devolver la información de juegos de mesa en el endpoint `GET /api/boardgames/search`
3. CRUD completo de juegos de mesa en la colección local (MongoDB)

---

## APIs Externas

### BoardGameGeek JSON API (PRINCIPAL)

- **Base URL:** `https://bgg.cc/api/v1`
- **Auth:** API Key/None
- **Rate limit:** Variable
- **Gratis:** Sí
- **Formato:** JSON
- **Total juegos:** 100,000+ (mismo catálogo que XML)
- **Documentación:** https://bgg.github.io/
- **Estado:** No oficial, pero devuelve JSON nativo

**Endpoints conocidos:**

| Uso | Endpoint |
|-----|----------|
| Buscar | `GET /search?query={query}` |
| Obtener juego | `GET /thing/{id}` |
| Colección | `GET /collection/{username}` |

**Ventajas:**
- ✅ Formato JSON nativo (sin necesidad de parsear XML)
- ✅ Más fácil de integrar con Spring Boot
- ✅ Mismo catálogo de 100k+ juegos
- ✅ Datos completos (publisher, diseñadores, categorías, mecánicas, ratings, imágenes)

**Desventajas:**
- ❌ API no oficial (puede desaparecer)
- ❌ Acceso inestable (Cloudflare protection)
- ❌ No garantiza disponibilidad a largo plazo

---

### BoardGameGeek XML API 2 (SECUNDARIA)

- **Base URL:** `https://boardgamegeek.com/xmlapi2`
- **Auth:** Cookies de sesión (login en BGG)
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí, con registro gratuito
- **Formato:** XML (requiere parseo)
- **Total juegos:** 100,000+
- **Documentación:** https://boardgamegeek.com/wiki/page/BGG_API2

**Endpoints útiles:**

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /search?query={query}&type=boardgame` |
| Obtener detalle | `GET /thing?id={id}&stats=1` |
| Colección usuario | `GET /collection/{username}?own=1` |

**Ventajas:**
- ✅ Catálogo más grande del mundo (100k+ juegos)
- ✅ API oficial mantenida por BGG
- ✅ Datos muy completos
- ✅ Comunidad activa y actualizada

**Desventajas:**
- ❌ Formato XML (requiere parseo con Jackson XML o JAXB)
- ❌ Requiere cookies de sesión para autenticación
- ❌ Rate limit estricto (10 req/segundo)
- ❌ API asíncrona en algunos endpoints (devuelve 202 Accepted mientras procesa)

---

## Estrategia de Implementación

1. **BGG JSON como primaria** — Búsqueda por nombre, 100k+ juegos, JSON nativo
2. **BGG XML2 como secundaria** — Fallback si JSON no disponible, requiere parseo XML
3. **Mapeo a dominio** — Convertir JSON/XML a DTOs de BoardGame unificados
4. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
5. **Cache** — Caché de resultados (TTL 1 hora) para reducir llamadas

### Flujo de Búsqueda

```
1. Cliente → GET /api/boardgames/search?name=catan
2. Backend → BGG JSON API (search)
3. Si hay resultados → Mapear a DTO → Convertir a JSON → Devolver
4. Si no hay resultados → BGG XML API (search)
5. Parsear XML → Mapear a DTO → Convertir a JSON → Devolver
6. Si no hay resultados → 404 Not Found
```

### Flujo de Detalle

```
1. Cliente → GET /api/boardgames/{id}
2. Backend → BGG JSON API (thing/{id})
3. Mapear a BoardGame detallado → Convertir a JSON → Devolver
```

---

## Estado: PLANIFICADO

---

## Pasos de Implementación

### Capa de Dominio

- [ ] Crear enum `BoardGameStatus`: OWNED, WISHLIST, PREVIOUSLY_OWNED, FOR_TRADE
- [ ] Crear documento `BoardGame.java` en `domain/model/`
  - id: String
  - title: String
  - description: String
  - yearPublished: Integer
  - minPlayers: Integer
  - maxPlayers: Integer
  - minPlaytime: Integer
  - maxPlaytime: Integer
  - publisher: String
  - designers: List<String>
  - categories: List<String>
  - mechanics: List<String>
  - imageUrl: String
  - thumbnailUrl: String
  - bggRating: Double
  - bggId: String (ID externo de BGG)
  - status: BoardGameStatus
  - notes: String (notas personales)
  - dateAdded: LocalDateTime
- [ ] Crear interface `BoardGameRepository.java` en `domain/port/out/`
- [ ] Crear interface `ExternalBoardGameCatalogClient.java` en `domain/port/out/`
- [ ] Crear interface `BoardGameUseCase.java` en `domain/port/in/`
- [ ] Crear interface `BoardGameSearchUseCase.java` en `domain/port/in/`

### Capa de Aplicación

- [ ] Crear `BoardGameService.java` en `application/service/`
- [ ] Crear `BoardGameSearchService.java` en `application/service/`

### Capa de Infraestructura - Persistencia

- [ ] Crear `BoardGameEntity.java` en `infrastructure/adapter/out/persistence/`
- [ ] Crear `SpringDataBoardGameRepository.java` en `infrastructure/adapter/out/persistence/`
- [ ] Crear `BoardGameEntityMapper.java` en `infrastructure/adapter/out/persistence/`
- [ ] Crear `BoardGamePersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`

### Capa de Infraestructura - Clientes Externos

- [ ] Crear `BggJsonClient.java` en `infrastructure/adapter/out/bgg/json/`
  - Cliente primario con RestTemplate
  - JSON nativo (más sencillo)
- [ ] Crear `BggXmlClient.java` en `infrastructure/adapter/out/bgg/xml/`
  - Usar RestTemplate con cookies de autenticación
  - Usar Jackson XML para parsear respuestas
  - Implementar reintentos en caso de 202 Accepted
- [ ] Crear `BoardGameJsonMapper.java` - Mapeo JSON → BoardGame
- [ ] Crear `BoardGameXmlMapper.java` - Mapeo XML → BoardGame

### Capa de Infraestructura - Web

- [ ] Crear `BoardGameController.java` en `infrastructure/adapter/in/web/`
- [ ] Crear `BoardGameRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [ ] Crear `BoardGameResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [ ] Crear `BoardGameSearchResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [ ] Crear `BoardGameDtoMapper.java` en `infrastructure/adapter/in/web/dto/`

### Configuración

- [ ] Agregar propiedades BGG a `application.properties`:
  ```properties
  # BoardGameGeek API
  bgg.api.base-url=https://boardgamegeek.com/xmlapi2
  bgg.api.json-url=https://bgg.cc/api/v1
  bgg.auth.username=${BGG_USERNAME:}
  bgg.auth.password=${BGG_PASSWORD:}
  bgg.api.retry-attempts=3
  bgg.api.retry-delay-ms=2000
  ```

### Tests

- [ ] `BoardGameServiceTest.java` — Tests unitarios de CRUD
- [ ] `BoardGameControllerTest.java` — Tests de integración MockMvc
- [ ] `BggXmlClientTest.java` — Tests del cliente BGG XML con mock server
- [ ] `BggJsonClientTest.java` — Tests del cliente BGG JSON con mock server
- [ ] `BoardGamePersistenceAdapterTest.java` — Tests del adaptador de persistencia
- [ ] `BoardGameSearchServiceTest.java` — Tests del flujo de búsqueda con fallback
- [ ] `BoardGameXmlMapperTest.java` — Tests de mapeo XML → BoardGame
- [ ] `BoardGameJsonMapperTest.java` — Tests de mapeo JSON → BoardGame

---

## Criterios de Aceptación

- [ ] El endpoint `GET /api/boardgames/search?name=catan` devuelve resultados de BGG XML
- [ ] Si BGG XML no devuelve resultados, el endpoint usa BGG JSON como fallback
- [ ] Los resultados incluyen: título, año, jugadores (min/max), duración, publisher, diseñadores, categorías, mecánicas, imagen, rating
- [ ] El endpoint `GET /api/boardgames/{id}` devuelve el detalle completo de un juego
- [ ] El endpoint `GET /api/boardgames` devuelve lista vacía al inicio
- [ ] Se puede crear un juego de mesa vía `POST /api/boardgames`
- [ ] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/boardgames/{id}`
- [ ] Los tests pasan (`mvn verify`)
- [ ] El respeta arquitectura hexagonal (dependencias hacia dentro)
- [ ] El flujo de búsqueda con fallback funciona correctamente
- [ ] Se puede buscar por nombre parcial (búsqueda flexible)
- [ ] Los errores de API externa se manejan correctamente (503, 429, timeout)

---

## Estructura de Paquetes Esperada

```
com.wikicollection/
├── domain/
│   └── model/
│       ├── BoardGame
│       ├── BoardGameStatus
│       └── BoardGameSearchResult
│   └── port/
│       ├── in/
│       │   ├── BoardGameUseCase
│       │   └── BoardGameSearchUseCase
│       └── out/
│           ├── BoardGameRepository
│           └── ExternalBoardGameCatalogClient
├── application/
│   └── service/
│       ├── BoardGameService
│       └── BoardGameSearchService
├── infrastructure/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   ├── BoardGameController
│   │   │   └── dto/
│   │   │       ├── BoardGameRequest
│   │   │       ├── BoardGameResponse
│   │   │       ├── BoardGameSearchResponse
│   │   │       └── BoardGameDtoMapper
│   │   └── out/
│   │       ├── bgg/
│   │       │   ├── json/
│   │       │   │   └── BggJsonClient
│   │       │   ├── xml/
│   │       │   │   └── BggXmlClient
│   │       │   └── mapper/
│   │       │       ├── BoardGameJsonMapper
│   │       │       └── BoardGameXmlMapper
│   │       └── persistence/
│   │           ├── BoardGameEntity
│   │           ├── SpringDataBoardGameRepository
│   │           ├── BoardGameEntityMapper
│   │           └── BoardGamePersistenceAdapter
│   └── config/
│       └── BggClientConfig
```

---

## Configuración en application.properties

```properties
# BoardGameGeek JSON API (PRINCIPAL)
bgg.api.base-url=https://bgg.cc/api/v1
bgg.api.retry-attempts=3
bgg.api.retry-delay-ms=2000

# BoardGameGeek XML API (SECUNDARIA - fallback)
bgg.api.xml-url=https://boardgamegeek.com/xmlapi2
bgg.auth.username=${BGG_USERNAME:}
bgg.auth.password=${BGG_PASSWORD:}

# RAWG API Key (ya existente)
rawg.api-key=${RAWG_API_KEY:}

# Google Books API Key (ya existente)
google.books.api-key=${GOOGLE_BOOKS_API_KEY:}
```

---

## Notas

- **BGG XML2 es la API principal** — Catálogo más grande del mundo (100k+ juegos), datos muy completos
- **BGG JSON es la API secundaria** — Fallback con mismo catálogo, formato más sencillo, pero inestable
- La búsqueda se hace por título con el query param `name` (consistente con Games y Books)
- **Importante:** BGG XML API devuelve 202 Accepted en algunos casos y hay que reintentar
- Se recomienda cachear la lista de juegos (TTL 1 hora) para evitar llamadas repetidas
- La autenticación de BGG requiere login previo para obtener cookies
- **Librería recomendada:** Jackson XML (`com.fasterxml.jackson.dataformat:jackson-dataformat-xml`)
- Documentación BGG XML2: https://boardgamegeek.com/wiki/page/BGG_API2
- Documentación BGG JSON (no oficial): https://bgg.github.io/

---

## Referencias

- [BGG XML API2](https://boardgamegeek.com/wiki/page/BGG_API2)
- [BGG API Terms of Use](https://boardgamegeek.com/wiki/page/XML_API_Terms_Of_Use)
- [BGG JSON API (no oficial)](https://bgg.github.io/)
- [pyBGG - Python BGG API](https://github.com/jaramir/pyBGG)
- [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml)
