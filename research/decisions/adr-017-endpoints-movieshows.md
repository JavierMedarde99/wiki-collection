# ADR-017: Endpoints base renombrados a /api/movieshows

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

El endpoint original `/api/movies` era ambiguo y no reflejaba que gestiona películas Y series.

## Decisión

Renombrar a `/api/movieshows` para mayor claridad semántica.

## Consecuencias

- ✅ Nombre más descriptivo
- ✅ Consistencia con el modelo MovieShow
- ❌ Rompe compatibilidad con clientes anteriores
