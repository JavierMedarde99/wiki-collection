# APIs Externas — Películas y Series

## Estado: Planificado (Fase 5)

---

## Opciones Investigadas

### 1. The Movie Database (TMDB) - Recomendada

- **Base URL:** `https://api.themoviedb.org/3`
- **Auth:** API Key (gratuita con registro)
- **Rate limit:** ~40 requests/segundo
- **Gratis:** Sí, con registro
- **Formato:** JSON
- **Documentación:** https://developer.themoviedb.org/docs

### 2. OMDb API

- **Base URL:** `https://www.omdbapi.com`
- **Auth:** API Key (gratuita con registro)
- **Rate limit:** 1,000/día (gratis)
- **Gratis:** Sí, limitado
- **Formato:** JSON

### 3. TVMaze API

- **Base URL:** `https://api.tvmaze.com`
- **Auth:** No requerida
- **Rate limit:** No especificado
- **Gratis:** Sí
- **Formato:** JSON

---

## Endpoints Planificados (TMDB)

| Uso | Endpoint |
|-----|----------|
| Buscar película | `GET /search/movie?query={query}` |
| Buscar serie | `GET /search/tv?query={query}` |
| Detalle película | `GET /movie/{id}` |
| Detalle serie | `GET /tv/{id}` |
| Créditos | `GET /movie/{id}/credits` |
| Imágenes | `GET /movie/{id}/images` |

---

## Ejemplo de Respuesta (TMDB)

```json
{
  "page": 1,
  "results": [
    {
      "id": 278,
      "title": "Cadena perpetua",
      "overview": "Andy Dufresne es encarcelado por matar a su esposa...",
      "poster_path": "/yNzwxmV0y1fv63F4qS4sN2yEvxu.jpg",
      "release_date": "1994-09-23",
      "vote_average": 8.7
    }
  ],
  "total_results": 1234,
  "total_pages": 62
}
```

---

## Decisiones Pendientes

- [ ] Elegir TMDB como API primaria
- [ ] Definir modelo de datos (MovieShow)
- [ ] Implementar cliente en backend
- [ ] Tests con mock server
