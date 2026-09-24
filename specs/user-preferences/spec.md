# Spec: Colecciones Personales y Visibilidad (User Preferences)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** UserPreferences (domain/model/UserPreferences.java)

---

## Dominio

### Entidad UserPreferences

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único |
| userId | String | ✅ | ID del usuario (unique, indexed) |
| activeCollections | Map<String, Boolean> | ✅ | Mapa de colecciones activadas (books, games, boardgames, magic, decks, movieshows) |
| collectionVisibility | Map<String, String> | ✅ | Visibilidad por colección: "PUBLIC" o "PRIVATE" |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |

### Enums

**CollectionType:** `BOOKS("books")`, `GAMES("games")`, `BOARDGAMES("boardgames")`, `MAGIC("magic")`, `DECKS("decks")`, `MOVIE_SHOWS("movieshows")`

**CollectionVisibility:** `PUBLIC`, `PRIVATE`

### Esquema de activeCollections

```json
{
  "books": true,
  "games": true,
  "boardgames": false,
  "magic": true,
  "decks": true,
  "movieshows": false
}
```

### Esquema de collectionVisibility

```json
{
  "books": "PUBLIC",
  "games": "PRIVATE",
  "boardgames": "PUBLIC",
  "magic": "PUBLIC",
  "decks": "PUBLIC",
  "movieshows": "PUBLIC"
}
```

### Configuración por Defecto (al registrarse)

| Colección | Activa | Visibilidad |
|-----------|--------|-------------|
| books | true | PUBLIC |
| games | true | PUBLIC |
| boardgames | true | PUBLIC |
| magic | true | PUBLIC |
| decks | true | PUBLIC |
| movieshows | true | PUBLIC |

> Todas las colecciones activadas y públicas por defecto.

---

## Puertos (Interfaces de Dominio)

### UserPreferencesUseCase (in)
```java
public interface UserPreferencesUseCase {
    UserPreferences getPreferences(String userId);
    UserPreferences updatePreferences(String userId, UserPreferencesRequest request);
    UserPreferences setActiveCollections(String userId, Map<String, Boolean> collections);
    UserPreferences setCollectionVisibility(String userId, Map<String, CollectionVisibility> visibility);
    List<CollectionType> getActiveCollections(String userId);
    boolean isCollectionActive(String userId, CollectionType type);
    boolean isCollectionPublic(String userId, CollectionType type);
}
```

### UserProfileUseCase (in)
```java
public interface UserProfileUseCase {
    PublicProfileResponse getPublicProfile(String username);
    Page<Book> getPublicBooks(String username, Pageable pageable);
    Page<Game> getPublicGames(String username, Pageable pageable);
    Page<BoardGame> getPublicBoardGames(String username, Pageable pageable);
    Page<MagicCard> getPublicMagicCards(String username, Pageable pageable);
    Page<Deck> getPublicDecks(String username, Pageable pageable);
    Page<MovieShow> getPublicMovieShows(String username, Pageable pageable);
}
```

### UserPreferencesRepository (out)
```java
public interface UserPreferencesRepository {
    Optional<UserPreferences> findByUserId(String userId);
    UserPreferences save(UserPreferences preferences);
}
```

### UserProfilePort (out)
```java
public interface UserProfilePort {
    PublicProfileResponse getPublicProfile(String username);
    Page<Book> getPublicBooks(String username, Pageable pageable);
    Page<Game> getPublicGames(String username, Pageable pageable);
    Page<BoardGame> getPublicBoardGames(String username, Pageable pageable);
    Page<MagicCard> getPublicMagicCards(String username, Pageable pageable);
    Page<Deck> getPublicDecks(String username, Pageable pageable);
    Page<MovieShow> getPublicMovieShows(String username, Pageable pageable);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `UserPreferencesService` | CRUD de preferencias de colección (activas + visibilidad) |
| `UserProfileService` | Perfiles públicos y listados de colecciones públicas |
| `OwnerResolver` | Resuelve ownerId del usuario autenticado |
| `OwnerScopeResolver` | Resuelve scope de visibilidad (mine/other/all) |
| `OwnershipValidator` | Valida ownership para operaciones de escritura |

---

## Controlador REST

### UserPreferencesController

**Base:** `/api/v1/preferences`

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/v1/preferences` | Obtener preferencias del usuario | ✅ |
| PUT | `/api/v1/preferences` | Actualizar preferencias completas | ✅ |
| PATCH | `/api/v1/preferences/active-collections` | Activar/desactivar colecciones | ✅ |
| PATCH | `/api/v1/preferences/collection-visibility` | Cambiar visibilidad de colecciones | ✅ |
| GET | `/api/v1/preferences/active-collections` | Obtener solo colecciones activas | ✅ |

### UserProfileController

**Base:** `/api/v1/users`

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/v1/users/{username}` | Perfil público de un usuario | Público |
| GET | `/api/v1/users/{username}/books` | Libros públicos de un usuario | Público |
| GET | `/api/v1/users/{username}/games` | Juegos públicos de un usuario | Público |
| GET | `/api/v1/users/{username}/boardgames` | Juegos de mesa públicos | Público |
| GET | `/api/v1/users/{username}/magic` | Cartas Magic públicas | Público |
| GET | `/api/v1/users/{username}/decks` | Mazos públicos | Público |
| GET | `/api/v1/users/{username}/movieshows` | Películas/series públicas | Público |

### Listados con Visibilidad (Fase 9+)

Todos los endpoints GET de colecciones aceptan parámetro `owner`:

| Parámetro | Descripción | Auth requerido |
|-----------|-------------|----------------|
| `owner=mine` (default) | Solo los del usuario autenticado | Sí |
| `owner=other` | Solo de otros usuarios con colección pública | Sí |
| `owner=all` | Todos (propios + públicos de otros) | Sí |

Ejemplo:
```
GET /api/v1/books?owner=other      → libros de otros (públicos)
GET /api/v1/books?owner=mine       → libros del usuario actual
GET /api/v1/books?owner=all        → todos (propios + públicos de otros)
```

---

## DTOs

### UserPreferencesResponse

```json
{
  "id": "string",
  "userId": "string",
  "activeCollections": {
    "books": true,
    "games": true,
    "boardgames": false,
    "magic": true,
    "decks": true,
    "movieshows": false
  },
  "collectionVisibility": {
    "books": "PUBLIC",
    "games": "PUBLIC",
    "boardgames": "PUBLIC",
    "magic": "PUBLIC",
    "decks": "PUBLIC",
    "movieshows": "PUBLIC"
  }
}
```

### UserPreferencesRequest

```json
{
  "activeCollections": {
    "books": true,
    "games": false,
    "boardgames": true,
    "magic": true,
    "decks": true,
    "movieshows": false
  },
  "collectionVisibility": {
    "books": "PUBLIC",
    "games": "PRIVATE",
    "boardgames": "PUBLIC",
    "magic": "PUBLIC",
    "decks": "PUBLIC",
    "movieshows": "PUBLIC"
  }
}
```

### ActiveCollectionsRequest

```json
{
  "collections": {
    "books": true,
    "games": false,
    "boardgames": true,
    "magic": true,
    "decks": true,
    "movieshows": false
  }
}
```

### CollectionVisibilityRequest

```json
{
  "visibility": {
    "books": "PUBLIC",
    "games": "PRIVATE",
    "boardgames": "PUBLIC",
    "magic": "PUBLIC",
    "decks": "PUBLIC",
    "movieshows": "PUBLIC"
  }
}
```

### PublicProfileResponse

```json
{
  "username": "string",
  "displayName": "string",
  "avatarUrl": "string",
  "bio": "string",
  "publicCollections": [
    {
      "type": "books",
      "count": 25,
      "visibility": "PUBLIC"
    }
  ]
}
```

### PublicCollectionSummary

```json
{
  "type": "books",
  "count": 25,
  "visibility": "PUBLIC"
}
```

---

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | Preferencias obtenidas/actualizadas |
| 200 | Perfil obtenido |
| 400 | Datos inválidos |
| 401 | No autenticado |
| 404 | Preferencias no encontradas / Usuario no encontrado |

---

## Visibilidad de Elementos en Detalle

| Situación | Visibilidad |
|-----------|-------------|
| Usuario es el dueño | Ve todo (incluyendo notas, valoraciones) |
| Colección es pública | Ve los metadatos pero NO notas/valoraciones |
| Colección es privada y no es el dueño | 403 Forbidden ("Esta colección es privada") |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `UserPreferencesServiceTest` | Unitario | CRUD de preferencias (Fase 9) |
| `UserPreferencesControllerTest` | Integración | Endpoints de preferencias (Fase 9) |
| `UserProfileServiceTest` | Unitario | Perfil público (Fase 9) |
| `UserProfileControllerTest` | Integración | Endpoints de perfil público (Fase 9) |
| `BookServiceOwnerFilterTest` | Unitario | Filtro owner=mine/other/all (Fase 9) |
| `GameServiceOwnerFilterTest` | Unitario | Filtro owner en juegos (Fase 9) |
| `BoardGameServiceOwnerFilterTest` | Unitario | Filtro owner en board games (Fase 9) |
| `MagicCardServiceOwnerFilterTest` | Unitario | Filtro owner en magic (Fase 9) |
| `DeckServiceOwnerFilterTest` | Unitario | Filtro owner en decks (Fase 9) |
| `MovieShowServiceOwnerFilterTest` | Unitario | Filtro owner en movieshows (Fase 9) |
| `AuthServiceCreatePreferencesTest` | Unitario | Crear preferencias al registrar (Fase 9) |
| `UserPreferencesMigrationTest` | Integración | Migración de preferencias por defecto |
| `UserOwnedBackfillMigrationTest` | Integración | Poblar user_owned con datos existentes |
| `OwnershipValidatorTest` | Unitario | Validación de ownership |
| `OwnerResolverTest` | Unitario | Resolución de ownerId |
| `OwnerScopeResolverTest` | Unitario | Resolución de scope de visibilidad |

---

## Frontend

### Nuevos Componentes

| Componente | Responsabilidad |
|------------|-----------------|
| `CollectionPreferencesPanel` | Toggle de colecciones activas + visibilidad |
| `CollectionVisibilityBadge` | Badge que indica público/private |
| `UserProfileHeader` | Cabecera del perfil público |
| `OtherCollectionsSection` | Sección "Otras colecciones de..." |

### Nuevas Páginas

| Página | Ruta | Auth | Descripción |
|--------|------|------|-------------|
| `PreferencesPage` | `/preferencias` | Protegido | Panel de preferencias de colección |
| `PublicProfilePage` | `/perfil/:username` | Público | Perfil público de otro usuario |

### Nuevas Rutas

```
/preferencias        → ProtectedRoute → PreferencesPage
/perfil/:username    → PublicProfilePage
```

### Modificaciones a Componentes Existentes

- **Navbar:** Mostrar solo las colecciones activas del usuario autenticado
- **HomePage:** Mostrar solo contadores de colecciones activas
- **ListPages:** Pestañas "Mi Colección" / "Otras Colecciones"
- **AuthContext:** Añadir `activeCollections`, `preferences`, `refreshActiveCollections()`, `refreshPreferences()`

### Modificaciones a Hooks Existentes

| Hook | Cambio |
|------|--------|
| `useCollectionPreferences` | Carga y guardado de preferencias de colección (active + visibility) |
| `useAuth` (AuthContext) | Añadir activeCollections + preferences |

---

## Criterios de Aceptación

- [x] Un usuario puede activar/desactivar colecciones desde preferencias
- [x] Un usuario puede marcar cada colección como pública o privada
- [x] Los endpoints GET aceptan parámetro `owner` (mine/other/all)
- [x] Al listar con `owner=other` solo se muestran colecciones públicas
- [x] El perfil público muestra solo colecciones públicas activadas
- [x] El navbar muestra solo las colecciones activas del usuario
- [x] El Home muestra solo contadores de colecciones activas
- [x] Las páginas de listado muestran pestañas "Mi colección / Otras colecciones"
- [x] Un usuario sin auth puede ver perfiles públicos de otros
- [x] Un usuario no puede ver colecciones privadas de otros (403)
- [x] Las preferencias se crean por defecto al registrarse
- [x] Usuarios existentes obtienen preferencias por defecto al arrancar
- [x] Los tests pasan (`mvn verify` y `npm test`)
- [x] JaCoCo mantiene ≥80% cobertura

---

## Estado de Implementación

Fase 9 completada. Todos los componentes implementados:
- ✅ Domain: UserPreferences.java, CollectionVisibility.java, CollectionType.java
- ✅ Ports: UserPreferencesUseCase.java, UserProfileUseCase.java, UserPreferencesRepository.java, UserProfilePort.java
- ✅ Application: UserPreferencesService.java, UserProfileService.java, OwnerResolver.java, OwnerScopeResolver.java, OwnershipValidator.java, UserPreferencesMigration.java, UserOwnedBackfillMigration.java
- ✅ Infrastructure: UserPreferencesController.java, UserProfileController.java, UserPreferencesEntity.java, UserPreferencesEntityMapper.java, UserPreferencesPersistenceAdapter.java, SpringDataUserPreferencesRepository.java, UserOwnedEntity.java, UserProfilePersistenceAdapter.java, UserPreferencesRequest.java, UserPreferencesResponse.java, ActiveCollectionsRequest.java, CollectionVisibilityRequest.java, PublicProfileResponse.java, PublicCollectionSummary.java
- ✅ Tests: 17 archivos de test backend (incluye OwnerFilter tests) + 8 archivos de test frontend
