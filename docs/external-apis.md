# APIs Externas

## Google Books API

- **Base URL:** `https://www.googleapis.com/books/v1/volumes`
- **Auth:** API Key (opcional para búsquedas básicas, recomendado para producción)
- **Rate limit:** 100 requests/100 segundos (con API Key), 10/sin key
- **Gratis:** Sí, con API Key gratuita desde Google Cloud Console

### Endpoints útiles

| Uso | Endpoint |
|-----|----------|
| Buscar por título | `GET /volumes?q=intitle:{title}` |
| Buscar por autor | `GET /volumes?q=inauthor:{author}` |
| Buscar por ISBN | `GET /volumes?q=isbn:{isbn}` |
| Obtener por ID | `GET /volumes/{volumeId}` |

### Campos de respuesta relevantes

- `volumeInfo.title` — Título
- `volumeInfo.authors[]` — Lista de autores
- `volumeInfo.publisher` — Editorial
- `volumeInfo.publishedDate` — Fecha publicación
- `volumeInfo.description` — Sinopsis
- `volumeInfo.industryIdentifiers[]` — ISBN (type: ISBN_13 o ISBN_10)
- `volumeInfo.pageCount` — Páginas
- `volumeInfo.categories[]` — Géneros
- `volumeInfo.imageLinks.thumbnail` — URL portada
- `volumeInfo.language` — Idioma

## Open Library API

- **Base URL:** `https://openlibrary.org`
- **Auth:** No requerida
- **Rate limit:** ~100 requests/minuto
- **Gratis:** Sí, completamente

### Endpoints útiles

| Uso | Endpoint |
|-----|----------|
| Buscar | `GET /search.json?q={query}` |
| Buscar por ISBN | `GET /isbn/{isbn}.json` |
| Obtener obra | `GET /works/{workId}.json` |

### Campos de respuesta relevantes (search)

- `numFound` — Total de resultados
- `docs[].title` — Título
- `docs[].author_name[]` — Autores
- `docs[].isbn[]` — ISBNs disponibles
- `docs[].first_publish_year` — Año publicación
- `docs[].cover_i` — ID de portada (URL: `https://covers.openlibrary.org/b/id/{cover_i}.jpg`)
- `docs[].publisher[]` — Editoriales

## Comparativa

| Característica | Google Books | Open Library |
|----------------|--------------|--------------|
| Portadas | ✅ Alta calidad | ✅ Calidad media |
| Sinopsis | ✅ Detalladas | ⚠️ Básicas |
| ISBN lookup | ✅ | ✅ |
| Rate limit generoso | ✅ (con key) | ⚠️ |
| Datos en español | ✅ | ⚠️ |

**Decisión:** Usar **Google Books API** como primaria. **Open Library** como fallback.
