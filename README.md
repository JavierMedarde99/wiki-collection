# Wiki-Collection

Repositorio de documentación para el proyecto Wiki-Collection.

## Proyecto

Wiki-Collection es una aplicación web para gestionar colecciones personales de libros, videojuegos, juegos de mesa, cartas Magic: The Gathering, películas/series y mazos Commander.

## Stack Tecnológico

|| Capa | Tecnología ||
||------|------------||
|| Backend | Java 25 + Spring Boot 4.1.1 ||
|| Base de datos | MongoDB + Spring Data ||
|| Frontend | React 18.3 + Vite 5 + Tailwind 3 + TypeScript 5 ||
|| APIs externas | Google Books / RAWG / FreeToGame / BoardGameGeek XML / Scryfall / Steam / TMDB ||
|| Documentación | springdoc-openapi (Swagger 3.1.0) ||
|| Testing | JUnit 5 + Mockito + MockWebServer + Jacoco (80% cobertura) ||
|| Build | Maven (backend) + npm (frontend) ||
|| Caché | Spring Cache + Caffeine (6 cachés en memoria) ||
|| Seguridad | Spring Security + JJWT 0.12.6 (JWT) ||

## Repositorios

|| Repo | Descripción | Estado ||
||------|-------------|--------||
|| [backend-collection](https://github.com/JavierMedarde99/backend-collection) | Java 25 + Spring Boot 4 | ✅ Activo ||
|| [frontend-collection](https://github.com/JavierMedarde99/frontend-collection) | React + Vite | ✅ Activo ||
|| [wiki-collection](https://github.com/JavierMedarde99/wiki-collection) | Este repo, documentación | ✅ Activo ||

## Estado de las Fases

|| Fase | Descripción | Estado ||
||------|-------------|--------||
|| Fase 1 | Libros (Google Books API) | ✅ Completada ||
|| Fase 2 | Videojuegos (RAWG + FreeToGame + Steam) | ✅ Completada ||
|| Fase 3 | Juegos de Mesa (BoardGameGeek XML) | ✅ Completada ||
|| Fase 4 | Cartas Magic: The Gathering (Scryfall) | ✅ Completada ||
|| Fase 4.1 | Mazos Commander (Scryfall + gestión mazos) | ✅ Completada ||
|| Fase 5 | Películas/Series (TMDB) | ✅ Completada ||
|| Fase 6 | Imágenes (Catbox.moe) | ✅ Completada ||
|| Fase 7 | Caché Caffeine en búsquedas externas | ✅ Completada ||
|| Fase 8 | Autenticación de usuarios (JWT + Spring Security) | ✅ Completada ||
|| Fase 9 | Colecciones Personales y Visibilidad | ✅ Completada ||

## Estructura de la Documentación

```
docs/
├── 00-home.md                        # Índice principal
├── 01-requisitos.md                  # Requisitos funcionales y no funcionales
├── 02-arquitectura/                  # Arquitectura general
│   ├── README.md                     # Diagrama de alto nivel
│   ├── 02.1-backend.md              # Estructura hexagonal del backend
│   └── 02.2-frontend.md             # Estructura React del frontend
├── 03-base-de-datos/                 # Modelo de datos MongoDB
│   ├── README.md                     # Índice de colecciones
│   ├── 03.1-tablas.md               # Esquemas de entidades
│   └── 03.2-relaciones.md           # Relaciones entre entidades
├── 04-autenticacion.md               # Autenticación
├── 05-api/                           # Documentación de endpoints
│   ├── README.md                     # Todos los endpoints + DTOs
│   ├── 05.1-usuarios.md             # Endpoints de usuarios
│   ├── 05.2-autenticacion.md        # Endpoints de autenticación
│   └── externos/                     # Documentación de APIs externas
│       ├── externas-books.md         # Google Books API
│       ├── externas-videogames.md    # RAWG + FreeToGame
│       ├── externas-steam.md         # Steam Web API
│       ├── externas-boardgames.md    # BoardGameGeek XML
│       ├── externas-magic.md         # Scryfall
│       └── externas-movies.md        # TMDB
├── 06-frontend/                      # Documentación del frontend
│   ├── README.md                     # Páginas + stack
│   ├── 06.1-componentes.md          # Catálogo de componentes
│   └── 06.2-navegacion.md           # Rutas + navegación
├── 07-backend/                       # Documentación del backend
│   ├── README.md                     # Estructura de paquetes + endpoints + excepciones + configs + cachés
│   ├── 07.1-servicios.md            # Servicios + clientes externos
│   └── 07.2-persistencia.md         # Repositories + mappers
├── 08-deploy.md                      # Plan de despliegue
├── 09-testing.md                     # Inventario de tests
├── 10-decisiones-tecnicas.md         # ADRs del proyecto
├── 11-problemas-conocidos.md         # Known issues
├── 12-changelog.md                   # Historial de versiones
├── 13-fase-1-libros.md               # Plan: Libros
├── 14-fase-2-juegos.md               # Plan: Juegos
├── 15-fase-3-juegos-mesa.md          # Plan: Juegos de Mesa
├── 16-fase-4-magic.md                # Plan: Magic
├── 17-fase-5-movieshows.md           # Plan: Películas/Series
├── 18-fase-7-caffeine-search.md      # Plan: Caché Caffeine
├── 19-fase-8-autenticacion.md        # Plan: Autenticación
└── 20-fase-9-colecciones-personales.md # Plan: Colecciones Personales
```

## APIs Externas Integradas

|| API | Uso | Fase | Estado ||
||-----|-----|------|--------||
|| Google Books | Búsqueda de libros | 1 | ✅ ||
|| RAWG | Búsqueda de videojuegos | 2 | ✅ ||
|| FreeToGame | Fallback videojuegos | 2 | ✅ ||
|| Steam Web API | Logros de juegos | 2 | ✅ ||
|| BoardGameGeek XML | Búsqueda juegos de mesa | 3 | ✅ ||
|| Scryfall | Búsqueda cartas Magic | 4 | ✅ ||
|| TMDB | Búsqueda películas/series | 5 | ✅ ||
|| Catbox.moe | Hosting de imágenes | 6 | ✅ ||

## Convenciones

- **Idioma:** Toda la documentación está en español
- **Formato:** Solo markdown, sin HTML ni diagramas como código
- **Nombres de repos:** Usar `backend-collection` y `frontend-collection` (nombres reales en GitHub)

## Cómo Contribuir

1. Editar el archivo `docs/*.md` correspondiente
2. Actualizar `docs/00-home.md` si se añade un nuevo documento
3. Para cambios transversales (nueva entidad), actualizar tanto `03-base-de-datos/03.1-tablas.md` como `05-api/README.md`
