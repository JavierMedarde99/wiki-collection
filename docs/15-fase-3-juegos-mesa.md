# Fase 3: Colección de Juegos de Mesa

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 (arquitectura hexagonal) para:

1. Hacer llamadas a la API externa **BoardGameGeek XML API 2** para buscar juegos de mesa
2. Devolver la información de juegos de mesa en el endpoint `GET /api/boardgames/search`
3. CRUD completo de juegos de mesa en la colección local (MongoDB)

---

## APIs Externas

### BoardGameGeek XML API 2 (ÚNICA)

- **Base URL:** `https://boardgamegeek.com/xmlapi2`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí
- **Formato:** XML (parseado con Jackson XML)
- **Total juegos:** 100,000+
- **Documentación:** https://boardgamegeek.com/wiki/page/BGG_XML_API2

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
- ❌ Formato XML (requiere parseo con Jackson XML)
- ❌ Rate limit estricto (10 req/segundo)
- ❌ API asíncrona en algunos endpoints (devuelve 202 Accepted mientras procesa)

---

## Estrategia de Implementación

1. **BGG XML API 2 como única fuente** — Búsqueda por nombre, 100k+ juegos, parseo XML con Jackson
2. **Mapeo a dominio** — Convertir XML a DTOs de BoardGame unificados
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 2s (BGG devuelve 202 Accepted mientras procesa)

### Flujo de Búsqueda

```
1. Cliente → GET /api/boardgames/search?name=catan
2. Backend → BGG XML API (search?query=catan)
3. Parsear XML → Mapear a DTO → Convertir a JSON → Devolver
```

### Flujo de Detalle

```
1. Cliente → GET /api/boardgames/{id}
2. Backend → MongoDB (ya persistido)
3. Mapear a BoardGame detallado → Convertir a JSON → Devolver
```

---

## Estado: ✅ COMPLETADA

Todos los pasos fueron implementados y verificados:

- [x] Crear enum `BoardGameStatus`: OWNED, WISHLIST
- [x] Crear documento `BoardGame.java` en `domain/model/`
- [x] Crear interface `BoardGameRepository.java` en `domain/port/out/`
- [x] Crear interface `ExternalBoardGameCatalogClient.java` en `domain/port/out/`
- [x] Crear interface `BoardGameUseCase.java` en `domain/port/in/`
- [x] Crear interface `BoardGameSearchUseCase.java` en `domain/port/in/`
- [x] Crear `SpringDataBoardGameRepository.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `BoardGameEntity.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `BoardGameEntityMapper.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `BoardGamePersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`
- [x] Crear `BoardGameService.java` en `application/service/`
- [x] Crear `BoardGameSearchService.java` en `application/service/`
- [x] Crear `BggXmlClient.java` en `infrastructure/adapter/out/bgg/xml/`
- [x] Crear `BoardGameXmlMapper.java` - Mapeo XML → BoardGame
- [x] Crear `BoardGameController.java` en `infrastructure/adapter/in/web/`
- [x] Crear `BoardGameRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `BoardGameResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `BoardGameSearchResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [x] Crear `BoardGameDtoMapper.java` en `infrastructure/adapter/in/web/dto/`
- [x] Crear `BggClientConfig.java` en `infrastructure/config/`
- [x] `BoardGameServiceTest.java` — Tests unitarios de CRUD
- [x] `BoardGameControllerTest.java` — Tests de integración MockMvc
- [x] `BggXmlClientTest.java` — Tests del cliente BGG XML con mock server
- [x] `BoardGamePersistenceAdapterTest.java` — Tests del adaptador de persistencia
- [x] `BoardGameSearchServiceTest.java` — Tests del flujo de búsqueda
- [x] `BoardGameXmlMapperTest.java` — Tests de mapeo XML → BoardGame
- [x] `BoardGameStatusMigrationTest.java` — Tests de migración de estado

## Criterios de Aceptación

- [x] El endpoint `GET /api/boardgames/search?name=catan` devuelve resultados de BGG XML
- [x] Los resultados incluyen: título, año, jugadores (min/max), duración, publisher, diseñadores, categorías, mecánicas, imagen, rating
- [x] El endpoint `GET /api/boardgames/{id}` devuelve el detalle completo de un juego
- [x] El endpoint `GET /api/boardgames` devuelve lista vacía al inicio
- [x] Se puede crear un juego de mesa vía `POST /api/boardgames`
- [x] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/boardgames/{id}`
- [x] Los tests pasan (`mvn verify`)
- [x] El respeta arquitectura hexagonal (dependencias hacia dentro)
- [x] Se puede buscar por nombre parcial (búsqueda flexible)
- [x] Los errores de API externa se manejan correctamente (503, 429, timeout)

---

## Estructura de Paquetes Final

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
│   │       │   ├── xml/
│   │       │   │   └── BggXmlClient
│   │       │   └── mapper/
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
# BoardGameGeek XML API (ÚNICA)
bgg.api.xml-url=https://boardgamegeek.com/xmlapi2
bgg.api.retry-attempts=3
bgg.api.retry-delay-ms=2000

# RAWG API Key (Fase 2)
rawg.api-key=${RAWG_API_KEY:}

# Google Books API Key (Fase 1)
google.books.api-key=${GOOGLE_BOOKS_API_KEY:}
```

---

## Notas

- **BGG XML API 2 es la única fuente** — Búsqueda por nombre, 100k+ juegos, parseo XML con Jackson
- La búsqueda se hace por título con el query param `name` (consistente con Games y Books)
- **Importante:** BGG XML API devuelve 202 Accepted en algunos casos y hay que reintentar
- Se recomienda cachear la lista de juegos (TTL 1 hora) para evitar llamadas repetidas
- **Librería usada:** Jackson XML (`com.fasterxml.jackson.dataformat:jackson-dataformat-xml`)
- Documentación BGG XML2: https://boardgamegeek.com/wiki/page/BGG_API2

---

## Referencias

- [BGG XML API2](https://boardgamegeek.com/wiki/page/BGG_API2)
- [BGG API Terms of Use](https://boardgamegeek.com/wiki/page/XML_API_Terms_Of_Use)
- [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml)
