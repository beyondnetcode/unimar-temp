# Resumen Visual Ejecutivo (Mermaid)

## 1) Panorama de arquitectura

```plantuml
@startuml
rectangle Usuarios
rectangle "ExtranetWeb\nMVC/WebAPI"
rectangle "Extranet.Interfaces"
rectangle "Extranet.business"
rectangle "Extranet.data"
rectangle "SQL Server" as DB
rectangle "Servicios WCF/SAP" as WCF
rectangle "PortalWebApi" as API

Usuarios --> "ExtranetWeb\nMVC/WebAPI"
"ExtranetWeb\nMVC/WebAPI" --> "Extranet.Interfaces"
"Extranet.Interfaces" --> "Extranet.business"
"Extranet.business" --> "Extranet.data"
"Extranet.data" --> DB
"ExtranetWeb\nMVC/WebAPI" --> WCF
"ExtranetWeb\nMVC/WebAPI" --> API
@enduml
```

## 2) Mapa ejecutivo de riesgo

### Matriz de Riesgo (Impacto vs Urgencia)

| Riesgo | Impacto | Urgencia | Cuadrante | Acción |
|--------|---------|----------|-----------|--------|
| **Credenciales expuestas** | 0.95 | 0.96 | CRÍTICA (I) | ⚠️ Atencion INMEDIATA (P0, <48h) |
| **Endpoints HTTP críticos** | 0.90 | 0.92 | CRÍTICA (I) | ⚠️ Atencion INMEDIATA (P0, <48h) |
| **Debug en config base** | 0.72 | 0.80 | ALTA (II) | 🔴 Atender pronto (<1 semana) |
| **Cookies sin hardening completo** | 0.78 | 0.76 | ALTA (II) | 🔴 Atender pronto (<1 semana) |
| **Falta auth/csrf declarativo** | 0.82 | 0.84 | ALTA (II) | 🔴 Atender pronto (<1 semana) |
| **Deuda framework legacy** | 0.62 | 0.58 | MEDIA (III) | 🟡 Monitorear y planificar (Roadmap L/P) |

**Leyenda Cuadrantes:**
- **I (Alto impacto + Alta urgencia):** Atención inmediata - Bloqueantes de producción
- **II (Alto impacto + Baja urgencia / Bajo impacto + Alta urgencia):** Atender en próximas semanas
- **III (Bajo impacto + Baja urgencia):** Monitorear y planificar

```plantuml
@startuml
rectangle "Cuadrante 1\n(CRÍTICA)\nAtencion INMEDIATA" as Q1 #FFCCcc
rectangle "Cuadrante 2\n(ALTA)\nAtender pronto" as Q2 #FFFF99
rectangle "Cuadrante 3\n(MEDIA)\nMonitorear" as Q3 #CCFFCC
rectangle "Cuadrante 4\n(BAJA)\nPlanificar" as Q4 #CCCCFF
@enduml
```

**Ubicación en matriz (estimado):**
- Credenciales: Esquina superior derecha (Q1) - CRÍTICA
- Endpoints HTTP: Esquina superior derecha (Q1) - CRÍTICA
- Debug flag: Cuadrante superior izquierdo (Q2) - ALTA
- Cookie security: Cuadrante superior izquierdo (Q2) - ALTA
- Auth/CSRF: Cuadrante superior izquierdo (Q2) - ALTA
- Framework legacy: Centro/cuadrante inferior (Q3) - MEDIA/BAJA

## 3) Ruta de remediacion

```plantuml
@startuml
project Ruta de remediacion por fases

' Fase 0
[Fase 0: Bloqueantes criticos] starts 2026-04-29 and ends 2026-05-02
[Fase 0: Bloqueantes criticos] is colored in #FF6666

' Fase 1
[Fase 1: Estabilizacion temprana] starts 2026-05-02 and ends 2026-05-16
[Fase 1: Estabilizacion temprana] is colored in #FFCC66

' Fase 2
[Fase 2: Hardening de seguridad] starts 2026-05-16 and ends 2026-06-06
[Fase 2: Hardening de seguridad] is colored in #FFFF66

' Fase 3
[Fase 3: Optimizacion funcional] starts 2026-06-06 and ends 2026-07-06
[Fase 3: Optimizacion funcional] is colored in #66FF66

' Fase 4
[Fase 4: Modernizacion estructural] starts 2026-07-06 and ends 2026-08-31
[Fase 4: Modernizacion estructural] is colored in #66FFFF

@enduml
```

## 4) Ruta de salida a produccion ASAP

```plantuml
@startuml
start
:Inicio validacion bloqueantes;
if (Credenciales rotadas?) then (No)
  :Rotar y revocar;
  :Re-validar;
else (Si)
  :Credenciales OK;
endif
if (TLS operativo en flujos criticos?) then (No)
  :Habilitar TLS + validar;
  :Re-validar;
else (Si)
  :TLS OK;
endif
if (IIS/AppPool estable?) then (No)
  :Ajustar baseline infraestructura;
  :Re-validar;
else (Si)
  :Infraestructura OK;
endif
if (Smoke E2E OK?) then (No)
  :Corregir bloqueantes funcionales;
  :Re-validar;
else (Si)
  :Funcionalidad OK;
endif
if (Logging + runbook + rollback?) then (No)
  :Completar operacion y soporte;
  :Re-validar;
else (Si)
  :Soporte OK;
endif
:GO CONDICIONADO A PRODUCCION;
:Monitoreo reforzado primeras 72h;
stop
@enduml
```
