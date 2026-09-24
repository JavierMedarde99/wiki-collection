# Futuro: Seguimiento de Lectura (Reading Progress)

**Fase:** 24 (no implementada)

## Descripción

Mejorar el seguimiento de lectura de libros más allá del simple estado (TO_READ, READING, COMPLETED). Posibles features:

### Ideas
- **Porcentaje de lectura:** el usuario indica cuántas páginas ha leído de un libro (ej: "150 de 320 páginas"). Esto permite tener una barra de progreso.
- **Fechas detalladas:** registrar fecha de inicio de lectura, fecha de finalización, y fechas de cada sesión de lectura (opcional).
- **Historial de lectura:** ver todos los libros leídos en un período determinado (ej: "Libros leídos en 2026"), con fechas.
- **Objetivos de lectura:** el usuario se propone leer X libros en un año, y la app muestra el progreso hacia esa meta.
- **Series y colecciones:** agrupar libros por serie (ej: "Harry Potter" → 7 libros) y ver el progreso de la serie completa.

## Capacidades afectadas

- `books` — Nuevos campos en Book (readingProgress, series, etc.)
- Posible nueva entidad `ReadingSession` si implementamos tracking de sesiones.

## Diseño de datos sugerido

```json
{
  "id": "...",
  "title": "...",
  "author": "...",
  "pages": 320,
  "state": "READING",
  "readingProgress": {
    "pagesRead": 150,
    "percentage": 46.875,
    "startDate": "2026-01-15",
    "estimatedEndDate": "2026-02-15"
  },
  "series": {
    "name": "Harry Potter",
    "position": 3,
    "totalInSeries": 7
  },
  "sessions": [
    {
      "date": "2026-01-20",
      "pagesRead": 30,
      "durationMinutes": 45
    }
  ]
}
```

## APIs necesarias

N/A — no requiere nuevas APIs externas, solo cambios en el modelo de Book.

## Consideraciones de diseño

- Si añadimos `ReadingSession`, necesitamos un repositorio y un CRUD endpoint.
- El cálculo del porcentaje y las fechas estimadas puede ser sobrecogedor si el usuario no quiere entrar en tanto detalle. Podríamos tener un modo simple (solo estado) y un modo avanzado (con progreso).

## Prioridad: Baja

La funcionalidad actual de libros (TO_READ, READING, COMPLETED) cubre el caso de uso básico.

---

*Ver también: fase 29 (Lista de deseos + fecha de adquisición para libros).*
