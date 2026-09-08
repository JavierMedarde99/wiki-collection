# APIs Externas — Juegos de Mesa

## Estado: Planificado (Fase 3)

---

## API Seleccionada

### BoardGameGeek JSON API

- **Base URL:** `https://bgg.cc/api/v1`
- **Auth:** No requerida (API Key opcional)
- **Rate limit:** Variable
- **Gratis:** Sí
- **Formato:** JSON
- **Total juegos:** 100,000+
- **Documentación:** https://bgg.github.io/
- **Estado:** No oficial, devuelve JSON nativo

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

## Ejemplos de Respuesta

### Búsqueda (`GET /search?query=catan`)

```json
{
  "items": [
    {
      "id": "13",
      "name": "Catan",
      "yearpublished": "1995",
      "minplayers": "3",
      "maxplayers": "4",
      "minplaytime": "60",
      "maxplaytime": "120",
      "thumbnail": "https://...",
      "image": "https://...",
      "description": "Descripción del juego...",
      "publisher": "KOSMOS",
      "designers": ["Klaus Teuber"],
      "categories": ["Economic", "Negotiation"],
      "mechanics": ["Dice Rolling", "Modular Board", "Trading"],
      "rating": "7.2"
    }
  ],
  "total": 1234
}
```

### Detalle (`GET /thing/13`)

```json
{
  "id": "13",
  "name": "Catan",
  "yearpublished": "1995",
  "minplayers": "3",
  "maxplayers": "4",
  "minplaytime": "60",
  "maxplaytime": "120",
  "thumbnail": "https://...",
  "image": "https://...",
  "description": "Descripción completa del juego...",
  "publisher": "KOSMOS",
  "designers": ["Klaus Teuber"],
  "categories": ["Economic", "Negotiation"],
  "mechanics": ["Dice Rolling", "Modular Board", "Trading"],
  "rating": "7.2",
  "usersrated": "50000",
  "average": "7.2",
  "bayesaverage": "7.1",
  "stddev": "1.2",
  "weight": "2.3",
  "owned": "80000",
  "wanting": "500",
  "wishing": "2000",
  "numcomments": "10000",
  "numweights": "500",
  "averageweight": "2.3"
}
```

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
| `rating` | `bggRating` | Double | Rating promedio BGG |

---

## Estrategia de Implementación

1. **BGG JSON como primaria** — Búsqueda por nombre, 100k+ juegos, JSON nativo
2. **Mapeo a dominio** — Convertir JSON a DTOs de BoardGame
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Cache** — Caché de resultados (TTL 1 hora) para reducir llamadas

### Flujo de Búsqueda

```
1. Cliente → GET /api/boardgames/search?name=catan
2. Backend → BGG JSON API (search?query=catan)
3. Mapear a DTO → Convertir a JSON → Devolver
4. Si no hay resultados → 404 Not Found
```

### Flujo de Detalle

```
1. Cliente → GET /api/boardgames/{id}
2. Backend → BGG JSON API (thing/{id})
3. Mapear a BoardGame detallado → Convertir a JSON → Devolver
```

---

## Endpoints Planificados (Backend)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/boardgames/search?name={query}` | Buscar juegos de mesa |
| GET | `/api/boardgames/{id}` | Obtener detalle de un juego |
| POST | `/api/boardgames` | Crear juego en colección local |
| GET | `/api/boardgames` | Listar colección local |
| PUT | `/api/boardgames/{id}` | Actualizar juego en colección |
| DELETE | `/api/boardgames/{id}` | Eliminar juego de colección |

---

## Decisiones Pendientes

- [ ] Definir estrategia de cache (Redis vs caché en memoria)
- [ ] Implementar cliente BGG JSON en backend
- [ ] Tests con mock server

---

## Referencias

- [BGG JSON API (no oficial)](https://bgg.github.io/)
- [pyBGG - Python BGG API](https://github.com/jaramir/pyBGG)
