# Futuro: Scanner de Código de Barras (Barcode Scanner)

**Fase:** 22 (no implementada)
**Estado:** 📋 Por hacer / investigación

## Descripción

Permitir al usuario añadir items a su colección escaneando códigos de barras (ISBN para libros, códigos de barras de juegos de mesa, códigos de producto de videojuegos) usando la cámara del dispositivo móvil.

### Casos de uso principales
- Escanear ISBN-13 de un libro → buscar en Google Books API y añadir a la colección
- Escanear código de barras de un juego de mesa → buscar en BoardGameGeek y añadir
- Escanear código de producto de un videojuego → buscar en RAWG y añadir
- Reconocer códigos de barras en imágenes (fotografías de códigos)

## Capacidades afectadas

- `books` — Añadir libros mediante ISBN escaneado
- `board-games` — Añadir juegos de mesa mediante código de barras
- `games` — Añadir videojuegos mediante código de producto

## APIs necesarias

- **Google Books API** — ya integrada (búsqueda por ISBN funciona)
- **BoardGameGeek XML API** — ya integrada (búsqueda por ID de BGG; necesitaríamos mapear código de barras → BGG ID)
- **RAWG API** — ya integrada (búsqueda por nombre; el código de barras podría no estar disponible)

## Consideraciones de diseño

- **Frontend:** necesitamos acceso a la cámara del dispositivo. En el navegador, esto se hace con `navigator.mediaDevices.getUserMedia()` o con una librería como `html5-qrcode` o `jsQR`.
- **Timeout de escaneo:** el scanner debe dar feedback visual (cámara activa, esquinas de escaneo, flash si está disponible).
- **Múltiples formatos:** el scanner debe soportar al menos EAN-13, ISBN-13 (que es EAN-13 con prefijo 978/979), UPC-A.
- **Códigos de barras de juegos de mesa:** BGG no tiene un endpoint oficial para buscar por código de barras. Una opción es usar un servicio externo que mapee códigos de barras a BGG IDs, o permitir al usuario introducir manualmente el BGG ID después del escaneo.
- **Códigos de barras de videojuegos:** los códigos de barras de videojuegos (ej: códigos UPC en la carátula) no tienen un mapeo universal a RAWG IDs. En muchos casos, el escaneo podría fallar y el usuario tendría que buscar manualmente.

## Alternativas / decisiones técnicas

- ¿Usar `html5-qrcode` (soporta códigos de barras y QR) o `jsQR` (solo QR)?
- ¿Integración nativa (React Native) o web (PWA)?
- ¿Permitir escanear códigos de barras desde fotos guardadas, o solo en tiempo real?

## Prioridad: Media

Dependiendo del interés del usuario y del tiempo disponible.
