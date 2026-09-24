# Futuro: Valoración de Juegos de Mesa (Board Game Rating)

**Fase:** 28 (no implementada)

## Descripción

Mejorar la valoración de juegos de mesa más allá del estado (OWNED/WISHLIST) y de los números de BGG (bggRating). Permitir al usuario:

### Ideas
- **Puntuación personal:** valorar un juego de mesa del 1 al 10 (o 1-5, dependiendo de la preferencia del usuario).
- **Número de jugadas:** registrar cuántas veces se ha jugado un juego.
- **Fecha de última jugada:** fecha de la última sesión de juego.
- **Jugadores preferidos:** para un juego, registrar con quién suele jugarse (opcional).
- **Dificultad percibida:** valorar la dificultad del juego (muy fácil, fácil, medio, difícil, muy difícil).

## Capacidades afectadas

- `board-games` — Nuevos campos en BoardGame (personalRating, playCount, lastPlayedDate, preferredPlayers, difficulty).

## Diseño de datos sugerido

```json
{
  "id": "...",
  "title": "Catan",
  "bggRating": 7.5,
  "personalRating": 8,
  "playCount": 12,
  "lastPlayedDate": "2026-09-01",
  "preferredPlayers": [
    { "id": "...", "username": "Ana" },
    { "id": "...", "username": "Carlos" }
  ],
  "difficulty": "MEDIUM"  // VERY_EASY, EASY, MEDIUM, HARD, VERY_HARD
}
```

## APIs necesarias

N/A — no requiere nuevas APs externas.

## Consideraciones de diseño

- El campo `difficulty` podría ser un enum con valores predefinidos.
- Si añadimos `preferredPlayers`, necesitamos una relación con la entidad User (pero esto podría no estar implementado en la fase 9 — ver colecciones personales).

## Prioridad: Baja

El estado actual de los juegos de mesa (OWNED/WISHLIST) es suficiente para una colección básica.

---

*Ver también: fase 3 (Juegos de Mesa actual).*
