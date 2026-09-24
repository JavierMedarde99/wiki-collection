# Proposal — Reorganización de Wiki-Collection como Base de Conocimiento Spec-Driven

## Why

El repositorio `wiki-collection` concentra en `docs/` tres capas de documentación superpuestas (descripción del sistema, planes de implementación por fases y referencia técnica por capa) sin separación clara entre **qué hace el sistema** (especificaciones), **cómo está construido** (arquitectura/referencia) y **investigación/contexto** (decisiones, APIs externas, futuros). Esta mezcla dificulta:

- Encontrar la especificación de un endpoint o regla de negocio sin navegar entre 9 fases + README de API + tablas de datos.
- Distinguir documentación canónica de documentación duplicada o desactualizada (ej: `04-autenticacion.md` diz "no implementado" pero la fase 8 ya existe; el endpoint de ISBN aparece como existente en `05-api/README.md` y como pendiente en `22-futuro-barcode-scanner.md`).
- Añadir nuevas funcionalidades sin un lugar claro donde documentar el contrato antes de implementar.

Convertir el wiki en una base de conocimiento **spec-driven** con Directorios separados por propósito (`docs/research/` para contexto e investigación, `docs/specs/` para especificaciones de comportamiento/contractos, `docs/architecture/` para estructura técnica) permite:

- Especificación autocontenida por capability (colección, auth, preferencias, imágenes).
- Referencia de arquitectura y modelo de datos como documentación secundaria que apunta a las specs.
- Investigación (APIs externas, futuros, decisiones técnicas) separada de lo que el sistema hace hoy.

## What Changes

### Nuevo esquema de directorios (BREAKING para enlaces existentes)

```
docs/
├── 00-home.md              # Índice actualizado — todo lo demás en su nuevo lugar
├── research/               # Contexto, decisiones, investigación (no especificaciones)
│   ├── decisions/          # ADR consolidados
│   ├── known-issues/       # Problemas conocidos
│   ├── external-apis/      # Documentación de APIs externas (contratos con terceros)
│   └── future/             # Investigación de funcionalidades futuras (no implementadas)
│
├── specs/                  # Especificaciones de comportamiento e interfaces
│   ├── collection/         # Por cada colección: spec.md + api-contract.md
│   │   ├── books/
│   │   ├── games/
│   │   ├── board-games/
│   │   ├── magic-cards/
│   │   ├── decks/
│   │   └── movie-shows/
│   ├── auth/
│   ├── user-preferences/
│   └── image-storage/
│
├── architecture/           # Estructura técnica del sistema
│   ├── overview.md
│   ├── backend.md
│   ├── frontend.md
│   └── deployment.md
│
└── data-model/             # Modelo de datos (esquema + relaciones)
    ├── schema.md
    └── relationships.md
```

### Cambios específicos

- **Consolidación de estructura hexagonal:** Las 9 fases + `07-backend/README.md` + `02-arquitectura/02.1-backend.md` describen la misma estructura de paquetes. Se mantiene una única versión canónica en `docs/architecture/backend.md`. Las fases individuales se convierten en specs que enlazan a la arquitectura en lugar de repetirla.
- **Separación de API contracts:** `docs/05-api/README.md` se desarma en `api-contract.md` por cada capability. Cada spec es autocontenido: reglas de negocio + endpoints + request/response + códigos de error + filtros.
- **Eliminación de documentación obsoleta:** `04-autenticacion.md`, `05-api/05.1-usuarios.md`, `05-api/05.2-autenticacion.md` se absorben en `docs/specs/auth/` (su contenido "no implementado" es falso). Se conserva el historial en `docs/research/` si se decide.
- **Investigación externa consolidada:** Los 8 archivos `externas-*.md` + `steam-api-key-guide.md` se unifican en `docs/research/external-apis/` con nombres descriptivos.
- **Futuros investigados separados:** Los 8 documentos `22`–`29` pasan a `docs/research/future/` (están en estado "investigación/no implementado" o "completado pero documentado como futuro").

### Archivos que NO se tocan (solo se reorganizan)

- `docs/13-fase-1-libros.md` → contenido migrado a `docs/specs/collection/books/spec.md` pero el archivo original se conserva hasta que se verifica que la spec cubre lo mismo.
- `docs/14-fase-2-juegos.md` → idem para games.
- `docs/15-fase-3-juegos-mesa.md` → idem para board-games.
- `docs/16-fase-4-magic.md` → idem para magic-cards.
- `docs/17-fase-5-movieshows.md` → idem para movie-shows.
- `docs/19-fase-8-autenticacion.md` → idem para auth.
- `docs/20-fase-9-colecciones-personales.md` → idem para user-preferences.

## Capabilities

### New Capabilities

- `collection/books`: Especificación de la colección de libros — reglas de negocio, estados, campos, endpoints CRUD + búsqueda (Google Books), filtros, visibilidad.
- `collection/games`: Especificación de la colección de videojuegos — reglas, estados, campos, endpoints CRUD + búsqueda (RAWG + FreeToGame fallback), logros Steam, filtros, visibilidad.
- `collection/board-games`: Especificación de la colección de juegos de mesa — reglas, estados, campos, endpoints CRUD + búsqueda (BGG XML), filtros, visibilidad.
- `collection/magic-cards`: Especificación de la colección de cartas Magic — reglas, estados, campos, endpoints de listado/detalle/eliminación + añadir desde Scryfall, búsqueda, filtros, visibilidad. Nota: backend sin save/update genérico.
- `collection/decks`: Especificación de mazos Commander — reglas, estados del mazo (DRAFT/COMPLETE/INVALID), endpoints CRUD + gestión de cartas, validación de reglas, filtros, visibilidad.
- `collection/movie-shows`: Especificación de películas/series — reglas, estados, campos, endpoints CRUD + búsqueda (TMDB), filtros por tipo/estado, visibilidad.
- `auth`: Especificación de autenticación — flujos de registro/login/refresh, JWT (access 15min + refresh 7días), reglas de visibilidad por rol, gestión de tokens.
- `user-preferences`: Especificación de preferencias de colección — colecciones activas, visibilidad pública/privada, perfiles públicos, endpoints de preferencias y perfiles.
- `image-storage`: Especificación de almacenamiento de imágenes — subida a Catbox, eliminación, validaciones, endpoints.

### Modified Capabilities

- Ninguna. El cambio es puramente organizativo/documental. El comportamiento del sistema no cambia. Las fases existentes (1-9) ya implementan lo que describen; la reorganización no altera los requisitos, solo Cambia dónde se documentan.

## Impact

- **Enlaces rotos:** Cualquier enlace interno que apunte a las rutas antiguas de `docs/` necesita actualización. `00-home.md` es el índice principal; todos los demás documentos que enlenezan a las rutas movidas deben actualizarse.
- **Documentación externa:** El archivo `AGENTS.md` (instructions para agentes) referencia la estructura actual de `docs/`. Necesita actualización para reflejar la nueva organización.
- **Repositorios de código (backend-collection, frontend-collection):** No se ven afectados. Este repo solo contiene documentación.
- **Testing:** `docs/09-testing.md` mantiene su contenido (lista de tests por feature). Puede enlazar a las specs respectivas en lugar de repetir información.
- **Changelog:** `docs/12-changelog.md` se mantiene como registro histórico del proyecto. No se modifica en esta migración.

---

## Decisiones Tomadas

| # | Decisión | Valor |
|---|----------|-------|
| D1 | Estado de la Fase 9 | **Completada** — se corrige la inconsistencia del documento original que marcaba "📋 PLANIFICADA" internamente. La spec de `user-preferences` reflejará el estado real. |
| D2 | Archivos de fase (13-21) | **Stubs** — se convierten en archivos de redirect con enlace a la spec correspondiente. El contenido histórico se preserva en la nueva ubicación de las specs. |
| D3 | `docs/01-requisitos.md` | **Se mantiene** — en su ubicación actual. La tabla de navegación en `00-home.md` se actualiza para enlazarlo. Se verifica consistencia con las specs creadas. |

---

## Documentación existente clasificada

---

## Documentación existente clasificada

### DOC — Descripción del sistema (hoy)

| Archivo | Clasificación | Ubicación propuesta |
|---------|---------------|---------------------|
| `docs/02-arquitectura/README.md` | DOC | `docs/architecture/overview.md` |
| `docs/02-arquitectura/02.1-backend.md` | DOC | `docs/architecture/backend.md` (canónico) |
| `docs/02-arquitectura/02.2-frontend.md` | DOC | `docs/architecture/frontend.md` |
| `docs/06-frontend/README.md` | DOC | `docs/architecture/frontend.md` (parcial) o appendix |
| `docs/06-frontend/06.1-componentes.md` | DOC | `docs/architecture/frontend.md` (appendix) |
| `docs/06-frontend/06.2-navegacion.md` | DOC | `docs/architecture/frontend.md` (appendix) |
| `docs/07-backend/README.md` | DOC | `docs/architecture/backend.md` (parcial, descartar redundancia) |
| `docs/07-backend/07.1-servicios.md` | DOC | `docs/architecture/backend.md` (appendix) |
| `docs/07-backend/07.2-persistencia.md` | DOC | `docs/data-model/schema.md` + `docs/architecture/backend.md` |
| `docs/08-deploy.md` | DOC | `docs/architecture/deployment.md` |
| `docs/09-testing.md` | DOC | `docs/testing.md` (mantener, enlaza a specs) |
| `docs/03-base-de-datos/README.md` | DOC | `docs/data-model/schema.md` |
| `docs/03-base-de-datos/03.1-tablas.md` | DOC | `docs/data-model/schema.md` (canónico) |
| `docs/03-base-de-datos/03.2-relaciones.md` | DOC | `docs/data-model/relationships.md` |

### SPEC — Especificaciones de comportamiento (hoy en fases)

| Archivo | Clasificación | Ubicación propuesta |
|---------|---------------|---------------------|
| `docs/13-fase-1-libros.md` | SPEC | `docs/specs/collection/books/spec.md` |
| `docs/14-fase-2-juegos.md` | SPEC | `docs/specs/collection/games/spec.md` |
| `docs/15-fase-3-juegos-mesa.md` | SPEC | `docs/specs/collection/board-games/spec.md` |
| `docs/16-fase-4-magic.md` | SPEC | `docs/specs/collection/magic-cards/spec.md` |
| `docs/17-fase-5-movieshows.md` | SPEC | `docs/specs/collection/movie-shows/spec.md` |
| `docs/20-fase-9-colecciones-personales.md` | SPEC (📋 planificada) | `docs/specs/user-preferences/spec.md` |

### API CONTRACTS — Interfaces documentadas

| Archivo | Clasificación | Ubicación propuesta |
|---------|---------------|---------------------|
| `docs/05-api/README.md` | SPEC (contrato global) | Desarmado en `specs/*/api-contract.md` |
| `docs/05-api/05.1-usuarios.md` | SPEC (obsoleto — conflicto con fase 8) | Absorbido en `docs/specs/auth/api-contract.md` |
| `docs/05-api/05.2-autenticacion.md` | SPEC (obsoleto — conflicto con fase 8) | Absorbido en `docs/specs/auth/api-contract.md` |

### RESEARCH — Contexto e investigación

| Archivo | Clasificación | Ubicación propuesta |
|---------|---------------|---------------------|
| `docs/05-api/externas/externas-books.md` | RESEARCH | `docs/research/external-apis/google-books.md` |
| `docs/05-api/externas/externas-videogames.md` | RESEARCH | `docs/research/external-apis/rawg.md` + `freetogame.md` |
| `docs/05-api/externas/externas-steam.md` | RESEARCH | `docs/research/external-apis/steam.md` |
| `docs/05-api/externas/externas-boardgames.md` | RESEARCH | `docs/research/external-apis/boardgamegeek.md` |
| `docs/05-api/externas/externas-magic.md` | RESEARCH | `docs/research/external-apis/scryfall.md` |
| `docs/05-api/externas/externas-movies.md` | RESEARCH | `docs/research/external-apis/tmdb.md` |
| `docs/05-api/externas/externas-image-hosting.md` | RESEARCH | `docs/research/external-apis/catbox.md` |
| `docs/05-api/externas/steam-api-key-guide.md` | RESEARCH | `docs/research/external-apis/steam-api-key-guide.md` |
| `docs/10-decisiones-tecnicas.md` | RESEARCH | `docs/research/decisions/adr.md` |
| `docs/11-problemas-conocidos.md` | RESEARCH | `docs/research/known-issues/issues.md` |

### FUTURE — Investigación de funcionalidades futuras

| Archivo | Clasificación | Ubicación propuesta |
|---------|---------------|---------------------|
| `docs/22-futuro-barcode-scanner.md` | RESEARCH (completado) | `docs/research/future/barcode-scanner.md` |
| `docs/23-futuro-deck-import.txt.md` | RESEARCH | `docs/research/future/deck-import-text.md` |
| `docs/24-futuro-reading-progress.md` | RESEARCH | `docs/research/future/reading-progress.md` |
| `docs/25-futuro-streaming-platforms.md` | RESEARCH | `docs/research/future/streaming-platforms.md` |
| `docs/26-futuro-generos-colecciones.md` | RESEARCH | `docs/research/future/genres-analysis.md` |
| `docs/27-futuro-generos-books-games-boardgames.md` | RESEARCH | `docs/research/future/genres-implementation.md` |
| `docs/28-futuro-boardgame-rating.md` | RESEARCH | `docs/research/future/boardgame-rating.md` |
| `docs/29-futuro-books-wishlist-acquisitiondate.md` | RESEARCH | `docs/research/future/books-wishlist-acquisitiondate.md` |

### OBSOLETE — Información desactualizada

| Archivo | Motivo |
|---------|--------|
| `docs/04-autenticacion.md` | Etiqueta "no implementado" contradice fase 8 completada |
| `docs/05-api/05.1-usuarios.md` | "No implementado, planificado" pero endpoints existen |
| `docs/05-api/05.2-autenticacion.md` | "No implementado, planificado" pero endpoints existen |
| `docs/02-arquitectura/02.1-backend.md` (parte) | Versiones anteriores de algunos conteos |
| `docs/08-deploy.md` (parte) | Contiene texto en coreano ("동일한 지역 선택") mezclado con español |

### DUPLICATE — Información duplicada

| Contenido | Archivos duplicados | Canónico propuesto |
|-----------|---------------------|--------------------|
| Estructura hexagonal (paquetes, clases por capa) | Fases 1-5, 8, 9 + `07-backend/README.md` + `02-arquitectura/02.1-backend.md` | `docs/architecture/backend.md` |
| Mapeo campos APIs externas | `05-api/README.md` (tabla) + `externas-*.md` (detalle) + fases (fragmentos) | `docs/research/external-apis/*.md` |
| Lista de tests por feature | `09-testing.md` + fases individuales (checklist) | `docs/09-testing.md` |
| Criterios de aceptación | `01-requisitos.md` + fases individuales + `05-api/README.md` (implícito) | Fases → specs (canónico por capability) |
| Modelo de datos (campos, tipos) | `03.1-tablas.md` + `07-backend/07.2-persistencia.md` + fases | `docs/data-model/schema.md` |

### NEEDS_REVIEW — Inconsistencias a resolver antes de migrar

1. **ISBN endpoint:** ¿Existe `GET /api/v1/books/search?isbn=...` o no? `05-api/README.md` lo marca ✅, `22-futuro-barcode-scanner.md` lo describe como pendiente. Verificar en código antes de migrar.

2. **Fase 9 completada vs planificada:** `00-home.md` marca ✅ completada; `20-fase-9-colecciones-personales.md` marca 📋 planificada con todos los checks pendientes. Decidir cuál es el estado real antes de migrar.

3. **Estado de tests:** `AGENTS.md` dice 56 tests; `09-testing.md` dice 68. Verificar número real.

4. **Steam client:** ¿`SteamCatalogueClient` y `SteamAchievementsClient` coexisten o es uno solo? `07-backend/README.md` lista ambos, `07.1-servicios.md` solo describe `GameAchievementsService`.

5. **ADR-014+ truncado:** `10-decisiones-tecnicas.md` se corta en línea 200 sin completar AD-014.

6. **GamePlatform campo:** ¿Existe campo `genre: String` en `Game.java`? `27-futuro-generos-books-games-boardgames.md` lo afirma pero no está documentado en `03.1-tablas.md`.

---

## Estructura propuesta detallada

```
docs/
├── 00-home.md                    # Índice — enlaces a nueva estructura
├── testing.md                    # (ex 09-testing.md) — lista de tests por feature
│
├── research/
│   ├── decisions/
│   │   └── adr.md               # ADR consolidados (AD-001 a AD-015+)
│   ├── known-issues/
│   │   └── issues.md            # Problemas conocidos
│   ├── external-apis/
│   │   ├── google-books.md
│   │   ├── rawg.md
│   │   ├── freetogame.md
│   │   ├── boardgamegeek.md
│   │   ├── scryfall.md
│   │   ├── tmdb.md
│   │   ├── steam.md
│   │   ├── catbox.md
│   │   └── steam-api-key-guide.md
│   └── future/
│       ├── barcode-scanner.md
│       ├── deck-import-text.md
│       ├── reading-progress.md
│       ├── streaming-platforms.md
│       ├── genres-analysis.md
│       ├── genres-implementation.md
│       ├── boardgame-rating.md
│       └── books-wishlist-acquisitiondate.md
│
├── specs/
│   ├── collection/
│   │   ├── books/
│   │   │   ├── spec.md           # Reglas, estados, campos, visibilidad
│   │   │   └── api-contract.md  # Endpoints, DTOs, filtros, errores
│   │   ├── games/
│   │   │   ├── spec.md
│   │   │   └── api-contract.md
│   │   ├── board-games/
│   │   │   ├── spec.md
│   │   │   └── api-contract.md
│   │   ├── magic-cards/
│   │   │   ├── spec.md
│   │   │   └── api-contract.md
│   │   ├── decks/
│   │   │   ├── spec.md
│   │   │   └── api-contract.md
│   │   └── movie-shows/
│   │       ├── spec.md
│   │       └── api-contract.md
│   ├── auth/
│   │   ├── spec.md               # JWT, flujos, roles, reglas de visibilidad
│   │   └── api-contract.md      # /api/v1/auth/* endpoints
│   ├── user-preferences/
│   │   ├── spec.md               # Active collections, visibility, public profiles
│   │   └── api-contract.md      # /api/v1/preferences, /api/v1/users/{username}/*
│   └── image-storage/
│       ├── spec.md
│       └── api-contract.md      # POST /upload, DELETE /{filename}
│
├── architecture/
│   ├── overview.md               # Diagrama + principios (ex 02-arquitectura/README.md)
│   ├── backend.md                # Hexagonal, paquetes, puertos, adaptadores (canónico)
│   ├── frontend.md               # Stack React, estructura, components, rutas
│   └── deployment.md            # Atlas + Render + Vercel (ex 08-deploy.md)
│
└── data-model/
    ├── schema.md                 # Tablas/colecciones, campos, tipos, índices (ex 03.1-tablas.md)
    └── relationships.md          # Relaciones entre entidades (ex 03.2-relaciones.md)
```

---

## Mapping archivo actual → nuevo archivo

### docs/actual → docs/nuevo

| Archivo actual | Nuevo archivo | Acción |
|---------------|---------------|--------|
| `docs/00-home.md` | `docs/00-home.md` | Actualizar enlaces |
| `docs/01-requisitos.md` | `docs/00-home.md` (sección Requisitos) o eliminar | Contenido ya cubierto por specs |
| `docs/02-arquitectura/README.md` | `docs/architecture/overview.md` | Mover |
| `docs/02-arquitectura/02.1-backend.md` | `docs/architecture/backend.md` | Mover (canónico) |
| `docs/02-arquitectura/02.2-frontend.md` | `docs/architecture/frontend.md` | Mover (renombrado) |
| `docs/03-base-de-datos/README.md` | `docs/data-model/schema.md` | Integrar |
| `docs/03-base-de-datos/03.1-tablas.md` | `docs/data-model/schema.md` | Integrar (canónico) |
| `docs/03-base-de-datos/03.2-relaciones.md` | `docs/data-model/relationships.md` | Mover |
| `docs/04-autenticacion.md` | Eliminar (contenido ya en fase 8) | Descartar |
| `docs/05-api/README.md` | Se desarma en specs/*/api-contract.md | Desarmar |
| `docs/05-api/05.1-usuarios.md` | Absorbido en `docs/specs/auth/api-contract.md` | Eliminar |
| `docs/05-api/05.2-autenticacion.md` | Absorbido en `docs/specs/auth/api-contract.md` | Eliminar |
| `docs/05-api/externas/externas-books.md` | `docs/research/external-apis/google-books.md` | Mover |
| `docs/05-api/externas/externas-videogames.md` | `docs/research/external-apis/rawg.md` | Mover |
| `docs/05-api/externas/externas-steam.md` | `docs/research/external-apis/steam.md` | Mover |
| `docs/05-api/externas/externas-boardgames.md` | `docs/research/external-apis/boardgamegeek.md` | Mover |
| `docs/05-api/externas/externas-magic.md` | `docs/research/external-apis/scryfall.md` | Mover |
| `docs/05-api/externas/externas-movies.md` | `docs/research/external-apis/tmdb.md` | Mover |
| `docs/05-api/externas/externas-image-hosting.md` | `docs/research/external-apis/catbox.md` | Mover |
| `docs/05-api/externas/steam-api-key-guide.md` | `docs/research/external-apis/steam-api-key-guide.md` | Mover |
| `docs/06-frontend/README.md` | `docs/architecture/frontend.md` | Integrar |
| `docs/06-frontend/06.1-componentes.md` | `docs/architecture/frontend.md` (appendix) | Integrar |
| `docs/06-frontend/06.2-navegacion.md` | `docs/architecture/frontend.md` (appendix) | Integrar |
| `docs/07-backend/README.md` | `docs/architecture/backend.md` | Integrar (parcial, descartar redundancia) |
| `docs/07-backend/07.1-servicios.md` | `docs/architecture/backend.md` (appendix) | Integrar |
| `docs/07-backend/07.2-persistencia.md` | `docs/data-model/schema.md` + `docs/architecture/backend.md` | Integrar |
| `docs/08-deploy.md` | `docs/architecture/deployment.md` | Mover |
| `docs/09-testing.md` | `docs/testing.md` | Mover |
| `docs/10-decisiones-tecnicas.md` | `docs/research/decisions/adr.md` | Mover |
| `docs/11-problemas-conocidos.md` | `docs/research/known-issues/issues.md` | Mover |
| `docs/12-changelog.md` | `docs/12-changelog.md` | Mantener ( historial) |
| `docs/13-fase-1-libros.md` | `docs/specs/collection/books/spec.md` | Mover (contenido) |
| `docs/14-fase-2-juegos.md` | `docs/specs/collection/games/spec.md` | Mover (contenido) |
| `docs/15-fase-3-juegos-mesa.md` | `docs/specs/collection/board-games/spec.md` | Mover (contenido) |
| `docs/16-fase-4-magic.md` | `docs/specs/collection/magic-cards/spec.md` | Mover (contenido) |
| `docs/17-fase-5-movieshows.md` | `docs/specs/collection/movie-shows/spec.md` | Mover (contenido) |
| `docs/18-fase-7-caffeine-search.md` | `docs/architecture/backend.md` (sección Cache) | Integrar |
| `docs/19-fase-8-autenticacion.md` | `docs/specs/auth/spec.md` | Mover (contenido) |
| `docs/20-fase-9-colecciones-personales.md` | `docs/specs/user-preferences/spec.md` | Mover (contenido) |
| `docs/22-futuro-barcode-scanner.md` | `docs/research/future/barcode-scanner.md` | Mover |
| `docs/23-futuro-deck-import.txt.md` | `docs/research/future/deck-import-text.md` | Mover |
| `docs/24-futuro-reading-progress.md` | `docs/research/future/reading-progress.md` | Mover |
| `docs/25-futuro-streaming-platforms.md` | `docs/research/future/streaming-platforms.md` | Mover |
| `docs/26-futuro-generos-colecciones.md` | `docs/research/future/genres-analysis.md` | Mover |
| `docs/27-futuro-generos-books-games-boardgames.md` | `docs/research/future/genres-implementation.md` | Mover |
| `docs/28-futuro-boardgame-rating.md` | `docs/research/future/boardgame-rating.md` | Mover |
| `docs/29-futuro-books-wishlist-acquisitiondate.md` | `docs/research/future/books-wishlist-acquisitiondate.md` | Mover |

---

## Specs que deberían crearse

Cada capability recibe un par de archivos: `spec.md` (reglas de negocio, estados, campos, visibilidad) y `api-contract.md` (endpoints, DTOs, filtros, errores).

### 1. `specs/collection/books/`

**spec.md:**
- Objetivo: colección de libros con búsqueda en Google Books
- Estados: TO_READ, READING, COMPLETED (+ WISHLIST si está implementado)
- Tipos: MANGA, NOVEL, GRAPHIC_NOVEL
- Campos: id, externalId, title, descripcion, author, pages, type, state, comment, start (0-5), startDate, endDate, frontpage, isbn?, pagesRead?
- Visibilidad: notas y valoración privadas (solo dueño)
- Regla: `pagesRead <= pages` si pages definido
- Regla: ISBN se limpia de guiones/espacios
- Regla: varias ediciones con mismo ISBN permitidas

**api-contract.md:**
- Endpoints: GET /books, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=..., GET /search?isbn=..., GET /search?author=...
- Filtros: ?state=, ?type=, ?owner= (mine/other/all)
- Response shape: BookResponse
- Error codes: 404 (not found), 409 (conflict)
- Paginación: Spring Data (page, size, sort)

### 2. `specs/collection/games/`

**spec.md:**
- Objetivo: colección de videojuegos con búsqueda en RAWG + FreeToGame fallback
- Estados: PLAYING, COMPLETED, WISHLIST, ABANDONED
- Plataformas: PC, PS2, PS3, WII_U, SWITCH
- Campos: id, externalId, title, platform, thumbnailUrl, status, userRating (1-5), comment, dateAdded, dateCompleted, externalSource, steamAppId
- Logros Steam: endpoint separado GET /{gameId}/achievements?steamId=...
- Estrategia de búsqueda: RAWG primero → FreeToGame fallback

**api-contract.md:**
- Endpoints: GET /games, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=..., GET /{gameId}/achievements
- Filtros: ?platform=, ?status=, ?owner=
- Parámetro `obtainPlatinum` (no documentado en README actual, verificar)

### 3. `specs/collection/board-games/`

**spec.md:**
- Objetivo: colección de juegos de mesa con búsqueda en BGG XML API
- Estados: OWNED, WISHLIST (solo estos dos, no PREVIOUSLY_OWNED ni FOR_TRADE)
- Campos: id, title, description, yearPublished, minPlayers, maxPlayers, minPlaytime, maxPlaytime, publisher, designers[], categories[], mechanics?, imageUrl, thumbnailUrl, bggRating, bggId, status, notes, dateAdded
- BGG XML: formato XML, parseo con Jackson, reintentos 2s por 202 Accepted

**api-contract.md:**
- Endpoints: GET /boardgames, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=...
- Filtros: ?status=, ?owner=

### 4. `specs/collection/magic-cards/`

**spec.md:**
- Objetivo: colección de cartas Magic desde Scryfall
- Condiciones: MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR
- Idiomas: ENGLISH, SPANISH, FRENCH, GERMAN, ITALIAN, PORTUGUESE, JAPANESE, CHINESE
- Campos: id, scryfallId, oracleId, name, language, releaseDate, manaCost, convertedManaCost, type, text, power, toughness, loyalty, colors[], colorIdentity[], keywords[], rarity, setCode, setName, artist, frame, borderColor, layout, legalities, priceUsd, priceEur, imageUrl, imageLargeUrl, artCropUrl, condition, isFoil, quantity, notes, dateAdded
- Regla: backend sin save/update genérico. Solo `addFromScryfall` + delete.

**api-contract.md:**
- Endpoints: GET /magic, GET /{id}, POST /scryfall/{scryfallId}, DELETE /{id}, GET /search?name=..., GET /commanders?colors=...
- Filtros: ?name=, ?rarity=, ?color=, ?type=, ?owner=

### 5. `specs/collection/decks/`

**spec.md:**
- Objetivo: mazos Commander con validación de reglas
- Estados del mazo: DRAFT, COMPLETE, INVALID
- Campos: id, name, description, commander, commanderColors[], cards[] (DeckCard), createdAt, updatedAt
- DeckCard: cardName, quantity, inCollection, isProxy, manaCost, typeLine, colorIdentity[], imageUrl, scryfallId
- Regla: validación Commander (1 comandante, ≤100 cartas, ≤3 copias por carta no básica, etc.)

**api-contract.md:**
- Endpoints: GET /decks, GET /{id}, POST, PUT /{id}, DELETE /{id}, POST /{id}/cards, DELETE /{id}/cards/{scryfallId}, GET /{id}/status
- Filtros: ?name=, ?owner=
- Request: DeckCardRequest (scryfallId, quantity)
- Response: DeckStatusResponse (status, message)

### 6. `specs/collection/movie-shows/`

**spec.md:**
- Objetivo: películas/series con búsqueda en TMDB
- Tipos: MOVIE, TV
- Estados: WATCHING, WATCHED, PLAN_TO_WATCH (sin WISHLIST)
- Campos: id, externalId (unique), title, overview, releaseDate, posterUrl, backdropUrl, voteAverage, mediaType, status, userRating, comment, dateAdded, dateCompleted, externalSource, createdAt, updatedAt
- Regla: externalId único para evitar duplicados
- Idioma: es-ES

**api-contract.md:**
- Endpoints: GET /movieshows, GET /{id}, POST, PUT /{id}, DELETE /{id}, GET /search?name=...&mediaType=...
- Filtros: ?status=, ?mediaType=, ?owner=

### 7. `specs/auth/`

**spec.md:**
- Flujos: register, login, refresh token
- JWT: access token (15 min) + refresh token (7 días)
- Passwords: BCrypt
- Roles: USER, ADMIN (admin por defecto vía env vars)
- Reglas de visibilidad:
  - Anónimo: ver colecciones públicas de cualquier usuario
  - Usuario: ver sus propias notas/valoraciones (privadas)
  - Admin: ver/gestionar todos los datos
- ownerId = "system" para datos pre-autenticación
- Refresh tokens en localStorage (no en MongoDB)

**api-contract.md:**
- Endpoints: POST /auth/register, POST /auth/login, POST /auth/refresh, GET /auth/me
- Request/response: RegisterRequest, LoginRequest, RefreshTokenRequest, AuthResponse, UserResponse

### 8. `specs/user-preferences/`

**spec.md:**
- Colecciones activas: books, games, boardgames, magic, decks, movieshows (todas activas por defecto)
- Visibilidad: PUBLIC o PRIVATE por colección (todas PUBLIC por defecto)
- Perfil público: /perfil/{username} muestra username, displayName, avatarUrl, bio, colecciones públicas con contadores
- Colecciones inactivas: se ocultan en UI pero no se borran de BD
- Lazy creation: si no tiene preferences, se crean on-demand
- Backwards compatible: endpoints sin parámetro owner = owner=mine

**api-contract.md:**
- Preferencias: GET /preferences, PUT /preferences, PATCH /preferences/active-collections, PATCH /preferences/collection-visibility, GET /preferences/active-collections
- Perfiles: GET /users/{username}, GET /users/{username}/books, /games, /boardgames, /magic, /decks, /movieshows
- Response shapes: UserPreferencesResponse, PublicProfileResponse, PublicCollectionSummary

### 9. `specs/image-storage/`

**spec.md:**
- Almacenamiento en Catbox.moe (no filesystem local)
- Imágenes de perfil y items subidas por usuario
- Imágenes de APIs externas (Google Books, Scryfall, RAWG, TMDB) usan URLs originales

**api-contract.md:**
- Endpoints: POST /images/upload, DELETE /images/{filename}
- Validaciones: 5MB máximo, MIME: image/jpeg, image/png, image/gif, image/webp, magic bytes
- Configuración: catbox.api.base-url, catbox.userhash (solo para eliminar)
- Response: ImageResponse (url, filename)

---

## Información duplicada (resumen)

1. **Estructura hexagonal × 11:** Fases 1-5, 8, 9 (8 archivos) + `07-backend/README.md` + `02-arquitectura/02.1-backend.md`. Canónico: `docs/architecture/backend.md`.

2. **Mapeo campos APIs externas × 3:** `05-api/README.md` + `externas-*.md` (8 archivos) + fases (fragmentos). Canónico: `docs/research/external-apis/*.md`.

3. **Lista de tests × 2:** `09-testing.md` + fases (checklist). Canónico: `docs/testing.md`.

4. **Criterios de aceptación × 3:** `01-requisitos.md` + fases + `05-api/README.md` (implícito). Canónico: specs por capability.

5. **Modelo de datos × 3:** `03.1-tablas.md` + `07-backend/07.2-persistencia.md` + fases. Canónico: `docs/data-model/schema.md`.

6. **Auth endpoints × 3:** `04-autenticacion.md` + `05.1-usuarios.md` + `05.2-autenticacion.md` + `19-fase-8-autenticacion.md`. Canónico: `docs/specs/auth/`.

---

## Información ambigua (resumen)

| # | Ambigüedad | Impacto |
|---|------------|---------|
| A1 | ¿Endpoint ISBN existe? | `05-api/README.md` ✅ vs `22-futuro-barcode-scanner.md` 📝 |
| A2 | ¿Fase 9 completada? | Índice ✅ vs documento 📋 |
| A3 | ¿Cuántos tests? | 56 (AGENTS.md) vs 68 (09-testing.md) |
| A4 | Steam: ¿1 o 2 clientes? | SteamCatalogueClient + SteamAchievementsClient vs 1 servicio |
| A5 | ¿Game.java tiene `genre: String`? | Documentado en futuro doc pero no en tablas. |
| A6 | ADR-014+ truncado | `10-decisiones-tecnicas.md` se corta sin completar |
| A7 | Fase 7 (Caffeine) no tiene documento de fase numerado | Falta `18-fase-7-caffeine-search.md` que sí existe pero no está en índice de capas hexagonales |
| A8 | `08-deploy.md` tiene texto coreano mezclado | Error de codificación no resuelto |

---

## Preguntas que necesitan decisión humana

### Q1: ¿Qué hacer con la contradicción de Fase 9 (completada vs planificada)?
- **Opción A:** Declarar que está completada en el índice → actualizar `20-fase-9-colecciones-personales.md` con checks completados.
- **Opción B:** Declarar que está planificada → actualizar `00-home.md` para que diga 📋.
- **Opción C:** Dejar como está hasta que se verifique en el código backend cuánto está implementado.
- **Recomendación:** Opción C — verificar en `backend-collection` qué de la Fase 9 está realmente implementado antes de decidir.

### Q2: ¿Qué hacer con los 3 archivos de auth obsoletos (`04`, `05.1`, `05.2`)?
- **Opción A:** Eliminarlos directamente — su contenido está en `19-fase-8-autenticacion.md` y se migrará a `docs/specs/auth/`.
- **Opción B:** Conservarlos en `docs/research/obsolete/` como historial de cómo se pensó inicialmente.
- **Recomendación:** Opción A — son stubs que dicen "no implementado" cuando ya lo está. No aportan valor histórico.

### Q3: ¿Se mantiene `docs/01-requisitos.md` o se integra en el índice?
- **Opción A:** Mantener como documento separado de alto nivel (resumen ejecutivo de requisitos funcionales + no funcionales).
- **Opción B:** Integrar como sección de `docs/00-home.md` y eliminar el archivo.
- **Opción C:** Eliminar porque las specs de cada capability ya cubren los requisitos.
- **Recomendación:** Opción C — las 9 specs nuevas (books, games, board-games, magic-cards, decks, movie-shows, auth, user-preferences, image-storage) ya documentan los requisitos funcionales. Los no funcionales (rendimiento, disponibilidad, escalabilidad, seguridad, mantenibilidad, UX) pueden ir en `docs/architecture/overview.md` o en `docs/00-home.md`.

### Q4: ¿Se conservan los archivos de fase originales (`13`–`21`) después de migrar su contenido a specs?
- **Opción A:** Conservar como historial de cómo se implementó cada fase (con timestamp de migración).
- **Opción B:** Eliminar y mover todo el contenido a las specs.
- **Opción C:** Conservar pero redirigir a las specs (archivo stub con "ver doc correspondiente en specs/...").
- **Recomendación:** Opción C — reducir duplicación pero mantener referencia histórica. Cada archivo de fase se convierte en un redirect breve: "Este documento describe la implementación de [capability]. Ver especificación actual en `specs/collection/X/spec.md`."

### Q5: ¿Se mantiene `docs/12-changelog.md` en su ubicación actual?
- Sí, el changelog es registro histórico del proyecto. No se mueve. Se podría renombrar a `docs/history/changelog.md` para ser consistente con la nueva estructura, pero no es necesario.

### Q6: ¿El archivo `docs/README.md` (en raíz del repo, fuera de docs/) entra en la migración?
- No se leyó. Verificar si existe y si es relevante. Probablemente es inicio del repo con información de índice sobre la wiki. Si existe, se mantiene tal cual.

### Q7: ¿Se consolidan `26-futuro-generos-colecciones.md` y `27-futuro-generos-books-games-boardgames.md`?
- Ambos cubren géneros en colecciones. `26` es análisis (qué APIs tienen género, qué no). `27` es plan de implementación. Se pueden consolidar en un solo documento `docs/research/future/genres.md` con dos secciones claras (análisis + plan).

---

## Estructura de la propuesta OpenSpec

Este documento es el `proposal.md` del cambio `reorganize-wiki-as-spec`. Los siguientes artifacts se crearán en la fase de specs:

```
openspec/changes/reorganize-wiki-as-spec/
├── proposal.md           # Este documento
├── design.md             # Estructura de directorios + reglas de ubicación
├── tasks.md              # Lista de tareas de migración
└── specs/
    ├── collection-books-spec.md
    ├── collection-games-spec.md
    ├── collection-board-games-spec.md
    ├── collection-magic-cards-spec.md
    ├── collection-decks-spec.md
    ├── collection-movie-shows-spec.md
    ├── auth-spec.md
    ├── user-preferences-spec.md
    └── image-storage-spec.md
```

---

## Próximos pasos

1. **Decidir las preguntas Q1-Q7** antes de ejecutar cualquier migración.
2. **Verificar A1-A8** en el código de `backend-collection` y `frontend-collection`.
3. **Crear las specs** (un archivo `.md` por capability) siguiendo la estructura propuesta.
4. **Migrar documentación existente** a la nueva estructura (sin borrar, solo mover + crear redirects).
5. **Actualizar `00-home.md`** con la nueva tabla de navegación.
6. **Actualizar `AGENTS.md`** con la nueva estructura de documentación.
