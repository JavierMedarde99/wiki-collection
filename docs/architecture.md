# Arquitectura

## Stack Tecnológico

| Capa      | Tecnología        | Motivo                          |
|-----------|-------------------|---------------------------------|
| Backend   | Java 25 + Spring Boot 4 | Robusto, tipado, estándar enterprise |
| Base de datos | MongoDB + Spring Data | Documentos flexibles, integración nativa |
| Frontend  | React + Vite      | Componentes, ecosistema amplio  |
| APIs externas | Google Books / Open Library | Gratuitas, sin auth para búsqueda |

## Patrón de Arquitectura

- **Backend:** MVC con separación de controllers, services, repositories (Spring Data)
- **Frontend:** Componentes funcionales con hooks, servicios para llamadas API
- **Comunicación:** REST JSON entre frontend y backend

## Estructura de Repositorios

```
wiki-collection/         Este repo — documentación
wiki-collection-backend/ Repo propio — Java 25 + Spring Boot 4
wiki-collection-frontend/ Repo propio — React + Vite
```

## Decisiones

1. **Java + Spring Boot sobre Node.js:** Tipado estático, mejor para proyectos que crecerán con múltiples entidades. Spring Data MongoDB proporciona integración nativa con MongoDB.
2. **MongoDB sobre SQL:** Los items de colección tienen atributos variables (un libro tiene autor, un juego tiene plataforma). MongoDB permite esquemas flexibles por entidad.
3. **Búsqueda externa + BD local:** La búsqueda de nuevos libros usa Google Books/Open Library. Los libros añadidos a la colección se guardan en BD local con datos enriquecidos por el usuario.
4. **Wiki en repo separada:** Toda la documentación vive en `docs/` para versionarla junto al código.
5. **Repositorios separados:** Backend y frontend tienen su propio pipeline de CI/CD, despliegue y ciclo de vida.
