# Fase 1: Colección de Libros

## Objetivo
Permitir al usuario buscar libros vía API externa y gestionar su colección personal de libros.

## Pasos

### Paso 1: Búsqueda con API externa
- [ ] Configurar Google Books API Key
- [ ] Crear endpoint proxy `/api/books/search` en el backend
- [ ] Implementar fallback a Open Library si Google Books falla
- [ ] Crear componente de búsqueda en el frontend
- [ ] Mostrar resultados con portada, título, autor, año

### Paso 2: CRUD Backend
- [ ] Configurar proyecto Spring Boot con dependencias: Web, Data MongoDB, Validation
- [ ] Configurar conexión MongoDB en application.properties
- [ ] Crear documento Book con anotaciones Spring Data
- [ ] Crear repository BookRepository
- [ ] Crear service BookService con lógica de negocio
- [ ] Crear controller BookController con endpoints REST
- [ ] Añadir paginación y filtros
- [ ] Añadir validaciones con Bean Validation

### Paso 3: Frontend
- [ ] Configurar proyecto React con Vite
- [ ] Crear servicio de API (booksService)
- [ ] Página de búsqueda
- [ ] Página de mi colección (lista/grid)
- [ ] Página de detalle de libro
- [ ] Formulario para añadir/editar libros
- [ ] Filtros por estado, tags, ordenación

## Criterios de Aceptación
- [ ] Buscar "Cien años de soledad" devuelve resultados correctos
- [ ] Añadir un libro a la colección desde búsqueda
- [ ] Ver lista de libros filtrada por estado
- [ ] Editar valoración y notas de un libro
- [ ] Eliminar un libro de la colección
- [ ] La wiki en `docs/` está actualizada
