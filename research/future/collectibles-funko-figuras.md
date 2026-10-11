# Funko Pop y Figuras — nuevas colecciones

**Estado:** 📋 Investigación — sin fase asignada
**Fecha:** 2026-10-11
**Decisión asociada (2026-10-11):** se tratarán como **nuevas colecciones del producto**; **sin precios**; estrategia de imágenes pendiente.

## Descripción

Dos colecciones nuevas, distintas de las actuales (libros, juegos, juegos de mesa, Magic, mazos, películas/series):

- **Funko Pop** — catálogo de vinilos Pop!, con serie, número, exclusividad, Vaulted e índice de rareza.
- **Figuras** — anime/merch: Nendoroid, Figma, escala, PVC, etc.

## Resumen de Viabilidad

| Colección | Fuente | Viable | Motivo |
|-----------|--------|:------:|--------|
| **Funko** | [HobbyQuantHQ](../external-apis/hobbyquanthq.md) (REST) | ✅ **Sí** | API REST normal con API key: mismo patrón que RAWG / Google Books |
| **Figuras** | [myfigurecollection-api](../external-apis/myfigurecollection.md) (Python) | ⚠️ **No como integración de backend** | No hay API oficial; es scraping local y Cloudflare bloquea servidores |

**Conclusión:** Funko se puede construir con la arquitectura actual. Figuras queda **bloqueada técnicamente** hasta que cambie la situación de acceso a MFC.

## Capacidades Afectadas

- **Entidades nuevas:** `Funko` y (bloqueada) `Figure`.
- **Controllers:** `/api/v1/funkos`, eventualmente `/api/v1/figures`.
- **Visibilidad:** filtro `owner` (`mine`/`other`/`all`), colecciones activas y estadísticas — igual que el resto.
- **Caché:** nuevas cachés `funkoSearch`, `funkoList`, `funkoDetail`…
- **Frontend:** páginas de listado/detalle, búsqueda, formulario, y escáner de barcode (solo figuras, vía `search_by_barcode`).
- **Docs a tocar si se prioriza:** `data-model/schema.md`, `architecture/backend.md`, `architecture/frontend-*.md`, `README.md`, `requirements.md`.

## APIs Externas Necesarias

| Colección | Ficha | Acceso |
|-----------|-------|--------|
| Funko | [hobbyquanthq.md](../external-apis/hobbyquanthq.md) | REST + header `x-api-key` |
| Figuras | [myfigurecollection.md](../external-apis/myfigurecollection.md) | Librería Python local (scraping) |

## Modelo de Datos Propuesto (normalizado)

Todo dato externo se normaliza a un modelo propio, para no depender del formato de cada fuente.

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

**Figura:**

```json
{
  "id": "internal-123",
  "type": "figure",
  "source": "mfc",
  "externalId": 287,
  "name": "Aria - Alice Carroll - Maa - 1/6",
  "brand": "Toy's Works",
  "manufacturer": "Toy's Works",
  "character": "Alice Carroll",
  "scale": "1/6",
  "height": null,
  "barcode": "4543341130624",
  "releaseDate": null,
  "imageUrl": null
}
```

Ambos suman los campos comunes de colección (propietario, estado, notas, fecha de adquisición, imagen propia…).

## Consideraciones de Diseño

1. **Funko — patrón idéntico a RAWG/Google Books:** cliente HTTP con API key, caché Caffeine (`funkoSearch`), reintentos, mapeo a dominio y `owner` filter. Riesgo bajo.
2. **Figuras — no seguir el patrón actual:** no hay API a la que llamar. Opciones descritas en la ficha (uso local / sidecar Python / reimplementar en Java / esperar API oficial); ninguna es limpia hoy.
3. **Imágenes (decisión pendiente):** las fuentes entregan URLs externas (`images.hobbydb.com`, MFC). Falta decidir si se descargan y suben a Catbox (consistente con el proyecto) o se hace hotlink. Riesgos: enlaces muertos y dependencia del CDN ajeno.
4. **Precios: fuera de alcance** (decisión del 2026-10-11). HobbyQuantHQ los tiene "en evolución" y MFC solo da listings con `maybe_available`.
5. **Barcode:** en figuras vía `search_by_barcode` (lookup, no búsqueda). **Funko no tiene endpoint de barcode** — el escáner solo aplica a figuras.
6. **Zona de despliegue:** una posible integración de figuras **no puede vivir en un hosting/VPS** (Cloudflare bloquea la IP). Si algún día se hace, tiene que correr en el equipo del usuario.

## Prioridad Estimada

| Colección | Prioridad | Comentario |
|-----------|-----------|------------|
| Funko | **Media** | Integración de bajo riesgo, mismo patrón que las existentes |
| Figuras | **Baja / bloqueada** | Sin vía de acceso viable desde el backend |

## Fases Propuestas (cuando se prioricen)

- **Fase 30:** Funko Pop (HobbyQuantHQ).
- **Fase 31:** Figuras (MyFigureCollection) — bloqueada hasta que exista una vía de acceso.

> Los números son una propuesta de ordenación; no están asignados en `README.md` ni en `requirements.md`.

## Nota Legal

`myfigurecollection-api` es MIT, pero **los datos siguen siendo de MyFigureCollection y sus colaboradores**; el propio proyecto lo advierte. HobbyQuantHQ define sus condiciones y límites en su documentación.

---

|*Documentación informativa; no representa un compromiso de implementación.*|
