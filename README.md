# Wiki-Collection

Repositorio de documentación para el proyecto Wiki-Collection.

## Proyecto

Wiki-Collection es una aplicación web para gestionar colecciones personales de libros, videojuegos, juegos de mesa, cartas Magic: The Gathering, películas/series y mazos Commander.

## Stack Tecnológico

| Capa | Tecnología |
|------|------------|
| Backend | Java 25 + Spring Boot 4.1.1 |
| Base de datos | MongoDB + Spring Data |
| Frontend | React 18.3 + Vite 5 + Tailwind 3 + TypeScript 5 |
| APIs externas | Google Books / RAWG / FreeToGame / BoardGameGeek XML / Scryfall / Steam / TMDB |
| Documentación | springdoc-openapi (Swagger 3.1.0) |
| Testing | JUnit 5 + Mockito + MockWebServer + JaCoCo (80% cobertura) |
| Build | Maven (backend) + npm (frontend) |
| Caché | Spring Cache + Caffeine (6 cachés en memoria) |
| Seguridad | Spring Security + JJWT 0.12.6 (JWT) |

## Repositorios

| Repo | Descripción | Estado |
|------|-------------|--------|
| [backend-collection](https://github.com/JavierMedarde99/backend-collection) | Java 25 + Spring Boot 4 | ✅ Activo |
| [frontend-collection](https://github.com/JavierMedarde99/frontend-collection) | React + Vite | ✅ Activo |
| [wiki-collection](https://github.com/JavierMedarde99/wiki-collection) | Este repo, documentación | ✅ Activo |

## Estado de las Fases

| Fase | Descripción | Estado |
|------|-------------|--------|
| Fase 1 | Libros (Google Books API) | ✅ Completada |
| Fase 2 | Videojuegos (RAWG + FreeToGame + Steam) | ✅ Completada |
| Fase 3 | Juegos de Mesa (BoardGameGeek XML) | ✅ Completada |
| Fase 4 | Cartas Magic: The Gathering (Scryfall) | ✅ Completada |
| Fase 4.1 | Mazos Commander (Scryfall + gestión mazos) | ✅ Completada |
| Fase 5 | Películas/Series (TMDB) | ✅ Completada |
| Fase 6 | Imágenes (Catbox.moe) | ✅ Completada |
| Fase 7 | Caché Caffeine en búsquedas externas | ✅ Completada |
| Fase 8 | Autenticación de usuarios (JWT + Spring Security) | ✅ Completada |
| Fase 9 | Colecciones Personales y Visibilidad | ✅ Completada |
| Fase 22 | Scanner de código de barras (ISBN) | ✅ Completada |
| Fase 24 | Seguimiento de lectura (parcial: progreso básico) | ✅ Completada (parcial) |
| Fase 25 | Plataformas de Streaming (parcial: providers + badges) | ✅ Completada (parcial) |
| Fase 26 | Géneros para colecciones | ✅ Completada |
| Fase 27 | Géneros específicos por tipo de colección | ✅ Completada |
| Fase 23 | Importación de mazos Commander | 📋 Pendiente |
| Fase 28 | Valoración de juegos de mesa | 📋 Pendiente |
| Fase 29 | Lista de deseos y adquisición de libros | 📋 Pendiente |

## Estructura de la Documentación

```
specs/
├── books/                              # ✅ Libros (Google Books)
│   ├── spec.md
│   └── api-contract.md
├── games/                              # ✅ Videojuegos (RAWG + FreeToGame)
│   ├── spec.md
│   └── api-contract.md
├── board-games/                        # ✅ Juegos de Mesa (BGG XML)
│   ├── spec.md
│   └── api-contract.md
├── magic-cards/                        # ✅ Magic: The Gathering (Scryfall)
│   ├── spec.md
│   └── api-contract.md
├── decks/                              # ✅ Mazos Commander (Scryfall + gestión)
│   ├── spec.md
│   └── api-contract.md
├── movie-shows/                        # ✅ Películas/Series (TMDB)
│   ├── spec.md
│   └── api-contract.md
├── auth/                               # ✅ Autenticación (JWT + Spring Security)
│   ├── spec.md
│   └── api-contract.md
└── user-preferences/                   # ✅ Colecciones Personales y Visibilidad
    ├── spec.md
    └── api-contract.md

architecture/
├── overview.md                         # Visión general del proyecto
├── backend.md                          # Arquitectura hexagonal del backend (Java 25 + Spring Boot 4)
├── frontend.md                         # Estructura del frontend (React + Vite + Tailwind)
├── frontend-routes.md                  # 30 rutas del frontend
├── frontend-components.md              # Catálogo de componentes frontend
├── backend-services-detail.md          # Detalle de servicios backend
├── api-contract-index.md               # Índice de contratos de API
├── testing.md                          # Inventario de tests (56 backend + 5 frontend)
└── deployment.md                       # Plan de despliegue (MongoDB Atlas + Render + Vercel)

research/
├── auth.md                             # Investigación de autenticación
├── architecture.md                     # Decisiones arquitectónicas
├── business-rules.md                   # Reglas de negocio
├── validation.md                       # Validaciones
├── known-issues.md                     # Problemas conocidos
├── decisions/                          # ADRs (Architecture Decision Records)
│   ├── index.md
│   ├── adr-001-mongodb.md
│   ├── adr-007-wiki-repo-separado.md
│   ├── adr-008-bgg-xml-juegos-mesa.md
│   ├── adr-009-scryfall-magic.md
│   ├── adr-010-spring-boot-4-java-25.md
│   ├── adr-011-jacoco-cobertura.md
│   ├── adr-012-boardgamestatus-reducido.md
│   ├── adr-013-tmdb-movieshows.md
│   ├── adr-014-commander-validacion.md
│   ├── adr-015-moviestatus-sin-wishlist.md
│   └── adr-017-endpoints-movieshows.md
└── future/                             # Funcionalidades futuras e investigación
    ├── index.md                        # Índice de funcionalidades futuras
    ├── barcode-scanner.md              # ✅ Fase 22: Scanner de código de barras
    ├── reading-progress.md             # ✅ Fase 24: Seguimiento de lectura (parcial)
    ├── streaming-platforms.md          # ✅ Fase 25: Plataformas de streaming (parcial)
    ├── genres-collections.md           # ✅ Fase 26: Géneros para colecciones
    ├── genres-books-games-boardgames.md # ✅ Fase 27: Géneros específicos por tipo
    ├── deck-import.md                  # 📋 Fase 23: Importación de mazos (pendiente)
    ├── boardgame-rating.md             # 📋 Fase 28: Valoración de juegos de mesa (pendiente)
    └── books-wishlist.md               # 📋 Fase 29: Wishlist + adquisición libros (pendiente)
```

## APIs Externas Integradas

| API | Uso | Fase | Estado |
|-----|-----|------|--------|
| Google Books | Búsqueda de libros | 1 | ✅ |
| RAWG | Búsqueda de videojuegos | 2 | ✅ |
| FreeToGame | Fallback videojuegos | 2 | ✅ |
| Steam Web API | Logros de juegos | 2 | ✅ |
| BoardGameGeek XML | Búsqueda juegos de mesa | 3 | ✅ |
| Scryfall | Búsqueda cartas Magic | 4 | ✅ |
| TMDB | Búsqueda películas/series | 5 | ✅ |
| Catbox.moe | Hosting de imágenes | 6 | ✅ |

## Convenciones

- **Idioma:** Toda la documentación está en español
- **Formato:** Solo markdown, sin HTML ni diagramas como código
- **Nombres de repos:** Usar `backend-collection` y `frontend-collection` (nombres reales en GitHub)

## Cómo Contribuir

1. Editar los archivos en `specs/*.md`, `architecture/*.md` o `research/*.md` según corresponda
2. Para cambios transversales (nueva entidad), actualizar el spec de la entidad + api-contract + architecture/backend.md
3. Mantener `research/future/index.md` actualizado con el estado de las funcionalidades futuras
