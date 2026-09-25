# Funcionalidades Futuras — Wiki-Collection

Esta carpeta contiene investigación sobre funcionalidades planificadas para futuras fases del proyecto.

## Índice

| # | Feature | Estado | Archivo |
|---|---------|--------|---------|
| 22 | Scanner de código de barras | ✅ **Implementado** (libros) · 📋 Pendiente (board games, juegos) | `future/barcode-scanner.md` |
| 23 | Importación de mazos existentes | 📋 Por hacer | `future/deck-import.md` |
| 24 | Seguimiento de lectura | ✅ **Implementado** (parcial: páginas leídas, fechas, barra progreso) · 📋 Pendiente (sesiones, objetivos, series) | `future/reading-progress.md` |
| 25 | Plataformas de streaming | ✅ **Implementado** (parcial: TMDB streaming providers, badges, refresh) · 📋 Pendiente (plataformas suscritas, filtrado, notificaciones) | `future/streaming-platforms.md` |
| 26 | Géneros para colecciones | 📋 Por hacer | `future/genres-collections.md` |
| 27 | Géneros específicos para libros/juegos/juegos de mesa | 📋 Por hacer (depende de 26) | `future/genres-books-games-boardgames.md` |
| 28 | Valoración de juegos de mesa | 📋 Por hacer | `future/boardgame-rating.md` |
| 29 | Lista de deseos y fecha de adquisición para libros | 📋 Por hacer | `future/books-wishlist.md` |

## Problemas documentados (issues)

### Backend (JavierMedarde99/backend-collection)

| # | Fase | Issue | Estado |
|---|------|-------|--------|
| 352 | 22 (resto) | Extender Barcode Scanner a Board Games y Videojuegos | 📋 Pendiente |
| 353 | 23 | Importación de mazos Commander existentes | 📋 Pendiente |
| 354 | 26 | Añadir campos de género a colecciones | 📋 Pendiente |
| 355 | 27 | Géneros específicos por tipo de colección | 📋 Pendiente (depende de 26) |
| 356 | 28 | Valoración personal y estadísticas de uso para juegos de mesa | 📋 Pendiente |
| 357 | 29 | Lista de deseos (WISHLIST) y fecha/precio de adquisición para libros | 📋 Pendiente |
| 361 | 25 (resto) | Añadir `deepLinkUrl` a `StreamingProvider` y mejorar auto-llenado de géneros | 📋 Pendiente |
| 362 | 24 (resto) | Historial de sesiones de lectura (ReadingSession), objetivos anuales (ReadingGoal) y series de libros | 📋 Pendiente |
| 363 | 25 (resto) | Marcar plataformas suscritas por usuario y filtrado de movieshows por plataformas | 📋 Pendiente |

### Frontend (JavierMedarde99/frontend-collection)

| # | Fase | Issue | Estado |
|---|------|-------|--------|
| 458 | 22 (resto) | Extender Barcode Scanner a Board Games y Videojuegos | 📋 Pendiente |
| 459 | 23 | UI de importación masiva de mazos Commander (Scryfall/JSON/local) | 📋 Pendiente |
| 460 | 24 (resto) | Historial de sesiones de lectura, objetivos anuales y series de libros | 📋 Pendiente |
| 461 | 25 (resto) | Plataformas suscritas, filtrado por plataformas y deep links | 📋 Pendiente |
| 462 | 23 | UI de importación masiva de mazos Commander (Scryfall/JSON/local) | 📋 Pendiente |
| 463 | 26 | Componentes de gestión de géneros (checkboxes, badges, filtros) | 📋 Pendiente |

## Patrón

Cada funcionalidad futura sigue este formato:

1. **Título** — Nombre claro de la funcionalidad
2. **Descripción** — Qué hace y por qué es útil
3. **Capacidades afectadas** — Qué entities/models se ven afectados
4. **APIs externas necesarias** — Si requiere nuevas integraciones
5. **Consideraciones de diseño** — Decisión técnica, impacto en arquitectura
6. **Prioridad estimada** — Baja / Media / Alta

---

|*Esta documentación es informativa y no representa un compromiso de implementación. Las decisiones finales se documentan en `research/decisions/` cuando se priorizan.*