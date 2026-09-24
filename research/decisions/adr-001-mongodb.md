# ADR-001: MongoDB sobre SQL

**Fecha:** 2024-01-01
**Estado:** Aceptada

## Contexto

Los items de colección tienen atributos variables (un libro tiene autor, un juego tiene plataforma).

## Decisión

Usar MongoDB para permitir esquemas flexibles por entidad.

## Consecuencias

- ✅ Flexibilidad de esquemas
- ✅ Escalabilidad horizontal
- ❌ No hay joins nativos
- ❌ No hay integridad referencial
