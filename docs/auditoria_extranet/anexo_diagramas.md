# Anexo de Diagramas - SIS_INTRANET (equivalente Extranet)

Nota: los modelos E/R y tablas son inferidos desde entidades y capas de codigo, al no encontrarse modelo fisico SQL oficial en el repositorio.

Navegacion:
- [Resumen ejecutivo e indice](resumen_ejecutivo_indice.md)
- [Informe tecnico principal](informe_tecnico.md)
- [Resumen visual ejecutivo](resumen_visual_ejecutivo.md)

## 1) UML - Class Diagram (vista simplificada por capas)

```plantuml
@startuml
class ExtranetWebController {
  +ActionResult Login()
  +JsonResult ValidarUsuario()
}

class InterfacesFacade {
  +Seguridad.autenticar()
  +AsignacionVacios.*
  +DevolucionCntr.*
}

class BusinessBLL {
  +TR_DVA_DOCHBLL
  +TR_DVA_Reserva_CabBLL
  +TR_DVA_Reserva_DetBLL
  +TR_UNI_ReclamoBLL
}

class DataDAL {
  +TR_DVA_DOCHDAL
  +TR_DVA_Reserva_CabDAL
  +TR_DVA_Reserva_DetDAL
  +TR_UNI_ReclamoDAL
  +ExecuteReader()
}

class Entities {
  +TR_DVA_DOCH
  +TR_DVA_Reserva_Cab
  +TR_DVA_Reserva_Det
  +TR_UNI_Reclamo
  +TB_Usuario
}

ExtranetWebController --> InterfacesFacade
InterfacesFacade --> BusinessBLL
BusinessBLL --> DataDAL
BusinessBLL --> Entities
DataDAL --> Entities
@enduml
```

## 2) UML - Sequence Diagram (login y sesion)

```plantuml
@startuml
autonumber
actor U as Usuario
participant C as LoginController
participant I as Interfaces.Seguridad
participant A as AutenticacionApiRest
participant API as PortalWebApi

U -> C: POST ValidarUsuario(usuario,password)
C -> I: autenticar(usuario,password,ip,navegador)
I --> C: resp (ok/error)
alt resp > 0
  C -> A: CrearCookieAutenticacion()
  A -> API: POST Seguridad/login
  API --> A: Cookies X-Auth-Cookie/X-Refresh-Token
  A --> C: DatosUsuarioAplicacion
  C --> U: JSON resp=true
else resp <= 0
  C --> U: JSON resp=false + mensaje
end
@enduml
```

## 3) UML - Flowchart (Go-Live ASAP condicionado)

```plantuml
@startuml
start
:Inicio ventana Go-Live ASAP;
if (6 bloqueantes cerrados?) then (No)
  :NO-GO: cerrar bloqueantes y revalidar;
  end
else (Si)
  :GO CONDICIONADO;
  :Fase 1: estabilizacion temprana;
  :Fase 2: hardening seguridad;
  :Fase 3: optimizacion funcional y datos;
  :Fase 4: modernizacion estructural;
  end
endif
@enduml
```

## 4) C4 - Context

```plantuml
@startuml C4Context
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml
title Contexto - SIS_INTRANET (Extranet)

Person(usuario, "Usuario Externo/Interno", "Opera reservas, documentos y reclamos")
System(sis, "SIS_INTRANET/Extranet", "Portal web MVC + Web API")
System_Ext(db, "SQL Server", "BD_EXTRANET / BD_INTRANET")
System_Ext(wcf, "Servicios WCF SAP/Internos", "Integraciones operativas")
System_Ext(api, "PortalWebApi", "Autenticacion y menu")

Rel(usuario, sis, "Usa", "HTTPS")
Rel(sis, db, "Consulta/actualiza", "T-SQL/SP")
Rel(sis, wcf, "Consume", "HTTP/HTTPS")
Rel(sis, api, "Autentica y consume menu", "HTTP/HTTPS")

@enduml
```

## 5) C4 - Container

```plantuml
@startuml C4Container
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
title Contenedores - SIS_INTRANET/Extranet

Person(usuario, "Usuario")
System_Boundary(s1, "Extranet") {
  Container(web, "ExtranetWeb", "ASP.NET MVC 4 + Web API 4", "UI y endpoints API")
  Container(interfaces, "Extranet.Interfaces", ".NET Class Library", "Fachadas de negocio")
  Container(bll, "Extranet.business", ".NET Class Library", "Reglas y orquestacion")
  Container(dal, "Extranet.data", ".NET Class Library", "Acceso a datos via SP")
  ContainerDb(db, "SQL Server", "BD_EXTRANET / BD_INTRANET", "Datos operativos")
}

Rel(usuario, web, "Navega/Opera", "HTTPS")
Rel(web, interfaces, "Invoca")
Rel(interfaces, bll, "Invoca")
Rel(bll, dal, "Invoca")
Rel(dal, db, "CRUD/SP", "T-SQL")

@enduml
```

## 6) Glosario de tablas principales (inferido)

| Tabla logica | Descripcion inferida |
|---|---|
| TR_DVA_DOCH | Cabecera documental operativa |
| TR_DVA_DOCH_ITEM | Detalle/posiciones asociadas a documento |
| TR_DVA_Reserva_Cab | Cabecera de reserva de cita/atencion |
| TR_DVA_Reserva_Det | Detalle de reserva por contenedor/slot |
| TR_DVA_Reserva_Hist | Historico de cambios de reserva |
| TR_DVA_SolicitudVacio | Solicitud principal de vacios |
| TR_DVA_SolicitudVacioDetalle | Detalle de solicitud de vacios |
| TR_DVA_SolicitudVacioDoc | Adjuntos/documentos de solicitud |
| TR_UNI_Reclamo | Reclamo principal |
| TR_UNI_Reclamo_Doc | Documentos de reclamo |
| TM_UNI_Empresa | Maestro de empresa |
| SG_Usuario / TB_Usuario | Seguridad/acceso de usuario |

## 7) Modelo E/R general (inferido)

```plantuml
@startuml
entity TM_UNI_EMPRESA {
  *ID_EMPRESA : integer
  --
  CT_RAZON_SOCIAL : string
  CT_RUC : string
}

entity TM_UNI_EMPRESAOPERADOR {
  *ID_OPERADOR : integer
  ID_EMPRESA : integer (FK)
}

entity TB_USUARIO {
  *ctUsuario : string
  *cnIdPersona : integer
  --
  CT_EMAIL : string
  CT_ESTADO : string
}

entity TR_DVA_DOCH {
  *ID_DOCH : integer
  ctNumeroBL : string
  cnIdPersona : integer (FK)
  --
  CT_ESTADO : string
  FE_REGISTRO : date
}

entity TR_DVA_DOCH_ITEM {
  *ID_DOCT : integer
  ID_DOCH : integer (FK)
  --
  CT_POSICION : string
  DU_IMPORTE : decimal
}

entity TR_DVA_RESERVA_CAB {
  *cnID_ReservaCab : integer
  ID_DOCH : integer (FK)
  --
  FE_RESERVA : date
}

entity TR_DVA_RESERVA_DET {
  *cnID_ReservaDet : integer
  cnID_ReservaCab : integer (FK)
  --
  FE_HORA_RESERVA : datetime
  CT_ESTADO : string
}

entity TR_DVA_RESERVA_HIST {
  *ID_HISTORICO : integer
  cnID_ReservaDet : integer (FK)
}

entity TR_DVA_SOLICITUDVACIO {
  *ID_SOLICITUD : integer
  ID_DOCH : integer (FK)
}

entity TR_DVA_SOLICITUDVACIODETALLE {
  *ID_DETALLE : integer
  ID_SOLICITUD : integer (FK)
}

entity TR_DVA_SOLICITUDVACIODOC {
  *ID_DOC : integer
  ID_SOLICITUD : integer (FK)
}

entity TR_UNI_RECLAMO {
  *ID_RECLAMO : integer
  cnIdPersona : integer (FK)
}

entity TR_UNI_RECLAMO_DOC {
  *ID_RECLAMO_DOC : integer
  ID_RECLAMO : integer (FK)
}

TM_UNI_EMPRESA ||--o{ TM_UNI_EMPRESAOPERADOR : posee
TB_USUARIO ||--o{ TR_DVA_DOCH : registra
TR_DVA_DOCH ||--o{ TR_DVA_DOCH_ITEM : contiene
TR_DVA_DOCH ||--o{ TR_DVA_RESERVA_CAB : origina
TR_DVA_RESERVA_CAB ||--o{ TR_DVA_RESERVA_DET : detalla
TR_DVA_RESERVA_DET ||--o{ TR_DVA_RESERVA_HIST : historiza
TR_DVA_DOCH ||--o{ TR_DVA_SOLICITUDVACIO : relaciona
TR_DVA_SOLICITUDVACIO ||--o{ TR_DVA_SOLICITUDVACIODETALLE : detalla
TR_DVA_SOLICITUDVACIO ||--o{ TR_DVA_SOLICITUDVACIODOC : adjunta
TB_USUARIO ||--o{ TR_UNI_RECLAMO : crea
TR_UNI_RECLAMO ||--o{ TR_UNI_RECLAMO_DOC : adjunta
@enduml
```

## 8) E/R por dominio

### 8.1 Transmisiones

```plantuml
@startuml
entity TR_DVA_DOCH {
  *ID_DOCH : integer
}
entity TR_DVA_DOCP {
  *ID_DOCP : integer
  ID_DOCH : integer (FK)
}
entity TR_DVA_DOCH_ITEM {
  *ID_DOCT : integer
  ID_DOCH : integer (FK)
}
entity TR_DVA_DOPL {
  *ID_DOPL : integer
  ID_DOCH : integer (FK)
}
entity TR_DVA_DSER {
  *ID_DSER : integer
  ID_DOPL : integer (FK)
}
TR_DVA_DOCH ||--o{ TR_DVA_DOCP : planifica
TR_DVA_DOCH ||--o{ TR_DVA_DOCH_ITEM : contiene
TR_DVA_DOCH ||--o{ TR_DVA_DOPL : opera
TR_DVA_DOPL ||--o{ TR_DVA_DSER : servicios
@enduml
```

### 8.2 Operativo/Comercial

```plantuml
@startuml
entity TR_DVA_DOCH {
  *ID_DOCH : integer
}
entity TR_DVA_RESERVA_CAB {
  *cnID_ReservaCab : integer
  ID_DOCH : integer (FK)
}
entity TR_DVA_RESERVA_DET {
  *cnID_ReservaDet : integer
  cnID_ReservaCab : integer (FK)
}
entity TR_DVA_RESERVA_HIST {
  *ID_HISTORICO : integer
  cnID_ReservaDet : integer (FK)
}
entity TC_DVA_PARAMETROS_PROCESO {
  *ID_PARAMETRO : integer
}
entity TC_DVA_OPERACIONES_HORARIO {
  *ID_HORARIO : integer
  ID_PARAMETRO : integer (FK)
}
entity TC_DVA_OPERACIONES_HORARIO_DET {
  *ID_HORARIO_DET : integer
  ID_HORARIO : integer (FK)
}
TR_DVA_DOCH ||--o{ TR_DVA_RESERVA_CAB : genera
TR_DVA_RESERVA_CAB ||--o{ TR_DVA_RESERVA_DET : detalle
TR_DVA_RESERVA_DET ||--o{ TR_DVA_RESERVA_HIST : cambios
TC_DVA_PARAMETROS_PROCESO ||--o{ TC_DVA_OPERACIONES_HORARIO : define
TC_DVA_OPERACIONES_HORARIO ||--o{ TC_DVA_OPERACIONES_HORARIO_DET : descompone
@enduml
```

### 8.3 Seguridad/Acceso

```plantuml
@startuml
entity SG_USUARIO {
  *ID_USUARIO : integer
}
entity TB_USUARIO {
  *ctUsuario : string
  SG_USUARIO_ID : integer (FK)
}
entity TB_MENU {
  *ID_MENU : integer
  ctUsuario : string (FK)
}
entity TM_UNI_PERSONA {
  *cnIdPersona : integer
}
entity TM_UNI_EMPRESA {
  *ID_EMPRESA : integer
}
SG_USUARIO ||--|| TB_USUARIO : representa
TB_USUARIO ||--o{ TB_MENU : accede
TB_USUARIO }o--|| TM_UNI_PERSONA : referencia
TB_USUARIO }o--|| TM_UNI_EMPRESA : patrocina
@enduml
```

### 8.4 Reclamos

```plantuml
@startuml
entity TR_UNI_RECLAMO {
  *ID_RECLAMO : integer
  ctUsuario : string (FK)
  ID_MOTIVO : integer (FK)
}
entity TR_UNI_RECLAMO_DOC {
  *ID_RECLAMO_DOC : integer
  ID_RECLAMO : integer (FK)
}
entity TM_UNI_MOTIVORECLAMO {
  *ID_MOTIVO : integer
  CT_DESCRIPCION : string
}
entity TB_USUARIO {
  *ctUsuario : string
}
TR_UNI_RECLAMO ||--o{ TR_UNI_RECLAMO_DOC : adjunta
TM_UNI_MOTIVORECLAMO ||--o{ TR_UNI_RECLAMO : clasifica
TB_USUARIO ||--o{ TR_UNI_RECLAMO : registra
@enduml
```

### 8.5 Visitas/Reservas

```plantuml
@startuml
entity TR_DVA_RESERVA_CAB {
  *cnID_ReservaCab : integer
}
entity TR_DVA_RESERVA_DET {
  *cnID_ReservaDet : integer
  cnID_ReservaCab : integer (FK)
  ID_ESTADO : integer (FK)
}
entity TR_DVA_RESERVA_HIST {
  *ID_HISTORICO : integer
  cnID_ReservaDet : integer (FK)
}
entity TC_DVA_ESTADODEVOLUCION {
  *ID_ESTADO : integer
  CT_ESTADO : string
}
TR_DVA_RESERVA_CAB ||--o{ TR_DVA_RESERVA_DET : contiene
TR_DVA_RESERVA_DET ||--o{ TR_DVA_RESERVA_HIST : audita
TC_DVA_ESTADODEVOLUCION ||--o{ TR_DVA_RESERVA_DET : estado
@enduml
```

### 8.6 Deposito Vacios

```plantuml
@startuml
entity TR_DVA_SOLICITUDVACIO {
  *ID_SOLICITUD : integer
}
entity TR_DVA_SOLICITUDVACIODETALLE {
  *ID_DETALLE : integer
  ID_SOLICITUD : integer (FK)
}
entity TR_DVA_SOLICITUDVACIODOC {
  *ID_DOC : integer
  ID_SOLICITUD : integer (FK)
}
entity TR_DVA_SOLICITUDVACIOSERVICIOS {
  *ID_SERVICIO : integer
  ID_SOLICITUD : integer (FK)
}
entity TR_DVA_SOLICITUDVACIOPLAN {
  *ID_PLAN : integer
  ID_SOLICITUD : integer (FK)
}
TR_DVA_SOLICITUDVACIO ||--o{ TR_DVA_SOLICITUDVACIODETALLE : detalle
TR_DVA_SOLICITUDVACIO ||--o{ TR_DVA_SOLICITUDVACIODOC : documento
TR_DVA_SOLICITUDVACIO ||--o{ TR_DVA_SOLICITUDVACIOSERVICIOS : servicio
TR_DVA_SOLICITUDVACIO ||--o{ TR_DVA_SOLICITUDVACIOPLAN : plan
@enduml
```

## 9) Diccionario de datos resumido (entidades clave, inferido)

| Entidad | Clave probable | Campos relevantes (observados por convencion) | Uso de negocio |
|---|---|---|---|
| TR_DVA_DOCH | ID_DOCH | ctNumeroBL, estados, usuario/terminal, fechas | Cabecera de proceso operativo |
| TR_DVA_DOCH_ITEM | ID_DOCT | importes, material, posicion, estado pago | Detalle transaccional y comisiones |
| TR_DVA_Reserva_Cab | cnID_ReservaCab | ID_DOCH, ctNumeroBL, fecha reserva, cliente | Reserva cabecera |
| TR_DVA_Reserva_Det | cnID_ReservaDet | fecha/hora reserva, tipo horario, contenedor, estado | Ejecucion de cita |
| TR_DVA_Reserva_Hist | (id historico) | fecha anterior/nueva, usuario, terminal, tipo horario | Auditoria de cambios |
| TR_DVA_SolicitudVacio | (id solicitud) | datos solicitud, operador, estado | Solicitud de vacios |
| TR_UNI_Reclamo | (id reclamo) | motivo, estado, usuario, fechas | Gestion de reclamos |
| TB_Usuario | cnIdPersona/ctUsuario | correo, tipo usuario, estado proceso | Identidad operativa |

## 10) Nota sobre visualización de diagramas

**Diagramas convertidos a PlantUML (gratuito y libre):**

Todos los diagramas en este documento utilizan la sintaxis **PlantUML**, que es:
- ✅ 100% gratuito y open-source
- ✅ Soportado en múltiples plataformas y herramientas
- ✅ Compatible con editores online

**Formas de visualizar los diagramas PlantUML:**

1. **Online (recomendado - sin instalación):**
   - [PlantUML Online Editor](https://www.plantuml.com/plantuml/uml/)
   - Copia y pega cualquier bloque `@startuml...@enduml` y presiona "Submit"

2. **En VS Code (extensión gratuita):**
   - Instala: "PlantUML" de jebbs
   - Abre archivo .md con diagrama y presiona Alt+D para preview

3. **Línea de comandos (Docker):**
   ```bash
   docker run --rm -v /ruta/al/archivo:/data plantuml /data/archivo.md
   ```

4. **GitHub (automático):**
   - Los bloques PlantUML se renderizan automáticamente en README.md y archivos .md de GitHub