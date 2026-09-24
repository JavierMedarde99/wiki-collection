# ADR-005: Google Books como Primaria para Libros

**Fecha:** 2024-01-01
**Estado:** Aceptada

## Contexto

Necesidad de una API de libros con buena cobertura.

## Decisión

Usar Google Books API como primaria, Open Library como fallback.

## Consecuencias

- ✅ Millones de libros
- ✅ Datos de calidad
- ⚠️ Requiere API key para producción
- ❌ Rate limit sin key
