---
type: adr
title: "ADR-011: JaCoCo para Cobertura de Tests"
description: "JaCoCo con umbral de cobertura del 80 por ciento"
tags: [adr, testing]
---

# ADR-011: JaCoCo para Cobertura de Tests

**Fecha:** 2026-09
**Estado:** Aceptada

## Contexto

Necesidad de medir y asegurar la calidad de los tests.

## Decisión

Usar Jacoco con umbral mínimo de 80% de cobertura de línea.

## Consecuencias

- ✅ Medición automática de cobertura
- ✅ Gate de calidad en CI
- ❌ Puede ser restrictivo inicialmente
