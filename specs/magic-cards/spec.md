# Spec: Cartas Magic: The Gathering (Magic Cards)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** MagicCard (domain/model/MagicCard.java)

---

## Dominio

### Entidad MagicCard

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| scryfallId | String | ❌ | ID único de Scryfall |
| oracleId | String | ❌ | ID de oracle (único por carta) |
| name | String | ✅ | Nombre de la carta |
| language | Enum (MagicCardLanguage) | ❌ | ENGLISH, SPANISH, FRENCH, GERMAN, ITALIAN, PORTUGUESE, JAPANESE, CHINESE |
| releaseDate | String | ❌ | Fecha de lanzamiento del set |
| manaCost | String | ❌ | Coste de maná (ej: `{2}{R}{R}`) |
| convertedManaCost | Double | ❌ | Coste de maná convertido |
| type | String | ❌ | Tipo de carta (ej: "Instant", "Creature") |
| text | String | ❌ | Texto/rules de la carta |
| power | String | ❌ | Fuerza (criaturas) |
| toughness | String | ❌ | Resistencia (criaturas) |
| loyalty | String | ❌ | Lealtad (planeswalkers) |
| colors | List<String> | ❌ | Colores de la carta |
| colorIdentity | List<String> | ❌ | Identidad de color |
| keywords | List<String> | ❌ | Palabras clave |
| rarity | String | ❌ | Rareza (common, uncommon, rare, mythic) |
| setCode | String | ❌ | Código del set |
| setName | String | ❌ | Nombre del set |
| artist | String | ❌ | Artista de la ilustración |
| frame | String | ❌ | Año del frame |
| borderColor | String | ❌ | Color del borde |
| layout | String | ❌ | Layout de la carta |
| legalities | Map<String,String> | ❌ | Legalidades por formato |
| priceUsd | String | ❌ | Precio en USD |
| priceEur | String | ❌ | Precio en EUR |
| imageUrl | String | ❌ | URL de imagen normal |
| imageLargeUrl | String | ❌ | URL de imagen grande |
| artCropUrl | String | ❌ | URL de arte recortado |
| condition | Enum (MagicCardCondition) | ❌ | MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR |
| isFoil | Boolean | ❌ | Si es foil |
| quantity | Integer | ❌ | Cantidad de copias |
| notes | String | ❌ | Notas personales |
| dateAdded | LocalDateTime | ❌ | Cuándo se añadió a la colección |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Enums

**MagicCardCondition:** `MINT`, `NEAR_MINT`, `EXCELLENT`, `GOOD`, `POOR`

**MagicCardLanguage:** `ENGLISH`, `SPANISH`, `FRENCH`, `GERMAN`, `ITALIAN`, `PORTUGUESE`, `JAPANESE`, `CHINESE`

### Reglas de Negocio

- **NO hay POST/PUT genéricos para Magic** — las cartas solo se pueden añadir desde Scryfall
- **MagicCardUseCase solo tiene:** `search`, `findById`, `addFromScryfall`, `delete`
- **externalId es reemplazado por scryfallId + oracleId** para identificación única

---

## Puertos (Interfaces de Dominio)

### MagicCardUseCase (in)
```java
public interface MagicCardUseCase {
    List<MagicCard> search(MagicCardSearchCriteria criteria, Pageable pageable);
    MagicCard findById(String id);
    MagicCard addFromScryfall(String scryfallId, String ownerId);
    void delete(String id, String ownerId);
}
```

### MagicCardSearchUseCase (in)
```java
public interface MagicCardSearchUseCase {
    List<MagicCardSearchResult> searchExternal(String query);
    MagicCardSearchResult findById(String scryfallId);
    List<MagicCardSearchResult> searchCommanders(List<String> colors);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `MagicCardService` | Listado, detalle, eliminación + addFromScryfall |
| `MagicCardSearchService` | Búsqueda externa Scryfall + mapeo a MagicCardSearchResult |

---

## API Externa: Scryfall

- **Base URL:** `https://api.scryfall.com`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo (recomendado)
- **Gratis:** Sí, totalmente gratuita
- **Formato:** JSON
- **Total cartas:** 70,000+
- **Total sets:** 1,049+
- **Estado:** Activa y mantenida (2026)

### Endpoints usados

| Uso | Endpoint |
|-----|----------|
| Buscar por nombre (fuzzy) | `GET /cards/named?fuzzy={name}` |
| Buscar por nombre (exacto) | `GET /cards/named?exact={name}` |
| Búsqueda avanzada | `GET /cards/search?q={query}` |
| Carta por ID | `GET /cards/{id}` |
| Comandantes por colores | `GET /cards/search?q=is:commander+t:{colors}` |

### Mapeo Scryfall → MagicCardSearchResult

| Campo Scryfall | Campo SearchResult | Notas |
|----------------|--------------------|-------|
| `id` | `scryfallId` | ID único de Scryfall |
| `oracle_id` | `oracleId` | ID de oracle |
| `name` | `name` | Nombre de la carta |
| `mana_cost` | `manaCost` | Coste de maná |
| `type` | `type` | Tipo de carta |
| `rarity` | `rarity` | Rareza |
| `set` | `setCode` | Código del set |
| `set_name` | `setName` | Nombre del set |
| `image_uris.normal` | `imageUrl` | URL de imagen |
| `prices.usd` | `priceUsd` | Precio en USD |
| `colors` | `colors` | Colores |
| `color_identity` | `colorIdentity` | Identidad de color |
| `oracle_text` | `text` | Texto/rules |

---

## Controlador REST

**Base:** `/api/v1/magic`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/magic` | Listar cartas con paginación y filtros |
| GET | `/api/v1/magic/{id}` | Obtener carta por ID |
| POST | `/api/v1/magic/scryfall/{scryfallId}` | Añadir carta desde Scryfall (auth requerido) |
| DELETE | `/api/v1/magic/{id}` | Eliminar carta (204 No Content, auth requerido) |
| GET | `/api/v1/magic/search?name={query}` | Buscar en Scryfall |
| GET | `/api/v1/magic/commanders?colors={colors}` | Buscar comandantes por colores |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Buscar por nombre (LIKE case-insensitive) |
| `rarity` | String | Filtrar por rareza |
| `color` | String | Filtrar por color |
| `type` | String | Filtrar por tipo |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |

### Nota Importante

> El backend NO expone POST/PUT genéricos para Magic. Las cartas solo se pueden añadir desde Scryfall con `POST /scryfall/{scryfallId}`.

---

## DTOs

### MagicCardResponse (GET)

```json
{
  "id": "string",
  "scryfallId": "string",
  "oracleId": "string",
  "name": "string",
  "language": "ENGLISH | SPANISH | ...",
  "releaseDate": "string",
  "manaCost": "string",
  "convertedManaCost": "number",
  "type": "string",
  "text": "string",
  "power": "string",
  "toughness": "string",
  "loyalty": "string",
  "colors": ["string"],
  "colorIdentity": ["string"],
  "keywords": ["string"],
  "rarity": "string",
  "setCode": "string",
  "setName": "string",
  "artist": "string",
  "frame": "string",
  "borderColor": "string",
  "layout": "string",
  "legalities": {},
  "priceUsd": "string",
  "priceEur": "string",
  "imageUrl": "string",
  "imageLargeUrl": "string",
  "artCropUrl": "string",
  "condition": "MINT | NEAR_MINT | ...",
  "isFoil": "boolean",
  "quantity": "integer",
  "notes": "string",
  "dateAdded": "datetime",
  "ownerId": "string"
}
```

### MagicCardSearchResponse (resultado de búsqueda externa)

```json
{
  "scryfallId": "string",
  "name": "string",
  "manaCost": "string",
  "type": "string",
  "rarity": "string",
  "setCode": "string",
  "setName": "string",
  "imageUrl": "string",
  "priceUsd": "string",
  "colors": ["string"],
  "colorIdentity": ["string"],
  "text": "string"
}
```

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `MagicCardNotFoundException` | 404 | Carta Magic no encontrada |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `MagicCardServiceTest` | Unitario | Listado/eliminación de cartas Magic |
| `MagicCardControllerTest` | Integración | Endpoints de Magic |
| `MagicCardSearchServiceTest` | Unitario | Búsqueda en Scryfall |
| `ScryfallClientTest` | Unitario | Cliente Scryfall con mock |
| `MagicCardPersistenceAdapterTest` | Integración | Persistencia de Magic |
| `MagicCardMapperTest` | Unitario | Mapeo JSON → MagicCard |
| `MagicCardDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
| `MagicCardServiceOwnerFilterTest` | Unitario | Filtro owner en magic (Fase 9) |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/magic/search?name=lightning+bolt` devuelve resultados de Scryfall
- [x] Los resultados incluyen: nombre, mana cost, tipo, texto, colores, rareza, set, artista, imagen, precios
- [x] El endpoint `GET /api/v1/magic/{id}` devuelve el detalle completo de una carta
- [x] El endpoint `GET /api/v1/magic` devuelve lista vacía al inicio
- [x] Se puede eliminar una carta vía `DELETE /api/v1/magic/{id}` (auth requerido)
- [x] Se puede añadir una carta vía `POST /api/v1/magic/scryfall/{scryfallId}` (auth requerido)
- [x] Se puede buscar por nombre parcial (búsqueda fuzzy)
- [x] Los errores de API externa se manejan correctamente (503, 429, timeout)
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)
- [x] El backend no expone save/update para magic cards — solo search, findById, addFromScryfall, delete

---

## Estado de Implementación

Fase 4 completada. Todos los componentes implementados:
- ✅ Domain: MagicCard.java, MagicCardCondition.java, MagicCardLanguage.java, MagicCardSearchCriteria.java, MagicCardSearchResult.java
- ✅ Ports: MagicCardUseCase.java, MagicCardSearchUseCase.java, MagicCardRepository.java, ExternalMagicCardCatalogClient.java
- ✅ Application: MagicCardService.java, MagicCardSearchService.java
- ✅ Infrastructure: ScryfallClient.java, MagicCardMapper.java, MagicCardController.java, MagicCardEntity.java, MagicCardPersistenceAdapter.java, SpringDataMagicCardRepository.java, MagicCardResponse.java, MagicCardSearchResponse.java, MagicCardDtoMapper.java
- ✅ Config: ScryfallClientConfig.java
- ✅ Tests: 8 archivos de test
