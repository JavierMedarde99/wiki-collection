---
type: adr
title: "ADR-002: Arquitectura Hexagonal"
description: "Arquitectura hexagonal para aislar el dominio"
tags: [adr, arquitectura]
---

# ADR-002: Arquitectura Hexagonal

**Fecha:** 2024-01-01
**Estado:** Aceptada

## Contexto

Necesidad de separar el dominio de frameworks y tecnologías externas.

## Decisión

Implementar arquitectura hexagonal (puertos y adaptadores).

## Consecuencias

- ✅ Testabilidad
- ✅ Mantenibilidad
- ✅ Independencia de frameworks
- ❌ Más código boilerplate
- ❌ Curva de aprendizaje
