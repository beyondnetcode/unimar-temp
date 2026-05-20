# Analisis de Arquitectura Unificada - Extranet + Intranet

## 1. Current Tier/Component Mapping

### Tier 0 - External / Internet Edge
| Component | Extranet | Intranet | Shared |
|---|---|---|---|
| Usuario Externo (Clientes, Operadores) | YES | - | - |
| Usuario Interno (Staff, Admin) | - | YES | - |
| Usuario Movil | - | YES | - |
| Firewall / WAF perimetral | ASSUMED | ASSUMED | YES |

### Tier 1 - DMZ (Web Layer)
| Component | Extranet | Intranet | Shared |
|---|---|---|---|
| IIS Server (AppPool: ExtranetWeb) | YES | - | - |
| IIS Server (AppPool: SIS_INTRANET) | - | YES | - |
| Terminacion HTTPS | YES | YES | YES |
| ExtranetWeb (ASP.NET MVC 4 + Web API 4) | YES | - | - |
| SIS_INTRANET Web (ASP.NET MVC 4 + Web API 4) | - | YES | - |

### Tier 2 - Application / Service Layer
| Component | Extranet | Intranet | Shared |
|---|---|---|---|
| Extranet.Interfaces (Fachadas de negocio) | YES | - | - |
| Extranet.business (Reglas y orquestacion) | YES | - | - |
| Extranet.data (Acceso a datos via SP) | YES | - | - |
| INTRANET.Interfaces (Integraciones + Control accesos) | - | YES | - |
| INTRANET.business (Reglas de negocio y orquestacion) | - | YES | - |
| INTRANET.data (ADO.NET + Stored Procedures) | - | YES | - |
| INTRANET.objects (Entidades + DTOs) | - | YES | - |
| IntranetService (Proxy SOAP/WCF) | - | YES | - |
| PortalWebApi (Autenticacion centralizada) | YES | - | ASSUMED |
| API REST Externa (Servicios complementarios) | - | YES | - |

### Tier 3 - Integration Layer (External Services)
| Component | Extranet | Intranet | Shared |
|---|---|---|---|
| Servicios WCF SAP (ws_extranet/wcf.svc) | YES | YES | YES |
| Servicios WCF (wcfAgencia.svc) | YES | YES | YES |
| PortalWebApi (Auth + Menu) | YES | - | - |
| API REST Externa | - | YES | - |

### Tier 4 - Data Layer
| Component | Extranet | Intranet | Shared |
|---|---|---|---|
| SQL Server (instancia) | YES | YES | YES |
| BD_EXTRANET | YES (primary) | YES (read) | YES |
| BD_INTRANET | YES (read/write) | YES (primary) | YES |
| BD_BI | - | YES | YES |

---

## 2. Shared vs Exclusive Components

### Exclusively Extranet
- ExtranetWeb (MVC/WebAPI application)
- Extranet.Interfaces
- Extranet.business
- Extranet.data
- Domain: Deposito de vacios, reclamos, documentos operativos, visitas

### Exclusively Intranet
- SIS_INTRANET Web (MVC/WebAPI application)
- INTRANET.Interfaces
- INTRANET.business
- INTRANET.data
- INTRANET.objects
- IntranetService (SOAP/WCF proxy)
- Domain: Transmisiones aduaneras, movimientos logisticos, seguridad interna

### Shared
- SQL Server instance (3 databases)
- WCF Services (SAP integration, agency services)
- .NET Framework 4.5, MVC 4, Web API 4, ADO.NET stack
- IIS infrastructure (shared or separate servers)
- Cross-database access (BD_EXTRANET read by Intranet, BD_INTRANET read/write by Extranet)
- Master data tables (TM_UNI_Empresa, TM_UNI_Persona exist in both databases)

---

## 3. Communication Flows

### Primary Flows
```
External User -> Firewall/WAF -> IIS (DMZ) -> Web App -> Business Layer -> Data Layer -> SQL Server
Internal User -> Firewall/WAF -> IIS (DMZ) -> SIS_INTRANET -> Business/Interfaces -> IntranetService -> WCF External
Mobile User   -> Firewall/WAF -> IIS (DMZ) -> SIS_INTRANET -> Business/Interfaces -> WCF External
```

### Cross-System Flows
```
Extranet -> BD_INTRANET (read/write for shared operations)
Intranet -> BD_EXTRANET (read for master data)
Both -> WCF Services (SAP, agency services)
Extranet -> PortalWebApi (authentication)
Intranet -> API REST Externa (complementary services)
```

### Authentication Flows
```
Extranet: User -> ExtranetWeb -> Extranet.Interfaces.Seguridad -> PortalWebApi -> Cookies/Tokens
Intranet: User -> SIS_INTRANET -> Seguridad.obtenerDatosUsuario -> crearSesion -> Cookie
```

---

## 4. Gaps and Assumptions

### Confirmed Gaps
1. **No physical deployment diagram** - Server topology, load balancers, network configuration not documented
2. **No network architecture diagram** - VLANs, subnets, firewall rules between zones missing
3. **No CI/CD pipeline** - Deployment strategy, versioning, release process not shown
4. **No monitoring/logging infrastructure** - APM, alerting, log aggregation not documented
5. **No backup/DR strategy** - Database replication, disaster recovery not shown
6. **BLOBs in database** - VARBINARY storage causing over-fetching (ADR-0004)
7. **ConnectionStrings in plaintext** - Web.config without RSA encryption (ADR-0003)

### Key Assumptions
1. **IIS configuration** - Extranet and Intranet likely share IIS infrastructure with separate AppPools (web farms mentioned in ADR-0003)
2. **Firewall/WAF** - Inferred from HTTPS usage and Extranet internet accessibility
3. **PortalWebApi** - Only documented for Extranet; Intranet may use different auth mechanism or diagram is incomplete
4. **API REST Externa** - Only documented for Intranet; may be a real architectural difference or gap
5. **BD_BI** - Mentioned in Intranet context but usage details unknown; assumed for BI/reporting
6. **Cross-database access** - Both systems access each other's databases; exact table-level dependencies not mapped
7. **Session management** - Cookie-based auth, but session sharing between Extranet/Intranet not detailed

---

## 5. Consolidation Plan

### Phase 1: Documentation Consolidation (Immediate)
- [x] Create unified tier/deployment diagram (this deliverable)
- [ ] Map cross-database table dependencies explicitly
- [ ] Document all shared master data synchronization patterns
- [ ] Clarify PortalWebApi usage across both systems

### Phase 2: Security Hardening (Short-term)
- [ ] Encrypt ConnectionStrings using RSA Protected Configuration (ADR-0003)
- [ ] Externalize BLOBs from database to dedicated storage (ADR-0004)
- [ ] Implement consistent authentication mechanism across both systems
- [ ] Add HTTPS enforcement and TLS hardening on all endpoints

### Phase 3: Architecture Modernization (Medium-term)
- [ ] Separate Extranet and Intranet into distinct deployment units
- [ ] Implement API gateway for unified external access
- [ ] Create shared service layer for common business logic
- [ ] Establish database access patterns (eliminate cross-database direct access)

### Phase 4: Infrastructure Improvements (Long-term)
- [ ] Implement proper load balancing and web farm configuration
- [ ] Add monitoring, logging, and APM infrastructure
- [ ] Establish CI/CD pipeline with automated testing
- [ ] Design and implement backup/DR strategy

---

## 6. Unified Deployment Diagram

### PlantUML Version
File: `docs/diagramas/deployment_unificado.puml`

Visualize:
- Online: [PlantUML Editor](http://www.plantuml.com/plantuml/uml/)
- VS Code: Extension "PlantUML" (jebbs) -> Alt+D

### Mermaid Version
File: `docs/diagramas/deployment_unificado.mmd`

Visualize:
- Online: [Mermaid Live Editor](https://mermaid.live/)
- VS Code: Extension "Markdown Preview Mermaid Support"
- GitHub: Renders automatically in .md files

### Tier Structure
| Tier | Name | Key Components |
|------|------|----------------|
| 0 | External/Internet | Users (External, Internal, Mobile) |
| 1 | Perimeter/DMZ | Firewall/WAF, IIS Web Farm |
| 2 | Application | ExtranetWeb, SIS_INTRANET, Business/Data layers |
| 3 | Integration | WCF Services, PortalWebApi, REST APIs |
| 4 | Data | SQL Server (BD_EXTRANET, BD_INTRANET, BD_BI) |
