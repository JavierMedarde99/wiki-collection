# ADR-009: Scryfall para Cartas Magic

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

Necesidad de una API de cartas Magic gratuita y completa.

## Decisión

Usar Scryfall API (70k+ cartas, sin auth, búsqueda fuzzy, precios, legalidades).

## Consecuencias

- ✅ Totalmente gratuita
- ✅ Sin autenticación
- ✅ 70,000+ cartas
- ✅ Datos muy completos
- ❌ Rate limit de ~10 req/segundo
