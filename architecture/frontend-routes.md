# Rutas del Frontend — Wiki-Collection

## 30 Rutas Implementadas

### Pública (anónimo puede acceder)
| Ruta | Componente | Descripción |
|------|-------------|-------------|
| `/` | `HomePage` | Página principal con estadísticas |
| `/login` | `LoginPage` | Formulario de login |
| `/register` | `RegisterPage` | Formulario de registro |
| `/perfil/:username` | `PublicProfilePage` | Perfil público de un usuario |
| `/coleccion` | `BookListPage` | Lista de libros |
| `/coleccion/:id` | `BookDetailPage` | Detalle de libro |
| `/juegos` | `GameListPage` | Lista de juegos |
| `/juegos/:id` | `GameDetailPage` | Detalle de juego |
| `/juegos/:id/logros` | `GameAchievementsPage` | Logros de Steam de un juego |
| `/boardgames` | `BoardGameListPage` | Lista de juegos de mesa |
| `/boardgames/:id` | `BoardGameDetailPage` | Detalle de juego de mesa |
| `/magic` | `MagicListPage` | Lista de cartas Magic |
| `/magic/:id` | `MagicDetailPage` | Detalle de carta Magic |
| `/magic/mazos` | `DeckListPage` | Lista de mazos Commander |
| `/magic/mazos/:id` | `DeckDetailPage` | Detalle de mazo |
| `/movieshows` | `MovieShowListPage` | Lista de películas/series |
| `/movieshows/:id` | `MovieShowDetailPage` | Detalle de película/serie |

### Protegido (requiere auth)
| Ruta | Componente | Descripción |
|------|-------------|-------------|
| `/perfil` | `ProfilePage` | Perfil propio (editable) |
| `/preferencias` | `PreferencesPage` | Panel de preferencias de colección |
| `/coleccion/nuevo` | `BookCreatePage` | Crear libro |
| `/coleccion/editar/:id` | `BookEditPage` | Editar libro |
| `/juegos/nuevo` | `GameCreatePage` | Crear juego |
| `/juegos/editar/:id` | `GameEditPage` | Editar juego |
| `/boardgames/nuevo` | `BoardGameCreatePage` | Crear juego de mesa |
| `/boardgames/:id/editar` | `BoardGameEditPage` | Editar juego de mesa |
| `/magic/nuevo` | `MagicCreatePage` | Crear carta Magic (desde Scryfall) |
| `/magic/mazos/nuevo` | `DeckCreatePage` | Crear mazo Commander |
| `/magic/mazos/:id/editar` | `DeckEditPage` | Editar mazo |
| `/movieshows/nuevo` | `MovieShowCreatePage` | Crear película/serie |
| `/movieshows/editar/:id` | `MovieShowEditPage` | Editar película/serie |

## Navegación entre páginas

- **Home → Listado:** Desde HomePage se accede a cada listado mediante los links del navbar
- **Listado → Detalle:** Cada tarjeta en la lista enlaza al detalle
- **Detalle → Edición:** Botón "Editar" en el detalle (solo visible si eres el propietario)
- **Detalle → Eliminación:** Botón "Eliminar" con ConfirmDialog
- **Auth → Perfil:** Tras login/register, se redirige al perfil propio `/perfil`
- **Perfil público → Sus colecciones:** Desde `/perfil/:username` se puede navegar a sus listados públicos

## ProtectedRoute

- Si no hay usuario autenticado, redirige a `/login` con `replace`
- Si hay usuario autenticado, renderiza el componente hijo

## Navbar

- Muestra enlaces a todas las colecciones activas del usuario
- Si no hay usuario, muestra solo enlaces públicos (Home, Login, Register)
- Si hay usuario, muestra: Home, cada colección activa, Perfil, Preferencias, Logout

## Búsqueda Global (Ctrl+K / Cmd+K)

- Atajo global que abre `GlobalSearch`
- Busca en todos los endpoints de búsqueda externa (Google Books, RAWG, BGG, Scryfall, TMDB)
- Permite navegar directamente al detalle del elemento encontrado
