# Wiki — Proyecto Colección

Documentación del proyecto de gestión de colecciones personales.

## Navegación

| Sección | Descripción |
|---------|-------------|
| [01-requisitos](./01-requisitos.md) | Requisitos funcionales y no funcionales |
| [02-arquitectura](./02-arquitectura/) | Arquitectura general, backend y frontend |
| [03-base-de-datos](./03-base-de-datos/) | Modelo de datos, tablas y relaciones |
| [04-autenticacion](./04-autenticacion.md) | Autenticación y autorización |
| [05-api](./05-api/) | Endpoints, usuarios, autenticación, APIs externas |
| [06-frontend](./06-frontend/) | Estructura, componentes y navegación |
| [07-backend](./07-backend/) | Estructura, servicios y persistencia |
| [08-deploy](./08-deploy.md) | Despliegue |
| [09-testing](./09-testing.md) | Testing |
| [10-decisiones-tecnicas](./10-decisiones-tecnicas.md) | Decisiones técnicas |
| [11-problemas-conocidos](./11-problemas-conocidos.md) | Problemas conocidos |
| [12-changelog](./12-changelog.md) | Changelog |
| [13-fase-1-libros](./13-fase-1-libros.md) | Fase 1: Libros |
| [14-fase-2-juegos](./14-fase-2-juegos.md) | Fase 2: Juegos |
| [15-fase-3-juegos-mesa](./15-fase-3-juegos-mesa.md) | Fase 3: Juegos de Mesa |
| [16-fase-4-magic](./16-fase-4-magic.md) | Fase 4: Magic: The Gathering |
| [17-fase-5-movieshows](./17-fase-5-movieshows.md) | Fase 5: Películas/Series (TMDB) |

## Stack Tecnológico

| Capa | Tecnología |
|------|------------|
| Backend | Java 25 + Spring Boot 4.1.1 |
| Base de datos | MongoDB + Spring Data |
| Frontend | React 18.3 + Vite 5 + Tailwind 3 + TypeScript 5 |
| APIs externas | Google Books / RAWG / FreeToGame / BoardGameGeek XML / Scryfall / Steam / TMDB |
| Documentación | springdoc-openapi (Swagger 3.1.0) |
| Testing | JUnit 5 + Mockito + MockWebServer + Jacoco (80% cobertura) |
| Build | Maven (backend) + npm (frontend) |

## Repositorios

- **wiki-collection** — Este repo, documentación
- **backend-collection** — Java 25 + Spring Boot 4 (https://github.com/JavierMedarde99/backend-collection)
- **frontend-collection** — React + Vite (https://github.com/JavierMedarde99/frontend-collection)

## Estado del Proyecto

| Fase | Descripción | Estado |
|------|-------------|--------|
| Fase 1 | Libros (Google Books) | ✅ Completada |
| Fase 2 | Videojuegos (RAWG + FreeToGame + Steam) | ✅ Completada |
| Fase 3 | Juegos de Mesa (BoardGameGeek XML) | ✅ Completada |
| Fase 4 | Cartas Magic (Scryfall) | ✅ Completada |
| Fase 4.1 | Mazos Commander (Scryfall + gestión mazos) | ✅ Completada |
| Fase 5 | Películas/Series (TMDB) | ✅ Completada |
| Fase 6 | Imágenes (Catbox.moe) | ✅ Completada |
| Auth | Autenticación de usuarios | 📋 Planificada |
