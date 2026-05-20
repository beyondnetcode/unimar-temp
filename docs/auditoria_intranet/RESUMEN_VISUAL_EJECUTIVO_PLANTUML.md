# Resumen Visual Ejecutivo (PlantUML)

Documento relacionado:

- Ver resumen e indice: [RESUMEN_EJECUTIVO_E_INDICE.md](RESUMEN_EJECUTIVO_E_INDICE.md)
- Ver informe tecnico: [AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md](AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md)
- Ver anexo de arquitectura: [ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md](ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md)

**Nota:** PlantUML es una alternativa libre y de código abierto a Mermaid. Puedes visualizar los diagramas usando:
- Extensión PlantUML para VS Code (gratuita)
- PlantUML Online: http://www.plantuml.com/plantuml/uml/
- Generar PNG/SVG via CLI: `plantuml diagram.md`

## Indice visual

1. [Panorama de arquitectura](#1-panorama-de-arquitectura)
2. [Mapa ejecutivo de riesgo](#2-mapa-ejecutivo-de-riesgo)
3. [Ruta de salida a produccion ASAP](#3-ruta-de-salida-a-produccion-asap)
4. [Plan progresivo de optimizacion](#4-plan-progresivo-de-optimizacion)

## 1) Panorama de arquitectura

```plantuml
@startuml
skinparam backgroundColor #FEFEFE
skinparam defaultFontSize 11

rectangle "Usuarios Web y Movil" as U #E8F4F8
rectangle "SIS_INTRANET Web\nASP.NET MVC 4 / Web API 4" as W #f4d35e
rectangle "Business Layer" as B #FFE8B8
rectangle "Data Layer" as D #FFD9B3
rectangle "SQL Server" as DB #9ad1d4
rectangle "Interfaces" as I #FFE8B8
rectangle "Servicios WCF / REST" as S #ee964b

U --> W
W --> B
B --> D
D --> DB
W --> I
I --> S
@enduml
```

## 2) Mapa ejecutivo de riesgo

```plantuml
@startuml
skinparam backgroundColor #FFF5F5
skinparam defaultFontSize 11

rectangle "Riesgo global: Alto-Critico" as R0 #FF6B6B
rectangle "Seguridad\nCredenciales y TLS" as R1 #FFB3B3
rectangle "Continuidad\nEstabilidad operativa" as R2 #FFB3B3
rectangle "Arquitectura\nMonolito legacy" as R3 #FFB3B3
rectangle "Datos\nNormalizacion y sobrelectura" as R4 #FFB3B3

R0 --> R1
R0 --> R2
R0 --> R3
R0 --> R4

R1 --> rectangle "Remediacion prioritaria pre Go-Live" as A1 #90EE90
R2 --> rectangle "Hardening de infraestructura" as A2 #90EE90
R3 --> rectangle "Controles compensatorios" as A3 #90EE90
R4 --> rectangle "Optimizacion progresiva por dominios" as A4 #90EE90
@enduml
```

## 3) Ruta de salida a produccion ASAP

```plantuml
@startuml
skinparam backgroundColor #F0F8FF
skinparam defaultFontSize 11

rectangle "Produccion ASAP\nsin caidas" as P0 #4DA6FF
rectangle "Rotar\ncredenciales" as P1 #90EE90
rectangle "HTTPS/TLS en\nflujos criticos" as P2 #90EE90
rectangle "Validar IIS\nAppPool recursos" as P3 #90EE90
rectangle "Smoke tests\nE2E minimos" as P4 #90EE90
rectangle "Logging y\nmonitoreo" as P5 #90EE90
rectangle "Go-Live\n+ rollback" as P6 #32CD32

P0 --> P1
P1 --> P2
P2 --> P3
P3 --> P4
P4 --> P5
P5 --> P6
@enduml
```

## 4) Plan progresivo de optimizacion

```plantuml
@startuml
title Ruta progresiva posterior al arranque
skinparam backgroundColor #FFFACD
skinparam defaultFontSize 10

rectangle "Fase 0: Go-Live rapido y estable\nMinimos bloqueantes" as F0 #90EE90

rectangle "Fase 1: Estabilizacion temprana\nCorreccion de incidentes de alta recurrencia" as F1 #FFD700

rectangle "Fase 2: Hardening de seguridad\nReduccion de superficie de ataque" as F2 #FFA500

rectangle "Fase 3: Optimizacion funcional y de datos\nNormalizacion focalizada y rendimiento" as F3 #FF8C00

rectangle "Fase 4: Modernizacion estructural\nReduccion de deuda tecnica legacy" as F4 #FF6347

F0 --> F1 : Semanas 1-2
F1 --> F2 : Semanas 3-4
F2 --> F3 : Mes 2-3
F3 --> F4 : Mes 4+

@enduml
```
