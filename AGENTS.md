# Wiki-Collection — Agent Instructions

Documentation-only repo for the Wiki-Collection project (personal collection manager for books, games, board games, magic cards, movies/shows, commander decks). **No code here** — markdown only. Validation is editorial, not executable.

Source repos: `backend-collection` (Java 25 + Spring Boot 4.1.1 + MongoDB) and `frontend-collection` (React 18.3 + Vite 5 + Tailwind 3 + TypeScript 7 + React Router 6).

## Where the truth lives

This wiki is a mirror of two code repos. **The code wins over the docs.** When they disagree, fix the doc — and say so.

Verified against source on 2026-10-02. Re-verified against `origin/main` on 2026-10-19 (that re-check found the cache count and the controller count both wrong, and the board-game expansions already shipped). Re-verify before citing; these repos move fast.

## Source repos on this machine

| Repo | Local path | Branch at last check |
|------|-----------|----------------------|
| Backend | `/home/javi/Documentos/java-proyects/backend-collection` | detached HEAD (stale — read `origin/main`) |
| Frontend | `/home/javi/orca/frontend-collection` | `feature/platform-catalog-endpoint` (deleted on the remote) |

- **The local checkouts are stale — do not read the spec from them.** As of 2026-10-19 the backend checkout is on a **detached HEAD** (no branch) and the frontend one sits on `feature/platform-catalog-endpoint`, a branch that no longer exists on the remote. Reading those trees told me the board-game expansions were unimplemented when `origin/main` had shipped them. Verify with `git fetch` and then read `origin/main` directly:
  ```bash
  git ls-tree -r --name-only origin/main -- src/main/java
  git show origin/main:path/to/File.java
  git log --oneline -5 origin/main
  ```
- `/home/javi/orca/workspaces/backend-collection` and `.../frontend-collection` are **empty shells** holding unrelated projects (`brill`, `batfish`). Don't search there.
- No `backend-collection` under `/home/javi/orca/`.

## Commands (run in the source repos, never here)

Backend:
```bash
mvn test      # unit + integration tests
mvn verify    # tests + JaCoCo gate — hard fails below 80% line coverage
```

Frontend — **only these four scripts exist**:
```bash
npm test      # vitest run
npm run build
npm run dev
npm run preview
```

There is **no** `test:coverage` script and no coverage plugin configured.

## Documentation structure

```
specs/{capability}/spec.md          # Behaviour: business rules, states, enums
specs/{capability}/api-contract.md  # Endpoints, query params, request/response, error codes
architecture/                       # Technical structure: backend, frontend, routes, components, testing, deployment
research/external-apis/{name}.md    # Third-party API contracts: auth, rate limits, endpoints, retry/cache
research/future/{name}.md           # Unimplemented feature research
research/decisions/adr-NNN-*.md     # Architecture decision records
data-model/schema.md                # MongoDB collections, fields, indexes
requirements.md                     # Functional requirements per phase
changelog.md                        # Historical record (= the OKF "log")
README.md                           # Front page: stack, repos, phase status, doc structure
index.md                            # OKF bundle index (okf_version)
```

Every directory listed above also has an `index.md` (OKF directory listing, no frontmatter).

Capabilities under `specs/`: `books`, `games`, `board-games`, `magic-cards`, `decks`, `movie-shows`, `auth`, `user-preferences`.

Missing by design/omission — do not assume they exist:
- `specs/image-storage/` was planned (Catbox) but never written. Image storage is covered in `architecture/backend.md` + `research/external-apis/catbox.md`.
- `data-model/relationships.md` was planned but never written.

## OKF bundle

This repo is an **Open Knowledge Format (OKF v0.1)** bundle (Google Cloud, 2026-06-12). The repo
root **is** the bundle root — there is no `wiki/` subdirectory.

- **Concept documents** — every `.md` under `specs/`, `architecture/`, `research/` and
  `data-model/`, plus `requirements.md` and `changelog.md`, starts with a YAML frontmatter block
  whose `type` key is **required**. `title`, `description` and `tags` are expected; `title` must
  match the document's H1.
- **Reserved filenames** — `index.md` (directory listing) and `log.md` must never be concept
  documents. Index files carry no frontmatter; only the bundle-root `index.md` carries
  `okf_version`. There is no `log.md`: `changelog.md` plays that role.
- **`index.md` per directory** — the repo root plus `specs/`, `architecture/`, `research/`,
  `research/external-apis/` and `data-model/` each have one. Keep them in sync when adding or
  renaming documents; each entry reuses the linked document's `description`.
- **`type` vocabulary** (controlled — keep it small):

  | type | applies to |
  |------|-----------|
  | `spec` | `specs/{capability}/spec.md` |
  | `api-contract` | `specs/{capability}/api-contract.md` |
  | `architecture` | `architecture/*.md` |
  | `adr` | `research/decisions/adr-NNN-*.md` |
  | `external-api` | `research/external-apis/*.md` |
  | `future-feature` | `research/future/*.md` |
  | `research` | top-level `research/*.md` |
  | `data-model` | `data-model/schema.md` |
  | `requirement` | `requirements.md` |
  | `changelog` | `changelog.md` |

- **Outside the bundle** — `AGENTS.md`, `README.md`, `openspec/**` and `.hermes/**` are
  project-level artifacts, not concept documents: no OKF frontmatter.

## OpenSpec workflow

`openspec/` holds a spec-driven planning layer. Read this before touching it.

- `openspec/config.yaml` — `schema: spec-driven`. Its `context:` block is **empty**; project context for AI tooling is not configured there.
- `openspec/changes/reorganize-wiki-as-spec/` — one **active, unarchived** change. Its `tasks.md` has **every checkbox unchecked**, yet the migration it describes already happened on disk. Treat the task list as stale, not as work to redo.
- `openspec/specs/` is **empty** — main specs were never synced from deltas.
- `openspec/changes/archive/` is empty.
- The change artifacts (proposal/design/tasks) describe a **`docs/`-prefixed layout that does not exist**. The real directories sit at the repo root. Trust disk over artifacts.
- **The `openspec` CLI is not installed on this machine.** The 6 skills in `.hermes/skills/openspec-*` declare `compatibility: Requires openspec CLI` and will not run. Edit change artifacts by hand; don't burn turns on CLI invocations.
- `.hermes/skills/` is a Hermes-harness skill directory, not OpenCode's. `.hermes/plans/2026-08-29_*.md` is an obsolete early plan (Java 17, Spring Boot 3, `docs/` paths) — ignore its content.

## Conventions

- **Language:** all docs in Spanish. `AGENTS.md` stays in English for agent readability.
- **Format:** markdown only. No HTML, no diagrams-as-code.
- **Canonical sources** — one fact, one file: entity fields → `data-model/schema.md`; hexagonal structure → `architecture/backend.md`; per-endpoint contract → `specs/{capability}/api-contract.md`. Link instead of duplicating.
- **Cross-cutting change** (new entity or capability): update `specs/{capability}/spec.md` + `specs/{capability}/api-contract.md` + `data-model/schema.md` + `architecture/backend.md` together.
- **Index:** update `README.md` phase-status table when a phase changes state. Keep `research/future/index.md` current.
- **Phase checklists are living.** Don't tick a box unless the feature works in the source repos.

## Pitfalls

- **Don't create build files.** No `pom.xml`, `package.json`, `Dockerfile`, etc. in this repo.
- **Don't add features to the wiki that don't exist in code.** The wiki mirrors reality, not intent.
- **Stale files that repeat wrong facts.** `README.md` and `architecture/testing.md` carried wrong stack versions and test counts; they were corrected on 2026-10-02 but the backend/frontend feature-branch drift means counts move. Re-derive with `find src/test -name '*Test.java' | wc -l` and `find src -name '*.test.ts*' | wc -l` rather than copying numbers between docs.
- **MagicCard has no save/update.** Only `addFromScryfall` + delete. Cards come from Scryfall. `GET /magic/scryfall/{scryfallId}/printings` lists all printings of a card (for the user to choose which one to save).
- **Images go to Catbox.moe**, not local filesystem: `ImageStorageController` + `CatboxClient`. The exception is `CatboxUploadException` — there is no `ImageNotFoundException`.
- **MongoDB property is `spring.mongodb.uri`**, NOT `spring.data.mongodb.uri`.
- **JaCoCo gate is real:** `mvn verify` fails below 80% line coverage (`BUNDLE` element, `LINE`/`COVEREDRATIO` 0.80, jacoco 0.8.15). Don't dodge it with `-DskipTests`.
- **Caffeine is in-memory, no Redis.** There are **44** distinct cache names, all listed in `CacheConfig.CACHE_NAMES`: the collection `*Search` caches (`bookSearch`, `gameSearch`, `boardgameSearch`, `magicSearch`, `movieSearch`, `commanderSearch`, `amiiboSearch`, `funkoSearch`), `bookEnrich`, `platformSearch`, `magicPrintings`, `funkoCategories`, the external-catalog caches `boardgameExpansions` and `gameAdditions`, `*Detail`/`*List` per collection (book, game, boardgame, movie, magic, deck, amiibo, funko) plus `userProfileDetail`, the per-user lists (`userBooksList`, `userGamesList`, `userBoardGamesList`, `userMagicCardsList`, `userDecksList`, `userMovieShowsList`, `userStreamsList`, `userAmiibosList`, `userFunkosList`), `userPreferencesDetail`, `userActiveCollections`, `userPreferenceFlags`, and `stats`. List/detail keys include the viewer id. This count is re-derived from `origin/main`, not from the local checkout — the local checkouts sit on stale branches (see above). Verify with `CacheConfig` — count changes with each new feature.
- **`RateLimitFilter`** exists and is easy to miss: `OncePerRequestFilter` on auth endpoints, `app.rate-limit.auth-per-minute` (default 100), per-IP Caffeine window, responds **429**.
- **13 controllers** under `/api/v1`: `auth`, `boardgames`, `books`, `decks`, `games`, `images`, `magic`, `movieshows`, `preferences`, `stats`, `users`, plus the two collectibles ones, `amiibos` and `funkos`.
- **`owner` filter on every collection GET:** `mine` (default) / `other` / `all`. `other` and `all` require auth.
- **Auth:** JWT access 15 min + refresh 7 days. `POST`/`PUT`/`DELETE` require auth, `GET` is public; ownership is checked on writes.
- **Deck import is async (Fase 23):** `POST /decks/{id}/imports` (multipart) and `/imports/text` return **202** + a job URL — poll `GET /decks/{id}/imports/{jobId}`. The deck only appears in the response when `status=COMPLETED`; `FAILED` means the deck was left untouched. Jobs live in memory (`DeckImportJobStore`), 429 when the queue is full.
- **Broken link to fix if you're in `architecture/`:** `overview.md` still points at `docs/00-home.md`, which no longer exists — the front page is `README.md`.