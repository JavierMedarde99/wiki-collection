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
- [ ] Buscar juegos en FreeToGame API
- [ ] Añadir juegos a la colección local
- [ ] Listar juegos con filtros
- [ ] Editar juegos existentes
- [ ] Eliminar juegos de la colección
- [ ] Valores de estado: PLAYING, COMPLETED, WISHLIST, ABANDONED

### Futuras Fases
- [ ] Fase 3: Juegos de mesa
- [ ] Fase 4: Cartas Magic
- [ ] Fase 5: Películas/Series
- [ ] Autenticación de usuarios
- [ ] Perfiles de usuario

## Requisitos No Funcionales

- **Rendimiento:** Respuesta < 200ms para endpoints locales
- **Disponibilidad:** 99.5% uptime
- **Escalabilidad:** Soportar 1000+ items por colección
- **Seguridad:** API key para APIs externas, validación de inputs
- **Mantenibilidad:** Arquitectura hexagonal, tests > 80% cobertura
- **UX:** Interfaz responsive, carga < 3s
