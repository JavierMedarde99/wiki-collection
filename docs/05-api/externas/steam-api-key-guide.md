# Guía: Cómo Obtener una Steam API Key

## ¿Qué es la Steam API Key?

La Steam API Key es un token de autenticación gratuito que te permite acceder a la Steam Web API para obtener información sobre juegos, logros, estadísticas de jugadores, etc.

## ¿Dónde se Obtiene?

**URL:** https://steamcommunity.com/dev/apikey

## Paso a Paso

### 1. Iniciar Sesión en Steam

1. Abre tu navegador y ve a https://steamcommunity.com/login
2. Inicia sesión con tu cuenta de Steam
   - Si no tienes cuenta, crea una en https://store.steampowered.com/join/
   - **Nota:** Necesitas haber comprado al menos un juego en Steam para poder generar una API key (esto es una medida anti-spam de Valve)

### 2. Ir a la Página de API Key

1. Una vez logueado, ve a https://steamcommunity.com/dev/apikey
2. Verás un formulario con los campos:
   - **Domain:** Un nombre descriptivo para identificar tu aplicación (ej: "wiki-collection", "mi-app-juegos", etc.)
   - **Accept the terms:** Checkbox para aceptar los términos

### 3. Rellenar el Formulario

1. En **Domain**, escribe un nombre descriptivo (no tiene que ser un dominio real):
   - Ejemplos válidos: `wiki-collection`, `mi-proyecto-juegos`, `localhost`
   - Este campo es solo para tu referencia, no afecta al funcionamiento

2. Acepta los términos de uso marcando el checkbox

3. Haz clic en **"Register"**

### 4. Obtener tu Key

1. Después de registrar, verás una página con tu API Key:
   ```
   Key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
   ```
2. **Copia y guarda la key en un lugar seguro**
3. No compartas tu key públicamente (no la subas a GitHub, foros, etc.)

### 5. Configurar en tu Proyecto

En tu `application.properties` de Spring Boot:

```properties
# Steam API Key
steam.api.key=TU_KEY_AQUI
```

O en tu `.env`:

```env
STEAM_API_KEY=TU_KEY_AQUI
```

## ¿Qué Puedo Hacer con la Key?

| Endpoint | ¿Requiere Key? | Descripción |
|----------|----------------|-------------|
| `storesearch` | ❌ No | Buscar juegos por nombre |
| `GetGlobalAchievementPercentagesForApp` | ❌ No | Porcentajes globales de logros |
| `GetSchemaForGame` | ✅ Sí | Esquema completo de logros |
| `GetPlayerAchievements` | ✅ Sí | Logros de un jugador |
| `GetNumberOfCurrentPlayers` | ❌ No | Jugadores online actuales |

## Límites de Uso

| Tipo | Límite |
|------|--------|
| Sin key | ~200 requests/5 minutos |
| Con key | ~100,000 requests/día |
| Rate limit | ~1 request/segundo recomendado |

## Preguntas Frecuentes

### ¿Es gratuita?
Sí, completamente gratuita. Solo necesitas una cuenta de Steam con al menos un juego comprado.

### ¿Puedo tener múltiples keys?
Sí, puedes registrar múltiples keys para diferentes proyectos/dominios.

### ¿Qué hago si mi key se filtra?
1. Ve a https://steamcommunity.com/dev/apikey
2. Haz clic en **"Revoke"** junto a tu key
3. Registra una nueva key
4. Actualiza tu aplicación con la nueva key

### ¿La key expira?
No, la key no expira. Solo puede ser revocada por Valve si detectas uso abusivo.

### ¿Puedo usarla en producción?
Sí, pero ten en cuenta los límites de rate limit. Para aplicaciones con mucho tráfico, implementa caché.

## Solución de Problemas

### "Access is denied. Retrying will not help."
- Tu key es inválida o ha sido revocada
- Genera una nueva key en https://steamcommunity.com/dev/apikey

### "Method 'X' not found in interface 'Y'"
- Estás usando un endpoint incorrecto
- Verifica la documentación en https://developer.valvesoftware.com/wiki/Steam_Web_API

### 429 Too Many Requests
- Has excedido el rate limit
- Implementa backoff retry con exponential delay
- Espera unos segundos antes de reintentar

### No puedo registrarme (no me aparece el formulario)
- Necesitas tener al menos un juego comprado en Steam
- Asegúrate de estar logueado correctamente
- Prueba con otro navegador o limpia las cookies

## Enlaces Útiles

- **Registrar key:** https://steamcommunity.com/dev/apikey
- **Documentación oficial:** https://developer.valvesoftware.com/wiki/Steam_Web_API
- **Foro de desarrolladores:** https://dev.doit.wisc.edu/steam/
- **Lista de endpoints:** https://api.steampowered.com/ISteamWebAPIUtil/GetSupportedAPIList/v1/
