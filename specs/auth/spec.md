# Spec: Autenticación (Auth)

**Estado:** ✅ Completada
**Capa:** Domain + Application + Infrastructure
**Modelo:** User (domain/model/User.java)

---

## Dominio

### Entidad User

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único |
| username | String | ✅ | Nombre de usuario (unique, 3-20 chars, regex `^[a-zA-Z0-9_]+$`) |
| email | String | ✅ | Email (unique, validado, formato email) |
| password | String | ✅ | Hash BCrypt (nunca se devuelve en responses) |
| displayName | String | ❌ | Nombre para mostrar |
| avatarUrl | String | ❌ | URL de avatar |
| bio | String | ❌ | Biografía corta (max 200 chars) |
| createdAt | LocalDateTime | Auto | Fecha de registro |
| updatedAt | LocalDateTime | Auto | Última actualización |

### Entidad AuthSession (record)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| accessToken | String | JWT access token |
| refreshToken | String | JWT refresh token |
| user | User | Usuario autenticado |

### UserOwned (record)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| userId | String | ID del usuario |
| username | String | Nombre de usuario |
| displayName | String | Nombre para mostrar |

### Reglas de Negocio

- **BCrypt** con strength 10 para passwords
- **JWT access token:** expiración 15 minutos (900000 ms)
- **JWT refresh token:** expiración 7 días (604800000 ms)
- **GET endpoints son públicos** — cualquier usuario (anónimo) puede ver colecciones
- **POST/PUT/DELETE requieren autenticación** — solo usuarios registrados pueden crear/editar/eliminar
- **Ownership:** un usuario solo puede editar/eliminar sus propios elementos
- **Admin por defecto:** usuario admin creado al arrancar (configurado por env vars: ADMIN_USERNAME, ADMIN_PASSWORD, ADMIN_EMAIL)
- **Datos existentes:** se asocian a `ownerId = "system"` (cuenta admin por defecto)

---

## Puertos (Interfaces de Dominio)

### AuthUseCase (in)
```java
public interface AuthUseCase {
    AuthResponse register(RegisterRequest request);
    AuthResponse login(LoginRequest request);
    AuthResponse refreshToken(String refreshToken);
}
```

### UserUseCase (in)
```java
public interface UserUseCase {
    User findById(String id);
    User findByUsername(String username);
    User findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
    User save(User user);
}
```

### UserRepository (out)
```java
public interface UserRepository {
    Optional<User> findById(String id);
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
    User save(User user);
}
```

---

## Services

| Service | Responsabilidad |
|---------|-----------------|
| `AuthService` | Login, register, refresh token |
| `JwtService` | Generación y validación de JWT (access + refresh) |
| `UserDetailsServiceImpl` | UserDetailsService para Spring Security (carga por userId) |

---

## Configuración

### application.properties

```properties
# JWT
app.jwt.secret=${JWT_SECRET:clave-cambiar-en-produccion-min-256-bits}
app.jwt.access-token-expiration=900000      # 15 minutos
app.jwt.refresh-token-expiration=604800000  # 7 días

# Admin por defecto
app.admin.username=${ADMIN_USERNAME:admin}
app.admin.password=${ADMIN_PASSWORD:admin123}
app.admin.email=${ADMIN_EMAIL:admin@local.dev}
```

### Generar JWT_SECRET

```bash
openssl rand -base64 32
```

---

## Controlador REST

### AuthController

**Base:** `/api/v1/auth`

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| POST | `/api/v1/auth/register` | Registrar usuario | Público |
| POST | `/api/v1/auth/login` | Login (devuelve accessToken + refreshToken) | Público |
| POST | `/api/v1/auth/refresh` | Refrescar accessToken | Público |
| GET | `/api/v1/auth/me` | Obtener usuario actual | Auth |

### DTOs de Auth

#### RegisterRequest
```json
{
  "username": "string (3-20 chars, regex ^[a-zA-Z0-9_]+$)",
  "email": "string (formato email, unique)",
  "password": "string (min 8 chars)",
  "displayName": "string"
}
```

#### LoginRequest
```json
{
  "username": "string",
  "password": "string"
}
```

#### RefreshTokenRequest
```json
{
  "refreshToken": "string"
}
```

#### AuthResponse
```json
{
  "accessToken": "string",
  "refreshToken": "string",
  "user": {
    "id": "string",
    "username": "string",
    "displayName": "string",
    "avatarUrl": "string",
    "bio": "string",
    "createdAt": "datetime"
  }
}
```

#### UserResponse
```json
{
  "id": "string",
  "username": "string",
  "displayName": "string",
  "avatarUrl": "string",
  "bio": "string",
  "createdAt": "datetime"
}
```

---

## Seguridad

### Spring Security Config

- `@EnableWebSecurity` + `@EnableMethodSecurity`
- `SessionCreationPolicy.STATELESS` (sin sesiones HTTP)
- CSRF deshabilitado (API REST + JWT)
- **Reglas de acceso:**
  - `GET /api/v1/**` → permitAll (público)
  - `/api/v1/auth/**` → permitAll
  - `/swagger-ui/**`, `/v3/api-docs/**` → permitAll
  - Todo lo demás → authenticated

### JwtAuthenticationFilter

- `OncePerRequestFilter` que intercepta cada request
- Extrae Bearer token del header Authorization
- Valida el token con JwtService
- Si válido, carga el UserDetails y setea el SecurityContext
- Si no hay token o es inválido, continua (no bloquea — reglas de acceso lo manejan)

### @CurrentUser Annotation

- `@Target(ElementType.PARAMETER)` + `@Retention(RUNTIME)`
- Resolver: `CurrentUserHandlerMethodArgumentResolver`
- Extrae userId del SecurityContext y lo inyecta como parámetro String en controllers

---

## Frontend

### AuthContext (React Context)

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| user | UserResponse \| null | Usuario autenticado |
| accessToken | string \| null | Token de acceso actual |
| login | (username, password) => Promise | Iniciar sesión |
| register | (data) => Promise | Registrar nuevo usuario |
| logout | () => void | Cerrar sesión |
| isAuthenticated | boolean | ¿Hay usuario autenticado? |
| activeCollections | string[] | Colecciones activas del usuario |
| refreshActiveCollections | () => Promise | Refrescar colecciones activas |
| preferences | UserPreferences \| null | Preferencias de colección |
| refreshPreferences | () => Promise | Refrescar preferencias |

### ProtectedRoute

Componente que redirige a `/login` si no hay usuario autenticado.

### Axios Interceptor (authFetch)

- **Request:** añade Bearer token a Authorization header si existe
- **Response:** intercepta 401, intenta refresh token automáticamente, si falla → logout + redirect a /login

---

## Visibilidad

| Acción | Anónimo | Usuario registrado | Admin |
|--------|---------|-------------------|-------|
| Ver colecciones de otros | ✅ | ✅ | ✅ |
| Ver notas/valoraciones propias | ❌ | ✅ | ✅ |
| Ver notas/valoraciones de otros | ❌ | ❌ | ✅ |
| Crear elementos | ❌ | ✅ | ✅ |
| Editar/eliminar elementos propios | ❌ | ✅ | ✅ |
| Editar/eliminar cualquier elemento | ❌ | ❌ | ✅ |

---

## Tests

| Test | Tipo | Descripción |
|------|------|-------------|
| `AuthServiceTest` | Unitario | Registro, login, refresh token |
| `JwtServiceTest` | Unitario | Generación y validación de tokens |
| `AuthControllerTest` | Integración | Endpoints de auth con MockMvc |
| `JwtAuthenticationFilterTest` | Unitario | Filtro con SecurityContext |
| `SecurityConfigTest` | Unitario | Configuración de seguridad |
| `AuthDataMigrationTest` | Integración | Creación de admin por defecto |
| `UserDetailsServiceImplTest` | Unitario | Carga de usuario para Spring Security |
| `CurrentUserHandlerMethodArgumentResolverTest` | Unitario | Resolver @CurrentUser |
| `AuthJwtFlowTest` | Integración | Flujo completo auth JWT |
| `SecurityMatrixTest` | Integración | Verificación de reglas de acceso |

---

## Criterios de Aceptación

- [x] Un usuario puede registrarse vía `POST /api/v1/auth/register`
- [x] Un usuario puede logearse vía `POST /api/v1/auth/login`
- [x] El access token expira en 15 minutos
- [x] El refresh token expira en 7 días
- [x] Se puede refrescar el access token vía `POST /api/v1/auth/refresh`
- [x] Los endpoints GET son públicos (sin auth)
- [x] Los endpoints POST/PUT/DELETE requieren auth
- [x] Un usuario solo puede editar/eliminar sus propios elementos
- [x] Las notas/valoraciones son privadas (solo visibles por el dueño)
- [x] Las colecciones son públicas (visibles por anónimos)
- [x] El frontend redirige a login si intenta crear sin auth
- [x] El frontend maneja expiración de token con refresh automático
- [x] Los tests pasan (`mvn verify` y `npm test`)
- [x] JaCoCo mantiene ≥80% cobertura

---

## Estado de Implementación

Fase 8 completada. Todos los componentes implementados:
- ✅ Domain: User.java, UserOwned.java, AuthSession.java, AuthTokens.java
- ✅ Ports: AuthUseCase.java (no existe como interfaz separada, la lógica está en AuthService), UserUseCase.java, UserRepository.java, UserDetailsService (Spring Security)
- ✅ Application: AuthService.java, JwtService.java, UserDetailsServiceImpl.java
- ✅ Infrastructure: AuthController.java, JwtAuthenticationFilter.java, SecurityConfig.java, UserEntity.java, UserPersistenceAdapter.java, SpringDataUserRepository.java, UserEntityMapper.java, AuthResponse.java, RegisterRequest.java, LoginRequest.java, RefreshTokenRequest.java, UserResponse.java
- ✅ Config: CurrentUser.java, CurrentUserHandlerMethodArgumentResolver.java, AuthDataMigration.java
- ✅ Exceptions: UserAlreadyExistsException.java, EmailAlreadyExistsException.java, UserNotFoundException.java, InvalidTokenException.java, ForbiddenException.java, UnauthenticatedException.java
- ✅ Tests: 10 archivos de test backend
