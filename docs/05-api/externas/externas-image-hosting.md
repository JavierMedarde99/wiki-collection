# APIs Externas — Image Hosting (Catbox.moe)

## Catbox.moe (HOSTING DE IMÁGENES)

- **Base URL:** `https://catbox.moe/user/api.php`
- **Auth:** No requerida
- **Rate limit:** No documentado (uso razonable)
- **Gratis:** Sí, totalmente gratuito
- **Registro:** No necesario
- **Formato:** multipart/form-data
- **Respuesta:** URL directa de la imagen (texto plano)

### Uso de la API

#### Subir imagen

```bash
curl -s -F "reqtype=fileupload" -F "fileToUpload=@/ruta/a/imagen.png" https://catbox.moe/user/api.php
```

**Respuesta exitosa:**
```
https://files.catbox.moe/abc123.png
```

**Parámetros:**

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| `reqtype` | String | Sí | Valor fijo: `fileupload` |
| `fileToUpload` | File | Sí | Archivo de imagen a subir |

### Ejemplo en Java (Spring Boot)

```java
@Service
public class CatboxImageService {

    private final RestClient restClient;

    public CatboxImageService(RestClient.Builder restClientBuilder) {
        this.restClient = restClientBuilder
                .baseUrl("https://catbox.moe")
                .build();
    }

    public String uploadImage(byte[] imageBytes, String filename) {
        String response = restClient.post()
                .uri("/user/api.php")
                .contentType(MediaType.MULTIPART_FORM_DATA)
                .body(createMultipartBody(imageBytes, filename))
                .retrieve()
                .body(String.class);

        // Devuelve la URL directa de la imagen
        return response;
    }

    private MultiValueMap<String, Object> createMultipartBody(byte[] imageBytes, String filename) {
        MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
        body.add("reqtype", "fileupload");
        body.add("fileToUpload", new ByteArrayResource(imageBytes) {
            @Override
            public String getFilename() {
                return filename;
            }
        });
        return body;
    }
}
```

### Ejemplo en JavaScript/TypeScript (Frontend)

```typescript
async function uploadImage(file: File): Promise<string> {
    const formData = new FormData();
    formData.append('reqtype', 'fileupload');
    formData.append('fileToUpload', file);

    const response = await fetch('https://catbox.moe/user/api.php', {
        method: 'POST',
        body: formData
    });

    if (!response.ok) {
        throw new Error('Error al subir imagen');
    }

    const url = await response.text();
    return url; // https://files.catbox.moe/abc123.png
}
```

### Casos de Uso en Wiki-Collection

1. **Imágenes de perfil de usuario** — Subir avatar y obtener URL
2. **Imágenes de items** — Fotos propias de libros, juegos, cartas, etc.
3. **Backups de imágenes** — Respaldo de URLs externas que puedan expirar

### Ventajas

- ✅ Totalmente gratuito
- ✅ Sin registro ni API key
- ✅ URL directa inmediata
- ✅ Sin límite de tamaño documentado
- ✅ Soporta PNG, JPG, GIF, WebP, etc.
- ✅ Anónimo

### Desventajas

- ❌ Sin documentación oficial formal
- ❌ Sin garantía de uptime (servicio comunitario)
- ❌ Sin soporte técnico
- ❌ Puede tener rate limits no documentados

### Alternativas Evaluadas (No Funcionales)

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

### Recomendación

**Usar Catbox.moe como servicio principal de image hosting** para casos donde el usuario necesite subir imágenes propias. Para imágenes de items de APIs externas (Google Books, Scryfall, RAWG, TMDB), seguir usando las URLs originales de cada API.

---

## Integración con el Frontend

### Flujo de Subida

```
1. Usuario selecciona imagen en el frontend
2. Frontend → POST a Catbox.moe (multipart/form-data)
3. Catbox devuelve URL directa
4. Frontend almacena la URL en el estado/formulario
5. URL se persiste junto con el item (libro, juego, etc.)
```

### Consideraciones

- **Tamaño máximo:** Catbox no documenta límite, pero se recomienda < 10MB
- **Formatos soportados:** PNG, JPG, GIF, WebP, BMP, SVG
- **URL permanente:** Las URLs no expiran (según política del servicio)
- **HTTPS:** Todas las URLs usan HTTPS
