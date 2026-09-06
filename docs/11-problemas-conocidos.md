# Problemas Conocidos

## Backend

### Issue #40: Credenciales MongoDB hardcodeadas en application.properties

**Estado:** OPEN
**Severidad:** Alta

**Descripción:** El archivo `application.properties` contiene credenciales de MongoDB directamente en el código fuente.

**Impacto:** Riesgo de seguridad si el repositorio es público o se comparte.

**Solución esperada:** Parametrizar con variable de entorno `SPRING_MONGODB_URI`.

---

### Issue #41: GameSearchService lanza IllegalArgumentException en búsqueda fallback

**Estado:** OPEN
**Severidad:** Media

**Descripción:** Si RAWG devuelve resultados vacíos, se hace fallback a FreeToGame. Sin embargo, si FreeToGame también falla, se devuelve lista vacía sin distinguir entre "no hay resultados" y "error de API".

**Impacto:** Errores 500 inesperados cuando las APIs externas fallan.

**Solución esperada:** Manejo de excepciones más robusto con respuestas HTTP significativas.

---

### Issue #44: BookDtoMapper.toDomain() no valida unicidad de externalId

**Estado:** OPEN
**Severidad:** Media

**Descripción:** Al crear un libro desde Google Books, no hay validación para evitar duplicados por `externalId`. `SpringDataBookRepository` no tiene método `findByExternalId` (a diferencia de `SpringDataGameRepository`).

**Impacto:** Posibles duplicados en la colección.

**Solución esperada:** Agregar validación de unicidad o método `findByExternalId`.

---

### Issue #43: RAWGClient.getAllGames() no está expuesto en el controlador

**Estado:** OPEN
**Severidad:** Baja

**Descripción:** `RAWGClient` y `FreeToGameClient` implementan `getAllGames()` pero no hay endpoint en `GameController` que lo exponga.

**Impacto:** Funcionalidad implementada pero no accesible.

**Solución esperada:** Exponer endpoint `GET /api/games/discover` o eliminar el método.

---

## Frontend

### Issue #41: tailwind.config.js no detecta archivos TypeScript (.tsx/.ts)

**Estado:** OPEN
**Severidad:** Media

**Descripción:** `content: ['./index.html', './src/**/*.{js,jsx}']` no incluye `.tsx` ni `.ts`.

**Impacto:** Clases de Tailwind pueden no funcionar correctamente.

**Solución esperada:** Cambiar a `content: ['./index.html', './src/**/*.{js,jsx,ts,tsx}']`.

---

### Issue #42: HomePage.tsx documentado como huérfano pero está enrutado

**Estado:** OPEN
**Severidad:** Baja

**Descripción:** `AGENTS.md` dice que `HomePage.jsx` está huérfano, pero en `App.tsx` la ruta `/` renderiza `HomePage`. Además, el archivo es `.tsx` no `.jsx`.

**Impacto:** Documentación confusa para colaboradores.

**Solución esperada:** Actualizar `AGENTS.md`.

---

### Issue #43: window.alert inconsistente en BookSearch y BookEditPage

**Estado:** OPEN
**Severidad:** Baja

**Descripción:** Los errores se muestran con `window.alert()` en lugar de componentes React estilizados.

**Impacto:** UX inconsistente.

**Solución esperada:** Usar componente de toast/notification.

---

### Issue #36: No hay tests configurados en el frontend

**Estado:** OPEN
**Severidad:** Media

**Descripción:** No hay framework de testing configurado (Jest, Vitest, Testing Library).

**Impacto:** No hay verificación automática de regresiones.

**Solución esperada:** Configurar Vitest + React Testing Library.

---

### Issue #37: HomePage hace 4 llamadas API para estadísticas

**Estado:** OPEN
**Severidad:** Baja

**Descripción:** `fetchStats()` hace 4 llamadas paralelas para obtener totales por estado.

**Impacto:** 4 requests donde podría haber 1.

**Solución esperada:** Crear endpoint `/api/books/stats` en el backend.

---

## Documentación

### Wiki desactualizada

**Estado:** EN PROGRESO
**Severidad:** Baja

**Descripción:** La wiki necesita actualización con el estado real de los repositorios.

**Plan:** Actualizar documentación con los issues encontrados en la revisión.
