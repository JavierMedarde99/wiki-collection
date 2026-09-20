# Futuro — Plataformas de streaming en películas/series

## Estado: Investigación / No implementado

## 1. Qué es

Mostrar en la ficha de cada película/serie qué plataformas de streaming la tienen disponible en España (Netflix, HBO Max, Disney+, Amazon Prime, etc.) mediante los logos de los proveedores, con posibilidad de hacer click para ir a buscar el título en esa plataforma.

**Objetivo:** el usuario ve "¿Dónde ver Matrix?" y la respuesta es: Netflix, HBO Max, Amazon Prime — con logos clicables que abren la búsqueda del título en cada plataforma.

---

## 2. API externa: TMDB Watch Providers

### 2.1 Endpoints relevantes

```
GET /3/movie/{movie_id}/watch/providers?country=ES
GET /3/tv/{series_id}/watch/providers?country=ES
GET /3/watch/providers/regions                        → países soportados
GET /3/watch/providers/movie?watch_region=ES          → proveedores activos en ES (para movies)
GET /3/watch/providers/tv?watch_region=ES             → proveedores activos en ES (para TV)
```

### 2.2 Response real de TMDB

```json
{
  "id": 550,
  "results": {
    "ES": {
      "flatrate": [
        { "provider_id": 10,  "provider_name": "Netflix",       "logo_path": "/xqNq5mkjpH0kVqQzgK26kKgF2sv.jpg", "provider_country": "US" },
        { "provider_id": 22,  "provider_name": "HBO Max",       "logo_path": "/oqAbd7Gk2YQMnR4jM0p9mlp6Bj2.jpg", "provider_country": "ES" },
        { "provider_id": 51,  "provider_name": "Disney+",       "logo_path": "/8aO0lruPWAi4Jyk23UUGnMqC2Q3.jpg", "provider_country": "US" }
      ],
      "buy": [
        { "provider_id": 100, "provider_name": "Google Play",   "logo_path": "...", "provider_country": "US" }
      ],
      "rent": [
        { "provider_id": 101, "provider_name": "Apple TV",      "logo_path": "...", "provider_country": "ES" }
      ]
    }
  }
}
```

**Campos por proveedor:**
- `provider_id` — ID numérico estable de TMDB
- `provider_name` — "Netflix", "HBO Max", "Amazon Prime Video"...
- `logo_path` — ruta relativa. URL completa: `https://image.tmdb.org/t/p/original{logo_path}`
- `provider_country` — país del proveedor

**Tipos de acceso:**
- `flatrate` → suscripción (Netflix, HBO, Disney+...)
- `buy` → compra digital
- `rent` → alquiler

### 2.3 Qué NO devuelve TMDB (importante)

**TMDB NO incluye URLs o deep links a las plataformas.** No hay campo `url`, `link`, `href` ni similar. Esta es una limitación conocida y deliberada de TMDB — los deep links son específicos de cada servicio y cambian.

### 2.4 Datos por país

- TMDB soporta cerca de 50 países con datos de proveedores.
- Para España se usa `?country=ES` o `?watch_region=ES`.
- No todas las películas/series tienen datos de proveedores (TMDB los recopila de fuentes públicas).
- Para series existe endpoint por temporada: `/3/tv/{id}/season/{n}/watch/providers`.

---

## 3. Backend — Cambios necesarios

### 3.1 Estado actual

**`TmdbClient.java`** (documentación oficial: `docs/05-api/externas/externas-movies.md`):
- Solo hace búsqueda (`/search/movie`, `/search/tv`) y las peticiones básicas de detalle.
- No consulta el endpoint `/watch/providers`.
- `TmdbEntry` (record interno) solo tiene: id, title, name, overview, release_date, first_air_date, poster_path, backdrop_path, vote_average.

**`MovieShow.java`** (domain model):
```java
// Campos actuales
id | externalId | title | overview | releaseDate | posterUrl | backdropUrl |
voteAverage | mediaType | status | userRating | comment | dateAdded |
dateCompleted | externalSource | createdAt | updatedAt
```
**Cero campos de streaming.**

### 3.2 Cliente TMDB — nuevo método

**Archivo:** `infrastructure/adapter/out/tmdb/TmdbClient.java`

Añadir método que consulta `/watch/providers`:

```java
@Override
public WatchProvidersResult getWatchProviders(Long tmdbId, MovieMediaType mediaType, String country) {
    String path = mediaType == MovieMediaType.MOVIE
        ? "/movie/{id}/watch/providers"
        : "/tv/{id}/watch/providers";
    WatchProvidersResponse response = executeWithRetry(() ->
        tmdbRestClient.get()
            .uri(uriBuilder -> uriBuilder
                .path(path)
                .build(tmdbId))
            .retrieve()
            .body(WatchProvidersResponse.class));
    if (response == null || response.results() == null) {
        return new WatchProvidersResult(null, Collections.emptyMap());
    }
    return new WatchProvidersResult(response.id(), response.results());
}
```

**Response records (nuevos):**

```java
@JsonIgnoreProperties(ignoreUnknown = true)
record WatchProvidersResponse(
    Long id,
    Map<String, WatchProvidersByCountry> results) {}

@JsonIgnoreProperties(ignoreUnknown = true)
record WatchProvidersByCountry(
    List<WatchProvider> flatrate,
    List<WatchProvider> buy,
    List<WatchProvider> rent) {}

@JsonIgnoreProperties(ignoreUnknown = true)
record WatchProvider(
    Integer providerId,
    String providerName,
    String logoPath,
    String providerCountry) {}
```

### 3.3 Modelo de dominio — nuevos campos

**Archivo:** `domain/model/MovieShow.java`

Añadir campos opcionales para persistir los proveedores:

```java
private List<StreamingProvider> streamingProviders;
private String watchCountry;     // país usado para consultar proveedores (ej: "ES")
```

```java
@Getter @Setter
public class StreamingProvider {
    private Integer providerId;       // ID de TMDB del proveedor
    private String providerName;      // "Netflix", "HBO Max"...
    private String logoUrl;           // URL completa del logo
    private ProviderAccessType type;  // FLATRATE, BUY, RENT
    private String deepLinkUrl;       // URL construida (ver sección 4)
}
```

```java
public enum ProviderAccessType {
    FLATRATE,   // suscripción
    BUY,        // compra digital
    RENT        // alquiler
}
```

**Archivo:** `infrastructure/adapter/out/persistence/MovieShowEntity.java`

Añadir con `@Field`:
```java
@Field("streamingProviders")
private List<StreamingProviderEntity> streamingProviders;

@Field("watchCountry")
private String watchCountry;
```

Necesario un `StreamingProviderEntity` (subdocumento) con los mismos campos mapeados.

### 3.4 DTOs

**`MovieShowRequest.java`** — añadir:
```java
List<StreamingProviderRequest> streamingProviders,
String watchCountry,
```

**`MovieShowResponse.java`** — añadir lo mismo (para que el frontend reciba los datos).

**`StreamingProviderRequest.java`** (nuevo DTO):
```java
public record StreamingProviderRequest(
    Integer providerId,
    String providerName,
    String logoUrl,
    String type,          // "FLATRATE" | "BUY" | "RENT"
    String deepLinkUrl
) {}
```

### 3.5 Servicio — consulta de proveedores

Opcional pero recomendado: un método en `MovieShowService` que, dado un `MovieShow`, consulta TMDB y devuelve los proveedores. Esto permite:

1. Al crear/editar desde búsqueda de TMDB: ya se tienen los datos del SearcResult, pero los proveedores requieren petición extra.
2. Al ver la ficha: si no se tienen persistidos, se pueden consultar al vuelo (o mostrarlo vacío y dejar que el usuario refresque).

**Flujo recomendado:**
1. Al crear/actualizar un MovieShow desde búsqueda → consultar `getWatchProviders` → persistir en MovieShow.
2. Al visualizar MovieShow → mostrar los persistidos (cero petición extra en tiempo de vista).
3. Botón "Actualizar proveedores" en detalle → re-consulta TMDB y actualiza.

### 3.6 Archivos afectados — resumen

| Capa | Archivo | Cambio |
|------|---------|--------|
| **Domain** | `MovieShow.java` | Añadir `streamingProviders` + `watchCountry` |
| **Domain** | `StreamingProvider.java` (nuevo) | Clase anidada o archivo nuevo con providerId, providerName, logoUrl, type, deepLinkUrl |
| **Domain** | `ProviderAccessType.java` (nuevo enum) | FLATRATE, BUY, RENT |
| **Entity** | `MovieShowEntity.java` | Añadir campos con `@Field` + `StreamingProviderEntity` |
| **Entity mapper** | `MovieShowEntityMapper.java` | Mapear lista de providers ida/vuelta |
| **DTO request** | `MovieShowRequest.java` | Añadir campos opcionales |
| **DTO response** | `MovieShowResponse.java` | Añadir campos |
| **DTO mapper** | `MovieShowDtoMapper.java` | Mapear proveedores domain ↔ DTO |
| **External client** | `TmdbClient.java` | Añadir `getWatchProviders()` + records de response |
| **Service** | `MovieShowService.java` | Opcional: método auxiliar para consultar + persistir proveedores |
| **Controller** | `MovieShowController.java` | Opcional: endpoint para refrescar proveedores |

---

## 4. Mapeo de provider_id a URLs de plataformas (Opción A)

### 4.1 Tabla de proveedores relevantes para España

| provider_id | provider_name | Logo TMDB | Plataforma | URL de búsqueda propuesta |
|-------------|---------------|-----------|------------|---------------------------|
| 10 | Netflix | `/xqNq5mkjpH0kVqQzgK26kKgF2sv.jpg` | Netflix | `https://www.netflix.com/search?q={title}` |
| 22 | HBO Max | `/oqAbd7Gk2YQMnR4jM0p9mlp6Bj2.jpg` | Max (HBO Max) | `https://www.max.com/search?q={title}` |
| 51 | Disney+ | `/8aO0lruPWAi4Jyk23UUGnMqC2Q3.jpg` | Disney+ | `https://www.disneyplus.com/search?q={title}` |
| 110 | Amazon Prime Video | (logo propio) | Amazon Prime | `https://www.primevideo.com/search?q={title}` |
| 103 | Apple TV | (logo propio) | Apple TV | `https://tv.apple.com/search?q={title}` |
| 121 | Google Play Movies | (logo propio) | Google Play | `https://play.google.com/store/movies?q={title}` |
| 128 | Rakuten TV | (logo propio) | Rakuten TV | `https://www.rakuten.com/search?q={title}` |
| 153 | Filmin | (logo propio) | Filmin | `https://www.filmin.com/search?q={title}` |
| 136 | MUBI | (logo propio) | MUBI | `https://mubi.com/en/search?q={title}` |
| 332 | Paramount+ | (logo propio) | Paramount+ | `https://www.paramountplus.com/search?q={title}` |
| 368 | Star+ | (logo propio) | Star+ | `https://www.starplus.com/search?q={title}` |
| 449 | Movistar Plus+ | (logo propio) | Movistar Plus+ | `https://plus.movistar.es/search?q={title}` |

### 4.2 Implementación del mapeo

**Archivo:** `infrastructure/adapter/out/tmdb/ProviderUrlMapper.java` (nuevo)

```java
@Component
public class ProviderUrlMapper {

    // Mapa de provider_id → URL base de búsqueda
    // Los placeholders {title} se reemplazan con el título URL-encoded
    private static final Map<Integer, String> PROVIDER_URLS = Map.of(
        10,  "https://www.netflix.com/search?q={title}",
        22,  "https://www.max.com/search?q={title}",
        51,  "https://www.disneyplus.com/search?q={title}",
        110, "https://www.primevideo.com/search?q={title}",
        103, "https://tv.apple.com/search?q={title}",
        121, "https://play.google.com/store/movies?q={title}",
        128, "https://www.rakuten.com/search?q={title}",
        153, "https://www.filmin.com/search?q={title}",
        136, "https://mubi.com/en/search?q={title}",
        332, "https://www.paramountplus.com/search?q={title}",
        368, "https://www.starplus.com/search?q={title}",
        449, "https://plus.movistar.es/search?q={title}"
    );

    /**
     * Construye la URL de búsqueda para un provider dado.
     * Si el provider_id no está mapeado, devuelve null (solo logo informativo, sin enlace).
     */
    public String buildDeepLink(Integer providerId, String title) {
        String template = PROVIDER_URLS.get(providerId);
        if (template == null) {
            return null;
        }
        String encodedTitle = UriUtils.encode(title, StandardCharsets.UTF_8);
        return template.replace("{title}", encodedTitle);
    }

    /**
     * Obtiene la URL del logo de TMDB completo.
     */
    public String buildLogoUrl(String logoPath) {
        if (logoPath == null || logoPath.isBlank()) {
            return null;
        }
        return "https://image.tmdb.org/t/p/original" + logoPath;
    }
}
```

**Comportamiento:**
- Si el `provider_id` está en el mapa → se genera `deepLinkUrl` con la búsqueda del título.
- Si el `provider_id` NO está en el mapa → `deepLinkUrl = null`. El frontend muestra solo el logo sin enlace (Opción C para ese proveedor en particular).

### 4.3 Extensibilidad

El mapa es un `Map.of(...)` estático, pero se puede externalizar a `application.properties` o a una tabla lookup si se quiere hacer editable sin recompilar:

```properties
# application.properties (alternativa avanzada)
tmdb.providers.netflix.id=10
tmdb.providers.netflix.url=https://www.netflix.com/search?q={title}
tmdb.providers.hbo.id=22
tmdb.providers.hbo.url=https://www.max.com/search?q={title}
...
```

Para el alcance inicial, el `Map.of` hardcode es suficiente y simple.

### 4.4 Proveedores sin URL mapeada

TMDB tiene muchos más provider_ids además de los listados (cientos, incluyendo cadenas de TV por pago, tiendas de VOD regionales, etc.). No todos merecen deep link. La regla:

- **Tiendas de streaming grandes con presencia en España** → incluir en el mapa.
- **Proveedores pequeños, regionales, o sin web clara** → solo logo informativo sin enlace.
- **Compra/rent individual** → el deep link a Google Play, Apple TV, Rakuten sí es útil. Para otros proveedores de buy/rent no mapeados, solo logo.

---

## 5. Frontend — Cambios necesarios

### 5.1 Tipos TypeScript

**Archivo:** `src/types/MovieShow.ts`

```typescript
export interface StreamingProvider {
  providerId: number;
  providerName: string;
  logoUrl?: string;
  type: 'FLATRATE' | 'BUY' | 'RENT';
  deepLinkUrl?: string;   // si está definido, el logo es clicables
}

export interface MovieShow {
  id: string;
  externalId?: string;
  title: string;
  overview?: string;
  releaseDate?: string;
  posterUrl?: string;
  backdropUrl?: string;
  voteAverage?: number;
  mediaType: MediaType;
  status: MovieShowStatus;
  userRating?: number;
  comment?: string;
  dateAdded?: string;
  dateCompleted?: string;
  externalSource?: string;
  streamingProviders?: StreamingProvider[];  // ← NUEVO
  watchCountry?: string;                      // ← NUEVO
}
```

**Archivo:** `src/types/MovieShowFormData.ts` (o dentro de `MovieShow.ts`)

Añadir `streamingProviders?: StreamingProvider[]` y `watchCountry?: string`.

### 5.2 Nuevo componente: `StreamingProviderBadges`

**Archivo:** `src/components/StreamingProviderBadges.tsx`

**Props:**
```typescript
interface StreamingProviderBadgesProps {
  providers: StreamingProvider[];   // lista de proveedores del MovieShow
  title: string;                    // título para construir deep links si hace falta
  showLabels?: boolean;             // mostrar "Netflix" debajo del logo (default: true)
  size?: 'sm' | 'md' | 'lg';        // tamaño de los logos
}
```

**Comportamiento:**
- Por cada provider en `providers`:
  - Si `logoUrl` existe → mostrar `<img>` con el logo de TMDB.
  - Si `deepLinkUrl` existe → envolver en `<a href={deepLinkUrl} target="_blank" rel="noopener noreferrer">`.
  - Si no hay `deepLinkUrl` → solo logo, sin link (cursor default, title="No disponible enlace directo").
  - Si `showLabels` → poner debajo el `providerName` en texto pequeño.

**Ejemplo visual:**
```
[Netflix logo] [HBO Max logo] [Disney+ logo] [Amazon logo] ...
   Netflix        HBO Max        Disney+       Amazon
```

**Si no hay proveedores:** mostrar mensaje "No hay información de streaming disponible para este título en España".

### 5.3 Actualización de componentes existentes

**`MovieShowCard.tsx`** (listado):
- Añadir una fila pequeña de logos de proveedores debajo del título, o solo los logos como badges miniaturas.
- Solo mostrar si `streamingProviders` tiene elementos y `providers.length > 0`.
- Tamaño pequeño (`sm`), sin labels o con label muy discretos.

**`MovieShowDetailPage.tsx`**:
- Añadir sección "Disponible en" con `StreamingProviderBadges` en tamaño `md` o `lg`.
- Mostrar separación por tipo: "Suscripción", "Compra", "Alquiler" si hay proveedores de cada tipo.
- Botón "Actualizar proveedores" que llama a un endpoint del backend para re-consultar TMDB (opcional).

**`MovieShowForm.tsx`**:
- No mostrar campo de proveedores en el formulario de creación/edición manual.
- Los proveedores se poblan automáticamente al crear/actualizar desde búsqueda de TMDB (el servicio backend los consulta y los persiste).
- El usuario no edita manualmente los proveedores — se obtienen de TMDB.

### 5.4 API client

**Archivo:** `src/api/movieshowsApi.ts`

No cambia la estructura básica (GET/POST/PUT/DELETE). Los nuevos campos `streamingProviders` ya vienen en `MovieShowResponse` del backend, así que el frontend los recibe automáticamente sin cambios en el API client.

**Opcional:** añadir función `refreshMovieShowProviders(id: string)` que llama a un endpoint `POST /api/v1/movieshows/{id}/refresh-providers` (si se implementa en backend) para re-consultar TMDB sin editar el resto del documento.

### 5.5 Constantes

**Archivo:** `src/constants/movieshows.ts`

No requiere cambios significativos. Opcionalmente añadir constantes para tamaños de logos si se usan varios.

---

## 6. Flujo completo de usuario

### 6.1 Crear película/serie con proveedores (desde búsqueda TMDB)

```
MovieShowCreatePage
├── Usuario escribe "Matrix" en MovieShowSearch
├── TMDB devuelve resultados (título, sinopsis, póster...)
├── Usuario selecciona Matrix (ID 550)
├── Backend (MovieShowService.crearDesdeBusqueda):
│   ├── Crea MovieShow con datos básicos de búsqueda
│   ├── Llama a TmdbClient.getWatchProviders(550, MOVIE, "ES")
│   ├── TMDB devuelve: Netflix, HBO Max, Amazon Prime, Google Play...
│   ├── ProviderUrlMapper.construye deep links para cada uno
│   ├── Persiste streamingProviders en el MovieShow
│   └── Devuelve MovieShow completo al frontend
└── Frontend navega a MovieShowDetailPage → ya tiene los proveedores
```

### 6.2 Ver película/serie con proveedores

```
MovieShowDetailPage
├── Título: Matrix (1999)
├── Póster + sinopsis + fecha
├── ...
├── Sección "Disponible en España" (solo si status !== PLAN_TO_WATCH o siempre)
│   ├── [Netflix logo] [HBO Max logo] [Amazon Prime logo] [Google Play logo]
│   │   Netflix         HBO Max         Amazon Prime     Google Play
│   └── (cada logo es un enlace a la búsqueda del título en esa plataforma)
└── ...
```

### 6.3 Actualizar proveedores manualmente

```
MovieShowDetailPage
├── ...
├── Botón "⟳ Actualizar disponibilidad" (oculto si no hay usuario autenticado, o siempre)
├── Click → POST /api/v1/movieshows/{id}/refresh-providers
├── Backend re-consulta TMDB /watch/providers?country=ES
├── Frontend recibe MovieShow actualizado con nuevos proveedores
└── StreamingProviderBadges se re-renderiza
```

---

## 7. Testing

### Backend

- `TmdbClientTest` → mock del REST client, test de `getWatchProviders()` con response válido y con response vacío.
- `ProviderUrlMapperTest` → test de `buildDeepLink()` para providers mapeados y no mapeados, test de encoding del título.
- `MovieShowDtoMapperTest` → test de mapeo de `streamingProviders` domain ↔ response/request.
- `MovieShowEntityMapperTest` → test de mapeo entity ↔ domain con lista de providers.
- `MovieShowServiceTest` → si se añade método de consulta+persistencia de proveedores.

### Frontend

- `StreamingProviderBadges.test.tsx` → render con providers con y sin deepLinkUrl, verificar que los enlaces tienen href correcto cuando existe.
- `MovieShowCard.test.tsx` → verificar que los badges aparecen cuando existen proveedores y no aparecen cuando no.
- `MovieShowDetailPage.test.tsx` → verificar la sección de proveedores se renderiza.

---

## 8. Riesgos y consideraciones

### 8.1 Rate limit de TMDB

El endpoint `/watch/providers` cuenta hacia el rate limit de TMDB (~40 req/s). Si se consulta al crear cada película, no es problema (una petición extra por creación). Si se hace refresh masivo, hay que limitar concurrencia.

**Estrategia:** no hacer consulta automática al listar ni al ver — solo al crear/actualizar, o bien con botón explícito de "actualizar".

### 8.2 Datos incompletos

No todas las películas tienen datos de proveedores en TMDB. Es normal que un título devuelva `"results": {}` o country faltante.

**Comportamiento:** mostrar "No disponible en streaming" o simplemente no mostrar la sección si no hay proveedores.

### 8.3 Cambios en los nombres de las plataformas

- HBO Max → Max (2023, EE.UU.) / sigue siendo HBO Max en algunos países.
- Los logos de TMDB se actualizan cuando las plataformas cambian de branding.

El `provider_name` viene de TMDB, así que si TMDB lo actualiza, el frontend lo refleja automáticamente. Los deep links construidos con el mapa estático no dependen del nombre, solo del provider_id.

### 8.4 Deep links no garantizados

Las URLs de búsqueda construidas (ej: `netflix.com/search?q=Matrix`) son funcionales pero no son deep links oficiales ni garantizados por las plataformas. Podrían cambiar sin aviso.

**Mitigación:** si una plataforma cambia su esquema de búsqueda, se actualiza el `PROVIDER_URLS` en `ProviderUrlMapper`. El impacto es local y fácil de corregir.

### 8.5 Proveedores sin logo

Algunos provider_ids de TMDB no tienen `logo_path`. En ese caso:
- Se puede generar un placeholder con las iniciales del proveedor (ej: "NP" para Netflix si no hay logo).
- O solo mostrar el texto del `provider_name` sin logo.

### 8.6 Privacidad y visibilidad (Fase 9)

Si la colección es PÚBLICA, los proveedores de streaming son información que se muestra a otros usuarios. No hay problema de privacidad (son datos públicos de TMDB).

Si la colección es PRIVADA, el comportamiento es el mismo que el resto de campos: solo el dueño ve los proveedores, o se filtran según la lógica de visibilidad existente.

---

## 9. Relación con otras futuras mejoras

| Feature | Relación |
|---------|----------|
| **Progreso de lectura (fase 24)** | Independiente. Ambos son mejoras de colecciones existentes (libros vs pelis). |
| **Barcode scanner (fase 22)** | Independiente. Scan de ISBN para libros. |
| **Import de mazos (fase 23)** | Independiente. Magic decks. |
| **Estadísticas** | Los proveedores de streaming podrían dar estadísticas como "qué plataformas uso más" (Netflix vs HBO vs Disney+), pero eso es una extensión posterior. |

---

## 10. Resumen de cambios

### Backend

| Componente | Cambio | Prioridad |
|------------|--------|-----------|
| `TmdbClient.java` | Añadir `getWatchProviders()` + records de response | Alta |
| `ProviderUrlMapper.java` (nuevo) | Mapa provider_id → URL + buildDeepLink + buildLogoUrl | Alta |
| `MovieShow.java` | Añadir `streamingProviders` + `watchCountry` | Alta |
| `StreamingProvider.java` (nuevo) | Clase con providerId, providerName, logoUrl, type, deepLinkUrl | Alta |
| `ProviderAccessType.java` (nuevo enum) | FLATRATE, BUY, RENT | Alta |
| `MovieShowEntity.java` | Añadir campos + `StreamingProviderEntity` | Alta |
| `MovieShowEntityMapper.java` | Mapear providers | Alta |
| `MovieShowRequest.java` | Añadir campos opcionales | Media |
| `MovieShowResponse.java` | Añadir campos | Media |
| `MovieShowDtoMapper.java` | Mapear providers | Media |
| `MovieShowService.java` | Opcional: método consulta+persistencia proveedores | Media |
| `MovieShowController.java` | Opcional: endpoint refresh-providers | Baja |
| Tests | TmdbClient, ProviderUrlMapper, mappers, entity | Media |

### Frontend

| Componente | Cambio | Prioridad |
|------------|--------|-----------|
| `src/types/MovieShow.ts` | Añadir `StreamingProvider` + campos en `MovieShow` + `MovieShowFormData` | Alta |
| `StreamingProviderBadges.tsx` (nuevo) | Componente de logos + enlaces | Alta |
| `MovieShowCard.tsx` | Añadir badges de proveedores en listado | Media |
| `MovieShowDetailPage.tsx` | Añadir sección "Disponible en" | Media |
| `MovieShowForm.tsx` | No cambia (proveedores se auto-populan) | Baja |
| `movieshowsApi.ts` | No cambia (los datos vienen en la response existente) | Baja |

### Documentación

| Archivo | Cambio |
|---------|--------|
| `docs/03-base-de-datos/03.1-tablas.md` | Añadir campos `streamingProviders` y `watchCountry` a tabla MovieShows |
| `docs/05-api/README.md` | Actualizar `MovieShowRequest` y `MovieShowResponse` con nuevos campos |
| `docs/05-api/externas/externas-movies.md` | Añadir sección sobre Watch Providers endpoint |
| `docs/06-frontend/06.1-componentes.md` | Añadir `StreamingProviderBadges` |
| `docs/00-home.md` | Añadir entrada en índice |

---

## 11. Esfuerzo estimado

| Área | Tiempo |
|-------|--------|
| Backend: cliente TMDB + mapper URLs + domain/entity/DTO/mapper | 3-5 horas |
| Backend: tests | 1-2 horas |
| Frontend: tipos + componente badges + integración en card y detail | 2-4 horas |
| Frontend: tests | 1 hora |
| Documentación | 30-60 min |
| **Total** | **7-12 horas** |

**Dependencia crítica:** Ninguna. El feature es aislado en la colección de películas/series.
