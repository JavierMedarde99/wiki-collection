---
type: external-api
title: "AmiiboAPI — catálogo de amiibo"
description: "Contrato de AmiiboAPI: catálogo, filtros y mapeo al modelo Amiibo"
tags: [api-externa, amiibo]
---

# AmiiboAPI — catálogo de amiibo

**Investigación para:** colección de amiibo (entidad `Amiibo`, endpoints `/api/v1/amiibos`)
**Estado:** ✅ Activa, gratuita y **ya integrada en el backend**
**Verificado en vivo:** 2026-10-15 (contrato reverificado con `curl --max-time 20`, sin clave)

> La ficha se escribió como investigación previa, pero el código de `main` ya la consume: PR #467 «feat(amiibo): colección de Amiibo (cliente, persistencia, servicio y API)», fusionado el 2026-10-13. `research/future/collectibles-funko-amiibo.md` todavía la clasifica como investigación futura; el código manda.

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://amiiboapi.org/api` |
| **Auth** | Ninguna (sin API key ni secreto) |
| **Métodos** | Solo lectura. `GET` y `HEAD` responden 200; `OPTIONS` responde 200 (preflight CORS); `POST` responde **405** |
| **Rate limit** | No documentado ni expuesto en cabeceras. La documentación oficial **recomienda cachear** en el cliente |
| **Gratis** | Sí — la web declara que seguirá siendo pública; cuesta ~50 $/mes y vive de donaciones |
| **Formato** | JSON (`Content-Type: application/json`); soporta `gzip` |
| **CORS** | `Access-Control-Allow-Origin: *` en respuestas y en el preflight `OPTIONS` |
| **Catálogo** | 950 amiibo · 32 series de amiibo · 119 entradas de serie de juego (48 nombres distintos) · 685 entradas de personaje (660 nombres distintos) · 5 tipos |
| **Imágenes** | `raw.githubusercontent.com/8bitDream/AmiiboAPI` (rama `master`; rutas `images/` para PNG y `webp/` para WebP) |
| **Última actualización de datos** | `2026-09-19T10:32:16` (`GET /lastupdated/`) |
| **Documentación** | https://amiiboapi.org/docs/ — FAQ en https://amiiboapi.org/faq/ |

Notas de comportamiento:

- La raíz `/api/` responde **404**: hay que usar siempre un endpoint concreto (`/api/amiibo/`).
- Las rutas sin barra final responden **308** y redirigen a la versión con barra (`/api/amiibo` → `/api/amiibo/`).
- No hay paginación en servidor: `/amiibo/` devuelve el catálogo completo (~427 KB sin `showusage`, ~145 KB para 32 resultados con `showusage`).

## Endpoints

| Uso | Endpoint | Notas |
|-----|----------|-------|
| Listado / búsqueda | `GET /amiibo/` | Filtros por query string (ver abajo); es el **único** endpoint que consume el backend |
| Tipos de amiibo | `GET /type/` | 5 pares `{key, name}`. ⚠️ **No existe `/amiibotype/`** (404); la ruta real es `/type/` |
| Series de amiibo | `GET /amiiboseries/` | 32 pares `{key, name}` |
| Series de juego | `GET /gameseries/` | 119 pares `{key, name}` (48 nombres distintos: varias claves comparten nombre) |
| Personajes | `GET /character/` | 685 pares `{key, name}` (660 nombres distintos) |
| Última actualización | `GET /lastupdated/` | `{"lastUpdated": "2026-09-19T10:32:16.038284"}` |

Los cuatro endpoints de catálogo (`/type/`, `/amiiboseries/`, `/gameseries/`, `/character/`) siguen el mismo patrón y admiten dos filtros:

| Filtro | Resultado | Ejemplo |
|--------|-----------|---------|
| sin filtro | **lista** completa de `{key, name}` | `GET /type/` → 5 entradas |
| `?name=` | coincidencia **parcial y sin distinguir mayúsculas**; devuelve **lista** | `GET /amiiboseries/?name=Super Mario` → `[{key: "0x01", …}]` |
| `?key=` | coincidencia exacta; devuelve **un objeto**, no una lista | `GET /character/?key=0x0000` → `{key: "0x0000", name: "Mario"}` |

## Filtros de `/amiibo/` (verificados uno a uno el 2026-10-15)

| Filtro | ¿Funciona? | Ejemplo | Resultado |
|--------|:----------:|---------|-----------|
| `name` | ✅ coincidencia parcial, **sin distinguir mayúsculas** | `?name=mario` / `?name=Mario` | 32 |
| `name` (subcadena) | ✅ | `?name=ario` | 41 |
| `name` (vacío) | ⚠️ se ignora | `?name=` | 950 |
| `type` | ✅ acepta **nombre o clave**, sin distinguir mayúsculas | `?type=Figure` / `?type=figure` / `?type=0x02` | 258 / 258 / 5 (`Yarn`) |
| `amiiboSeries` (**camelCase**) | ✅ acepta nombre o clave | `?amiiboSeries=Super Smash Bros.` / `?amiiboSeries=0x10` | 96 / 1 |
| `amiiboseries` (minúscula) | ❌ **se ignora** → devuelve todo | `?amiiboseries=Super Smash Bros.` | 950 |
| `gameseries` (**minúscula**) | ✅ acepta nombre o clave | `?gameseries=Animal Crossing` / `?gameseries=0x22c` | 527 / 1 |
| `gameSeries` (**camelCase**) | ❌ **se ignora** → devuelve todo | `?gameSeries=Animal Crossing` | 950 |
| `character` | ✅ parcial, sin distinguir mayúsculas | `?character=pikachu` / `?character=Mario` | 2 / 30 |
| `head` | ✅ | `?head=00000000` | 7 |
| `tail` | ✅ | `?tail=00000002` | 1 |
| `id` | ✅ `head`+`tail`; devuelve **un objeto**, no una lista | `?id=0000000000000002` | objeto único |
| `showusage` | ✅ añade los juegos compatibles (ver abajo) | `?name=mario&showusage` | +`games3DS`/`gamesWiiU`/`gamesSwitch`/`gamesSwitch2` |
| `sort=name` | ✅ orden alfabético ascendente | `?sort=name` | 950 ordenados |
| `sort=type` | ✅ agrupa por tipo en el orden de `/type/` | `?sort=type` | 258 `Figure` + 674 `Card` + 5 `Yarn` + 9 `Band` + 4 `Block` |
| `sort` con valor desconocido | ❌ se ignora (orden por defecto) | `?sort=nameZ` | 950 sin ordenar |
| combinación | ✅ los filtros se acumulan | `?name=mario&type=Figure` | 11 |

> ⚠️ **Los nombres de parámetro distinguen mayúsculas/minúsculas.** Es una trampa real: `amiiboSeries` funciona y `amiiboseries` no; al revés que `gameseries`, que solo funciona en minúscula. Además, **los parámetros desconocidos se ignoran en silencio**: `?nam=mario` o `?foo=bar` no dan error, devuelven el catálogo completo (950).

**`showusage`:** basta con que el parámetro esté presente; el valor da igual (`?showusage`, `?showusage=true` y `?showusage=false` producen el mismo resultado). Añade cuatro listas por plataforma: `games3DS` (778 amiibo con datos), `gamesWiiU` (597), `gamesSwitch` (887) y `gamesSwitch2` (213). Cada entrada tiene la forma `{gameName, gameID[], amiiboUsage[]}`, donde `amiiboUsage` es `[{Usage, write}]`.

> ⚠️ `showusage` **no funciona en `?id=`**: `?id=0000000000000002&showusage` devuelve el mismo objeto sin las listas de juegos.

## Forma de las Respuestas

Listados (`/amiibo/` con o sin filtros):

```json
{ "amiibo": [ { "…": "amiibo" } ] }
```

`?id=` y las consultas `?key=` de los catálogos rompen el patrón: devuelven el objeto directo, no una lista.

```json
{ "amiibo": { "…": "amiibo" } }
```

Comportamiento cuando no hay coincidencias (**no es uniforme**):

| Consulta | HTTP | Cuerpo |
|----------|:----:|--------|
| `/amiibo/?name=zzzzzzzzzz` (sin resultados) | **200** | `{"amiibo":[]}` |
| `/amiibo/?id=FFFFFFFFFFFFFFFF` (`id` hexadecimal que no existe) | **200** | `{"amiibo":null}` |
| `/amiibo/?id=zz` (`id` no hexadecimal) | **400** | `{"code":400,"error":"…"}` |
| `/amiibo/?id=` (`id` vacío) | **400** | `{"code":400,"error":"…"}` |
| `/amiiboseries/?name=zzz` (sin resultados) | **404** | `{"code":404,"error":"…"}` |

Es decir: un `id` con forma hexadecimal y **longitud libre** (probado con 1, 3, 6, 15, 16, 18 caracteres) que no está en el catálogo devuelve **200 + `null`**, mientras que un `id` vacío o con caracteres no hexadecimales devuelve **400**. Los catálogos secundarios sí devuelven **404** cuando el filtro no encuentra nada.

## Campos de un Amiibo

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `amiiboSeries` | string | Serie del amiibo (p. ej. `Super Smash Bros.`) |
| `character` | string | Personaje |
| `gameSeries` | string | Serie de juego |
| `name` | string | Nombre del amiibo |
| `head` | hex (8) | Primera mitad del identificador |
| `tail` | hex (8) | Segunda mitad del identificador |
| `type` | string | `Figure`, `Card`, `Yarn`, `Band`, `Block` |
| `image` | string | PNG |
| `imgwebp` | string | WebP del mismo icono (**no lo consume el backend**) |
| `release` | object | `{au, eu, jp, na}` con fechas ISO `yyyy-mm-dd`; una región sin fecha viene `null` |
| `games3DS`, `gamesWiiU`, `gamesSwitch`, `gamesSwitch2` | array | Solo con `showusage`; `{gameName, gameID[], amiiboUsage[]}` |
| `variant` | string | Documentado en `/docs/` como campo opcional — **ausente en las 950 fichas actuales**, no se consume |

Datos verificados sobre el catálogo completo (950 fichas):

- **`head` + `tail` forman el identificador único**: 8 + 8 hex, 950 combinaciones distintas, sin duplicados.
- `name`, `character`, `amiiboSeries` y `gameSeries` están presentes en las 950.
- `release` está presente en las 950, pero con regiones nulas: `au` 49, `eu` 140, `na` 34, `jp` 18.
- `image` e `imgwebp` están presentes en las 950 y siempre apuntan a `raw.githubusercontent.com/8bitDream/AmiiboAPI/master/`.

### Tipos en el Catálogo

| Tipo | Clave `/type/` | Cantidad | Nota |
|------|:--------------:|---------:|------|
| `Card` | `0x01` | 674 | Cartas |
| `Figure` | `0x00` | 258 | **El tipo que interesa como figura** |
| `Band` | `0x03` | 9 | |
| `Yarn` | `0x02` | 5 | |
| `Block` | `0x04` | 4 | |

`GET /type/` devuelve los 5 en el orden `Figure`, `Card`, `Yarn`, `Band`, `Block`; `?sort=type` sobre `/amiibo/` usa ese mismo orden.

## Uso en el Backend y Mapeo al Modelo Interno

Cliente real: `src/main/java/com/wikicollection/infrastructure/adapter/out/amiibo/AmiiboApiClient.java` (implementa el puerto `ExternalAmiiboCatalogClient`). **Solo usa `GET /amiibo/`**; de la API no toca `/type/`, `/amiiboseries/`, `/gameseries/`, `/character/` ni `/lastupdated/`.

Dos llamadas, ambas construidas con los parámetros exactos (el cliente documenta en su javadoc la trampa de mayúsculas):

| Operación | Petición | Detalles |
|-----------|----------|----------|
| `searchByName(query)` | `GET /amiibo/?name=<query>&showusage=true` | Resultado cacheado en Caffeine `amiiboSearch` con clave = query (`app.cache.ttl.amiibos=3600s`) |
| `findById(externalId)` | `GET /amiibo/?id=<externalId>` | Sin `showusage` y sin caché en el cliente |

Campos que **sí** consume (record `AmiiboApiItem`): `head`, `tail`, `name`, `character`, `amiiboSeries`, `gameSeries`, `type`, `image`, `release.{au,eu,jp,na}` y `games3DS`/`gamesWiiU`/`gamesSwitch`/`gamesSwitch2` (de cada juego solo `gameName`, record `AmiiboApiGame`). **No** consume `imgwebp` ni `variant`, y descarta `gameID` y `amiiboUsage`. Las respuestas se deserializan con `@JsonIgnoreProperties(ignoreUnknown = true)`, así que el resto del payload se ignora en silencio.

| Campo AmiiboAPI | Campo en `AmiiboSearchResult` / `Amiibo` | Transformación |
|------------------|------------------------------------------|----------------|
| `head` + `tail` | `externalId` | Concatenación directa (16 hex); `externalSource` = `amiiboapi` |
| `name` | `name` | Directo |
| `character` | `character` | Directo |
| `amiiboSeries` | `amiiboSeries` | Directo |
| `gameSeries` | `gameSeries` | Directo |
| `type` | `amiiboType` (`AmiiboType`) | `CARD`/`FIGURE`/`BAND`/`YARN`/`BLOCK`; valor desconocido o vacío → `null` (tolerante, no error) |
| `release` | `releaseDates` (`AmiiboReleaseDates`) | `au`/`eu`/`jp`/`na` como `LocalDate`; ausente o no parseable → `null` por región |
| `image` | `imageUrl` | Directo. Estrategia de imágenes: **decisión pendiente** |
| `games3DS` + `gamesWiiU` + `gamesSwitch` + `gamesSwitch2` | `compatibleGames` (`List<String>`) | Aplanado en **una sola lista** con solo `gameName`; se descartan nulos/vacíos y se aplica `distinct()` (un juego repetido en varias plataformas aparece una vez). Orden: 3DS, Wii U, Switch, Switch 2, manteniendo el orden interno de la API |
| `imgwebp` | — | **No se consume** |
| `variant` | — | **No se consume** (además, ausente en el catálogo actual) |

La entidad `Amiibo` añade los campos comunes de colección (propietario, estado, notas, fecha de alta y fecha de adquisición). AmiiboAPI **no expone ningún campo de precio**: los precios quedan fuera del alcance de esta integración.

Gestión de errores y robustez del cliente:

- **Reintentos:** 4 intentos con 1 s de espera (`TOTAL_ATTEMPTS = 4`, `retryIntervalMillis = 1000`). Solo se reintentan errores 5xx y fallos de conexión/timeout; un 4xx se relanza de inmediato.
- **Timeouts HTTP** (compartidos con el resto de clientes, `HttpClientProperties`): conexión 5 s, lectura 10 s.
- `searchByName`: un 200 con `{"amiibo": []}` o `{"amiibo": null}` se traduce a lista vacía (vacío ≠ error).
- `findById`: `{"amiibo": null}` y cualquier 4xx se traducen a `Optional.empty()`.
- El javadoc del cliente afirma que AmiiboAPI responde 400 «cuando el id no existe»; en vivo eso solo ocurre con un `id` **no hexadecimal o vacío**. Con un `id` hexadecimal inexistente la API responde 200 + `null` (el test del cliente usa `zzzzzzzzzzzzzzzz` para simular el 400). El cliente cubre ambos casos, pero la premisa del comentario no es exacta.

Endpoint expuesto hacia el frontend: `GET /api/v1/amiibos/search?name=<texto>&page=&size=` (`name` obligatorio). La API externa no pagina, así que el backend corta la página en memoria.

## Limitaciones

- ❌ **Sin búsqueda por código de barras.**
- ❌ **Sin precios**: AmiiboAPI no expone mercado ni valoración.
- ❌ `gameSeries` en camelCase no filtra (solo `gameseries` en minúscula) y `amiiboseries` en minúscula tampoco (solo `amiiboSeries`).
- ❌ Los parámetros desconocidos se ignoran en silencio: un typo devuelve el catálogo entero (950) sin avisar.
- ❌ **Respuestas incoherentes ante «sin resultados»**: 200 + `[]` en `/amiibo/`, 200 + `null` con `?id=` hexadecimal inexistente, 400 con `?id=` malformado y 404 en los catálogos secundarios.
- ❌ Sin cabeceras de rate limit: no se puede saber cuánta cuota queda. La documentación solo pide «cachear».
- ❌ Sin paginación en servidor: filtrar sin acotar descarga el catálogo completo.
- ⚠️ `gamesSwitch2` existe en las respuestas reales pero **no aparece en `/docs/`**: hay que tratarlo como campo no documentado.
- ⚠️ `variant` está documentado en `/docs/` pero no aparece en ninguna de las 950 fichas actuales.
- ⚠️ Imágenes alojadas en GitHub raw de un repo de terceros: riesgo de enlace muerto.
- ⚠️ **Riesgo de continuidad:** la FAQ explica que el servicio original anunció su cierre para el 31 de diciembre de 2025 y que lo continúa un colaborador desde un fork. Es un proyecto sin SLA, «principalmente para fines educativos», sin relación con Nintendo y con términos que pueden cambiar sin aviso.

## Configuración

| Propiedad | Valor | Notas |
|-----------|-------|-------|
| `amiiboapi.base-url` | `https://amiiboapi.org/api` | Sobrescribible por entorno; también es el valor por defecto en `RestClientConfig` (`amiiboRestClient`) |
| `app.cache.ttl.amiibos` | `3600s` | TTL de la caché Caffeine `amiiboSearch` (clave = texto buscado) |

Sin clave ni secreto: no requiere variable de entorno. Timeouts y reintentos son los descritos arriba (`http.client.*` y las constantes del cliente).

## Referencias

- [AmiiboAPI](https://amiiboapi.org/)
- [Documentación](https://amiiboapi.org/docs/)
- [FAQ](https://amiiboapi.org/faq/)
- [Repositorio (código e imágenes): 8bitDream/AmiiboAPI](https://github.com/8bitDream/AmiiboAPI)
