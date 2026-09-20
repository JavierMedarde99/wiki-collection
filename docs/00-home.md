# Wiki — Proyecto Colección

Documentación del proyecto de gestión de colecciones personales.

## Navegación

|| Sección | Descripción ||
||---------|-------------||
|| [01-requisitos](./01-requisitos.md) | Requisitos funcionales y no funcionales ||
|| [02-arquitectura](./02-arquitectura/) | Arquitectura general, backend y frontend ||
|| [03-base-de-datos](./03-base-de-datos/) | Modelo de datos, tablas y relaciones ||
|| [04-autenticacion](./04-autenticacion.md) | Autenticación y autorización ||
|| [05-api](./05-api/) | Endpoints, usuarios, autenticación, APIs externas ||
|| [06-frontend](./06-frontend/) | Estructura, componentes y navegación ||
|| [07-backend](./07-backend/) | Estructura, servicios y persistencia ||
|| [08-deploy](./08-deploy.md) | Despliegue (Atlas + Render + Vercel) ||
|| [09-testing](./09-testing.md) | Testing ||
|| [10-decisiones-tecnicas](./10-decisiones-tecnicas.md) | Decisiones técnicas ||
|| [11-problemas-conocidos](./11-problemas-conocidos.md) | Problemas conocidos ||
|| [12-changelog](./12-changelog.md) | Changelog ||
|| [13-fase-1-libros](./13-fase-1-libros.md) | Fase 1: Libros ||
|| [14-fase-2-juegos](./14-fase-2-juegos.md) | Fase 2: Juegos ||
|| [15-fase-3-juegos-mesa](./15-fase-3-juegos-mesa.md) | Fase 3: Juegos de Mesa ||
|| [16-fase-4-magic](./16-fase-4-magic.md) | Fase 4: Magic: The Gathering ||
|| [17-fase-5-movieshows](./17-fase-5-movieshows.md) | Fase 5: Películas/Series (TMDB) ||
|| [18-fase-7-caffeine-search](./18-fase-7-caffeine-search.md) | Fase 7: Caché Caffeine en búsquedas ||
|| [19-fase-8-autenticacion](./19-fase-8-autenticacion.md) | Fase 8: Autenticación de Usuarios ||
||| [20-fase-9-colecciones-personales](./20-fase-9-colecciones-personales.md) | Fase 9: Colecciones Personales y Visibilidad | ✅ Completada ||
|||| [22-futuro-barcode-scanner](./22-futuro-barcode-scanner.md) | 📋 Futuro: Escáner de código de barras para libros (ISBN) ||
|||| [23-futuro-deck-import.txt](./23-futuro-deck-import.txt.md) | 📋 Futuro: Importar mazos Magic desde texto plano ||
|||| [24-futuro-reading-progress](./24-futuro-reading-progress.md) | 📋 Futuro: Progreso de lectura en libros (páginas leídas + %) ||
|||| [25-futuro-streaming-platforms](./25-futuro-streaming-platforms.md) | 📋 Futuro: Plataformas de streaming en películas/series (TMDB Watch Providers) ||
|||| [26-futuro-generos-colecciones](./26-futuro-generos-colecciones.md) | 📋 Futuro: Géneros en colecciones (Books, Games, BoardGames, MovieShows) ||
|||| [27-futuro-generos-books-games-boardgames](./27-futuro-generos-books-games-boardgames.md) | 📋 Futuro: Géneros y mecánicas en Books, Games y BoardGames (implementación) ||
|||| [28-futuro-boardgame-rating](./28-futuro-boardgame-rating.md) | 📋 Futuro: Calificación personal (stars + comentario) en juegos de mesa (OWNED) ||
|||| [29-futuro-books-wishlist-acquisitiondate](./29-futuro-books-wishlist-acquisitiondate.md) | 📋 Futuro: Wishlist + fecha de adquisición en libros ||


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

## Repositorios

- **wiki-collection** — Este repo, documentación
- **backend-collection** — Java 25 + Spring Boot 4 (https://github.com/JavierMedarde99/backend-collection)
- **frontend-collection** — React + Vite (https://github.com/JavierMedarde99/frontend-collection)

## Estado del Proyecto

|| Fase | Descripción | Estado ||
||------|-------------|--------||
|| Fase 1 | Libros (Google Books) | ✅ Completada ||
|| Fase 2 | Videojuegos (RAWG + FreeToGame + Steam) | ✅ Completada ||
|| Fase 3 | Juegos de Mesa (BoardGameGeek XML) | ✅ Completada ||
|| Fase 4 | Cartas Magic (Scryfall) | ✅ Completada ||
|| Fase 4.1 | Mazos Commander (Scryfall + gestión mazos) | ✅ Completada ||
|| Fase 5 | Películas/Series (TMDB) | ✅ Completada ||
|| Fase 6 | Imágenes (Catbox.moe) | ✅ Completada ||
|| Fase 7 | Caché Caffeine en búsquedas externas | ✅ Completada ||
|| Fase 8 | Autenticación de Usuarios (JWT + Spring Security) | ✅ Completada ||
|| Fase 9 | Colecciones Personales y Visibilidad | ✅ Completada ||
