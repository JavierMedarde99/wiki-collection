# APIs Externas — Cartas Magic: The Gathering

## Estado: Planificado (Fase 4)

---

## Opciones Investigadas

### 1. Scryfall API (Recomendada)

- **Base URL:** `https://api.scryfall.com`
- **Auth:** No requerida
- **Rate limit:** ~10 requests/segundo
- **Gratis:** Sí
- **Formato:** JSON
- **Documentación:** https://scryfall.com/docs/api

### 2. MTGJSON

- **Base URL:** Archivos JSON descargables
- **Auth:** No requerida
- **Gratis:** Sí
- **Formato:** JSON files

---

## Endpoints Planificados (Scryfall)

| Uso | Endpoint |
|-----|----------|
| Buscar carta | `GET /cards/search?q={query}` |
| Carta por ID | `GET /cards/{id}` |
| Carta por nombre | `GET /cards/named?fuzzy={name}` |
| Sets | `GET /sets` |
| Set por código | `GET /sets/{code}` |

---

## Ejemplo de Respuesta (Scryfall)

```json
{
  "object": "card",
  "id": "ae92e6c0-3b8d-4c5e-8b1a-2f3c4d5e6f7a",
  "name": "Lightning Bolt",
  "mana_cost": "{R}",
  "type_line": "Instant",
  "oracle_text": "Lightning Bolt deals 3 damage to any target.",
  "set": "lea",
  "set_name": "Limited Edition Alpha",
  "rarity": "common",
  "image_uris": {
    "normal": "https://cards.scryfall.io/normal/front/a/e/ae92e6c0...jpg"
  }
}
```

---

## Decisiones Pendientes

- [ ] Elegir Scryfall como API primaria
- [ ] Definir modelo de datos (MagicCard)
- [ ] Implementar cliente en backend
- [ ] Tests con mock server
