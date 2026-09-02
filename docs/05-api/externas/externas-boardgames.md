# APIs Externas — Juegos de Mesa

## Estado: Planificado (Fase 3)

---

## Opciones Investigadas

### 1. BoardGameGeek API

- **Base URL:** `https://boardgamegeek.com/xmlapi2`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí
- **Formato:** XML (no JSON)
- **Documentación:** https://boardgamegeek.com/wiki/page/BGG_API

### 2. BGGL (BoardGameGeek JSON API)

- **API no oficial:** `https://bgg.cc/api/v1`
- **Auth:** No requerida
- **Gratis:** Sí
- **Formato:** JSON

### 3. LUDOTEKA API

- **Base URL:** Por definir
- **Auth:** Por determinar
- **Cobertura:** Principalmente español

---

## Endpoints Planificados (BGG)

| Uso | Endpoint |
|-----|----------|
| Buscar | `GET /search?query={query}&type=boardgame` |
| Obtener juego | `GET /thing?id={id}` |
| Colección usuario | `GET /collection/{username}` |

---

## Decisiones Pendientes

- [ ] Elegir API primaria (BGG vs alternativas)
- [ ] Definir estrategia de mapeo XML→JSON
- [ ] Implementar cliente en backend
- [ ] Tests con mock server
