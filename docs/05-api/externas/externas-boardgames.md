# APIs Externas — Juegos de Mesa

## Estado: Implementado (Fase 3)

---

## API Seleccionada

### BoardGameGeek XML API (ÚNICA)

- **Base URL:** `https://boardgamegeek.com/xmlapi2`
- **Auth:** No requerida
- **Rate limit:** Variable
- **Gratis:** Sí
- **Formato:** XML (parseado con Jackson XML)
- **Total juegos:** 100,000+
- **Documentación:** https://boardgamegeek.com/wiki/page/BGG_XML_API2

---

## Endpoints Disponibles

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /search?query={query}` |
| Obtener juego | `GET /thing/{id}` |
| Colección usuario | `GET /collection/{username}` |

---

## Parámetros de Búsqueda

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `query` | String | Término de búsqueda | `?query=catan` |

---

## Mapeo de Campos BGG → BoardGame

| Campo BGG | Campo BoardGame | Tipo | Notas |
|-----------|-----------------|------|-------|
| `id` | `bggId` | String | ID externo de BGG |
| `name` | `title` | String | Nombre del juego |
| `yearpublished` | `yearPublished` | Integer | Año de publicación |
| `minplayers` | `minPlayers` | Integer | Mínimo de jugadores |
| `maxplayers` | `maxPlayers` | Integer | Máximo de jugadores |
| `minplaytime` | `minPlaytime` | Integer | Duración mínima (min) |
| `maxplaytime` | `maxPlaytime` | Integer | Duración máxima (min) |
| `description` | `description` | String | Descripción completa |
| `thumbnail` | `thumbnailUrl` | String | URL de miniatura |
| `image` | `imageUrl` | String | URL de imagen completa |
| `publisher` | `publisher` | String | Editorial/publicador |
| `designers` | `designers` | List<String> | Lista de diseñadores |
| `categories` | `categories` | List<String> | Categorías del juego |
| `mechanics` | `mechanics` | List<String> | Mecánicas de juego |
| `rating` | `bggRating` | BigDecimal | Rating promedio BGG |

---

## Estrategia de Implementación

1. **BGG XML API como única fuente** — Búsqueda por nombre, 100k+ juegos, parseo XML con Jackson
2. **Mapeo a dominio** — Convertir XML a DTOs de BoardGame unificados
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 2s (BGG devuelve 202 Accepted mientras procesa)

### Flujo de Búsqueda

```
1. Cliente → GET /api/boardgames/search?name=catan
2. Backend → BGG XML API (search?query=catan)
3. Parsear XML → Mapear a DTO → Convertir a JSON → Devolver
```

### Flujo de Detalle

```
1. Cliente → GET /api/boardgames/{id}
2. Backend → MongoDB (ya persistido)
3. Mapear a BoardGame detallado → Convertir a JSON → Devolver
```

---

## Implementación en el Backend

### BggXmlClient (Spring Boot)

```java
@Slf4j
@Component("bggXmlClient")
public class BggXmlClient implements ExternalBoardGameCatalogClient {
    
    private final RestTemplate bggXmlRestTemplate;
    private final String baseUrl;
    private final int retryAttempts;
    private final long retryDelayMs;
    private final BoardGameXmlMapper mapper;
    
    public BggXmlClient(@Qualifier("bggXmlRestTemplate") RestTemplate bggXmlRestTemplate,
                        @Value("${bgg.api.xml-url:https://boardgamegeek.com/xmlapi2}") String baseUrl,
                        @Value("${bgg.api.retry-attempts:3}") int retryAttempts,
                        @Value("${bgg.api.retry-delay-ms:2000}") long retryDelayMs,
                        BoardGameXmlMapper mapper) {
        this.bggXmlRestTemplate = bggXmlRestTemplate;
        this.baseUrl = baseUrl;
        this.retryAttempts = retryAttempts;
        this.retryDelayMs = retryDelayMs;
        this.mapper = mapper;
    }
    
    @Override
    public List<BoardGameSearchResult> search(String query) {
        // Usa RestTemplate para llamadas HTTP
        // Usa Jackson XML para parsear respuestas
        // Implementa reintentos en caso de 202 Accepted
    }
}
```

---

## Referencias

- [BGG XML API 2](https://boardgamegeek.com/wiki/page/BGG_XML_API2)
- [pyBGG - Python BGG API](https://github.com/jaramir/pyBGG)
