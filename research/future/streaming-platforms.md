# Futuro: Plataformas de Streaming (Streaming Platforms)

**Fase:** 25 (no implementada)

## Descripción

Mejorar la integración con plataformas de streaming para películas y series, permitiendo al usuario:

### Ideas
- **Disponibilidad en streaming:** para una película/series, mostrar en qué plataformas está disponible (Netflix, HBO, Disney+, Amazon Prime, etc.).
- **Marcar plataformas suscritas:** el usuario indica qué plataformas tiene suscritas, y la app filtra las películas/series disponibles en esas plataformas.
- **Link a plataforma:** en el detalle de una película/series, mostrar el enlace directo a la plataforma donde está disponible.
- **Notificaciones de novedades:** cuando una película/series que el usuario tiene en PLAN_TO_WATCH llega a una plataforma donde está suscrito, notificar al usuario.

## Capacidades afectadas

- `movieshows` — Nuevos campos en MovieShow (plataformas disponibles), y posible nueva entidad `StreamingPlatform`.

## APIs necesarias

- **TMDb API:** actualmente solo usamos búsqueda y imágenes. Para disponibilidad de streaming, necesitaríamos una API que proporcione esta información (ej: JustWatch API, o Reelgood). TMDB no proporciona datos de disponibilidad en streaming de forma oficial.
- **JustWatch API:** permite buscar en qué servicios está disponible una película/series. Tiene un tier gratuito con límite de peticiones.

## Diseño de datos sugerido

```json
{
  "id": "...",
  "title": "...",
  "mediaType": "MOVIE",
  "status": "PLAN_TO_WATCH",
  "platforms": [
    {
      "name": "Netflix",
      "available": true,
      "link": "https://www.netflix.com/title/..."
    },
    {
      "name": "Amazon Prime",
      "available": false
    }
  ]
}
```

## Consideraciones de diseño

- La disponibilidad de streaming varía por región. La API de JustWatch permite filtrar por país. Necesitaríamos guardar la región del usuario o detectarla automáticamente.
- Las plataformas y sus enlaces cambian con frecuencia. Esto requiere actualización periódica de los datos, o búsqueda en tiempo real cuando el usuario visita el detalle.
- La integración con JustWatch añade una nueva dependencia externa. Necesitaríamos una clave API.

## Prioridad: Baja

La aplicación actual permite llevar un registro personal de películas/series vistas. La disponibilidad en streaming es una feature "nice to have" que añade complejidad sin mejorar el núcleo de la aplicación.

---

*Ver también: fase 17 (Películas/Series actual).*
