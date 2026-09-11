# Requisitos

## Requisitos Funcionales

### Fase 1: Libros
- [x] Buscar libros en Google Books API
- [x] Añadir libros a la colección local
- [x] Listar libros con filtros por estado
- [x] Editar libros existentes
- [x] Eliminar libros de la colección
- [x] Valores de estado: TO_READ, READING, COMPLETED

### Fase 2: Videojuegos
- [x] Buscar juegos en RAWG API (principal) y FreeToGame (secundaria)
- [x] Añadir juegos a la colección local
- [x] Listar juegos con filtros
- [x] Editar juegos existentes
- [x] Eliminar juegos de la colección
- [x] Valores de estado: PLAYING, COMPLETED, WISHLIST, ABANDONED
- [x] Obtener logros de Steam para juegos vinculados

### Fase 3: Juegos de Mesa
- [x] Buscar juegos de mesa en BoardGameGeek (XML API)
- [x] Añadir juegos de mesa a la colección local
- [x] Listar juegos de mesa con filtros
- [x] Editar juegos de mesa existentes
- [x] Eliminar juegos de mesa de la colección
- [x] Valores de estado: OWNED, WISHLIST

### Fase 4: Cartas Magic
- [x] Buscar cartas en Scryfall API
- [x] Añadir cartas a la colección local
- [x] Listar cartas con filtros
- [x] Editar cartas existentes
- [x] Eliminar cartas de la colección
- [x] Gestión de condición, idioma, foil, cantidad

### Fase 4.1: Mazos Commander
- [x] Buscar comandantes en Scryfall API
- [x] Crear mazos con comandante, descripción y colores
- [x] Añadir cartas a mazos desde Scryfall
- [x] Quitar cartas de mazos
- [x] Validar estado del mazo (DRAFT, COMPLETE, INVALID)
- [x] Listar, editar y eliminar mazos

### Fase 5: Películas/Series
- [x] Buscar películas/series en TMDB API
- [x] Añadir películas/series a la colección local
- [x] Listar películas/series con filtros por estado y tipo
- [x] Editar películas/series existentes
- [x] Eliminar películas/series de la colección
- [x] Valores de estado: WATCHING, WATCHED, PLAN_TO_WATCH
- [x] Filtrar búsqueda externa por tipo (MOVIE, TV)

### Futuras Fases
- [ ] Autenticación de usuarios
- [ ] Perfiles de usuario

## Requisitos No Funcionales
- **Rendimiento:** Respuesta < 200ms para endpoints locales
- **Disponibilidad:** 99.5% uptime
- **Escalabilidad:** Soportar 1000+ items por colección
- **Seguridad:** API key para APIs externas, validación de inputs
- **Mantenibilidad:** Arquitectura hexagonal, tests > 80% cobertura (Jacoco)
- **UX:** Interfaz responsive, carga < 3s
