# Arquitectura Unificada - Extranet + Intranet

## Diagrama

El diagrama consolidado se encuentra en: `docs/diagramas/arquitectura_unificada.puml`

Para visualizarlo:
- **Online**: [PlantUML Editor](http://www.plantuml.com/plantuml/uml/)
- **VS Code**: Extension "PlantUML" (jebbs) -> Alt+D

---

## Componentes Principales

### Exclusivos de Extranet

| Componente | Tecnologia | Funcion |
|---|---|---|
| ExtranetWeb | ASP.NET MVC 4 + Web API 4 | Portal web para usuarios externos. UI y endpoints API |
| Extranet.Interfaces | .NET Class Library | Fachadas de negocio (Seguridad, AsignacionVacios, DevolucionCntr) |
| Extranet.business | .NET Class Library | Reglas de negocio y orquestacion (Reservas, Reclamos, Documentos) |
| Extranet.data | .NET Class Library | Acceso a datos via Stored Procedures (T-SQL) |

**Dominios que gestiona**: Deposito de vacios (reservas/citas), reclamos, documentos operativos, visitas.

### Exclusivos de Intranet

| Componente | Tecnologia | Funcion |
|---|---|---|
| SIS_INTRANET Web | ASP.NET MVC 4 + Web API 4 | Portal web para usuarios internos. Controllers, Views, APIs |
| INTRANET.Interfaces | .NET Framework 4.5 | Integraciones, control de accesos, repositorio de envios |
| INTRANET.business | .NET Framework 4.5 | Reglas de negocio y orquestacion |
| INTRANET.data | .NET Framework 4.5 | Acceso a datos ADO.NET y Stored Procedures |
| INTRANET.objects | .NET Framework 4.5 | Entidades y DTOs compartidos entre capas |
| IntranetService | .NET Framework 4.5 | Proxy SOAP/WCF para servicios externos |

**Dominios que gestiona**: Transmisiones aduaneras (manifiestos, DAMs, contenedores), movimientos logisticos, seguridad interna (usuarios, areas, gerencias).

### Compartidos

| Componente | Funcion |
|---|---|
| SQL Server (BD_EXTRANET, BD_INTRANET, BD_BI) | Persistencia operativa. Ambas aplicaciones leen/escriben en bases de datos cruzadas |
| Servicios WCF (ws_extranet/wcf.svc, wcfAgencia.svc) | Integraciones con SAP y sistemas internos corporativos |
| .NET Framework 4.5, MVC 4, Web API 4, ADO.NET | Stack tecnologico comun |
| IIS | Servidor web que hostea ambos aplicaciones (posiblemente en AppPools separados) |

### Sistemas Externos

| Sistema | Funcion | Consumido por |
|---|---|---|
| PortalWebApi | Autenticacion centralizada y gestion de menu | Extranet |
| API REST Externa | Servicios complementarios | Intranet |

---

## Flujos de Comunicacion

### Flujo de Autenticacion (Extranet)
```
Usuario -> ExtranetWeb (POST ValidarUsuario)
  -> Extranet.Interfaces.Seguridad.autenticar()
  -> PortalWebApi (POST Seguridad/login)
  <- Cookies X-Auth-Cookie / X-Refresh-Token
  <- DatosUsuarioAplicacion
  <- JSON resp=true/false
```

### Flujo de Autenticacion (Intranet)
```
Usuario -> SIS_INTRANET Web (POST /Login/ValidarUsuario)
  -> Seguridad.obtenerDatosUsuario()
  -> Seguridad.autenticar(usuario, password, ip, navegador, appId)
  -> CrearCookieAutenticacion()
  -> crearSesion()
  <- JSON resp=true/false
```

### Flujo de Solicitud HTTP (Ambos lados)
```
Cliente Web -> Ruteo MVC/WebAPI
  -> Verificacion de sesion activa
  -> Verificacion de acceso por menu/filtro
  -> Controller Action -> Business Layer -> Data Layer
  -> (fork) SQL Server + WCF/REST externos
  <- Respuesta de datos -> View o JsonResult
```

### Flujo de Datos (Capas internas)
```
Web -> Interfaces -> Business -> Data -> SQL Server (SP)
                              -> IntranetService -> WCF Externos (SOAP)
```

### Integracion Cruzada (Extranet <-> Intranet)
- **BD_EXTRANET** es accedida desde Intranet para lectura de maestros (Empresa, Persona)
- **BD_INTRANET** es accedida desde Extranet para operaciones compartidas
- **Servicios WCF** son consumidos por ambos sistemas para integraciones operativas
- **Maestros compartidos**: TM_UNI_Empresa, TM_UNI_Persona existen en ambas bases

---

## Zonas de Seguridad e Infraestructura

| Zona | Componentes | Proteccion |
|---|---|---|
| Externa / Internet | Usuarios externos, moviles | Firewall/WAF perimetral |
| DMZ | Servidores IIS (Extranet + Intranet) | Terminacion HTTPS |
| Interna | Servicios WCF, APIs externas | Red corporativa |
| Datos | SQL Server (3 bases de datos) | Acceso restringido por red |

**Problemas de seguridad identificados (ADRs)**:
- **ADR-0003**: ConnectionStrings en Web.config sin cifrar (texto plano)
- **ADR-0004**: BLOBs almacenados directamente en BD (VARBINARY), causando over-fetching

---

## Suposiciones y Gaps

### Suposiciones

1. **IIS separado o AppPools separados**: Los diagramas originales no especifican si Extranet e Intranet corren en el mismo servidor IIS o en servidores distintos. Se asume que comparten infraestructura IIS pero con AppPools separados.

2. **Firewall/WAF perimetral**: No existe diagrama de red explicito. Se infiere la existencia de un firewall/WAF dado que Extranet es accesible desde internet y ambas usan HTTPS.

3. **PortalWebApi como sistema independiente**: El diagrama de contexto de Extranet muestra PortalWebApi como sistema externo de autenticacion. No se detalla su infraestructura ni si es compartido con Intranet (Intranet no lo menciona explicitamente).

4. **Red interna para WCF**: Se asume que los servicios WCF estan en la red corporativa interna, accesibles desde ambos servidores IIS.

5. **BD_BI**: Mencionada en el diagrama de Intranet pero sin detalles de uso. Se asume que es para Business Intelligence/reporting.

6. **Web Farm**: ADR-0003 menciona "granjas de servidores" (web farms), lo que sugiere que podria haber balanceo de carga con multiples servidores IIS.

### Gaps en los Diagramas Originales

1. **No hay diagrama de despliegue fisico**: No se muestra la topologia de servidores, balanceadores, o configuracion de red.

2. **No hay diagrama de red**: Falta la arquitectura de red (VLANs, subnets, reglas de firewall entre zonas).

3. **No se detalla la relacion Extranet-Intranet a nivel de codigo**: Ambos sistemas comparten bases de datos pero no hay un diagrama que muestre las dependencias cruzadas de tablas.

4. **PortalWebApi solo en Extranet**: Intranet no menciona PortalWebApi en su contexto. Es posible que Intranet use un mecanismo de autenticacion diferente o que el diagrama este incompleto.

5. **API REST Externa solo en Intranet**: Extranet no menciona API REST externa. Podria ser un gap o una diferencia real de arquitectura.

6. **No hay detalle de CI/CD**: No se muestra pipeline de despliegue, versionado, o estrategia de release.

7. **No hay monitoreo/logging**: No se muestran sistemas de logging, monitoreo, alertas, o APM.

8. **No hay backup/DR**: No se muestra estrategia de backup, disaster recovery, o replicacion de bases de datos.

9. **BLOBs en BD**: ADR-0004 identifica que archivos binarios estan almacenados en tablas SQL. No hay componente de storage externo en los diagramas actuales (es una mejora pendiente).

10. **No hay detalle de sesiones**: No se muestra como se gestiona la sesion entre ambos lados (cookie-based, server-side, token).
