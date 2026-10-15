# Catbox.moe — Hosting de imágenes

**Investigación para:** Fase 6 — Almacenamiento de imágenes (subida y borrado de imágenes de la colección)
**Verificado en vivo:** 2026-10-15 (parcial: sin subir archivos)

> Comprobación en vivo limitada a propósito: **no se subió ningún archivo** a Catbox para evitar un efecto
> secundario real en un servicio externo. Ver [Verificación en vivo](#verificación-en-vivo) para lo que sí se
> comprobó y lo que quedó sin verificar.

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://catbox.moe` |
| **Endpoint de API** | `https://catbox.moe/user/api.php` |
| **Host de archivos** | `https://files.catbox.moe` |
| **Auth** | No requerida para subir. `userhash` (cuenta) opcional; **obligatorio para borrar** |
| **Rate limit** | No documentado en la web de Catbox; no verificable sin subir |
| **Gratis** | Sí, sin registro para subir |
| **Formato** | `multipart/form-data` (upload) y `application/x-www-form-urlencoded` (delete); respuesta en texto plano |
| **Documentación oficial** | No hay API formal documentada. Solo la FAQ: `https://catbox.moe/faq.php` |
| **Estado** | Activo (2026-10-15): `https://catbox.moe/` responde **200** |

## Upload / eliminación (endpoints y parámetros)

### Upload — `POST /user/api.php`

Enviado como `multipart/form-data`:

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `reqtype` | String | Sí | Valor fijo `fileupload` |
| `fileToUpload` | File | Sí | Archivo a subir |
| `userhash` | String | No | Solo lo envía el backend si `catbox.userhash` no está vacío. Asocia la subida a la cuenta |

Ejemplo:

```bash
curl -s -F "reqtype=fileupload" -F "fileToUpload=@/ruta/a/imagen.png" https://catbox.moe/user/api.php
```

**Respuesta exitosa (200):** la URL pública en texto plano, del estilo `https://files.catbox.moe/abc123.png`.
El backend exige que la respuesta empiece por `http`; si no, lanza `CatboxUploadException`.

### Eliminación — `POST /user/api.php`

Enviado como `application/x-www-form-urlencoded`:

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `reqtype` | String | Sí | Valor fijo **`deletefiles`** (plural) |
| `userhash` | String | Sí | Hash de la cuenta de Catbox |
| `files` | String | Sí | **Solo el nombre del archivo** (`abc123.png`), no la URL completa |

```bash
curl -s -d "reqtype=deletefiles" -d "files=abc123.png" -d "userhash=TU_USERHASH" https://catbox.moe/user/api.php
```

Sin `userhash`, Catbox responde **412** con `No userhash provided!` y no borra nada. El backend corta antes de
llamar: si `catbox.userhash` está vacío lanza `IllegalArgumentException` (→ HTTP **400**).

## Verificación en vivo

| Comprobación | Comando (resumen) | Resultado |
|--------------|-------------------|-----------|
| Host principal | `curl -s -o /dev/null -w "%{http_code}" https://catbox.moe/` | **200** |
| Cabeceras del host | `curl -I https://catbox.moe/` | `HTTP/2 200`, `server: nginx` |
| Host de archivos | `curl -s -o /dev/null -w "%{http_code}" https://files.catbox.moe/` | **301** |
| `api.php` sin parámetros | `curl -s -w "%{http_code}" https://catbox.moe/user/api.php` | **412** (cuerpo vacío) |
| Upload incompleto (sin `fileToUpload`) | `curl -s -w "%{http_code}" -F "reqtype=fileupload" https://catbox.moe/user/api.php` | **412**, cuerpo `No files given.` |
| Delete incompleto (sin `userhash`) | `curl -s -w "%{http_code}" -F "reqtype=deletefiles" -F "files=__nonexistent__.png" .../user/api.php` | **412**, cuerpo `No userhash provided!` |

Notas:

- El borrado probado **no borró nada**: Catbox rechazó la petición por falta de `userhash`.
- Dato de la web oficial: la portada indica *"Uploads up to 200 MB are allowed"*. La FAQ añade que los archivos
  anónimos se conservan hasta 2 años sin accesos, que los de cuenta son permanentes, que no se permiten las
  extensiones `.exe`, `.scr`, `.cpl`, `.doc*`, `.jar`, y que un GIF no puede superar 20 MB.

### Lo que NO se pudo verificar en vivo

- **La subida real (200 + URL devuelta).** No se ejecutó para no subir un archivo a un servicio externo. El
  contrato de subida se deriva del código del backend y de sus tests con mock (`CatboxClientTest`), no de una
  llamada real.
- **El borrado real.** Igual motivo; además requiere un `userhash` válido que no se dispone a mano.
- **El rate limit.** Catbox no publica un número y no se probó con carga.

## Mapeo en el sistema (flujo completo backend → Catbox → URL)

1. El cliente sube la imagen (`ImageUpload.tsx` del frontend llama a `uploadImage()` en `src/api/imagesApi.ts`)
   mediante `POST /api/v1/images/upload` como `multipart/form-data` con el campo `file`. Requiere autenticación
   (`anyRequest().authenticated()`; solo los `GET` de `/api/v1/**` son públicos).
2. `ImageStorageController.upload()` delega en `ImageStorageService.store()`.
3. El servicio **valida** el archivo (tamaño, MIME, *magic bytes*) y construye el nombre: usa el nombre original o,
   si viene vacío, `upload` + extensión derivada del MIME.
4. `CatboxClient.upload()` hace un POST `multipart/form-data` a `catbox.api.base-url + /user/api.php` con
   `reqtype=fileupload`, `fileToUpload` (y `userhash` si está configurado).
5. Catbox guarda el archivo y devuelve su URL pública (`https://files.catbox.moe/NOMBRE`), que `CatboxClient`
   devuelve al servicio y este al controlador.
6. El controlador responde **201 Created**, con cabecera `Location: URL` y cuerpo
   `ImageResponse { url, filename }`, donde `filename` es el trozo de la URL tras el último `/`.
7. El cliente guarda esa URL en el campo de imagen de la entidad correspondiente (`frontpage`, `imageUrl`,
   `thumbnailUrl`, `posterUrl`, `backdropUrl`, `avatarUrl`, `imageLargeUrl`…). En MongoDB solo se guarda la URL;
   el binario vive en Catbox.
8. Para eliminar: `DELETE /api/v1/images/{filename}` (autenticado) → `ImageStorageService.delete()` valida el
   nombre → `CatboxClient.delete()` → `POST /user/api.php` con `reqtype=deletefiles`, `files` y `userhash`.

**Manejo de errores:**

- Fallo de red o respuesta 4xx/5xx de Catbox → `CatboxUploadException` → HTTP **502 Bad Gateway**
  (`GlobalExceptionHandler.handleCatboxError`). Es decir, un rechazo de Catbox (por ejemplo el 412 de una petición
  malformada) el cliente lo ve como **502**, no como 400, y sin el detalle de Catbox.
- Nombre de archivo inválido en el borrado o `catbox.userhash` vacío → `IllegalArgumentException` → HTTP **400**.
- **No existe `ImageNotFoundException`.** El `@ApiResponse(responseCode = "404", ...)` del `DELETE` está declarado
  en la anotación pero no hay ninguna excepción ni lógica que lo produzca: un borrado no encontrado no se
  distingue de otros fallos. El frontend, de hecho, tolera el 404 al borrar (`if (!res.ok && res.status !== 404)`),
  pero el backend no lo emite.

**Observación (revisando el checkout local del frontend, en rama de trabajo):** `uploadedFilename()` en
`src/api/imagesApi.ts` detecta las imágenes borrables buscando el marcador `/api/v1/images/` dentro de la URL.
El backend devuelve URLs de `https://files.catbox.moe/...`, que no contienen ese marcador, así que esa función
devolvería `null` y el borrado automático no se dispararía para esas URLs. Conviene re-verificarlo sobre `main`
antes de darlo por bueno.

## Validaciones del backend

| Validación | Valor | Dónde |
|------------|-------|-------|
| Archivo vacío | Rechazado (400) | `ImageStorageService.validate` |
| Tamaño máximo | `5242880` bytes (5 MB) | `app.image.max-size` |
| MIME permitido | `image/jpeg`, `image/png`, `image/webp`, `image/gif` | `app.image.allowed-types` |
| Extensión | Solo las 4 anteriores, derivadas del MIME | `ImageStorageService.EXTENSION_BY_MIME` |
| *Magic bytes* | JPEG `FF D8 FF`; PNG `89 50 4E 47`; GIF `47 49 46 38`; WebP `RIFF` + `WEBP` en bytes 8–11 | `matchesMagicBytes` |
| Límite multipart | `max-file-size=5MB`, `max-request-size=10MB` | `application.properties` |
| Nombre en borrado | No vacío, sin `..`, sin `/`, sin `\` | `ImageStorageService.delete` / `CatboxClient.delete` |
| `userhash` en borrado | Obligatorio; si falta, 400 sin llamar a Catbox | `CatboxClient.delete` |
| Respuesta de subida | Debe empezar por `http`, si no → `CatboxUploadException` (502) | `CatboxClient.upload` |

El backend **no** limita el tamaño al de Catbox: su tope propio (5 MB) es más estricto que el de Catbox (200 MB).
Tampoco valida el tamaño de la respuesta más allá de comprobar que sea una URL.

## Configuración en el proyecto

| Propiedad | Valor por defecto | Uso |
|-----------|-------------------|-----|
| `catbox.api.base-url` | `https://catbox.moe` | Base del endpoint `/user/api.php` |
| `catbox.userhash` | `${CATBOX_USERHASH:}` (vacío) | Obligatorio para borrar; si está, también se envía al subir |
| `app.image.max-size` | `5242880` | Tope de la validación del servicio |
| `app.image.allowed-types` | `image/jpeg,image/png,image/webp,image/gif` | MIME aceptados |
| `spring.servlet.multipart.max-file-size` | `5MB` | Límite del contenedor |
| `spring.servlet.multipart.max-request-size` | `10MB` | Límite del contenedor |
| `http.client.connect-timeout` | `5s` | Timeout del `RestClient` |
| `http.client.read-timeout` | `10s` | Timeout del `RestClient` |

- El bean `catboxRestClient` se define en `RestClientConfig` con `baseUrl` `https://catbox.moe`; aun así
  `CatboxClient` construye la URI absoluta con `catbox.api.base-url + /user/api.php`.
- **Sin reintentos**: a diferencia de Scryfall, BGG o TMDB, Catbox no tiene `retry-attempts`/`retry-delay-ms`.

## Ventajas / Limitaciones

**Ventajas**

- Gratuito y sin registro/API key para subir.
- URL pública inmediata, lista para guardar en la entidad.
- Límite de subida amplio (200 MB según la web); el backend lo recorta a 5 MB por su cuenta.
- No recomprime ni modifica el archivo (FAQ), y no borra metadatos.
- Los archivos de cuenta son permanentes (los anónimos duran hasta 2 años sin accesos).

**Limitaciones**

- Sin API formal documentada más allá de la FAQ; los endpoints se conocen por el código y por prueba directa.
- Rate limit no documentado ni verificado.
- Servicio comunitario: sin garantía de uptime ni soporte.
- El borrado exige cuenta y `userhash`; sin él, `DELETE` siempre devuelve 400.
- Uso comercial restringido (la FAQ exige aprobación previa), relevante de cara a un despliegue público.
- Bloqueos conocidos por país/proveedor (Australia, Irán, Turquía, Comcast, Spectrum…) y posibles listas DNS.
- Contenido prohibido → borrado del archivo, baneo de cuenta y *blacklist* de IP.
- Respuestas de error con **412** (no 400/422), que el backend traduce a **502**.
- Al convertir cualquier error de Catbox en 502, el cliente pierde el motivo real del rechazo.

## Referencias

- [Catbox.moe](https://catbox.moe/)
- [FAQ de Catbox](https://catbox.moe/faq.php)
- Endpoint de API: `https://catbox.moe/user/api.php`
- Código (repo `JavierMedarde99/backend-collection`):
  - `src/main/java/com/wikicollection/infrastructure/adapter/out/catbox/CatboxClient.java`
  - `src/main/java/com/wikicollection/application/service/ImageStorageService.java`
  - `src/main/java/com/wikicollection/domain/port/out/ImageHostingClient.java`
  - `src/main/java/com/wikicollection/application/exception/CatboxUploadException.java`
  - `src/main/java/com/wikicollection/infrastructure/adapter/in/web/ImageStorageController.java`
  - `src/main/resources/application.properties`
  - Tests: `CatboxClientTest.java`, `ImageStorageServiceTest.java`, `ImageStorageControllerTest.java`

**Documentación interna relacionada:** `data-model/schema.md` (Imágenes, Fase 6), `architecture/backend.md`
(adaptador `catbox/`, `ImageStorageController`), `architecture/deployment.md` (`CATBOX_USERHASH`).
