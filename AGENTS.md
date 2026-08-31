# Wiki-Collection — Agent Instructions

## What This Repo Is

This is the **documentation-only** wiki for the Wiki-Collection project (a personal collection manager for books, games, board games, magic cards, and movies/shows). It contains **no code** — only markdown files in `docs/`.

The actual source lives in separate repos:
- Backend: `wiki-collection-backend` — Java 25 + Spring Boot 4 + MongoDB
- Frontend: `wiki-collection-frontend` — React + Vite

## Documentation Structure

```
docs/
├── architecture.md      # Tech stack, architectural decisions, repo layout
├── data-models.md       # Entity schemas (Book, Game, BoardGame, MagicCard, MovieShow)
├── external-apis.md     # Google Books API + Open Library API reference
├── api-reference.md     # Backend REST endpoints (base: localhost:8080/api)
├── phase-1-books.md     # Phase 1 implementation plan (books backend)
└── README.md            # Docs index
```

## Conventions

- **Language:** All documentation is in Spanish. Write new docs in Spanish to match.
- **Format:** Markdown only. No HTML, no diagrams-as-code unless they render in a browser.
- **Entity docs:** `data-models.md` uses a table per entity with columns: Campo, Tipo, Requerido, Descripción. Follow this pattern for new entities.
- **API docs:** `api-reference.md` documents path, query params, and response shape per endpoint. Follow this pattern for new endpoints.
- **Phase plans:** Each phase gets its own `phase-N-<topic>.md` with a numbered checklist (`- [ ]`), steps grouped by layer (config → model → repo → service → controller → tests), and a "Criterios de Aceptación" section.

## How to Contribute

1. Edit the relevant `docs/*.md` file directly.
2. Update `docs/README.md` index if adding a new doc.
3. For cross-cutting changes (e.g., adding a new entity), update both `data-models.md` (schema) and `api-reference.md` (endpoints) together.

## Pitfalls

- **Do not add code, configs, or scripts to this repo.** No `pom.json`, `package.json`, `Dockerfile`, `.github/workflows/`, etc. If you need to document a config value, write it in markdown — don't create the actual file.
- **This repo has no build, test, or lint commands.** There is no `npm test`, `mvn`, or CI. Validation is purely editorial (correctness of documented schemas, endpoints, and plans).
- **Don't confuse this with the backend repo.** The backend uses Java 25 + Spring Boot 4 (not Java 17 + Spring Boot 3.x as an older plan file suggests — `architecture.md` is authoritative).
- **Phase plans are living checklists.** Items get checked off as the backend/frontend repos implement them. Don't check items unless the implementation actually exists and works.
