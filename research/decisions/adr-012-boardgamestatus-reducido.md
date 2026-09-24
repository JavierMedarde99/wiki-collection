# ADR-012: BoardGameStatus Reducido (OWNED, WISHLIST)

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

El enum BoardGameStatus original tenía 4 valores (OWNED, WISHLIST, PREVIOUSLY_OWNED, FOR_TRADE) pero la implementación real solo usa 2.

## Decisión

Reducir el enum a OWNED y WISHLIST para simplificar el modelo de datos.

## Consecuencias

- ✅ Modelo más simple y alineado con la implementación
- ❌ Menos granularidad en el estado del juego de mesa
