# Solicitud de Información de Infraestructura

## Contexto

Se requiere complementar los diagramas de arquitectura unificada (Extranet + Intranet) con los nombres reales de servidores, direcciones IP y configuraciones de red para ambos entornos: **Producción** y **Ajustes/UAT**.

---

## Información Requerida por Tier

### Tier 1 - Perímetro / DMZ

| Componente | Dato Requerido | Producción | Ajustes/UAT |
|------------|---------------|------------|-------------|
| **Firewall / WAF** | Nombre del servidor / appliance | | |
| | Dirección IP | | |
| | Modelo / Versión | | |
| **Load Balancer** | Nombre del equipo | | |
| | Dirección IP | | |
| | Algoritmo de balanceo | | |

### Tier 2 - Servidores de Aplicación (IIS)

| Componente | Dato Requerido | Producción | Ajustes/UAT |
|------------|---------------|------------|-------------|
| **IIS Server 1** (AppPool: ExtranetWeb) | Nombre del servidor (hostname) | | |
| | Dirección IP | | |
| | Sistema operativo (versión) | | |
| | Versión de IIS | | |
| | Versión de .NET Framework | | |
| **IIS Server 2** (AppPool: SIS_INTRANET) | Nombre del servidor (hostname) | | |
| | Dirección IP | | |
| | Sistema operativo (versión) | | |
| | Versión de IIS | | |
| | Versión de .NET Framework | | |

### Tier 3 - Integración

| Componente | Dato Requerido | Producción | Ajustes/UAT |
|------------|---------------|------------|-------------|
| **Servidor WCF SAP** | Nombre del servidor (hostname) | | |
| | Dirección IP | | |
| | Endpoint URL | | |
| **PortalWebApi** | Nombre del servidor (hostname) | | |
| | Dirección IP | | |
| | Endpoint URL | | |
| **API REST Externa** | Proveedor / Sistema | | |
| | Endpoint URL | | |

### Tier 4 - Datos (SQL Server)

| Componente | Dato Requerido | Producción | Ajustes/UAT |
|------------|---------------|------------|-------------|
| **SQL Server** | Nombre del servidor (hostname) | | |
| | Dirección IP | | |
| | Versión de SQL Server | | |
| | Instancia (nombre) | | |
| **BD_EXTRANET** | Nombre de la base de datos | | |
| | Tamaño estimado | | |
| **BD_INTRANET** | Nombre de la base de datos | | |
| | Tamaño estimado | | |
| **BD_BI** | Nombre de la base de datos | | |
| | Tamaño estimado | | |

### Componentes Pendientes

| Componente | Dato Requerido | Producción | Ajustes/UAT |
|------------|---------------|------------|-------------|
| **File Server / Storage** (BLOBs) | Nombre del servidor o cuenta de almacenamiento | | |
| | Dirección IP o URL | | |
| | Tipo (NAS, SAN, Azure Blob, etc.) | | |
| **Monitoreo / Logging** | Sistema utilizado (ELK, AppInsights, etc.) | | |
| | Nombre del servidor / endpoint | | |
| | Dirección IP o URL | | |

---

## Información Adicional de Red

| Dato | Producción | Ajustes/UAT |
|------|------------|-------------|
| **VLANs configuradas** | | |
| **Subnets por zona** | | |
| **Reglas de Firewall entre tiers** | | |
| **Certificados SSL** (dominio, vigencia) | | |
| **Políticas de Backup** | | |
| **Estrategia de DR** | | |

---

## Resumen de Servidores a Identificar

| # | Tier | Componente | Hostname (Prod) | Hostname (UAT) |
|---|------|------------|-----------------|----------------|
| 1 | 1 | Firewall / WAF | | |
| 2 | 1 | Load Balancer | | |
| 3 | 2 | IIS Server 1 (ExtranetWeb) | | |
| 4 | 2 | IIS Server 2 (SIS_INTRANET) | | |
| 5 | 3 | Servidor WCF SAP | | |
| 6 | 3 | PortalWebApi | | |
| 7 | 4 | SQL Server | | |
| 8 | Pend. | File Server / Storage | | |
| 9 | Pend. | Monitoreo / Logging | | |
