# Futuro: Lista de Deseos y Fecha de Adquisición para Libros (Books Wishlist + Acquisition Date)

**Fase:** 29
**Estado:** 📋 Por hacer

## Descripción

Mejorar el seguimiento de la colección de libros añadiendo:

### Ideas
- **Lista de deseos separada:** libros que el usuario quiere leer pero aún no tiene (WISHLIST). Esto sería un estado adicional a TO_READ, READING, COMPLETED. O bien, una lista separada.
- **Fecha de adquisición:** para los libros que el usuario ya tiene, registrar la fecha de compra o recepción (ej: regalo de Navidad, compra en librería, descarga digital).
- **Precio de adquisición:** opcional, el precio que pagó por el libro (para hacer estadísticas de gasto en libros).
- **Editorial y año de edición:** datos adicionales que el usuario puede querer registrar (ej: "Editorial Planeta, 2015").
- **ISBN-10:** además del ISBN-13, permitir registrar ISBN-10 (algunos libros antiguos solo tienen ISBN-10).

## Capacidades afectadas

- `books` — Nuevos campos: wishlist (boolean o estado adicional), acquisitionDate, acquisitionPrice, publisher, publicationYear, isbn10.

## Diseño de datos sugerido

```json
{
  "id": "...",
  "title": "1984",
  "author": "George Orwell",
  "isbn13": "9780451524935",
  "isbn10": "0451524934",
  "state": "COMPLETED",
  "wishlist": false,
  "acquisitionDate": "2023-05-15",
  "acquisitionPrice": 14.99,
  "publisher": "Editorial Planeta",
  "publicationYear": 1949,
  "pages": 328
}
```

## APIs necesarias

N/A — no requiere nuevas APIs externas. Google Books API devuelve algunos de estos datos (editorial, año de publicación, ISBN-10), por lo que podrían auto-llenarse al añadir un libro desde Google Books.

## Estado actual del código

`Book.java` tiene:
- `String isbn` — campo ISBN (actualmente solo ISBN-13, sin distinción isbn10/isbn13)
- `String comment` — notas personales

No tiene: `wishlist` (ni estado WISHLIST en BookState enum), `acquisitionDate`, `acquisitionPrice`, `publisher`, `publicationYear`, `isbn10`.

`BookState` enum actual: `TO_READ`, `READING`, `COMPLETED`. Sin `WISHLIST`.

`BookSearchResult` (resultado de búsqueda Google Books) sí tiene `publisher` y `publishedDate` como campos mapeables desde la API, pero no se persisten en `Book`.

## Consideraciones de diseño

- ¿La lista de deseos es un estado más (TO_READ, READING, COMPLETED, WISHLIST, OWNED) o una lista separada? La fase 1 actual usa TO_READ/READING/COMPLETED. Añadir WISHLIST como estado sería más coherente con la API de Google Books (que tiene "wantToRead" en algunos contextos).
- ¿El precio de adquisición es un campo opcional o requerido? Probablemente opcional.
- ¿Los campos de editorial y año de publicación se auto-llenan desde Google Books o los edita el usuario?
- ISBN-10: el campo `isbn` actualmente es un String genérico. Se podría normalizar a ISBN-13 internamente y permitir introducir ISBN-10 que se convierta automáticamente.

## Prioridad: Media

Permite mejorar la organización de la colección de libros si el usuario quiere mantener un registro más detallado.

---

*Ver también: fase 1 (Libros actual) y fase 24 (Seguimiento de lectura).*