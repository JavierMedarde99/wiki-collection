# Decisiones Técnicas

## AD-001: MongoDB sobre SQL

**Fecha:** 2024-01-01
**Estado:** Aceptada

**Contexto:** Los items de colección tienen atributos variables (un libro tiene autor, un juego tiene plataforma).

**Decisión:** Usar MongoDB para permitir esquemas flexibles por entidad.

**Consecuencias:**
- ✅ Flexibilidad de esquemas
- ✅ Escalabilidad horizontal
- ❌ No hay joins nativos
- ❌ No hay integridad referencial

## AD-002: Arquitectura Hexagonal

**Fecha:** 2024-01-01
**Estado:** Aceptada

**Contexto:** Necesidad de separar el dominio de frameworks y tecnologías externas.

**Decisión:** Implementar arquitectura hexagonal (puertos y adaptadores).

**Consecuencias:**
- ✅ Testabilidad
- ✅ Mantenibilidad
- ✅ Independencia de frameworks
- ❌ Más código boilerplate
- ❌ Curva de aprendizaje

## AD-003: Búsqueda Externa + BD Local

**Fecha:** 2024-01-01
**Estado:** Aceptada

**Contexto:** Los usuarios necesitan buscar libros/juegos para añadir a su colección.

**Decisión:** Usar APIs externas para búsqueda, guardar en BD local con datos enriquecidos.

**Consecuencias:**
- ✅ Acceso a millones de items
- ✅ Datos locales personalizables
- ❌ Dependencia de APIs externas
- ❌ Datos externos pueden cambiar

## AD-004: FreeToGame para Juegos

**Fecha:** 2024-01-15
**Estado:** Aceptada

**Contexto:** Necesidad de una API gratuita de videojuegos sin autenticación.

**Decisión:** Usar FreeToGame API (gratis, sin API key, ~415 juegos).

**Consecuencias:**
- ✅ Sin autenticación
- ✅ 100% gratis
- ❌ No soporta búsqueda por nombre
- ❌ Catálogo limitado (~415 juegos)

**Mitigación:** Implementar filtrado por nombre en el backend.

## AD-005: Google Books como Primaria para Libros

**Fecha:** 2024-01-01
**Estado:** Aceptada

**Contexto:** Necesidad de una API de libros con buena cobertura.

**Decisión:** Usar Google Books API como primaria, Open Library como fallback.

**Consecuencias:**
- ✅ Millones de libros
- ✅ Datos de calidad
- ⚠️ Requiere API key para producción
- ❌ Rate limit sin key

## AD-006: Repositorios Separados

**Fecha:** 2024-01-01
**Estado:** Aceptada

**Contexto:** Backend y frontend tienen ciclos de vida diferentes.

**Decisión:** Repositorios separados para backend, frontend y documentación.

**Consecuencias:**
- ✅ CI/CD independiente
- ✅ Despliegue independiente
- ❌ Gestión de múltiples repos
- ❌ Sincronización de issues

## AD-007: Wiki en Repo Separada

**Fecha:** 2024-01-01
**Estado:** Aceptada

**Contexto:** Necesidad de versionar la documentación junto al código.

**Decisión:** Toda la documentación vive en `docs/` en un repo separado.

**Consecuencias:**
- ✅ Documentación versionada
- ✅ PRs para cambios en docs
- ❌ No vive junto al código
