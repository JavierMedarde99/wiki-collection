---
type: external-api
title: "Steam API Key — Guía de obtención y configuración"
description: "Guía para obtener y configurar la STEAM_API_KEY"
tags: [api-externa, steam]
---

# Steam API Key — Guía de obtención y configuración

**Investigación para:** Fase 2 — Colección de Videojuegos (logros de Steam)

**Verificado en vivo:** 2026-10-15 (parcial: sin sesión de Steam; el formulario de alta no es accesible sin login)

**Ficha relacionada:** `steam.md` (endpoints, mapeo y limitaciones de la Steam Web API)

## ¿Qué es la Steam API Key?

Es un token gratuito que da acceso a los métodos de la Steam Web API que no son públicos, como el esquema de logros de un juego (`GetSchemaForGame`) o el progreso de un jugador (`GetPlayerAchievements`). Estos métodos **no funcionan sin clave**: verificado en vivo, sin `key` responden `400` con el cuerpo `Required parameter 'key' is missing`.

Valve distingue dos tipos de clave:

| Tipo | Para quién | Acceso |
|------|-----------|--------|
| **User key** | Cualquier cuenta de Steam | Métodos públicos + los que exigen autenticación de usuario (logros, estadísticas de un jugador) |
| **Publisher key** | Cuentas de Steamworks (estudios) | Todo lo anterior + métodos exclusivos de publisher; se crea desde Steamworks, no desde la página pública |

Este proyecto solo necesita una **user key**.

## ¿Dónde se obtiene?

**URL:** https://steamcommunity.com/dev/apikey

Comprobado en vivo: sin sesión iniciada, esa URL **redirige al login** (`https://steamcommunity.com/login/home/?goto=%2Fdev%2Fapikey`, HTTP `200`). El formulario de alta no se ve ni se puede verificar sin una cuenta.

## Requisitos

Según la [documentación oficial de autenticación](https://partner.steamgames.com/doc/webapi_overview/auth), para una user key basta con:

1. Una cuenta de Steam.
2. Un nombre de dominio que asociar a la clave (puede ser descriptivo, no hace falta que sea real).
3. Aceptar los [Steam Web API Terms of Use](https://steamcommunity.com/dev/apiterms).

**Matiz importante y no verificado:** la creencia extendida de que "hay que haber comprado al menos un juego en Steam" **no aparece en la documentación oficial**. Lo que sí aplica en la práctica es que la cuenta no sea *limitada* (cuentas que nunca han gastado dinero en Steam no ven el formulario). No se pudo comprobar en este entorno porque hace falta iniciar sesión; trátalo como "probable, sin confirmar".

## Paso a paso

### 1. Iniciar sesión

Accede a https://steamcommunity.com/login y entra con tu cuenta de Steam. Si tienes Steam Guard activo, tendrás que introducir el código.

### 2. Ir a la página de registro

Navega a https://steamcommunity.com/dev/apikey. Si no has iniciado sesión, te redirigirá al login.

### 3. Rellenar el formulario

- **Domain:** un nombre descriptivo para identificar tu aplicación. Ejemplos: `wiki-collection`, `mi-proyecto-juegos`, `localhost`. No afecta al funcionamiento de la clave.
- Aceptar los términos de uso.
- Pulsar **Register**.

### 4. Copiar la clave

Tras registrar, la página muestra la clave. Cópiala y guárdala en un gestor de contraseñas. **No la subas al repositorio**, ni la pegues en foros o capturas.

En esa misma página puedes revocar la clave si se filtra.

## Configurar en el proyecto

El nombre correcto de la propiedad es **`steam.api-key`** (con guion medio, no `steam.api.key`). Está en `src/main/resources/application.properties`:

```
# Steam API
steam.api-key=${STEAM_API_KEY:}
```

La forma recomendada es **exportar la variable de entorno** `STEAM_API_KEY` y no escribir la clave en ningún fichero versionado. En desarrollo, un `.env` local o la configuración de tu IDE sirven; en producción, el gestor de secretos del despliegue.

```
STEAM_API_KEY=TU_KEY_AQUI
```

Sin la variable, el default es cadena vacía: las llamadas de logros fallarán con `400` y el backend devolverá una lista de logros vacía sin error visible. La búsqueda de AppID (`storesearch`) sí sigue funcionando porque no requiere clave.

## ¿Qué se puede hacer con la clave?

| Endpoint | ¿Requiere clave? | ¿Lo usa el backend? | Descripción |
|----------|------------------|---------------------|-------------|
| `GET /ISteamUserStats/GetSchemaForGame/v2` | ✅ Sí | ✅ Sí | Esquema completo de logros de un juego |
| `GET /ISteamUserStats/GetPlayerAchievements/v1` | ✅ Sí | ✅ Sí | Logros desbloqueados por un jugador |
| `GET https://store.steampowered.com/api/storesearch` | ❌ No | ✅ Sí | Buscar AppID por nombre (endpoint de tienda, no de la Web API) |
| `GET /ISteamUserStats/GetGlobalAchievementPercentagesForApp/v0002` | ❌ No | ❌ No | Porcentajes globales de logros (verificado: `200` sin clave) |
| `GET /ISteamUserStats/GetNumberOfCurrentPlayers/v1` | ❌ No | ❌ No | Jugadores online actuales |

## Límites de uso

| Concepto | Valor |
|----------|-------|
| Tope oficial | **100.000 llamadas/día** por clave (Steam Web API Terms of Use, §2) |
| Respuesta al superarlo | `429 Too Many Requests` |
| Límite por minuto | No documentado oficialmente |

La ficha antigua de este wiki hablaba de "~200 requests/5 minutos sin clave" y "~1 request/segundo": **ninguna de las dos cifras está respaldada por Valve**. El único límite publicado es el diario.

No hay nivel de servicio garantizado: Valve puede cambiar, suspender o limitar la API o tu clave sin aviso (Terms of Use, §11).

La clave es **única para toda la aplicación**, así que el consumo de todos los usuarios suma contra el mismo cupo de 100.000/día. Si el uso crece, la mitigación real es caché (que este proyecto **no** tiene para logros) y no repartir claves.

## Preguntas frecuentes

### ¿Es gratuita?

Sí. Solo necesitas cuenta de Steam y aceptar los términos.

### ¿Puedo tener varias claves?

Sí, puedes registrar varias, cada una con su dominio descriptivo. Para este proyecto basta una.

### ¿La clave expira?

La documentación no indica caducidad por tiempo. Puede revocarse: tú mismo desde la página de registro, o Valve si detecta abuso.

### ¿Qué hago si se filtra?

1. Entra en https://steamcommunity.com/dev/apikey con tu sesión.
2. Revoca la clave comprometida.
3. Registra una nueva.
4. Actualiza `STEAM_API_KEY` en el entorno y reinicia el servicio.

Recuerda que el proyecto envía la clave como **query param** (`key=...`), no como cabecera, así que puede aparecer en logs de proxy y trazas. Valve admite también la cabecera `x-webapi-key` como alternativa más discreta.

### ¿Puedo usarla en producción?

Sí, respetando el tope diario. Para un proyecto personal como Wiki-Collection, 100.000 llamadas/día es holgado **siempre que se añada caché**; hoy cada carga de la pantalla de logros son 1–2 llamadas a Steam sin cachear.

## Solución de problemas

### `400 Bad Request` — `Required parameter 'key' is missing`

Verificado en vivo: es lo que responde la API cuando no envías `key`. En el proyecto significa que `steam.api-key` está vacío o que `STEAM_API_KEY` no llegó al proceso. Revisa la variable de entorno y reinicia.

### `401` / `403` — `Access is denied. Retrying will not help. Please verify your key= parameter.`

Verificado en vivo con una clave inválida (`403`, cuerpo HTML). La clave no es válida o ha sido revocada. Reintentar con la misma clave no sirve: genera una nueva. Si tienes restricción por IP, revisa también la allow-list de la clave.

### `429 Too Many Requests`

Superaste el cupo de 100.000 llamadas/día. Espera a que se renueve o reduce el consumo con caché. **Este cliente no reintenta** (`SteamAchievementsClient` captura el error y devuelve lista vacía), así que un `429` se ve como "sin logros", no como error.

### `Method 'X' not found in interface 'Y'`

El endpoint o la versión no existen. Comprueba la ruta contra https://partner.steamgames.com/doc/webapi/ISteamUserStats.

### No aparece el formulario de registro

- Puede que no hayas iniciado sesión: la URL redirige al login.
- La cuenta podría estar *limitada* (sin ninguna compra). Este extremo no está documentado oficialmente y no se pudo verificar aquí.

## Referencias

- **Registrar clave:** https://steamcommunity.com/dev/apikey
- **Términos de uso (límite diario):** https://steamcommunity.com/dev/apiterms
- **Autenticación con Web API Keys:** https://partner.steamgames.com/doc/webapi_overview/auth
- **Visión general de la Web API:** https://partner.steamgames.com/doc/webapi_overview
- **Interfaz ISteamUserStats:** https://partner.steamgames.com/doc/webapi/ISteamUserStats
- **Lista de interfaces soportadas:** https://api.steampowered.com/ISteamWebAPIUtil/GetSupportedAPIList/v1/
- Ficha de la API en este wiki: `research/external-apis/steam.md`
