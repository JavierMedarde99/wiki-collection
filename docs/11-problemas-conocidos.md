# Problemas Conocidos

## Backend

### Issue #17: Error de autenticación de MongoDB en arranque

**Estado:** OPEN
**Severidad:** Alta

**Descripción:** Credenciales hardcodeadas en `application.properties` causan fallo de autenticación.

**Solución:** Parametrizar con variable de entorno `SPRING_MONGODB_URI`.

**PR relacionado:** #18 (pendiente mergear)

### FreeToGame no soporta búsqueda por nombre

**Estado:** WORKAROUND
**Severidad:** Media

**Descripción:** La API de FreeToGame ignora los parámetros `title`, `name` y `search`. Devuelve todos los juegos.

**Workaround:** El backend descarga todos los juegos y filtra por título en memoria usando Java Streams.

**Mitigación futura:** Cachear la lista de juegos con TTL de 1 hora.

## Frontend

### Issue #3: Eliminar libro desde lista

**Estado:** OPEN
**Severidad:** Baja

**Descripción:** La eliminación desde la lista funciona pero falta confirmación visual mejorada.

**PR relacionado:** #10 (mergeado, falta confirmar cierre)

## Documentación

### Wiki desactualizada

**Estado:** EN PROGRESO
**Severidad:** Baja

**Descripción:** La wiki necesita reorganización y actualización con los últimos cambios.

**Plan:** Reestructurar en carpetas por tema (este PR).
