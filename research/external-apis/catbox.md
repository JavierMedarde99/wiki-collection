# Catbox.moe — Image Hosting

**Investigación para:** Fase 6 — Almacenamiento de Imágenes

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://catbox.moe/user/api.php` |
| **Auth** | No requerida (upload), userhash (delete) |
| **Rate limit** | No documentado (uso razonable) |
| **Gratis** | Sí, totalmente gratuito |
| **Registro** | No necesario para upload |
| **Formato** | multipart/form-data (upload), texto plano (response) |
| **Estado** | Activo (2026) |

## Upload de Imagen

```bash
curl -s -F "reqtype=fileupload" -F "fileToUpload=@/ruta/a/imagen.png" https://catbox.moe/user/api.php
```

**Respuesta exitosa:**
```
https://files.catbox.moe/abc123.png
```

### Parámetros de Upload

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `reqtype` | String | Sí | Valor fijo: `fileupload` |
| `fileToUpload` | File | Sí | Archivo de imagen a subir |
| `savename` | String | No | Nombre personalizado (opcional) |

## Eliminación de Imagen

Para eliminar se requiere un userhash (obtenido tras registrarse en Catbox):

```bash
curl -s -F "reqtype=deletefile" -F "file=Catbox.moe/abc123.png" -F "userhash=TU_USERHASH" https://catbox.moe/user/api.php
```

## Mapeo en el Sistema

El flujo completo es:

1. Cliente sube imagen vía `POST /api/v1/images/upload`
2. Backend valida (5MB máximo, MIME permitido, magic bytes)
3. Backend sube a Catbox vía `CatboxClient`
4. Backend devuelve `ImageResponse` con `url` y `filename`
5. URL se almacena en el campo de imagen de cada entidad (`frontpage`, `thumbnailUrl`, `posterUrl`, etc.)
6. Para eliminar: `DELETE /api/v1/images/{filename}` → elimina de Catbox

## Casos de Uso en Wiki-Collection

1. **Imágenes de perfil de usuario** — Subir avatar y obtener URL
2. **Imágenes de items** — Fotos propias de libros, juegos, cartas, etc.
3. **Backups de imágenes** — Respaldo de URLs externas que puedan expirar

## Ventajas

- ✅ Totalmente gratuito
- ✅ Sin registro ni API key para upload
- ✅ URL directa inmediata
- ✅ Sin límite de tamaño documentado
- ✅ Soporta PNG, JPG, GIF, WebP, BMP, SVG
- ✅ Anónimo

## Desventajas

- ❌ Sin documentación oficial formal
- ❌ Sin garantía de uptime (servicio comunitario)
- ❌ Sin soporte técnico
- ❌ Puede tener rate limits no documentados
- ❌ Eliminación requiere userhash (registro necesario)

## Alternativas Evaluadas (No Funcionales)

| Servicio | Estado |
|----------|--------|
| **ImgBB** | Requiere API key (registro) |
| **Freeimage.host** | Requiere API key / bloqueado |
| **sm.ms** | Requiere API key |
| **0x0.st** | Cerrado (spam) |
| **transfer.sh** | No disponible |
| **pomf.lain.la** | Cerrado |
| **files.fm** | Cloudflare challenge |
| **file.io** | API no disponible sin cuenta |
| **bayfiles.com** | No funcional |
| **telegra.ph** | Error en upload |

## Validaciones del Backend

| Validación | Valor |
|------------|-------|
| Tamaño máximo | 5 MB |
| MIME permitido | image/jpeg, image/png, image/gif, image/webp |
| Verificación magic bytes | Sí |

## Configuración

| Propiedad | Descripción |
|-----------|-------------|
| `catbox.api.base-url` | URL base de Catbox (por defecto: `https://catbox.moe`) |
| `catbox.userhash` | Userhash para poder borrar imágenes (opcional) |

## Referencias

- [Catbox.moe](https://catbox.moe/)
- [Catbox API (no oficial)](https://catbox.moe/user/api.php)
