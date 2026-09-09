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

## AD-004: RAWG como Principal para Juegos (reemplaza FreeToGame)

**Fecha:** 2024-01-15 (actualizada 2026)
**Estado:** Aceptada

**Contexto:** FreeToGame no soporta búsqueda por nombre y su catálogo es limitado (~415 juegos).

**Decisión:** Usar RAWG como API principal (búsqueda por nombre nativa, 500k+ juegos, 100k req/mes gratis). FreeToGame se mantiene como fallback.

**Consecuencias:**
- ✅ Búsqueda por nombre nativa
- ✅ 500,000+ juegos
- ✅ Datos muy completos
- ⚠️ Requiere API key para producción

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

## AD-008: BGG XML API 2 como Única para Juegos de Mesa

**Fecha:** 2026-09
**Estado:** Aceptada

**Contexto:** Necesidad de una API de juegos de mesa con buena cobertura.

**Decisión:** Usar BGG XML API 2 como única fuente (100k+ juegos, parseo XML con Jackson). Se descartó BGG JSON API por ser no oficial e inestable.

**Consecuencias:**
- ✅ API oficial mantenida por BGG
- ✅ 100,000+ juegos
- ✅ Datos muy completos
- ❌ Formato XML (requiere parseo)
- ❌ API asíncrona (devuelve 202 Accepted)

## AD-009: Scryfall para Cartas Magic

**Fecha:** 2026-09
**Estado:** Aceptada

**Contexto:** Necesidad de una API de cartas Magic gratuita y completa.

**Decisión:** Usar Scryfall API (70k+ cartas, sin auth, búsqueda fuzzy, precios, legalidades).

**Consecuencias:**
- ✅ Totalmente gratuita
- ✅ Sin autenticación
- ✅ 70,000+ cartas
- ✅ Datos muy completos
- ❌ Rate limit de ~10 req/segundo

## AD-010: Spring Boot 4.1.1 + Java 25

**Fecha:** 2026-09
**Estado:** Aceptada

**Contexto:** Actualización del stack backend.

**Decisión:** Usar Spring Boot 4.1.1 con Java 25 y Spring Data MongoDB.

**Consecuencias:**
- ✅ Últimas características de Java
- ✅ Spring Boot 4 con soporte para Java 25
- ✅ Mejoras de rendimiento
- ⚠️ Requiere JDK 25+

## AD-011: Jacoco para Cobertura de Tests

**Fecha:** 2026-09
**Estado:** Aceptada

**Contexto:** Necesidad de medir y asegurar la calidad de los tests.

**Decisión:** Usar Jacoco con umbral mínimo de 80% de cobertura de línea.

**Consecuencias:**
- ✅ Medición automática de cobertura
- ✅ Gate de calidad en CI
- ❌ Puede ser restrictivo inicialmente

## AD-012: BoardGameStatus Reducido (OWNED, WISHLIST)

**Fecha:** 2026-09
**Estado:** Aceptada

**Contexto:** El enum BoardGameStatus original tenía 4 valores (OWNED, WISHLIST, PREVIOUSLY_OWNED, FOR_TRADE) pero la implementación real solo usa 2.

**Decisión:** Reducir el enum a OWNED y WISHLIST para simplificar el modelo de datos.

**Consecuencias:**
- ✅ Modelo más simple y alineado con la implementación
- ❌ Menos granularidad en el estado del juego de mesa
