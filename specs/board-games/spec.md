# Spec: Juegos de Mesa (Board Games)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** BoardGame (domain/model/BoardGame.java)

---

## Dominio

### Entidad BoardGame

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| title | String | ✅ | Nombre del juego de mesa |
| description | String | ❌ | Descripción del juego |
| yearPublished | Integer | ❌ | Año de publicación |
| minPlayers | Integer | ❌ | Mínimo de jugadores |
| maxPlayers | Integer | ❌ | Máximo de jugadores |
| minPlaytime | Integer | ❌ | Duración mínima (minutos) |
| maxPlaytime | Integer | ❌ | Duración máxima (minutos) |
| publisher | String | ❌ | Editorial/publicador |
| designers | List<String> | ❌ | Lista de diseñadores |
| categories | List<String> | ❌ | Categorías del juego |
| mechanics | List<String> | ❌ | Mecánicas de juego |
| imageUrl | String | ❌ | URL de imagen completa |
| thumbnailUrl | String | ❌ | URL de miniatura |
| bggRating | BigDecimal | ❌ | Rating promedio BGG (0-10) |
| bggId | String | ❌ | ID externo de BoardGameGeek |
| status | Enum (BoardGameStatus) | ✅ | OWNED, WISHLIST |
| notes | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió a la colección |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Enum

**BoardGameStatus:** `OWNED`, `WISHLIST`

> **Nota:** El enum original tenía 4 valores (OWNED, WISHLIST, PREVIOUSLY_OWNED, FOR_TRADE) pero la implementación real solo usa 2. AD-012: se redujo a OWNED y WISHLIST.

### Reglas de Negocio

- **BggId es único** — no se pueden duplicar juegos por ID de BGG
- **Status only has 2 values** — OWNED y WISHLIST (no PREVIOUSLY_OWNED ni FOR_TRADE)

---

## Puertos (Interfaces de Dominio)

### BoardGameUseCase (in)
```java
public interface BoardGameUseCase {
    BoardGame save(BoardGame game, String ownerId);
    BoardGame update(String id, BoardGame updates, String ownerId);
    void delete(String id, String ownerId);
    BoardGame findById(String id);
    Page<BoardGame> search(BoardGameSearchCriteria criteria, Pageable pageable);
}
```

### BoardGameSearchUseCase (in)
```java
public interface BoardGameSearchUseCase {
    List<BoardGameSearchResult> searchExternal(String query);
    BoardGameSearchResult findById(String bggId);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `BoardGameService` | CRUD de juegos de mesa + validaciones |
| `BoardGameSearchService` | Búsqueda externa BGG XML + parseo + mapeo |

---

## API Externa: BoardGameGeek XML API 2

- **Base URL:** `https://boardgamegeek.com/xmlapi2`
- **Auth:** No requerida
- **Rate limit:** Variable (~10 requests/segundo recomendado)
- **Gratis:** Sí
- **Formato:** XML (parseado con Jackson XML)
- **Total juegos:** 100,000+
- **Estado:** Activa y mantenida por BGG

### Endpoints usados

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /search?query={query}&type=boardgame` |
| Obtener detalle | `GET /thing?id={id}&stats=1` |
| Colección usuario | `GET /collection/{username}?own=1` |

### Mapeo BGG XML → BoardGameSearchResult

| Campo BGG | Campo SearchResult | Notas |
|-----------|--------------------|-------|
| `id` | `bggId` | ID externo de BGG |
| `name` | `title` | Nombre del juego |
| `yearpublished` | `yearPublished` | Año de publicación |
| `minplayers` | `minPlayers` | Mínimo de jugadores |
| `maxplayers` | `maxPlayers` | Máximo de jugadores |
| `minplaytime` | `minPlaytime` | Duración mínima |
| `maxplaytime` | `maxPlaytime` | Duración máxima |
| `description` | `description` | Descripción |
| `thumbnail` | `thumbnailUrl` | Miniatura |
| `image` | `imageUrl` | Imagen completa |
| `publisher` | `publisher` | Editorial |
| `designers` | `designers` | Lista de diseñadores |
| `categories` | `categories` | Categorías |
| `mechanics` | `mechanics` | Mecánicas |
| `rating` | `bggRating` | Rating promedio |

---

## Controlador REST

**Base:** `/api/v1/boardgames`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/boardgames` | Listar juegos de mesa (pagina, filtra por name/status/owner) |
| GET | `/api/v1/boardgames/{id}` | Obtener juego de mesa por ID |
| POST | `/api/v1/boardgames` | Crear juego de mesa (auth requerido) |
| PUT | `/api/v1/boardgames/{id}` | Actualizar juego de mesa (auth requerido, ownership verificado) |
| DELETE | `/api/v1/boardgames/{id}` | Eliminar juego de mesa (204 No Content, auth requerido) |
| GET | `/api/v1/boardgames/search?name={query}` | Buscar en BoardGameGeek (XML API) |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Buscar por título |
| `status` | Enum | Filtrar por estado (OWNED, WISHLIST) |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |

---

## DTOs

### BoardGameRequest (POST/PUT)

```json
{
  "title": "string (obligatorio)",
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer (min 1)",
  "maxPlayers": "integer (min 1)",
  "minPlaytime": "integer (min 1)",
  "maxPlaytime": "integer (min 1)",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number (0-10)",
  "bggId": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date"
}
```

### BoardGameResponse (GET)

```json
{
  "id": "string",
  "title": "string",
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer",
  "maxPlayers": "integer",
  "minPlaytime": "integer",
  "maxPlaytime": "integer",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number",
  "bggId": "string",
  "status": "OWNED | WISHLIST",
  "notes": "string",
  "dateAdded": "date",
  "ownerId": "string"
}
```

### BoardGameSearchResponse (resultado de búsqueda externa)

```json
{
  "bggId": "string",
  "title": "string",
  "description": "string",
  "yearPublished": "integer",
  "minPlayers": "integer",
  "maxPlayers": "integer",
  "minPlaytime": "integer",
  "maxPlaytime": "integer",
  "publisher": "string",
  "designers": ["string"],
  "categories": ["string"],
  "mechanics": ["string"],
  "imageUrl": "string",
  "thumbnailUrl": "string",
  "bggRating": "number",
  "externalSource": "string"
}
```

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `BoardGameNotFoundException` | 404 | Juego de mesa no encontrado |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `BoardGameServiceTest` | Unitario | CRUD de juegos de mesa |
| `BoardGameControllerTest` | Integración | Endpoints de juegos de mesa |
| `BoardGameSearchServiceTest` | Unitario | Búsqueda BGG |
| `BggXmlClientTest` | Unitario | Cliente BGG XML con mock |
| `BoardGamePersistenceAdapterTest` | Integración | Persistencia de juegos de mesa |
| `BoardGameXmlMapperTest` | Unitario | Mapeo XML → BoardGame |
| `BoardGameStatusMigrationTest` | Unitario | Migración de estado |
| `BoardGameServiceOwnerFilterTest` | Unitario | Filtro owner en board games (Fase 9) |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/boardgames/search?name=catan` devuelve resultados de BGG XML
- [x] Los resultados incluyen: título, año, jugadores (min/max), duración, publisher, diseñadores, categorías, mecánicas, imagen, rating
- [x] El endpoint `GET /api/v1/boardgames/{id}` devuelve el detalle completo de un juego
- [x] El endpoint `GET /api/v1/boardgames` devuelve lista vacía al inicio
- [x] Se puede crear un juego de mesa vía `POST /api/v1/boardgames` (auth requerido)
- [x] Se puede actualizar/eliminar un juego vía `PUT`/`DELETE /api/v1/boardgames/{id}` (auth + ownership)
- [x] Se puede buscar por nombre parcial (búsqueda flexible)
- [x] Los errores de API externa se manejan correctamente (503, 429, timeout)
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)

---

## Estado de Implementación

Fase 3 completada. Todos los componentes implementados:
- ✅ Domain: BoardGame.java, BoardGameStatus.java, BoardGameSearchCriteria.java, BoardGameSearchResult.java
- ✅ Ports: BoardGameUseCase.java, BoardGameSearchUseCase.java, BoardGameRepository.java, ExternalBoardGameCatalogClient.java
- ✅ Application: BoardGameService.java, BoardGameSearchService.java
- ✅ Infrastructure: BggXmlClient.java, BoardGameXmlMapper.java, BoardGameController.java, BoardGameEntity.java, BoardGamePersistenceAdapter.java, SpringDataBoardGameRepository.java, BoardGameDtoMapper.java, BoardGameSearchResponse.java
- ✅ Config: BggClientConfig.java, StringToBoardGameStatusConverter.java
- ✅ Tests: 8 archivos de test
