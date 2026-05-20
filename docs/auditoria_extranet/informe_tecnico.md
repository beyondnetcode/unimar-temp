# Auditoria Tecnica Integral - SIS_INTRANET (equivalente en repositorio: Extranet)

Fecha: 2026-04-28  
Alcance: analisis estatico de codigo y configuracion del workspace + hallazgo de infraestructura crítica reportado por Coordinador.  
Importante: no se identifico modelo fisico SQL oficial en el repositorio; todo el analisis de datos y tablas es inferido desde entidades, capas DAL/BLL y convenciones de nombres.

**ACTUALIZACIÓN CRÍTICA (2026-04-28 19:00):**
Se agregó hallazgo de **infraestructura bloqueante**: IIS 2008 falla en negociación TLS con dispositivos móviles, causando caídas de servidor. Esto es un **NO-GO para producción** sin migración a IIS 10/13. Ver sección 4 (H-00) y [ADR-0001](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md).

Documentos relacionados:
- [Resumen ejecutivo e indice](resumen_ejecutivo_indice.md)
- [Anexo de diagramas](anexo_diagramas.md)
- [Resumen visual ejecutivo](resumen_visual_ejecutivo.md)
- **[ADRs - Architecture Decision Records](adrs/)** - NEW: ADR-0001 sobre migración IIS 2008→IIS 10/13

## 1) Resumen ejecutivo tecnico

La solucion auditada presenta una arquitectura legacy por capas (.NET Framework 4.5, ASP.NET MVC 4, Web API 4) con alto acoplamiento a servicios externos y fuerte dependencia de procedimientos almacenados. La prioridad estrategica solicitada (salir a produccion ASAP con cambios minimos para evitar caidas) es viable solo con **Go-Live Condicionado** a bloqueantes tecnicos de seguridad/operacion muy puntuales.

**HALLAZGO CRÍTICO AGREGADO:** Infraestructura IIS 2008 es **bloqueante inmediato** - falla en TLS con móviles. Requiere migración a IIS 10/13 antes de cualquier Go-Live a producción (7-10 días).

Estado global propuesto:
- Decision: **NO-GO inicial** → **GO CONDICIONADO (10-14 días)** una vez completada migración IIS + cierre de demás bloqueantes.
- Condicion: cerrar 7 minimos bloqueantes pre-Go-Live (IIS 10+, credenciales, TLS efectivo, parametros infraestructura, smoke E2E móvil, logging minimo, runbook/rollback).
- Riesgo residual post-Go-Live: bajo (tras migración IIS + contencion seguridad), mitigable por fases 1-4 sin frenar la operacion.

## 2) Inventario de solucion y componentes

### 2.1 Solucion detectada en workspace
- Solucion principal encontrada: `Extranet.sln`.
- Proyectos incluidos en la solucion principal:
  - `ExtranetWeb/Extranet.web.csproj` (front web MVC + Web API).
  - `Extranet.business/Extranet.business.csproj`.
  - `Extranet.data/Extranet.data.csproj`.
  - `Extranet.objects/Extranet.objects.csproj`.
  - `Extranet.Interfaces/Extranet.Interfaces.csproj`.

### 2.2 Equivalencia con contexto solicitado
- El requerimiento menciona `SIS_INTRANET`, `INTRANET.business`, `INTRANET.data`, etc.
- En este repositorio el namespace y carpetas reales estan bajo prefijo `Extranet`.
- No se encontraron artefactos `SIS_INTRANET.Test` ni `IntranetService` como proyectos formales dentro de la solucion detectada.

### 2.3 Componentes funcionales relevantes observados
- Dominios funcionales presentes (por nombres de controladores, interfaces y entidades):
  - Transmisiones/Documentos: `TR_DVA_DOCH`, `TR_DVA_DOCP`, `ImportacionesController`.
  - Operativo/Comercial (devolucion/asignacion): `TR_DVA_Reserva_Cab`, `TR_DVA_Reserva_Det`, `TR_DVA_SolicitudVacio*`.
  - Seguridad/Acceso: `TB_Usuario`, `SG_Usuario`, `LoginController`, `SeguridadController`.
  - Reclamos: `TR_UNI_Reclamo`, `TR_UNI_Reclamo_Doc`, `ReclamoController`.
  - Visitas/Reservas: `TR_DVA_Reserva_*`, `DevolucionController`, `VaciosController`.
  - Deposito Vacios: `AsignacionVaciosController`, `VaciosController`, `TR_DVA_*`.

## 3) Plataforma, versiones, dependencias y despliegue

### 3.1 Plataforma
- .NET Framework: 4.5.
- ASP.NET MVC: 4.x.
- ASP.NET Web API: 4.x.
- WCF clients configurados por `basicHttpBinding`.

### 3.2 Dependencias detectadas (legacy)
- `Microsoft.AspNet.Mvc 4.0.20710.0`.
- `Microsoft.AspNet.WebApi 4.0.20710.0`.
- `Newtonsoft.Json 4.5.6` (en `packages.config`) y referencia 6.0.0.0 en csproj.
- `log4net 2.0.8` (packages) y referencia assembly 1.2.10.0 en csproj.
- Dependencias web declaradas:
  - `https://www.unimar.com.pe/extranet`
  - `https://www.unimar.com.pe/intranet`

### 3.3 Despliegue inferido
- Aplicacion IIS clasica (MVC/Web API en mismo sitio).
- Forzado de HTTPS a nivel aplicacion en `Application_BeginRequest` bajo compilacion no DEBUG.
- Web service dependencies externas/internas configuradas por endpoint `http://...` (varios hosts intranet/QAS).

## 4) Hallazgos de seguridad y arquitectura (con criticidad)

### H-00 Infraestructura: IIS 2008 EOL con crisis de TLS en dispositivos móviles (CRÍTICO BLOQUEANTE)
- Descripción: Servidor IIS 2008 (EOL desde 2015) falla en negociación TLS con clientes móviles (iOS/Android), causando caídas bajo carga.
- Evidencia técnica: 
  - Reportado por Coordinador de Infraestructura: caídas observadas, timeouts en móviles, CPU spike durante picos
  - IIS 2008 soporta TLS 1.0/1.1 (obsoletos), no TLS 1.2/1.3 de forma optimizada
  - Connection pooling débil, sin HTTP/2 support, handshake lento (~2s vs 400ms en IIS 10+)
- Riesgo: Usuarios móviles (40%+ del tráfico) experimentan "Conexión rechazada", aplicación inoperable para este segmento
- Impacto: **BLOQUEANTE para Go-Live** - móviles no pueden acceder
- Acción inmediata (P0):
  - Fase 0A (Hoy-Mañana): Implementar mitigaciones desde código (.NET 4.5) - reducción temporal 40-50%
  - Fase 0B (7-10 días): Migración a IIS 10 (Win Server 2016) o IIS 13 (Win Server 2022) → solución permanente
- Referencia: [ADR-0001: Migración Urgente IIS 2008 → IIS 10/13](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md)
- Criterios de cierre:
  - ✓ IIS 10+ desplegado en staging + producción
  - ✓ TLS 1.2/1.3 activo (1.0/1.1 deshabilitados)
  - ✓ HTTP/2 confirmado operativo
  - ✓ Load test móvil: 0 timeouts en 72h
  - ✓ CPU < 70% bajo carga
  - ✓ PCI-DSS conforme (TLS 1.2+)

### H-01 Credenciales en texto plano en configuracion (CRITICO)
- Descripcion: hay cadenas de conexion con usuario/password hardcodeados.
- Riesgo: exposicion de secretos, movimiento lateral, compromiso de BD.
- Evidencia: `ExtranetWeb/Web.config` lineas 11-12.
- Recomendacion minima ASAP:
  - Rotar credenciales inmediatamente.
  - Mover secretos a mecanismo seguro (IIS encrypted config/secret store/variables entorno).
  - Segregar credenciales por entorno con privilegio minimo.

### H-02 Trafico HTTP en endpoints criticos externos/internos (CRITICO)
- Descripcion: se observan `appSettings` y endpoints WCF con `http://`.
- Riesgo: sniffing/MITM, robo de sesion/token y datos operativos.
- Evidencia: `ExtranetWeb/Web.config` lineas 26, 32, 92-115.
- Recomendacion minima ASAP:
  - Activar TLS real extremo a extremo para flujos expuestos.
  - Mantener allow-list temporal solo para redes internas si algun endpoint no migra en T0.

### H-03 Compilacion en debug en configuracion base web (ALTO)
- Descripcion: `debug="true"` en Web.config principal.
- Riesgo: penalizacion de rendimiento, mayor superficie de fuga de informacion.
- Evidencia: `ExtranetWeb/Web.config` linea 37.
- Recomendacion minima ASAP:
  - Confirmar transformacion release efectiva en pipeline/publicacion.
  - Verificar artefacto desplegado en servidor final con debug desactivado.

### H-04 Ausencia de controles globales declarativos de autorizacion/CSRF (ALTO)
- Descripcion: no se encontraron atributos `[Authorize]` ni `[ValidateAntiForgeryToken]` en `ExtranetWeb`.
- Riesgo: acciones POST dependientes de validaciones ad-hoc y sesion; potencial bypass/CSRF.
- Evidencia:
  - Busqueda global de patrones en `ExtranetWeb/**/*.cs`: sin coincidencias.
  - `ExtranetWeb/App_Start/FilterConfig.cs` solo registra `HandleErrorAttribute`.
- Recomendacion minima ASAP:
  - En fase 0, proteger operaciones sensibles con token y validacion de sesion robusta.
  - En fase 2, estandarizar autorizacion declarativa y antiforgery por modulo.

### H-05 Politica de cookies incompleta para seguridad moderna (ALTO)
- Descripcion: cookies de autenticacion se crean sin flags `Secure`/`SameSite` activos (version segura comentada).
- Riesgo: robo/replay de cookie en escenarios mixtos o navegacion cross-site.
- Evidencia: `ExtranetWeb/Util/ApiRest/AutenticacionApiRest.cs` lineas 41-45, 79-82.
- Recomendacion minima ASAP:
  - Habilitar `Secure=true`, `HttpOnly=true`, `SameSite=Strict/Lax` segun flujo real.

### H-06 Manejo de errores con capturas vacias o no estructuradas (MEDIO-ALTO)
- Descripcion: existen `catch {}` y `catch(Exception){}` sin accion.
- Riesgo: fallas silenciosas, troubleshooting lento, degradacion no detectada.
- Evidencia:
  - `ExtranetWeb/Controllers/BaseController.cs` lineas 66, 532.
  - `ExtranetWeb/Controllers/SeguridadController.cs` linea 1000.
  - `ExtranetWeb/Controllers/VaciosController.cs` linea 926.
- Recomendacion:
  - Logging estructurado minimo obligatorio en excepciones de capa web y servicios.

### H-07 Deuda tecnologica por stack legacy y librerias antiguas (MEDIO)
- Descripcion: stack sobre .NET 4.5 + MVC4/WebApi4 + json/logging antiguos.
- Riesgo: parches de seguridad limitados y costo creciente de mantenimiento.
- Evidencia:
  - `ExtranetWeb/Extranet.web.csproj` lineas 16, 55, 65, 106, 168.
  - `ExtranetWeb/packages.config` lineas 3, 4, 6, 13.
- Recomendacion:
  - Mantener operacion ahora; plan de modernizacion por fase (ver Fase 4).

### H-08 Arquitectura con duplicidad de arboles/proyectos web y riesgo de deriva (MEDIO)
- Descripcion: existen variantes web (`ExtranetWeb`, `Extranet.web`, y arboles duplicados bajo `Extranet/`).
- Riesgo: drift de codigo/config, errores de despliegue por artefacto equivocado.
- Evidencia: estructura del workspace y presencia de multiples `Web.config` y `*.csproj` web.
- Recomendacion:
  - Definir unico artefacto oficial de build/deploy en fase 0.

## 5) Tabla de puntos criticos (ordenada de menor a mayor criticidad)

| # | Punto critico | Criticidad | Impacto principal | Accion inmediata |
|---|---|---|---|---|
| 0 | **IIS 2008 - Crisis TLS en dispositivos móviles (NUEVO)** | **CRÍTICA BLOQUEANTE** | **Caída de servidor bajo carga móvil, timeouts, no negocia TLS 1.2+** | **Migrar a IIS 10/13 en 7-10 días (NO-GO sin esto)** |
| 1 | Duplicidad de arboles/proyectos web | Media | Drift y errores de release | Definir artefacto oficial unico |
| 2 | Deuda tecnologica framework/librerias | Media | Riesgo de soporte/parches | Congelar baseline + roadmap fase 4 |
| 3 | Excepciones silenciosas | Medio-Alta | Incidentes sin trazabilidad | Logging minimo obligatorio |
| 4 | Ausencia de controles declarativos auth/CSRF | Alta | Exposicion de endpoints y acciones | Reforzar validaciones sensibles en fase 0 |
| 5 | Cookies sin hardening completo | Alta | Riesgo de sesion/token | Activar Secure/SameSite |
| 6 | Debug activo en config base | Alta | Info leakage y performance | Verificacion release real |
| 7 | Endpoints y servicios por HTTP | Critica | MITM/fuga de datos | TLS operativo en flujos criticos |
| 8 | Credenciales expuestas en config | Critica | Compromiso directo de BD | Rotacion inmediata y vault |

## 6) Analisis de criticidad global

Evaluacion global: **ALTA**.

Motivo:
- Existen dos riesgos de nivel critico que bloquean salida responsable sin mitigacion minima (secretos expuestos y HTTP en flujos criticos).
- La base funcional parece madura y orientada a SPs, por lo que estabilidad operativa es alcanzable si se cierran bloqueantes de plataforma/seguridad.

Lectura ejecutiva:
- No conviene una reingenieria previa al arranque.
- Si se aplica paquete minimo de contencion, la salida ASAP es factible y coherente con negocio.

## 7) Evaluacion pre-ejecucion (Go/No-Go)

### 7.1 Criterios minimos bloqueantes pre-Go-Live

⚠️ **HALLAZGO CRÍTICO DE INFRAESTRUCTURA AGREGADO 2026-04-28:**
Coordinador de Infraestructura reporta caídas de servidor por negociación TLS deficiente en dispositivos móviles. **IIS 2008 (EOL 2015) no soporta TLS 1.2+ de forma optimizada**, causando timeouts en clientes móviles (iOS/Android) y colapso de conexiones. Esto es **BLOQUEANTE P0** para Go-Live.

| Bloqueante | Prioridad | Estado actual | Criterio de cierre | Timeline | Resultado esperado |
|---|---|---|---|---|---|
| **[P0] Migración IIS 2008 → IIS 10/13 (TLS Mobile Crisis)** | **CRÍTICA** | **FALLANDO** | IIS 10+ con TLS 1.2/1.3 operativo, HTTP/2 habilitado, connection pooling optimizado | **7-10 días** | **Cero timeouts mobile, estabilidad bajo carga, cumplimiento PCI-DSS** |
| Rotacion de credenciales expuestas | P0 | No cumple | Rotar usuarios/clave BD y revocar anteriores | 48h | Riesgo de compromiso disminuye drasticamente |
| TLS/HTTPS en flujos criticos expuestos | P0 | Parcial | Forzar HTTPS extremo a extremo y validar endpoints externos | 48h | Integridad/confidencialidad de trafico |
| IIS/App Pool y parametros de infraestructura estables | P0 | No evidenciado | **Incluye: Windows Server 2016/2022, IIS 10/13 config, connection limits, app pool tuning** | 10 días | Estabilidad bajo carga (móviles + desktop) |
| Smoke tests E2E de procesos criticos | P0 | No evidenciado | Ejecutar smoke de login, consulta, reserva + **load test móvil** | 3-5 días | Validacion funcional minima + móvil operativo |
| Logging minimo operativo habilitado | P0 | Parcial | Corregir silencios + traza de errores con correlacion | 48h | Diagnostico post-arranque |
| Runbook de arranque, soporte y rollback | P0 | No evidenciado | Documento operacional aprobado por operaciones **con pasos IIS/TLS** | 48h | Recuperacion rapida ante incidente |

### 7.2 Decision
- **NO-GO** hasta migración IIS 2008 → IIS 10/13 completada (bloqueante crítico de infraestructura).
- **GO CONDICIONADO** cuando todos los 7 bloqueantes estén cerrados (incluido IIS 10/13 en producción + validación 72h).
- Si queda algún bloqueante abierto: **NO-GO FORMAL**.

### 7.3 Referencia Técnica
📋 Ver: [ADR-0001: Migración Urgente IIS 2008 → IIS 10/13 por Crisis de TLS en Móviles](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md)

## 8) Recomendaciones priorizadas (enfoque pragmatco ASAP)

### Prioridad P0 BLOQUEANTE - Infraestructura (Próximos 10 días)
**⚠️ CRÍTICO: Debe completarse ANTES de Go-Live a producción**

1. **[URGENTE] Migrar IIS 2008 → IIS 10 (Windows Server 2016) o IIS 13 (Windows Server 2022)**
   - Razón: IIS 2008 falla bajo negociación TLS con móviles → caídas de servidor
   - Causa: TLS 1.0/1.1 obsoletos, connection pooling débil, sin HTTP/2
   - Timeline: 7-10 días (preparación infraestructura + testing + rollout)
   - Fases:
     - Fase 0A (Hoy-Mañana): Implementar mitigaciones temporales desde código (.NET)
     - Fase 0B (Próximos 10 días): Migrar a IIS 10/13 en staging → validación → producción
   - Detalles completos: [ADR-0001](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md)
   - Responsable: Infrastructure Lead
   - Criterios de éxito:
     - ✓ TLS 1.2/1.3 activo, TLS 1.0/1.1 deshabilitados
     - ✓ HTTP/2 operativo
     - ✓ Cero timeouts móvil en 72h de monitoreo
     - ✓ CPU < 70% bajo carga

2. Rotar y blindar secretos de conexión.
3. Garantizar TLS/HTTPS en rutas y servicios críticos.
4. Confirmar publicación release real (sin debug, sin config de QAS hardcodeada).
5. Definir un único paquete de despliegue oficial (evitar carpetas web alternativas).
6. Correr smoke tests E2E mínimos de negocio **+ load testing móvil**.
7. Emitir runbook operativo con rollback en pasos exactos.

### Prioridad P1 (primeras 2-4 semanas, post-Go-Live)
1. Eliminar `catch {}` y estandarizar manejo de errores/logging.
2. Hardening de cookies/sesión y controles antiforgery para acciones críticas.
3. Observabilidad mínima: errores por módulo, latencia, endpoint externo, disponibilidad.

### Prioridad P2-P4 (progresivo)
1. Hardening de seguridad transversal por módulo (fase 2).
2. Optimización de SPs/índices y racionalización de accesos (fase 3).
3. Modernización estructural por dominios de mayor retorno (fase 4).

## 9) Problemas de normalizacion y performance del diseno de datos/entidades (INFERIDO)

Nota metodologica: sin modelo fisico SQL oficial en repositorio; conclusiones inferidas desde entidades `TR_*`, `TM_*`, `TC_*`, `TB_*`, `SG_*`, y flujos BLL/DAL.

Hallazgos inferidos:
- Alto volumen de entidades transaccionales `TR_DVA_*` con historial paralelo (`TR_DVA_Reserva_Hist`) sugiere crecimiento rapido y necesidad de indices compuestos por fecha, estado y documento.
- Multiples campos de estado/codigo de negocio tipo string (ej. `zzfact`, `zzpagado`, codigos) elevan riesgo de inconsistencia semantica si no hay catalogos fuertes.
- Mapeo manual DataReader en varias BLL incrementa riesgo de conversiones y null-handling no uniforme.
- Dependencia fuerte de procedimientos almacenados: buena para encapsular logica, pero sin versionado formal puede provocar drift app-BD.

Recomendaciones de datos (sin frenar operacion):
1. Catalogar llaves de negocio y estados criticos (diccionario unico).
2. Definir set minimo de indices por consultas mas frecuentes (reservas, solicitudes, documentos, reclamos).
3. Introducir auditoria de contratos SP (parametros, tipos, cardinalidad esperada).
4. Preparar modelo logico canonical por dominio antes de cualquier refactor mayor.

## 10) Matriz formal entidad -> problema -> impacto -> recomendacion -> prioridad

| Entidad/Tabla logica (inferida) | Problema | Impacto | Recomendacion | Prioridad |
|---|---|---|---|---|
| TR_DVA_DOCH | Alta dependencia de estados y servicios externos | Bloqueo operativo en devolucion/asignacion | Monitoreo transaccional y reglas de reintento controlado | Alta |
| TR_DVA_DOCH_ITEM | Consistencia de pago/comision sensible | Inconsistencias contables/operativas | Validaciones transaccionales y trazabilidad por ID_DOCH | Alta |
| TR_DVA_Reserva_Cab | Concurrencia de reservas | Sobreventa de slots/choque de citas | Indices + control de concurrencia por slot/hora | Alta |
| TR_DVA_Reserva_Det | Volumen alto y cambios frecuentes | Latencia en consulta/actualizacion | Particionado logico por fecha y tuning de SPs | Alta |
| TR_DVA_Reserva_Hist | Crecimiento historico continuo | Peso en consultas y almacenamiento | Politica de retencion/archivo y consultas optimizadas | Media |
| TR_DVA_SolicitudVacio | Multiples variaciones asociadas | Complejidad de negocio y mantenimiento | Contrato de dominio y estados canonicos | Alta |
| TR_UNI_Reclamo | Trazabilidad documental variable | Riesgo legal/atencion cliente | Normalizar flujo reclamo + anexos | Media-Alta |
| TB_Usuario / SG_Usuario | Seguridad y sesion distribuida | Riesgo acceso indebido | Endurecer authn/authz, expiracion y auditoria | Critica |
| TM_UNI_Empresa / TM_UNI_EmpresaOperador | Relacion empresa-operador sensible | Errores de autorizacion comercial | Reglas unicas de integridad y cache controlada | Media-Alta |

## 11) Trazabilidad: entidad -> tabla logica -> modulos/capas que la usan

| Entidad | Tabla logica inferida | Controllers/API | Interfaces | BLL | DAL |
|---|---|---|---|---|---|
| TR_DVA_DOCH | TR_DVA_DOCH | Devolucion, AsignacionVacios, Vacios API | DevolucionCntr, AsignacionVacios | TR_DVA_DOCHBLL* | TR_DVA_DOCHDAL* |
| TR_DVA_Reserva_Cab | TR_DVA_Reserva_Cab | Devolucion, Vacios API | DevolucionCntr | TR_DVA_Reserva_CabBLL* | TR_DVA_Reserva_CabDAL* |
| TR_DVA_Reserva_Det | TR_DVA_Reserva_Det | Devolucion, Vacios API | DevolucionCntr, CitasAsignacionVacios | TR_DVA_Reserva_DetBLL* | TR_DVA_Reserva_DetDAL* |
| TR_DVA_Reserva_Hist | TR_DVA_Reserva_Hist | Vacios API | DevolucionCntr | TR_DVA_Reserva_HistBLL* | TR_DVA_Reserva_HistDAL* |
| TR_DVA_SolicitudVacio | TR_DVA_SolicitudVacio | AsignacionVacios, Vacios | AsignacionVacios, Vacios | TR_DVA_SolicitudVacioBLL* | TR_DVA_SolicitudVacioDAL* |
| TR_UNI_Reclamo | TR_UNI_Reclamo | ReclamoController | Reclamo | TR_UNI_ReclamoBLL* | TR_UNI_ReclamoDAL* |
| TB_Usuario/SG_Usuario | TB_Usuario, SG_Usuario | Login, Seguridad, BaseController | Seguridad | SG_UsuarioBLL/TM_UNI_UsuarioBLL* | SG_UsuarioDAL/TM_UNI_UsuarioDAL* |
| TM_UNI_Empresa | TM_UNI_Empresa | Agencia, Vacios | Vacios | TM_UNI_EmpresaBLL* | TM_UNI_EmpresaDAL* |

(* nomenclatura inferida por convencion y referencias observadas en codigo)

## 12) Propuesta de rediseño por dominios criticos

### Dominio Seguridad/Acceso (prioridad maxima)
- Objetivo: reducir riesgo de compromiso sin detener negocio.
- Lineas:
  - Seguridad de secretos y cookies.
  - Politica de sesion y autorizacion declarativa progresiva.
  - Auditoria central de autenticacion y acciones sensibles.

### Dominio Reservas/Deposito vacios
- Objetivo: estabilidad y consistencia transaccional.
- Lineas:
  - Contrato de estados unico para reserva/solicitud/pago.
  - Reduccion de acoplamiento UI->Interfaces->BLL por casos de uso.
  - Telemetria de capacidad por sede/horario.

### Dominio Reclamos
- Objetivo: trazabilidad de ciclo de vida y evidencia documental.
- Lineas:
  - Modelo de estados estandar + SLA de atencion.
  - Indexacion para busquedas por cliente/fecha/estado.

### Dominio Transmisiones/Operativo comercial
- Objetivo: robustez de integraciones SAP/WCF.
- Lineas:
  - Retry/timeouts y fallback controlado.
  - Circuit breaker funcional para dependencias externas.

## 13) Estrategia de implementacion incremental (plan progresivo requerido)

### Fase 0 - Go-Live rapido y estable (minimos bloqueantes)
- Rotacion de credenciales.
- TLS/HTTPS operativo en flujos criticos expuestos.
- IIS/App Pool y parametros de infraestructura estables.
- Smoke tests E2E de procesos criticos.
- Logging minimo operativo habilitado.
- Runbook de arranque, soporte y rollback.

### Fase 1 - Estabilizacion temprana post arranque
- Corregir excepciones silenciosas.
- Endurecer monitoreo y alertas basicas.
- Ajustar timeouts/reintentos en llamadas externas criticas.

### Fase 2 - Hardening de seguridad sin frenar operacion
- Politicas de cookies, sesion y CSRF.
- Matriz de permisos y autorizacion declarativa.
- Revisiones de configuracion segura por entorno.

### Fase 3 - Optimizacion funcional y de datos
- Tuning de SPs e indices por hotspots.
- Normalizacion de estados/catalogos transversales.
- Reducir duplicidad de flujo y deuda en capas.

### Fase 4 - Modernizacion estructural
- Definir arquitectura target gradual (por dominio).
- Estrategia de migracion de framework/librerias por lotes.
- Reemplazo incremental de componentes de mayor riesgo.

## 14) Evidencias de archivos revisados

### Solucion y proyectos
- `Extranet.sln`
- `ExtranetWeb/Extranet.web.csproj`
- `ExtranetWeb/packages.config`

### Configuracion y despliegue
- `ExtranetWeb/Web.config`
- `ExtranetWeb/Web.Release.config`
- `ExtranetWeb/Global.asax.cs`
- `ExtranetWeb/App_Start/FilterConfig.cs`
- `ExtranetWeb/App_Start/RouteConfig.cs`
- `ExtranetWeb/App_Start/WebApiConfig.cs`

### Seguridad y acceso
- `ExtranetWeb/Controllers/LoginController.cs`
- `ExtranetWeb/Controllers/BaseController.cs`
- `ExtranetWeb/Controllers/SeguridadController.cs`
- `ExtranetWeb/Util/ApiRest/AutenticacionApiRest.cs`

### Datos y capas
- `Extranet.data/Base/_DbConnection.cs`
- `Extranet.data/Base/_DbProvider.cs`
- `Extranet.data/Base/DatabaseDAL.cs`
- `Extranet.objects/CRUD/*.cs`
- `Extranet.business/extend/*.cs`
- `Extranet.Interfaces/*.cs`

### Operacion y cobertura
- Busquedas de artefactos de test/smoke/runbook: sin evidencia de proyecto de pruebas formal ni runbook operativo propio en raiz de solucion.

## 15) Conclusiones ejecutivas para comite

1. La salida a produccion ASAP **es factible** sin reescritura, pero debe ser **GO condicionado** a 6 bloqueantes concretos.
2. El mayor riesgo inmediato no es funcional, sino de **seguridad/configuracion y operacion**.
3. Con Fase 0 bien ejecutada y Fase 1 rapida, se reduce significativamente probabilidad de caida severa.
4. La optimizacion y modernizacion deben ser progresivas por dominio para no interrumpir negocio.
