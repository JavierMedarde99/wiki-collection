# ADR-008: BGG XML API 2 como Única para Juegos de Mesa

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

Necesidad de una API de juegos de mesa con buena cobertura.

## Decisión

Usar BGG XML API 2 como única fuente (100k+ juegos, parseo XML con Jackson). Se descartó BGG JSON API por ser no oficial e inestable.

## Consecuencias

- ✅ API oficial mantenida por BGG
- ✅ 100,000+ juegos
- ✅ Datos muy completos
- ❌ Formato XML (requiere parseo)
- ❌ API asíncrona (devuelve 202 Accepted)
