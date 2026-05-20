# Diagramas de Arquitectura / Architecture Diagrams

<div align="center">

## Idioma / Language

[**Español**](#español) | [**English**](#english)

</div>

---

<a name="español"></a>

# Español

## Vista General

| Diagrama | Descripción | Formatos |
|----------|-------------|----------|
| [Análisis de Arquitectura Unificada](ANALISIS_ARQUITECTURA_UNIFICADA.md) | Mapeo de tiers, componentes compartidos/exclusivos, flujos de comunicación y plan de consolidación | Markdown |
| [Arquitectura Unificada](ARQUITECTURA_UNIFICADA.md) | Documentación de componentes Extranet + Intranet, flujos y zonas de seguridad | Markdown + PlantUML |

## Diagramas de Despliegue

| Diagrama | Descripción | Formatos |
|----------|-------------|----------|
| [Despliegue Unificado](deployment_unificado.puml) | Diagrama de despliegue consolidado por tiers (5 niveles) | PlantUML |
| [Despliegue Unificado](deployment_unificado.mmd) | Diagrama de despliegue consolidado por tiers (5 niveles) | Mermaid |
| [Vista Física de Servidores](vista_fisica_servidores.puml) | Topología física de servidores, zonas de red e infraestructura | PlantUML |
| [Vista Física de Servidores](vista_fisica_servidores.mmd) | Topología física de servidores, zonas de red e infraestructura | Mermaid |
| [Vista Física de Servidores](vista_fisica_servidores.md) | Topología física de servidores con inventario y zonas de red | Markdown + Mermaid |

## Diagramas por Sistema

### Extranet

| Diagrama | Descripción | Formato |
|----------|-------------|---------|
| [Contexto C4](extranet/c4/contexto.puml) | Vista de contexto del sistema Extranet | PlantUML |
| [Contenedores C4](extranet/c4/contenedores.puml) | Contenedores y componentes de Extranet | PlantUML |
| [Diagrama de Clases](extranet/uml/class_diagram.puml) | Capas y dependencias de Extranet | PlantUML |
| [Diagrama de Secuencia](extranet/uml/sequence_diagram.puml) | Flujo de autenticación | PlantUML |
| [Diagrama de Actividad](extranet/uml/activity_diagram.puml) | Flujo de solicitud HTTP | PlantUML |

### Intranet

| Diagrama | Descripción | Formato |
|----------|-------------|---------|
| [Contexto C4](intranet/c4/contexto.puml) | Vista de contexto del sistema Intranet | PlantUML |
| [Contenedores C4](intranet/c4/contenedores.puml) | Contenedores y componentes de Intranet | PlantUML |
| [Diagrama de Clases](intranet/uml/class_diagram_capas.puml) | Capas y dependencias de Intranet | PlantUML |
| [Diagrama de Secuencia](intranet/uml/sequence_diagram_auth.puml) | Flujo de autenticación | PlantUML |
| [Diagrama de Actividad](intranet/uml/activity_diagram_http.puml) | Flujo de solicitud HTTP | PlantUML |

## Modelos de Datos (E/R)

### Extranet

| Diagrama | Descripción | Formato |
|----------|-------------|---------|
| [E/R General](extranet/er/er_general.puml) | Modelo entidad-relación general | PlantUML |
| [Dominio: Transmisiones](extranet/er/dominio_transmisiones.puml) | Manifiestos, DAMs, Contenedores | PlantUML |
| [Dominio: Operativo/Comercial](extranet/er/dominio_operativo_comercial.puml) | Reservas, citas, horarios | PlantUML |
| [Dominio: Seguridad/Acceso](extranet/er/dominio_seguridad_acceso.puml) | Usuarios, áreas, gerencias | PlantUML |
| [Dominio: Reclamos](extranet/er/dominio_reclamos.puml) | Cabecera y documentos de reclamos | PlantUML |
| [Dominio: Visitas/Reservas](extranet/er/dominio_visitas_reservas.puml) | Reservas y estados de devolución | PlantUML |
| [Dominio: Depósito Vacíos](extranet/er/dominio_deposito_vacios.puml) | Solicitudes de vacíos | PlantUML |

### Intranet

| Diagrama | Descripción | Formato |
|----------|-------------|---------|
| [Dominio: Transmisiones](intranet/er/dominio_transmisiones.puml) | Manifiestos, DAMs, Contenedores | PlantUML |
| [Dominio: Operativo/Comercial](intranet/er/dominio_operativo_comercial.puml) | Reservas, citas, horarios | PlantUML |
| [Dominio: Seguridad/Acceso](intranet/er/dominio_seguridad_acceso.puml) | Usuarios, áreas, gerencias | PlantUML |
| [Dominio: Reclamos](intranet/er/dominio_reclamos.puml) | Cabecera y documentos de reclamos | PlantUML |
| [Dominio: Visitas/Reservas](intranet/er/dominio_visitas_reservas.puml) | Reservas y estados de devolución | PlantUML |
| [Dominio: Depósito Vacíos](intranet/er/dominio_deposito_vacios.puml) | Solicitudes de vacíos | PlantUML |

---

<a name="english"></a>

# English

## Overview

| Diagram | Description | Formats |
|---------|-------------|---------|
| [Unified Architecture Analysis](ANALISIS_ARQUITECTURA_UNIFICADA.md) | Tier mapping, shared/exclusive components, communication flows, consolidation plan | Markdown |
| [Unified Architecture](ARQUITECTURA_UNIFICADA.md) | Extranet + Intranet components documentation, flows, security zones | Markdown + PlantUML |

## Deployment Diagrams

| Diagram | Description | Formats |
|---------|-------------|---------|
| [Unified Deployment](deployment_unificado.puml) | Consolidated deployment diagram by tiers (5 levels) | PlantUML |
| [Unified Deployment](deployment_unificado.mmd) | Consolidated deployment diagram by tiers (5 levels) | Mermaid |
| [Physical Server View](vista_fisica_servidores.puml) | Physical server topology, network zones, infrastructure | PlantUML |
| [Physical Server View](vista_fisica_servidores.mmd) | Physical server topology, network zones, infrastructure | Mermaid |
| [Physical Server View](vista_fisica_servidores.md) | Physical server topology with inventory and network zones | Markdown + Mermaid |

## System Diagrams

### Extranet

| Diagram | Description | Format |
|---------|-------------|--------|
| [C4 Context](extranet/c4/contexto.puml) | Extranet system context view | PlantUML |
| [C4 Containers](extranet/c4/contenedores.puml) | Extranet containers and components | PlantUML |
| [Class Diagram](extranet/uml/class_diagram.puml) | Extranet layers and dependencies | PlantUML |
| [Sequence Diagram](extranet/uml/sequence_diagram.puml) | Authentication flow | PlantUML |
| [Activity Diagram](extranet/uml/activity_diagram.puml) | HTTP request flow | PlantUML |

### Intranet

| Diagram | Description | Format |
|---------|-------------|--------|
| [C4 Context](intranet/c4/contexto.puml) | Intranet system context view | PlantUML |
| [C4 Containers](intranet/c4/contenedores.puml) | Intranet containers and components | PlantUML |
| [Class Diagram](intranet/uml/class_diagram_capas.puml) | Intranet layers and dependencies | PlantUML |
| [Sequence Diagram](intranet/uml/sequence_diagram_auth.puml) | Authentication flow | PlantUML |
| [Activity Diagram](intranet/uml/activity_diagram_http.puml) | HTTP request flow | PlantUML |

## Data Models (E/R)

### Extranet

| Diagram | Description | Format |
|---------|-------------|--------|
| [General E/R](extranet/er/er_general.puml) | General entity-relationship model | PlantUML |
| [Domain: Transmissions](extranet/er/dominio_transmisiones.puml) | Manifests, DAMs, Containers | PlantUML |
| [Domain: Operations/Commercial](extranet/er/dominio_operativo_comercial.puml) | Reservations, appointments, schedules | PlantUML |
| [Domain: Security/Access](extranet/er/dominio_seguridad_acceso.puml) | Users, areas, departments | PlantUML |
| [Domain: Claims](extranet/er/dominio_reclamos.puml) | Claims header and documents | PlantUML |
| [Domain: Visits/Reservations](extranet/er/dominio_visitas_reservas.puml) | Reservations and return states | PlantUML |
| [Domain: Empty Deposits](extranet/er/dominio_deposito_vacios.puml) | Empty container requests | PlantUML |

### Intranet

| Diagram | Description | Format |
|---------|-------------|--------|
| [Domain: Transmissions](intranet/er/dominio_transmisiones.puml) | Manifests, DAMs, Containers | PlantUML |
| [Domain: Operations/Commercial](intranet/er/dominio_operativo_comercial.puml) | Reservations, appointments, schedules | PlantUML |
| [Domain: Security/Access](intranet/er/dominio_seguridad_acceso.puml) | Users, areas, departments | PlantUML |
| [Domain: Claims](intranet/er/dominio_reclamos.puml) | Claims header and documents | PlantUML |
| [Domain: Visits/Reservations](intranet/er/dominio_visitas_reservas.puml) | Reservations and return states | PlantUML |
| [Domain: Empty Deposits](intranet/er/dominio_deposito_vacios.puml) | Empty container requests | PlantUML |

---

## Cómo Visualizar los Diagramas / How to View Diagrams

### PlantUML

| Método / Method | Instrucción / Instructions |
|-----------------|---------------------------|
| **Online** | [PlantUML Editor](http://www.plantuml.com/plantuml/uml/) - Copiar y pegar el contenido |
| **VS Code** | Instalar extensión "PlantUML" (jebbs) → Abrir archivo → `Alt+D` |
| **CLI** | `plantuml archivo.puml` |

### Mermaid

| Método / Method | Instrucción / Instructions |
|-----------------|---------------------------|
| **Online** | [Mermaid Live Editor](https://mermaid.live/) - Copiar y pegar el contenido |
| **VS Code** | Extensión "Markdown Preview Mermaid Support" |
| **GitHub** | Se renderiza automáticamente en archivos `.md` |
