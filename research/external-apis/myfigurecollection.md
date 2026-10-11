# MyFigureCollection — acceso programático a figuras

**Investigación para:** futura colección de figuras (sin fase asignada)
**Estado:** ⚠️ **Sin API oficial.** Única vía: librería Python local (verificado el 2026-10-11)

## Conclusión de Viabilidad (lo importante)

- ❌ **MFC no tiene API pública oficial.** Su propio club de API ([#349](https://myfigurecollection.net/club/349)) lleva desde 2018 prometiendo una "API v4" que nunca llega.
- ❌ **No hay REST de terceros viable.** [`Tenji-hin/tenji-api`](https://github.com/Tenji-hin/tenji-api) (MIT) existe, pero su último push es del **2023-09-12**: está abandonado. La propia librería documenta que *"tenji murió"*.
- ✅ **Única vía que funciona:** [`myfigurecollection-api`](https://pypi.org/project/myfigurecollection-api/) — librería Python (MIT) que lee el HTML público y lo convierte en modelos tipados.
- ⚠️ **Es una herramienta local por diseño.** Cloudflare bloquea por fingerprint TLS y por reputación de IP: un servicio alojado en un servidor sería bloqueado en días. Debe ejecutarse en el equipo del usuario.

## Por Qué un Cliente HTTP Normal Falla

`requests`, `httpx` y `aiohttp` reciben **403 "Just a moment…"** de Cloudflare, manden las cabeceras que manden: el filtro es el **fingerprint TLS**, no el user-agent.

La librería usa [`curl_cffi`](https://github.com/lexiforest/curl_cffi) con `impersonate="chrome"`, que reproduce el handshake real de Chrome y obtiene 200. **Ese detalle es crítico**: cambiarlo rompe todo.

## Instalación

Requiere **Python ≥ 3.10**:

```bash
pip install myfigurecollection-api
mfc-api item 287   # comprobación
```

## API de la Librería (métodos)

| Método | Devuelve |
|--------|----------|
| `get_item(item_id)` | Ficha completa: categoría, imagen, personajes, compañías, artistas, lanzamientos (fecha/edición/barcode), escala, altura, valoración, nº de propietarios |
| `search_items(query, page, category_id)` | Búsqueda, **50 por página**, con paginación |
| `search_by_barcode(barcode)` | Ficha a partir de JAN/UPC |
| `get_profile(username)` | Fecha de alta, visitas, rango, campos "about", conteos por categoría |
| `get_collection(username, status, page)` | **50 por página**; `status` ∈ `owned`, `ordered`, `wished`, `favorites` |
| `get_user_lists(username, page)` | Listas publicadas por un usuario |
| `get_list(list_id, page)` | Lista: propietario, descripción, etiquetas y artículos |
| `get_club(club_id)` | Descripción, hilos, comentarios recientes, staff y miembros |
| `get_club_members(club_id, page)` | Plantilla completa de miembros |
| `get_entry_items(entry_id)` | Todo lo de un origen/personaje (p. ej. Chiikawa: 7.247 artículos), 50 por página |
| `iter_collection(username, max_pages)` | Itera la colección página a página respetando el rate limit |
| `transport.get_bytes(url)` | Descarga una imagen pasando por el rate limiter |

`Item.picture` es la vista de 256 px; `Item.picture_large` es el original.

## Uso desde el Shell (CLI)

```bash
mfc-api item 287
mfc-api barcode 4543341130624
mfc-api collection USUARIO --status owned --page 2
```

Todo imprime JSON por stdout; los logs van a stderr.

## Servidor MCP

`mfc-api` sin argumentos arranca un servidor **MCP por stdio** con 12 herramientas (las mismas que la librería). Pensado para asistentes de IA, **no** para ser consumido por HTTP desde un backend.

## Rate Limit y Caché

| Variable de entorno | Por defecto | Significado |
|---------------------|-------------|-------------|
| `MFC_API_RATE_LIMIT` | `1.0` | Segundos entre peticiones |
| `MFC_API_CACHE_TTL` | `3600` | Vida de la caché en segundos (`0` la desactiva) |
| `MFC_API_CACHE_DIR` | `~/.cache/mfc-api` | Directorio de caché |
| `MFC_API_LOG_LEVEL` | `WARNING` | Nivel de log (a stderr) |

- Un **429** lanza `MFCRateLimitedError` con `retry_after`; **no** reintenta solo.
- **No subir el rate limit**: Cloudflare bloquea peticiones excesivas y quema la IP.

## Códigos de Barras

`search_by_barcode("4543341130624")` devuelve `{matched: bool, item}`.

- Es una **consulta por clave, no un buscador**.
- `matched: false` **no** significa que el código no exista: significa que **MFC no lo registró**, algo habitual en lanzamientos occidentales.

> **Decisión del 2026-10-11:** la librería también expone tiendas y listings con precios, pero **no se documentan** en la wiki (fuera de alcance por ahora).

## Integración con el Backend Java — Opciones

| Opción | Viable | Notas |
|--------|:------:|-------|
| **A. Solo uso local (CLI / librería)** | ✅ | Sin integración con el backend. Es el escenario para el que se diseñó |
| **B. Sidecar Python junto al backend** | ⚠️ | Funciona si el backend corre en un equipo doméstico; en un VPS/hosting Cloudflare lo bloqueará |
| **C. Reimplementar en Java** | ❌ | Exigiría impersonar el handshake TLS de Chrome (curl_cffi) desde Java; extremadamente frágil |
| **D. Esperar API oficial** | ❌ | Anunciada desde 2018, sin fecha |

**Recomendación:** no integrar figuras en el backend mientras la única vía sea scraping local.

## Limitaciones

- ❌ No es una API oficial; depende de la estructura HTML de MFC.
- ❌ Puede romperse si MFC cambia su web.
- ❌ Cloudflare puede bloquear peticiones excesivas.
- ❌ Pensada para ejecución local, no como servicio alojado.
- ❌ Rate limit por defecto de 1 petición/segundo.
- ❌ No todos los productos tienen barcode registrado.

## Referencias

- [GitHub: ssskay/myfigurecollection-api](https://github.com/ssskay/myfigurecollection-api)
- [PyPI: myfigurecollection-api](https://pypi.org/project/myfigurecollection-api/)
- [MyFigureCollection](https://myfigurecollection.net/)
- [Tenji-hin/tenji-api (abandonado, 2023)](https://github.com/Tenji-hin/tenji-api)
