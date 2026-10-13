# Decisiones del Proyecto — Colección de Amiibo

> Este documento registra las decisiones humanas tomadas durante la especificación de la colección de Amiibo.

## Decisiones Registradas

### DA1: Alcance del SDD — solo especificación

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** En esta iteración el Spec-Driven se limita al **wiki**: OpenSpec change + specs canónicas (`specs/amiibos/`, `data-model/schema.md`, `architecture/backend.md`, `architecture/frontend-*.md`). La implementación en `backend-collection` y `frontend-collection` se hace después, con sus propios PRs y TDD.
- **Justificación:** el flujo del proyecto es spec primero, código después; el usuario prefiere PRs separados por feature.

### DA2: Amiibo y Funko en cambios OpenSpec separados

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** Dos changes independientes: `feat-amiibo-collection` (este) y uno posterior `feat-funko-collection`. **Amiibo primero**.
- **Justificación:** cada colección tiene API, modelo y fases propias. Amiibo es la más barata (sin auth ni secreto, catálogo estable) y sirve de plantilla para Funko.

### DA3: Imágenes por hotlink

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** Guardar la `imageUrl` externa directamente (`raw.githubusercontent.com`), sin descargar ni subir a Catbox.
- **Justificación:** el resto de colecciones ya usan URLs externas (portadas de libros de Google, imágenes de Scryfall). Es consistente y cero infraestructura extra.
- **Riesgo asumido:** enlace muerto si el repo de imágenes de tercero cambia o desaparece. El usuario lo acepta.

### DA4: Colección Mongo separada

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** Colección `AMIIBOS` propia, con entidad `Amiibo` y endpoints `/api/v1/amiibos`. **No** se crea una colección genérica «collectibles» con discriminador `type` (modelo «normalizado» de la research descartado).
- **Justificación:** patrón del proyecto (una colección por tipo), sin acoplamiento con Funko, y cada entidad refleja sus campos reales sin opcionales compartidos.

### DA5: Estados OWNED / WISHLIST

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** `AmiiboStatus` con 2 valores: `OWNED`, `WISHLIST`.
- **Justificación:** mismo patrón que board-games (AD-012 redujo su enum a esos dos). Suficiente para coleccionables físicos.

### DA6: Sin precios y sin código de barras

- **Fecha:** 2026-10-11 (decisión original, ratificada el 2026-10-13)
- **Decisor:** Javier Medarde
- **Decisión:** Fuera de alcance en ambas colecciones nuevas.
- **Justificación:** AmiiboAPI es un catálogo, no un mercado: ni precios ni barcode. HobbyQuantHQ tiene precios «en evolución» que no se documentan.

### DA7: Sin endpoints de catálogo de series

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** No se exponen `/amiiboseries/` ni `/gameseries/` como endpoints de soporte del formulario.
- **Justificación:** el formulario de «añadir a colección» se alimenta del resultado de la búsqueda (que ya trae amiiboSeries y gameSeries). Mantiene el mínimo viable; se puede añadir después si el frontend lo necesita.

### DA8: Fase 31 propuesta, sin asignar

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** La implementación se registraría como **Fase 31**, pero no se asigna ni se marca en `requirements.md`/`README.md` hasta que se priorice.
- **Justificación:** el wiki documenta realidad, no intención. Investigación proponía Fase 30 (Funko) / 31 (Amiibo); los números no están asignados.