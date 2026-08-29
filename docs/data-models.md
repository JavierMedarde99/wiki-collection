# Modelos de Datos

## Book (Libro)

```javascript
{
  _id: ObjectId,           // MongoDB
  title: String,           // Título del libro
  authors: [String],       // Lista de autores
  isbn: String,            // ISBN-13 o ISBN-10 (único)
  publisher: String,       // Editorial
  publishedDate: Date,     // Fecha de publicación
  description: String,     // Sinopsis
  pageCount: Number,       // Número de páginas
  categories: [String],    // Géneros/categorías
  coverImage: String,      // URL a la portada
  language: String,        // Idioma (es, en, etc.)
  
  // Campos de colección personal
  status: String,          // 'reading' | 'completed' | 'wishlist' | 'abandoned'
  userRating: Number,      // 1-5 (valoración personal)
  notes: String,           // Notas personales
  dateAdded: Date,         // Cuándo se añadió a la colección
  dateCompleted: Date,     // Cuándo se terminó (si aplica)
  tags: [String],          // Tags personalizados
  
  // Metadata de la API externa
  externalSource: String,  // 'google_books' | 'open_library'
  externalId: String,      // ID en la API externa
  
  createdAt: Date,
  updatedAt: Date
}
```

## Game (Videojuego) — Fase 2

```javascript
{
  _id: ObjectId,
  title: String,
  platform: String,        // PS5, PC, Switch, etc.
  publisher: String,
  releaseDate: Date,
  genre: [String],
  coverImage: String,
  status: String,          // 'playing' | 'completed' | 'backlog' | 'wishlist'
  userRating: Number,
  hoursPlayed: Number,
  notes: String,
  dateAdded: Date,
  createdAt: Date,
  updatedAt: Date
}
```

## BoardGame (Juego de Mesa) — Fase 3

```javascript
{
  _id: ObjectId,
  title: String,
  publisher: String,
  minPlayers: Number,
  maxPlayers: Number,
  playTime: Number,        // Minutos
  categories: [String],
  coverImage: String,
  userRating: Number,
  dateAcquired: Date,
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

## MagicCard (Carta Magic) — Fase 4

```javascript
{
  _id: ObjectId,
  name: String,
  manaCost: String,
  type: String,
  rarity: String,
  set: String,
  text: String,
  power: String,
  toughness: String,
  imageUrl: String,
  quantity: Number,
  condition: String,       // 'mint' | 'near_mint' | 'excellent' | ...
  language: String,
  foil: Boolean,
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

## MovieShow (Película/Serie) — Fase 5

```javascript
{
  _id: ObjectId,
  title: String,
  type: String,            // 'movie' | 'series'
  director: String,
  cast: [String],
  genre: [String],
  releaseYear: Number,
  runtime: Number,         // Minutos (película) o duración media (serie)
  seasons: Number,         // Solo series
  episodes: Number,        // Solo series
  posterUrl: String,
  status: String,          // 'watching' | 'completed' | 'wishlist' | 'dropped'
  userRating: Number,
  watchedDate: Date,
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```
