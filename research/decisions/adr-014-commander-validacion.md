# ADR-014: Mazos Commander con Validación de Reglas

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

Los usuarios necesitan gestionar mazos Commander de Magic y validar que cumplen las reglas del formato.

## Decisión

Implementar DeckValidator que evalúa el estado del mazo (DRAFT, COMPLETE, INVALID) según número de cartas, identidad de color del comandante, etc.

## Consecuencias

- ✅ Validación automática de reglas Commander
- ✅ Feedback claro al usuario (razones de invalidación)
- ⚠️ Las reglas pueden cambiar con nuevas ediciones
