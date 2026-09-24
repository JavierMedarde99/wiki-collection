# Steam Web API — Logros de Videojuegos

**Investigación para:** Fase 2 — Colección de Videojuegos (Steam Achievements)

## API General

| Propiedad | Valor |
|-----------|-------|
| **Base URL** | `https://api.steampowered.com/ISteamUserStats/` |
| **Auth** | API Key (gratuita en https://steamcommunity.com/dev/apikey) |
| **Rate limit** | ~200 requests/5 minutos (sin key), ~100,000/día (con key) |
| **Gratis** | Sí, con registro gratuito |
| **Documentación** | https://developer.valvesoftware.com/wiki/Steam_Web_API |
| **Cobertura** | Todos los juegos de Steam con logros |
| **Estado** | Activa (2026) |

## Endpoints de Logros

| Uso | Endpoint | Auth |
|-----|----------|------|
| Buscar juego por nombre (obtener AppID) | `GET https://store.steampowered.com/api/storesearch/?term={nombre}&l={lang}&cc={country}` | No |
| Porcentajes globales de logros | `GET /GetGlobalAchievementPercentagesForApp/v0002/?gameid={appid}` | No |
| Esquema de logros (nombres, descripciones, iconos) | `GET /GetSchemaForGame/v2/?key={key}&appid={appid}` | Sí |
| Logros de un jugador específico | `GET /GetPlayerAchievements/v1/?key={key}&steamid={steamid}&appid={appid}` | Sí |

## Mapeo: Steam → AchievementsSummary (Modelo Interno)

| Campo Steam | Campo Interno | Notas |
|-------------|---------------|-------|
| `name` (achievement) | `name` | Nombre del logro |
| `description` | `description` | Descripción del logro |
| `achieved` | `achieved` | Boolean: logrado o no |
| `iconclosed` / `iconlarge` | `iconUrl` | URL del icono |

## Estrategia de Implementación

1. **Buscar AppID** usando Store Search (sin API key necesaria)
2. **Obtener esquema** con API key para nombres y descripciones de logros
3. **Obtener logros del jugador** con API key y SteamID
4. **Combinar** esquema + logros del jugador → AchievementsSummary

## Límites de Uso

| Tipo | Límite |
|------|--------|
| Sin API key | ~200 requests/5 minutos |
| Con API key | ~100,000 requests/día |
| Rate recomendado | ~1 request/segundo |

## Requisitos para API Key

- Necesitas tener al menos un juego comprado en Steam
- Registro en https://steamcommunity.com/dev/apikey
- La key no expira (solo puede ser revocada por Valve)

## Referencias

- [Steam Web API Documentation](https://developer.valvesoftware.com/wiki/Steam_Web_API)
- [Steam API Key Registration](https://steamcommunity.com/dev/apikey)
