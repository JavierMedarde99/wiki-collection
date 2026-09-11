# Frontend

## Stack

- **Framework:** React 18.3+
- **Build:** Vite 5
- **Routing:** React Router v6
- **Estilos:** Tailwind CSS 3
- **Estado:** useState, useEffect, useCallback
- **HTTP Client:** Fetch API (custom wrapper)
- **Tipado:** TypeScript 5 (tsx)
- **Testing:** Vitest + React Testing Library

## Estructura de Carpetas

```
src/
├── api/                 # Llamadas API
│   ├── booksApi.ts      # Endpoints de libros
│   ├── gamesApi.ts      # Endpoints de juegos
│   ├── boardgamesApi.ts # Endpoints de juegos de mesa
│   ├── magicApi.ts      # Endpoints de cartas Magic
│   ├── movieshowsApi.ts # Endpoints de películas/series
│   └── deckApi.ts       # Endpoints de mazos Commander
├── components/          # Componentes reutilizables
│   ├── BoardGameCard.tsx
│   ├── BoardGameForm.tsx
│   ├── BoardGameSearch.tsx
│   ├── BoardGameStatusBadge.tsx
│   ├── BookCard.tsx
│   ├── BookForm.tsx
│   ├── BookSearch.tsx
│   ├── ConfirmDialog.tsx
│   ├── DeckCommanderImage.tsx
│   ├── EmptyState.tsx
│   ├── ErrorBanner.tsx
│   ├── GameCard.tsx
│   ├── GameForm.tsx
│   ├── GamePlatformBadge.tsx
│   ├── GamePlatinumBadge.tsx
│   ├── GameSearch.tsx
│   ├── GameStatusBadge.tsx
│   ├── MagicCard.tsx
│   ├── MovieShowCard.tsx
│   ├── MovieShowForm.tsx
│   ├── MovieShowSearch.tsx
│   ├── MovieShowStatusBadge.tsx
│   ├── Navbar.tsx
│   ├── Skeleton.tsx
│   ├── Spinner.tsx
│   ├── StarRating.tsx
│   └── StatusBadge.tsx
├── constants/           # Constantes y enums
│   ├── boardGames.ts    # Labels y colores de estados de juegos de mesa
│   ├── books.ts         # Labels y colores de tipos/estados de libros
│   ├── decks.ts         # Labels y colores de estados de mazos / maná
│   ├── games.ts         # Labels y colores de plataformas/estados de juegos
│   ├── magic.ts         # Labels y colores de condiciones/idiomas de Magic
│   └── movieshows.ts    # Labels y colores de estados/tipos de películas/series
├── pages/               # Páginas/vistas
│   ├── BoardGameCreatePage.tsx
│   ├── BoardGameDetailPage.tsx
│   ├── BoardGameEditPage.tsx
│   ├── BoardGameListPage.tsx
│   ├── BookCreatePage.tsx
│   ├── BookDetailPage.tsx
│   ├── BookEditPage.tsx
│   ├── BookListPage.tsx
│   ├── DeckCreatePage.tsx
│   ├── DeckDetailPage.tsx
│   ├── DeckEditPage.tsx
│   ├── DeckListPage.tsx
│   ├── GameAchievementsPage.tsx
│   ├── GameCreatePage.tsx
│   ├── GameDetailPage.tsx
│   ├── GameEditPage.tsx
│   ├── GameListPage.tsx
│   ├── HomePage.tsx
│   ├── MagicCreatePage.tsx
│   ├── MagicDetailPage.tsx
│   ├── MagicEditPage.tsx
│   ├── MagicListPage.tsx
│   ├── MovieShowCreatePage.tsx
│   ├── MovieShowDetailPage.tsx
│   ├── MovieShowEditPage.tsx
│   └── MovieShowListPage.tsx
├── __tests__/           # Tests
│   ├── BookCard.test.tsx
│   ├── BookForm.test.tsx
│   └── BookSearch.test.tsx
├── types/               # Definiciones de tipos
│   ├── Api.ts
│   ├── BoardGame.ts
│   ├── Book.ts
│   ├── BookState.ts
│   ├── BookType.ts
│   ├── Deck.ts
│   ├── Game.ts
│   ├── GameApi.ts
│   ├── GamePlatform.ts
│   ├── GameStatus.ts
│   ├── index.ts
│   ├── Magic.ts
│   ├── MovieShow.ts
│   ├── MovieShowStatus.ts
│   ├── MovieType.ts
│   └── index.ts
├── App.tsx              # Componente raíz con rutas
├── main.tsx             # Punto de entrada
└── index.css            # Estilos globales + Tailwind
```

## Páginas Implementadas

| Página | Ruta | Estado |
|--------|------|--------|
| Home | `/` | ✅ |
| Lista de libros | `/coleccion` | ✅ |
| Detalle de libro | `/coleccion/:id` | ✅ |
| Crear libro | `/nuevo` | ✅ |
| Editar libro | `/editar/:id` | ✅ |
| Lista de juegos | `/juegos` | ✅ |
| Crear juego | `/juegos/nuevo` | ✅ |
| Editar juego | `/juegos/editar/:id` | ✅ |
| Detalle de juego | `/juegos/:id` | ✅ |
| Logros de juego | `/juegos/:id/logros` | ✅ |
| Lista de juegos de mesa | `/boardgames` | ✅ |
| Crear juego de mesa | `/boardgames/nuevo` | ✅ |
| Detalle de juego de mesa | `/boardgames/:id` | ✅ |
| Editar juego de mesa | `/boardgames/:id/editar` | ✅ |
| Lista de cartas Magic | `/magic` | ✅ |
| Crear carta Magic | `/magic/nuevo` | ✅ |
| Detalle de carta Magic | `/magic/:id` | ✅ |
| Editar carta Magic | `/magic/:id/editar` | ✅ |
| Lista de mazos | `/magic/mazos` | ✅ |
| Crear mazo | `/magic/mazos/nuevo` | ✅ |
| Detalle de mazo | `/magic/mazos/:id` | ✅ |
| Editar mazo | `/magic/mazos/:id/editar` | ✅ |
| Lista de películas/series | `/movieshows` | ✅ |
| Crear película/serie | `/movieshows/nuevo` | ✅ |
| Editar película/serie | `/movieshows/editar/:id` | ✅ |
| Detalle de película/serie | `/movieshows/:id` | ✅ |
