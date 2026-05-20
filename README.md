# UNIMAR - Technical Audit & Modernization Repository

<div align="center">

## Idioma / Language

[**Español**](#español) | [**English**](#english)

</div>

---

<a name="english"></a>

# English

## Project Overview

This repository contains the complete technical audit documentation for UNIMAR's web ecosystem (**Intranet** and **Extranet** systems), including security assessments, architecture diagrams, Architecture Decision Records (ADRs), and a phased modernization roadmap.

**Systems Under Audit:**
- **Extranet** (`Extranet.sln`) - External-facing portal for clients and users (40%+ mobile traffic)
- **Intranet** (`SIS_INTRANET.sln`) - Internal corporate core system managing security, reservations, claims, and customs transmissions

**Technology Stack:** .NET Framework 4.5, ASP.NET MVC 4, Web API 4, IIS 2008, SQL Server

**Audit Date:** April 28, 2026

**Global Status:** 🔴 **CRITICAL RISK** - Severe vulnerabilities and operational blocker detected (IIS 2008 TLS crisis on mobile devices)

---

## Quick Start - Where to Begin

| Audience | Start Here | Reading Time |
|----------|-----------|--------------|
| **Executives / C-Level** | [Consolidated Audit - Executive Summary](docs/AUDITORIA_CONSOLIDADA_UNIMAR.md#1-resumen-ejecutivo-unificado) | 10 min |
| **Project Managers** | [Agile Work Plan](docs/PLAN_TRABAJO_AGILE.md) | 15 min |
| **Technical Leads / Architects** | [Extranet Technical Report](docs/auditoria_extranet/informe_tecnico.md) / [Intranet Technical Report](docs/auditoria_intranet/AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md) | 45-60 min |
| **Security Team** | [Consolidated OWASP Analysis](docs/ANALISIS_OWASP_CONSOLIDADO.md) | 20 min |
| **Infrastructure Team** | [Immediate Action - IIS 2008](docs/auditoria_extranet/ACCION_INMEDIATA_IIS2008.md) | 15 min |
| **Developers** | [ADRs Index](docs/adrs/) + [Library Inventory](docs/auditoria_extranet/05_INVENTARIO_LIBRERIAS.md) | 30 min |

---

## Critical Finding

### IIS 2008 TLS Mobile Crisis

**IIS 2008 fails TLS negotiation with modern mobile devices, causing server crashes.**

| Aspect | Detail |
|--------|--------|
| **Impact** | 40%+ users (mobile) cannot access the system |
| **Status** | 🔴 BLOCKER - Immediate Go-Live prevention |
| **Solution** | Migrate to IIS 10/13 on Windows Server 2016/2022 |
| **Timeline** | 7-10 days for migration + validation |
| **Immediate Action** | Deploy code mitigations (Phase 0A) today |

**References:**
- [ADR-0001: IIS 2008 TLS Mobile Crisis](docs/auditoria_extranet/adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md)
- [Immediate Action Checklist](docs/auditoria_extranet/ACCION_INMEDIATA_IIS2008.md)

---

## Documentation Index

### Consolidated Audit

| Document | Description | Link |
|----------|-------------|------|
| **Consolidated Audit** | Unified findings from Intranet + Extranet audits, executive summary, global risks, strategic roadmap | [AUDITORIA_CONSOLIDADA_UNIMAR.md](docs/AUDITORIA_CONSOLIDADA_UNIMAR.md) |
| **OWASP Security Analysis** | Mapping of all findings against OWASP Top 10 (2021), risk assessment, remediation priorities | [ANALISIS_OWASP_CONSOLIDADO.md](docs/ANALISIS_OWASP_CONSOLIDADO.md) |
| **Agile Work Plan** | Scrum squads, sprint planning, Gantt chart, effort estimation, phased timeline | [PLAN_TRABAJO_AGILE.md](docs/PLAN_TRABAJO_AGILE.md) |

### Extranet Audit

| Document | Description | Link |
|----------|-------------|------|
| **Index & Executive Summary** | Go/No-Go checklist, traffic light dashboard, master timeline, role-based guide | [INDEX.md](docs/auditoria_extranet/INDEX.md) |
| **Executive Summary** | Decision recommendation, 7 blockers, phased roadmap | [resumen_ejecutivo_indice.md](docs/auditoria_extranet/resumen_ejecutivo_indice.md) |
| **Technical Report** | Full inventory (5 .NET projects), findings H-00 to H-08, criticality table, Go/No-Go evaluation | [informe_tecnico.md](docs/auditoria_extranet/informe_tecnico.md) |
| **Immediate Action (IIS 2008)** | Phase 0A (Dev team, 4-6h) + Phase 0B (Infra team, 7-10 days) checklists | [ACCION_INMEDIATA_IIS2008.md](docs/auditoria_extranet/ACCION_INMEDIATA_IIS2008.md) |
| **Visual Executive Summary** | Architecture panorama, risk matrix, remediation timeline, Go-Live checklist | [resumen_visual_ejecutivo.md](docs/auditoria_extranet/resumen_visual_ejecutivo.md) |
| **Diagram Annex** | UML, C4, E/R diagrams with data dictionary | [anexo_diagramas.md](docs/auditoria_extranet/anexo_diagramas.md) |
| **Library Inventory** | 20+ dependencies, update waves, risk vs effort matrix, CI automation script | [05_INVENTARIO_LIBRERIAS.md](docs/auditoria_extranet/05_INVENTARIO_LIBRERIAS.md) |
| **Visualization Guide** | How to view PlantUML diagrams (online, VS Code, CLI) | [README_VISUALIZACION.md](docs/auditoria_extranet/README_VISUALIZACION.md) |
| **What's New** | Latest updates and changes | [WHATS_NEW.md](docs/auditoria_extranet/WHATS_NEW.md) |

### Intranet Audit

| Document | Description | Link |
|----------|-------------|------|
| **Executive Summary & Index** | Go-Live ASAP approach, progressive optimization plan, documentation index | [RESUMEN_EJECUTIVO_E_INDICE.md](docs/auditoria_intranet/RESUMEN_EJECUTIVO_E_INDICE.md) |
| **Full Technical Report** | Complete audit of SIS_INTRANET solution, findings, criticality table, data analysis | [AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md](docs/auditoria_intranet/AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md) |
| **Architecture Diagrams Annex (PlantUML)** | UML, C4, E/R diagrams in individual `.puml` files ready for IDE | [ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md](docs/auditoria_intranet/ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md) |
| **Architecture Diagrams Annex (Mermaid)** | Same diagrams in Mermaid format | [ANEXO_DIAGRAMAS_ARQUITECTURA_MERMAID.md](docs/auditoria_intranet/ANEXO_DIAGRAMAS_ARQUITECTURA_MERMAID.md) |
| **Visual Executive Summary (PlantUML)** | Executive-level visual summary with PlantUML diagrams | [RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md](docs/auditoria_intranet/RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md) |
| **Visual Executive Summary (Mermaid)** | Executive-level visual summary with Mermaid diagrams | [RESUMEN_VISUAL_EJECUTIVO_MERMAID.md](docs/auditoria_intranet/RESUMEN_VISUAL_EJECUTIVO_MERMAID.md) |
| **Library Inventory** | External library inventory and update plan | [05_INVENTARIO_LIBRERIAS.md](docs/auditoria_intranet/05_INVENTARIO_LIBRERIAS.md) |
| **PlantUML Visualization Instructions** | How to visualize PlantUML diagrams | [INSTRUCCIONES_VISUALIZAR_PLANTUML.md](docs/auditoria_intranet/INSTRUCCIONES_VISUALIZAR_PLANTUML.md) |

### Architecture Decision Records (ADRs)

| ADR | Title | Severity | Link |
|-----|-------|----------|------|
| **ADR-0001** | IIS 2008 → IIS 10/13 Migration for TLS Mobile Crisis | 🔴 P0 - Go-Live Blocker | [Extranet ADR](docs/auditoria_extranet/adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md) |
| **ADR-0002** | Safe Framework & Library Upgrade Strategy (Max Compatibility) | Technical Debt / Security | [ADR-0002](docs/adrs/ADR_0002_Upgrade_Path_Compatibility.md) |
| **ADR-0003** | Secret Protection & Connection String Encryption | 🔴 Critical Security | [ADR-0003](docs/adrs/ADR_0003_Proteccion_Secretos_WebConfig.md) |
| **ADR-0004** | BLOB Externalization & Data Over-fetching Mitigation | High Performance / Scalability | [ADR-0004](docs/adrs/ADR_0004_Externalizacion_Blobs_Optimizacion_Datos.md) |

**ADR Indexes:**
- [Consolidated ADRs Index](docs/adrs/)
- [Extranet ADRs Index](docs/auditoria_extranet/adrs/README.md)
- [Intranet ADRs Index](docs/auditoria_intranet/adrs/README.md)

### Architecture Diagrams

Full diagram repository with PlantUML and Mermaid formats:

| Category | Description | Link |
|----------|-------------|------|
| **Unified Architecture** | Combined Extranet + Intranet component documentation, flows, security zones | [ARQUITECTURA_UNIFICADA.md](docs/diagramas/ARQUITECTURA_UNIFICADA.md) |
| **Unified Architecture Analysis** | Tier mapping, shared/exclusive components, consolidation plan | [ANALISIS_ARQUITECTURA_UNIFICADA.md](docs/diagramas/ANALISIS_ARQUITECTURA_UNIFICADA.md) |
| **Deployment Diagrams** | 5-tier consolidated deployment, physical server topology | [Deployment (PlantUML)](docs/diagramas/deployment_unificado.puml) / [Deployment (Mermaid)](docs/diagramas/deployment_unificado.mmd) |
| **Physical Server View** | Server topology, network zones, inventory | [Server View (PlantUML)](docs/diagramas/vista_fisica_servidores.puml) / [Server View (Mermaid)](docs/diagramas/vista_fisica_servidores.mmd) / [Server View (Markdown)](docs/diagramas/vista_fisica_servidores.md) |
| **Extranet Diagrams** | C4 Context, C4 Containers, Class, Sequence, Activity diagrams | [extranet/](docs/diagramas/extranet/) |
| **Intranet Diagrams** | C4 Context, C4 Containers, Class, Sequence, Activity diagrams | [intranet/](docs/diagramas/intranet/) |
| **Entity-Relationship Models** | General + 6 domain E/R diagrams per system | [Extranet ER](docs/diagramas/extranet/er/) / [Intranet ER](docs/diagramas/intranet/er/) |
| **Diagram Index** | Complete catalog of all diagrams with descriptions | [README.md](docs/diagramas/README.md) |

---

## Executive Summary

### Current State

Both UNIMAR web platforms (Intranet and Extranet) operate on **end-of-life technology** (IIS 2008, .NET Framework 4.5, ASP.NET MVC 4) with multiple severe vulnerabilities. The Extranet is already experiencing **blocking operational instability** due to TLS incompatibility with mobile devices.

### Key Findings

| # | Finding | Severity | Systems |
|---|---------|----------|---------|
| 1 | IIS 2008 TLS negotiation failure with mobile devices | 🔴 Critical | Extranet |
| 2 | Database credentials exposed in plaintext (Web.config) | 🔴 Critical | Both |
| 3 | Unencrypted HTTP traffic on sensitive endpoints | 🔴 Critical | Both |
| 4 | No CSRF protection on POST operations | 🟠 High | Both |
| 5 | Weak, non-centralized authorization | 🟠 High | Both |
| 6 | Insecure production configurations (debug=true) | 🟠 High | Both |
| 7 | Data over-fetching and wide entities | 🟠 High | Intranet |
| 8 | Silent error handling (empty catch blocks) | 🟡 Medium | Both |
| 9 | Legacy technical debt (2012-era libraries) | 🟡 Medium | Both |
| 10 | Session cookies without security flags | 🟡 Medium | Both |

### Recommended Decision

**Conditional Go-Live (Phase 0):** Execute production launch assuming controlled technical debt, resolving only critical blockers within 10-14 days. Then execute structured phases of stabilization and hardening.

---

## Remediation Roadmap

| Phase | Description | Timeline | Responsible |
|-------|-------------|----------|-------------|
| **0A** | Code mitigations (TLS) + secret rotation | 48 hours | Dev Team |
| **0B** | IIS 2008 → IIS 10/13 migration | 7-10 days | Infra Team |
| **1** | Post-launch stabilization (logging, timeouts) | Month 1 | Dev Team |
| **2** | Security hardening (CSRF, cookies, auth) | Months 2-3 | Dev Team |
| **3** | Database optimization (BLOBs, DTOs) | Months 4-6 | Dev Team |
| **4** | Architecture modernization (.NET 8+, REST, CI/CD) | Months 6+ | Modernization Squad |

---

## Repository Structure

```
unimar-temp/
├── README.md                              ← This file
├── LICENSE
├── .gitignore
│
└── docs/
    ├── AUDITORIA_CONSOLIDADA_UNIMAR.md     ← Consolidated audit (start here)
    ├── ANALISIS_OWASP_CONSOLIDADO.md       ← OWASP Top 10 security analysis
    ├── PLAN_TRABAJO_AGILE.md               ← Agile work plan with Gantt chart
    │
    ├── adrs/                               ← Architecture Decision Records
    │   ├── ADR_0002_Upgrade_Path_Compatibility.md
    │   ├── ADR_0003_Proteccion_Secretos_WebConfig.md
    │   └── ADR_0004_Externalizacion_Blobs_Optimizacion_Datos.md
    │
    ├── auditoria_extranet/                 ← Extranet audit deliverables
    │   ├── INDEX.md                        ← Extranet complete index
    │   ├── informe_tecnico.md              ← Technical report
    │   ├── ACCION_INMEDIATA_IIS2008.md     ← Immediate action checklist
    │   ├── anexo_diagramas.md              ← Diagram annex
    │   ├── 05_INVENTARIO_LIBRERIAS.md      ← Library inventory
    │   ├── adrs/                           ← Extranet-specific ADRs
    │   └── ...
    │
    ├── auditoria_intranet/                 ← Intranet audit deliverables
    │   ├── RESUMEN_EJECUTIVO_E_INDICE.md   ← Executive summary & index
    │   ├── AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md  ← Technical report
    │   ├── ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md
    │   ├── 05_INVENTARIO_LIBRERIAS.md      ← Library inventory
    │   ├── adrs/                           ← Intranet-specific ADRs
    │   └── ...
    │
    └── diagramas/                          ← Architecture diagrams
        ├── README.md                       ← Diagram index (bilingual)
        ├── ARQUITECTURA_UNIFICADA.md       ← Unified architecture
        ├── ANALISIS_ARQUITECTURA_UNIFICADA.md
        ├── deployment_unificado.puml/.mmd
        ├── vista_fisica_servidores.puml/.mmd/.md
        ├── extranet/                       ← Extranet diagrams (C4, UML, ER)
        └── intranet/                       ← Intranet diagrams (C4, UML, ER)
```

---

## How to View Diagrams

### PlantUML

| Method | Instructions |
|--------|-------------|
| **Online** | [PlantUML Editor](http://www.plantuml.com/plantuml/uml/) - Copy and paste content |
| **VS Code** | Install "PlantUML" extension (jebbs) → Open file → `Alt+D` |
| **CLI** | `plantuml file.puml` |

### Mermaid

| Method | Instructions |
|--------|-------------|
| **Online** | [Mermaid Live Editor](https://mermaid.live/) - Copy and paste content |
| **VS Code** | Extension "Markdown Preview Mermaid Support" |
| **GitHub** | Renders automatically in `.md` files |

---

## External References

| Resource | Link |
|----------|------|
| TLS Management (Microsoft) | [docs.microsoft.com/.../manage-tls](https://docs.microsoft.com/en-us/windows-server/security/tls/manage-tls) |
| HTTP/2 in IIS 10+ | [docs.microsoft.com/.../http2-on-iis](https://docs.microsoft.com/en-us/iis/get-started/whats-new-in-iis-10/http2-on-iis) |
| SSL Labs Test | [ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/) |
| OWASP Top 10 (2021) | [owasp.org/Top10/](https://owasp.org/Top10/) |
| Azure Blob Storage | [learn.microsoft.com/.../storage-blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/) |
| RSA Protected Configuration | [learn.microsoft.com/.../dtkwfdky](https://learn.microsoft.com/en-us/previous-versions/aspnet/dtkwfdky(v=vs.100)) |

---

## Change Control

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2026-05-20 | Initial README with complete documentation index | Technical Audit Team |

---

<a name="español"></a>

# Español

## Descripción del Proyecto

Este repositorio contiene la documentación completa de la auditoría técnica del ecosistema web de UNIMAR (sistemas **Intranet** y **Extranet**), incluyendo evaluaciones de seguridad, diagramas de arquitectura, Architecture Decision Records (ADRs) y una hoja de ruta de modernización por fases.

**Sistemas Auditados:**
- **Extranet** (`Extranet.sln`) - Portal orientado a clientes y usuarios externos (40%+ tráfico móvil)
- **Intranet** (`SIS_INTRANET.sln`) - Sistema core corporativo de uso interno que gestiona seguridad, reservas, reclamos y transmisiones aduaneras

**Stack Tecnológico:** .NET Framework 4.5, ASP.NET MVC 4, Web API 4, IIS 2008, SQL Server

**Fecha de Auditoría:** 28 de abril de 2026

**Estado Global:** 🔴 **RIESGO CRÍTICO** - Vulnerabilidades severas y bloqueo operativo detectado (crisis TLS de IIS 2008 en dispositivos móviles)

---

## Inicio Rápido - Por Dónde Empezar

| Audiencia | Comenzar Aquí | Tiempo de Lectura |
|-----------|--------------|-------------------|
| **Ejecutivos / C-Level** | [Auditoría Consolidada - Resumen Ejecutivo](docs/AUDITORIA_CONSOLIDADA_UNIMAR.md#1-resumen-ejecutivo-unificado) | 10 min |
| **Project Managers** | [Plan de Trabajo Agile](docs/PLAN_TRABAJO_AGILE.md) | 15 min |
| **Líderes Técnicos / Arquitectos** | [Informe Técnico Extranet](docs/auditoria_extranet/informe_tecnico.md) / [Informe Técnico Intranet](docs/auditoria_intranet/AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md) | 45-60 min |
| **Equipo de Seguridad** | [Análisis OWASP Consolidado](docs/ANALISIS_OWASP_CONSOLIDADO.md) | 20 min |
| **Equipo de Infraestructura** | [Acción Inmediata - IIS 2008](docs/auditoria_extranet/ACCION_INMEDIATA_IIS2008.md) | 15 min |
| **Desarrolladores** | [Índice de ADRs](docs/adrs/) + [Inventario de Librerías](docs/auditoria_extranet/05_INVENTARIO_LIBRERIAS.md) | 30 min |

---

## Hallazgo Crítico

### Crisis TLS Móvil de IIS 2008

**IIS 2008 falla la negociación TLS con dispositivos móviles modernos, causando caídas del servidor.**

| Aspecto | Detalle |
|---------|---------|
| **Impacto** | 40%+ usuarios (móvil) no pueden acceder al sistema |
| **Estado** | 🔴 BLOQUEANTE - Prevención inmediata de Go-Live |
| **Solución** | Migrar a IIS 10/13 en Windows Server 2016/2022 |
| **Plazo** | 7-10 días para migración + validación |
| **Acción Inmediata** | Desplegar mitigaciones de código (Fase 0A) hoy |

**Referencias:**
- [ADR-0001: Crisis TLS Móvil IIS 2008](docs/auditoria_extranet/adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md)
- [Checklist de Acción Inmediata](docs/auditoria_extranet/ACCION_INMEDIATA_IIS2008.md)

---

## Índice de Documentación

### Auditoría Consolidada

| Documento | Descripción | Enlace |
|-----------|-------------|--------|
| **Auditoría Consolidada** | Hallazgos unificados de auditorías Intranet + Extranet, resumen ejecutivo, riesgos globales, hoja de ruta estratégica | [AUDITORIA_CONSOLIDADA_UNIMAR.md](docs/AUDITORIA_CONSOLIDADA_UNIMAR.md) |
| **Análisis de Seguridad OWASP** | Mapeo de hallazgos contra OWASP Top 10 (2021), evaluación de riesgos, prioridades de remediación | [ANALISIS_OWASP_CONSOLIDADO.md](docs/ANALISIS_OWASP_CONSOLIDADO.md) |
| **Plan de Trabajo Agile** | Squads Scrum, planificación de sprints, diagrama Gantt, estimación de esfuerzo, cronograma por fases | [PLAN_TRABAJO_AGILE.md](docs/PLAN_TRABAJO_AGILE.md) |

### Auditoría Extranet

| Documento | Descripción | Enlace |
|-----------|-------------|--------|
| **Índice y Resumen Ejecutivo** | Checklist Go/No-Go, semáforo de riesgos, cronograma maestro, guía por roles | [INDEX.md](docs/auditoria_extranet/INDEX.md) |
| **Resumen Ejecutivo** | Recomendación de decisión, 7 bloqueantes, hoja de ruta por fases | [resumen_ejecutivo_indice.md](docs/auditoria_extranet/resumen_ejecutivo_indice.md) |
| **Informe Técnico** | Inventario completo (5 proyectos .NET), hallazgos H-00 a H-08, tabla de criticidad, evaluación Go/No-Go | [informe_tecnico.md](docs/auditoria_extranet/informe_tecnico.md) |
| **Acción Inmediata (IIS 2008)** | Fase 0A (equipo Dev, 4-6h) + Fase 0B (equipo Infra, 7-10 días) checklists | [ACCION_INMEDIATA_IIS2008.md](docs/auditoria_extranet/ACCION_INMEDIATA_IIS2008.md) |
| **Resumen Visual Ejecutivo** | Panorama de arquitectura, matriz de riesgos, cronograma de remediación, checklist Go-Live | [resumen_visual_ejecutivo.md](docs/auditoria_extranet/resumen_visual_ejecutivo.md) |
| **Anexo de Diagramas** | Diagramas UML, C4, E/R con diccionario de datos | [anexo_diagramas.md](docs/auditoria_extranet/anexo_diagramas.md) |
| **Inventario de Librerías** | 20+ dependencias, oleadas de actualización, matriz riesgo vs esfuerzo, script automatización CI | [05_INVENTARIO_LIBRERIAS.md](docs/auditoria_extranet/05_INVENTARIO_LIBRERIAS.md) |
| **Guía de Visualización** | Cómo visualizar diagramas PlantUML (online, VS Code, CLI) | [README_VISUALIZACION.md](docs/auditoria_extranet/README_VISUALIZACION.md) |
| **Novedades** | Últimas actualizaciones y cambios | [WHATS_NEW.md](docs/auditoria_extranet/WHATS_NEW.md) |

### Auditoría Intranet

| Documento | Descripción | Enlace |
|-----------|-------------|--------|
| **Resumen Ejecutivo e Índice** | Enfoque Go-Live ASAP, plan de optimización progresiva, índice de documentación | [RESUMEN_EJECUTIVO_E_INDICE.md](docs/auditoria_intranet/RESUMEN_EJECUTIVO_E_INDICE.md) |
| **Informe Técnico Completo** | Auditoría completa de SIS_INTRANET, hallazgos, tabla de criticidad, análisis de datos | [AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md](docs/auditoria_intranet/AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md) |
| **Anexo Diagramas Arquitectura (PlantUML)** | Diagramas UML, C4, E/R en archivos `.puml` individuales listos para IDE | [ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md](docs/auditoria_intranet/ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md) |
| **Anexo Diagramas Arquitectura (Mermaid)** | Mismos diagramas en formato Mermaid | [ANEXO_DIAGRAMAS_ARQUITECTURA_MERMAID.md](docs/auditoria_intranet/ANEXO_DIAGRAMAS_ARQUITECTURA_MERMAID.md) |
| **Resumen Visual Ejecutivo (PlantUML)** | Resumen visual a nivel ejecutivo con diagramas PlantUML | [RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md](docs/auditoria_intranet/RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md) |
| **Resumen Visual Ejecutivo (Mermaid)** | Resumen visual a nivel ejecutivo con diagramas Mermaid | [RESUMEN_VISUAL_EJECUTIVO_MERMAID.md](docs/auditoria_intranet/RESUMEN_VISUAL_EJECUTIVO_MERMAID.md) |
| **Inventario de Librerías** | Inventario de librerías externas y plan de actualización | [05_INVENTARIO_LIBRERIAS.md](docs/auditoria_intranet/05_INVENTARIO_LIBRERIAS.md) |
| **Instrucciones Visualización PlantUML** | Cómo visualizar diagramas PlantUML | [INSTRUCCIONES_VISUALIZAR_PLANTUML.md](docs/auditoria_intranet/INSTRUCCIONES_VISUALIZAR_PLANTUML.md) |

### Architecture Decision Records (ADRs)

| ADR | Título | Severidad | Enlace |
|-----|--------|-----------|--------|
| **ADR-0001** | Migración IIS 2008 → IIS 10/13 para Crisis TLS Móvil | 🔴 P0 - Bloqueante Go-Live | [ADR Extranet](docs/auditoria_extranet/adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md) |
| **ADR-0002** | Estrategia de Actualización Segura de Framework y Librerías | Deuda Técnica / Seguridad | [ADR-0002](docs/adrs/ADR_0002_Upgrade_Path_Compatibility.md) |
| **ADR-0003** | Protección de Secretos y Cifrado de Cadenas de Conexión | 🔴 Seguridad Crítica | [ADR-0003](docs/adrs/ADR_0003_Proteccion_Secretos_WebConfig.md) |
| **ADR-0004** | Externalización de BLOBs y Mitigación de Over-fetching | Alto Rendimiento / Escalabilidad | [ADR-0004](docs/adrs/ADR_0004_Externalizacion_Blobs_Optimizacion_Datos.md) |

**Índices de ADRs:**
- [Índice ADRs Consolidados](docs/adrs/)
- [Índice ADRs Extranet](docs/auditoria_extranet/adrs/README.md)
- [Índice ADRs Intranet](docs/auditoria_intranet/adrs/README.md)

### Diagramas de Arquitectura

Repositorio completo de diagramas en formatos PlantUML y Mermaid:

| Categoría | Descripción | Enlace |
|-----------|-------------|--------|
| **Arquitectura Unificada** | Documentación combinada de componentes Extranet + Intranet, flujos, zonas de seguridad | [ARQUITECTURA_UNIFICADA.md](docs/diagramas/ARQUITECTURA_UNIFICADA.md) |
| **Análisis Arquitectura Unificada** | Mapeo de tiers, componentes compartidos/exclusivos, plan de consolidación | [ANALISIS_ARQUITECTURA_UNIFICADA.md](docs/diagramas/ANALISIS_ARQUITECTURA_UNIFICADA.md) |
| **Diagramas de Despliegue** | Despliegue consolidado de 5 niveles, topología física de servidores | [Despliegue (PlantUML)](docs/diagramas/deployment_unificado.puml) / [Despliegue (Mermaid)](docs/diagramas/deployment_unificado.mmd) |
| **Vista Física de Servidores** | Topología de servidores, zonas de red, inventario | [Vista Servidores (PlantUML)](docs/diagramas/vista_fisica_servidores.puml) / [Vista Servidores (Mermaid)](docs/diagramas/vista_fisica_servidores.mmd) / [Vista Servidores (Markdown)](docs/diagramas/vista_fisica_servidores.md) |
| **Diagramas Extranet** | Contexto C4, Contenedores C4, Clases, Secuencia, Actividad | [extranet/](docs/diagramas/extranet/) |
| **Diagramas Intranet** | Contexto C4, Contenedores C4, Clases, Secuencia, Actividad | [intranet/](docs/diagramas/intranet/) |
| **Modelos Entidad-Relación** | Modelo general + 6 diagramas E/R por dominio y sistema | [ER Extranet](docs/diagramas/extranet/er/) / [ER Intranet](docs/diagramas/intranet/er/) |
| **Índice de Diagramas** | Catálogo completo de todos los diagramas con descripciones | [README.md](docs/diagramas/README.md) |

---

## Resumen Ejecutivo

### Estado Actual

Ambas plataformas web de UNIMAR (Intranet y Extranet) operan sobre **tecnología obsoleta** (IIS 2008, .NET Framework 4.5, ASP.NET MVC 4) con múltiples vulnerabilidades severas. La Extranet ya experimenta **inestabilidad operativa bloqueante** por incompatibilidad TLS con dispositivos móviles.

### Hallazgos Principales

| # | Hallazgo | Severidad | Sistemas |
|---|----------|-----------|-----------|
| 1 | Fallo de negociación TLS de IIS 2008 con dispositivos móviles | 🔴 Crítico | Extranet |
| 2 | Credenciales de BD expuestas en texto plano (Web.config) | 🔴 Crítico | Ambos |
| 3 | Tráfico HTTP sin cifrar en endpoints sensibles | 🔴 Crítico | Ambos |
| 4 | Sin protección CSRF en operaciones POST | 🟠 Alto | Ambos |
| 5 | Autorización débil y no centralizada | 🟠 Alto | Ambos |
| 6 | Configuraciones inseguras de producción (debug=true) | 🟠 Alto | Ambos |
| 7 | Over-fetching de datos y entidades anchas | 🟠 Alto | Intranet |
| 8 | Manejo silencioso de errores (bloques catch vacíos) | 🟡 Medio | Ambos |
| 9 | Deuda técnica legacy (librerías de 2012) | 🟡 Medio | Ambos |
| 10 | Cookies de sesión sin banderas de seguridad | 🟡 Medio | Ambos |

### Decisión Recomendada

**Go-Live Condicionado (Fase 0):** Ejecutar el lanzamiento a producción asumiendo deuda técnica controlada, resolviendo únicamente los bloqueantes críticos en un plazo de 10-14 días. Posteriormente, ejecutar fases estructuradas de estabilización y hardening.

---

## Hoja de Ruta de Remediación

| Fase | Descripción | Plazo | Responsable |
|------|-------------|-------|-------------|
| **0A** | Mitigaciones de código (TLS) + rotación de secretos | 48 horas | Equipo Dev |
| **0B** | Migración IIS 2008 → IIS 10/13 | 7-10 días | Equipo Infra |
| **1** | Estabilización post-lanzamiento (logging, timeouts) | Mes 1 | Equipo Dev |
| **2** | Hardening de seguridad (CSRF, cookies, autenticación) | Meses 2-3 | Equipo Dev |
| **3** | Optimización de base de datos (BLOBs, DTOs) | Meses 4-6 | Equipo Dev |
| **4** | Modernización arquitectónica (.NET 8+, REST, CI/CD) | Meses 6+ | Squad Modernización |

---

## Estructura del Repositorio

```
unimar-temp/
├── README.md                              ← Este archivo
├── LICENSE
├── .gitignore
│
└── docs/
    ├── AUDITORIA_CONSOLIDADA_UNIMAR.md     ← Auditoría consolidada (comenzar aquí)
    ├── ANALISIS_OWASP_CONSOLIDADO.md       ← Análisis de seguridad OWASP Top 10
    ├── PLAN_TRABAJO_AGILE.md               ← Plan de trabajo agile con diagrama Gantt
    │
    ├── adrs/                               ← Architecture Decision Records
    │   ├── ADR_0002_Upgrade_Path_Compatibility.md
    │   ├── ADR_0003_Proteccion_Secretos_WebConfig.md
    │   └── ADR_0004_Externalizacion_Blobs_Optimizacion_Datos.md
    │
    ├── auditoria_extranet/                 ← Entregables auditoría Extranet
    │   ├── INDEX.md                        ← Índice completo Extranet
    │   ├── informe_tecnico.md              ← Informe técnico
    │   ├── ACCION_INMEDIATA_IIS2008.md     ← Checklist acción inmediata
    │   ├── anexo_diagramas.md              ← Anexo de diagramas
    │   ├── 05_INVENTARIO_LIBRERIAS.md      ← Inventario de librerías
    │   ├── adrs/                           ← ADRs específicos de Extranet
    │   └── ...
    │
    ├── auditoria_intranet/                 ← Entregables auditoría Intranet
    │   ├── RESUMEN_EJECUTIVO_E_INDICE.md   ← Resumen ejecutivo e índice
    │   ├── AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md  ← Informe técnico
    │   ├── ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md
    │   ├── 05_INVENTARIO_LIBRERIAS.md      ← Inventario de librerías
    │   ├── adrs/                           ← ADRs específicos de Intranet
    │   └── ...
    │
    └── diagramas/                          ← Diagramas de arquitectura
        ├── README.md                       ← Índice de diagramas (bilingüe)
        ├── ARQUITECTURA_UNIFICADA.md       ← Arquitectura unificada
        ├── ANALISIS_ARQUITECTURA_UNIFICADA.md
        ├── deployment_unificado.puml/.mmd
        ├── vista_fisica_servidores.puml/.mmd/.md
        ├── extranet/                       ← Diagramas Extranet (C4, UML, ER)
        └── intranet/                       ← Diagramas Intranet (C4, UML, ER)
```

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

---

## Referencias Externas

| Recurso | Enlace |
|---------|--------|
| Gestión de TLS (Microsoft) | [docs.microsoft.com/.../manage-tls](https://docs.microsoft.com/en-us/windows-server/security/tls/manage-tls) |
| HTTP/2 en IIS 10+ | [docs.microsoft.com/.../http2-on-iis](https://docs.microsoft.com/en-us/iis/get-started/whats-new-in-iis-10/http2-on-iis) |
| Prueba SSL Labs | [ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/) |
| OWASP Top 10 (2021) | [owasp.org/Top10/](https://owasp.org/Top10/) |
| Azure Blob Storage | [learn.microsoft.com/.../storage-blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/) |
| Configuración RSA Protegida | [learn.microsoft.com/.../dtkwfdky](https://learn.microsoft.com/en-us/previous-versions/aspnet/dtkwfdky(v=vs.100)) |

---

## Control de Cambios

| Versión | Fecha | Cambios | Autor |
|---------|-------|---------|-------|
| 1.0 | 2026-05-20 | README inicial con índice completo de documentación | Equipo de Auditoría Técnica |
