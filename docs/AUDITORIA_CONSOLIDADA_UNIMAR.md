# 📘 Auditoría Consolidada Unimar

## 1. Resumen Ejecutivo Unificado

**Diagnóstico Situacional:** La infraestructura y el ecosistema web de Unimar (Intranet y Extranet) se encuentran en estado de **obsolescencia crítica** y **alta vulnerabilidad**. Ambas plataformas operan sobre tecnologías fuera de ciclo de vida (IIS 2008, .NET Framework 4.5, ASP.NET MVC 4) y dependencias con múltiples vulnerabilidades severas (CVEs críticos en SharpZipLib, Newtonsoft, jQuery). A nivel operativo, esto ya ha detonado **inestabilidad bloqueante** en la Extranet, donde la incompatibilidad de TLS con dispositivos móviles genera caídas de servicio. A nivel de seguridad, existe exposición de credenciales en texto plano y tráfico de red no cifrado.

**El Dilema Estratégico (Recuperación vs. Descarte):** La acumulación profunda de deuda técnica (monolitos acoplados, dependencias de 2012, over-fetching en base de datos) hace inviable una remediación "in-place" definitiva sin interrumpir la operación. La directiva debe asumir que el código actual es **altamente frágil y costoso de mantener**. Se debe ejecutar un plan de mitigación progresiva a corto/medio plazo para asegurar la continuidad, mientras en paralelo se evalúa el ROI de **descartar los monolitos legacy** para migrar hacia una nueva arquitectura (ej. microservicios, .NET 8, React/Angular) en lugar de intentar reescribir un sistema obsoleto.

**Decisión Directiva:** Se recomienda autorizar un **Go-Live Condicionado (Fase 0)**. El lanzamiento se ejecutará asumiendo deuda técnica controlada, resolviendo únicamente los bloqueantes críticos en un plazo de 10-14 días. Posteriormente, se ejecutarán fases estructuradas de estabilización y hardening, culminando en la decisión ejecutiva final de reescritura o descarte.

**Anexos y Documentación de Respaldo:**
Para una exploración técnica a detalle, referirse a los siguientes documentos consolidados:
*   **Auditoría Extranet:** [Índice y Resumen Ejecutivo](auditoria_extranet/INDEX.md) | [Informe Técnico](auditoria_extranet/informe_tecnico.md) | [Acción Inmediata](auditoria_extranet/ACCION_INMEDIATA_IIS2008.md) | [ADR-001 (IIS 2008)](auditoria_extranet/adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md) | [Inventario de Librerías](auditoria_extranet/05_INVENTARIO_LIBRERIAS.md)
*   **Auditoría Intranet:** [Índice y Resumen Ejecutivo](auditoria_intranet/RESUMEN_EJECUTIVO_E_INDICE.md) | [Informe Técnico](auditoria_intranet/AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md) | [Anexo Diagramas](auditoria_intranet/ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md) | [Inventario de Librerías](auditoria_intranet/05_INVENTARIO_LIBRERIAS.md)

## 2. Contexto y Alcance
Este documento consolida las auditorías técnicas de los frentes web de Unimar:
*   **Extranet (`Extranet.sln`)**: Portal orientado a clientes y usuarios externos (incluyendo un 40%+ de tráfico móvil). Actualmente afectado por incidentes críticos de infraestructura (caídas de servidor).
*   **Intranet (`SIS_INTRANET.sln`)**: Sistema core corporativo de uso interno. Presenta arquitectura monolítica acoplada que gestiona flujos críticos (seguridad, reservas, reclamos, transmisiones).
El objetivo es fusionar hallazgos, eliminar redundancias y trazar una hoja de ruta única que garantice la continuidad operativa y una modernización progresiva.

## 3. Arquitectura Actual
### Vista Intranet
*   Monolito en .NET Framework 4.5, MVC 4 y Web API 4.
*   Alta dependencia de Procedimientos Almacenados (SPs).
*   Entidades de datos desnormalizadas (ej. carga eager de BLOBs y entidades extremadamente anchas).
*   Modelo de autorización no estándar (filtros customizados en lugar de atributos globales).

### Vista Extranet
*   Mismo stack tecnológico base (.NET 4.5, MVC 4).
*   Duplicidad de árboles de proyecto web (`ExtranetWeb` vs `Extranet.web`), riesgo de deriva de código.
*   Hospedado en infraestructura severamente desactualizada (Windows Server 2008 / IIS 2008).

### Integración entre ambas
*   Se comunican y dependen de servicios WCF/SOAP legacy (ej. `IntranetService`).
*   Configuraciones cruzadas mediante endpoints HTTP no cifrados.
*   Comparten objetos de negocio y contratos que propagan la deuda técnica en ambas direcciones.

## 4. Hallazgos Consolidados
### Críticos
1.  **Infraestructura Obsoleta y Crisis TLS (Extranet):** IIS 2008 falla negociando TLS 1.2+ con móviles, botando conexiones y saturando el CPU. (Bloqueante).
2.  **Secretos Expuestos (Transversal):** Cadenas de conexión a BD con usuarios y contraseñas en texto plano dentro de los `Web.config`.
3.  **Transporte Inseguro (Transversal):** Múltiples endpoints, WCF y recursos configurados sobre `http://`, exponiendo tráfico a sniffing o ataques MITM.

### Altos
4.  **Ausencia de Controles CSRF:** No se usa `[ValidateAntiForgeryToken]` en operaciones POST en ninguno de los sistemas.
5.  **Autorización Débil y no Centralizada:** Falta de `[Authorize]`, dependiendo de sesiones manuales o validaciones en API incompletas.
6.  **Configuraciones Inseguras de Producción:** Extranet presenta `debug="true"` en Web.config base; Intranet presenta `validateRequest="false"` en vistas, abriendo la puerta a fuga de datos y XSS.
7.  **Over-fetching y Modelado de Datos (Intranet):** Entidades anchas, mezcla de DTOs y persistencia, y BLOBs (byte[]) hidratados junto con la metadata.

### Medios
8.  **Manejo de Errores Silencioso:** Abuso de `catch {}` vacíos y excepciones genéricas sin correlación ni logging adecuado.
9.  **Deuda Tecnológica Legacy:** Framework .NET 4.5, librerías obsoletas críticas transversales (MVC 4, WebApi 4, Newtonsoft antiguo) y específicas con vulnerabilidades severas (Extranet: jQuery 2.x, Bootstrap 3.x; Intranet: SharpZipLib 0.86, ClosedXML).
10. **Cookies sin Hardening:** Cookies de sesión generadas sin banderas `Secure` y `SameSite`.

### Bajos
11. Falta de pipeline automatizado (CI/CD) y validación SAST/SCA en repositorios.
12. Ausencia de un modelo formalizado físico SQL en el repositorio.

## 5. Análisis de Seguridad (Estándar OWASP Top 10)
Los ecosistemas evaluados presentan violaciones críticas en 8 de las 10 categorías principales de **OWASP Top 10 (2021)**. El mayor riesgo proviene de operar bajo tecnologías EOL (End-of-Life) como IIS 2008 y componentes con vulnerabilidades severas sin parchar (A06: Componentes Vulnerables).
Las brechas más destacadas mapeadas al estándar incluyen:
*   **A01: Broken Access Control:** Falta de `[Authorize]` global.
*   **A02: Cryptographic Failures:** TLS obsoleto, credenciales en texto plano en Web.config.
*   **A03: Injection:** `validateRequest="false"` sin saneamiento complementario.
*   **A04: Insecure Design:** Ausencia de tokens Anti-CSRF en transacciones POST.
*   **A05: Security Misconfiguration:** `debug="true"` en producción, error de configuración en cookies.
*   **A06: Vulnerable Components:** Múltiples dependencias con CVEs (Newtonsoft antiguo, SharpZipLib, jQuery 2.x).

*Referencia completa: [Análisis de Seguridad Consolidado (OWASP Top 10)](ANALISIS_OWASP_CONSOLIDADO.md)*

## 6. Riesgos Globales
### Seguridad
Altamente susceptibles a exfiltración de base de datos (por credenciales expuestas), robo de sesiones de usuarios por MITM (ausencia de TLS) y toma de control de acciones de usuario interno/externo (falta de CSRF/Auth centralizado).
### Operacionales
Inestabilidad inminente bajo carga (ya evidenciada en Extranet con móviles). Fuerte dificultad de diagnóstico post-falla debido a logging defectuoso y excepciones silenciosas.
### Arquitectónicos
Fragilidad en las integraciones por acoplamiento WCF, y degradación de performance progresiva de BD por hidratación excesiva de datos (over-fetching y falta de segregación de lecturas/escrituras).

## 7. Análisis Comparativo
### Diferencias clave
*   **Criticidad de Disponibilidad:** Extranet enfrenta un colapso inminente en dispositivos móviles (P0), mientras que Intranet tiene problemas de rendimiento interno pero no caídas generalizadas por plataforma de acceso.
*   **Exposición:** Extranet es atacable públicamente; Intranet cuenta con la capa protectora de la red corporativa (aunque un atacante interno o movimiento lateral comprometería la BD).

### Brechas
*   Las validaciones de estado en base de datos están fuertemente delegadas a SPs, generando un desfase entre la lógica del código y el esquema real de datos.
*   Brecha de observabilidad: Ninguno de los aplicativos expone telemetría ni métricas de salud (health checks).

## 8. Problemas Transversales
*   **Identidad y Accesos Descentralizados:** Ambos reinventan la rueda con tokens o cookies custom, sin integrar un Identity Provider o JWT estándar.
*   **Configuración Local Hardcodeada:** Ambos dependen intensivamente de transformaciones ineficientes o variables en `Web.config` que mezclan entornos.

## 9. Recomendaciones Estratégicas
1.  **Contención antes que Reescritura:** No iniciar grandes refactors de código ni actualizar el framework base sin antes migrar el sistema operativo y tapar las brechas directas (Credenciales y TLS).
2.  **Separación de DTOs (Data Transfer Objects):** Evitar de inmediato pasar objetos que contienen BLOBs (como `TR_UNI_Archivo`) directo a las interfaces y controladores.
3.  **Pipeline Único:** Definir en fase temprana un único artefacto de compilación para Extranet para frenar el "drift" de código entre directorios duplicados.
4.  **Estandarizar Identidad:** Como proyecto paralelo, evaluar centralizar el control de acceso de ambas plataformas hacia un proveedor corporativo (SSO/OAuth2).

## 10. Plan de Acción
### Quick Wins (Fase 0 - Antes de 14 días)
*   **Infraestructura:** Migrar entorno Extranet a IIS 10/13 (Windows Server 2016/2022).
*   **Secretos:** Rotar claves de BD y moverlas a almacenes seguros (Vault / Env Vars).
*   **Comunicaciones:** Forzar HTTPS (`Rewrite` rule) y cambiar endpoints WCF internos a `https://`.
*   **Estabilidad:** Activar logging de excepciones crítico.

### Mediano plazo (Fases 1 y 2 - 1 a 3 meses)
*   Implementar atributos `[ValidateAntiForgeryToken]` en vistas transaccionales.
*   Activar atributos `Secure` y `SameSite` en todas las cookies.
*   Revisar consultas pesadas (Over-fetching) aislando campos de auditoría o de contenido binario en Intranet.
*   Centralizar el manejo de excepciones (`HandleErrorAttribute` extendido).

### Largo plazo (Fases 3 y 4 - 3 a 6+ meses)
*   Desacoplamiento de Servicios: Reemplazar WCF por API REST modernas en dominios clave (ej. Reservas y Reclamos).
*   Modernizar Stack: Planificar migración a .NET 8+.
*   Implementar CI/CD con escaneo de seguridad estático (SAST).

## 11. Roadmap Estratégico

| Fase | Hito | Tiempo | Condición |
|---|---|---|---|
| **0A** | Mitigación código TLS y rotación secretos | 48h | Requisito Pre-Go-Live |
| **0B** | Migración Extranet a IIS 10+ (WinServer 2016/2022) | 7-10 días | **Go-Live Condicionado** |
| **1** | Estabilización post-lanzamiento (Logging y Timeouts) | Mes 1 | Operación sin disrupciones |
| **2** | Hardening Seguridad (CSRF, Cookies, Auth) | Mes 2-3 | Ventanas mantenimiento |
| **3** | Saneamiento Base Datos (Proyecciones, DTOs) | Mes 4-6 | Según refactor |
| **4** | Modernización Arquitectura (.NET 8+, REST, CI/CD) | Mes 6+ | Proyectos CapEx |

---

### ✔ One-Page Ejecutivo: Estrategia de Continuidad y Modernización
**Asunto:** Auditoría Técnica y Estratégica de Soluciones Web Unimar (Intranet/Extranet)
**Estado Global:** 🔴 Riesgo Crítico (Vulnerabilidades severas y bloqueante operativo en Extranet).

**Diagnóstico Ejecutivo:**
Ambas plataformas sostienen la operación del negocio pero operan sobre infraestructura y código obsoleto (.NET 4.5, IIS 2008). Esto se ha traducido en un colapso inminente en Extranet (caída de servicio ante usuarios móviles por falla TLS) y fugas graves de seguridad (credenciales expuestas y tráfico HTTP). Intentar modernizar el sistema actual antes del lanzamiento detendría el negocio indefinidamente.

**Plan de Acción Directivo (Fases Progresivas):**
*   **Fase 0 - Contención Inmediata (NO-GO a GO Condicionado):** Detener salida a producción por 10-14 días para realizar migración de emergencia a IIS 10+, rotación de credenciales expuestas y forzar cifrado HTTPS.
*   **Fase 1 - Estabilización y Hardening (1-3 Meses):** Con el sistema operando, mitigar librerías críticas (actualizar Newtonsoft, eliminar SharpZipLib antiguo) e inyectar validaciones de seguridad básicas (Tokens Anti-CSRF, cookies seguras).
*   **Fase 2 - Optimización Táctica (3-6 Meses):** Refactorizar cuellos de botella severos en base de datos (eliminar consultas masivas de BLOBs) para evitar degradación de rendimiento.
*   **Fase 3 - Descarte o Re-Migración (6+ Meses):** Dada la fragilidad del stack de 2012, frenar inversiones de código profundo en el monolito actual. Iniciar el diseño arquitectónico de una nueva plataforma moderna (REST APIs, .NET 8, Frontend SPA) para reemplazar y "apagar" progresivamente el sistema actual.

*(El desglose detallado en días, organización de squads y el cronograma visual Gantt se encuentra en el [Plan de Trabajo Agile](PLAN_TRABAJO_AGILE.md)).*

### ✔ Top 10 Riesgos Críticos
1.  **Caída de Servicio (Móviles)**: Falla de negociación TLS en IIS 2008 (Extranet).
2.  **Compromiso de BD**: Credenciales expuestas en archivos `.config` (Ambos).
3.  **Ataque de Intermediario (MITM)**: Integraciones y accesos por `http://` (Ambos).
4.  **Ataque Cross-Site Request Forgery (CSRF)**: Ausencia de tokens anti-CSRF en POSTs (Ambos).
5.  **Secuestro de Sesiones**: Cookies de autenticación sin flags seguros (Ambos).
6.  **Desborde de Memoria/CPU**: Carga Eager de archivos BLOB y entidades anchas en memoria (Intranet).
7.  **Inyecciones y XSS**: Parámetro `validateRequest="false"` sin filtros sanitizadores complementarios (Intranet).
8.  **Pérdida de Observabilidad**: Manejo silente de excepciones (`catch{}`) (Ambos).
9.  **Despliegues Equivocados**: Duplicidad de proyectos web en soluciones (Extranet).
10. **Explotación de Vulnerabilidades Legacy**: Stack desactualizado sin parches de seguridad (MVC4/WebApi4) (Ambos).

### ✔ Backlog Técnico Priorizado

| Épica | Descripción | Prioridad |
|---|---|---|
| **Infraestructura** | Provisionar y migrar servidor Extranet de IIS 2008 a IIS 10+. Pruebas de estrés móviles. | P0 - Bloqueante |
| **Seguridad Perimetral**| Implementar **WAF (Virtual Patching)** frente a IIS 10 para bloquear ataques OWASP conocidos. | P0 - Bloqueante |
| **Seguridad Base** | Rotar credenciales en base de datos y ocultar connection strings (Vault/MachineKeys). | P0 - Bloqueante |
| **Redes Seguras** | Forzar HTTPS en todo acceso y reconfigurar WCF para rechazar HTTP plano. | P0 - Bloqueante |
| **Observabilidad** | Eliminar los `catch {}` vacíos. Agregar log4net a nivel aplicación global. | P1 - Alta |
| **Protección Web** | Activar atributos `Secure` y `SameSite` en generador de cookies. | P1 - Alta |
| **Hardening MVC** | Auditar e inyectar `[ValidateAntiForgeryToken]` en vistas/controladores POST transaccionales. | P2 - Media |
| **Librerías Externas** | Integrar **SCA** en pipeline. Actualización por oleadas de dependencias severas (Newtonsoft, SharpZipLib). | P2 - Media |
| **Optimización Datos** | **Externalizar BLOBs a File Storage** (ej. Azure Blob) reduciendo carga masiva en SQL Server. | P2 - Media |
| **Limpieza Repo** | Eliminar proyecto web duplicado de Extranet y estandarizar Pipeline de build. | P3 - Baja |
| **Modernización** | Desplegar **API Gateway** e iniciar refactor gradual de servicios WCF hacia APIs REST modernas. | P4 - Roadmap |

---

### ✔ Índice General de Documentos Anexos
Para explorar en profundidad todos los entregables generados durante la auditoría técnica, por favor revise los siguientes enlaces:

#### 🌐 Auditoría Extranet
*   [Índice General y Resumen Ejecutivo](auditoria_extranet/INDEX.md)
*   [Informe Técnico Detallado](auditoria_extranet/informe_tecnico.md)
*   [Plan de Acción Inmediata (IIS 2008)](auditoria_extranet/ACCION_INMEDIATA_IIS2008.md)
*   [ADR-001: Crisis TLS y Servidor Móvil](auditoria_extranet/adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md)
*   [Inventario y Plan de Librerías](auditoria_extranet/05_INVENTARIO_LIBRERIAS.md)

#### 🏢 Auditoría Intranet
*   [Índice y Resumen Ejecutivo](auditoria_intranet/RESUMEN_EJECUTIVO_E_INDICE.md)
*   [Informe Técnico Completo](auditoria_intranet/AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md)
*   [Anexo de Diagramas de Arquitectura (UML/C4/ER)](auditoria_intranet/ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md)
*   [Inventario y Plan de Librerías](auditoria_intranet/05_INVENTARIO_LIBRERIAS.md)

#### 🛡️ Estrategia y Seguridad Transversal
*   [Análisis de Seguridad Consolidado (OWASP Top 10)](ANALISIS_OWASP_CONSOLIDADO.md)
*   [Plan de Trabajo Agile (incluye Cronograma Gantt)](PLAN_TRABAJO_AGILE.md)

#### 🎨 Repositorio de Diagramas (Archivos .puml)
*   [Diagramas Extranet (UML, C4, E/R)](diagramas/extranet/)
*   [Diagramas Intranet (UML, C4, E/R)](diagramas/intranet/)
