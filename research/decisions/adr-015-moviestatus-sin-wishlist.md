# ADR-015: MovieStatus sin WISHLIST

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

El enum MovieStatus original incluía WISHLIST pero la implementación final no lo requiere.

## Decisión

Reducir el enum a WATCHING, WATCHED, PLAN_TO_WATCH para alinear con el dominio de visualización.

## Consecuencias

- ✅ Modelo más simple y claro
- ✅ Alineado con la funcionalidad real
- ❌ Menos granularidad en la intención de visualización
