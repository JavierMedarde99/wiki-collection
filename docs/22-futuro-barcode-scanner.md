# Escáner de código de barras para libros (ISBN)

## Estado: Completado

## 1. Principio: el barcode de un libro es su ISBN-13

Los libros físicos usan códigos de barras **EAN-13**. El número que codifican
es el ISBN-13 completo (prefijo 978 o 979 + grupo + editorial + título + check digit).

```
ISBN 978-84-9838-267-1  →  código EAN-13 escaneado: 9788498382671
```

No hay conversión necesaria: el barcode **es** el ISBN-13. Solo hay que limpiar
guiones/espacios y usar los 13 dígitos directamente en Google Books.

ISBN-10 (viejo formato) no viene en barcode físico — solo ISBN-13 es relevante
para escaneo de códigos de barras.

---

## 2. Backend — Endpoint de búsqueda por ISBN

### 2.1 Estado actual

Google Books API soporta búsqueda por ISBN desde el principio:
```
GET /volumes?q=isbn:9788498382671
```

El `GoogleBooksClient` puede hacer esta petición (el método existe en la interfaz
`ExternalBookCatalogClient`), pero el endpoint REST de la app no lo expone.

Actualmente solo hay:
```
GET /api/v1/books/search?name={query}   → busca por título
```

### 2.2 Endpoint propuesto

```
GET /api/v1/books/search?isbn={isbn}
```

**Implementación mínima:**

BookController detecta si viene `isbn` o `name` y delega al servicio correcto:

```java
@GetMapping("/search")
public ResponseEntity<List<BookSearchResult>> search(
    @RequestParam(required = false) String name,
    @RequestParam(required = true) String isbn
) {
    if (isbn != null && !isbn.isBlank()) {
        return bookSearchService.searchByIsbn(isbn);
    }
    return bookSearchService.searchByName(name);
}
```

BookSearchService delega al client externo:
```java
public List<BookSearchResult> searchByIsbn(String isbn) {
    // Normalizar: quitar guiones, espacios
    String cleanIsbn = isbn.replaceAll("[\\s-]", "");
    return externalBookCatalogClient.searchByIsbn(cleanIsbn);
}
```

GoogleBooksClient hace la petición:
```java
public List<BookSearchResult> searchByIsbn(String isbn) {
    var response = restClient.get()
        .uri("/volumes?q=isbn:{isbn}", Map.of("isbn", isbn))
        .retrieve()
        .toResponseBody(GoogleBooksResponse.class);
    return mapToSearchResults(response.getItems());
}
```

### 2.3 Campos a añadir al modelo de Book

El changelog v1.5.0 eliminó el campo `isbn` del modelo. Para soportar el flujo
de escaneo con persistencia local, hay que volver a añadirlo:

- `Book.java` (domain) → campo `isbn: String?` (opcional)
- `BookEntity.java` → campo `isbn`
- `BookRequest.java` → campo `isbn` opcional
- `BookResponse.java` → campo `isbn`
- `BookDtoMapper.java` → mapear isbn

No requiere índice único — varios libros pueden tener el mismo ISBN si el usuario
tiene varias ediciones físicas.

### 2.4 Campos de BookSearchResult necesarios

El result de búsqueda debe incluir lo que necesita el frontend para pre-llenar
el formulario de creación:

- `id` (externalId de Google Books)
- `title`
- `authors` (lista → front toma el primero)
- `description`
- `coverImage` (thumbnail URL)
- `pageCount` → mapear a `pages`
- `isbn` → el ISBN-13 del libro

---

## 3. Frontend — Acceso a cámara desde React

### 3.1 Web API nativa: `navigator.mediaDevices.getUserMedia`

Disponible en todos los navegadores modernos (Chrome, Firefox, Safari, Edge).

Requiere:
- **HTTPS** en producción (Vercel ya lo tiene).
- **localhost** para desarrollo (sin HTTPS, funciona igual).
- **Permiso del usuario** (el navegador muestra dialogo nativo).

```typescript
const stream = await navigator.mediaDevices.getUserMedia({
  video: { facingMode: 'environment' }  // cámara trasera en móvil
});
videoRef.current.srcObject = stream;
```

No hay API nativa de "decodificar barcode" — el navegador solo da acceso al
stream de video. La decodificación corre por cuenta de una librería JS.

### 3.2 Librerías JS para decodificar códigos de barras

| Librería | NPM | Soporta EAN-13 | QR | Comentario |
|----------|-----|----------------|-----|------------|
| **html5-qrcode** | `html5-qrcode` | Sí (EAN-13, EAN-8, UPC-A, UPC-E…) | Sí | Wrapper auto-contenido. La más fácil de integrar. **Recomendada**. |
| **quagga2** | `quagga2` | Sí (solo barras lineales) | No | Más ligero pero requiere montar video target manualmente. |
| **@zxing/library** | `@zxing/library` | Sí | Sí | Port de ZXing a JS. Maduro pero más verboso de configurar. |

Recomendación: `html5-qrcode`. Es auto-contenido, tiene API simple, y soporta
EAN-13 directamente.

### 3.3 Flujo del componente BookBarcodeScanner

El componente es un modal auto-contenido con tres responsabilidades:
1. Iniciar cámara trasera al montar
2. Escanear frames hasta detectar un EAN-13
3. Detener cámara y notificar el ISBN detectado

```typescript
// Props del componente
interface BookBarcodeScannerProps {
  open: boolean;
  onClose: () => void;
  onBarcodeDetected: (isbn: string) => void;
}
```

Flujo:
1. Usuario clickea botón "📷 Escanear ISBN" en BookCreatePage
2. Se abre modal con vista de cámara
3. El usuario alinea el código de barras dentro del marco de escaneo
4. Al detectar EAN-13 → se dispara `onBarcodeDetected(isbn)`
5. El modal se cierra automáticamente y se detiene la cámara (cleanup)
6. El ISBN se rellena en el campo del formulario
7. Se dispara automáticamente `GET /api/v1/books/search?isbn={isbn}`
8. Se muestran resultados pre-llenados para el usuario

### 3.4 Consideraciones técnicas

- **iOS Safari**: puede fallar si el video no es visible en el momento del start.
  Asegurar que el modal esté montado y visible antes de inicializar el scanner.
- **Android Chrome**: funciona bien con `facingMode: "environment"`.
- **Detección múltiple**: el scanner dispara el callback en cada frame donde
  detecta un código. Por eso hay que detener el scanner inmediatamente tras el
  primer éxito (`scanner.stop()` en el callback).
- **qrbox**: para EAN-13 (formato horizontal), un rectángulo ancho y bajo funciona
  mejor que un cuadrado. Ej: `{ width: 300, height: 100 }`.
- **Permiso denegado**: manejar en el error handler con mensaje amigable
  ("Permiso de cámara denegado. Puedes introducir el ISBN manualmente").

---

## 4. Flujo completo de usuario

```
BookCreatePage
├── Nombre       [input texto]
├── Autor        [input texto]
├── ISBN         [input texto]     ← campo visible, editable manualmente
│                 [📷 Escanear ISBN]  ← botón cámara al lado
├── Tipo         [select MANGA/NOVEL/GRAPHIC_NOVEL]
├── Estado       [select TO_READ/READING/COMPLETED]
├── ... resto campos ...
└── [Crear libro]

Al click en "📷 Escanear ISBN":
  → Modal de cámara (BookBarcodeScanner)
  → User alinea barcode
  → Detectado EAN-13 → onBarcodeDetected("9788498382671")
  → Modal se cierra
  → ISBN rellenado en campo
  → Auto-búsqueda: GET /api/v1/books/search?isbn=9788498382671
  → Resultados mostrados en panel de selección
  → User clickea uno → formulario pre-llenado (título, autor, portada, isbn…)
```

El ISBN puede ser introducido manualmente o por cámara. Ambas vías van por el
mismo endpoint de búsqueda y el formulario queda igual.

---

## 5. Dependencias a añadir

### Frontend
```
npm install html5-qrcode
```

### Backend
Ninguna nueva — el cliente `GoogleBooksClient` ya puede buscár por ISBN.
Solo hay que añadir el parámetro `isbn` al endpoint y un método en `BookSearchService`.

---

## 6. Testing

- Backend: test de `BookSearchService.searchByIsbn()` mocking `ExternalBookCatalogClient`.
- Backend: test de `BookController` con parámetro `isbn` MockMvc.
- Frontend: test de `BookBarcodeScanner` mockeando la librería `html5-qrcode`
  (no es posible testear cámara real en CI).

---

## 7. Relación con otras futuras mejoras

Este feature es complementario al import de mazos por texto (docs/23-futuro-deck-import.txt.md).
Ambos siguen el mismo patrón: **input rápido desde una fuente externa → backend
resuelve e importa al modelo local**. Se pueden implementar en cualquier orden.
