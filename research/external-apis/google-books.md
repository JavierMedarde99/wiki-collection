# Google Books API

**Investigación para:** Fase 1 — Colección de Libros

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://www.googleapis.com/books/v1/volumes` |
| **Auth** | API Key (opcional para desarrollo, recomendado para producción) |
| **Rate limit** | 100 requests/100 segundos (con API Key), 10/sin key |
| **Gratis** | Sí, con API Key gratuita desde Google Cloud Console |
| **Documentación** | https://developers.google.com/books/docs/overview |
| **Estado** | Activa y mantenida (2026) |

## Endpoints Útiles

| Uso | Endpoint |
|-----|----------|
| Buscar por título | `GET /volumes?q=intitle:{title}` |
| Buscar por autor | `GET /volumes?q=inauthor:{author}` |
| Buscar por ISBN | `GET /volumes?q=isbn:{isbn}` |
| Buscar general | `GET /volumes?q={query}` |
| Obtener por ID | `GET /volumes/{volumeId}` |

## Mapeo de Campos: Google Books → Book (Modelo Interno)

| Campo Google Books | Campo Interno (Book) | Notas |
|-------------------|----------------------|-------|
| `id` | `externalId` | ID único del volumen |
| `volumeInfo.title` | `title` | Título |
| `volumeInfo.authors[0]` | `author` | Primer autor (string singular) |
| `volumeInfo.description` | `descripcion` | Sinopsis |
| `volumeInfo.pageCount` | `pages` | Número de páginas |
| `volumeInfo.imageLinks.thumbnail` | `frontpage` | URL de portada |
| `volumeInfo.language` | `language` | Idioma (es, en, etc.) |
| `volumeInfo.publishedDate` | — | No se usa actualmente |
| `volumeInfo.publisher` | — | No se usa actualmente |

## Notas de Implementación

- La búsqueda se hace con `intitle:` para coincidir con el query param `name` del backend
- Los autores vuelven como lista; el backend normaliza a string singular (`authors[0]`)
- Campos como `publisher` y `publishedDate` se obtienen de la API pero no se persisten
- Open Library está disponible como fallback complementario

## Referencias

- [Google Books API Documentation](https://developers.google.com/books/docs/overview)
- [Open Library API](https://openlibrary.org/developers/api) (fallback complementario)
