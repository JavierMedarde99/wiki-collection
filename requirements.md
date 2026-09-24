# Requisitos — Wiki-Collection

## Requisitos Funcionales

### Fase 1: Libros ✅ Completada

- [x] Buscar libros en Google Books API
- [x] Añadir libros a la colección local
- [x] Listar libros con filtros por estado
- [x] Editar libros existentes
- [x] Eliminar libros de la colección
- [x] Valores de estado: TO_READ, READING, COMPLETED

### Fase 2: Videojuegos ✅ Completada

- [x] Buscar juegos en RAWG API (principal) y FreeToGame (secundaria)
- [x] Añadir juegos a la colección local
- [x] Listar juegos con filtros
- [x] Editar juegos existentes
- [x] Eliminar juegos de la colección
- [x] Valores de estado: PLAYING, COMPLETED, WISHLIST, ABANDONED
- [x] Obtener logros de Steam para juegos vinculados

### Fase 3: Juegos de Mesa ✅ Completada

- [x] Buscar juegos de mesa en BoardGameGeek (XML API)
- [x] Añadir juegos de mesa a la colección local
- [x] Listar juegos de mesa con filtros
- [x] Editar juegos de mesa existentes
- [x] Eliminar juegos de mesa de la colección
- [x] Valores de estado: OWNED, WISHLIST

### Fase 4: Cartas Magic ✅ Completada

- [x] Buscar cartas en Scryfall API
- [x] Añadir cartas a la colección local (desde Scryfall)
- [x] Listar cartas con filtros
- [x] Editar cartas existentes — **NO IMPLEMENTADO** (ver nota abajo)
- [x] Eliminar cartas de la colección
- [x] Gestión de condición, idioma, foil, cantidad

> **Nota:** El backend NO expone POST/PUT genéricos para Magic. Las cartas solo se pueden añadir desde Scryfall con `POST /scryfall/{scryfallId}`. MagicCardUseCase solo tiene: `search`, `findById`, `addFromScryfall`, `delete`.

### Fase 4.1: Mazos Commander ✅ Completada

- [x] Buscar comandantes en Scryfall API
- [x] Crear mazos con comandante, descripción y colores
- [x] Añadir cartas a mazos desde Scryfall
- [x] Quitar cartas de mazos
- [x] Validar estado del mazo (DRAFT, COMPLETE, INVALID)
- [x] Listar, editar y eliminar mazos

### Fase 5: Películas/Series ✅ Completada

- [x] Buscar películas/series en TMDB API
- [x] Añadir películas/series a la colección local
- [x] Listar películas/series con filtros por estado y tipo
- [x] Editar películas/series existentes
- [x] Eliminar películas/series de la colección
- [x] Valores de estado: WATCHING, WATCHED, PLAN_TO_WATCH
- [x] Filtrar búsqueda externa por tipo (MOVIE, TV)

### Fase 6: Imágenes ✅ Completada

- [x] Subir imágenes a Catbox.moe
- [x] Eliminar imágenes de Catbox.moe
- [x] Validación de tamaño (5MB) y MIME (jpeg, png, gif, webp)
- [x] Verificación de magic bytes

### Fase 7: Caché Caffeine ✅ Completada

- [x] Implementar caché Caffeine para búsquedas externas
- [x] 6 cachés: bookSearch, gameSearch, boardgameSearch, magicSearch, commanderSearch, movieSearch
- [x] TTL configurable, tamaño máximo 500 por caché

### Fase 8: Autenticación ✅ Completada

- [x] Registro de usuarios (username, email, password)
- [x] Login con JWT (access token 15min + refresh token 7días)
- [x] Refresh de access token
- [x] GET endpoints públicos (sin auth)
- [x] POST/PUT/DELETE requieren autenticación
- [x] Ownership: un usuario solo puede editar/eliminar sus propios elementos
- [x] Notas y valoraciones privadas (solo visibles por el dueño)
- [x] Perfiles públicos de usuarios
- [x] Admin por defecto configurable por env vars

### Fase 9: Colecciones Personales ✅ Completada

- [x] Activar/desactivar colecciones por usuario
- [x] Visibilidad de colecciones (PUBLIC / PRIVATE)
- [x] Listados con filtro `owner` (mine/other/all)
- [x] Perfil público de usuario con colecciones públicas
- [x] Preferencias creadas por defecto al registrarse
- [x] Users existing obtienen preferencias por defecto al arrancar

## Requisitos No Funcionales

- **Rendimiento:** Respuesta < 200ms para endpoints locales
- **Disponibilidad:** 99.5% uptime
- **Escalabilidad:** Soportar 1000+ items por colección
- **Seguridad:** API key para APIs externas, validación de inputs
- **Mantenibilidad:** Arquitectura hexagonal, tests > 80% cobertura (Jacoco)
- **UX:** Interfaz responsive, carga < 3s
