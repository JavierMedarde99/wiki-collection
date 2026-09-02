# APIs Externas — Libros

## Google Books API

- **Base URL:** `https://www.googleapis.com/books/v1/volumes`
- **Auth:** API Key (opcional para búsquedas básicas, recomendado para producción)
- **Rate limit:** 100 requests/100 segundos (con API Key), 10/sin key
- **Gratis:** Sí, con API Key gratuita desde Google Cloud Console
- **Documentación:** https://developers.google.com/books/docs/overview

### Endpoints útiles

| Uso | Endpoint |
|-----|----------|
| Buscar por título | `GET /volumes?q=intitle:{title}` |
| Buscar por autor | `GET /volumes?q=inauthor:{author}` |
| Buscar por ISBN | `GET /volumes?q=isbn:{isbn}` |
| Buscar general | `GET /volumes?q={query}` |
| Obtener por ID | `GET /volumes/{volumeId}` |

### Ejemplo de Request

```http
GET https://www.googleapis.com/books/v1/volumes?q=intitle+inauthor:+intitle:harry+potter&maxResults=5
```

### Ejemplo de Respuesta

```json
{
  "kind": "books#volumes",
  "totalItems": 1234,
  "items": [
    {
      "kind": "books#volume",
      "id": "XmG1zgEACAAJ",
      "etag": "abc123",
      "volumeInfo": {
        "title": "Harry Potter y la piedra filosofal",
        "authors": ["J.K. Rowling"],
        "publisher": "Salamandra",
        "publishedDate": "1999",
        "description": "La historia de un joven mago...",
        "industryIdentifiers": [
          {"type": "ISBN_13", "identifier": "9788498382671"},
          {"type": "ISBN_10", "identifier": "8498382670"}
        ],
        "pageCount": 256,
        "categories": ["Fantasy"],
        "imageLinks": {
          "thumbnail": "https://books.google.com/books/content?id=XmG1zgEACAAJ&printsec=frontcover&img=1&zoom=1"
        },
        "language": "es"
      }
    }
  ]
}
```

### Campos de respuesta relevantes

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `totalItems` | Integer | Total de resultados |
| `items[].id` | String | ID único del volumen |
| `items[].volumeInfo.title` | String | Título |
| `items[].volumeInfo.authors[]` | List | Lista de autores |
| `items[].volumeInfo.publisher` | String | Editorial |
| `items[].volumeInfo.publishedDate` | String | Fecha publicación |
| `items[].volumeInfo.description` | String | Sinopsis |
| `items[].volumeInfo.industryIdentifiers[]` | List | ISBN (type: ISBN_13 o ISBN_10) |
| `items[].volumeInfo.pageCount` | Integer | Número de páginas |
| `items[].volumeInfo.categories[]` | List | Géneros |
| `items[].volumeInfo.imageLinks.thumbnail` | String | URL portada |
| `items[].volumeInfo.language` | String | Idioma (es, en, etc.) |

### Mapeo a Modelo Interno

| Google Books | Interno (Book) |
|--------------|----------------|
| `id` | `externalId` |
| `volumeInfo.title` | `title` |
| `volumeInfo.authors[0]` | `author` |
| `volumeInfo.description` | `description` |
| `volumeInfo.pageCount` | `pages` |
| `volumeInfo.imageLinks.thumbnail` | `frontpage` |
| `volumeInfo.language` | `language` |

---

## Open Library API

- **Base URL:** `https://openlibrary.org`
- **Auth:** No requerida
- **Rate limit:** ~100 requests/minuto
- **Gratis:** Sí, completamente
- **Documentación:** https://openlibrary.org/developers/api

### Endpoints útiles

| Uso | Endpoint |
|-----|----------|
| Buscar | `GET /search.json?q={query}` |
| Buscar por título | `GET /search.json?title={title}` |
| Buscar por autor | `GET /search.json?author={author}` |
| Buscar por ISBN | `GET /isbn/{isbn}.json` |
| Obtener obra | `GET /works/{workId}.json` |
| Obtener autor | `GET /authors/{authorId}.json` |

### Ejemplo de Request

```http
GET https://openlibrary.org/search.json?q=harry+potter&limit=5
```

### Ejemplo de Respuesta

```json
{
  "numFound": 5678,
  "start": 0,
  "docs": [
    {
      "key": "/works/OL82563W",
      "title": "Harry Potter and the Philosopher's Stone",
      "author_name": ["J. K. Rowling"],
      "isbn": ["9780747532699", "0747532699"],
      "first_publish_year": 1997,
      "cover_i": 123456,
      "publisher": ["Bloomsbury"],
      "language": ["eng"]
    }
  ]
}
```

### Campos de respuesta relevantes

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `numFound` | Integer | Total de resultados |
| `docs[].key` | String | Path de la obra (para requests posteriores) |
| `docs[].title` | String | Título |
| `docs[].author_name[]` | List | Autores |
| `docs[].isbn[]` | List | ISBNs disponibles |
| `docs[].first_publish_year` | Integer | Año publicación |
| `docs[].cover_i` | Integer | ID de portada (URL: `https://covers.openlibrary.org/b/id/{cover_i}.jpg`) |
| `docs[].publisher[]` | List | Editoriales |
| `docs[].language[]` | List | Idiomas (código ISO 639-2) |

### Obtener Portadas

```
https://covers.openlibrary.org/b/id/{cover_i}-S.jpg   (Small)
https://covers.openlibrary.org/b/id/{cover_i}-M.jpg   (Medium)
https://covers.openlibrary.org/b/id/{cover_i}-L.jpg   (Large)
```

### Mapeo a Modelo Interno

| Open Library | Interno (Book) |
|--------------|----------------|
| `docs[].key` | `externalId` (prefijo `OL:`) |
| `docs[].title` | `title` |
| `docs[].author_name[0]` | `author` |
| `docs[].first_publish_year` | `publishedDate` |
| `docs[].cover_i` | `frontpage` (URL construida) |
| `docs[].publisher[0]` | `publisher` |

---

## Comparativa

| Característica | Google Books | Open Library |
|----------------|--------------|--------------|
| Auth | Opcional (key) | No |
| Portadas | ✅ Alta calidad | ✅ Calidad media |
| Sinopsis | ✅ Detalladas | ⚠️ Básicas |
| ISBN lookup | ✅ | ✅ |
| Rate limit generoso | ✅ (con key) | ⚠️ |
| Datos en español | ✅ | ⚠️ |
| API Key | Recomendado | No necesaria |
| Total items | Millones | Millones |

---

## Estrategia de Implementación

1. **Google Books como primaria** — Mayor calidad de datos
2. **Open Library como fallback** — Si Google Books no tiene resultados
3. **Combinación** — Usar Google Books para búsqueda y Open Library para portadas si es necesario

### Flujo de Búsqueda

```
1. Cliente → GET /api/books/search?name=harry+potter
2. Backend → Google Books API (search)
3. Si hay resultados → Mapear y devolver
4. Si no hay resultados → Open Library API (search)
5. Mapear y devolver
```

---

## Implementación en el Backend

### GoogleBooksClient (Spring Boot)

```java
@Component
public class GoogleBooksClient implements ExternalBookCatalogClient {
    
    private final RestClient restClient;
    
    public GoogleBooksClient(RestClient.Builder builder) {
        this.restClient = builder
            .baseUrl("https://www.googleapis.com/books/v1")
            .build();
    }
    
    @Override
    public List<BookSearchResult> search(String query) {
        try {
            var response = restClient.get()
                .uri(uriBuilder -> uriBuilder
                    .path("/volumes")
                    .queryParam("q", "intitle:" + query)
                    .queryParam("maxResults", 10)
                    .build())
                .retrieve()
                .toResponseBody();
            
            // Parsear JSON y mapear a BookSearchResult
            // ...
            
        } catch (Exception e) {
            // Graceful degradation
            return List.of();
        }
    }
}
```

---

## Errores Comunes

### Google Books

| Error | Causa | Solución |
|-------|-------|----------|
| 403 | API key inválida o excedido | Verificar key o crear nueva |
| 429 | Rate limit exponencial | Backoff retry |
| Vacío sin resultados | Query muy específica | Simplificar búsqueda |

### Open Library

| Error | Causa | Solución |
|-------|-------|----------|
| 429 | Rate limit (~100/min) | Reducir frecuencia |
| Cover no existe | `cover_i` es null | Usar placeholder |
| Datos inconsistentes | Fuentes múltiples | Validar antes de mapear |
