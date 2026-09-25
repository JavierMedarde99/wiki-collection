# Futuro: Plataformas de Streaming (Streaming Platforms)

**Fase:** 25
**Estado:** ✅ Implementado (parcial: TMDB streaming providers + badges) · 📋 Pendiente (plataformas suscritas, filtrado, notificaciones)

## Lo que ya está implementado

### Streaming providers via TMDB — ✅ Completo

**Backend — `MovieShow.java`:**
- `List<StreamingProvider> streamingProviders` — lista de plataformas donde está disponible
- `String watchCountry` — país de visualización (para filtrar disponibilidad por región)

**Backend — `StreamingProvider.java`:**
```java
Integer providerId;
String providerName;
String logoUrl;
ProviderAccessType type; // FLATRATE, BUY, RENT
```

**Backend — Endpoints:**
- `POST /api/v1/movieshows/{id}/refresh-providers` — re-consulta los proveedores de streaming en TMDB y actualiza el MovieShow
- `MovieShowController.java`: `refreshProviders()` → `movieShowUseCase.refreshStreamingProviders(id, userId)`

**Backend — `WatchProvidersClient.java`** (ExternalMovieCatalogClient):
- Consome TMDB para obtener los streaming providers de una película/serie
- Usa el endpoint de TMDB de watch providers (disponibilidad por país)

**Backend — DTOs:**
- `MovieShowRequest.java` incluye `List<StreamingProviderRequest> streamingProviders` y `String watchCountry`
- `MovieShowResponse.java` incluye `streamingProviders` y `watchCountry`
- `StreamingProviderRequest.java` / `StreamingProviderResponse.java`

**Frontend — Componentes:**
- `components/StreamingProviderBadges.tsx`: muestra los logos de plataformas agrupados por tipo de acceso (Suscripción/Compra/Alquiler). Compatible con logos (URL) o texto plano si no hay logo.
- `types/MovieShow.ts`: interfaz `StreamingProvider` con `providerId`, `providerName`, `logoUrl`, `type`, `deepLinkUrl`

**Tests:** `StreamingProviderBadges.test.tsx`, `MovieShowControllerTest.java`

### Lo que queda por implementar

#### Marcar plataformas suscritas por usuario — 📋 Pendiente

El usuario indica qué plataformas tiene suscritas (Netflix, HBO, Disney+, Amazon Prime, etc.). Requiere:
- Nueva entidad `UserStreamingPlatform` (userId → lista de providerIds suscritos)
- O campo en `UserPreferences`: `List<Integer> subscribedProviderIds`
- UI en PreferencesPage para marcar/desmarcar plataformas

#### Filtrado de movieshows por plataformas suscritas — 📋 Pendiente

Una vez que el usuario tiene plataformas suscritas, filtrar la lista de movieshows para mostrar solo aquellas disponibles en al menos una de esas plataformas. Requiere:
- Endpoint adicional o parámetro de filtro en `GET /api/v1/movieshows?platforms=Netflix,HBO`
- Lógica en backend que compute la intersección entre streamingProviders del item y las suscritas del usuario

#### Link a plataforma — 📋 Pendiente

El `StreamingProvider` idealmente debería incluir `deepLinkUrl` (ya definido en el tipo frontend como `deepLinkUrl?: string` pero no implementado en backend). Cada plataforma tendría su URL directa al título.

#### Notificaciones de novedades — 📋 Pendiente

Cuando un movie show en `PLAN_TO_WATCH` llega a una plataforma donde el usuario está suscrito, notificar. Requiere:
- Mecanismo de polling (backend revisa периодически) o webhook
- Sistema de notificaciones (push, email, o in-app)
- Esta es la feature más compleja de las pendientes

## Estado actual de investigación

El documento original asumía que las plataformas de streaming no estaban implementadas y proponía usar **JustWatch API** como fuente. 

En realidad:
- **La fuente usada es TMDB** (no JustWatch). TMDB sí expone datos de streaming providers.
- Los `streamingProviders` (con logos, tipo de acceso) **ya están implementados** en backend y frontend.
- `watchCountry` ya está en `MovieShow`.

Quedan pendientes las features de usuario: suscripción propia, filtrado, deep links, notificaciones.

## Consideraciones de diseño

- `StreamingProvider.type` usa `ProviderAccessType` (FLATRATE/BUY/RENT) — enum del backend. Está documentado en el dominio pero no en el spec de movie-shows (falta añadir).
- `deepLinkUrl` está en el tipo frontend (`MovieShow.ts`) pero no en el dominio backend (`StreamingProvider.java`). Habría que añadirlo al backend si se quiere funcionalidad de deep link.
- Para notificaciones: sería preferible un sistema de "alertas" within-app en vez de push/email real (más simple, sin dependencias externas adicionales).

## Prioridad

- **Marcar plataformas suscritas:** Media (mejora UX directa)
- **Filtrado por plataformas:** Media (consecuencia natural de lo anterior)
- **Deep links:** Baja (nice to have)
- **Notificaciones:** Baja (complejidad alta, beneficio dudoso para una app personal)

---

*Ver también: fase 5 (Películas/Series), fase 18 (phase 5 docs).*