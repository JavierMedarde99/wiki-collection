# Open Library — enriquecimiento de portada y descripción por ISBN

**Investigación para:** Fase 22 — Scanner de código de barras (ISBN); enriquecimiento de la búsqueda por ISBN

**Verificado en vivo:** 2026-10-15 (peticiones `curl --max-time 20` sin clave; código leído de `JavierMedarde99/backend-collection`)

> Open Library **no** es el catálogo primario de libros del proyecto: lo es Google Books (ver `google-books.md`).
> Aquí solo se documenta el uso real que hace el backend: rellenar portada y descripción de un resultado de Google Books que viene incompleto.

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL (datos)** | `https://openlibrary.org` — fijada en `RestClientConfig`, bean `openLibraryRestClient` |
| **Base URL (portadas)** | `https://covers.openlibrary.org` — fija en la constante `COVER_URL_PATTERN` del cliente |
| **Auth** | No requerida |
| **Rate limit** | 1 petición/segundo sin identificar; 3 peticiones/segundo si se envía `User-Agent` con nombre de app + email de contacto. Portadas: 100 peticiones/IP cada 5 min, límite que **no** aplica a las claves `CoverID` (`/b/id/…`) ni `OLID` |
| **Gratis** | Sí (iniciativa de Internet Archive, sin registro obligatorio) |
| **Formato** | JSON (también RDF/XML y YAML en los mismos recursos) |
| **Registro** | La documentación pide registrar la aplicación y enviar `User-Agent` identificativo. El backend **no** hace ninguna de las dos cosas |
| **Documentación oficial** | https://openlibrary.org/developers/api |
| **Estado** | Activa y mantenida; la página de APIs se editó por última vez el 2026-10-09 |

## Endpoints

Solo hay **un** archivo en el paquete del adaptador (`infrastructure/adapter/out/openlibrary/OpenLibraryClient.java`), y consume exactamente dos endpoints de datos más uno de portadas.

| Uso en el backend | Endpoint | Parámetros | Notas |
|-------------------|----------|-----------|-------|
| Edición por ISBN (1ª llamada) | `GET /isbn/{isbn}.json` | `isbn` tal cual llega tras normalizar (se quitan espacios y guiones) | Responde **302** a `/books/{OLID}.json`, que es el JSON real (200). Nunca devuelve 404 si el ISBN existe en el catálogo |
| Descripción de la obra (2ª llamada) | `GET /works/{workKey}.json` | `workKey` = `edition.works[0].key` (viene con la barra inicial, p. ej. `/works/OL45804W`) | Solo se llama si falta la descripción y la edición trae `works` |
| Portada | `GET https://covers.openlibrary.org/b/id/{coverId}-M.jpg` | `coverId` = `edition.covers[0]` | URL construida en el código, no por el cliente RestClient. Tamaño fijo `M` (mediano) |

Endpoints de Open Library **no** usados por el backend (no aparecen en el código, aunque existen): `/search.json` (verificado 200), `/authors/{key}.json`, `/books/{olid}.json` (solo se llega por redirección), la API de Subjects y la de Lists.

> Detalle comprobado: `/isbn/{isbn}/editions.json` **no** es una ruta independiente en la práctica — para `9780140328721` redirige igualmente a `/books/OL7353617M.json` (200), el mismo JSON que `/isbn/{isbn}.json`. El código no la usa, así que no afecta al proyecto.

### Verificación en vivo (2026-10-15)

| Petición | Código HTTP | Forma de la respuesta |
|----------|-------------|----------------------|
| `/isbn/9780140328721.json` | **302** → `/books/OL7353617M.json`, que responde **200** `application/json` (1.754 B) | Edición: `title`, `authors[].key`, `covers` (`[15152634, 8739161, -1]`), `works[].key` (`/works/OL45804W`), `number_of_pages`, `publishers`, `isbn_10`, `isbn_13`, `languages`… |
| `/works/OL45804W.json` | **200** `application/json` (2.660 B) | `description` es un **string** |
| `/works/OL103123W.json` | **200** `application/json` (4.836 B) | `description` es un **objeto** `{"type": "/type/text", "value": "…"}` |
| `covers.openlibrary.org/b/id/240727-M.jpg` | **200** `image/jpeg` (10.729 B) | JPEG real, `cache-control: public`, `expires` a 100 años |
| `covers.openlibrary.org/b/id/15152634-M.jpg` | **200** `image/jpeg` (24.354 B) | Es el `covers[0]` de la edición del ISBN de prueba → la portada funciona con el primer id |
| `covers.openlibrary.org/b/id/-1-M.jpg` | **200** `image/gif` (43 B) | `-1` aparece en los arrays `covers` de ediciones reales (p. ej. en la última posición del array del ISBN de prueba) y su URL devuelve una imagen en blanco, no un 404 |
| `/isbn/9788498382671.json` | **404** `text/html` | ISBN con formato válido que Open Library no tiene. Es casualmente el ISBN del fixture de `OpenLibraryClientTest` |
| `/isbn/0-14-032872-6.json` | **302** → `/books/OL7353617M.json` | Acepta ISBN-10 con guiones |
| `/isbn/9999999999999.json` | **302** → `/books/OL20350183M.json` (200) | Devuelve una edición cuyo `isbn_13` es literalmente `9999999999999`: el emparejamiento es por campo ISBN del catálogo, sin validar el dígito de control |
| `/isbn/9781234567897.json` | **302** → `/books/OL25427832M.json` (200) | La edición devuelta lista `isbn_10: ["123456789X"]`, no el ISBN pedido → el emparejamiento puede no ser exacto |
| `/isbn/abcdef.json` | **404** | Entrada no numérica |
| `/search.json?q=fahrenheit+451&limit=1` | **200** | Endpoint existente, pero **no** usado por el backend |

### Redirecciones (302)

`/isbn/{isbn}.json` responde siempre 302 (a `/books/{OLID}.json`) cuando encuentra edición. El cliente lo sigue de forma transparente porque:

- `HttpClientProperties.restClientBuilder()` usa `SimpleClientHttpRequestFactory`.
- Esa factoría invoca `HttpURLConnection.setInstanceFollowRedirects("GET".equals(httpMethod))` en spring-web 7.0.9 (comprobado en el bytecode del jar local), es decir, activa el seguimiento para GET.
- Comprobado además con una prueba suelta de `HttpURLConnection`: `GET /isbn/9780140328721.json` acaba en **200** con el JSON de `/books/OL7353617M.json`.

Ojo: con `java.net.http.HttpClient` por defecto (política `NEVER`) la misma petición devolvería **302** sin cuerpo JSON. Si algún día se cambia la factoría HTTP, el cliente dejaría de leer la edición.

## Mapeo: Open Library → modelo interno

El destino no es la entidad `Book`, sino el DTO de búsqueda `BookSearchResult` (`domain/model/BookSearchResult.java`), que es lo que devuelve `GET /api/v1/books/search`.

| Campo Open Library | Campo interno (`BookSearchResult`) | Tipo | Notas |
|--------------------|------------------------------------|------|-------|
| `covers[0]` | `coverImage` | String | Solo si `coverImage` venía vacío. Se transforma en `https://covers.openlibrary.org/b/id/{id}-M.jpg`; se toma **siempre el primer** id, sin filtrar |
| `works[0].key` | — (intermedio) | String | Se usa para construir `/works/{key}.json`; no se persiste |
| `works[0].description` (de la obra) | `description` | String | Solo si `description` venía vacía. Acepta las dos formas: `String` o `{"type": "/type/text", "value": "…"}`. Si es otra cosa, queda `null` |
| `covers`, `title`, `authors`, `publish_date`, `publishers`, `number_of_pages`, `isbn_10`, `isbn_13`, `languages`, `key`, `subjects`, `first_sentence` | — | — | **No se usan**: esos campos se conservan tal cual llegaron de Google Books |

Reglas de sustitución (en `OpenLibraryClient.enrich`):

- Solo actúa si `isbn` del resultado no está vacío y falta portada, descripción o ambas.
- Si ya había portada, no se toca; si ya había descripción, no se toca.
- El resto de campos se reenvían idénticos en un `BookSearchResult` nuevo (copia sin mutar el original).
- Después, en el frontend, `coverImage` pre-rellena el campo `frontpage` del formulario y `description` la sinopsis (`BookIsbnScan.tsx` / `BookSearch.tsx`); es la vía por la que acaban en el documento `Book` (ver `data-model/schema.md`).

## Flujo de enriquecimiento en el proyecto

**Quién llama:** `application/service/BookSearchService.searchByIsbn(...)` → `domain/port/out/ExternalBookEnricher` → `OpenLibraryClient`. El puerto tiene una sola implementación, así que Open Library es el único enriquecedor.

**Cuándo NO se llama:** en la búsqueda por título (`BookSearchService.search(query, …)`) no se invoca al enriquecedor en absoluto. El enriquecimiento existe solo en la ruta por ISBN.

**Disparador HTTP:** `GET /api/v1/books/search?isbn={isbn}` (público, sin auth; `isbn` limitado a 20 caracteres).

Pasos:

1. `BookController` delega en `searchByIsbn` cuando llega `isbn` (si llegan `isbn` y `name`, gana `isbn`).
2. `BookSearchService.normalizeIsbn` quita espacios y guiones.
3. `GoogleBooksClient.searchByIsbn(isbn)` hace la búsqueda principal en Google Books.
4. Por cada resultado, `OpenLibraryClient.enrich(result)`: si falta portada o descripción, llama a `/isbn/{isbn}.json`; si además falta la descripción y la edición tiene `works[0]`, llama a `/works/{key}.json`.
5. Todo queda **best-effort**: cualquier `RestClientException` (incluye 404, 5xx y timeouts) se registra con `log.warn` y se devuelve el resultado original sin enriquecer.

**Caché:** la caché declarada es `CacheConfig.BOOK_ENRICH` (nombre `bookEnrich`), Caffeine en memoria, `expireAfterWrite` de 3600 s (`app.cache.ttl.books`) y `maximumSize` 1000 (`app.cache.max-size`).

> **Aviso (reproducido, no visto con el backend arrancado).** La anotación real es
> `@Cacheable(cacheNames = CacheConfig.BOOK_ENRICH, key = "#result.isbn() ?: ''")`, y `#result`
> **no** está disponible al calcular la clave de un `@Cacheable` (solo en `@CachePut`, `@CacheEvict`
> y en `unless`). Reproducido con un banco de pruebas mínimo con `spring-context` 7.0.9 — la versión
> que resuelve Spring Boot 4.1.1 (`spring-boot-dependencies-4.1.1` → `spring-framework.version=7.0.9`):
> la evaluación de la clave lanza `SpelEvaluationException: EL1011E: Method call: Attempted to call
> method isbn() on null context object` **antes** de entrar al método, así que el cuerpo no llega a
> ejecutarse y la excepción se propaga hacia arriba (no la captura el `catch (RestClientException)`).
> Con la clave corregida a `#p0.isbn() ?: ''` la misma prueba funciona y la segunda llamada con el
> mismo ISBN sí es un acierto de caché. Los tests actuales no lo detectan: `OpenLibraryClientTest`
> instancia el cliente con `new` (sin proxy de caché) y `BookSearchServiceTest` mockea el puerto.
> No se ha arrancado el backend ni se ha llamado al endpoint para confirmar el efecto en producción.

**Reintentos: no hay.** A diferencia de Scryfall (`scryfall.api.retry-attempts`), BGG y TMDB, Open Library no tiene propiedades de reintento ni interceptor de reintentos en el código. Los únicos límites son los timeouts HTTP genéricos.

## Configuración en el proyecto

**No existe ninguna propiedad de Open Library** en `src/main/resources/application.properties` (búsqueda de `openlibrary` sin coincidencias). Por tanto:

| Elemento | Dónde está | Valor |
|----------|-----------|-------|
| URL base de datos | `infrastructure/config/RestClientConfig.java`, bean `openLibraryRestClient` | `https://openlibrary.org` (fija en el código) |
| URL base de portadas | `OpenLibraryClient.COVER_URL_PATTERN` | `https://covers.openlibrary.org/b/id/%s-M.jpg` (fija en el código) |
| API key | — | No existe: Open Library no la pide |
| Reintentos | — | No configurados |
| `User-Agent` identificativo | — | El código no envía ninguno (los docs lo piden para triplicar el límite) |
| Timeouts | `application.properties` (`http.client.*`) | `connect-timeout=5s`, `read-timeout=10s` |
| Caché | `application.properties` (`app.cache.*`) | `app.cache.ttl.books=3600s`, `app.cache.max-size=1000` |

## Limitaciones

- **Cobertura desigual.** Es un catálogo mantenido por voluntarios: hay ISBNs con formato válido que devuelven 404 (verificado: `9788498382671`), y entradas con ISBNs basura.
- **Emparejamiento no exacto.** El `302` de `/isbn/{isbn}.json` puede devolver una edición cuyo `isbn_13`/`isbn_10` no coincide con el pedido (visto con `9781234567897`). El cliente no lo comprueba: no valida que la edición recibida corresponda al ISBN solicitado.
- **La descripción es de la *obra*, no de la edición.** Si el catálogo tiene mal vinculada la edición a su obra, la sinopsis puede ser de otro libro. Además, todas las ediciones de una obra comparten la misma descripción.
- **`covers` puede traer `-1`** (valor centinela de "sin portada" en el catálogo; no está documentado en el Covers API oficial). El cliente toma `covers[0]` sin filtrar, así que una edición con `-1` en la primera posición produciría una portada en blanco con **HTTP 200** (un GIF de 43 B), sin error visible. No he encontrado en vivo ninguna edición con `-1` en primera posición: en el caso observado aparece en la última.
- **Sin `?default=false`.** Las portadas ausentes devuelven una imagen en blanco en vez de 404 (comportamiento por defecto del servicio de portadas), así que no hay señal de fallo.
- **Sin reintentos ni resiliencia.** Un 5xx o un timeout descarta el enriquecimiento de ese libro y sigue con el resto.
- **Rate limit bajo (1 req/s).** Cada ISBN puede costar 1–2 peticiones; en un escaneo en ráfaga conviene la caché por ISBN (que, tal como está escrita hoy, no está funcionando — ver el aviso de la sección de flujo).
- **Solo dos campos.** El enriquecimiento no aporta autores, páginas, editorial ni categorías: eso viene siempre de Google Books.
- **No verificado:** no he arrancado el backend ni he consultado `GET /api/v1/books/search?isbn=…`; el efecto real del fallo de caché descrito arriba está reproducido en aislamiento, no contra la aplicación en ejecución. Tampoco he comprobado si la aplicación está registrada ante Open Library (el código no manda credenciales ni identificador).

## Referencias

- [Open Library APIs (documentación oficial y límites)](https://openlibrary.org/developers/api)
- [Open Library Covers API](https://openlibrary.org/dev/docs/api/covers)
- [Work & Edition APIs](https://openlibrary.org/dev/docs/api/books)
- [Datos en bloque (dumps mensuales)](https://openlibrary.org/developers/dumps)
- [Código fuente de Open Library](https://github.com/internetarchive/openlibrary)
- Ficha relacionada: [`google-books.md`](google-books.md) — catálogo primario de libros
- Spec relacionada: [`../../specs/books/spec.md`](../../specs/books/spec.md) — modelo `BookSearchResult` y endpoint `/api/v1/books/search`
