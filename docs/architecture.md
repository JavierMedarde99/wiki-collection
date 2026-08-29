# Arquitectura

## Stack Tecnológico

| Capa      | Tecnología        | Motivo                          |
|-----------|-------------------|---------------------------------|
| Backend   | Node.js + Express | Ligero, rápido de prototipar    |
| Base de datos | MongoDB + Mongoose | Esquemas flexibles, JSON nativo |
| Frontend  | React + Vite      | Componentes, ecosistema amplio  |
| APIs externas | Google Books / Open Library | Gratuitas, sin auth para búsqueda |

## Patrón de Arquitectura

- **Backend:** MVC con separación de controllers, models y routes
- **Frontend:** Componentes funcionales con hooks, servicios para llamadas API
- **Comunicación:** REST JSON entre frontend y backend

## Decisiones

1. **MongoDB sobre SQL:** Los items de colección tienen atributos variables (un libro tiene autor, un juego tiene plataforma). MongoDB permite esquemas flexibles por entidad.
2. **Búsqueda externa + BD local:** La búsqueda de nuevos libros usa Google Books/Open Library. Los libros añadidos a la colección se guardan en BD local con datos enriquecidos por el usuario.
3. **Wiki en repo:** Toda la documentación vive en `docs/` para versionarla junto al código.
