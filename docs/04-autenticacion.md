# Autenticación y Autorización

## Estado Actual

La autenticación aún no está implementada. Todos los endpoints son públicos y no hay concepto de usuario.

## Planificación Futura

### Modelo de Usuario

```json
{
  "id": "ObjectId",
  "username": "string (único)",
  "email": "string (único)",
  "passwordHash": "string (bcrypt)",
  "roles": ["USER"],
  "createdAt": "LocalDateTime",
  "updatedAt": "LocalDateTime"
}
```

### Flujo de Autenticación

1. **Registro:** POST /api/auth/register → crea usuario, devuelve JWT
2. **Login:** POST /api/auth/login → valida credenciales, devuelve JWT
3. **Requests:** Header `Authorization: Bearer <token>`
4. **Refresh:** POST /api/auth/refresh → renueva JWT

### Seguridad

- **Passwords:** bcrypt con factor 12
- **JWT:** Access token (15 min) + Refresh token (7 días)
- **CORS:** Configurado para origen del frontend
- **Rate limiting:** 100 requests/minuto por IP

### Autorización

- **Roles:** USER, ADMIN
- **Aislamiento:** Los usuarios solo ven sus propios datos
- **Admin:** Puede ver/gestionar todos los datos
