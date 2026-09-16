# Fase 9: Colecciones Personales y Visibilidad

## Objetivo

Implementar la capacidad de que cada usuario configure qué colecciones quiere utilizar realmente y controlar la visibilidad de sus colecciones (públicas o privadas). Además, añadir páginas de perfil público para explorar las colecciones de otros usuarios.

---

## Resumen de Funcionalidad

### 1. Preferencias de Colección por Usuario

Cada usuario podrá activar/desactivar las colecciones que le interesan (Libros, Juegos, Board Games, Magic, Películas/Series, Mazos). Esto significa que:

- Si un usuario no usa la colección de Magic, no verá ni la sección en el navbar, ni la página de listado, ni los filtros relacionados.
- Los contadores en el Home solo reflejarán las colecciones activadas.

### 2. Visibilidad de Colecciones

Cada usuario podrá configurar individualmente la visibilidad de cada colección:

- **Pública** — otros usuarios (y anónimos) pueden ver sus elementos.
- **Privada** — solo el propietario puede ver los elementos.

### 3. Exploración de Colecciones de Otros Usuarios

Cada colección tendrá dos secciones:

- **Mi colección de [X]** — lista los elementos del usuario autenticado.
- **Otras colecciones de [X]** — lista elementos de otros usuarios que tienen esa colección **pública**.

### 4. Perfil Público

Cada usuario tendrá un perfil público en `/perfil/{username}` donde se muestran:
- Información del usuario (displayName, avatar, bio).
- Colecciones públicas activadas con sus contadores.
- Posibilidad de explorar sus elementos públicos.

---

## Modelo de Datos

### Colección: `user_preferences` (NUEVA)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único |
| userId | String | ✅ | ID del usuario (unique, indexed) |
| activeCollections | Map<String, Boolean> | ✅ | Mapa de colecciones activadas |
| collectionVisibility | Map<String, String> | ✅ | Visibilidad por colección (PUBLIC / PRIVATE) |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |

### Esquema de `activeCollections`

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

### Esquema de `collectionVisibility`

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

### Enum: `CollectionVisibility`

```java
public enum CollectionVisibility {
    PUBLIC,
    PRIVATE
}
```

### Enum: `CollectionType`

```java
public enum CollectionType {
    BOOKS("books"),
    GAMES("games"),
    BOARDGAMES("boardgames"),
    MAGIC("magic"),
    DECKS("decks"),
    MOVIESHOWS("movieshows");

    private final String fieldName;

    // Constructor, getter
}
```

---

## Configuración por Defecto

Cuando un usuario se registra, se crean preferencias por defecto:

| Colección | Activa | Visibilidad |
|-----------|--------|-------------|
| books | true | PUBLIC |
| games | true | PUBLIC |
| boardgames | true | PUBLIC |
| magic | true | PUBLIC |
| decks | true | PUBLIC |
| movieshows | true | PUBLIC |

Todas las colecciones activadas y públicas por defecto.

---

## API Endpoints

### Preferencias de Usuario

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/v1/preferences` | Obtener preferencias del usuario | ✅ |
| PUT | `/api/v1/preferences` | Actualizar preferencias completas | ✅ |
| PATCH | `/api/v1/preferences/active-collections` | Activar/desactivar colecciones | ✅ |
| PATCH | `/api/v1/preferences/collection-visibility` | Cambiar visibilidad de colecciones | ✅ |
| GET | `/api/v1/preferences/active-collections` | Obtener solo colecciones activas | ✅ |

### Perfiles Públicos

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/api/v1/users/{username}` | Perfil público de un usuario | Público |
| GET | `/api/v1/users/{username}/books` | Libros públicos de un usuario | Público |
| GET | `/api/v1/users/{username}/games` | Juegos públicos de un usuario | Público |
| GET | `/api/v1/users/{username}/boardgames` | Juegos de mesa públicos | Público |
| GET | `/api/v1/users/{username}/magic` | Cartas Magic públicas | Público |
| GET | `/api/v1/users/{username}/decks` | Mazos públicos | Público |
| GET | `/api/v1/users/{username}/movieshows` | Películas/series públicas | Público |

### Listados con Visibilidad

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/books?owner=other` | Ver libros de otros usuarios (públicos) |
| GET | `/api/v1/games?owner=other` | Ver juegos de otros usuarios (públicos) |
| ... similar para el resto | |

### Nuevos Parámetros en Endpoints Existentes

Todos los endpoints GET de listado aceptarán:

- `?owner=mine` (default) — solo los del usuario autenticado
- `?owner=other` — solo de otros usuarios con colección pública
- `?owner=all` — todos (requiere auth, combina propios + públicos)

Ejemplo:

```
GET /api/v1/books?owner=other      → libros de otros (públicos)
GET /api/v1/books?owner=mine       → libros del usuario actual
GET /api/v1/books?owner=all        → todos (propios + públicos de otros)
```

---

## Arquitectura Backend

### Nuevas Clases

```
com.wikicollection/
├── domain/
│   ├── model/
│   │   ├── UserPreferences.java          ← NUEVO
│   │   ├── CollectionVisibility.java     ← NUEVO (enum)
│   │   └── CollectionType.java           ← NUEVO (enum)
│   └── port/
│       ├── in/
│       │   ├── UserPreferencesUseCase.java  ← NUEVO
│       │   └── UserProfileUseCase.java      ← NUEVO
│       └── out/
│           ├── UserPreferencesRepository.java  ← NUEVO
│           └── UserProfilePort.java            ← NUEVO
├── application/
│   └── service/
│       ├── UserPreferencesService.java   ← NUEVO
│       └── UserProfileService.java       ← NUEVO
└── infrastructure/
    ├── adapter/
    │   ├── in/web/
    │   │   ├── UserPreferencesController.java  ← NUEVO
    │   │   ├── UserProfileController.java      ← NUEVO
    │   │   └── dto/
    │   │       ├── UserPreferencesRequest.java  ← NUEVO
    │   │       ├── ActiveCollectionsRequest.java ← NUEVO
    │   │       ├── CollectionVisibilityRequest.java ← NUEVO
    │   │       ├── UserPreferencesResponse.java ← NUEVO
    │   │       ├── PublicProfileResponse.java   ← NUEVO
    │   │       └── PublicCollectionSummary.java ← NUEVO
    │   └── out/
    │       └── persistence/
    │           ├── UserPreferencesEntity.java   ← NUEVO
    │           ├── UserPreferencesMapper.java  ← NUEVO
    │           ├── UserPreferencesPersistenceAdapter.java ← NUEVO
    │           └── SpringDataUserPreferencesRepository.java ← NUEVO
```

### Modificaciones en Clases Existentes

1. **BookController, GameController, etc.** — Añadir parámetro `owner` (mine/other/all) y filtrar según visibilidad.
2. **HomeController / StatsService** — Filtrar contadores por colecciones activas.
3. **AuthService** — Crear preferencias por defecto al registrar usuario.

---

## Implementación Paso a Paso

### 1. Dependencias

No se necesitan dependencias nuevas. Solo Java + Spring Boot existente.

---

### 2. Entidades de Dominio

```java
@Document(collection = "user_preferences")
public class UserPreferences {
    @Id
    private String id;

    @Indexed(unique = true)
    private String userId;

    private Map<String, Boolean> activeCollections;
    private Map<String, CollectionVisibility> collectionVisibility;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    // Getters, setters, builder
}
```

---

### 3. UserPreferencesUseCase

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

---

### 4. UserProfileUseCase

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

---

### 5. UserPreferencesController

```java
@RestController
@RequestMapping("/api/v1/preferences")
public class UserPreferencesController {

    @Autowired
    private UserPreferencesUseCase preferencesUseCase;

    @GetMapping
    public ResponseEntity<UserPreferencesResponse> getPreferences(@CurrentUser String userId) {
        var prefs = preferencesUseCase.getPreferences(userId);
        return ResponseEntity.ok(mapper.toResponse(prefs));
    }

    @PutMapping
    public ResponseEntity<UserPreferencesResponse> updatePreferences(
            @CurrentUser String userId,
            @Valid @RequestBody UserPreferencesRequest request) {
        var updated = preferencesUseCase.updatePreferences(userId, request);
        return ResponseEntity.ok(mapper.toResponse(updated));
    }

    @PatchMapping("/active-collections")
    public ResponseEntity<UserPreferencesResponse> setActiveCollections(
            @CurrentUser String userId,
            @Valid @RequestBody ActiveCollectionsRequest request) {
        var updated = preferencesUseCase.setActiveCollections(userId, request.collections());
        return ResponseEntity.ok(mapper.toResponse(updated));
    }

    @PatchMapping("/collection-visibility")
    public ResponseEntity<UserPreferencesResponse> setCollectionVisibility(
            @CurrentUser String userId,
            @Valid @RequestBody CollectionVisibilityRequest request) {
        var updated = preferencesUseCase.setCollectionVisibility(userId, request.visibility());
        return ResponseEntity.ok(mapper.toResponse(updated));
    }

    @GetMapping("/active-collections")
    public ResponseEntity<List<String>> getActiveCollections(@CurrentUser String userId) {
        var active = preferencesUseCase.getActiveCollections(userId);
        return ResponseEntity.ok(active.stream().map(CollectionType::getFieldName).toList());
    }
}
```

---

### 6. UserProfileController

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserProfileController {

    @Autowired
    private UserProfileUseCase profileUseCase;

    @GetMapping("/{username}")
    public ResponseEntity<PublicProfileResponse> getPublicProfile(@PathVariable String username) {
        return ResponseEntity.ok(profileUseCase.getPublicProfile(username));
    }

    @GetMapping("/{username}/books")
    public ResponseEntity<Page<BookResponse>> getPublicBooks(
            @PathVariable String username,
            Pageable pageable) {
        return ResponseEntity.ok(profileUseCase.getPublicBooks(username, pageable));
    }

    @GetMapping("/{username}/games")
    public ResponseEntity<Page<GameResponse>> getPublicGames(
            @PathVariable String username,
            Pageable pageable) {
        return ResponseEntity.ok(profileUseCase.getPublicGames(username, pageable));
    }

    // ... similar para boardgames, magic, decks, movieshows
}
```

---

### 7. Modificación de Endpoints GET Existentes

Ejemplo para BookController:

```java
@GetMapping
public ResponseEntity<Page<BookResponse>> listBooks(
        @RequestParam(defaultValue = "mine") String owner,
        @CurrentUser String currentUserId,
        BookSearchCriteria criteria,
        Pageable pageable) {

    Page<Book> books;

    switch (owner.toLowerCase()) {
        case "mine":
            if (currentUserId == null) {
                throw new UnauthorizedError();
            }
            criteria.setOwnerId(currentUserId);
            books = bookService.search(criteria, pageable);
            break;
        case "other":
            criteria.setExcludeOwnerId(currentUserId);
            criteria.setPublicOnly(true);  // Solo colecciones públicas
            books = bookService.search(criteria, pageable);
            break;
        case "all":
            if (currentUserId == null) {
                throw new UnauthorizedError();
            }
            criteria.setPublicOrOwnerId(currentUserId);
            books = bookService.search(criteria, pageable);
            break;
        default:
            throw new BadRequestException("Parámetro owner inválido");
    }

    return ResponseEntity.ok(books.map(bookMapper::toResponse));
}
```

---

### 8. AuthService: Crear Preferencias por Defecto

Modificar el método `register`:

```java
public AuthResponse register(RegisterRequest request) {
    // ... código existente de creación de user

    userRepository.save(user);

    // Crear preferencias por defecto
    userPreferencesService.createDefaultPreferences(user.getId());

    // ... generar tokens
}
```

---

### 9. Filtros de Visibilidad en Servicios

Modificar cada servicio de búsqueda (BookService, GameService, etc.) para soportar:

- Filtrar por `ownerId` cuando `owner=mine`.
- Excluir al usuario actual y filtrar solo públicos cuando `owner=other`.
- Combinar ambos cuando `owner=all`.

Necesitamos que los servicios consulten UserPreferences para determinar si la colección de otro usuario es pública.

```java
// Ejemplo en BookService.search()
if (criteria.getPublicOnly() != null && criteria.getPublicOnly()) {
    // Obtener usuarios que tienen Books pública
    List<String> publicBookUsers = preferencesUseCase
        .getUsersWithPublicCollection(CollectionType.BOOKS);
    criteria.setOwnerIds(publicBookUsers);
    criteria.setExcludeOwnerId(currentUserId);
}
```

---

## Implementación Frontend

### Nuevos Componentes

#### CollectionPreferencesPanel
Panel de configuración de preferencias de colección.

**Props:** Ninguno (usa AuthContext + API)

**Funcionalidad:**
- Toggle por cada colección para activar/desactivar.
- Toggle por cada colección para visibilidad (Pública/Privada).
- Botón "Guardar" que hace PATCH a la API.

#### CollectionVisibilityBadge
Badge que indica si una colección es pública o privada.

**Props:**
- `visibility` — "PUBLIC" | "PRIVATE"

#### UserProfileHeader
Cabecera del perfil público de un usuario.

**Props:**
- `user` — `PublicProfileResponse`

#### OtherCollectionsSection
Sección "Otras colecciones de..." que muestra elementos de otros usuarios.

**Props:**
- `collectionType` — `CollectionType`
- `elements` — Lista de elementos públicos de otros

---

### Nuevas Páginas

#### PreferencesPage
Página de configuración de preferencias.

**Ruta:** `/preferencias`

**Layout:**
```
┌─────────────────────────────────────────────────┐
│  Preferencias de Colección                       │
├─────────────────────────────────────────────────┤
│                                                  │
│  [✓] Libros              [Público ▼]            │
│  [✓] Juegos              [Público ▼]            │
│  [✓] Board Games         [Privado ▼]            │
│  [ ] Magic               [Público ▼]            │
│  [✓] Mazos               [Público ▼]            │
│  [✓] Películas/Series    [Público ▼]            │
│                                                  │
│              [ Guardar Cambios ]                 │
└─────────────────────────────────────────────────┘
```

#### PublicProfilePage
Página de perfil público de un usuario.

**Ruta:** `/perfil/{username}`

**Layout:**
```
┌─────────────────────────────────────────────────┐
│  ┌──────┐  Username                              │
│  │ Foto │  Display Name                          │
│  └──────┘  Bio...                                │
├─────────────────────────────────────────────────┤
│  Colecciones Públicas:                           │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐            │
│  │  📚     │ │  🎮     │ │  🎲     │            │
│  │ 25 Libros│ │ 12 Juegos│ │ 8 Board │            │
│  └─────────┘ └─────────┘ └─────────┘            │
├─────────────────────────────────────────────────┤
│  [ Ver Libros ] [ Ver Juegos ] [ Ver Board ]     │
└─────────────────────────────────────────────────┘
```

---

### Nuevas Rutas

```typescript
<Route path="/preferencias" element={
  <ProtectedRoute>
    <PreferencesPage />
  </ProtectedRoute>
} />

<Route path="/perfil/:username" element={
  <PublicProfilePage />
} />
```

---

### Modificaciones en Componentes Existentes

1. **Navbar** — Mostrar solo las colecciones activas del usuario autenticado (obtenidas de las preferencias).
2. **HomePage** — Mostrar solo contadores de colecciones activas.
3. **BookListPage, GameListPage, etc.** — Añadir pestañas o sección "Mi colección" y "Otras colecciones".
4. **ProtectedRoute** — Si una colección está desactivada, redirigir al Home en lugar de mostrar lista vacía.

---

### Modificaciones en AuthContext

```typescript
interface AuthContextType {
  user: UserResponse | null;
  accessToken: string | null;
  login: (username: string, password: string) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
  // NUEVOS
  activeCollections: string[];         // ["books", "games", ...]
  refreshActiveCollections: () => Promise<void>;
  preferences: UserPreferences | null;
  refreshPreferences: () => Promise<void>;
}
```

---

### Nuevo Hook: useCollectionPreferences

```typescript
// src/hooks/useCollectionPreferences.ts
export const useCollectionPreferences = () => {
  const { preferences, refreshPreferences } = useAuth();

  const isCollectionActive = (type: string): boolean => {
    return preferences?.activeCollections[type] ?? true;
  };

  const isCollectionPublic = (type: string): boolean => {
    return preferences?.collectionVisibility[type] === "PUBLIC";
  };

  return { preferences, refreshPreferences, isCollectionActive, isCollectionPublic };
};
```

---

## Frontend: UX de "Mi Colección / Otras Colecciones"

Cada página de listado tendrá dos modos:

### Pestañas

```
┌─────────────────────────────────────────────────┐
│  Libros                                          │
├─────────────────────────────────────────────────┤
│  [ Mi Colección ]  [ Otras Colecciones ]         │
├─────────────────────────────────────────────────┤
│  ... contenido según pestaña activa              │
└─────────────────────────────────────────────────┘
```

### Llamadas API

- **Mi Colección:** `GET /api/v1/books?owner=mine`
- **Otras Colecciones:** `GET /api/v1/books?owner=other`

---

## Visibilidad de Elementos en Detalle

### Regla de negocio

Al ver el detalle de un elemento (`GET /api/v1/books/{id}`):

- Si el usuario es el **dueño** → ve todo (incluyendo notas, valoraciones).
- Si la colección es **pública** → ve los metadatos pero NO notas/valoraciones.
- Si la colección es **privada** y no es el dueño → 403 Forbidden.

---

## Manejo de Errores

| Situación | HTTP | Mensaje |
|-----------|------|---------|
| Colección privada de otro usuario | 403 | "Esta colección es privada" |
| Usuario no tiene colección activa | 404 | "Colección no disponible" |
| Usuario no encontrado (perfil) | 404 | "Usuario no encontrado" |
| Preferencias no encontradas | 404 | "Preferencias no configuradas" |

---

## Tests

### Backend Tests Nuevos

| Test | Tipo | Descripción |
|------|------|-------------|
| UserPreferencesServiceTest | Unitario | CRUD de preferencias |
| UserPreferencesControllerTest | Integración | Endpoints de preferencias |
| UserProfileServiceTest | Unitario | Perfil público |
| UserProfileControllerTest | Integración | Endpoints de perfil público |
| BookServiceOwnerFilterTest | Unitario | Filtro owner=mine/other/all |
| GameServiceOwnerFilterTest | Integración | Filtro owner en juegos |
| BoardGameServiceOwnerFilterTest | Integración | Filtro owner en board games |
| MagicCardServiceOwnerFilterTest | Integración | Filtro owner en magic |
| DeckServiceOwnerFilterTest | Integración | Filtro owner en decks |
| MovieShowServiceOwnerFilterTest | Integración | Filtro owner en movieshows |
| AuthServiceCreatePreferencesTest | Unitario | Crear preferencias al registrar |

**Total backend: 11 archivos de test nuevos**

### Frontend Tests Nuevos

| Test | Tipo | Descripción |
|------|------|-------------|
| PreferencesPage.test.tsx | Unitario | Página de preferencias |
| CollectionPreferencesPanel.test.tsx | Unitario | Panel de preferencias |
| PublicProfilePage.test.tsx | Unitario | Página de perfil público |
| useCollectionPreferences.test.ts | Unitario | Hook de preferencias |
| NavbarActiveCollections.test.tsx | Unitario | Navbar filtra por activas |

**Total frontend: 5 archivos de test nuevos**

---

## Migración de Datos

1. **Usuarios existentes:** Al arrancar, si no tienen preferencias, se crean con valores por defecto (todas activas, todas públicas).
2. **Datos existentes (`ownerId = "system"`):** Se mantienen como están. Se consideran públicos por ser del sistema.

---

## Estructura de Paquetes Final

```
com.wikicollection/
├── domain/
│   ├── model/
│   │   ├── UserPreferences.java              ← NUEVO
│   │   ├── CollectionVisibility.java         ← NUEVO
│   │   └── CollectionType.java               ← NUEVO
│   └── port/
│       ├── in/
│       │   ├── UserPreferencesUseCase.java    ← NUEVO
│       │   └── UserProfileUseCase.java        ← NUEVO
│       └── out/
│           ├── UserPreferencesRepository.java ← NUEVO
│           └── UserProfilePort.java           ← NUEVO
├── application/
│   └── service/
│       ├── UserPreferencesService.java       ← NUEVO
│       └── UserProfileService.java           ← NUEVO
└── infrastructure/
    ├── adapter/
    │   ├── in/web/
    │   │   ├── UserPreferencesController.java ← NUEVO
    │   │   ├── UserProfileController.java     ← NUEVO
    │   │   └── dto/
    │   │       ├── UserPreferencesRequest.java ← NUEVO
    │   │       ├── ActiveCollectionsRequest.java ← NUEVO
    │   │       ├── CollectionVisibilityRequest.java ← NUEVO
    │   │       ├── UserPreferencesResponse.java ← NUEVO
    │   │       ├── PublicProfileResponse.java ← NUEVO
    │   │       └── PublicCollectionSummary.java ← NUEVO
    │   └── out/
    │       └── persistence/
    │           ├── UserPreferencesEntity.java  ← NUEVO
    │           ├── UserPreferencesMapper.java  ← NUEVO
    │           ├── UserPreferencesPersistenceAdapter.java ← NUEVO
    │           └── SpringDataUserPreferencesRepository.java ← NUEVO
    └── config/
        └── CollectionTypeConfig.java           ← NUEVO (bean names)
```

---

## Estado: 📋 PLANIFICADA

- [ ] Crear entidades de dominio (UserPreferences, CollectionVisibility, CollectionType)
- [ ] Crear UserPreferencesEntity + repositorio MongoDB
- [ ] Implementar UserPreferencesService
- [ ] Implementar UserProfileService
- [ ] Crear UserPreferencesController + DTOs
- [ ] Crear UserProfileController + DTOs
- [ ] Modificar AuthService para crear preferencias por defecto
- [ ] Modificar servicios existentes (BookService, GameService, etc.) para soportar owner filter
- [ ] Modificar controladores existentes para aceptar parámetro owner
- [ ] Frontend: CollectionPreferencesPanel
- [ ] Frontend: PreferencesPage
- [ ] Frontend: PublicProfilePage
- [ ] Frontend: UserProfileHeader
- [ ] Frontend: OtherCollectionsSection
- [ ] Frontend: useCollectionPreferences hook
- [ ] Frontend: Modificar AuthContext con activeCollections
- [ ] Frontend: Modificar Navbar para filtrar por colecciones activas
- [ ] Frontend: Añadir pestañas "Mi colección / Otras colecciones" en listados
- [ ] Tests backend (11 archivos)
- [ ] Tests frontend (5 archivos)
- [ ] Migración de datos para usuarios existentes

---

## Criterios de Aceptación

- [ ] Un usuario puede activar/desactivar colecciones desde preferencias
- [ ] Un usuario puede marcar cada colección como pública o privada
- [ ] Los endpoints GET aceptan parámetro `owner` (mine/other/all)
- [ ] Al listar con `owner=other` solo se muestran colecciones públicas
- [ ] El perfil público muestra solo colecciones públicas activadas
- [ ] El navbar muestra solo las colecciones activas del usuario
- [ ] El Home muestra solo contadores de colecciones activas
- [ ] Las páginas de listado muestran pestañas "Mi colección / Otras colecciones"
- [ ] Un usuario sin auth puede ver perfiles públicos de otros
- [ ] Un usuario no puede ver colecciones privadas de otros (403)
- [ ] Las preferencias se crean por defecto al registrarse
- [ ] Usuarios existentes obtienen preferencias por defecto al arrancar
- [ ] Los tests pasan (`mvn verify` y `npm test`)
- [ ] JaCoCo mantiene ≥80% cobertura

---

## Notas

- **Sin cambios de esquema en colecciones existentes.** Todo se maneja desde user_preferences.
- **Backwards compatible:** Los endpoints sin parámetro `owner` funcionan como `owner=mine`.
- **Cache de preferencias:** Se puede cachear las preferencias del usuario en sesión para evitar llamadas repetidas a MongoDB.
- **Lazy creation:** Si un usuario no tiene preferencias, se crean on-demand (no requiere migración masiva).
- **Colecciones inactivas no desaparecen de la BD.** Solo se ocultan en la UI. Si el usuario las reactiva, sus datos siguen ahí.
- **Visibilidad granular:** La visibilidad es por colección, no por elemento individual (simplicidad).
