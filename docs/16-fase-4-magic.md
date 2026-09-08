# Fase 4: Colección de Cartas Magic: The Gathering

## Objetivo

Configurar el backend Java 25 + Spring Boot 4 (arquitectura hexagonal) para:

1. Hacer llamadas a la API externa **Scryfall** para buscar cartas de Magic: The Gathering
2. Devolver la información de cartas en el endpoint `GET /api/magic/search`
3. CRUD completo de cartas en la colección local (MongoDB)

---

## API Externa

### Scryfall API

- **Base URL:** `https://api.scryfall.com`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí, totalmente gratuita
- **Total cartas:** 70,000+
- **Formato:** JSON
- **Documentación:** https://scryfall.com/docs/api

**Endpoints útiles:**

| Uso | Endpoint |
|-----|----------|
| Buscar por nombre (fuzzy) | `GET /cards/named?fuzzy={name}` |
| Buscar por nombre (exacto) | `GET /cards/named?exact={name}` |
| Búsqueda avanzada | `GET /cards/search?q={query}` |
| Carta por ID | `GET /cards/{id}` |
| Cartas aleatorias | `GET /cards/random` |

**Ventajas:**
- ✅ Totalmente gratuita
- ✅ Sin autenticación
- ✅ Formato JSON nativo
- ✅ 70,000+ cartas
- ✅ Datos muy completos (precios, legalidades, imágenes, artista, etc.)
- ✅ Búsqueda fuzzy (tolerante a errores)
- ✅ Búsqueda avanzada con lenguaje de consulta
- ✅ API activa y mantenida

**Desventajas:**
- ❌ Rate limit de ~10 req/segundo
- ❌ Sin soporte oficial para español (cartas en inglés por defecto)

---

## Estrategia de Implementación

1. **Scryfall como API primaria** — Búsqueda por nombre, 70k+ cartas, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a DTOs de MagicCard
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Cache** — Caché de resultados (TTL 1 hora) para reducir llamadas

### Flujo de Búsqueda

```
1. Cliente → GET /api/magic/search?name=lightning+bolt
2. Backend → Scryfall API (/cards/named?fuzzy=lightning+bolt)
3. Mapear a DTO → Convertir a JSON → Devolver
4. Si no hay resultados → 404 Not Found
```

### Flujo de Detalle

```
1. Cliente → GET /api/magic/{id}
2. Backend → Scryfall API (/cards/{id})
3. Mapear a MagicCard detallado → Convertir a JSON → Devolver
```

---

## Estado: PLANIFICADO

---

## Pasos de Implementación

### Capa de Dominio

- [ ] Crear enum `MagicCardCondition`: MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR
- [ ] Crear enum `MagicCardLanguage`: ENGLISH, SPANISH, FRENCH, GERMAN, ITALIAN, PORTUGUESE, JAPANESE, CHINESE
- [ ] Crear documento `MagicCard.java` en `domain/model/`
  - id: String
  - scryfallId: String (ID externo de Scryfall)
  - oracleId: String
  - name: String
  - language: String
  - releaseDate: String
  - manaCost: String
  - convertedManaCost: Double
  - type: String
  - text: String
  - power: String
  - toughness: String
  - loyalty: String
  - colors: List<String>
  - colorIdentity: List<String>
  - keywords: List<String>
  - rarity: String
  - setCode: String
  - setName: String
  - artist: String
  - frame: String
  - borderColor: String
  - layout: String
  - legalities: Map<String, String>
  - priceUsd: String
  - priceEur: String
  - imageUrl: String
  - imageLargeUrl: String
  - artCropUrl: String
  - condition: MagicCardCondition
  - isFoil: Boolean
  - quantity: Integer
  - notes: String
  - dateAdded: LocalDateTime
- [ ] Crear interface `MagicCardRepository.java` en `domain/port/out/`
- [ ] Crear interface `ExternalMagicCardCatalogClient.java` en `domain/port/out/`
- [ ] Crear interface `MagicCardUseCase.java` en `domain/port/in/`
- [ ] Crear interface `MagicCardSearchUseCase.java` en `domain/port/in/`

### Capa de Aplicación

- [ ] Crear `MagicCardService.java` en `application/service/`
- [ ] Crear `MagicCardSearchService.java` en `application/service/`

### Capa de Infraestructura - Persistencia

- [ ] Crear `MagicCardEntity.java` en `infrastructure/adapter/out/persistence/`
- [ ] Crear `SpringDataMagicCardRepository.java` en `infrastructure/adapter/out/persistence/`
- [ ] Crear `MagicCardEntityMapper.java` en `infrastructure/adapter/out/persistence/`
- [ ] Crear `MagicCardPersistenceAdapter.java` en `infrastructure/adapter/out/persistence/`

### Capa de Infraestructura - Clientes Externos

- [ ] Crear `ScryfallClient.java` en `infrastructure/adapter/out/scryfall/`
  - Cliente con RestTemplate
  - Búsqueda fuzzy por nombre
  - Búsqueda avanzada con query
  - Obtener por ID
- [ ] Crear `MagicCardMapper.java` - Mapeo JSON → MagicCard

### Capa de Infraestructura - Web

- [ ] Crear `MagicCardController.java` en `infrastructure/adapter/in/web/`
- [ ] Crear `MagicCardRequest.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [ ] Crear `MagicCardResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [ ] Crear `MagicCardSearchResponse.java` (DTO) en `infrastructure/adapter/in/web/dto/`
- [ ] Crear `MagicCardDtoMapper.java` en `infrastructure/adapter/in/web/dto/`

### Configuración

- [ ] Agregar propiedades Scryfall a `application.properties`:
  ```properties
  # Scryfall API
  scryfall.api.base-url=https://api.scryfall.com
  scryfall.api.retry-attempts=3
  scryfall.api.retry-delay-ms=1000
  ```

### Tests

- [ ] `MagicCardServiceTest.java` — Tests unitarios de CRUD
- [ ] `MagicCardControllerTest.java` — Tests de integración MockMvc
- [ ] `ScryfallClientTest.java` — Tests del cliente Scryfall con mock server
- [ ] `MagicCardPersistenceAdapterTest.java` — Tests del adaptador de persistencia
- [ ] `MagicCardSearchServiceTest.java` — Tests del flujo de búsqueda
- [ ] `MagicCardMapperTest.java` — Tests de mapeo JSON → MagicCard

---

## Criterios de Aceptación

- [ ] El endpoint `GET /api/magic/search?name=lightning+bolt` devuelve resultados de Scryfall
- [ ] Los resultados incluyen: nombre, mana cost, tipo, texto, colores, rareza, set, artista, imagen, precios
- [ ] El endpoint `GET /api/magic/{id}` devuelve el detalle completo de una carta
- [ ] El endpoint `GET /api/magic` devuelve lista vacía al inicio
- [ ] Se puede crear una carta vía `POST /api/magic`
- [ ] Se puede actualizar/eliminar una carta vía `PUT`/`DELETE /api/magic/{id}`
- [ ] Los tests pasan (`mvn verify`)
- [ ] El respeta arquitectura hexagonal (dependencias hacia dentro)
- [ ] Se puede buscar por nombre parcial (búsqueda fuzzy)
- [ ] Los errores de API externa se manejan correctamente (503, 429, timeout)

---

## Estructura de Paquetes Esperada

```
com.wikicollection/
├── domain/
│   └── model/
│       ├── MagicCard
│       ├── MagicCardCondition
│       └── MagicCardLanguage
│   └── port/
│       ├── in/
│       │   ├── MagicCardUseCase
│       │   └── MagicCardSearchUseCase
│       └── out/
│           ├── MagicCardRepository
│           └── ExternalMagicCardCatalogClient
├── application/
│   └── service/
│       ├── MagicCardService
│       └── MagicCardSearchService
├── infrastructure/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   ├── MagicCardController
│   │   │   └── dto/
│   │   │       ├── MagicCardRequest
│   │   │       ├── MagicCardResponse
│   │   │       ├── MagicCardSearchResponse
│   │   │       └── MagicCardDtoMapper
│   │   └── out/
│   │       ├── scryfall/
│   │       │   └── ScryfallClient
│   │       └── persistence/
│   │           ├── MagicCardEntity
│   │           ├── SpringDataMagicCardRepository
│   │           ├── MagicCardEntityMapper
│   │           └── MagicCardPersistenceAdapter
│   └── config/
│       └── ScryfallClientConfig
```

---

## Configuración en application.properties

```properties
# Scryfall API
scryfall.api.base-url=https://api.scryfall.com
scryfall.api.retry-attempts=3
scryfall.api.retry-delay-ms=1000

# BoardGameGeek JSON API (Fase 3)
bgg.api.base-url=https://bgg.cc/api/v1
bgg.api.retry-attempts=3
bgg.api.retry-delay-ms=2000

# RAWG API Key (Fase 2)
rawg.api-key=${RAWG_API_KEY:}

# Google Books API Key (Fase 1)
google.books.api-key=${GOOGLE_BOOKS_API_KEY:}
```

---

## Notas

- **Scryfall es la API elegida** — Totalmente gratuita, sin auth, 70k+ cartas, JSON nativo
- La búsqueda se hace por nombre con el query param `name` (consistente con Books, Games, BoardGames)
- Se recomienda cachear la lista de cartas (TTL 1 hora) para evitar llamadas repetidas
- Scryfall soporta búsqueda avanzada con su propio lenguaje de consulta
- Las imágenes de cartas están en alta resolución
- Los precios se actualizan diariamente
- Documentación Scryfall: https://scryfall.com/docs/api

---

## Referencias

- [Scryfall API Documentation](https://scryfall.com/docs/api)
- [Scryfall Search Syntax](https://scryfall.com/docs/syntax)
- [Scryfall API Rate Limits](https://scryfall.com/docs/api#rate-limits)
