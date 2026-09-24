# BoardGameGeek XML API 2

**Investigación para:** Fase 3 — Colección de Juegos de Mesa

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://boardgamegeek.com/xmlapi2` |
| **Auth** | No requerida |
| **Rate limit** | Variable (~10 requests/segundo recomendado) |
| **Gratis** | Sí |
| **Formato** | XML (parseado con Jackson XML) |
| **Total juegos** | 100,000+ |
| **Documentación** | https://boardgamegeek.com/wiki/page/BGG_XML_API2 |
| **Estado** | Activa y mantenida por BGG (2026) |

## Endpoints Útiles

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /search?query={query}&type=boardgame` |
| Obtener detalle | `GET /thing?id={id}&stats=1` |
| Colección usuario | `GET /collection/{username}?own=1` |

## Mapeo: BGG XML → BoardGame (Modelo Interno)

| Campo BGG | Campo Interno (BoardGame) | Notas |
|-----------|--------------------------|-------|
| `id` | `bggId` | ID externo de BGG |
| `name` | `title` | Nombre del juego |
| `yearpublished` | `yearPublished` | Año de publicación |
| `minplayers` | `minPlayers` | Mínimo de jugadores |
| `maxplayers` | `maxPlayers` | Máximo de jugadores |
| `minplaytime` | `minPlaytime` | Duración mínima (minutos) |
| `maxplaytime` | `maxPlaytime` | Duración máxima (minutos) |
| `description` | `description` | Descripción completa |
| `thumbnail` | `thumbnailUrl` | URL de miniatura |
| `image` | `imageUrl` | URL de imagen completa |
| `publisher` | `publisher` | Editorial/publicador |
| `designers` | `designers` | Lista de diseñadores |
| `categories` | `categories` | Categorías del juego |
| `mechanics` | `mechanics` | Mecánicas de juego |
| `rating` | `bggRating` | Rating promedio BGG (0-10) |

## Estrategia de Implementación

1. **BGG XML API como única fuente** — Búsqueda por nombre, 100k+ juegos, parseo XML con Jackson
2. **Mapeo a dominio** — Convertir XML a DTOs de BoardGame unificados
3. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
4. **Reintentos automáticos** — 3 intentos con delay de 2s (BGG devuelve 202 Accepted mientras procesa)

## Limitaciones

- ❌ Formato XML (requiere parseo con Jackson XML)
- ❌ Rate limit estricto
- ❌ API asíncrona en algunos endpoints (devuelve 202 Accepted mientras procesa)

## Referencias

- [BGG XML API2](https://boardgamegeek.com/wiki/page/BGG_API2)
- [BGG API Terms of Use](https://boardgamegeek.com/wiki/page/XML_API_Terms_Of_Use)
- [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml)
