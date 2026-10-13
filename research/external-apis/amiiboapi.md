# AmiiboAPI — catálogo de amiibo

**Investigación para:** futura colección de amiibo (sin fase asignada)
**Estado:** ✅ Activa y gratuita — contrato verificado en vivo el 2026-10-13

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://amiiboapi.org/api` |
| **Auth** | Ninguna (sin API key) |
| **Rate limit** | No documentado ni expuesto en cabeceras |
| **Gratis** | Sí — la web declara que seguirá siendo pública (cuesta ~50 $/mes y vive de donaciones) |
| **Formato** | JSON |
| **CORS** | `Access-Control-Allow-Origin: *` |
| **Catálogo** | 950 amiibo · 32 series amiibo · 48 series de juego · 660 personajes |
| **Imágenes** | Alojadas en `raw.githubusercontent.com` (repo `8bitDream/AmiiboAPI`) |
| **Última actualización de datos** | 2026-09-19 (`GET /lastupdated/`) |
| **Documentación** | https://amiiboapi.org/ |

> ⚠️ La raíz `/api/` responde **404**. Hay que usar endpoints concretos (`/api/amiibo/`).

## Endpoints

| Uso | Endpoint | Notas |
|-----|----------|-------|
| Listado / búsqueda | `GET /amiibo/` | Filtros por query string (ver abajo) |
| Series de amiibo | `GET /amiiboseries/` | 32 pares `{key, name}`; admite `?name=` |
| Series de juego | `GET /gameseries/` | Pares `{key, name}` |
| Personajes | `GET /character/` | 660 pares `{key, name}` |
| Última actualización | `GET /lastupdated/` | `{"lastUpdated": "…"}` |
| Tipos | — | **No existe** `/amiibotype/` (404); el tipo es un campo del amiibo |

## Filtros de `/amiibo/` (verificados uno a uno)

| Filtro | ¿Funciona? | Ejemplo | Resultado |
|--------|:----------:|---------|-----------|
| `name` | ✅ coincidencia **parcial** | `?name=mario` | 32 |
| `name` (parcial) | ✅ | `?name=ario` | 41 |
| `type` | ✅ | `?type=Figure` | 258 |
| `amiiboSeries` | ✅ | `?amiiboSeries=Super Smash Bros.` | 96 |
| `character` | ✅ parcial | `?character=pikachu` | 2 |
| `gameseries` (**minúscula**) | ✅ | `?gameseries=Animal Crossing` | 527 |
| `gameSeries` (**camelCase**) | ❌ **se ignora** → devuelve todo | `?gameSeries=…` | 950 |
| `head` | ✅ | `?head=00000000` | 7 |
| `id` | ✅ devuelve **un objeto**, no una lista | `?id=0000000000000002` | objeto único |
| `showusage` | ✅ añade los juegos compatibles | `?name=mario&showusage` | +`games3DS`/`gamesWiiU`/`gamesSwitch`/`gamesSwitch2` |
| `sort` | ✅ | `?sort=name` | orden alfabético |

> ⚠️ **Los nombres de parámetro distinguen mayúsculas/minúsculas.** Es una trampa real: `amiiboSeries` funciona y `amiiboseries` no; al revés que `gameseries`, que solo funciona en minúscula. Además, **los parámetros desconocidos se ignoran en silencio**: un typo no da error, devuelve el catálogo completo.

## Forma de las Respuestas

```json
{ "amiibo": [ { "…": "amiibo" } ] }
```

⚠️ **`?id=` rompe el patrón:** devuelve el objeto directo, no una lista.

```json
{ "amiibo": { "…": "amiibo" } }
```

Sin resultados → `{"amiibo": []}` con **HTTP 200** (no 404).

## Campos de un Amiibo

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `amiiboSeries` | string | Serie del amiibo (p. ej. `Super Smash Bros.`) |
| `character` | string | Personaje |
| `gameSeries` | string | Serie de juego |
| `name` | string | Nombre del amiibo |
| `head` | hex (8) | Primera mitad del identificador |
| `tail` | hex (8) | Segunda mitad del identificador |
| `type` | string | `Card`, `Figure`, `Band`, `Yarn`, `Block` |
| `image` | string | PNG |
| `imgwebp` | string | WebP (mismo icono) |
| `release` | object | `{au, eu, jp, na}` con fechas ISO |

**`head` + `tail` forman el identificador único** del amiibo.

### Tipos en el Catálogo

| Tipo | Cantidad | Nota |
|------|---------:|------|
| `Card` | 674 | Cartas |
| `Figure` | 258 | **El tipo que interesa como figura** |
| `Band` | 9 | |
| `Yarn` | 5 | |
| `Block` | 4 | |

Con `showusage` se añaden `games3DS`, `gamesWiiU`, `gamesSwitch` y `gamesSwitch2`: listas de `{gameName, gameID[], amiiboUsage[]}`.

## Mapeo Propuesto → Modelo Interno (Amiibo)

| Campo AmiiboAPI | Campo interno | Notas |
|------------------|---------------|-------|
| `head` + `tail` | `externalId` (+ `source: amiiboapi`) | Identificador único |
| `name` | `name` | |
| `character` | `character` | |
| `amiiboSeries` | `amiiboSeries` | |
| `gameSeries` | `gameSeries` | |
| `type` | `type` | `Figure` es el tipo "coleccionable" |
| `release` | `releaseDates` | Fechas por región |
| `image` / `imgwebp` | `imageUrl` | Ver decisión de imágenes |

El resto de campos son los comunes a toda colección del proyecto (propietario, estado, notas, fecha de adquisición, imagen propia).

## Limitaciones

- ❌ **Sin búsqueda por código de barras.**
- ❌ `gameSeries` en camelCase no filtra (solo `gameseries` en minúscula).
- ❌ Los parámetros desconocidos se ignoran en silencio: un typo devuelve el catálogo entero.
- ❌ Sin cabeceras de rate limit: no se puede saber cuánta cuota queda.
- ⚠️ Imágenes alojadas en GitHub raw de un repo de terceros: riesgo de enlace muerto.
- ⚠️ No hay precios (lógico: es un catálogo, no un mercado).

## Configuración Propuesta

| Propiedad | Valor |
|-----------|-------|
| `amiiboapi.base-url` | `https://amiiboapi.org/api` |

Sin clave ni secreto: no requiere variable de entorno.

## Referencias

- [AmiiboAPI](https://amiiboapi.org/)
- [Repositorio de imágenes: 8bitDream/AmiiboAPI](https://github.com/8bitDream/AmiiboAPI)
