# Tasks — Migración de Wiki-Collection a Base de Conocimiento Spec-Driven

## Contexto

Este documento lista las tareas para migrar la documentación existente en `docs/` a la nueva estructura propuesta en `design.md`. Las tareas se han agrupado por dependencias: primero setup y resolución de preguntas, luego creación de specs, luego migración de documentación existente, finalmente verificación.

**Prerrequisitos:** `proposal.md` y `design.md` deben estar aprobados antes de ejecutar estas tareas.

---

## 0. Resolver preguntas abiertas (ANTES de empezar)

Estas preguntas deben resolverse antes de ejecutar cualquier tarea de migración, porque afectan el contenido de los documentos resultantes.

Las siguientes ya están resueltas (ver `decisions.md`):

- ✅ 0.1 Resolver Q1: ¿Fase 9 está completada o planificada? → **RESUELTO: Completada** (ver DF1 en decisions.md)
- ✅ 0.2 Resolver Q2: ¿Qué hacer con los 3 archivos de auth obsoletos? → **RESUELTO: Eliminar** (ya en tarea 3.1)
- ✅ 0.3 Resolver Q3: ¿Se mantiene `01-requisitos.md`? → **RESUELTO: Se mantiene** (ver DF3 en decisions.md)

Pendientes de resolver:

- [ ] 0.4 Resolver Q4: ¿Los archivos de fase (13-21) se convierten en stubs o se eliminan? → **RESUELTO: Stubs** (ver DF2 en decisions.md)
- [ ] 0.5 Resolver Q5: ¿Se mantiene el changelog en su ubicación actual o se mueve a `docs/history/`?
- [ ] 0.6 Resolver Q6: Verificar si existe `docs/README.md` (en raíz del repo) y si es relevante para la migración.
- [ ] 0.7 Resolver Q7: ¿Qué hacer con `30-cambio-implementado-game-platforms.md`? Decidir entre mover a research, eliminar o mantener.
- [ ] 0.8 Resolver A1: Verificar en `backend-collection` si existe el endpoint de búsqueda por ISBN y actualizar `proposal.md` con el estado real.
- [ ] 0.9 Resolver A3: Verificar número real de tests (56 vs 68) corriendo tests en `backend-collection` y actualizar `proposal.md`.
- [ ] 0.10 Resolver A4: Verificar en `backend-collection` si `SteamCatalogueClient` y `SteamAchievementsClient` coexisten o si son uno solo.
- [ ] 0.11 Resolver A5: Verificar en `backend-collection` si `Game.java` tiene campo `genre: String`.
- [ ] 0.12 Resolver A6: Completar AD-014+ en `10-decisiones-tecnicas.md` (actualmente truncado).
- [ ] 0.13 Resolver A7: Decidir si `18-fase-7-caffeine-search.md` se convierte en stub o se integra directamente en `architecture/backend.md`.
- [ ] 0.14 Resolver A8: Corregir el texto coreano en `08-deploy.md` ("동일한 지역 선택") antes de migrar.

**Verificación:** `proposal.md` y `design.md` no contienen preguntas sin responder.

---

## 1. Crear estructura de directorios

- [ ] 1.1 Crear directorios `docs/research/decisions/`, `docs/research/known-issues/`, `docs/research/external-apis/`, `docs/research/future/`
- [ ] 1.2 Crear directorios `docs/specs/collection/books/`, `docs/specs/collection/games/`, `docs/specs/collection/board-games/`, `docs/specs/collection/magic-cards/`, `docs/specs/collection/decks/`, `docs/specs/collection/movie-shows/`
- [ ] 1.3 Crear directorios `docs/specs/auth/`, `docs/specs/user-preferences/`, `docs/specs/image-storage/`
- [ ] 1.4 Crear directorios `docs/architecture/` y `docs/data-model/`

**Verificación:** `find docs -type d | sort` muestra todos los directorios esperados.

---

## 2. Crear especificaciones (specs)

Estas tareas crean los archivos `spec.md` y `api-contract.md` para cada capability. El contenido se basa en los documentos de fase existentes (13-21) que ya tienen el contenido.

### 2.1 Colección de Libros

- [ ] 2.1.1 Crear `docs/specs/collection/books/spec.md` con: objetivo, reglas de negocio (states TO_READ/READING/COMPLETED, types MANGA/NOVEL/GRAPHIC_NOVEL, pagesRead <= pages si definido, ISBN limpiado de guiones/espacios, varias ediciones con mismo ISBN permitidas, notas y valoración privadas), campos de dominio (resumen), visibilidad, criterios de aceptación, estado actual ✅.
- [ ] 2.1.2 Crear `docs/specs/collection/books/api-contract.md` con: base path `/api/v1/books`, endpoints GET /books, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=..., GET /search?isbn=..., GET /search?author=..., filtros ?state= ?type= ?owner=, response shape BookResponse, códigos de error 404/409, paginación Spring Data.

**Verificación:** Ambos archivos existen y contienen las secciones del template.

### 2.2 Colección de Videojuegos

- [ ] 2.2.1 Crear `docs/specs/collection/games/spec.md` con: objetivo, reglas de negocio (states PLAYING/COMPLETED/WISHLIST/ABANDONED, plataformas PC/PS2/PS3/WII_U/SWITCH, estrategia de búsqueda RAWG → FreeToGame fallback, logros Steam), campos de dominio, visibilidad, criterios de aceptación, estado actual ✅.
- [ ] 2.2.2 Crear `docs/specs/collection/games/api-contract.md` con: endpoints GET /games, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=..., GET /{gameId}/achievements?steamId=..., filtros ?platform= ?status= ?owner=, response shapes, códigos de error, paginación.

**Verificación:** Ambos archivos existen.

### 2.3 Colección de Juegos de Mesa

- [ ] 2.3.1 Crear `docs/specs/collection/board-games/spec.md` con: objetivo, reglas de negocio (states OWNED/WISHLIST, BGG XML API 2 como única fuente, reintentos por 202 Accepted, parseo XML con Jackson), campos de dominio, visibilidad, criterios de aceptación, estado actual ✅.
- [ ] 2.3.2 Crear `docs/specs/collection/board-games/api-contract.md` con: endpoints GET /boardgames, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=..., filtros ?status= ?owner=, response shapes, códigos de error, paginación.

**Verificación:** Ambos archivos existen.

### 2.4 Colección de Cartas Magic

- [ ] 2.4.1 Crear `docs/specs/collection/magic-cards/spec.md` con: objetivo, reglas de negocio (condiciones MINT/NEAR_MINT/EXCELLENT/GOOD/PLAYED/POOR, idiomas, backend sin save/update genérico — solo addFromScryfall + delete, Scryfall como API primaria), campos de dominio, visibilidad, criterios de aceptación, estado actual ✅.
- [ ] 2.4.2 Crear `docs/specs/collection/magic-cards/api-contract.md` con: endpoints GET /magic, GET /{id}, POST /scryfall/{scryfallId}, DELETE /{id}, GET /search?name=..., GET /commanders?colors=..., filtros ?name= ?rarity= ?color= ?type= ?owner=, response shapes, códigos de error, paginación.

**Verificación:** Ambos archivos existen.

### 2.5 Mazos Commander

- [ ] 2.5.1 Crear `docs/specs/collection/decks/spec.md` con: objetivo, reglas de negocio (estados del mazo DRAFT/COMPLETE/INVALID, validación de reglas Commander, Scryfall para comandantes y cartas), campos de dominio, visibilidad, criterios de aceptación, estado actual ✅.
- [ ] 2.5.2 Crear `docs/specs/collection/decks/api-contract.md` con: endpoints GET /decks, GET /{id}, POST, PUT /{id}, DELETE /{id}, POST /{id}/cards, DELETE /{id}/cards/{scryfallId}, GET /{id}/status, filtros ?name= ?owner=, DTOs DeckCardRequest y DeckStatusResponse, códigos de error, paginación.

**Verificación:** Ambos archivos existen.

### 2.6 Películas/Series

- [ ] 2.6.1 Crear `docs/specs/collection/movie-shows/spec.md` con: objetivo, reglas de negocio (types MOVIE/TV, states WATCHING/WATCHED/PLAN_TO_WATCH sin WISHLIST, TMDB como API primaria, idioma es-ES, externalId único para evitar duplicados), campos de dominio, visibilidad, criterios de aceptación, estado actual ✅.
- [ ] 2.6.2 Crear `docs/specs/collection/movie-shows/api-contract.md` con: endpoints GET /movieshows, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=...&mediaType=..., filtros ?status= ?mediaType= ?owner=, response shapes, códigos de error, paginación.

**Verificación:** Ambos archivos existen.

### 2.7 Autenticación

- [ ] 2.7.1 Crear `docs/specs/auth/spec.md` con: objetivo, reglas de negocio (flujos register/login/refresh, JWT access 15min + refresh 7días, BCrypt, roles USER/ADMIN, reglas de visibilidad por rol, ownerId = "system" para datos pre-auth, refresh tokens en localStorage), campos de dominio, criterios de aceptación, estado actual ✅.
- [ ] 2.7.2 Crear `docs/specs/auth/api-contract.md` con: endpoints POST /auth/register, POST /auth/login, POST /auth/refresh, GET /auth/me, request/response shapes (RegisterRequest, LoginRequest, RefreshTokenRequest, AuthResponse, UserResponse), códigos de error 400/401/403/409.

**Verificación:** Ambos archivos existen.

### 2.8 Preferencias de Usuario

- [ ] 2.8.1 Crear `docs/specs/user-preferences/spec.md` con: objetivo, reglas de negocio (colecciones activas por defecto todas activas y públicas, visibilidad PUBLIC/PRIVATE por colección, perfiles públicos, colecciones inactivas ocultas pero no borradas, lazy creation, backwards compatible), campos de dominio, criterios de aceptación, estado actual según verificación en 0.1.
- [ ] 2.8.2 Crear `docs/specs/user-preferences/api-contract.md` con: endpoints de preferencias GET/PUT/PATCH /preferences y sub-endpoints, endpoints de perfil público GET /users/{username} y submirrors, response shapes UserPreferencesResponse/PublicProfileResponse/PublicCollectionSummary, códigos de error 403/404.

**Verificación:** Ambos archivos existen.

### 2.9 Almacenamiento de Imágenes

- [ ] 2.9.1 Crear `docs/specs/image-storage/spec.md` con: objetivo, reglas de negocio (Catbox.moe como hosting, imágenes de perfil e items subidas por usuario, imágenes de APIs externas usan URLs originales, no filesystem local), campos de dominio, criterios de aceptación, estado actual ✅.
- [ ] 2.9.2 Crear `docs/specs/image-storage/api-contract.md` con: endpoints POST /images/upload, DELETE /images/{filename}, validaciones 5MB/MIME/magic bytes, configuración catbox.api.base-url y catbox.userhash, response ImageResponse (url, filename), códigos de error 400/502.

**Verificación:** Ambos archivos existen.

---

## 3. Migración de documentación existente

### 3.1 Eliminar documentación obsoleta (sin contenido útil)

- [ ] 3.1.1 Eliminar `docs/04-autenticacion.md` (contenido obsoleto, ya en fase 8)
- [ ] 3.1.2 Eliminar `docs/05-api/05.1-usuarios.md` (contenido absorbido en specs/auth/)
- [ ] 3.1.3 Eliminar `docs/05-api/05.2-autenticacion.md` (contenido absorbido en specs/auth/)

**Verificación:** Los archivos no existen.

### 3.2 Mover documentación de investigación

- [ ] 3.2.1 Mover `docs/05-api/externas/externas-books.md` → `docs/research/external-apis/google-books.md`
- [ ] 3.2.2 Mover `docs/05-api/externas/externas-videogames.md` → `docs/research/external-apis/rawg.md`
- [ ] 3.2.3 Mover `docs/05-api/externas/externas-steam.md` → `docs/research/external-apis/steam.md`
- [ ] 3.2.4 Mover `docs/05-api/externas/externas-boardgames.md` → `docs/research/external-apis/boardgamegeek.md`
- [ ] 3.2.5 Mover `docs/05-api/externas/externas-magic.md` → `docs/research/external-apis/scryfall.md`
- [ ] 3.2.6 Mover `docs/05-api/externas/externas-movies.md` → `docs/research/external-apis/tmdb.md`
- [ ] 3.2.7 Mover `docs/05-api/externas/externas-image-hosting.md` → `docs/research/external-apis/catbox.md`
- [ ] 3.2.8 Mover `docs/05-api/externas/steam-api-key-guide.md` → `docs/research/external-apis/steam-api-key-guide.md`
- [ ] 3.2.9 Mover `docs/10-decisiones-tecnicas.md` → `docs/research/decisions/adr.md`
- [ ] 3.2.10 Mover `docs/11-problemas-conocidos.md` → `docs/research/known-issues/issues.md`
- [ ] 3.2.11 Mover `docs/22-futuro-barcode-scanner.md` → `docs/research/future/barcode-scanner.md`
- [ ] 3.2.12 Mover `docs/23-futuro-deck-import.txt.md` → `docs/research/future/deck-import-text.md`
- [ ] 3.2.13 Mover `docs/24-futuro-reading-progress.md` → `docs/research/future/reading-progress.md`
- [ ] 3.2.14 Mover `docs/25-futuro-streaming-platforms.md` → `docs/research/future/streaming-platforms.md`
- [ ] 3.2.15 Mover `docs/26-futuro-generos-colecciones.md` → `docs/research/future/genres-analysis.md` (y consolidar con 27 en genera.md si se decide)
- [ ] 3.2.16 Mover `docs/27-futuro-generos-books-games-boardgames.md` → `docs/research/future/genres-implementation.md`
- [ ] 3.2.17 Mover `docs/28-futuro-boardgame-rating.md` → `docs/research/future/boardgame-rating.md`
- [ ] 3.2.18 Mover `docs/29-futuro-books-wishlist-acquisitiondate.md` → `docs/research/future/books-wishlist-acquisitiondate.md`

**Verificación:** Los archivos originales ya no existen y los nuevos están en su ubicación.

### 3.3 Convertir archivos de fase a stubs

- [ ] 3.3.1 Reescribir `docs/13-fase-1-libros.md` como stub: "# Fase 1: Libros (histórico)\n\nVer especificación actualizada: [specs/collection/books](./specs/collection/books/)"
- [ ] 3.3.2 Reescribir `docs/14-fase-2-juegos.md` como stub análogo
- [ ] 3.3.3 Reescribir `docs/15-fase-3-juegos-mesa.md` como stub análogo
- [ ] 3.3.4 Reescribir `docs/16-fase-4-magic.md` como stub análogo
- [ ] 3.3.5 Reescribir `docs/17-fase-5-movieshows.md` como stub análogo
- [ ] 3.3.6 Reescribir `docs/19-fase-8-autenticacion.md` como stub análogo
- [ ] 3.3.7 Reescribir `docs/20-fase-9-colecciones-personales.md` como stub análogo

**Verificación:** Los archivos contienen solo el texto de redirect.

### 3.4 Mover/integrees documentación técnica

- [ ] 3.4.1 Mover `docs/02-arquitectura/README.md` → `docs/architecture/overview.md`
- [ ] 3.4.2 Mover `docs/02-arquitectura/02.1-backend.md` → integrar contenido en `docs/architecture/backend.md`
- [ ] 3.4.3 Mover `docs/02-arquitectura/02.2-frontend.md` → integrar contenido en `docs/architecture/frontend.md`
- [ ] 3.4.4 Mover `docs/07-backend/README.md` → integrar contenido relevante en `docs/architecture/backend.md`
- [ ] 3.4.5 Mover `docs/07-backend/07.1-servicios.md` → integrar en `docs/architecture/backend.md` (sección servicios)
- [ ] 3.4.6 Mover `docs/07-backend/07.2-persistencia.md` → integrar en `docs/architecture/backend.md` + `docs/data-model/schema.md`
- [ ] 3.4.7 Mover `docs/06-frontend/README.md` → integrar en `docs/architecture/frontend.md`
- [ ] 3.4.8 Mover `docs/06-frontend/06.1-componentes.md` → integrar en `docs/architecture/frontend.md`
- [ ] 3.4.9 Mover `docs/06-frontend/06.2-navegacion.md` → integrar en `docs/architecture/frontend.md`
- [ ] 3.4.10 Mover `docs/08-deploy.md` → `docs/architecture/deployment.md`
- [ ] 3.4.11 Mover `docs/09-testing.md` → `docs/testing.md`
- [ ] 3.4.12 Mover `docs/03-base-de-datos/README.md` → integrar en `docs/data-model/schema.md`
- [ ] 3.4.13 Mover `docs/03-base-de-datos/03.1-tablas.md` → integrar en `docs/data-model/schema.md`
- [ ] 3.4.14 Mover `docs/03-base-de-datos/03.2-relaciones.md` → `docs/data-model/relationships.md`
- [ ] 3.4.15 Integrar `docs/18-fase-7-caffeine-search.md` → sección de caché en `docs/architecture/backend.md`

**Verificación:** Los archivos originales ya no existen (o son stubs) y el contenido está en su nuevo destino.

---

## 4. Actualizar índices y referencias

### 4.1 Actualizar `docs/00-home.md`

- [ ] 4.1.1 Reescribir tabla de navegación con nuevos paths
- [ ] 4.1.2 Actualizar tabla "Estado del Proyecto" con estado real de Fase 9 (según decisión 0.1)
- [ ] 4.1.3 Eliminar referencias a archivos que ya no existen

**Verificación:** `docs/00-home.md` enlaza correctamente a todos los nuevos documentos.

### 4.2 Actualizar `docs/testing.md`

- [ ] 4.2.1 Actualizar referencias a archivos que se movieron (si las hay)
- [ ] 4.2.2 Verificar número de tests (según decisión 0.9)

**Verificación:** `docs/testing.md` es consistente con la nueva estructura.

### 4.3 Actualizar `AGENTS.md`

- [ ] 4.3.1 Actualizar la tabla "Documentación Structure" con la nueva organización
- [ ] 4.3.2 Actualizar las convenciones si es necesario

**Verificación:** `AGENTS.md` refleja la nueva estructura de directorios.

---

## 5. Documentación pendiente de decisión

### 5.1 `docs/01-requisitos.md`

- [ ] 5.1.1 Mantener `docs/01-requisitos.md` en su ubicación actual. Se actualizará la tabla de navegación en `00-home.md` para enlazarlo.
- [ ] 5.1.2 Verificar que los requisitos funcionales en `01-requisitos.md` sean consistentes con las specs creadas (si no lo son, actualizar `01-requisitos.md` o las specs).

### 5.2 `docs/30-cambio-implementado-game-platforms.md`

- [ ] 5.2.1 Decidir y ejecutar según respuesta a Q7: mover a research, eliminar o mantener

### 5.3 `docs/12-changelog.md`

- [ ] 5.3.1 Decidir y ejecutar según respuesta a Q5: mantener en ubicación actual o mover a `docs/history/`

---

## 6. Verificación final

- [ ] 6.1 Verificar que `docs/00-home.md` enlaza a todos los documentos existentes
- [ ] 6.2 Verificar que no hay enlaces rotos internos (documentos que enlazan a archivos que ya no existen)
- [ ] 6.3 Verificar que las 9 specs + api-contracts existen y son autocontenidas
- [ ] 6.4 Verificar que `docs/architecture/backend.md` es la única fuente canónica de la estructura hexagonal
- [ ] 6.5 Verificar que `docs/data-model/schema.md` es la única fuente canónica del modelo de datos
- [ ] 6.6 Confirmar que no se borró contenido sin migrar (verificar que cada archivo eliminado tiene su destino o se decidió eliminar intencionalmente)

**Verificación:** Ejecutar `find docs -name "*.md" | sort` y comparar con la lista esperada en `design.md`.

---

## Notas de implementación

- **Orden de ejecución:** Completar todas las tareas del grupo 0 antes de empezar el grupo 1. Los grupos 1-3 son independientes entre sí, pero el grupo 4 debe ejecutarse después del grupo 3.
- **No se debe tocar código:** Este cambio es solo documentación. No modificar `backend-collection` ni `frontend-collection`.
- **Respetar contenido existente:** Al integrar documentación en nuevos archivos (ej: `architecture/backend.md`), no omitir información relevante del documento original.
- **Consistencia de estado:** Si la Fase 9 está realmente implementada, las specs de user-preferences deben reflejar eso. Si no, deben marcarlo como 📋 planificado.
