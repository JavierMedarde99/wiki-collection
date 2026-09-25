# Futuro: Scanner de Código de Barras (Barcode Scanner)

**Fase:** 22
**Estado:** ✅ Implementado (libros) · 📋 Pendiente (board games, videojuegos)

## Lo que ya está implementado

### Libros (ISBN — ✅ Completo)

**Backend:**
- `Book.java` tiene campo `isbn: String`
- `BookRequest.java` / `BookResponse.java` incluyen `isbn`
- `BookController.java`: `GET /api/v1/books/search?isbn={isbn}` → busca en Google Books por ISBN
- `BookSearchUseCase.java`: `searchByIsbn(String isbn)` implementado
- `GoogleBooksClient.java`: soporta búsqueda por ISBN (`/volumes?q=isbn:{isbn}`)

**Frontend:**
- `components/BookBarcodeScanner.tsx`: modal que usa `html5-qrcode` para escanear EAN-13/ISBN con la cámara. Normaliza el código detectado y devuelve el ISBN.
- `components/BookIsbnScan.tsx`: pestaña en `BookCreatePage` que permite:
  - Escanear ISBN con la cámara (o introducirlo manualmente)
  - Buscar en Google Books por ese ISBN
  - Seleccionar resultado e ir al formulario de creación con datos pre-llenados
  - Capturar startDate, endDate, start (rating), comment, pages durante la creación
- `api/booksApi.ts`: `searchBooksByIsbn(isbn, page, size)` → llama a `GET /api/v1/books/search?isbn=...`
- `types/Book.ts`: interfaz `Book` incluye `isbn?: string`

**Tests:** `BookBarcodeScanner.test.tsx`, `BookIsbnScan.test.tsx`

### Lo que queda por implementar

#### Board games (BGG barcode → BGG ID) — 📋 Pendiente

Los juegos de mesa no tienen código de barras estandarizado como los libros (ISBN). BoardGameGeek asigna un ID numérico a cada juego, pero no hay un código de barras físico estandarizado que se pueda escanear directamente.

**Enfoques posibles:**
1. **EAN-13 en envases:** algunos juegos publicados comercialmente tienen código EAN-13 en el envase. Sería necesario un mapeo EAN → BGG ID (ej: vía una API externa o tabla manual).
2. **Barcode escanea → busca por nombre:** si no hay mapeo, el usuario escanea (o introduce) y el sistema busca el juego por nombre en BGG.
3. **Introducir BGG ID manualmente:** en el formulario de creación, añadir un campo para BGG ID (numérico) que redirija a la búsqueda en BGG.

#### Videojuegos (UPC → RAWG) — 📋 Pendiente

Los videojuegos tienen códigos UPC-A/EAN-13 en sus carátulas físicas. RAWG no expone un endpoint de búsqueda por UPC. Se podría:

1. Usar el UPC para buscar en Google Shopping / otras fuentes y cruzar con RAWG por nombre.
2. Directamente buscar en RAWG por nombre tras el escaneo (como hace la app actualmente, pero sin tener que tipear el nombre).

## Estado actual de investigación

El documento original asumía que el barcode scanner no estaba implementado. La fase 22 para libros está **completa** — el código existe y funciona. Solo quedan las extensiones a board games y videojuegos.

## Consideraciones de diseño

- `html5-qrcode` ya está integrado y funcionando para ISBN. Reusar la misma librería para otros códigos de barras.
- El componente `BookBarcodeScanner` está desacoplado (recibe `onBarcodeDetected: (isbn: string) => void`), por lo que puede reutilizarse para otros flujos con mínimos cambios.
- Para board games: no existe un estándar de código de barras → BGG ID. Habría que documentar qué enfoque se usa.

## Prioridad

- **Libros:** — (ya implementado)
- **Board games:** Media (depende de si hay juegos con códigos EAN rastreables)
- **Videojuegos:** Baja (RAWG no soporta UPC nativamente; requeriría un paso intermedio)

---

*Ver también: fase 1 (Libros), fase 3 (Juegos de mesa), fase 2 (Videojuegos).*