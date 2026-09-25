# Futuro: Importación de Mazos Existentes (Deck Import)

**Fase:** 23
**Estado:** 📋 Por hacer

## Descripción

Permitir importar mazos Commander existentes desde fuentes externas, para que el usuario no tenga que añadir cartas una a una.

### Fuentes posibles
- **Scryfall:** soporta listas de cartas en formato JSON. Un mazo existente podría importarse desde una URL que apunte a un JSON con las cartas.
- **Moxfield / Deckbox / other deck builders:** algunos servicios permiten exportar mazos en formato compatible. Necesitaríamos mapear los nombres de cartas a Scryfall IDs.
- **Archivo local:** el usuario carga un archivo JSON/CSV con las cartas del mazo.

## Capacidades afectadas

- `decks` — Creación de mazos desde datos externos
- `magic-cards` — Posible integración con buscador de cartas por nombre

## APIs necesarias

- **Scryfall API** — ya integrada (búsqueda de cartas por nombre; `search?q=name:{nombre}`)
- Posible integración con **Moxfield API** o **Deckbox API** si existen y son públicas.

## Estado actual del código

No existe implementación actual. El backend tiene:
- `DeckUseCase.addCard(String deckId, String scryfallId, int quantity, String ownerId)` — añade carta por scryfallId
- `DeckController.POST /{id}/cards` — endpoint para añadir carta a mazo existente
- `DeckCard` como subdocumento embebido en `Deck`

Pero no hay un endpoint ni flujo de "importar mazo completo desde una lista de nombres de cartas".

## Consideraciones de diseño

- **Importación por nombre de carta:** dado que Scryfall soporta búsqueda por nombre con fuzzy search, la importación por lista de nombres es factible. Pero hay cartas con nombres duplicados o similares (ej: "Lightning Bolt" vs "Shock"). El importador debe permitir al usuario confirmar o elegir entre resultados ambiguos.
- **Identidad de color del comandante:** el importador debe inferir los colores del comandante a partir de los datos importados.
- **Validación de reglas:** después de la importación, el mazo debe pasar por la misma validación que un mazo creado manualmente (ver `DeckValidator`).
- **Ritmo de importación:** si el mazo tiene 200 cartas, y cada carta requiere una búsqueda en Scryfall, el importador podría tardar varios segundos. Podríamos usar batch search si Scryfall lo permite (Scryfall soporta búsquedas con `q=name:{nombre1} OR name:{nombre2} OR ...` pero tiene límites de longitud de query).

## Prioridad: Media

Dependiendo del interés del usuario en usar otros servicios de construcción de mazos.

---

*Ver también: fase 4.1 (Mazos Commander).*