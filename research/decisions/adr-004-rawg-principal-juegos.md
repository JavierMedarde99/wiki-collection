# ADR-004: RAWG como Principal para Juegos

**Fecha:** 2024-01-15 (actualizada 2026)
**Estado:** Aceptada

## Contexto

FreeToGame no soporta búsqueda por nombre y su catálogo es limitado (~415 juegos).

## Decisión

Usar RAWG como API principal (búsqueda por nombre nativa, 500k+ juegos, 100k req/mes gratis). FreeToGame se mantiene como fallback.

## Consecuencias

- ✅ Búsqueda por nombre nativa
- ✅ 500,000+ juegos
- ✅ Datos muy completos
- ⚠️ Requiere API key para producción
