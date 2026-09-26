# Funcionalidades Futuras — Wiki-Collection

Esta carpeta contiene investigación sobre funcionalidades planificadas para futuras fases del proyecto.

## Índice

| # | Feature | Estado | Archivo |
|---|---------|--------|---------|
|| 22 | Scanner de código de barras | ✅ **Implementado** | `future/barcode-scanner.md` |
|| 23 | Importación de mazos existentes | 📋 Por hacer | `future/deck-import.md` |
|| 24 | Seguimiento de lectura | ✅ **Implementado (parcial)** | `future/reading-progress.md` |
|| 25 | Plataformas de streaming | ✅ **Implementado (parcial)** | `future/streaming-platforms.md` |
|| 26 | Géneros para colecciones | ✅ **Implementado** | `future/genres-collections.md` |
|| 27 | Géneros específicos para libros/juegos/juegos de mesa | ✅ **Implementado** | `future/genres-books-games-boardgames.md` |
|| 28 | Valoración de juegos de mesa | 📋 Por hacer | `future/boardgame-rating.md` |
|| 29 | Lista de deseos y fecha de adquisición para libros | 📋 Por hacer | `future/books-wishlist.md` |

## Problemas documentados (issues)

### Backend (JavierMedarde99/backend-collection)

| # | Fase | Issue | Estado |
|---|------|-------|--------|
| 353 | 23 | Importación de mazos Commander existentes | 📋 Pendiente |
| 356 | 28 | Valoración personal y estadísticas de uso para juegos de mesa | 📋 Pendiente |
| 357 | 29 | Lista de deseos (WISHLIST) y fecha/precio de adquisición para libros | 📋 Pendiente |
| 420 | 6 | [Spec] Fase 6: Image Storage - crear spec formal (Catbox.moe) — CERRADO (no necesario) | ✅ Cerrado |
| 421 | 7 | [Spec] Fase 7: Caffeine Cache - crear spec formal (6 cachés) — CERRADO (no necesario) | ✅ Cerrado |
| 422 | 25 | Fase 25: Plataformas suscritas, filtrado y deep links — CERRADO (deep links no se aplican) | ✅ Cerrado |
| 423 | 26 | Fase 26: Géneros para colecciones — CERRADO (ya implementado) | ✅ Cerrado |
| 424 | 27 | Fase 27: Géneros específicos por tipo — CERRADO (ya implementado) | ✅ Cerrado |
| 425 | 24 | Fase 24: Historial de sesiones, objetivos y series (backend) | 📋 Pendiente |
| 426 | - | [Spec] user-preferences: actualizar api-contract.md (endpoints reales vs documentados) | 📋 Pendiente |
| 428 | 26 | Añadir campo genres a MovieShow — CERRADO (ya implementado) | ✅ Cerrado |
| 429 | 26 | Añadir campo genres a Book, Game y BoardGame — CERRADO (ya implementado) | ✅ Cerrado |
| 430 | 4 | Fase 4: Añadir endpoints save/update para Magic cards — CERRADO (ya implementado) | ✅ Cerrado |
| 431 | 22 | [Spec] Fase 22: Actualizar spec de libros (pagesRead, isbn, startDate, endDate) | 📋 Pendiente |
| 432 | 4.1 | Fase 4.1: Añadir deckCount a DeckResponse — CERRADO (ya implementado) | ✅ Cerrado |

### Frontend (JavierMedarde99/frontend-collection)

| # | Fase | Issue | Estado |
|---|------|-------|--------|
| 462 | 23 | Fase 23 (frontend): UI de importación masiva de mazos Commander (Scryfall/JSON/local) | 📋 Pendiente |
| 463 | 26 | Fase 26 (frontend): componentes de gestión de géneros — CERRADO (ya implementado) | ✅ Cerrado |
| 482 | 25 | Fase 25: Plataformas suscritas, filtrado y deep links (frontend) — CERRADO (deep links no se aplican) | ✅ Cerrado |
| 485 | 28 | Fase 28: Valoración personal y estadísticas de uso para juegos de mesa (frontend) | 📋 Pendiente |
| 486 | 29 | Fase 29: Lista de deseos (WISHLIST) y fecha/precio de adquisición (frontend) | 📋 Pendiente |
| 487 | 27 | Fase 27: Géneros específicos por tipo (frontend) — CERRADO (ya implementado) | ✅ Cerrado |
| 488 | 24 | Fase 24: Historial de sesiones, objetivos y series (frontend) | 📋 Pendiente |
| 489 | - | [Spec] Actualizar documentación para reflejar código real del frontend | 📋 Pendiente |
| 490 | 22 | [Spec] Crear spec formal para Fase 22 (Barcode Scanner) — Implementado | 📋 Pendiente |
| 491 | 4.1 | Fase 4.1: Añadir deckCount a tipos y componentes (frontend) — CERRADO (ya implementado) | ✅ Cerrado |

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