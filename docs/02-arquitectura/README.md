# Arquitectura General

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
