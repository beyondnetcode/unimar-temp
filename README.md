# UNIMAR - Arquitectura Unificada y Diagramas

## Documentación de Arquitectura

| Documento | Descripción | Enlace |
|-----------|-------------|--------|
| **Arquitectura Unificada** | Componentes combinados Extranet + Intranet, flujos de comunicación y zonas de seguridad | [ARQUITECTURA_UNIFICADA.md](docs/diagramas/ARQUITECTURA_UNIFICADA.md) |
| **Análisis de Arquitectura Unificada** | Mapeo de tiers, componentes compartidos/exclusivos, plan de consolidación | [ANALISIS_ARQUITECTURA_UNIFICADA.md](docs/diagramas/ANALISIS_ARQUITECTURA_UNIFICADA.md) |

## Diagramas de Arquitectura

### Despliegue Unificado

Diagrama de despliegue consolidado por tiers (5 niveles):

```mermaid
flowchart TB
    UE["Usuario Externo\n(Clientes, Operadores)"]
    UI["Usuario Interno\n(Staff, Administradores)"]
    UM["Usuario Movil\n(Apps/Browser)"]

    WAF["Firewall / WAF\n(HTTPS Termination)"]
    
    IIS1["IIS Node 1\nAppPool: ExtranetWeb"]
    IIS2["IIS Node 2\nAppPool: SIS_INTRANET"]

    EW["ExtranetWeb\nMVC + Web API"]
    EI["Extranet.Interfaces"]
    EB["Extranet.business"]
    ED["Extranet.data"]

    IW["SIS_INTRANET Web\nMVC + Web API"]
    II["INTRANET.Interfaces"]
    IB["INTRANET.business"]
    ID["INTRANET.data"]
    IO["INTRANET.objects"]
    IS["IntranetService\nProxy SOAP/WCF"]

    WSAP["Servicios WCF SAP\nws_extranet/wcf.svc"]
    WAGE["Servicios WCF Agencia\nwcfAgencia.svc"]
    PAPI["PortalWebApi\nAuth centralizada"]
    AREST["API REST Externa"]

    SQL["SQL Server"]
    BDE["BD_EXTRANET"]
    BDI["BD_INTRANET"]
    BDBI["BD_BI"]

    UE -->|HTTPS| WAF
    UI -->|HTTPS| WAF
    UM -->|HTTPS| WAF

    WAF -->|HTTPS Terminado| IIS1
    WAF -->|HTTPS Terminado| IIS2

    IIS1 -->|Hosts| EW
    IIS2 -->|Hosts| IW

    EW --> EI
    EI --> EB
    EB --> ED

    IW --> IB
    IW --> II
    IW --> IO
    IW --> IS
    IB --> ID
    II --> IS
    ID --> IO

    EW -->|Auth| PAPI
    EI -->|Consume| WSAP
    IS -->|SOAP/WCF| WSAP
    IS -->|SOAP/WCF| WAGE
    II -->|HTTP/HTTPS| AREST

    ED -->|CRUD/SP| BDE
    ED -->|Cross-DB| BDI
    ID -->|ADO.NET/SP| BDI
    ID -->|Read Maestros| BDE

    SQL --- BDE
    SQL --- BDI
    SQL --- BDBI
    BDE <-->|Datos compartidos| BDI

    classDef extranet fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    classDef intranet fill:#FFF3E0,stroke:#F57C00,stroke-width:2px
    classDef external fill:#E8F5E9,stroke:#388E3C,stroke-width:2px
    classDef data fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px
    classDef security fill:#FFFDE7,stroke:#FBC02D,stroke-width:2px

    class EW,EI,EB,ED extranet
    class IW,II,IB,ID,IO,IS intranet
    class WSAP,WAGE,PAPI,AREST external
    class SQL,BDE,BDI,BDBI data
    class WAF security
```

| Tier | Nombre | Componentes |
|------|--------|-------------|
| 0 | Externa/Internet | Usuarios (Externos, Internos, Móviles) |
| 1 | Perímetro/DMZ | Firewall/WAF, Granja IIS |
| 2 | Aplicación | ExtranetWeb, SIS_INTRANET, Capas Business/Data |
| 3 | Integración | Servicios WCF, PortalWebApi, APIs REST |
| 4 | Datos | SQL Server (BD_EXTRANET, BD_INTRANET, BD_BI) |

**Archivos:** [PlantUML](docs/diagramas/deployment_unificado.puml) / [Mermaid](docs/diagramas/deployment_unificado.mmd)

---

### Vista Física de Servidores

Topología física de servidores, zonas de red e infraestructura:

```mermaid
flowchart TB
    UE["Usuario Externo"]
    UI["Usuario Interno"]
    UM["Usuario Movil"]

    WAF["Firewall / WAF"]
    LB["Load Balancer"]

    IIS1["IIS Server 1\nAppPool: ExtranetWeb"]
    IIS2["IIS Server 2\nAppPool: SIS_INTRANET"]

    WCF["Servidor WCF SAP"]
    PAPI["PortalWebApi"]
    AREST["API REST Externa"]

    SQL["SQL Server"]
    BDE["BD_EXTRANET"]
    BDI["BD_INTRANET"]
    BDBI["BD_BI"]

    BLOB["File Server / S3\n(Pendiente)"]
    ELK["Monitoreo\n(Pendiente)"]

    UE --> WAF
    UI --> WAF
    UM --> WAF

    WAF --> LB
    LB --> IIS1
    LB --> IIS2

    IIS1 --> PAPI
    IIS1 --> WCF
    IIS2 --> WCF
    IIS2 --> AREST

    IIS1 --> BDE
    IIS1 --> BDI
    IIS2 --> BDI
    IIS2 --> BDE

    SQL --- BDE
    SQL --- BDI
    SQL --- BDBI
    BDE <--> BDI

    BDE -.-> BLOB
    BDI -.-> BLOB

    IIS1 -.-> ELK
    IIS2 -.-> ELK
    SQL -.-> ELK
```

**Archivos:** [PlantUML](docs/diagramas/vista_fisica_servidores.puml) / [Mermaid](docs/diagramas/vista_fisica_servidores.mmd) / [Markdown](docs/diagramas/vista_fisica_servidores.md)

---

### Arquitectura Unificada

Diagrama de arquitectura consolidado (Extranet + Intranet):

**Archivos:** [PlantUML](docs/diagramas/arquitectura_unificada.puml)

---

## Cómo Visualizar los Diagramas

### PlantUML

| Método | Instrucciones |
|--------|--------------|
| **Online** | [Editor PlantUML](http://www.plantuml.com/plantuml/uml/) - Copiar y pegar contenido |
| **VS Code** | Instalar extensión "PlantUML" (jebbs) → Abrir archivo → `Alt+D` |
| **CLI** | `plantuml archivo.puml` |

### Mermaid

| Método | Instrucciones |
|--------|--------------|
| **Online** | [Editor Mermaid Live](https://mermaid.live/) - Copiar y pegar contenido |
| **VS Code** | Extensión "Markdown Preview Mermaid Support" |
| **GitHub** | Se renderiza automáticamente en archivos `.md` |
