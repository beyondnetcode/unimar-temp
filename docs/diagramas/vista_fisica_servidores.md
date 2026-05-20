# Vista Fisica - Infraestructura de Servidores

## Diagrama

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

## Server Inventory

| Server | OS | Role | AppPool/Service |
|--------|----|------|----------------|
| IIS Server 1 | Windows Server 2016/2019 | Web Frontend | ExtranetWeb |
| IIS Server 2 | Windows Server 2016/2019 | Web Frontend | SIS_INTRANET |
| WCF SAP Server | Windows Server 2016 | Integration | ws_extranet/wcf.svc, wcfAgencia.svc |
| PortalWebApi Server | Windows/Linux | Auth Service | PortalWebApi |
| API REST Server | Windows/Linux | External Services | API REST Externa |
| SQL Server | Windows Server 2016/2019 | Database | SQL Server 2016/2019 |

## Network Zones

| Zone | Components | Access |
|------|------------|--------|
| External | Users | Public |
| Perimeter | WAF, Load Balancer | Port 443 |
| DMZ | IIS Servers | Port 80 (from LB) |
| Services | WCF, APIs | Restricted to DMZ |
| Data | SQL Server | Restricted to App Servers |
