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
├── api/                 # Llamadas API y capa de autenticación
│   ├── authApi.ts       # Endpoints de autenticación (login, register, refresh, logout)
│   ├── authFetch.ts     # Fetch wrapper con interceptor de JWT (auto-refresh 401)
│   ├── authStore.ts     # Estado global de autenticación (user, token, métodos)
│   ├── boardgamesApi.ts # Endpoints de juegos de mesa
│   ├── booksApi.ts      # Endpoints de libros
│   ├── deckApi.ts       # Endpoints de mazos Commander
│   ├── errors.ts        # Manejo de errores HTTP (map de códigos a mensajes)
│   ├── gamesApi.ts      # Endpoints de juegos + logros Steam
│   ├── imagesApi.ts     # Endpoints de subida/eliminación de imágenes (Catbox)
│   ├── magicApi.ts      # Endpoints de cartas Magic + comandantes
│   ├── movieshowsApi.ts # Endpoints de películas/series
│   ├── preferencesApi.ts # Endpoints de preferencias de colección
│   ├── statsApi.ts      # Endpoint de estadísticas globales
│   ├── usersApi.ts      # Endpoints de perfiles públicos
│   └── types.ts         # Tipos compartidos de la capa API
├── components/          # Componentes reutilizables (43 archivos)
│   ├── ActionLink.tsx
│   ├── AuthNavigator.tsx
│   ├── BoardGameCard.tsx
│   ├── BoardGameForm.tsx
│   ├── BoardGameSearch.tsx
│   ├── BoardGameStatusBadge.tsx
│   ├── BookCard.tsx
│   ├── BookForm.tsx
│   ├── BookSearch.tsx
│   ├── Breadcrumbs.tsx
│   ├── CardMenu.tsx
│   ├── CollectionPreferencesPanel.tsx
│   ├── CollectionVisibilityBadge.tsx
│   ├── ConfirmDialog.tsx
│   ├── DeckCommanderImage.tsx
│   ├── EmptyState.tsx
│   ├── ErrorBanner.tsx
│   ├── ExportButton.tsx
│   ├── FilterPill.tsx
│   ├── FormSection.tsx
│   ├── GameCard.tsx
│   ├── GameForm.tsx
│   ├── GamePlatformBadge.tsx
│   ├── GamePlatinumBadge.tsx
│   ├── GameSearch.tsx
│   ├── GameStatusBadge.tsx
│   ├── GlobalSearch.tsx
│   ├── HelpModal.tsx
│   ├── ImageUpload.tsx
│   ├── MagicCard.tsx
│   ├── ManaColorDots.tsx
│   ├── MovieShowCard.tsx
│   ├── MovieShowForm.tsx
│   ├── MovieShowSearch.tsx
│   ├── MovieShowStatusBadge.tsx
│   ├── Navbar.tsx
│   ├── OwnerLine.tsx
│   ├── OwnerTabs.tsx
│   ├── Pagination.tsx
│   ├── ProfileView.tsx
│   ├── ProtectedRoute.tsx
│   ├── SearchField.tsx
│   ├── Skeleton.tsx
│   ├── SkeletonInline.tsx
│   ├── SortSelect.tsx
│   ├── Spinner.tsx
│   ├── StarRating.tsx
│   ├── StatusBadge.tsx
│   ├── ThemeToggle.tsx
│   ├── Toast.tsx
│   └── UserProfileHeader.tsx
├── constants/           # Constantes y enums (7 archivos)
│   ├── boardGames.ts    # Labels y colores de estados de juegos de mesa
│   ├── books.ts         # Labels y colores de tipos/estados de libros
│   ├── collections.ts   # Colecciones activas por defecto, tipos de colección
│   ├── decks.ts         # Labels y colores de estados de mazos / maná
│   ├── games.ts         # Labels y colores de plataformas/estados de juegos
│   ├── magic.ts         # Labels y colores de condiciones/idiomas de Magic
│   └── movieshows.ts    # Labels y colores de estados/tipos de películas/series
├── context/             # Contextos de React
│   └── AuthContext.tsx  # Proveedor de autenticación (user, token, login, logout, isAuthenticated)
├── hooks/               # Hooks personalizados (7 archivos)
│   ├── useBackFallback.ts    # Navegación atrás con fallback si history está vacío
│   ├── useCollectionPreferences.ts  # Carga y mutación de preferencias de colección
│   ├── useInfiniteScroll.ts  # Carga infinita con IntersectionObserver
│   ├── useListQuery.ts       # Query con React Query estilo (data, loading, error)
│   ├── usePagedList.ts       # Paginación con page/size/totalPages
│   ├── usePageTitle.ts       # Actualiza document.title desde una cadena
│   ├── useSearchShortcut.ts  # Atajo Ctrl+K / Cmd+K para búsqueda global
│   └── useUnsavedGuard.ts    # Intercepción de navegación con cambios sin guardar (beforeunload)
├── pages/               # Páginas/vistas (30 archivos)
│   ├── Auth/
│   │   ├── LoginPage.tsx
│   │   └── RegisterPage.tsx
│   ├── Profile/
│   │   ├── ProfilePage.tsx          # Perfil propio (editable)
│   │   ├── PreferencesPage.tsx      # Panel de preferencias de colección
│   │   └── PublicProfilePage.tsx    # Perfil público de otro usuario
│   ├── BookListPage.tsx
│   ├── BookCreatePage.tsx
│   ├── BookEditPage.tsx
│   ├── BookDetailPage.tsx
│   ├── GameListPage.tsx
│   ├── GameCreatePage.tsx
│   ├── GameEditPage.tsx
│   ├── GameDetailPage.tsx
│   ├── GameAchievementsPage.tsx
│   ├── BoardGameListPage.tsx
│   ├── BoardGameCreatePage.tsx
│   ├── BoardGameEditPage.tsx
│   ├── BoardGameDetailPage.tsx
│   ├── MagicListPage.tsx
│   ├── MagicDetailPage.tsx
│   ├── MagicCreatePage.tsx
│   ├── DeckListPage.tsx
│   ├── DeckCreatePage.tsx
│   ├── DeckEditPage.tsx
│   ├── DeckDetailPage.tsx
│   ├── MovieShowListPage.tsx
│   ├── MovieShowCreatePage.tsx
│   ├── MovieShowEditPage.tsx
│   ├── MovieShowDetailPage.tsx
│   └── HomePage.tsx
├── types/               # Definiciones de tipos TypeScript (15 archivos)
│   ├── index.ts         # Barra de tipos públicos y exports
│   ├── Api.ts           # Tipos de respuestas paginadas, errores
│   ├── Auth.ts          # User, LoginRequest, RegisterRequest, AuthResponse
│   ├── Preferences.ts   # UserPreferences, CollectionVisibility, ActiveCollections
│   ├── Book.ts
│   ├── BookState.ts
│   ├── BookType.ts
│   ├── BoardGame.ts
│   ├── Deck.ts
│   ├── Game.ts
│   ├── GameApi.ts       # GameSearchResult, AchievementsSummary
│   ├── GamePlatform.ts
│   ├── GameStatus.ts
│   ├── Magic.ts
│   ├── MovieShow.ts
│   ├── MovieShowStatus.ts
│   └── MovieType.ts
├── App.tsx              # Componente raíz con rutas (30 rutas + ProtectedRoute)
├── main.tsx             # Punto de entrada (ReactDOM.createRoot + AuthProvider)
└── index.css            # Estilos globales + Tailwind
```

## Páginas Implementadas (30 páginas)

|| Página | Ruta | Estado | Auth ||
||--------|------|--------|------||
|| Home | `/` | ✅ | Público ||
|| Login | `/login` | ✅ | Público ||
|| Register | `/register` | ✅ | Público ||
|| Perfil propio | `/perfil` | ✅ | Protegido ||
|| Preferencias | `/preferencias` | ✅ | Protegido ||
|| Perfil público | `/perfil/:username` | ✅ | Público ||
|| Lista de libros | `/coleccion` | ✅ | Público ||
|| Detalle de libro | `/coleccion/:id` | ✅ | Público ||
|| Crear libro | `/coleccion/nuevo` | ✅ | Protegido ||
|| Editar libro | `/coleccion/editar/:id` | ✅ | Protegido ||
|| Lista de juegos | `/juegos` | ✅ | Público ||
|| Crear juego | `/juegos/nuevo` | ✅ | Protegido ||
|| Editar juego | `/juegos/editar/:id` | ✅ | Protegido ||
|| Detalle de juego | `/juegos/:id` | ✅ | Público ||
|| Logros de juego | `/juegos/:id/logros` | ✅ | Público ||
|| Lista de juegos de mesa | `/boardgames` | ✅ | Público ||
|| Crear juego de mesa | `/boardgames/nuevo` | ✅ | Protegido ||
|| Detalle de juego de mesa | `/boardgames/:id` | ✅ | Público ||
|| Editar juego de mesa | `/boardgames/:id/editar` | ✅ | Protegido ||
|| Lista de cartas Magic | `/magic` | ✅ | Público ||
|| Crear carta Magic | `/magic/nuevo` | ✅ | Protegido ||
|| Detalle de carta Magic | `/magic/:id` | ✅ | Público ||
|| Lista de mazos | `/magic/mazos` | ✅ | Público ||
|| Crear mazo | `/magic/mazos/nuevo` | ✅ | Protegido ||
|| Detalle de mazo | `/magic/mazos/:id` | ✅ | Público ||
|| Editar mazo | `/magic/mazos/:id/editar` | ✅ | Protegido ||
|| Lista de películas/series | `/movieshows` | ✅ | Público ||
|| Crear película/serie | `/movieshows/nuevo` | ✅ | Protegido ||
|| Editar película/serie | `/movieshows/editar/:id` | ✅ | Protegido ||
|| Detalle de película/serie | `/movieshows/:id` | ✅ | Público ||

## APIs del Frontend (14 archivos)

|| Archivo | Responsabilidad ||
||---------|----------------||
|| `authApi.ts` | login, register, refreshToken, getCurrentUser ||
|| `authFetch.ts` | Fetch wrapper: añade Authorization header, maneja 401 con refresh automático ||
|| `authStore.ts` | Estado global: user, accessToken, login, logout, isAuthenticated ||
|| `booksApi.ts` | CRUD libros + search Google Books ||
|| `gamesApi.ts` | CRUD juegos + search RAWG + achievements Steam ||
|| `boardgamesApi.ts` | CRUD juegos de mesa + search BGG ||
|| `magicApi.ts` | Listado, detalle, search Scryfall, commanders, añadir desde Scryfall ||
|| `deckApi.ts` | CRUD mazos + añadir/quitar cartas + status ||
|| `movieshowsApi.ts` | CRUD películas/series + search TMDB ||
|| `imagesApi.ts` | Subir y eliminar imágenes (Catbox proxy) ||
|| `preferencesApi.ts` | GET/PUT/PATCH preferencias de colección ||
|| `statsApi.ts` | GET estadísticas globales ||
|| `usersApi.ts` | GET perfil público + colecciones públicas de usuario ||
|| `errors.ts` | Map de códigos HTTP a mensajes amigables + función handleError ||

## Hooks del Frontend (8 hooks)

|| Hook | Responsabilidad ||
||------|--------|----------------||
|| `useAuth` (AuthContext) | user, accessToken, login, logout, isAuthenticated, register ||
|| `useBackFallback` | Navegación atrás con fallback a ruta por defecto ||
|| `useCollectionPreferences` | Carga y guardado de preferencias de colección (active + visibility) ||
|| `useInfiniteScroll` | Carga infinita con IntersectionObserver + callback ||
|| `useListQuery` | Query stateful con loading, error, data ||
|| `usePagedList` | Paginación stateful (page, size, totalPages, pageNumbers) ||
|| `usePageTitle` | Actualiza document.title de forma declarativa ||
|| `useSearchShortcut` | Atajo global Ctrl+K / Cmd+K para abrir GlobalSearch ||
|| `useUnsavedGuard` | beforeunload + intercepto de navegación con formulario sin guardar ||

## Estado de la Wiki del Frontend

- `docs/06-frontend/06.1-componentes.md` — ✅ Cubre los 43 componentes (incluye fase 9: CollectionPreferencesPanel, CollectionVisibilityBadge, UserProfileHeader, OtherCollectionsSection)
- `docs/06-frontend/06.2-navegacion.md` — ✅ Cubre las 30 rutas + navegación entre páginas
- `docs/06-frontend/README.md` — 📋 Actualizar: estructura actual con auth, APIs, hooks, tipos nuevos
