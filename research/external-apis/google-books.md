# Google Books API

**Investigación para:** Fase 1 — Colección de Libros (búsqueda de catálogo externo y autocompletado de géneros)

**Verificado en vivo:** 2026-10-15

- `GET https://www.googleapis.com/books/v1/volumes?q=harry+potter` (sin `key`) → **HTTP 429** `RESOURCE_EXHAUSTED` (`quota_limit=defaultPerDayPerProject`, `quota_limit_value=0`). Repetido con `q=intitle:harry+potter`, `q=test` y `country=US`: mismo 429. Sin clave la búsqueda no responde hoy.
- `GET https://www.googleapis.com/books/v1/volumes/LCPqzgEACAAJ` (sin `key`) → **HTTP 429**, mismo error.
- `GET https://books.google.com/books?jscmd=viewapi&bibkeys=ISBN:9788466356718` (Dynamic Links, otro host, sin clave) → **HTTP 200** con `var _GBSBookInfo = {...}` correcto. Verificado también con dos ISBN separados por coma.
- No se ha podido verificar en vivo una respuesta **200** del endpoint `googleapis.com/books` porque la cuota anónima está a 0. No hay `GOOGLE_BOOKS_API_KEY` en el entorno.

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://www.googleapis.com/books` (cliente) + `/v1/volumes`; la Dynamic Links API vive en `https://books.google.com/books` |
| **Auth** | Clave de API en query param `key` (o token OAuth 2.0). La guía oficial indica que las peticiones de datos públicos «must be accompanied by an identifier», es decir, clave u OAuth. En la práctica sin clave hoy la cuota anónima es 0 y devuelve 429 |
| **Rate limit** | Esquema de cuotas de Google Cloud. La guía oficial de Books **no publica** una cifra fija de peticiones/segundo; el error devuelve `quota_limit=defaultPerDayPerProject`. No se pudo verificar el límite real con clave |
| **Gratis** | Sí, con clave de API gratuita de Google Cloud Console |
| **Formato** | JSON (los endpoints `googleapis.com`). La Dynamic Links API devuelve JavaScript/JSONP: `var _GBSBookInfo = {...}` |
| **Documentación oficial** | https://developers.google.com/books/docs/v1/using |
| **Estado** | Activa. `Last updated 2025-08-28` en la guía oficial |

## Endpoints

Endpoints realmente usados por `GoogleBooksClient`:

| Uso | Endpoint | Parámetros | Notas |
|-----|----------|------------|-------|
| Búsqueda por título | `GET /v1/volumes` | `q=intitle:{query}`, `langRestrict=es`, `startIndex`, `maxResults`, `key` (si hay) | El cliente antepone `intitle:` y fija `langRestrict=es`. Pide en trozos de hasta 40 (`GOOGLE_MAX_RESULTS`) hasta completar el `pageSize`; usa `totalItems` como total de la página |
| Búsqueda por ISBN | `GET /v1/volumes` | `q=isbn:{isbn}`, `key` (si hay) | **Sin** `langRestrict` (Google descarta en silencio ediciones sin etiquetar de idioma) y sin paginación |
| Volumen por id | `GET /v1/volumes/{volumeId}` | `key` (si hay) | Se usa en dos sitios: `getCategories(volumeId)` y el fallback de ISBN |
| Resolver ISBN → volumeId | `GET https://books.google.com/books` | `jscmd=viewapi`, `bibkeys=ISBN:{isbn}` | Dynamic Links API de Google, host distinto y sin clave. El cliente extrae el id con regex `[?&]id=([A-Za-z0-9_-]+)` del `info_url` |

Flujo de `searchByIsbn`: primero `q=isbn:`; si no hay resultados, resuelve el ISBN con la Dynamic Links API y consulta cada volumen por id.

## Mapeo: API → modelo interno

`GoogleBooksClient` no devuelve la entidad `Book` directamente: devuelve `BookSearchResult`. Al guardar, el `Book` se rellena por separado.

| Campo Google Books | `BookSearchResult` | `Book` | Notas |
|--------------------|---------------------|--------|-------|
| `id` | `id` | `externalId` | Id del volumen |
| `volumeInfo.title` | `title` | `title` | — |
| `volumeInfo.authors` | `authors` (`List<String>`) | `author` | `BookSearchResult` conserva la lista; la entidad `Book` solo tiene un `author` de tipo `String` |
| `volumeInfo.industryIdentifiers` | `isbn` | `isbn` | Se prefiere `ISBN_13`; si no hay, `ISBN_10` |
| `volumeInfo.imageLinks.thumbnail` | `coverImage` | `frontpage` | — |
| `volumeInfo.description` | `description` | `descripcion` | — |
| `volumeInfo.pageCount` | `pageCount` | `pages` | — |
| `volumeInfo.publisher` | `publisher` | `publisher` | Sí se persiste |
| `volumeInfo.publishedDate` | `publishedDate` | — | Texto crudo; precisión variable (`2011`, `2011-05`, `2011-05-03`) |
| `volumeInfo.publishedDate` (4 primeros dígitos) | `publicationYear` | `publicationYear` | Se normaliza a año; `null` si no es un año plausible |
| `volumeInfo.language` | `language` | — | No se persiste como campo propio |
| `volumeInfo.categories` | `categories` | `genres` | Relleno aparte: `getCategories(volumeId)` consulta el volumen y `BookService.fillEmptyGenres` asigna las categorías a `genres` solo si están vacíos |

## Configuración en el proyecto

Propiedad real en `src/main/resources/application.properties`:

```properties
# Google Books API
google.books.api-key=${GOOGLE_BOOKS_API_KEY:}
```

- Si la propiedad viene vacía, el cliente **omite** el query param `key`.
- El `RestClient` se construye en `RestClientConfig` con base `https://www.googleapis.com/books` y la política HTTP compartida: `http.client.connect-timeout=5s`, `http.client.read-timeout=10s`.
- La Dynamic Links API se llama con URL absoluta (`URI.create`), por lo que sobreescribe la base del `RestClient`.
- Método consumido desde `BookSearchService`; expuesto en `GET /api/v1/books/search` con los query params `name` (título) e `isbn` (tiene prioridad sobre `name`).

## Retry y caché

**Retry** (`withRetry`, solo en búsquedas y en las llamadas del fallback de ISBN):

- `TOTAL_ATTEMPTS = 4` → intento inicial + 3 reintentos.
- Espera **fija** de `1000 ms` entre intentos (`DEFAULT_RETRY_INTERVAL_MILLIS`), sin backoff exponencial.
- Reintenta ante `RestClientResponseException` (errores HTTP) y `ResourceAccessException` (timeouts/red). Tras agotar intentos, relanza la última excepción HTTP o, si no hubo, la de conexión.
- **`getCategories` no reintenta**: hace un único intento y devuelve lista vacía ante cualquier error HTTP o de conexión.

**Caché** (Caffeine, `CacheConfig`):

- `search`, `searchByIsbn` y `getCategories` comparten el nombre de caché `bookSearch` (`CacheConfig.BOOK_SEARCH`).
- Claves: `search` → `"{query}:{offset}:{pageSize}"`; `searchByIsbn` → `"isbn:{isbn}"`; `getCategories` → `"volume:{volumeId}"`.
- TTL: `app.cache.ttl.books=3600s` (1 hora). Sin Redis: la caché es en memoria.

## Limitaciones

- **Sin clave, hoy no funciona:** la cuota anónima de `googleapis.com/books` está a 0 y toda petición devuelve 429. La cifra de rate limit con clave no está publicada en la guía ni se pudo verificar aquí.
- El binario `application.properties` trae `google.books.api-key` vacío por defecto; si no se define `GOOGLE_BOOKS_API_KEY`, las búsquedas dependerán de la cuota anónima.
- `search` fija `langRestrict=es`, así que puede descartar ediciones sin etiqueta de idioma; el cliente ya lo evita en `searchByIsbn`.
- `publishedDate` tiene precisión variable; solo se deriva `publicationYear` y solo si los 4 primeros caracteres son un año plausible.
- `volumeInfo.authors` llega como lista; la entidad `Book` solo guarda un `author`.
- `q=isbn:` no siempre indexa ediciones sin vista previa; de ahí el fallback a la Dynamic Links API, que se documenta aparte de la Books API v1.
- La Dynamic Links API devuelve JavaScript, no JSON estricto; el cliente lo parsea con una expresión regular sobre el `info_url`. Su documentación la describe como interfaz client-side para JavaScript y señala que **no está pensada para consultas server-side u offline**, aunque el backend la invoca desde el servidor. No requiere clave de API.
- La FAQ de Dynamic Links avisa de que un volumen anómalo de peticiones puede activar protecciones (captcha) en Google Books.
- `imageLinks.thumbnail` es una URL de `books.google.com`; no todos los volúmenes tienen portada.
- Open Library no es un fallback de catálogo, sino un **enriquecedor** (`ExternalBookEnricher`): rellena portada/descripción cuando Google no las trae, y si falla se conserva el resultado de Google.
- No se pudo verificar en vivo ningún código 200 del endpoint de búsqueda; las conclusiones de mapeo provienen del código real y de los tests `GoogleBooksClientTest`, no de una respuesta real de hoy.

## Referencias

- [Using the API (Google Books)](https://developers.google.com/books/docs/v1/using)
- [Getting Started (Google Books)](https://developers.google.com/books/docs/v1/getting_started)
- [Dynamic Links / View API de Google Books](https://developers.google.com/books/docs/dynamic-links)
- [Open Library API](https://openlibrary.org/developers/api) (enriquecedor complementario, no fallback de catálogo)
