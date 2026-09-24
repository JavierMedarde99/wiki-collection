# ADR-013: TMDB como API para Películas/Series

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

Necesidad de una API de películas/series con buena cobertura y búsqueda por tipo.

## Decisión

Usar TMDB API (películas + series, búsqueda por nombre, paginación, imágenes, puntuaciones). Se descartó OMDb (limitado a 1000/día) y TVMaze (sin paginación).

## Consecuencias

- ✅ API oficial con catálogo masivo
- ✅ Búsqueda por nombre nativa
- ✅ Soporte para películas Y series
- ✅ Imágenes incluidas
- ⚠️ Requiere API key para producción
- ⚠️ Rate limit de ~40 req/segundo
