# APIs Externas — Juegos de Mesa

## Estado: Planificado (Fase 3)

---

## APIs Investigadas

### BoardGameGeek JSON API (PRINCIPAL)

- **Base URL:** `https://bgg.cc/api/v1`
- **Auth:** API Key/None
- **Rate limit:** Variable
- **Gratis:** Sí
- **Formato:** JSON
- **Total juegos:** 100,000+ (mismo catálogo que XML)
- **Documentación:** https://bgg.github.io/
- **Estado:** No oficial, pero devuelve JSON nativo

**Endpoints conocidos:**

| Uso | Endpoint |
|-----|----------|
| Buscar | `GET /search?query={query}` |
| Obtener juego | `GET /thing/{id}` |
| Colección | `GET /collection/{username}` |

**Ventajas:**
- ✅ Formato JSON nativo (sin necesidad de parsear XML)
- ✅ Más fácil de integrar con Spring Boot
- ✅ Mismo catálogo de 100k+ juegos
- ✅ Datos completos (publisher, diseñadores, categorías, mecánicas, ratings, imágenes)

**Desventajas:**
- ❌ API no oficial (puede desaparecer)
- ❌ Acceso inestable (Cloudflare protection)
- ❌ No garantiza disponibilidad a largo plazo

---

### BoardGameGeek XML API 2 (SECUNDARIA)

- **Base URL:** `https://boardgamegeek.com/xmlapi2`
- **Auth:** Cookies de sesión (login en BGG)
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí, con registro gratuito
- **Formato:** XML (requiere parseo)
- **Total juegos:** 100,000+
- **Documentación:** https://boardgamegeek.com/wiki/page/BGG_API2

**Endpoints útiles:**

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /search?query={query}&type=boardgame` |
| Obtener detalle | `GET /thing?id={id}&stats=1` |
| Colección usuario | `GET /collection/{username}?own=1` |

**Ventajas:**
- ✅ Catálogo más grande del mundo (100k+ juegos)
- ✅ API oficial mantenida por BGG
- ✅ Datos muy completos
- ✅ Comunidad activa y actualizada

**Desventajas:**
- ❌ Formato XML (requiere parseo con Jackson XML o JAXB)
- ❌ Requiere cookies de sesión para autenticación
- ❌ Rate limit estricto (10 req/segundo)
- ❌ API asíncrona en algunos endpoints (devuelve 202 Accepted mientras procesa)

---

### 3. Board Game Atlas (INACTIVA/NO DISPONIBLE)

- **Base URL:** `https://api.boardgameatlas.com/api`
- **Estado:** Dominio no resoluble (2026-09)
- **Nota:** Parece haber cerrado o migrado

---

### 4. Ludopedia API (BRASIL)

- **Base URL:** `https://ludopedia.com.br/api/v1`
- **Auth:** Token de usuario (obligatorio)
- **Cobertura:** Principalmente juegos en portugués/br Market
- **Formato:** JSON
- **Estado:** Requiere token de autenticación

**Endpoints:**

| Uso | Endpoint |
|-----|----------|
| Buscar juegos | `GET /api/v1/games` |
| Detalle | `GET /api/v1/games/{id}` |

**Ventajas:**
- ✅ Formato JSON
- ✅ Enfoque en mercado brasileño

**Desventajas:**
- ❌ Requiere token de autenticación
- ❌ Catálogo limitado a mercado brasileño
- ❌ Documentación limitada
- ❌ No es útil para español/europa

---

### 5. Spielbox API (NO DISPONIBLE)

- **Base URL:** `https://api.spielbox.com`
- **Estado:** Sin endpoints de juegos públicos
- **Nota:** Solo tiene WordPress REST API genérica sin custom post types de juegos

---

### 6. TheGamecrafter API (NO RECOMENDADA)

- **Base URL:** `https://www.thegamecrafter.com/api/v1`
- **Propósito:** Crear/comprar juegos de mesa custom
- **No es útil** para búsqueda de juegos de mesa existentes

---

## Comparativa de APIs

| Característica | BGG XML2 | BGG JSON (no oficial) | Ludopedia |
|----------------|----------|----------------------|-----------|
| Formato | XML | JSON | JSON |
| Auth | Cookies | API Key/None | Token |
| Catálogo | 100k+ | 100k+ | Limitado |
| Búsqueda por nombre | ✅ | ✅ | ✅ |
| Datos completos | ✅ | ✅ | ⚠️ Parcial |
| Estabilidad | ✅ | ⚠️ Inestable | ✅ |
| Mantenimiento | Activo | No oficial | Activo |
| Idioma principal | Inglés | Inglés | Portugués |

---

## Decisión Recomendada

**API Primaria:** BoardGameGeek JSON API

**Justificación:**
1. Devuelve JSON nativo (sin necesidad de parsear XML)
2. Más fácil de integrar con Spring Boot y RestTemplate
3. Catálogo de 100,000+ juegos con datos muy ricos (mecánicas, categorías, diseñadores, publishers, ratings)
4. La autenticación por cookies es manejable con Spring RestTemplate + cookies

**API Secundaria:** BoardGameGeek XML API 2

**Justificación:**
1. API oficial mantenida por BGG
2. Útil como fallback si la API JSON tiene problemas de Cloudflare
3. Requiere parseo XML con Jackson XML o JAXB

---

## Estrategia de Implementación

1. **BGG JSON como primaria** — Búsqueda por nombre, 100k+ juegos, JSON nativo
2. **BGG XML2 como secundaria** — Fallback si JSON no disponible, requiere parseo XML
3. **Mapeo a dominio** — Convertir JSON/XML a DTOs de BoardGame unificados
4. **Respuesta JSON al cliente** — El backend siempre devuelve JSON
5. **Cache** — Caché de resultados (TTL 1 hora) para reducir llamadas

### Flujo de Búsqueda

```
1. Cliente → GET /api/boardgames/search?name=catan
2. Backend → BGG JSON API (search)
3. Si hay resultados → Mapear a DTO → Convertir a JSON → Devolver
4. Si no hay resultados → BGG XML API (search)
5. Parsear XML → Mapear a DTO → Convertir a JSON → Devolver
6. Si no hay resultados → 404 Not Found
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

- [ ] Confirmar estrategia de autenticación BGG (cookies vs API key si disponible)
- [ ] Elegir librería de parseo XML (Jackson XML vs JAXB)
- [ ] Definir estrategia de cache (Redis vs caché en memoria)
- [ ] Implementar cliente BGG XML en backend
- [ ] Implementar cliente BGG JSON en backend (fallback)
- [ ] Tests con mock server

---

## Referencias

- [BGG XML API2 Documentation](https://boardgamegeek.com/wiki/page/BGG_API2)
- [BGG API Terms of Use](https://boardgamegeek.com/wiki/page/XML_API_Terms_Of_Use)
- [BGG JSON API (no oficial)](https://bgg.github.io/)
- [pyBGG - Python BGG API](https://github.com/jaramir/pyBGG)
- [Ludopedia API](https://ludopedia.com.br/api/v1)
