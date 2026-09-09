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

**Descripción:** Al crear un libro desde Google Books, no hay validación para evitar duplicados por `externalId`. `SpringDataBookRepository` tiene método `findByExternalId` pero no se usa para validación.

**Impacto:** Posibles duplicados en la colección.

**Solución esperada:** Agregar validación de unicidad o método `findByExternalId` en el servicio.

---

### Issue #43: RAWGClient.getAllGames() no está expuesto en el controlador

**Estado:** OPEN
**Severidad:** Baja

**Descripción:** `RAWGClient` y `FreeToGameClient` implementan métodos adicionales pero no hay endpoint en `GameController` que los exponga.

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

**Estado:** RESOLVED
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

### Issue #36: Tests en frontend

**Estado:** RESOLVED
**Severidad:** Media

**Descripción:** Se configuró Vitest + React Testing Library. Hay tests para BookCard, BookForm y BookSearch.

**Impacto:** Mejora la calidad del código.

**Solución esperada:** Añadir más tests para juegos y Magic.

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

**Estado:** RESOLVED
**Severidad:** Baja

**Descripción:** La wiki fue actualizada con el estado real de los repositorios backend y frontend.

**Plan:** Mantener documentación sincronizada con cada PR.
