# Decisiones del Proyecto — Colección de Funko Pop

> Este documento registra las decisiones humanas tomadas durante la especificación de la colección de Funko Pop.

## Decisiones Registradas

### DF1: Imágenes por hotlink

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** Guardar la `imageUrl` externa directamente (`images.hobbydb.com`), sin descargar ni subir a Catbox.
- **Justificación:** es la decisión ya tomada para Amiibo y el resto de colecciones (portadas de Google, imágenes de Scryfall). Consistente y sin infraestructura extra.
- **Riesgo asumido:** enlace muerto si HobbyDB cambia o retira las imágenes. El usuario lo acepta.

### DF2: Estados OWNED / WISHLIST

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** `FunkoStatus` con 2 valores: `OWNED`, `WISHLIST`.
- **Justificación:** mismo patrón que board-games (AD-012) y Amiibo. Suficiente para coleccionables físicos.

### DF3: Colección Mongo separada

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** Colección `FUNKOS` propia, con entidad `Funko` y endpoints `/api/v1/funkos`. **No** se crea una colección genérica «collectibles» con discriminador `type` (modelo «normalizado» de la research descartado).
- **Justificación:** patrón del proyecto (una colección por tipo), sin acoplamiento con Amiibo, y cada entidad refleja sus campos reales sin opcionales compartidos.

### DF4: Sin precios y sin código de barras

- **Fecha:** 2026-10-11 (decisión original, ratificada el 2026-10-13)
- **Decisor:** Javier Medarde
- **Decisión:** Fuera de alcance.
- **Justificación:** HobbyQuantHQ no expone barcode y sus precios están «en evolución» (no se documentan). El `rarityScore` de la API es un índice de la fuente, no un valor de mercado.

### DF5: Change separado y posterior a Amiibo

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** Change OpenSpec propio (`feat-funko-collection`), **después** de `feat-amiibo-collection`.
- **Justificación:** cada colección tiene API, modelo y fases propias. Amiibo es la más barata y sirve de plantilla; Funko añade API key y una fuente en beta.

### DF6: Categorías sí, navegación de catálogo no

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** Se expone `GET /api/v1/funkos/categories` (proxy de `/categories`) para poder poblar un filtro de categorías. **No** se expone un endpoint de navegación del catálogo completo (`/pops`, 15.458 pops).
- **Justificación:** el flujo de la colección es buscar → añadir; el mínimo viable con las categorías cubre el filtro del frontend. Se puede añadir navegación después si se necesita.

### DF7: `externalId` como String

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** El `id` entero de HobbyQuantHQ se guarda como **String** en `externalId`.
- **Justificación:** consistente con el `externalId` String del resto de colecciones y con la maquinaria de unicidad existente. Evita un tipo distinto solo para Funko.

### DF8: Fase 30 propuesta, sin asignar

- **Fecha:** 2026-10-13
- **Decisor:** Javier Medarde
- **Decisión:** La implementación se registraría como **Fase 30**, pero no se asigna ni se marca en `requirements.md`/`README.md` hasta que se priorice.
- **Justificación:** el wiki documenta realidad, no intención. La investigación proponía Fase 30 (Funko) / 31 (Amiibo); Amiibo se especificó primero pero los números no están asignados.
