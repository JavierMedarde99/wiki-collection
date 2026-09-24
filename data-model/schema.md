# Tablas / Colecciones — Wiki-Collection

## MongoDB

Se usa MongoDB como base de datos principal por su flexibilidad de esquemas, ideal para colecciones con atributos variables por entidad.

### Ventajas

- **Esquemas flexibles:** Cada entidad puede tener campos diferentes
- **Escalabilidad horizontal:** Sharding y replicación
- **Integración con Spring Data:** Repositorios listos para usar
- **JSON nativo:** Alineado con el formato de APIs REST

### Colecciones

| Colección | Descripción | Documento Entity |
|-----------|-------------|------------------|
| `books` | Libros de la colección | BookEntity |
| `games` | Videojuegos de la colección | GameEntity |
| `board_games` | Juegos de mesa | BoardGameEntity |
| `magic_cards` | Cartas Magic | MagicCardEntity |
| `movie_shows` | Películas y series | MovieShowEntity |
| `decks` | Mazos Commander | DeckEntity |
| `users` | Usuarios registrados | UserEntity |
| `user_preferences` | Preferencias de colección por usuario | UserPreferencesEntity |
| `user_owned` | Relación usuario-propietario (bypass de joins) | UserOwnedEntity |

### Índices

#### Books
- `id` (PK, automático)
- `externalId` (búsqueda de duplicados, unique)
- `state` (para filtros)
- `title` (para búsquedas de texto)

#### Games
- `id` (PK, automático)
- `externalId` (búsqueda de duplicados, unique)
- `status` (para filtros)
- `title` (para búsquedas de texto)
- `steamAppId` (para logros Steam)

#### Board Games
- `id` (PK, automático)
- `bggId` (búsqueda por ID externo BGG)
- `status` (para filtros)
- `title` (para búsquedas de texto)

#### Magic Cards
- `id` (PK, automático)
- `name` (para búsquedas de texto)
- `scryfallId` (ID externo de Scryfall)
- `oracleId` (ID de oracle, único por carta)

#### Movie Shows
- `id` (PK, automático)
- `externalId` (búsqueda de duplicados, unique)
- `title` (para búsquedas de texto)
- `status` (para filtros)

#### Decks
- `id` (PK, automático)
- `name` (para búsquedas por nombre)

#### Users
- `id` (PK, automático)
- `username` (unique, indexed)
- `email` (unique, indexed)

#### User Preferences
- `id` (PK, automático)
- `userId` (unique, indexed)

#### User Owned
- `id` (PK, automático)
- `userId` (indexed)

---

## Tablas / Colecciones

### Books (Libros) — Colección: books

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| externalId | String | ❌ | ID en API externa (Google Books) |
| title | String | ✅ | Título del libro |
| descripcion | String | ❌ | Sinopsis (nota: el backend usa 'descripcion', no 'description') |
| author | String | ✅ | Autor (string singular, no lista) |
| pages | Integer | ❌ | Número de páginas |
| type | Enum | ✅ | MANGA, NOVEL, GRAPHIC_NOVEL |
| state | Enum | ✅ | TO_READ, READING, COMPLETED |
| comment | String | ❌ | Notas personales |
| start | Integer (0-5) | ❌ | Valoración personal |
| startDate | LocalDate | ❌ | Fecha de inicio de lectura |
| endDate | LocalDate | ❌ | Fecha de fin de lectura |
| frontpage | String | ❌ | URL a la portada |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Games (Videojuegos) — Colección: games

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| externalId | String | ❌ | ID en RAWG/FreeToGame |
| title | String | ✅ | Título del juego |
| platform | Enum | ✅ | PC, PS2, PS3, WII_U, SWITCH |
| thumbnailUrl | String | ❌ | URL al thumbnail |
| status | Enum | ✅ | PLAYING, COMPLETED, WISHLIST, ABANDONED |
| userRating | Integer (1-5) | ❌ | Valoración personal |
| comment | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió |
| dateCompleted | LocalDate | ❌ | Cuándo se completó |
| externalSource | String | ❌ | RAWG, FreeToGame |
| steamAppId | String | ❌ | ID de juego en Steam (para logros) |
| obtainPlatinum | Boolean | ❌ | Si obtuvo trophy platinum |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Board Games (Juegos de Mesa) — Colección: board_games

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| title | String | ✅ | Nombre del juego de mesa |
| description | String | ❌ | Descripción del juego |
| yearPublished | Integer | ❌ | Año de publicación |
| minPlayers | Integer | ❌ | Mínimo de jugadores |
| maxPlayers | Integer | ❌ | Máximo de jugadores |
| minPlaytime | Integer | ❌ | Duración mínima (minutos) |
| maxPlaytime | Integer | ❌ | Duración máxima (minutos) |
| publisher | String | ❌ | Editorial/publicador |
| designers | List<String> | ❌ | Lista de diseñadores |
| categories | List<String> | ❌ | Categorías del juego |
| mechanics | List<String> | ❌ | Mecánicas de juego |
| imageUrl | String | ❌ | URL de imagen completa |
| thumbnailUrl | String | ❌ | URL de miniatura |
| bggRating | BigDecimal | ❌ | Rating promedio BGG (0-10) |
| bggId | String | ❌ | ID externo de BoardGameGeek |
| status | Enum | ✅ | OWNED, WISHLIST |
| notes | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió a la colección |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

**Nota:** El enum `BoardGameStatus` en backend solo tiene `OWNED` y `WISHLIST` (no `PREVIOUSLY_OWNED` ni `FOR_TRADE`). AD-012: BoardGameStatus reducido.

### Magic Cards (Cartas Magic) — Colección: magic_cards

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| scryfallId | String | ❌ | ID único de Scryfall |
| oracleId | String | ❌ | ID de oracle (único por carta) |
| name | String | ✅ | Nombre de la carta |
| language | Enum | ❌ | ENGLISH, SPANISH, FRENCH, GERMAN, ITALIAN, PORTUGUESE, JAPANESE, CHINESE |
| releaseDate | String | ❌ | Fecha de lanzamiento del set |
| manaCost | String | ❌ | Coste de maná (ej: `{2}{R}{R}`) |
| convertedManaCost | Double | ❌ | Coste de maná convertido |
| type | String | ❌ | Tipo de carta (ej: "Instant", "Creature") |
| text | String | ❌ | Texto/rules de la carta |
| power | String | ❌ | Fuerza (criaturas) |
| toughness | String | ❌ | Resistencia (criaturas) |
| loyalty | String | ❌ | Lealtad (planeswalkers) |
| colors | List<String> | ❌ | Colores de la carta |
| colorIdentity | List<String> | ❌ | Identidad de color |
| keywords | List<String> | ❌ | Palabras clave |
| rarity | String | ❌ | Rareza (common, uncommon, rare, mythic) |
| setCode | String | ❌ | Código del set |
| setName | String | ❌ | Nombre del set |
| artist | String | ❌ | Artista de la ilustración |
| frame | String | ❌ | Año del frame |
| borderColor | String | ❌ | Color del borde |
| layout | String | ❌ | Layout de la carta |
| legalities | Map<String,String> | ❌ | Legalidades por formato |
| priceUsd | String | ❌ | Precio en USD |
| priceEur | String | ❌ | Precio en EUR |
| imageUrl | String | ❌ | URL de imagen normal |
| imageLargeUrl | String | ❌ | URL de imagen grande |
| artCropUrl | String | ❌ | URL de arte recortado |
| condition | Enum | ❌ | MINT, NEAR_MINT, EXCELLENT, GOOD, PLAYED, POOR |
| isFoil | Boolean | ❌ | Si es foil |
| quantity | Integer | ❌ | Cantidad de copias |
| notes | String | ❌ | Notas personales |
| dateAdded | LocalDateTime | ❌ | Cuándo se añadió a la colección |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

### Movies/Shows (Películas/Series) — Colección: movie_shows

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| externalId | String | ❌ | ID en TMDB (unique) |
| title | String | ✅ | Título de la película/serie |
| overview | String | ❌ | Sinopsis |
| releaseDate | LocalDate | ❌ | Fecha de estreno |
| posterUrl | String | ❌ | URL del póster |
| backdropUrl | String | ❌ | URL de la imagen de fondo |
| voteAverage | Double | ❌ | Puntuación media TMDB |
| mediaType | Enum | ✅ | MOVIE, TV |
| status | Enum | ✅ | WATCHING, WATCHED, PLAN_TO_WATCH |
| userRating | Integer (1-5) | ❌ | Valoración personal |
| comment | String | ❌ | Notas personales |
| dateAdded | LocalDate | ❌ | Cuándo se añadió a la colección |
| dateCompleted | LocalDate | ❌ | Cuándo se terminó de ver |
| externalSource | String | ❌ | "TMDB" |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

**Nota:** El enum `MovieStatus` NO incluye WISHLIST (AD-015: MovieStatus sin WISHLIST). Solo 3 valores: WATCHING, WATCHED, PLAN_TO_WATCH.

### Decks (Mazos Commander) — Colección: decks

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| name | String | ✅ | Nombre del mazo |
| description | String | ❌ | Descripción del mazo |
| commander | String | ❌ | Nombre del comandante |
| commanderColors | List<String> | ❌ | Colores del comandante |
| cards | List<DeckCardEntity> | ❌ | Cartas del mazo |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |
| ownerId | String | ✅ | ID del usuario propietario (Fase 8+) |

#### DeckCardEntity (subdocumento embebido)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| cardName | String | Nombre de la carta |
| quantity | Integer | Cantidad de copias |
| inCollection | Boolean | Si está en la colección Magic |
| isProxy | Boolean | Si es un proxy |
| manaCost | String | Coste de maná |
| typeLine | String | Línea de tipo |
| colorIdentity | List<String> | Identidad de color |
| imageUrl | String | URL de imagen |
| scryfallId | String | ID en Scryfall |

### UserPreferences (Preferencias de Usuario) — Colección: user_preferences

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único |
| userId | String | ✅ | ID del usuario (unique, indexed) |
| activeCollections | Map<String, Boolean> | ✅ | Mapa de colecciones activadas (books, games, boardgames, magic, decks, movieshows) |
| collectionVisibility | Map<String, String> | ✅ | Visibilidad por colección: "PUBLIC" o "PRIVATE" |
| createdAt | LocalDateTime | Auto | Fecha de creación |
| updatedAt | LocalDateTime | Auto | Última actualización |

### User (Usuarios) — Colección: users

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único |
| username | String | ✅ | Nombre de usuario (unique, 3-20 chars, regex `^[a-zA-Z0-9_]+$`) |
| email | String | ✅ | Email (unique, validado) |
| password | String | ✅ | Hash BCrypt (nunca se devuelve) |
| displayName | String | ❌ | Nombre para mostrar |
| avatarUrl | String | ❌ | URL de avatar |
| bio | String | ❌ | Biografía corta (max 200 chars) |
| createdAt | LocalDateTime | Auto | Fecha de registro |
| updatedAt | LocalDateTime | Auto | Última actualización |

### UserOwned (Relación Usuario-Propietario) — Colección: user_owned

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único |
| userId | String | ✅ | ID del usuario |
| username | String | ✅ | Nombre de usuario |
| displayName | String | ❌ | Nombre para mostrar |

**Nota:** `UserOwnedBackfillMigration` (CommandLineRunner) pobla esta colección automáticamente para evitar joins en queries de visibilidad.

---

## Imágenes (Fase 6)

Las imágenes se almacenan en **Catbox.moe** (hosting externo), no en el filesystem local ni en MongoDB.

**Flujo:**
1. Cliente sube imagen vía `POST /api/v1/images/upload`
2. Backend valida (5MB máximo, MIME permitido: image/jpeg, image/png, image/gif, image/webp; magic bytes)
3. Backend sube a Catbox vía `CatboxClient`
4. Backend devuelve `ImageResponse` con `url` y `filename`
5. URL se almacena en el campo de imagen de cada entidad (`frontpage`, `thumbnailUrl`, `posterUrl`, etc.)
6. Para eliminar: `DELETE /api/v1/images/{filename}` → elimina de Catbox (requiere userhash)

**Archivos relacionados:**
- `ImageStorageController.java` — Controlador REST
- `CatboxClient.java` — Cliente API Catbox
- `ImageStorageService.java` — Servicio de validación + upload
- `ImageResponse.java` — DTO de respuesta
- `CatboxUploadException.java` — Excepción si falla el upload (HTTP 502)

**Nota:** La configuración `app.image.storage.path` ya no se usa (era para filesystem local).
