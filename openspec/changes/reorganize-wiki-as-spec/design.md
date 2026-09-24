# Design — Estructura de directorios para Wiki-Collection como Base de Conocimiento Spec-Driven

## Contexto

El repositorio `wiki-collection` contiene documentación de un proyecto de gestión de colecciones personales (libros, juegos, juegos de mesa, cartas Magic, mazos Commander, películas/series). El documento `proposal.md` establece el porqué y qué cambia. Este documento detalla el cómo: estructura de directorios, responsabilidades, reglas de ubicación y tratamiento de documentación existente.

---

## Estructura final propuesta

```
docs/
├── 00-home.md                    # Índice — tabla de navegación con nuevos paths
├── testing.md                    # Lista de tests por feature (ex 09-testing.md)
│
├── research/                    # Contexto, decisiones, investigación
│   ├── decisions/
│   │   └── adr.md               # ADR consolidados
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
│       ├── genres.md            # Consolidado: análisis + plan
│       ├── boardgame-rating.md
│       └── books-wishlist-acquisitiondate.md
│
├── specs/                       # Especificaciones de comportamiento
│   ├── collection/
│   │   ├── books/
│   │   │   ├── spec.md
│   │   │   └── api-contract.md
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
│   │   ├── spec.md
│   │   └── api-contract.md
│   ├── user-preferences/
│   │   ├── spec.md
│   │   └── api-contract.md
│   └── image-storage/
│       ├── spec.md
│       └── api-contract.md
│
├── architecture/                # Estructura técnica
│   ├── overview.md              # Principios + diagrama (ex 02-arquitectura/README.md)
│   ├── backend.md               # Hexagonal, paquetes, puertos, adaptadores (canónico)
│   ├── frontend.md              # Stack, componentes, rutas
│   └── deployment.md            # Atlas + Render + Vercel
│
└── data-model/                  # Modelo de datos
    ├── schema.md                # Colecciones, campos, tipos, índices (ex 03.1-tablas.md)
    └── relationships.md         # Relaciones entre entidades (ex 03.2-relaciones.md)
```

---

## Responsabilidades por directorio

### `docs/00-home.md`

Índice único del wiki. Contiene:
- Tabla de navegación con todos los paths.
- Stack tecnológico (resumen de 1 línea).
- Repositorios del proyecto.
- Tabla "Estado del proyecto" con fases 1-9 y su estado real.

No debe contener contenido detallado — solo enlaces.

### `docs/research/`

Contiene todo lo que es **conocimiento contextual** pero no especificación de lo que el sistema hace hoy:

- **decisions/adr.md:** Todos los ADR (AD-001 a AD-015+). Formato: tabla con ID, fecha, estado, contexto, decisión, consecuencias. Si un ADR está truncado (AD-014+), se completa antes de migrar.
- **known-issues/issues.md:** Problemas conocidos con severidad, estado, descripción, impacto, solución esperada.
- **external-apis/*.md:** Documentación de APIs externas (Google Books, RAWG, FreeToGame, BGG, Scryfall, TMDB, Steam, Catbox). Cada archivo documenta: auth, rate limits, endpoints relevantes, mapeo de campos a modelo interno, ejemplos de request/response. Son los "contratos" del sistema con terceros.
- **future/*.md:** Investigación de funcionalidades no implementadas. Cada documento describe: qué se quiere, estado actual, cambios necesarios backend/frontend, open questions.

### `docs/specs/`

Contiene especificaciones de **lo que el sistema hace** (comportamiento + interfaces):

**Estructura por capability:**

```
specs/<capability-path>/
├── spec.md              # Reglas de negocio + comportamiento + estado actual
└── api-contract.md      # Endpoints + DTOs + filtros + códigos de error
```

**spec.md** contiene:
- Objetivo de la capability.
- Reglas de negocio (requisitos funcionales detallados).
- Estados/enums relevantes.
- Campos del modelo de dominio (resumen, no tabla completa — esa está en `data-model/schema.md`).
- Visibilidad/permisos (si aplica).
- Criterios de aceptación (checklist con - [x] / - [ ]).
- Estado actual (✅ implementado, 📋 planificado, 🚧 en progreso).

**api-contract.md** contiene:
- Lista de endpoints con método, ruta, descripción.
- Parámetros (query params, path params, body).
- Response shape (JSON de ejemplo o descripción de campos).
- Filtros disponibles.
- Códigos de error (400, 404, 409, 403, 401, 500).
- Paginación (si aplica).

**Nota importante:** `api-contract.md` es la fuente canónica de endpoints. Si hay discrepancia entre este archivo y el código, este archivo tiene prioridad como documentación (el código puede no reflejar la documentación intencional).

### `docs/architecture/`

Contiene documentación técnica de **cómo está construido**:

- **overview.md:** Principios arquitectónicos (hexagonal, inversión de dependencias, testabilidad, independencia de frameworks). Diagrama de alto nivel. Stack tecnológico resumido.
- **backend.md:** Estructura de paquetes hexagonal (domain/model, domain/port/in, domain/port/out, application/service, infrastructure/adapter-in-web, infrastructure/adapter-out-persistence, infrastructure/config). Lista de servicios, puertos, excepciones, configs. Referencias a specs para detalles de comportamiento.
- **frontend.md:** Stack (React 18.3, Vite 5, Tailwind 3, TypeScript 5, React Router 6). Estructura de carpetas (api/, components/, constants/, context/, hooks/, pages/). Lista de componentes y rutas con referencia al código.
- **deployment.md:** Atlas + Render + Vercel. Variables de entorno requeridas. Pasos de despliegue.

### `docs/data-model/`

Contiene el modelo de datos como documento de referencia:

- **schema.md:** Tabla por colección con campos, tipos, requeridos, descripción, índices. Lista de colecciones MongoDB.
- **relationships.md:** Relaciones entre entidades (ej: DeckCard pertenece a Deck, UserPreferences pertenece a User, etc.).

---

## Reglas de ubicación

1. **Si describe lo que el sistema hace hoy (reglas, endpoints, comportamiento):** va en `docs/specs/`.
2. **Si describe cómo está construido (paquetes, estructura, stack):** va en `docs/architecture/`.
3. **Si es conocimiento contextual (decisiones, APIs externas, problemas, futuros):** va en `docs/research/`.
4. **Si es modelo de datos puro (campos, tipos, índices):** va en `docs/data-model/`.
5. **Si es registro histórico (changelog, versiones):** se mantiene en su ubicación actual o se mueve a `docs/history/` si se decide crear ese directorio.

---

## Tratamiento de documentación existente

### Archivos que se consolidan (contenido migrado, archivo original se convierte en redirect)

| Archivo original | Destino | Tipo de redirect |
|-----------------|---------|------------------|
| `docs/13-fase-1-libros.md` | `docs/specs/collection/books/spec.md` | Stub: "Ver especificación en specs/collection/books/" |
| `docs/14-fase-2-juegos.md` | `docs/specs/collection/games/spec.md` | Stub |
| `docs/15-fase-3-juegos-mesa.md` | `docs/specs/collection/board-games/spec.md` | Stub |
| `docs/16-fase-4-magic.md` | `docs/specs/collection/magic-cards/spec.md` | Stub |
| `docs/17-fase-5-movieshows.md` | `docs/specs/collection/movie-shows/spec.md` | Stub |
| `docs/19-fase-8-autenticacion.md` | `docs/specs/auth/spec.md` | Stub |
| `docs/20-fase-9-colecciones-personales.md` | `docs/specs/user-preferences/spec.md` | Stub |
| `docs/05-api/README.md` | `docs/specs/*/api-contract.md` (múltiples) | Eliminar (contenido redistribuido) |
| `docs/02-arquitectura/02.1-backend.md` | `docs/architecture/backend.md` | Eliminar (contenido integrado) |
| `docs/02-arquitectura/02.2-frontend.md` | `docs/architecture/frontend.md` | Eliminar (contenido integrado) |
| `docs/07-backend/README.md` | `docs/architecture/backend.md` | Eliminar (contenido integrado) |
| `docs/07-backend/07.1-servicios.md` | `docs/architecture/backend.md` | Eliminar (contenido integrado) |
| `docs/07-backend/07.2-persistencia.md` | `docs/architecture/backend.md` + `docs/data-model/schema.md` | Eliminar (contenido integrado) |
| `docs/06-frontend/README.md` | `docs/architecture/frontend.md` | Eliminar (contenido integrado) |
| `docs/06-frontend/06.1-componentes.md` | `docs/architecture/frontend.md` | Eliminar (contenido integrado) |
| `docs/06-frontend/06.2-navegacion.md` | `docs/architecture/frontend.md` | Eliminar (contenido integrado) |
| `docs/08-deploy.md` | `docs/architecture/deployment.md` | Eliminar |
| `docs/09-testing.md` | `docs/testing.md` | Eliminar |
| `docs/03-base-de-datos/README.md` | `docs/data-model/schema.md` | Eliminar (integrado) |
| `docs/03-base-de-datos/03.1-tablas.md` | `docs/data-model/schema.md` | Eliminar (integrado) |
| `docs/03-base-de-datos/03.2-relaciones.md` | `docs/data-model/relationships.md` | Eliminar |
| `docs/05-api/05.1-usuarios.md` | `docs/specs/auth/api-contract.md` | Eliminar (contenido integrado) |
| `docs/05-api/05.2-autenticacion.md` | `docs/specs/auth/api-contract.md` | Eliminar (contenido integrado) |
| `docs/04-autenticacion.md` | Eliminar | No aporta valor |
| `docs/05-api/externas/*.md` | `docs/research/external-apis/` | Mover (renombrar) |
| `docs/10-decisiones-tecnicas.md` | `docs/research/decisions/adr.md` | Mover (renombrar) |
| `docs/11-problemas-conocidos.md` | `docs/research/known-issues/issues.md` | Mover (renombrar) |
| `docs/22`–`29-futuro-*.md` | `docs/research/future/` | Mover (renombrar) |

### Archivos que se mantienen en su ubicación

| Archivo | Razón |
|---------|-------|
| `docs/00-home.md` | Índice — se actualiza con nuevos paths |
| `docs/01-requisitos.md` | A decidir (ver Q3 del proposal) |
| `docs/12-changelog.md` | Historial — no se mueve |
| `docs/18-fase-7-caffeine-search.md` | Técnicamente es documentación técnica, no una fase con checklist. Contenido se integra en `docs/architecture/backend.md` (sección Cache/Caffeine) |

---

## Contenido mínimo de cada spec (template)

### spec.md (template)

```markdown
# <Capability>

## Objetivo
<!-- Qué hace esta capability -->

## Reglas de negocio
<!-- Requisitos funcionales detallados -->

## Estados / Enums
<!-- Estados o enums relevantes con sus valores -->

## Campos de dominio
<!-- Resumen de campos clave. Ver docs/data-model/schema.md para tabla completa -->

## Visibilidad / Permisos
<!-- Quién puede ver/ hacer qué -->

## Criterios de aceptación
- [ ] <!-- criterio -->

## Estado actual
<!-- ✅ Implementado / 📋 Planificado / 🚧 En progreso -->

## Referencias
<!-- Enlaces a specs relacionadas, docs de arquitectura, etc. -->
```

### api-contract.md (template)

```markdown
# <Capability> — API Contract

## Base path
`/api/v1/<resource>`

## Endpoints

### GET /
<!-- Descripción, parámetros, response, errores -->

### GET /{id}
<!-- ... -->

### POST /
<!-- ... -->

### PUT /{id}
<!-- ... -->

### DELETE /{id}
<!-- ... -->

## Filtros
<!-- Query params disponibles -->

## Códigos de error
<!-- 400, 404, 409, 403, 401, 500 -->

## Paginación
<!-- Si aplica -->
```

---

## Relaciones entre documentos

```
docs/00-home.md
    ├────> docs/specs/collection/books/spec.md
    │           └───> docs/specs/collection/books/api-contract.md
    ├───> docs/specs/collection/games/spec.md
    │           └───> docs/specs/collection/games/api-contract.md
    ├───> docs/specs/collection/board-games/spec.md
    │           └───> docs/specs/collection/board-games/api-contract.md
    ├───> docs/specs/collection/magic-cards/spec.md
    │           └───> docs/specs/collection/magic-cards/api-contract.md
    ├───> docs/specs/collection/decks/spec.md
    │           └───> docs/specs/collection/decks/api-contract.md
    ├───> docs/specs/collection/movie-shows/spec.md
    │           └───> docs/specs/collection/movie-shows/api-contract.md
    ├───> docs/specs/auth/spec.md
    │           └───> docs/specs/auth/api-contract.md
    ├───> docs/specs/user-preferences/spec.md
    │           └───> docs/specs/user-preferences/api-contract.md
    ├───> docs/specs/image-storage/spec.md
    │           └───> docs/specs/image-storage/api-contract.md
    ├───> docs/architecture/overview.md
    │           ├───> docs/architecture/backend.md
    │           ├───> docs/architecture/frontend.md
    │           └───> docs/architecture/deployment.md
    ├───> docs/data-model/schema.md
    │           └───> docs/data-model/relationships.md
    └───> docs/testing.md

docs/architecture/backend.md
    ├───> docs/specs/collection/*/spec.md   (enlaces a comportamiento)
    ├───> docs/research/external-apis/     (enlaces a APIs externas usadas)
    └───> docs/data-model/schema.md        (enlaces a modelo)

docs/specs/*/api-contract.md
    └───> docs/architecture/backend.md     (contexto de implementación)
    └───> docs/data-model/schema.md        (modelo de datos relevante)

docs/research/external-apis/*.md
    └───> docs/specs/collection/*/api-contract.md   (qué endpoints de la API externa usa cada spec)

docs/research/future/*.md
    └───> docs/specs/collection/*/spec.md           (qué capacidad necesitaría para implementarse)
```

---

## OpenSpec alignment

Este diseño está alineado con el esquema `spec-driven` de OpenSpec:

- **`specs/`** mapea directamente a la sección `specs/` de OpenSpec. Cada capability (`collection/books`, `collection/games`, etc.) es una especificación autocontenida que define comportamiento e interfaces.
- **`design.md`** (en el cambio OpenSpec) describe la estructura de directorios y reglas de ubicación — lo que este documento hace.
- **`tasks.md`** (en el cambio OpenSpec) lista las tareas de migración: crear directorios, escribir specs, migrar redirects, actualizar índices.

La diferencia clave: OpenSpec espera que las specs estén en `openspec/specs/<capability>/spec.md` (dentro del directorio del cambio). En la wiki real, las specs vivirán en `docs/specs/<capability>/spec.md`. Esto es intencional: la wiki es la fuente de documentación del proyecto, no solo del cambio.

---

## Decisiones de diseño

### D1: Dos archivos por capability (spec.md + api-contract.md)

**Motivación:** Separar reglas de negocio (spec.md) de detalles de implementación de API (api-contract.md). Permite que un lector entienda qué hace la capability sin entrar en detalles de endpoints, y que un desarrollador encuentre rápidamente la signatura de un endpoint.

**Alternativas consideradas:**
- Un solo archivo por capability — más simple pero mezcla dos niveles de abstracción.
- Más de dos archivos — excesivo para el nivel de detalle actual.

**Decisión:** Dos archivos por capability.

### D2: stubs como redirects en lugar de eliminar

**Motivación:** Los archivos de fase (13-21) tienen contenido histórico (checklists, estructuras de paquetes, criterios de aceptación). Eliminarlos completamente perdería esa información. Convertirlos en stubs con enlace a la spec actual preserva la referencia histórica sin duplicar contenido.

**Alternativas consideradas:**
- Eliminar y mover todo el contenido — pierde historial.
- Conservar completo — duplica contenido.

**Decisión:** Stubs con enlace a la spec correspondiente.

### D3: `docs/research/external-apis/` para APIs externas

**Motivación:** Las APIs externas son contratos con terceros, no comportamiento del propio sistema. pertenecen a investigación, no a especificaciones. Además, varias specs pueden referenciar la misma API externa (ej: Books usa Google Books, Games usa RAWG).

**Alternativas consideradas:**
- Poner la documentación de cada API externa dentro de la spec de la colección que la usa — duplicaría información si una API es usada por múltiples capacidades.
- Poner en `docs/architecture/` — no es arquitectura del sistema propio.

**Decisión:** `docs/research/external-apis/` como documentación de referencia de APIs externas.

### D4: `docs/testing.md` separado del resto

**Motivación:** La lista de tests por feature es información de cobertura, no especificación de comportamiento ni arquitectura. Mantenerlo separado permite actualizarlo independientemente de las specs.

**Alternativas consideradas:**
- Integrar en `docs/architecture/backend.md` — testing es transversal, no parte de la arquitectura de una capa específica.
- Eliminar — la lista de tests tiene valor para conocer qué está testeado.

**Decisión:** `docs/testing.md` separado, enlaza a specs cuando es relevante.

---

## Resumen de cambios en cada directorio existente

| Directorio actual | Cambio |
|------------------|--------|
| `docs/00-home.md` | Actualizar tabla de navegación |
| `docs/01-requisitos.md` | **Se mantiene** — en su ubicación actual. Enlazar desde `00-home.md`. |
| `docs/02-arquitectura/` | Eliminar (contenido migrado a architecture/) |
| `docs/03-base-de-datos/` | Eliminar (contenido migrado a data-model/) |
| `docs/04-autenticacion.md` | Eliminar |
| `docs/05-api/` | Eliminar (contenido redistribuido a specs/ y research/external-apis/) |
| `docs/06-frontend/` | Eliminar (contenido migrado a architecture/frontend.md) |
| `docs/07-backend/` | Eliminar (contenido migrado a architecture/backend.md) |
| `docs/08-deploy.md` | Eliminar (migrado a architecture/deployment.md) |
| `docs/09-testing.md` | Eliminar (migrado a testing.md) |
| `docs/10-decisiones-tecnicas.md` | Eliminar (migrado a research/decisions/adr.md) |
| `docs/11-problemas-conocidos.md` | Eliminar (migrado a research/known-issues/issues.md) |
| `docs/12-changelog.md` | Mantener |
| `docs/13`–`21-fase-*.md` | Convertir a **stubs** con redirect a la spec correspondiente |
| `docs/22`–`29-futuro-*.md` | Mover a research/future/ |
| `docs/18-fase-7-caffeine-search.md` | Integrar en architecture/backend.md |
| `docs/30-cambio-implementado-game-platforms.md` | A decidir (ver Q7 del proposal) |

### Nuevos directorios a crear

- `docs/research/decisions/`
- `docs/research/known-issues/`
- `docs/research/external-apis/`
- `docs/research/future/`
- `docs/specs/collection/books/`
- `docs/specs/collection/games/`
- `docs/specs/collection/board-games/`
- `docs/specs/collection/magic-cards/`
- `docs/specs/collection/decks/`
- `docs/specs/collection/movie-shows/`
- `docs/specs/auth/`
- `docs/specs/user-preferences/`
- `docs/specs/image-storage/`
- `docs/architecture/`
- `docs/data-model/`
