# Carpetas del Frontend — Wiki-Collection

**stack frontend:** React 18.3 + Vite 5 + React Router v6 + Tailwind CSS 3 + TypeScript 7

## Estructura

```
frontend/
├── index.html
├── favicon.svg
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── tailwind.config.js
├── postcss.config.js
├── vite.config.ts
└── src/
    ├── main.tsx                          # Entry point
    ├── App.tsx                           # Root con BrowserRouter + lazy routes
    ├── index.css                         # Tailwind + global styles
    ├── constants.ts                      # Constantes (API_URL_VITE, COLORS)
    ├── types/
    │   ├── books.ts                      # Book, BookRequest, BookResponse, BookType, BookState
    │   ├── games.ts                      # Game, GameRequest, GameResponse, Platform, GameStatus
    │   ├── boardgames.ts                 # BoardGame, BoardGameRequest, BoardGameResponse, BoardGameStatus
    │   ├── magic.ts                      # MagicCard, MagicCardRequest, MagicCardResponse, MagicLanguage, MagicLoyalty, MagicBorderColor
    │   ├── decks.ts                      # Deck, DeckRequest, DeckResponse, DeckCard, CommanderColors, DeckStatus
    │   ├── movie-shows.ts                # MovieShow, MovieShowRequest, MovieShowResponse, MovieShowStatus, MovieShowMediaType
    │   ├── auth.ts                       # RegisterRequest, LoginRequest, AuthResponse, UserResponse, RefreshRequest
    │   ├── user-preferences.ts          # VisibilityRequest, UserVisibilityResponse, PublicUserCollections, PublicUserCollectionsPage
    │   └── index.ts                     # Export todos los tipos
    ├── hooks/
    │   ├── useAuth.ts                   # Auth state: isAuthenticated, currentUser, login, logout, forgotPassword
    │   ├── useBook.ts                   # CRUD libros: list, search, create, update, delete, getById
    │   ├── useGame.ts                   # CRUD juegos: list, search, create, update, delete, getById, achievements
    │   ├── useBoardgame.ts              # CRUD juegos de mesa: list, search, create, update, delete, getById
    │   ├── useMagic.ts                  # CRUD cartas Magic: list, search, addFromScryfall, delete, getById
    │   ├── useDeck.ts                   # CRUD mazos Commander: list, create, update, delete, getById, cards CRUD, commanders, status, pdf
    │   ├── useMovieShow.ts              # CRUD películas/series: list, search, create, update, delete, getById, images
    │   ├── useImageStorage.ts           # Upload imágenes: upload
    │   └── index.ts
    ├── components/
    │   ├── cards/                       # Cards de cada entidad
    │   │   ├── BookCard.tsx
    │   │   ├── GameCard.tsx
    │   │   ├── BoardGameCard.tsx
    │   │   ├── MagicCardCard.tsx
    │   │   ├── DeckCard.tsx
    │   │   ├── MovieShowCard.tsx
    │   │   └── index.ts
    │   ├── forms/                       # Formularios de cada entidad
    │   │   ├── BookForm.tsx
    │   │   ├── GameForm.tsx
    │   │   ├── BoardGameForm.tsx
    │   │   ├── MagicForm.tsx
    │   │   ├── DeckForm.tsx
    │   │   ├── MovieShowForm.tsx
    │   │   └── index.ts
    │   ├── search/                      # Barras de búsqueda
    │   │   ├── SearchBar.tsx
    │   │   ├── SearchResultItem.tsx
    │   │   └── index.ts
    │   ├── badge/                       # Badges de tipos y estados
    │   │   ├── TypeBadge.tsx
    │   │   ├── StatusBadge.tsx
    │   │   ├── PlatformBadge.tsx
    │   │   └── index.ts
    │   ├── ui/                          # Componentes UI genéricos
    │   │   ├── LoadingSpinner.tsx
    │   │   ├── ErrorMessage.tsx
    │   │   ├── EmptyState.tsx
    │   │   ├── Toast.tsx
    │   │   └── index.ts
    │   └── index.ts
    └── pages/
        ├── HomePage.tsx                 # Home (sin auth: login modal)
        ├── AuthPage.tsx                 # Login + Register (tabs)
        ├── BooksPage.tsx                # CRUD libros + search (Google Books)
        ├── GamesPage.tsx                # CRUD juegos + search (RAWG + FreeToGame)
        ├── BoardGamesPage.tsx           # CRUD juegos de mesa + search (BGG)
        ├── MagicPage.tsx                # CRUD cartas Magic + search (Scryfall) — solo eliminar y añadir desde Scryfall
        ├── DecksPage.tsx                # CRUD mazos Commander + planillas (list, cards, commanders, PDF)
        ├── MovieShowsPage.tsx           # CRUD películas/series + search (TMDB) + imágenes
        ├── ImagePage.tsx                # Upload imágenes a Catbox.moe
        ├── ForgotPasswordPage.tsx       # 404 página de recuperación de contraseña (no implementada)
        └── index.ts
```

## App.tsx — Lazy Loading

```tsx
import { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const HomePage = lazy(() => import('./pages/HomePage'));
const AuthPage = lazy(() => import('./pages/AuthPage'));
const BooksPage = lazy(() => import('./pages/BooksPage'));
const GamesPage = lazy(() => import('./pages/GamesPage'));
const BoardGamesPage = lazy(() => import('./pages/BoardGamesPage'));
const MagicPage = lazy(() => import('./pages/MagicPage'));
const DecksPage = lazy(() => import('./pages/DecksPage'));
const MovieShowsPage = lazy(() => import('./pages/MovieShowsPage'));
const ImagePage = lazy(() => import('./pages/ImagePage'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/auth" element={<AuthPage />} />
          <Route path="/books" element={<BooksPage />} />
          <Route path="/games" element={<GamesPage />} />
          <Route path="/boardgames" element={<BoardGamesPage />} />
          <Route path="/magic" element={<MagicPage />} />
          <Route path="/decks" element={<DecksPage />} />
          <Route path="/movieshows" element={<MovieShowsPage />} />
          <Route path="/image" element={<ImagePage />} />
          <Route path="*" element={<NotFoundPage />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

export default App;
```

## Referencia rápida: resources por página

| Página | Endpoints usados | Capabilities |
|--------|-----------------|--------------|
| `HomePage` | `/me/visibility`, `/users/{me}` | user-preferences |
| `BooksPage` | `/books`, `/books/search` | books |
| `GamesPage` | `/games`, `/games/search`, `/games/{id}/achievements` | games |
| `BoardGamesPage` | `/boardgames`, `/boardgames/search` | board-games |
| `MagicPage` | `/magic`, `/magic/search`, `/magic/scryfall/{id}` | magic-cards |
| `DecksPage` | `/decks`, `/decks/colors/{colors}/commanders`, `/decks/{id}`, `/decks/{id}/cards`, `/decks/{id}/cards/others`, `/decks/{id}/pdf`, `/decks/{id}/status` | decks |
| `MovieShowsPage` | `/movieshows`, `/movieshows/search`, `/movieshows/images/{id}` | movie-shows |
| `ImagePage` | `/image/upload` | image-storage |
