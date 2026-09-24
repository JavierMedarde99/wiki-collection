# Arquitectura General — Wiki-Collection

**stack:** Java 25 + Spring Boot 4.1.1 + MongoDB (backend) | React 18.3 + Vite 5 + TypeScript 7 + Tailwind CSS 3 (frontend)

## Patrón de Arquitectura

- **Backend:** Arquitectura hexagonal (puertos y adaptadores)
- **Frontend:** Componentes funcionales con hooks, servicios para llamadas API
- **Comunicación:** REST JSON entre frontend y backend

## Principios

1. **Separación de responsabilidades:** Cada capa tiene su propósito específico
2. **Inversión de dependencias:** Las dependencias apuntan hacia el dominio
3. **Testabilidad:** Cada componente se puede testear de forma aislada
4. **Independencia de frameworks:** El dominio no depende de Spring ni React

## Diagrama de Alto Nivel

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend (React)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Componentes │  │   Servicios  │  │  React Router       │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ HTTP/JSON
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Backend (Spring Boot)                     │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                   Infrastructure                         ││
│  │  ┌──────────┐  ┌──────────────┐  ┌──────────────────┐ ││
│  │  │Controller│  │Persistence   │  │External Clients  │ ││
│  │  └──────────┘  └──────────────┘  └──────────────────┘ ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │                   Application                            ││
│  │  ┌──────────────────────────────────────────────────┐  ││
│  │  │              Services (Use Cases)                 │  ││
│  │  └──────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │                     Domain                               ││
│  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌──────────────┐ ││
│  │  │ Models │  │ Ports  │  │Interfaces│  │ Value Objects│ ││
│  │  └────────┘  └────────┘  └────────┘  └──────────────┘ ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                              │
                              │ TCP/IP
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       MongoDB                                │
└─────────────────────────────────────────────────────────────┘
```

## Repositorios

| Repositorio | Lenguaje | Stack | Propósito |
|-------------|----------|-------|-----------|
| `backend-collection` | Java 25 | Spring Boot 4.1.1, MongoDB, Maven | API REST + lógica de negocio |
| `frontend-collection` | TypeScript 7 | React 18.3, Vite 5, Tailwind 3, React Router 6 | SPA + UX |
| `wiki-collection` | Markdown | Documentación | Wiki del proyecto |

## Estado del Proyecto

Ver `docs/00-home.md` para la tabla de estado por fases.
