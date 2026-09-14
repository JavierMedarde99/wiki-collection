# Wiki-Collection — Agent Instructions

Documentation-only wiki for the Wiki-Collection project (personal collection manager for books, games, board games, magic cards, movies/shows, commander decks). **No code** — only markdown in `docs/`.

Source repos:
- Backend: `backend-collection` — Java 25 + Spring Boot 4.1.1 + MongoDB (https://github.com/JavierMedarde99/backend-collection)
- Frontend: `frontend-collection` — React 18.3 + Vite 5 + Tailwind 3 + TypeScript 7 + React Router 6 (https://github.com/JavierMedarde99/frontend-collection)

## Documentation Structure

```
docs/
├── 00-home.md                        # Index + stack + repo links (UPDATE WHEN ADDING DOCS)
├── 01-requisitos.md                  # Functional/non-functional requirements
├── 02-arquitectura/
│   ├── README.md                     # High-level architecture diagram
│   ├── 02.1-backend.md              # Hexagonal backend structure (30 domain models, 7 controllers, 14 configs)
│   └── 02.2-frontend.md             # React frontend structure (43 components, 25 routes)
├── 03-base-de-datos/
│   ├── README.md                     # MongoDB collections + indexes
│   ├── 03.1-tablas.md               # 6 entity schemas (Book, Game, BoardGame, MagicCard, Deck, MovieShow)
│   └── 03.2-relaciones.md           # Cross-entity relationships
├── 04-autenticacion.md               # Auth (planned, not implemented)
├── 05-api/
│   ├── README.md                     # 34 endpoints + DTOs + error codes
│   ├── 05.1-usuarios.md             # User endpoints (planned)
│   ├── 05.2-autenticacion.md        # Auth endpoints (planned)
│   └── externas/
│       ├── externas-books.md         # Google Books API
│       ├── externas-videogames.md    # RAWG + FreeToGame
│       ├── externas-steam.md         # Steam Web API (achievements)
│       ├── steam-api-key-guide.md    # How to get Steam API Key
│       ├── externas-boardgames.md    # BoardGameGeek XML API
│       ├── externas-magic.md         # Scryfall API
│       ├── externas-movies.md        # TMDB API
│       └── externes-image-hosting.md # Catbox.moe image hosting
├── 06-frontend/
│   ├── README.md                     # Pages + stack
│   ├── 06.1-componentes.md          # 43 components (cards, forms, search, badges, UI)
│   └── 06.2-navegacion.md           # 25 routes + navigation map
├── 07-backend/
│   ├── README.md                     # Package structure + endpoints
│   ├── 07.1-servicios.md            # 17 services + 9 external clients (with Caffeine cache) + validators
│   └── 07.2-persistencia.md         # Repositories + mappers + Mongo config
├── 08-deploy.md                      # Deployment plan
├── 09-testing.md                     # 56 backend tests + 5 frontend tests + coverage
├── 10-decisiones-tecnicas.md         # Architecture decision records
├── 11-problemas-conocidos.md         # Known issues
├── 12-changelog.md                   # Version history
├── 13-fase-1-libros.md               # ✅ Books (Google Books)
├── 14-fase-2-juegos.md               # ✅ Video games (RAWG + FreeToGame + Steam)
├── 15-fase-3-juegos-mesa.md          # ✅ Board games (BGG XML)
├── 16-fase-4-magic.md                # ✅ Magic cards (Scryfall)
├── 17-fase-5-movieshows.md           # ✅ Movies/Shows (TMDB)
├── 18-fase-7-caffeine-search.md      # 📋 Caffeine cache for search APIs
└── 19-fase-8-autenticacion.md        # 📋 User authentication (JWT + Spring Security)
```

## Conventions

- **Language:** All docs in Spanish.
- **Format:** Markdown only. No HTML, no diagrams-as-code unless browser-renderable.
- **Schemas:** `03.1-tablas.md` uses one table per entity: Campo, Tipo, Requerido, Descripción.
- **API docs:** `05-api/README.md` documents path, query params, response shape, and error codes per endpoint.
- **Phase plans:** `NN-fase-N-topic.md` with `- [x]`/`- [ ]` checklist, layers (config → model → repo → service → controller → tests), "Criterios de Aceptación".
- **External API docs:** `05-api/externas/externas-{name}.md` documents auth, rate limits, endpoints, retry/cache strategy.
- **Index:** `00-home.md` — update on new docs, keep "Estado del Proyecto" table current.

## How to Contribute

1. Edit `docs/*.md` directly.
2. Update `docs/00-home.md` index.
3. Cross-cutting changes: update `03.1-tablas.md` (schema) + `05-api/README.md` (endpoints) + `07.1-servicios.md` (service/client) together.

## Pitfalls

- **No code in this repo.** Document config values in markdown — don't create `pom.xml`, `package.json`, `Dockerfile`, etc.
- **No build/test/lint commands.** Validation is editorial only — verify schemas, endpoints, plans match source repos.
- **Sync with source repos.** Wiki reflects `backend-collection` and `frontend-collection` reality. Don't document features that don't exist in those repos.
- **Phase checklists are living.** Don't check items unless implementation exists and works in source repos.
- **MagicCard has no save/update.** Backend only supports `addFromScryfall` — cards come from Scryfall, not manual creation.
- **Images use Catbox.moe, not filesystem local.** `ImageStorageController` + `CatboxClient` — no `ImageNotFoundException` (it's `CatboxUploadException`).
- **Caffeine is in-memory.** 6 caches (bookSearch, gameSearch, boardgameSearch, magicSearch, commanderSearch, movieSearch). No Redis.
- **Repo names:** Use `backend-collection` and `frontend-collection` (actual GitHub names).
