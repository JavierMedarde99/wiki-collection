# Spec: Mazos Commander (Decks)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** Deck (domain/model/Deck.java)

---

## Dominio

### Entidad Deck

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| name | String | ✅ | Nombre del mazo |
| description | String | ❌ | Descripción del mazo |
| commander | String | ❌ | Nombre del comandante |
| commanderColors | List<String> | ❌ | Colores del comandante |
| cards | List<DeckCard> | ❌ | Cartas del mazo (subdocumentos embebidos) |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Entidad DeckCard (subdocumento embebido)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| cardName | String | Nombre de la carta |
| quantity | Integer | Cantidad de copias |
| inCollection | Boolean | Si está en la colección Magic |
| isProxy | Boolean | Si es un proxy |
| manaCost | String | Coste de maná |
| typeLine | String | Línea de tipo |
| colorIdentity | List<String> | Identidad de color |
| imageUrl | String | URL de imagen |
| scryfallId | String | ID en Scryfall |

### Enum

**DeckStatus:** `DRAFT`, `COMPLETE`, `INVALID`

### DeckStatusReport (record de validación)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| status | DeckStatus | Estado del mazo |
| reasons | List<String> | Razones de invalidación (si any) |

### Reglas de Negocio (Commander)

- **DeckValidator** evalúa el estado del mazo:
  - `DRAFT` — mazo en construcción (requisitos mínimos no cumplidos)
  - `COMPLETE` — mazo cumple todas las reglas Commander
  - `INVALID` — mazo incumple reglas (con reasons explicativas)
- **Reglas validadas:** número de cartas (100 + comandante = 101), identidad de color del comandante, etc.
- **AD-014:** Mazos Commander con validación de reglas

---

## Puertos (Interfaces de Dominio)

### DeckUseCase (in)
```java
public interface DeckUseCase {
    Deck save(Deck deck, String ownerId);
    Deck update(String id, Deck updates, String ownerId);
    void delete(String id, String ownerId);
    Deck findById(String id);
    Page<Deck> search(DeckSearchCriteria criteria, Pageable pageable);
    Deck addCard(String deckId, String scryfallId, int quantity, String ownerId);
    void removeCard(String deckId, String scryfallId, String ownerId);
}
```

### DeckSearchUseCase (in)
```java
public interface DeckSearchUseCase {
    List<MagicCardSearchResult> searchCommanders(String colors);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `DeckService` | CRUD de mazos + gestión de cartas (add/remove) |
| `DeckSearchService` | Búsqueda de comandantes en Scryfall |
| `DeckValidator` | Validación de reglas Commander (DRAFT/COMPLETE/INVALID) |

---

## API Externa: Scryfall (Comandantes)

- **Base URL:** `https://api.scryfall.com`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Estado:** Activa

| Uso | Endpoint |
|-----|----------|
| Buscar comandantes por colores | `GET /cards/search?q=is:commander+t:{colors}` |
| Buscar por nombre | `GET /cards/named?fuzzy={name}` |

---

## Controlador REST

**Base:** `/api/v1/decks`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/decks` | Listar mazos (filtro por nombre opcional) |
| GET | `/api/v1/decks/{id}` | Obtener mazo por ID |
| POST | `/api/v1/decks` | Crear mazo (auth requerido) |
| PUT | `/api/v1/decks/{id}` | Actualizar mazo (auth requerido, ownership verificado) |
| DELETE | `/api/v1/decks/{id}` | Eliminar mazo (204 No Content, auth requerido) |
| POST | `/api/v1/decks/{id}/cards` | Añadir carta al mazo desde Scryfall |
| DELETE | `/api/v1/decks/{id}/cards/{scryfallId}` | Quitar carta del mazo |
| GET | `/api/v1/decks/{id}/status` | Estado del mazo (DRAFT/COMPLETE/INVALID) |

### Parámetros de Búsqueda

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `name` | String | Filtrar por nombre exacto |
| `owner` | String | Filtrar por propietario: `mine`, `other`, `all` (Fase 9+) |

---

## DTOs

### DeckRequest (POST/PUT)

```json
{
  "name": "string (obligatorio)",
  "description": "string",
  "commander": "string",
  "commanderColors": ["string"]
}
```

### DeckResponse (GET)

```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "commander": "string",
  "commanderColors": ["string"],
  "cards": [
    {
      "cardName": "string",
      "quantity": "integer",
      "inCollection": "boolean",
      "isProxy": "boolean",
      "manaCost": "string",
      "typeLine": "string",
      "colorIdentity": ["string"],
      "imageUrl": "string",
      "scryfallId": "string"
    }
  ],
  "createdAt": "datetime",
  "updatedAt": "datetime",
  "ownerId": "string"
}
```

### DeckCardRequest (POST /{id}/cards)

```json
{
  "scryfallId": "string (obligatorio)",
  "quantity": "integer (min 1)"
}
```

### DeckStatusResponse (GET /{id}/status)

```json
{
  "status": "DRAFT | COMPLETE | INVALID",
  "message": "string"
}
```

---

## Excepciones

| Excepción | HTTP | Descripción |
|-----------|------|-------------|
| `DeckNotFoundException` | 404 | Mazo no encontrado |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `DeckServiceTest` | Unitario | CRUD de mazos |
| `DeckControllerTest` | Integración | Endpoints de mazos |
| `DeckSearchServiceTest` | Unitario | Búsqueda de comandantes |
| `DeckValidationTest` | Unitario | Validación de mazos |
| `DeckPersistenceAdapterTest` | Integración | Persistencia de mazos |
| `DeckDtoMapperTest` | Unitario | Mapeo DTO ↔ Domain |
| `DeckModelTest` | Unitario | Modelo de dominio Deck |
| `DeckServiceOwnerFilterTest` | Unitario | Filtro owner en decks (Fase 9) |

---

## Criterios de Aceptación

- [x] El endpoint `GET /api/v1/decks` devuelve lista de mazos
- [x] El endpoint `GET /api/v1/decks/{id}` devuelve el mazo completo con cartas
- [x] Se puede crear un mazo vía `POST /api/v1/decks` (auth requerido)
- [x] Se puede actualizar/eliminar un mazo vía `PUT`/`DELETE /api/v1/decks/{id}` (auth + ownership)
- [x] Se puede añadir carta al mazo vía `POST /api/v1/decks/{id}/cards` (auth requerido)
- [x] Se puede quitar carta vía `DELETE /api/v1/decks/{id}/cards/{scryfallId}` (auth requerido)
- [x] El endpoint `GET /api/v1/decks/{id}/status` devuelve DRAFT/COMPLETE/INVALID con razones
- [x] El DeckValidator evalúa las reglas Commander correctamente
- [x] Los tests pasan (`mvn verify`)
- [x] Se respeta la arquitectura hexagonal (dependencias hacia dentro)

---

## Estado de Implementación

Fase 4.1 completada. Todos los componentes implementados:
- ✅ Domain: Deck.java, DeckCard.java, DeckStatus.java, DeckStatusReport.java
- ✅ Ports: DeckUseCase.java, DeckSearchUseCase.java, DeckRepository.java
- ✅ Application: DeckService.java, DeckSearchService.java, DeckValidator.java
- ✅ Infrastructure: DeckController.java, DeckEntity.java, DeckCardEntity.java, DeckPersistenceAdapter.java, SpringDataDeckRepository.java, DeckDtoMapper.java, DeckResponse.java, DeckCardResponse.java, DeckStatusResponse.java
- ✅ Tests: 8 archivos de test
