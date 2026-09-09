# Wiki-Collection — Agent Instructions

## What This Repo Is

Documentation-only wiki for the Wiki-Collection project (personal collection manager for books, games, board games, magic cards, movies/shows). Contains **no code** — only markdown in `docs/`.

Source repos:
- Backend: `backend-collection` — Java 25 + Spring Boot 4.1.1 + MongoDB (https://github.com/JavierMedarde99/backend-collection)
- Frontend: `frontend-collection` — React 18.3 + Vite 5 + Tailwind 3 + TypeScript 5 (https://github.com/JavierMedarde99/frontend-collection)

## Documentation Structure

```
docs/
├── 00-home.md                        # Index + stack + repo links
├── 01-requisitos.md                  # Functional/non-functional requirements
├── 02-arquitectura/
│   ├── README.md                     # High-level architecture diagram
│   ├── 02.1-backend.md              # Hexagonal backend structure
│   └── 02.2-frontend.md             # React frontend structure
├── 03-base-de-datos/
│   ├── README.md                     # MongoDB collections + indexes
│   ├── 03.1-tablas.md               # Entity schemas (all entities)
│   └── 03.2-relaciones.md           # Cross-entity relationships
├── 04-autenticacion.md               # Auth (planned, not implemented)
├── 05-api/
│   ├── README.md                     # All REST endpoints + DTOs
│   ├── 05.1-usuarios.md             # User endpoints (planned)
│   ├── 05.2-autenticacion.md        # Auth endpoints (planned)
│   └── externas/
│       ├── externas-books.md         # Google Books + Open Library
│       ├── externas-videogames.md    # RAWG + FreeToGame
│       ├── externas-boardgames.md    # BoardGameGeek JSON + XML
│       ├── externas-magic.md         # Scryfall
│       └── externas-movies.md        # TMDB (planned)
├── 06-frontend/
│   ├── README.md                     # Pages + stack
│   ├── 06.1-componentes.md          # Component catalog
│   └── 06.2-navegacion.md           # Routes + navigation
├── 07-backend/
│   ├── README.md                     # Package structure + endpoints
│   ├── 07.1-servicios.md            # All services + external clients
│   └── 07.2-persistencia.md         # Repositories + mappers + Mongo config
├── 08-deploy.md                      # Deployment plan
├── 09-testing.md                     # Test inventory + coverage
├── 10-decisiones-tecnicas.md         # Architecture decision records
├── 11-problemas-conocidos.md         # Known issues
├── 12-changelog.md                   # Version history
├── 13-fase-1-libros.md               # Phase 1: Books (✅ complete)
├── 14-fase-2-juegos.md               # Phase 2: Video games (✅ complete)
├── 15-fase-3-juegos-mesa.md          # Phase 3: Board games (✅ complete)
└── 16-fase-4-magic.md                # Phase 4: Magic cards (✅ complete)
```

## Conventions

- **Language:** All docs in Spanish. Write new docs in Spanish.
- **Format:** Markdown only. No HTML, no diagrams-as-code unless browser-renderable.
- **Entity schemas:** `03-base-de-datos/03.1-tablas.md` uses one table per entity with columns: Campo, Tipo, Requerido, Descripción.
- **API docs:** `05-api/README.md` documents path, query params, and response shape per endpoint.
- **Phase plans:** Each phase gets `NN-fase-N-topic.md` with numbered checklist (`- [x]`/`- [ ]`), steps grouped by layer (config → model → repo → service → controller → tests), and a "Criterios de Aceptación" section.
- **Index:** `00-home.md` is the navigation hub — update it when adding new docs.

## How to Contribute

1. Edit the relevant `docs/*.md` file directly.
2. Update `docs/00-home.md` index if adding a new doc.
3. For cross-cutting changes (e.g., new entity), update both `03-base-de-datos/03.1-tablas.md` (schema) and `05-api/README.md` (endpoints) together.

## Pitfalls

- **No code, configs, or scripts in this repo.** No `pom.xml`, `package.json`, `Dockerfile`, `.github/workflows/`, etc. Document config values in markdown — don't create the actual file.
- **No build/test/lint commands.** No `npm test`, `mvn`, or CI. Validation is purely editorial (correctness of schemas, endpoints, plans).
- **Don't confuse with backend/frontend repos.** Backend is `backend-collection` (not `wiki-collection-backend`); frontend is `frontend-collection` (not `wiki-collection-frontend`).
- **Phase plans are living checklists.** Don't check items unless the implementation actually exists and works in the source repos.
- **Repo names in docs:** Use `backend-collection` and `frontend-collection` (the actual GitHub repo names), not the older `wiki-collection-backend`/`wiki-collection-frontend`.
