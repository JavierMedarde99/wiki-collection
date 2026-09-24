# Autenticación — Wiki-Collection

**Capa:** Application + Infrastructure
**Estado:** ✅ Completada (Fase 8)

## Autenticación por JWT

El backend usa **JWT (JSON Web Tokens)** para autenticar usuarios. El flujo es:

1. El usuario envía sus credenciales a `/api/v1/auth/login`
2. Si son válidas, el backend genera un JWT y lo devuelve
3. El cliente incluye el JWT en las peticiones posteriores (header `Authorization: Bearer <token>`)
4. El backend valida el JWT usando `JwtAuthenticationFilter`

### Componentes JWT

| Componente | Propósito |
|------------|-----------|
| `JwtTokenProvider` (infrastructure/adapter/provider) | Genera y valida tokens JWT (depende de `io.jsonwebtoken:jjwt-api:0.12.6`) |
| `JwtAuthenticationFilter` (infrastructure/adapter/filter) | Intercepta peticiones HTTP y valida el token en el header Authorization |
| `JwtExceptionHandler` (infrastructure/adapter/handler) | Maneja excepciones JWT: token expirado, inválido, faltante → 401 o 422 |

### Almacenamiento del token en frontend

La sesión del usuario se basa en un JWT almacenado en `localStorage`. Al recargar la página, el frontend restaura el usuario desde el token:

```ts
// En useAuth: al montar, si hay token en localStorage, llama a /api/v1/auth/me
// para obtener la información del usuario actual.
```

### Pasos de implementación (Fase 8, checklist)

- [x] Añadir dependencia `io.jsonwebtoken:jjwt-api:0.12.6` (SSI: runtime scope)
- [x] Crear `JwtTokenProvider` (infraestructura, genera y valida JWT)
- [x] Crear `JwtAuthenticationFilter` (intercepta e inyecta `SecurityContext` con `Authentication`)
- [x] Crear controlador de auth: `POST /api/v1/auth/login`, `POST /api/v1/auth/register`
- [x] Crear servicio de autenticación: valida credenciales, devuelve JWT
- [x] Añadir `JwtAuthenticationFilter` al `SecurityFilterChain` (antes de `UsernamePasswordAuthenticationFilter`)
- [x] Añadir excepciones de auth: `AuthenticationException` → 401; `UserNotFoundException` → 404
- [x] Añadir controlador de registro: `POST /api/v1/auth/register` con validación de username (3-50 chars), email formato, password min 8 chars, username/email únicos
- [x] Añadir controlador de user: `GET/PUT/DELETE /api/v1/users/me` y `GET /api/v1/users/{id}` solo public (username, createdAt)
- [x] Añadir controlador de refresh: `POST /api/v1/auth/refresh` (recarga JWT desde refresh token)
- [x] Añadir controlador de logout: `POST /api/v1/auth/logout` (invalida access token si existe)
- [x] Añadir validaciones de DTO: `RegisterRequestValidator`, `LoginRequestValidator`, `RefreshRequestValidator`
- [x] Añadir DTOs: `RegisterRequest`, `RegisterResponse`, `LoginRequest`, `AuthResponse`, `RefreshRequest`, `RefreshResponse`, `UserResponse`, `UserDetailResponse`
- [x] Añadir dominio: `User` (entity model + mappers: Optional<User> → UserResponse, User → UserResponse)
- [x] Añadir repositories: `UserRepository` (findByUsername, findByEmail, existsByUsername, existsByEmail)
- [x] Añadir excepciones: `UsernameAlreadyExistsException`, `EmailAlreadyExistsException`, `AuthenticationException`, `RefreshTokenException`
- [x] Tests: `JwtTokenProviderTest`, `JwtAuthenticationFilterTest`, `AuthControllerTest`, `UserControllerTest`, `RefreshControllerTest`, `AuthServiceTest`

### Decisión de password

- La contraseña se guarda como hash BCrypt en la BD (no en texto plano).
- BCrypt es generado por Spring Security en la capa de dominio o service: `PasswordEncoder` (BCrypt).

### Reglas de validación de registro (Fase 8, UserService o RegisterRequestValidator)

- `username`: 3-50 caracteres, único, no vacío
- `email`: formato válido (regex o `@Email`), único
- `password`: mínimo 8 caracteres, no vacío

## API Endpoints de Auth

### Login

```
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "javi",
  "password": "secret123"
}
```

**Respuesta 200:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

**Respuesta 401 (credenciales incorrectas):**
```json
{
  "status": 401,
  "error": "Unauthorized",
  "message": "Credenciales inválidas"
}
```

### Register

```
POST /api/v1/auth/register
Content-Type: application/json

{
  "username": "javi",
  "email": "javi@example.com",
  "password": "secret123"
}
```

**Respuesta 201:**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "username": "javi",
  "email": "javi@example.com",
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

**Respuesta 409 (username o email duplicado):**
```json
{
  "status": 409,
  "error": "Conflict",
  "message": "El nombre de usuario o email ya existe"
}
```

### Refresh

```
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Respuesta 200:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

### Logout

```
POST /api/v1/auth/logout
Authorization: Bearer <token>
```

**Respuesta 204 — sin contenido.**

### Perfil del usuario autenticado

```
GET /api/v1/users/me
Authorization: Bearer <token>
```

**Respuesta 200:**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "username": "javi",
  "email": "javi@example.com",
  "enabled": true,
  "accountNonLocked": true,
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

### Actualizar perfil del usuario autenticado

```
PUT /api/v1/users/me
Authorization: Bearer <token>
Content-Type: application/json

{
  "username": "javi2",
  "email": "javi2@example.com",
  "enabled": true,
  "accountNonLocked": true
}
```

**Respuesta 200:** igual que `GET /users/me`.

### Eliminar perfil del usuario autenticado

```
DELETE /api/v1/users/me
Authorization: Bearer <token>
```

**Respuesta 204 — sin contenido.**

### Obtener usuario por ID (solo información pública)

```
GET /api/v1/users/{id}
```

**Respuesta 200:**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "username": "javi"
}
```

**Respuesta 404:** `{"status":404,"error":"Not Found","message":"Usuario no encontrado"}`

---

## Tabla de decisión: Spring Security

| Opción | Razón |
|--------|-------|
| `JwtAuthenticationFilter` antes que `UsernamePasswordAuthenticationFilter` | No usamos login por formulario; la autenticación es siempre vía JWT |
| `SecurityFilterChain` sin CSRF | API REST stateless, sin cookies de sesión |
| `JwtAuthenticationFilter` para reconocer usuario autenticado en tests | Permite @WithMockUser y pasar usuario al contexto para tests con auth |
| Firma HS256, 256 bits | JWT estándar, compatible con jjwt |
| Token expira en 1 hora | Seguridad razonable para una app personal |
| Refresh token expira en 7 días | Balance entre seguridad y experiencia de usuario |

---

## Tests (Fase 8)

- `JwtTokenProviderTest`: firma token, verifica firma, extrae claims, detecta token caducado
- `JwtAuthenticationFilterTest`: cuando hay header Authorization → SecurityContext tiene Authentication; cuando no hay → SecurityContext vacío; cuando token inválido → SecurityContext vacío y lanza excepción
- `AuthControllerTest`: login() con credenciales correctas devuelve 200 + JWT; login() con credenciales incorrectas devuelve 401; register() válido devuelve 201; register() con username duplicado devuelve 409; register() con email duplicado devuelve 409
- `UserControllerTest`: me() devuelve 200 con user; updateMe() válido devuelve 200; updateMe() con username duplicado devuelve 409; deleteMe() devuelve 204; getUser() devuelve 200 con user público; getUser() con ID inexistente devuelve 404
- `RefreshControllerTest`: refresh() válido devuelve 200 + JWT nuevo; refresh() con token inválido devuelve 401
- `AuthServiceTest`: authenticate() con usuario y password correctos devuelve AuthResponse; authenticate() con password incorrecto lanza AuthenticationException; register() con datos válidos guarda usuario y devuelve RegisterResponse; register() con datos inválidos lanza excepción de validación

---

## Decisiones técnicas (ADR-010 y relacionados, ver research/decisions/)

- Uso de JWT en vez de sesiones HTTP: la app es una SPA, y las sesiones HTTP no son prácticas para SPAs (necesitarían cookies con CSRF). JWT permite al frontend almacenar el token y enviarlo en cada petición.
- Token corto de vida (1 hora) + refresh token largo (7 días): permite al usuario permanecer autenticado sin tener que volver a hacer login frecuentemente, pero limita el tiempo de validez del token de acceso en caso de robo.
- Refresh token en el body de la petición, no como cookie: el frontend envía el refresh token desde localStorage; es responsabilidad del frontend proteger el refresh token (ej: no exponerlo a XSS). En un contexto de producción, se puede considerar alternativas más seguras (ej: cookies HttpOnly).

---

## Notas futuras (fuera de Fase 8)

- OAuth2: no implementado en esta fase. `OAuth2UserServicePort` se añadió como puerto pendiente de implementar cuando se integre un provider OAuth2 (Google, GitHub, etc.).
- Recuperación de contraseña: `ForgotPasswordApiController` existe como placeholder (devuelve 404). No implementada en esta fase.
