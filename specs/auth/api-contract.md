# API Contract: Autenticación (Auth)

**Capability:** auth
**Spec:** `specs/auth/spec.md`
**Estado:** ✅ Completada
**Versión:** Fase 8

---

## Base URL

```
http://localhost:8080/api/v1
```

## Endpoints

### Autenticación

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| POST | `/auth/register` | Registro de nuevo usuario | Público |
| POST | `/auth/login` | Login (devuelve JWT) | Público |
| POST | `/auth/refresh` | Refresh del token (devuelve nuevo JWT) | No auth (usa refresh token) |
| POST | `/auth/logout` | Logout (invalida access token) | ✅ Auth |

### User

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| GET | `/users/me` | Obtener usuario autenticado | ✅ Auth |
| PUT | `/users/me` | Actualizar usuario autenticado | ✅ Auth |
| DELETE | `/users/me` | Eliminar usuario autenticado | ✅ Auth |
| GET | `/users/{id}` | Obtener usuario por ID (solo campos públicos) | Público |

---

## Parámetros de Búsqueda

### Login (POST /auth/login)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `username` | String | ✅ | Nombre de usuario |
| `password` | String | ✅ | Contraseña |

### Registro (POST /auth/register)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `username` | String | ✅ | Nombre de usuario (3-50 chars, único) |
| `email` | String | ✅ | Email válido (único) |
| `password` | String | ✅ | Contraseña (min 8 chars) |

### Refresh (POST /auth/refresh)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `refreshToken` | String | ✅ | Refresh token (en body) |

---

## DTOs

### RegisterRequest

```json
{
  "username": "string",
  "email": "string",
  "password": "string"
}
```

### RegisterResponse

```json
{
  "id": "string",
  "username": "string",
  "email": "string",
  "createdAt": "datetime"
}
```

### AuthResponse

```json
{
  "accessToken": "string (JWT)",
  "refreshToken": "string",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

### RefreshRequest

```json
{
  "refreshToken": "string"
}
```

### RefreshResponse (mismo formato que AuthResponse)

### UserResponse (GET /users/{id} — públicos)

```json
{
  "id": "string",
  "username": "string"
}
```

### UserDetailResponse (GET /users/me — detallado)

```json
{
  "id": "string",
  "username": "string",
  "email": "string",
  "enabled": "boolean",
  "accountNonLocked": "boolean",
  "createdAt": "datetime"
}
```

---

## Códigos de Estado

| Código | Significado |
|--------|-------------|
| 200 | OK — token refrescado, perfil obtenido/actualizado |
| 201 | Created — usuario registrado |
| 204 | No Content — logout exitoso |
| 400 | Bad Request — credenciales inválidas, email formato no válido |
| 401 | Unauthorized — credenciales incorrectas, token inválido/expirado |
| 409 | Conflict — username o email duplicado |
| 404 | Not Found — usuario no encontrado |
| 422 | Unprocessable Entity — error al validar usuario (funciona como 400 con mensaje más específico) |

---

## Errores

### AuthenticationException (401)
```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "Credenciales inválidas",
  "path": "/api/v1/auth/login"
}
```

### UsernameAlreadyExistsException (409)
```json
{
  "timestamp": "...",
  "status": 409,
  "error": "Conflict",
  "message": "El nombre de usuario ya existe",
  "path": "/api/v1/auth/register"
}
```

### EmailAlreadyExistsException (409)
```json
{
  "timestamp": "...",
  "status": 409,
  "error": "Conflict",
  "message": "El email ya está registrado",
  "path": "/api/v1/auth/register"
}
```

### UserNotFoundException (404)
```json
{
  "timestamp": "...",
  "status": 404,
  "error": "Not Found",
  "message": "Usuario no encontrado",
  "path": "/api/v1/users/{id}"
}
```

---

## Notas

- **Register califica al usuario antes de guardar** — UserRegistrationValidator se ejecuta en la petición (username 3-50 chars, email formato, password min 8 chars).
- **Login devuelve JWT** — JWT firmado con HS256, expira en 3600 segundos (1h), tiene subject como username.
- **Refresh devuelve nuevo JWT** — el refresh token tiene mayor duración. El endpoint POST /auth/refresh espera el refresh token en el body (no en header).
- **Logout invalida el access token** — en Fase 8, el logout marca el token como inválido. No requiere refresh token.
- **Token storage:** JWT se almacena en localStorage del frontend. Refresh token se almacena en memoria o en una cookie segura.
- **Token-based security:** el token JWT se pasa en el header `Authorization: Bearer <token>`.
