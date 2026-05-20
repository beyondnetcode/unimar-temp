# Anexo de Diagramas de Arquitectura (PlantUML)

Documento relacionado al informe principal de auditoria:

- Ver resumen e indice: [RESUMEN_EJECUTIVO_E_INDICE.md](RESUMEN_EJECUTIVO_E_INDICE.md)
- Ver informe: [AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md](AUDITORIA_SOLUCION_SIS_INTRANET_2026-04-28.md)
- Ver resumen visual: [RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md](RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md)

**Nota sobre PlantUML:** Todos los diagramas están en formato PlantUML (gratuito y de código abierto).
- Visualización en VS Code: instala la extensión "PlantUML" (gratuita)
- Editor online: http://www.plantuml.com/plantuml/uml/
- Generación de imágenes: `plantuml -o output diagram.md`

## Indice

1. [Vista de capas y dependencias (UML - Class Diagram)](#1-vista-de-capas-y-dependencias-uml---class-diagram)
2. [Flujo de autenticacion web (UML - Sequence Diagram)](#2-flujo-de-autenticacion-web-uml---sequence-diagram)
3. [Flujo de solicitud HTTP en la aplicacion (UML - Activity Diagram)](#3-flujo-de-solicitud-http-en-la-aplicacion-uml---activity-diagram)
4. [C4 - Contexto del sistema](#4-c4---contexto-del-sistema)
5. [C4 - Contenedores principales](#5-c4---contenedores-principales)
6. [Glosario de tablas principales](#6-glosario-de-tablas-principales)
7. [Modelo E/R - Dominio de transmisiones](#7-modelo-er---dominio-de-transmisiones)
8. [Modelo E/R - Dominio operativo y comercial](#8-modelo-er---dominio-operativo-y-comercial)
9. [Modelo E/R - Dominio de seguridad y acceso](#9-modelo-er---dominio-de-seguridad-y-acceso)
10. [Modelo E/R - Dominio de reclamos](#10-modelo-er---dominio-de-reclamos)
11. [Modelo E/R - Dominio de visitas y reservas](#11-modelo-er---dominio-de-visitas-y-reservas)
12. [Modelo E/R - Dominio deposito vacios](#12-modelo-er---dominio-deposito-vacios)

## Indice visual del anexo

```plantuml
@startuml
skinparam backgroundColor #F5F5F5
skinparam defaultFontSize 10

rectangle "Anexo de Diagramas" as A #87CEEB
rectangle "Class Diagram" as U1 #B0E0E6
rectangle "Sequence Diagram" as U2 #B0E0E6
rectangle "Activity Diagram" as U3 #B0E0E6
rectangle "C4 Contexto" as C1 #B0E0E6
rectangle "C4 Contenedores" as C2 #B0E0E6
rectangle "Glosario" as G #D3D3D3
rectangle "ER Transmisiones" as E1 #DDA0DD
rectangle "ER Operativo" as E2 #DDA0DD
rectangle "ER Seguridad" as E3 #DDA0DD
rectangle "ER Reclamos" as E4 #DDA0DD
rectangle "ER Visitas" as E5 #DDA0DD
rectangle "ER DVA" as E6 #DDA0DD

A --> U1
A --> U2
A --> U3
A --> C1
A --> C2
A --> G
A --> E1
A --> E2
A --> E3
A --> E4
A --> E5
A --> E6
@enduml
```

## 1) Vista de capas y dependencias (UML - Class Diagram)

```plantuml
@startuml
!theme plain
skinparam backgroundColor #FEFEFE
skinparam classBackgroundColor #F0F8FF
skinparam classBorderColor #333

class ClienteWeb {
  +Navegador
  +Usuario Final
}

class SIS_INTRANET_Web {
  +Controllers
  +Views
  +Apis
  +App_Start
}

class INTRANET_business {
  +Reglas de negocio
  +Orquestacion
}

class INTRANET_data {
  +Acceso a datos
  +Stored Procedures
}

class INTRANET_objects {
  +Entidades
  +DTOs
}

class INTRANET_Interfaces {
  +Integraciones
  +Control de accesos
  +Repositorio de envios
}

class IntranetService {
  +Proxy SOAP/WCF
  +Servicios externos
}

class SQL_Server {
  +BD_INTRANET
  +BD_EXTRANET
  +BD_BI
}

class WCF_Externos {
  +ws_extranet/wcf.svc
  +wcfAgencia.svc
}

ClienteWeb --> SIS_INTRANET_Web : HTTP(S)
SIS_INTRANET_Web --> INTRANET_business : usa
SIS_INTRANET_Web --> INTRANET_Interfaces : usa
SIS_INTRANET_Web --> INTRANET_objects : usa
SIS_INTRANET_Web --> IntranetService : usa

INTRANET_business --> INTRANET_data : usa
INTRANET_business --> INTRANET_objects : usa
INTRANET_data --> INTRANET_objects : usa
INTRANET_data --> SQL_Server : ADO.NET/SP

INTRANET_Interfaces --> INTRANET_business : usa
INTRANET_Interfaces --> IntranetService : usa
IntranetService --> WCF_Externos : SOAP/WCF
@enduml
```

## 2) Flujo de autenticacion web (UML - Sequence Diagram)

```plantuml
@startuml
skinparam backgroundColor #FEFEFE
autonumber

actor Usuario as U
participant Browser as B
participant LoginController as L
participant Seguridad as S
participant AutenticacionAPI as A
participant Session as G

U -> B: Envia credenciales
B -> L: POST /Login/ValidarUsuario(usuario, password)
L -> S: obtenerDatosUsuario(usuario)
L -> S: autenticar(usuario, password, ip, navegador, appId)

alt autenticacion exitosa
  S --> L: resp > 0
  L -> A: CrearCookieAutenticacion(usuario, password)
  L -> G: crearSesion(usuario, resp)
  L --> B: Json(resp=true, notice="")
else autenticacion fallida
  S --> L: resp <= 0
  L --> B: Json(resp=false, notice=mensaje)
end
@enduml
```

## 3) Flujo de solicitud HTTP en la aplicacion (UML - Activity Diagram)

```plantuml
@startuml
skinparam backgroundColor #FEFEFE
start

:Cliente Web;
:Ruteo MVC/WebAPI;

if (Sesion activa?) then (No)
  :Redirigir a Login;
else (Si)
  if (Acceso permitido por menu/filtro?) then (No)
    :401 o redireccion a inicio;
  else (Si)
    :Controller Action;
    :Business Layer;
    :Data Layer;
    fork
      :SQL Server;
    fork again
      :WCF/REST externos;
    endfork
    :Respuesta de datos;
    :View o JsonResult;
  endif
endif

:Respuesta al Cliente;
stop
@enduml
```

## 4) C4 - Contexto del sistema

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

SHOW_PERSON_OUTLINE()

Person(usuario, "Usuario Intranet", "Opera modulos de negocio y consultas")
Person(movil, "Usuario Movil", "Usa funcionalidades moviles")

System(sis, "SIS_INTRANET", "Aplicacion web ASP.NET MVC/Web API para procesos operativos")

System_Ext(bd, "BD_INTRANET/BD_EXTRANET/BD_BI", "SQL Server - Persistencia")
System_Ext(wsSeg, "Servicios WCF Externos", "Autenticacion, menu, servicios de negocio")
System_Ext(apiRest, "API REST Externa", "Servicios complementarios")

Rel(usuario, sis, "Usa", "HTTPS")
Rel(movil, sis, "Usa", "HTTPS")
Rel(sis, bd, "Lee/Escribe", "ADO.NET/SP")
Rel(sis, wsSeg, "Consume", "SOAP/WCF")
Rel(sis, apiRest, "Consume", "HTTP/HTTPS")

SHOW_LEGEND()
@enduml
```

## 5) C4 - Contenedores principales

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

SHOW_PERSON_OUTLINE()

Person(usuario, "Usuario", "Usuario de intranet")

System_Boundary(c1, "SIS_INTRANET") {
  Container(web, "SIS_INTRANET Web", "ASP.NET MVC 4 / Web API 4", "UI, controllers, endpoints HTTP")
  Container(bll, "INTRANET.business", ".NET Framework 4.5", "Reglas de negocio y orquestacion")
  Container(dal, "INTRANET.data", ".NET Framework 4.5", "Acceso a datos ADO.NET y SP")
  Container(intf, "INTRANET.Interfaces", ".NET Framework 4.5", "Integraciones y utilidades")
  Container(svc, "IntranetService", ".NET Framework 4.5", "Clientes SOAP/WCF")
}

ContainerDb(sql, "SQL Server", "MSSQL", "Persistencia operativa")
System_Ext(wcf, "Servicios WCF", "Sistemas externos")

Rel(usuario, web, "Usa", "HTTPS")
Rel(web, bll, "Invoca")
Rel(web, intf, "Invoca")
Rel(web, svc, "Invoca")
Rel(bll, dal, "Invoca")
Rel(dal, sql, "Lee/Escribe", "ADO.NET/SP")
Rel(intf, svc, "Invoca")
Rel(svc, wcf, "Consume", "SOAP/WCF")

SHOW_LEGEND()
@enduml
```

## 6) Glosario de tablas principales

Nota: este glosario es inferido desde las clases de persistencia ubicadas en INTRANET.objects. No reemplaza un diccionario fisico oficial de base de datos, pero refleja las tablas mas relevantes observadas en el codigo.

| Tabla | Modulo | Proposito funcional | PK principal | Relaciones relevantes |
|---|---|---|---|---|
| TR_UNI_Manifiesto | Intranet | Cabecera de manifiestos para transmisiones aduaneras/logisticas | cnID_Manifiesto | 1:N con TR_UNI_Transmision |
| TR_UNI_Transmision | Intranet | Registro central de una transmision enviada/recibida | cnID_Transmision | N:1 con TR_UNI_Manifiesto; 1:N con DAM, contenedores y mensajes |
| TR_UNI_Archivo | Intranet | Almacenamiento de archivo binario de envio/respuesta | cnID_Archivo | Referenciado por TR_UNI_Transmision y TR_UNI_Respuesta |
| TR_UNI_Respuesta | Intranet | Resultado funcional de una transmision | cnID_Respuesta | N:1 con TR_UNI_Transmision; N:1 con TR_UNI_Archivo |
| TR_UNI_ContenedorTransmision | Intranet | Contenedores asociados a una transmision | cnID_ContenedorTransmision | N:1 con TR_UNI_Transmision |
| TR_UNI_PrecintoContenedor | Intranet | Precintos por contenedor transmitido | cnID_PrecintoContenedor | N:1 con TR_UNI_ContenedorTransmision |
| TR_UNI_DAMTransmision | Intranet | DAMs vinculadas a una transmision | cnID_DAMTransmision | N:1 con TR_UNI_Transmision |
| TR_UNI_DocumentoTransmision | Intranet | Documentos asociados a una DAM/transmision | cnID_DocumentoTransmision | N:1 con TR_UNI_DAMTransmision; N:1 con TR_UNI_Transmision |
| TR_UNI_MensajeTransmision | Intranet | Mensajes/errores/advertencias producidos por la transmision | cnID_MensajeTransmision | N:1 con TR_UNI_Transmision |
| TR_UNI_UsuariosSeg | Intranet | Datos de seguridad/usuarios de la aplicacion | no visible en lectura actual | Usada por autenticacion y permisos |
| TM_UNI_Usuario | Intranet | Maestro de usuarios internos | no visible en lectura actual | Relacionable con areas/gerencias/centros |
| TM_UNI_Area | Intranet | Maestro de areas organizacionales | no visible en lectura actual | Relacion organizacional |
| TM_UNI_Gerencia | Intranet | Maestro de gerencias | no visible en lectura actual | Relacion organizacional |
| TM_UNI_Centros | Intranet | Maestro de centros | no visible en lectura actual | Relacion organizacional |
| TM_UNI_Persona | Extranet | Maestro de persona/visitante/actor del proceso | no visible en lectura actual | Asociada a reclamos, reservas y movimientos |
| TM_UNI_Empresa | Extranet | Maestro de empresas/razon social | no visible en lectura actual | Asociada a personas y operaciones |
| TR_UNI_Reclamo | Extranet | Cabecera de reclamos operativos/comerciales | no visible en lectura actual | 1:N con TR_UNI_Reclamo_Doc |
| TR_UNI_Reclamo_Doc | Extranet | Adjuntos/documentos de reclamo | no visible en lectura actual | N:1 con TR_UNI_Reclamo |
| TR_UNI_MovimientoCab | Extranet | Cabecera de movimiento logistico | no visible en lectura actual | 1:N con detalle y contenedores |
| TR_UNI_MovimientoDet | Extranet | Detalle de movimiento logistico | no visible en lectura actual | N:1 con TR_UNI_MovimientoCab |
| TR_UNI_MovimientoCnt | Extranet | Contenedores asociados a movimientos | no visible en lectura actual | N:1 con TR_UNI_MovimientoCab |
| TR_DVA_Reserva_Cab | Extranet | Cabecera de reserva/cita de deposito vacios | no visible en lectura actual | 1:N con TR_DVA_Reserva_Det |
| TR_DVA_Reserva_Det | Extranet | Detalle de reserva/cita | no visible en lectura actual | N:1 con TR_DVA_Reserva_Cab |

## 7) Modelo E/R - Dominio de transmisiones

```plantuml
@startuml
!theme plain
skinparam linetype ortho
skinparam backgroundColor #FEFEFE

entity TR_UNI_MANIFIESTO {
  *cnID_Manifiesto (PK)
  cnViaTransporte
  ctCodigoAduana
  ctAnhoManifiesto
  ctNroManifiesto
  ctNave
}

entity TR_UNI_TRANSMISION {
  *cnID_Transmision (PK)
  cnID_Manifiesto (FK)
  cnID_Respuesta (FK)
  cnID_ArchivoEnvio (FK)
  cnID_ArchivoRespuesta (FK)
  ctCorrelativo
  ctDocumentoTransporte
  ctSerieVIN
  cnCodigoEstado
}

entity TR_UNI_ARCHIVO {
  *cnID_Archivo (PK)
  cdFechaHoraEnvio
  ctTipoArchivo
  ctNombreArchivo
  cbComprimido
  coData (BLOB)
}

entity TR_UNI_RESPUESTA {
  *cnID_Respuesta (PK)
  cnID_Transmision (FK)
  cdFechaHoraRespuesta
  cnCodigoEstado
  cnCantidadErrores
  cnCantidadAdvertencias
  cnCantidadMensajes
}

entity TR_UNI_CONTENEDOR_TRANSMISION {
  *cnID_ContenedorTransmision (PK)
  cnID_Transmision (FK)
  ctNumero
  ctTipo
  ctTamano
  cnTara
  cnPayload
}

entity TR_UNI_PRECINTO_CONTENEDOR {
  *cnID_PrecintoContenedor (PK)
  cnID_ContenedorTransmision (FK)
  ctTipo
  ctNumero
  ctCondicion
}

entity TR_UNI_DAM_TRANSMISION {
  *cnID_DAMTransmision (PK)
  cnID_Transmision (FK)
  ctNumeroDAM
  ctCodigoOperacion
  ctNroDocumentoDespachador
}

entity TR_UNI_DOCUMENTO_TRANSMISION {
  *cnID_DocumentoTransmision (PK)
  cnID_DAMTransmision (FK)
  cnID_Transmision (FK)
  ctDocumentoTransporte
  cbLigado
}

entity TR_UNI_MENSAJE_TRANSMISION {
  *cnID_MensajeTransmision (PK)
  cnID_Transmision (FK)
  ctTipo
  ctCodigo
  ctMensaje
}

TR_UNI_MANIFIESTO ||--o{ TR_UNI_TRANSMISION : agrupa
TR_UNI_ARCHIVO ||--o{ TR_UNI_TRANSMISION : archivo_envio
TR_UNI_ARCHIVO ||--o{ TR_UNI_RESPUESTA : archivo_respuesta
TR_UNI_TRANSMISION ||--o| TR_UNI_RESPUESTA : genera
TR_UNI_TRANSMISION ||--o{ TR_UNI_CONTENEDOR_TRANSMISION : contiene
TR_UNI_CONTENEDOR_TRANSMISION ||--o{ TR_UNI_PRECINTO_CONTENEDOR : registra
TR_UNI_TRANSMISION ||--o{ TR_UNI_DAM_TRANSMISION : declara
TR_UNI_DAM_TRANSMISION ||--o{ TR_UNI_DOCUMENTO_TRANSMISION : referencia
TR_UNI_TRANSMISION ||--o{ TR_UNI_MENSAJE_TRANSMISION : produce

@enduml
```

## 8) Modelo E/R - Dominio operativo y comercial

```plantuml
@startuml
!theme plain
skinparam linetype ortho
skinparam backgroundColor #FEFEFE

entity TM_UNI_EMPRESA {
  *id (PK)
  razonSocial
  ruc
}

entity TM_UNI_PERSONA {
  *id (PK)
  empresaId (FK)
  nombres
  apellidoPaterno
  apellidoMaterno
  documento
}

entity TR_UNI_RECLAMO {
  *id (PK)
  personaId (FK)
  datetime fechaRegistro
  estado
  descripcion
}

entity TR_UNI_RECLAMO_DOC {
  *id (PK)
  reclamoId (FK)
  rutaAdjunto
  nombreArchivo
}

entity TR_UNI_MOVIMIENTO_CAB {
  *id (PK)
  fechaMovimiento
  estado
  tipoOperacion
}

entity TR_UNI_MOVIMIENTO_DET {
  *id (PK)
  movimientoCabId (FK)
  descripcion
  cantidad
}

entity TR_UNI_MOVIMIENTO_CNT {
  *id (PK)
  movimientoCabId (FK)
  numeroContenedor
  tipoContenedor
}

entity TR_DVA_RESERVA_CAB {
  *id (PK)
  fechaReserva
  estado
  codigoReserva
}

entity TR_DVA_RESERVA_DET {
  *id (PK)
  reservaCabId (FK)
  servicio
  cantidad
}

TM_UNI_EMPRESA ||--o{ TM_UNI_PERSONA : agrupa
TM_UNI_PERSONA ||--o{ TR_UNI_RECLAMO : registra
TR_UNI_RECLAMO ||--o{ TR_UNI_RECLAMO_DOC : adjunta
TR_UNI_MOVIMIENTO_CAB ||--o{ TR_UNI_MOVIMIENTO_DET : detalla
TR_UNI_MOVIMIENTO_CAB ||--o{ TR_UNI_MOVIMIENTO_CNT : contiene
TR_DVA_RESERVA_CAB ||--o{ TR_DVA_RESERVA_DET : detalla

@enduml
```

## 9) Modelo E/R - Dominio de seguridad y acceso

```plantuml
@startuml
!theme plain
skinparam linetype ortho
skinparam backgroundColor #FEFEFE

entity TC_UNI_VISITANTE {
  *cnID_Visitante (PK)
  ctTipoVisitante
  cbEstado
}

entity TR_UNI_USUARIOSSEG {
  *id_Emple (PK)
  ID_TipDoc
  ctNroDoc
  cnID_Visitante (FK)
  Nombre
  ID_Area (FK)
  ID_Gerencia (FK)
  cbEstadoHabilitado
  cbBloqueado
}

entity TM_UNI_USUARIO {
  *ID_Usuario (PK)
  Usuario
  Estado
}

entity TM_UNI_AREA {
  *ID_Area (PK)
  ctArea
}

entity TM_UNI_GERENCIA {
  *ID_Gerencia (PK)
  ctGerencia
}

TC_UNI_VISITANTE ||--o{ TR_UNI_USUARIOSSEG : clasifica
TR_UNI_USUARIOSSEG }o--|| TM_UNI_AREA : pertenece
TR_UNI_USUARIOSSEG }o--|| TM_UNI_GERENCIA : depende
TR_UNI_USUARIOSSEG }o--o| TM_UNI_USUARIO : mapea
TM_UNI_GERENCIA ||--o{ TM_UNI_AREA : organiza

@enduml
```

## 10) Modelo E/R - Dominio de reclamos

```plantuml
@startuml
!theme plain
skinparam linetype ortho
skinparam backgroundColor #FEFEFE

entity TM_UNI_EMPRESA {
  *id (PK)
  ruc
  razonSocial
}

entity TM_UNI_PERSONA {
  *id (PK)
  nombres
  apellidos
  email
  telefono
}

entity TM_UNI_MOTIVO_RECLAMO {
  *cnID_MotivoReclamo (PK)
  descripcion
}

entity TC_UNI_RESPONSABLE_RECLAMO {
  *id (PK)
  responsable
}

entity TR_UNI_RECLAMO {
  *cnID_ReclamoCab (PK)
  ctClienteRUC
  cnID_MotivoReclamo (FK)
  ctResponsableReclamo
  ctDocumentoReferencia
  ctDescripcionReclamo
  ctEstadoReclamo
  cbProcede
}

entity TR_UNI_RECLAMO_DOC {
  *id (PK)
  reclamoId (FK)
  rutaAdjunto
  nombreArchivo
}

TM_UNI_EMPRESA ||--o{ TR_UNI_RECLAMO : reporta
TM_UNI_PERSONA ||--o{ TR_UNI_RECLAMO : contacta
TR_UNI_RECLAMO }o--|| TM_UNI_MOTIVO_RECLAMO : clasifica
TR_UNI_RECLAMO }o--o| TC_UNI_RESPONSABLE_RECLAMO : asigna
TR_UNI_RECLAMO ||--o{ TR_UNI_RECLAMO_DOC : adjunta

@enduml
```

## 11) Modelo E/R - Dominio de visitas y reservas

```plantuml
@startuml
!theme plain
skinparam linetype ortho
skinparam backgroundColor #FEFEFE

entity TC_UNI_VISITANTE {
  *cnID_Visitante (PK)
  ctTipoVisitante
  cbEstado
}

entity TM_UNI_PERSONA {
  *id (PK)
  nombres
  documento
}

entity TR_DVA_DOCH {
  *ID_DOCH (PK)
  numeroDocumento
  estado
}

entity TR_DVA_RESERVA_CAB {
  *cnID_ReservaCab (PK)
  ctNumeroBL
  cdFechaReserva
  ID_DOCH (FK)
  ctRUCCliente
  ctNombreCliente
  cnID_Visitante (FK)
  cnEstadoSolicitud
}

entity TR_DVA_RESERVA_DET {
  *id (PK)
  cnID_ReservaCab (FK)
  servicio
  cantidad
}

TC_UNI_VISITANTE ||--o{ TR_DVA_RESERVA_CAB : tipifica
TR_DVA_RESERVA_CAB }o--o| TM_UNI_PERSONA : solicita
TR_DVA_RESERVA_CAB }o--o| TR_DVA_DOCH : referencia
TR_DVA_RESERVA_CAB ||--o{ TR_DVA_RESERVA_DET : detalla

@enduml
```

## 12) Modelo E/R - Dominio deposito vacios

```plantuml
@startuml
!theme plain
skinparam linetype ortho
skinparam backgroundColor #FEFEFE

entity TR_DVA_DOCH {
  *ID_DOCH (PK)
  numeroDocumento
  fechaDocumento
  estado
}

entity TR_DVA_DOCH_ITEM {
  *id (PK)
  ID_DOCH (FK)
  item
  cantidad
}

entity TC_DVA_PARAMETRO {
  *id (PK)
  clave
  valor
}

entity TC_DVA_DEVOLUCION_HORAS_SERVICIO {
  *id (PK)
  rangoHorario
  estado
}

entity TC_DVA_CENTRO_PUERTO {
  *id (PK)
  centro
  puerto
}

entity TR_DVA_INGRESO_ASIGNACION {
  *id (PK)
  ID_DOCH (FK)
  estado
}

entity TR_DVA_INGRESO_DEVOLUCION {
  *id (PK)
  ID_DOCH (FK)
  estado
}

TR_DVA_DOCH ||--o{ TR_DVA_DOCH_ITEM : contiene
TC_DVA_PARAMETRO ||--o{ TR_DVA_DOCH : parametriza
TC_DVA_DEVOLUCION_HORAS_SERVICIO ||--o{ TR_DVA_DOCH : condiciona
TC_DVA_CENTRO_PUERTO ||--o{ TR_DVA_DOCH : ubica
TR_DVA_DOCH ||--o{ TR_DVA_INGRESO_ASIGNACION : asigna
TR_DVA_DOCH ||--o{ TR_DVA_INGRESO_DEVOLUCION : devuelve

@enduml
```
