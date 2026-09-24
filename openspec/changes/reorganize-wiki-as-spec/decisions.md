# Decisiones del Proyecto — Reorganización Wiki-Collection

> Este documento registra las decisiones humanas tomadas durante la reorganización del wiki como base de conocimiento spec-driven.

## Decisiones Registradas

### DF1: Estado de la Fase 9

- **Fecha:** 2026-09-24
- **Decisor:** Javier Medarde
- **Decisión:** La Fase 9 está **completada**.
- **Justificación:** Aunque el documento original `20-fase-9-colecciones-personales.md` marcaba su estado interno como "📋 PLANIFICADA", el índice `00-home.md` la refleja como ✅ completada. Para coherencia, la spec reflejará el estado real completada.
- **Acción:** Al crear `docs/specs/user-preferences/spec.md`, usar checkmarks `[x]` para todos los criterios de aceptación y marcar el estado como ✅ Implementado.

### DF2: Archivos de fase (13-21)

- **Fecha:** 2026-09-24
- **Decisor:** Javier Medarde
- **Decisión:** Los archivos de fase se convierten en **stubs** con redirect.
- **Justificación:** Preservar el contenido histórico de implementación (checklists, estructuras de paquetes, criterios de aceptación) sin duplicar información en la nueva estructura.
- **Acción:** Cada archivo de fase se reescribe con una breve descripción y enlace: "Ver especificación actualizada: `specs/collection/X/`".

### DF3: `docs/01-requisitos.md`

- **Fecha:** 2026-09-24
- **Decisor:** Javier Medarde
- **Decisión:** Se **mantiene** en su ubicación actual.
- **Justificación:** Es el documento de nivel más alto con requisitos funcionales y no funcionales del sistema. Tiene valor como resumen ejecutivo independiente de las specs detalladas.
- **Acción:** 
  - Se actualiza la tabla de navegación en `00-home.md` para enlazarlo.
  - Se verifica que los requisitos en `01-requisitos.md` sean consistentes con las specs creadas. Si hay inconsistencias, se actualiza `01-requisitos.md` para alinearlo.