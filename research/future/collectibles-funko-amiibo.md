# Funko Pop y Amiibo — nuevas colecciones

**Estado:** ✅ Implementado — sin fase asignada en la wiki
**Fecha:** 2026-10-13 (investigación) / **verificado 2026-10-15** (el código de `main` ya consume ambas APIs)
**Decisiones asociadas:** se tratan como **nuevas colecciones del producto**; **sin precios**; estrategia de imágenes pendiente; se descartó la integración de figuras (MyFigureCollection).

## Descripción

Dos colecciones nuevas, distintas de las actuales (libros, juegos, juegos de mesa, Magic, mazos, películas/series):

- **Funko Pop** — vinilos Pop!, con serie, número, exclusividad, Vaulted e índice de rareza.
- **Amiibo** — figuras/cartas con chip NFC de Nintendo, con personaje, series y juegos compatibles.

## Resumen de Viabilidad

| Colección | Fuente | Viable | Motivo |
|-----------|--------|:------:|--------|
| **Funko** | [HobbyQuantHQ](../external-apis/hobbyquanthq.md) (REST) | ✅ **Sí** | API REST con API key: mismo patrón que RAWG / Google Books |
| **Amiibo** | [AmiiboAPI](../external-apis/amiiboapi.md) (REST) | ✅ **Sí** | API REST pública, **sin auth ni key**, con CORS abierto |

Ambas encajan en la arquitectura actual. **Ninguna tiene búsqueda por código de barras** ni datos de precio.

## Descartado

- **Figuras (MyFigureCollection):** sin API oficial y con la única vía (librería Python local que scrapea HTML) bloqueada por Cloudflare en servidores. **Descartado** el 2026-10-13.

## Capacidades Afectadas

- **Entidades nuevas:** `Funko` y `Amiibo`.
- **Controllers:** `/api/v1/funkos`, `/api/v1/amiibos`.
- **Visibilidad:** filtro `owner` (`mine`/`other`/`all`), colecciones activas y estadísticas — igual que el resto.
- **Caché:** nuevas cachés `funkoSearch`, `funkoList`, `funkoDetail`, `amiiboSearch`, `amiiboList`, `amiiboDetail`…
- **Frontend:** páginas de listado/detalle, búsqueda y formulario. **Sin escáner de barcode** (ninguna fuente lo ofrece).
- **Docs a tocar si se prioriza:** `data-model/schema.md`, `architecture/backend.md`, `architecture/frontend-*.md`, `README.md`, `requirements.md`.

## APIs Externas Necesarias

| Colección | Ficha | Acceso |
|-----------|-------|--------|
| Funko | [hobbyquanthq.md](../external-apis/hobbyquanthq.md) | REST + header `x-api-key` (beta: `dev-key-change-me`) |
| Amiibo | [amiiboapi.md](../external-apis/amiiboapi.md) | REST pública, sin auth |

## Modelo de Datos Propuesto (normalizado)

**Funko:**

```json
{
  "id": "internal-456",
  "type": "funko",
  "source": "hobbyquanthq",
  "externalId": 1234,
  "name": "Batman",
  "series": "DC Comics",
  "number": "01",
  "category": "Comics",
  "exclusive": null,
  "vaulted": true,
  "releaseYear": 2012,
  "franchise": null,
  "popType": null,
  "rarityScore": 30,
  "imageUrl": null
}
```

**Amiibo:**

```json
{
  "id": "internal-789",
  "type": "amiibo",
  "source": "amiiboapi",
  "externalId": "0000000000000002",
  "name": "Mario",
  "character": "Mario",
  "amiiboSeries": "Super Smash Bros.",
  "gameSeries": "Super Mario",
  "amiiboType": "Figure",
  "releaseDates": { "au": "2014-11-29", "eu": "2014-11-28", "jp": "2014-12-06", "na": "2014-11-21" },
  "imageUrl": null
}
```

Ambos suman los campos comunes de colección (propietario, estado, notas, fecha de adquisición, imagen propia…).

## Consideraciones de Diseño

1. **Las dos siguen el patrón existente:** cliente HTTP, caché Caffeine, mapeo a dominio y filtro `owner`. Riesgo bajo.
2. **Amibo es el más simple de los dos:** no necesita API key ni secreto, así que no añade configuración sensible.
3. **Ojo con los filtros de AmiiboAPI:** distinguen mayúsculas (`amiiboSeries` sí, `amiiboseries` no; `gameseries` sí, `gameSeries` no) e ignoran parámetros desconocidos en silencio. El cliente debe construir los parámetros exactos, no dejarlos al usuario.
4. **Código de barras: descartado** para ambas (ninguna fuente lo expone).
5. **Imágenes (decisión pendiente):** Funko entrega URLs de `images.hobbydb.com` y Amiibo de `raw.githubusercontent.com` (repo de un tercero). Falta decidir si se descargan y suben a Catbox (consistente con el proyecto) o se hace hotlink.
6. **Precios: fuera de alcance** (decisión del 2026-10-11).

## Prioridad Estimada

| Colección | Prioridad | Comentario |
|-----------|-----------|------------|
| Amiibo | **Media-Alta** | Catálogo estable, sin auth, sin secretos: la integración más barata |
| Funko | **Media** | Integración de bajo riesgo, pero la API está en beta y sin garantía de contrato |

## Fases Propuestas (cuando se prioricen)

- **Fase 30:** Funko Pop (HobbyQuantHQ).
- **Fase 31:** Amiibo (AmiiboAPI).

> Los números son una propuesta de ordenación; no están asignados en `README.md` ni en `requirements.md`.

## Nota Legal

AmiiboAPI es un proyecto comunitario sin ánimo de lucro que se mantiene con donaciones; sus imágenes viven en un repositorio de GitHub de un tercero. HobbyQuantHQ define sus condiciones y límites en su documentación.

---

|*Documentación informativa; no representa un compromiso de implementación.*|
