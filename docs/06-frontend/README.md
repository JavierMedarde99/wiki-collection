# Frontend

## Stack

- **Framework:** React 18+
- **Build:** Vite
- **Routing:** React Router v6
- **Estilos:** Tailwind CSS
- **Estado:** useState, useEffect, Context API
- **HTTP Client:** Fetch API

## Estructura de Carpetas

```
src/
├── components/        # Componentes reutilizables
│   ├── books/         # Componentes específicos de libros
│   ├── games/         # Componentes específicos de juegos
│   └── ui/            # Componentes UI genéricos
├── pages/             # Páginas/vistas
│   ├── BookListPage.jsx
│   ├── BookCreatePage.jsx
│   ├── BookEditPage.jsx
│   ├── BookSearchPage.jsx
│   └── ...
├── services/          # Servicios para llamadas API
│   ├── api.js         # Cliente HTTP base
│   ├── books.js       # Endpoints de libros
│   └── games.js       # Endpoints de juegos
├── constants/         # Constantes y enums
│   └── books.js
├── hooks/             # Custom hooks
├── context/           # Context providers
├── App.jsx            # Componente raíz con rutas
└── main.jsx           # Punto de entrada
```

## Páginas Implementadas

| Página | Ruta | Estado |
|--------|------|--------|
| Lista de libros | `/books` | ✅ |
| Crear libro | `/books/new` | ✅ |
| Editar libro | `/books/:id/edit` | ✅ |
| Buscar libros | `/books/search` | ✅ |
| Lista de juegos | `/games` | ⏳ |
| Crear juego | `/games/new` | ⏳ |
| Editar juego | `/games/:id/edit` | ⏳ |
| Buscar juegos | `/games/search` | ⏳ |
