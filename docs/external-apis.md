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

### Ejemplo de respuesta (item)

```json
{
  "volumeInfo": {
    "title": "Cien años de soledad",
    "authors": ["Gabriel García Márquez"],
    "publisher": "Editorial Sudamericana",
    "publishedDate": "1967",
    "description": "La novela que...",
    "industryIdentifiers": [
      { "type": "ISBN_13", "identifier": "9780307474728" }
    ],
    "pageCount": 417,
    "categories": ["Fiction"],
    "imageLinks": {
      "thumbnail": "http://books.google.com/books/content?id=..."
    },
    "language": "es"
  }
}
```

## Open Library API

- **Base URL:** `https://openlibrary.org`
- **Auth:** No requerida
- **Rate limit:** ~100 requests/minuto (sin auth)
- **Gratis:** Sí, completamente

### Endpoints útiles

| Uso | Endpoint |
|-----|----------|
| Buscar | `GET /search.json?q={query}` |
| Buscar por ISBN | `GET /isbn/{isbn}.json` |
| Obtener obra | `GET /works/{workId}.json` |

### Ejemplo de respuesta (search)

```json
{
  "numFound": 123,
  "docs": [
    {
      "title": "Cien años de soledad",
      "author_name": ["Gabriel García Márquez"],
      "isbn": ["9780307474728"],
      "first_publish_year": 1967,
      "cover_i": 123456,
      "publisher": ["Editorial Sudamericana"]
    }
  ]
}
```

## Comparativa

| Característica | Google Books | Open Library |
|----------------|--------------|--------------|
| Portadas | ✅ Alta calidad | ✅ Calidad media |
| Sinopsis | ✅ Detalladas | ⚠️ Básicas |
| ISBN lookup | ✅ | ✅ |
| Rate limit generoso | ✅ (con key) | ⚠️ |
| Datos en español | ✅ | ⚠️ |

**Decisión:** Usar **Google Books API** como primaria (mejor calidad de datos, portadas, sinopsis). **Open Library** como fallback si Google Books no encuentra resultados.
