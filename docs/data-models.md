# Modelos de Datos

## Book (Libro)

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| id | String (ObjectId) | Auto | Identificador único MongoDB |
| title | String | ✅ | Título del libro |
| authors | List<String> | ✅ | Lista de autores |
| isbn | String | ❌ | ISBN-13 o ISBN-10 (único) |
| publisher | String | ❌ | Editorial |
| publishedDate | LocalDate | ❌ | Fecha de publicación |
| description | String | ❌ | Sinopsis |
| pageCount | Integer | ❌ | Número de páginas |
| categories | List<String> | ❌ | Géneros/categorías |
| coverImage | String | ❌ | URL a la portada |
| language | String | ❌ | Idioma (es, en, etc.) |
| status | Enum | ✅ | wishlist, reading, completed, abandoned |
| userRating | Integer (1-5) | ❌ | Valoración personal |
| notes | String | ❌ | Notas personales |
| tags | List<String> | ❌ | Tags personalizados |
| dateAdded | LocalDateTime | auto | Cuándo se añadió a la colección |
| dateCompleted | LocalDateTime | auto | Cuándo se terminó |
| externalSource | String | ❌ | google_books o open_library |
| externalId | String | ❌ | ID en la API externa |

## Game (Videojuego) — Fase 2

| Campo | Tipo | Requerido |
|-------|------|-----------|
| id | String (ObjectId) | Auto |
| title | String | ✅ |
| platform | String | ✅ |
| publisher | String | ❌ |
| releaseDate | LocalDate | ❌ |
| genre | List<String> | ❌ |
| coverImage | String | ❌ |
| status | Enum | ✅ |
| userRating | Integer (1-5) | ❌ |
| hoursPlayed | Double | ❌ |
| notes | String | ❌ |
| dateAdded | LocalDateTime | auto |

## BoardGame (Juego de Mesa) — Fase 3

| Campo | Tipo | Requerido |
|-------|------|-----------|
| id | String (ObjectId) | Auto |
| title | String | ✅ |
| publisher | String | ❌ |
| minPlayers | Integer | ❌ |
| maxPlayers | Integer | ❌ |
| playTime | Integer | ❌ |
| categories | List<String> | ❌ |
| coverImage | String | ❌ |
| userRating | Integer (1-5) | ❌ |
| dateAcquired | LocalDate | ❌ |
| notes | String | ❌ |

## MagicCard (Carta Magic) — Fase 4

| Campo | Tipo | Requerido |
|-------|------|-----------|
| id | String (ObjectId) | Auto |
| name | String | ✅ |
| manaCost | String | ❌ |
| type | String | ❌ |
| rarity | String | ❌ |
| set | String | ❌ |
| text | String | ❌ |
| power | String | ❌ |
| toughness | String | ❌ |
| imageUrl | String | ❌ |
| quantity | Integer | ❌ |
| condition | String | ❌ |
| language | String | ❌ |
| foil | Boolean | ❌ |
| notes | String | ❌ |

## MovieShow (Película/Serie) — Fase 5

| Campo | Tipo | Requerido |
|-------|------|-----------|
| id | String (ObjectId) | Auto |
| title | String | ✅ |
| type | Enum (movie, series) | ✅ |
| director | String | ❌ |
| cast | List<String> | ❌ |
| genre | List<String> | ❌ |
| releaseYear | Integer | ❌ |
| runtime | Integer | ❌ |
| seasons | Integer | ❌ |
| episodes | Integer | ❌ |
| posterUrl | String | ❌ |
| status | Enum | ✅ |
| userRating | Integer (1-5) | ❌ |
| watchedDate | LocalDate | ❌ |
| notes | String | ❌ |
