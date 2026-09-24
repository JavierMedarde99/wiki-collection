# Reglas de Negocio — Wiki-Collection

ADRs = decisiones técnicas/architecturales. Este archivo = reglas de negocio que han sido dictadas por el usuario / negocio.

## Reglas globales

- Los precios de catálogo, de bienes raíces, y similares, siempre se muestran sin formato (es decir, 15000.00, no 15.000,00). Esto es consistente con el formato de entrada del usuario (el usuario introduce precios sin formato).

---

## Reglas por capability

### Libros (Books)
- Un libro puede estar en la colección del usuario (TO_READ, READING, COMPLETED) y también en la lista de deseos (WISHLIST). La lista de deseos es.Visibility de la colección del usuario debería ser "wishlist", Exposure solo para el usuario. No se muestran en la colección pública.
- El sistema no comparte información de libros con terceros (la visibilidad de la colección es siempre controlada por el usuario).

### Videojuegos (Games)
- Un juego puede estar en la colección del usuario (PLAYING, COMPLETED, WISHLIST, ABANDONED). La lista de deseos es Visibility "wishlist", Exposure solo para el usuario.
- La clasificación de Steam por nivel de logros es una métrica de rendimiento que el usuario puede añadir manualmente. No se calcula automáticamente ni se usa para ninguna funcionalidad más allá de la visualización.

### Juegos de Mesa (Board Games)
- El classificador BGG es la fuente de información de clasificación de juegos de mesa (el usuario puede ver la clasificación de BGG para un juego y decidir si lo añade a su colección). La clasificación personal del usuario es un campo opcional que no afecta a la funcionalidad del sistema.
- Los juegos de mesa no tienen metadatos típicos como "plataformas" o "números de jugadores"; sólo datos básicos como nombre, descripción, year de publicación, min/max jugadores, min/max tiempo, publisher, designers, category, mechanics, imageUrl, thumbnailUrl, bggId, bggRating, category, notes, dateAdded.

### Cartas Magic (Magic Cards)
- En la sección de Magic nivel usuario, las cartas tienen sobresalientes: cantidad y condición (MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR).
- La tarjeta Magic que se añade al mazo Commander se consume del catálogo de cartas del usuario (si el usuario tiene la carta en su colección personal de cartas Magic) o se añade directamente desde Scryfall si el usuario quiere añadir una carta al mazo pero no la tiene en su colección personal.
- El mazo Commander tiene reglas estrictas: exactamente 1 comandante, máximo 1 excepción de cada tipo de carta, máximo 100 cartas en excepciones, exactamente 100 cartas en el deck principal + comandante (202 max). Estas reglas se validan antes de permitir guardar el mazo.

### Películas/Series (Movie Shows)
- Las películas y series siempre tienen un tipo de media (MOVIE o TV), una puntuación personal del usuario (1-5), un estado (WATCHING, WATCHED, PLAN_TO_WATCH), una reseña opcional, y fechas de inicio/fin de visualización.
- La clasificación de TMDB es la clasificación del público, no la del usuario. El usuario puede calificar la película/series él mismo, y esa calificación es independiente de la de TMDB.

### Mazos Commander (Decks)
- El comandante de un mazo Commander debe ser una carta válida que sea un comandante legal (is:commander en Scryfall).
- El comandante debe ser coherente con los colores declarados del mazo (la identidad de color del comandante debe ser un subconjunto de los colores declarados del mazo).
- El mazo tiene exactamente 100 cartas en el deck principal + comandante (101 cartas totales mínimo? no, 100 cartas en el mazo + comandante = 101, pero el límite máximo es 202 cartas totales). Las excepciones (sideboard) tienen un límite de 100 cartas, con máximo 1 copia de cada carta excepcional. La suma del deck principal + excepciones + comandante no debe exceder 202 cartas.

### Autenticación (Auth)
- Los usernames son únicos y distinguen mayúsculas de minúsculas (case-sensitive).
- Los emails son únicos e insensibles a mayúsculas (case-insensitive).
- Las contraseñas se almacenan como hash BCrypt, nunca en texto plano.

### Colecciones Personales / Visibilidad (User Preferences)
- El usuario puede decidir la visibilidad de cada colección (books, games, boardGames, magicCards, decks, movieShows) de forma independiente: PUBLIC o PRIVATE.
- La visibilidad por defecto es PRIVATE para todas las colecciones.
- Los perfiles públicos muestran solo el username y la fecha de creación del usuario.
- Las colecciones públicas muestran los items de las colecciones marcadas como PUBLIC, con todos sus detalles visibles.
- Las colecciones privadas son visibles solo para el propietario.

---

*Esta lista es una referencia de reglas de negocio que deben mantenerse en el codebase y en la documentación. Si hay contradicciones con el código, el código tiene prioridad (la regla real es lo que el código hace). Actualizar este file si las reglas cambian.*
