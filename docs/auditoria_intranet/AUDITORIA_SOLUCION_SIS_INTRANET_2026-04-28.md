# Auditoria de Software e Inventario Tecnico

Documento relacionado:

- Resumen e indice: [RESUMEN_EJECUTIVO_E_INDICE.md](RESUMEN_EJECUTIVO_E_INDICE.md)
- Anexo de diagramas: [ANEXO_DIAGRAMAS_ARQUITECTURA_MERMAID.md](ANEXO_DIAGRAMAS_ARQUITECTURA_MERMAID.md)

Fecha: 2026-04-28  
Solucion: SIS_INTRANET

## 1) Resumen ejecutivo

Esta auditoria se realizo con analisis estatico del codigo fuente y archivos de configuracion (sin despliegue ni ejecucion). La solucion es una aplicacion web ASP.NET MVC/Web API sobre .NET Framework 4.5 con arquitectura por capas (web, negocio, datos, objetos, integraciones y servicios).

Conclusiones principales:

- Hay exposicion de credenciales en texto plano en configuraciones.
- Se detecta uso extensivo de protocolos HTTP sin cifrado para integraciones internas/externas.
- No se evidencia uso de controles estandar de seguridad MVC para CSRF (ValidateAntiForgeryToken) ni autorizacion centralizada con Authorize.
- La plataforma y dependencias son legacy (MVC 4, Web API 4, .NET 4.5, librerias antiguas), con riesgo alto de vulnerabilidades conocidas y deuda tecnica.
- No se observa contenedorizacion ni pipeline CI/CD moderno en el repositorio.

## 2) Alcance y metodologia

Alcance analizado:

- Estructura de solucion y proyectos.
- Configuracion de plataforma y runtime.
- Seguridad de aplicacion y de integraciones.
- Riesgos arquitectonicos y operativos.
- Cobertura basica de pruebas.

Metodologia:

1. Inventario de artefactos de solucion y capas.
2. Extraccion de versiones y dependencias desde csproj/packages.config.
3. Revision de configuraciones web/service para seguridad y entorno.
4. Busqueda dirigida de patrones de riesgo (secretos, HTTP, controles de autenticacion, manejo de errores).
5. Clasificacion de criticidad por impacto x probabilidad.

## 3) Inventario de la solucion

### 3.1 Estructura general

- Solucion principal: SIS_INTRANET.sln
- Proyectos .NET principales:
  - SIS_INTRANET (Web MVC + Web API)
  - INTRANET.business (capa negocio)
  - INTRANET.data (capa acceso a datos)
  - INTRANET.objects (entidades/DTO)
  - INTRANET.Interfaces (integraciones y logica de interfaces)
  - IntranetService (consumo/abstraccion de servicios)
  - SIS_INTRANET.Test (pruebas unitarias)

Evidencia: SIS_INTRANET.sln (cabecera y Project entries)

### 3.2 Dependencias entre proyectos (alto nivel)

- SIS_INTRANET -> INTRANET.business, INTRANET.Interfaces, INTRANET.objects, IntranetService.
- INTRANET.business -> INTRANET.data, INTRANET.objects.
- INTRANET.data -> INTRANET.objects.
- INTRANET.Interfaces -> INTRANET.business, IntranetService.
- IntranetService -> INTRANET.business, INTRANET.objects.
- SIS_INTRANET.Test -> INTRANET.business, INTRANET.objects.

Evidencia: archivos .csproj (ProjectReference Include).

### 3.3 Dependencias externas de aplicación

**URLs externas del sistema:**
- Extranet: https://www.unimar.com.pe/extranet
- Intranet: https://www.unimar.com.pe/intranet

**Observacion:** Estas URLs representan los puntos de entrada de usuario del sistema. Actualmente estan documentadas bajo la sección de integraciones HTTP detectadas en Web.config.

### 3.4 Cobertura de pruebas

- Se detecta 1 archivo de prueba principal: SIS_INTRANET.Test/GrupoContenedoresTest.cs.
- Framework de pruebas legacy (MSTest VS10/VS2012).

Riesgo asociado: cobertura potencialmente baja para una solucion de alta criticidad funcional.

## 4) Plataforma, contenedor y versiones

### 4.1 Plataforma objetivo

- Framework: .NET Framework 4.5 (TargetFrameworkVersion v4.5 en todos los csproj principales).
- Tooling base: Visual Studio 2012 / MSBuild ToolsVersion 4.0.
- Hosting esperado: IIS/ASP.NET 4.x (web.config y handlers ASP.NET 4.0).

### 4.2 Contenedor/despliegue

- No se detectan Dockerfile, docker-compose, manifiestos Kubernetes ni artefactos de contenedorizacion propios.
- Conclusion: despliegue tradicional (IIS + App Pool) y/o servicios Windows, no contenedorizado en este repositorio.

### 4.3 Librerias y paquetes relevantes (detectados)

- ASP.NET MVC 4.0.20710.0
- ASP.NET Web API 4.0.20710.0
- Razor/WebPages 2.0.20710.0
- Newtonsoft.Json 5.0.4 (packages.config web), y referencia 4.5.0.0 en proyecto web
- NPOI 2.3.0
- SharpZipLib 0.86.0
- WebGrease 1.5.2
- DocumentFormat.OpenXml 2.7.2

Conclusiones de versionado:

- Stack claramente legacy y fuera de soporte moderno.
- Probabilidad alta de deuda de seguridad por CVEs historicos en librerias antiguas.

## 5) Hallazgos de seguridad y arquitectura

## 5.1 Criticos

1. Credenciales en texto plano en configuracion

- SIS_INTRANET/Web.config lineas 8-10: connection strings con User ID y Password visibles.
- SIS_INTRANET/Views/Web.config lineas 16-18: credencial de BD en texto plano.

Impacto:

- Compromiso de base de datos por fuga de repositorio, backups, logs o accesos internos.

2. Integraciones por HTTP (sin TLS)

- SIS_INTRANET/Web.config lineas 96-106: endpoints WCF por http://
- SIS_INTRANET/Web.config linea 23: RutaApiUNIDEPOT por http://
- SIS_INTRANET/Views/Shared/_LayoutMovil.cshtml multiples assets externos por http://
- Dependencias externas de aplicacion:
  - https://www.unimar.com.pe/extranet
  - https://www.unimar.com.pe/intranet

Impacto:

- Riesgo MITM, robo/manipulacion de trafico y credenciales de integracion.

## 5.2 Altos

3. Ausencia de proteccion CSRF en acciones POST MVC

- Se detectan muchas acciones [HttpPost] en controladores.
- No se detecta uso de [ValidateAntiForgeryToken] en controladores revisados.

Impacto:

- Posible ejecucion de acciones no deseadas via CSRF en sesiones autenticadas.

4. Autorizacion no estandarizada

- No se detecta uso de [Authorize] en controladores MVC/WebAPI.
- Se usa filtro custom por sesion (Session["UserInfo"]) en SIS_INTRANET/Filters/UsuarioAutenticadoFilter.cs.

Impacto:

- Mayor superficie de errores de implementacion y bypass por inconsistencias entre endpoints.

5. API Web expuesta sin control fuerte visible

- SIS_INTRANET/Apis/RepositorioEnviosController.cs expone acciones HttpPost sin atributo de autorizacion.
- Metodo ValidadCabecera solo devuelve header Authorization sin validacion criptografica robusta.

Impacto:

- Riesgo de uso indebido de endpoints y acceso no autorizado si no existen controles externos (WAF/reverse proxy).

## 5.3 Medios

6. Configuracion insegura de validacion de request en vistas

- SIS_INTRANET/Views/Web.config linea 53: validateRequest="false".

Impacto:

- Mayor riesgo de inyecciones de contenido (XSS/HTML malicioso) si no hay sanitizacion estricta en toda entrada/salida.

7. Manejo de errores con excepciones genericas y capturas vacias

- Varias ocurrencias de catch {} en controladores (por ejemplo BaseController, LoginController, VisitanteController).
- Uso amplio de throw new Exception con mensajes concatenados.

Impacto:

- Poca trazabilidad, diagnostico dificil y riesgo de ocultar fallas de seguridad/operacion.

8. Acoplamiento alto a servicios legacy SOAP/WCF

- Multiples endpoints basicHttpBinding y dependencias de IntranetService.

Impacto:

- Fragilidad de integracion, complejidad operativa y limitacion para evolucionar arquitectura.

## 6) Analisis de criticidad

Escala usada:

- Impacto: Bajo (1), Medio (2), Alto (3), Muy Alto (4)
- Probabilidad: Baja (1), Media (2), Alta (3), Muy Alta (4)
- Criticidad = Impacto x Probabilidad

### 6.1 Criticidad por dominio

- Seguridad de credenciales: Impacto 4, Probabilidad 4, Criticidad 16 (Critico)
- Transporte/integraciones HTTP: Impacto 4, Probabilidad 3, Criticidad 12 (Alto)
- CSRF en operaciones POST: Impacto 3, Probabilidad 3, Criticidad 9 (Alto)
- Autorizacion no centralizada: Impacto 3, Probabilidad 3, Criticidad 9 (Alto)
- Deuda de versionado legacy: Impacto 3, Probabilidad 4, Criticidad 12 (Alto)
- Observabilidad/manejo de errores: Impacto 2, Probabilidad 3, Criticidad 6 (Medio)
- Cobertura de pruebas limitada: Impacto 2, Probabilidad 3, Criticidad 6 (Medio)

### 6.2 Riesgo global de la solucion

Riesgo global estimado: Alto-Critico.

Razon:

- Coexistencia de secretos en claro + transporte no cifrado + stack legacy + controles de seguridad no estandarizados.

### 6.3 Tabla de puntos criticos a resolver (orden ascendente)

| Orden | Punto critico a resolver | Impacto | Probabilidad | Criticidad | Nivel |
|---|---|---:|---:|---:|---|
| 1 | Observabilidad y manejo de errores (catch vacios, excepciones genericas, baja trazabilidad) | 2 | 3 | 6 | Medio |
| 2 | Cobertura de pruebas limitada en procesos criticos | 2 | 3 | 6 | Medio |
| 3 | Configuracion insegura de validacion de request en vistas (validateRequest="false") | 2 | 3 | 6 | Medio |
| 4 | Ausencia de proteccion CSRF en acciones POST MVC | 3 | 3 | 9 | Alto |
| 5 | Autorizacion no centralizada/no estandarizada | 3 | 3 | 9 | Alto |
| 6 | Transporte e integraciones por HTTP sin TLS | 4 | 3 | 12 | Alto |
| 7 | Deuda de versionado legacy y librerias desactualizadas | 3 | 4 | 12 | Alto |
| 8 | Credenciales en texto plano en configuracion | 4 | 4 | 16 | Critico |

## 7) Auditoria de arquitectura (estado actual)

Fortalezas:

- Separacion en capas (web, negocio, datos, objetos) relativamente clara.
- Uso extensivo de procedimientos almacenados desde la capa de datos (reduce SQL inline directo).

Debilidades:

- Acoplamiento fuerte entre capas con referencias cruzadas y gran base monolitica.
- Uso de servicios SOAP/WCF legacy que complejiza interoperabilidad moderna.
- Control de seguridad heterogeneo (filtros custom, sesiones, validaciones dispersas).
- Falta de pipeline de calidad visible (sin CI/CD y sin automatizacion de seguridad detectada).

## 8) Analisis de normalizacion y performance de datos

Este analisis se realizo a partir de las entidades persistentes y clases de lectura en INTRANET.objects. No sustituye una revision directa del modelo fisico SQL, pero permite detectar patrones consistentes de diseno con impacto en calidad de datos, mantenibilidad y rendimiento.

### 8.1 Hallazgos de normalizacion

1. Mezcla de claves y descripciones en la misma entidad

- Ejemplo representativo: SG_Centro_Alm_Equipo contiene ids de referencia (id_Centro, id_Ubicacion, id_Zona, id_Equipo, id_Marca, id_Modelo, id_Estado) y al mismo tiempo campos descriptivos derivados (descripcion_centro, descripcion_ubicacion, descripcion_zona, descripcion_equipo, descripcion_marca, descripcion_modelo, descripcion_estado, codigo_equipo, nombre_equipo).
- Interpretacion: si esta clase representa una tabla fisica, existe fuerte desnormalizacion y duplicacion de datos descriptivos; si representa una proyeccion/consulta, el diseno de lectura esta mezclado con la capa de entidades persistentes.

Riesgo:

- Inconsistencia de datos, mayor costo de actualizacion y ambiguedad entre entidades transaccionales y DTOs de reporte.

2. Almacenamiento de snapshots textuales junto con referencias maestras

- Ejemplo: TR_UNI_MovimientoCab combina cnID_Persona con ctEmpresa en texto libre.
- Ejemplo: TR_DVA_Reserva_Cab almacena ctRUCCliente y ctNombreCliente junto con relaciones a visitante/proceso.
- Ejemplo: TR_UNI_Reclamo mantiene RUC, nombres de contacto, consignatario, responsable y descripciones amplias en la misma cabecera.

Interpretacion:

- Hay mezcla entre modelo normalizado y datos snapshot del momento operativo. Esto puede ser valido para trazabilidad historica, pero debe explicitarse como estrategia de snapshot; de lo contrario, implica violaciones de 3FN/BCNF en la tabla operacional.

Riesgo:

- Divergencia entre maestros y documentos operativos, dificultad de reconciliacion y mayor complejidad de reglas de actualizacion.

3. Jerarquias o relaciones autoreferenciadas poco explicitadas

- Ejemplo: TM_UNI_Empresa contiene idEmpresa e idEmpresaFK, lo que sugiere una relacion padre-hijo o consolidacion empresarial, pero sin metadatos de restriccion visibles a nivel de entidad.

Riesgo:

- Posibles ciclos, integridad referencial debil y consultas jerarquicas costosas si no existen indices/constraints apropiados.

4. Tipos semanticos modelados como string cuando deberian ser dominios mas estrictos

- Ejemplo: TC_DVA_DevolucionHorasServicio usa ctHoraIniRecpcion, ctHoraFinRecpcion y ctProximaCita como string.

Riesgo:

- Validacion inconsistente, comparaciones lentas, dificultad para indexar por rango horario y errores de conversion/reglas de negocio.

5. Entidades de lectura y entidades de persistencia mezcladas en el mismo namespace/capa

- Varias clases incluyen columnas de negocio, columnas derivadas y resultados ampliados de joins dentro del mismo objeto persistente (por ejemplo SG_Centro_Alm_Equipo, TR_UNI_Reclamo y extensiones del dominio Extranet).

Riesgo:

- Se diluye la separacion entre write model, read model y DTOs, dificultando evolucion, pruebas y afinamiento de consultas.

### 8.2 Hallazgos de performance

1. Carga eager de BLOBs en entidades generales

- Ejemplo: TR_UNI_Archivo carga coData como byte[] en el mismo objeto base.

Impacto:

- Alto consumo de memoria y ancho de banda entre capa de datos y negocio cuando se listan registros o se recuperan cabeceras sin necesidad del contenido binario.

Recomendacion:

- Separar metadata de archivo y payload binario, o usar consultas especializadas para descarga diferida.

2. Lectura completa de columnas en readData(IDataReader)

- El patron dominante hidrata todos los campos disponibles de cada entidad, incluso en objetos grandes.

Impacto:

- Over-fetching, mayor I/O de BD, mayor CPU de serializacion y mayor uso de memoria en listados masivos.

Recomendacion:

- Crear DTOs/proyecciones por caso de uso y limitar el SELECT a columnas necesarias.

3. Entidades anchas para operaciones de listado

- Ejemplo: TR_UNI_Reclamo contiene gran cantidad de atributos operativos, de contacto, analisis, conclusion y auditoria.
- Ejemplo: SG_Centro_Alm_Equipo concentra ids, descripciones y datos de inventario en un solo objeto.

Impacto:

- Filas mas anchas, peor densidad por pagina, mayor costo de lectura y peores tiempos de respuesta en grids/reportes.

Recomendacion:

- Separar cabecera transaccional, detalle y vistas/materializaciones de consulta.

4. Posible dependencia excesiva de joins para armar descripciones en tiempo real o, en el extremo opuesto, exceso de desnormalizacion para evitarlos

- El modelo sugiere dos patrones coexistentes: entidades con muchas FK y entidades con muchos campos descriptivos derivados.

Impacto:

- O bien se penaliza la consulta por joins complejos, o bien se penaliza consistencia/escritura por duplicacion. Ambos extremos son costosos si no se distinguen claramente tablas transaccionales de vistas de lectura.

Recomendacion:

- Adoptar un patron explicito de read models para pantallas e informes, manteniendo normalizado el nucleo transaccional.

5. Colecciones hijas embebidas en objetos de configuracion

- Ejemplo: TC_DVA_DevolucionHorasServicio contiene listas HorarioRegular, HorarioExtraordinario y FechasEspeciales.

Impacto:

- Riesgo de sobrecarga de serializacion/carga si siempre se recupera el agregado completo, incluso cuando solo se necesita la cabecera de configuracion.

Recomendacion:

- Definir endpoints/consultas especificas para cabecera y detalle, o aplicar carga selectiva.

### 8.3 Evaluacion de criticidad del diseno de datos

| Problema | Severidad | Justificacion |
|---|---|---|
| BLOBs cargados junto con metadata | Alto | Impacta memoria, red e I/O de forma directa |
| Entidades anchas y over-fetching | Alto | Afecta listados, APIs y tiempo de respuesta |
| Mezcla de entidades transaccionales y DTOs/proyecciones | Alto | Dificulta afinamiento y mantiene deuda arquitectonica |
| Desnormalizacion no explicitada en snapshots operativos | Medio-Alto | Riesgo de inconsistencia si no hay reglas claras |
| Tiempos/valores de dominio modelados como string | Medio | Reduce calidad de datos e indexabilidad |
| Jerarquias autoreferenciadas sin semantica visible | Medio | Puede afectar integridad y consultas recursivas |

### 8.4 Recomendaciones tecnicas de remediacion

1. Separar entidades transaccionales, DTOs de lectura y modelos de reporte en namespaces/capas distintas.
2. Dividir TR_UNI_Archivo en metadata y contenido binario, o usar descarga lazy del payload.
3. Revisar tablas snapshot para definir formalmente que datos deben congelarse historicamente y cuales deben provenir de maestros.
4. Normalizar atributos de tiempo/horario a tipos date/time/datetime donde corresponda.
5. Reducir anchura de entidades usadas en consultas frecuentes mediante proyecciones especificas.
6. Validar indices sobre todas las FK relevantes y columnas de busqueda funcional (correlativo, documento, manifiesto, persona, estado, fechas).
7. Documentar explicitamente que clases son entidades fisicas y cuales son vistas o resultados enriquecidos de consulta.

### 8.5 Matriz formal de problemas por tabla/entidad

| Tabla/Entidad | Problema dominante | Impacto tecnico | Recomendacion principal | Prioridad |
|---|---|---|---|---|
| TR_UNI_Archivo | BLOB byte[] coData en la entidad base | Alto consumo de memoria, I/O y serializacion innecesaria | Separar metadata de contenido y usar carga diferida para descargas | Alta |
| TR_UNI_Reclamo | Cabecera muy ancha con datos de cliente, contacto, analisis, conclusion y estado | Filas anchas, mantenimiento complejo, mayor costo de lectura y actualizacion | Dividir en cabecera, seguimiento/flujo y adjuntos o vistas de consulta | Alta |
| TR_DVA_Reserva_Cab | Mezcla de snapshot comercial, datos operativos y vinculos de visita/reserva | Riesgo de inconsistencia y sobrecarga en consultas operativas | Formalizar snapshot historico y mover descripciones a read models | Alta |
| TR_UNI_MovimientoCab | FK operativas mezcladas con texto descriptivo como ctEmpresa y datos extendidos de consulta | Ambiguedad entre entidad transaccional y resultado enriquecido | Separar entidad write model de DTO operacional/listado | Alta |
| SG_Centro_Alm_Equipo | IDs maestros y descripciones derivadas en el mismo objeto | Duplica semantica, induce desnormalizacion y sobrelectura | Tratarlo como DTO/vista o normalizar si persiste fisicamente | Media-Alta |
| TR_UNI_UsuariosSeg | Mezcla identidad, seguridad, organizacion y atributos de visitante | Acoplamiento transversal y ampliacion de joins o duplicacion | Separar perfil de seguridad, persona operativa y asignacion organizacional | Media-Alta |
| TC_DVA_DevolucionHorasServicio | Horas y semantica de proxima cita modeladas como string | Baja calidad de datos, comparaciones lentas y reglas fragiles | Convertir a tipos temporales y catalogos de dominio | Media |
| TM_UNI_Empresa | Autorreferencia idEmpresaFK con semantica no explicitada | Integridad y navegacion jerarquica inciertas | Definir jerarquia formal con constraints e indices | Media |

Orden de ataque recomendado de menor a mayor criticidad dentro de esta matriz:

1. TM_UNI_Empresa.
2. TC_DVA_DevolucionHorasServicio.
3. SG_Centro_Alm_Equipo.
4. TR_UNI_UsuariosSeg.
5. TR_UNI_MovimientoCab.
6. TR_DVA_Reserva_Cab.
7. TR_UNI_Reclamo.
8. TR_UNI_Archivo.

### 8.6 Trazabilidad entidad -> modulo -> capa de uso

La siguiente trazabilidad se infiere desde referencias directas en capas business, data, interfaces y controllers. Sirve para priorizar refactorizacion sin romper flujos criticos.

| Entidad | Capa de negocio / uso directo | Capa de datos / acceso | Observacion de criticidad |
|---|---|---|---|
| TR_UNI_Archivo | TR_UNI_ArchivoBLL lista y obtiene entidades completas | TR_UNI_ArchivoDAL | Riesgo alto porque el patron de listado ya hidrata el binario en escenarios generales |
| TR_UNI_MovimientoCab | TR_UNI_MovimientoCabBLL_Extend lista pendientes, placas, personas y operaciones | TR_UNI_MovimientoCabDAL_Extend usa SP palTR_UNI_MovimientoCab_* | Entidad usada en control operativo con campos extendidos para busqueda y pantalla |
| TR_DVA_Reserva_Cab | TR_DVA_Reserva_CabBLLextend, TR_DVA_Reserva_DetBLLextend, TR_DVA_DOCHBLL, ControlPuertas, ControlVisitantes | TR_DVA_Reserva_CabDAL y extensiones | Es un agregado transversal en citas, control de acceso y deposito vacios; cualquier cambio requiere estrategia incremental |
| TC_DVA_DevolucionHorasServicio | TC_DVA_DevolucionHorasServicioBLL y DisponibilidadFechaHoraBLL; expuesto por OperacionVaciosController | TC_DVA_DevolucionHorasServicioDAL | El modelo actual gobierna calculo de disponibilidad diaria, por eso cualquier ajuste de tipos debe ser compatible hacia atras |
| TR_UNI_Reclamo | TR_UNI_ReclamoBLL y procesos de adjuntos / seguimiento | TR_UNI_ReclamoDAL y TR_UNI_Reclamo_DocDAL | Alta sensibilidad funcional por combinacion de alta, consulta, adjuntos y workflow del reclamo |
| SG_Centro_Alm_Equipo | SG_Centro_Alm_EquipoBLL y SG_EquipoBLL | SG_Centro_Alm_EquipoDAL | Parece mas cercano a vista de consulta/inventario que a una entidad puramente normalizada |
| TR_UNI_UsuariosSeg | Autenticacion, control de accesos y reservas de visitantes | Acceso inferido desde consultas de control de acceso y seguridad | Modelo transversal a seguridad y operacion; su remodelado debe aislar permisos de datos personales |

Lectura arquitectonica derivada:

- Los objetos de datos no solo representan persistencia; tambien funcionan como contratos de integracion entre BLL, DAL e interfaces.
- Por esa razon, la deuda de normalizacion ya se propaga a la API interna del sistema, no solo al esquema SQL.
- La remediacion debe ser por capas, introduciendo contratos nuevos en paralelo antes de retirar los objetos actuales.

### 8.7 Propuesta de rediseño del modelo por dominios criticos

#### Dominio 1: Archivos y transmisiones

Problema actual:

- TR_UNI_Archivo concentra metadata y payload binario.

Rediseño propuesto:

- Tabla/entidad de metadata: identificador, nombre, tipo, tamano, hash, compresion, fecha, usuario.
- Tabla/almacen de contenido: archivo_id, payload o puntero externo.
- DTOs especificos: ArchivoListadoDTO, ArchivoDescargaDTO.

Beneficio:

- Reduce I/O en consultas generales y habilita politicas de retencion/almacenamiento mas eficientes.

#### Dominio 2: Reclamos

Problema actual:

- La cabecera TR_UNI_Reclamo mezcla captura comercial, contacto, analisis y resolucion.

Rediseño propuesto:

- ReclamoCabecera.
- ReclamoContactoSnapshot.
- ReclamoSeguimiento o ReclamoWorkflow.
- ReclamoAdjunto.
- Vista de consulta ReclamoConsultaDTO para bandejas y reportes.

Beneficio:

- Permite normalizar el flujo sin perder evidencia historica del contacto o del cliente al momento del reclamo.

#### Dominio 3: Reservas y deposito vacios

Problema actual:

- TR_DVA_Reserva_Cab concentra BL, cliente, DOCH, visitante, estado y datos derivados usados por control de acceso.

Rediseño propuesto:

- ReservaCabecera.
- ReservaClienteSnapshot.
- ReservaVisitanteRelacion.
- ReservaDetalle/Slot.
- Read models de control de acceso y dashboard operacional.

Beneficio:

- Aisla la transaccion operativa de las vistas enriquecidas y reduce acoplamiento entre citas y control de puertas.

#### Dominio 4: Configuracion horaria

Problema actual:

- TC_DVA_DevolucionHorasServicio usa strings para horas y codigos de comportamiento.

Rediseño propuesto:

- Reemplazar strings de hora por time/datetime.
- Catalogar el tipo de proxima cita en una tabla o enum persistente.
- Mantener HorarioRegular, HorarioExtraordinario y FechasEspeciales como tablas hijas especializadas.

Beneficio:

- Reglas mas simples, comparaciones mas eficientes y menor tasa de error por conversion manual.

#### Dominio 5: Seguridad y personas

Problema actual:

- TR_UNI_UsuariosSeg mezcla identidad personal, estado de seguridad y adscripcion organizacional.

Rediseño propuesto:

- UsuarioSeguridad.
- PersonaOperativa.
- UsuarioOrganizacion.
- Read model UsuarioAccesoResumen.

Beneficio:

- Mejora segregacion de responsabilidades, privacidad y trazabilidad de autorizacion.

### 8.8 Estrategia de implementacion incremental sugerida

1. Introducir DTOs de lectura nuevos en business/interfaces sin reemplazar aun las entidades legacy.
2. Separar primero el caso de mayor retorno tecnico: archivos binarios y listados de reclamos.
3. Crear stored procedures/proyecciones dedicadas para grids y dashboards de reservas/movimientos.
4. Migrar gradualmente reglas de horario a tipos fuertes manteniendo compatibilidad temporal con los strings actuales.
5. Aislar seguridad/persona/organizacion en modelos distintos antes de tocar filtros y control de acceso.
6. Retirar entidades mixtas solo cuando los consumidores de BLL, Interfaces y Controllers ya usen contratos nuevos.

## 9) Plan de evaluacion previa (antes de ejecutar en cualquier entorno)

Objetivo del plan: reducir riesgo antes de despliegue/ejecucion en QAS/PROD.

### 9.1 Enfoque ejecutivo de salida a produccion ASAP

Si la prioridad del negocio es ejecutar la solucion cuanto antes, el enfoque recomendado no es bloquear la salida hasta resolver toda la deuda tecnica, sino asegurar que el sistema pueda operar con estabilidad minima aceptable y riesgo controlado.

Principio rector:

- Primero producir estabilidad operativa.
- Despues endurecer seguridad y observabilidad.
- Finalmente optimizar datos, performance y modernizacion.

Minimo indispensable antes del arranque:

1. Rotacion de credenciales y validacion de accesos reales a base de datos y servicios.
2. TLS/HTTPS operativo en accesos expuestos y en integraciones criticas ajustables sin romper compatibilidad.
3. Configuracion estable de IIS/App Pool, permisos, rutas, temporales, recursos y timeouts.
4. Smoke test funcional extremo a extremo sobre login, operacion clave, consulta clave y servicio externo.
5. Logging minimo habilitado para diagnosticar errores de aplicacion, integracion y base de datos.
6. Procedimiento de rollback y soporte reforzado durante la salida.

No debe bloquear el Go-Live inicial (pero sí requiere mitigación urgente):

- La refactorizacion completa del modelo de datos.
- La normalizacion integral de entidades legacy.
- La modernizacion total de framework y librerias.
- La correccion completa de toda la deuda historica del codigo.
- La migración completa de IIS 2008 → IIS 10 (se ejecuta en paralelo, cutover en Mes 1).

Riesgo aceptado:

- Se acepta deuda tecnica controlada en favor de continuidad operativa, siempre que exista una fase posterior obligatoria de estabilizacion y remediacion.

### Fase 0 - Preparacion y gobierno (1-2 dias)

1. Definir owners por capa (web, negocio, datos, seguridad, infra).
2. Congelar cambios no urgentes durante la auditoria.
3. Levantar matriz de activos y datos sensibles (PII, credenciales, archivos, integraciones).
4. Definir funcionalidad minima para el Go-Live y funciones que pueden diferirse temporalmente.

Entregables:

- RACI de auditoria.
- Catalogo de activos y dependencias.
- Lista de alcance minimo para salida inicial.

### Fase 1 - Seguridad basica bloqueante (2-5 dias)

**BLOQUEANTE CRITICO REPORTADO:** IIS 2008 con fallo TLS en dispositivos móviles

Hallazgo: Coordinador de infraestructura líder ha detectado caídas de servidor al intentar dispositivos móviles negociar TLS con IIS 2008 (versión legacy, fuera de soporte). Ver [ADR-001: Migración IIS 2008 → IIS 10+](./adrs/ADR-001_Migracion_IIS2008_a_IIS10_TLS_Moviles.md).

Acciones bloqueantes:

1. **URGENTE - Infraestructura:** Iniciar provisioning de servidor IIS 10+ (Windows Server 2019/2022) en paralelo al Go-Live.
2. **URGENTE - Código:** Implementar mitigaciones en Web.config + Global.asax.cs (Sección 4 del ADR-001) para reducir renegociación TLS en IIS 2008.
3. Rotar inmediatamente credenciales expuestas y revocar las conocidas.
4. Mover secretos a almacen seguro de secretos/configuracion protegida.
5. Forzar TLS extremo a extremo en endpoints internos y externos.
6. Bloquear carga de recursos externos HTTP en layouts/views.
7. Activar politicas de cabeceras seguras en IIS/reverse proxy (HSTS, X-Content-Type-Options, X-Frame-Options, CSP base).
8. Validar configuracion de infraestructura para evitar caidas por App Pool, permisos, disco, temporales y conectividad.

**Criterio de Go-Live para Fase 0:**
- ✅ Cambios código (mitigación TLS) implementados y testeados en load test 500 usuarios móviles
- ✅ Usuarios móviles pueden conectar sin caídas TLS
- ✅ IIS 10+ en provisionamiento (será cutover en Mes 1 sin bloquear Go-Live inicial)

Criterio de salida:

- Cero secretos en repositorio.
- Cero endpoint productivo por HTTP.
- Infraestructura y smoke tests minimos validados.
- Usuarios móviles pueden conectar con TLS sin caídas.

### Fase 2 - Hardening de aplicacion (3-7 dias)

1. Incorporar [ValidateAntiForgeryToken] en formularios y acciones POST MVC.
2. Estandarizar autorizacion con atributos y/o middleware coherente por dominio.
3. Revisar endpoint API RepositorioEnvios para autenticacion fuerte (token firmado, expiracion, validacion de origen).
4. Eliminar catch vacios y centralizar manejo de excepciones/log.
5. Revisar validateRequest="false" y reemplazar por sanitizacion/encoding seguro controlado por caso de uso.
6. Priorizar primero correcciones que afecten estabilidad real antes que deuda cosmetica o de bajo impacto.

**Mejoras TLS post-Go-Live (Fase 2, Semana 2-3):**
- Completar migración IIS 2008 → IIS 10 en servidor dedicado (no bloquea Go-Live)
- Validar TLS 1.2/1.3 en IIS 10 versus TLS 1.0/1.1 en IIS 2008 actual
- Cutover DNS/LB a IIS 10 durante ventana baja (fin de semana)

Criterio de salida:

- Checklist OWASP ASVS L1 aprobado en endpoints criticos.

### Fase 3 - Calidad, pruebas y arquitectura (1-3 semanas)

1. Implementar pruebas unitarias y de integracion en casos criticos de negocio.
2. Agregar analisis SAST y dependencia (SCA) en pipeline.
3. Diseñar hoja de ruta de modernizacion (minimo a .NET Framework soportado o .NET LTS).
4. Plan de desacople progresivo de servicios WCF legacy.
5. Ejecutar optimizacion progresiva de consultas, entidades y datos conforme al analisis de la seccion 8.

Criterio de salida:

- Pipeline con gates de seguridad y calidad.
- Backlog priorizado de remediacion arquitectonica.

### Fase 4 - Go/No-Go pre ejecucion

Checklist minimo obligatorio antes de ejecutar en entorno compartido:

- Secretos rotados y fuera de repositorio.
- Endpoints externos/internos bajo TLS.
- Controles CSRF y autorizacion verificados en endpoints de escritura.
- Registro de logs de seguridad y trazabilidad de errores activo.
- Pruebas de humo y regresion minima ejecutadas.

Decision:

- GO solo si no quedan hallazgos criticos abiertos.
- NO-GO si persisten credenciales expuestas o trafico no cifrado en flujos sensibles.

### 9.2 Plan progresivo de optimizacion posterior al arranque

#### Etapa A - Estabilizacion operativa inmediata (0-2 semanas)

Objetivo:

- Evitar caidas, timeouts y errores repetitivos despues del Go-Live.

Acciones:

1. Monitorear incidentes de login, errores HTTP 500, timeouts y fallas de integracion.
2. Ajustar IIS, App Pool, recursos del servidor y conexiones segun carga real.
3. Priorizar fixes de estabilidad antes que redisenos estructurales.

#### Etapa B - Optimizacion puntual de alto retorno (2-6 semanas)

Objetivo:

- Bajar latencia y consumo sin tocar todo el nucleo funcional.

Acciones:

1. Separar BLOBs y metadata en archivos.
2. Afinar listados y consultas de entidades anchas.
3. Crear proyecciones especificas para pantallas de alta demanda.
4. Revisar stored procedures e indices de mayor criticidad operativa.

#### Etapa C - Saneamiento del modelo y desacoplamiento (6-12 semanas)

Objetivo:

- Reducir deuda del modelo de datos y del acoplamiento entre capas.

Acciones:

1. Separar DTOs de lectura de entidades transaccionales.
2. Formalizar snapshots operativos frente a datos maestros.
3. Desacoplar seguridad, reservas, movimientos y reclamos en contratos mas claros.

#### Etapa D - Modernizacion de plataforma (12+ semanas)

Objetivo:

- Reducir riesgo estructural de soporte y seguridad a mediano plazo.

Acciones:

1. Actualizar librerias legacy gradualmente.
2. Definir ruta de migracion de framework.
3. Incorporar CI/CD, pruebas automatizadas y controles de seguridad continuos.

## 10) Recomendaciones priorizadas

Prioridad inmediata (0-7 dias):

1. Rotacion/revocacion de credenciales expuestas.
2. Migrar todo endpoint/integracion a HTTPS.
3. Implementar proteccion CSRF y baseline de autorizacion consistente.

Prioridad corta (1-4 semanas):

4. Actualizar dependencias vulnerables y alinear versiones de Newtonsoft.Json.
5. Endurecer manejo de errores y monitoreo.
6. Aumentar cobertura de pruebas en procesos criticos.

Prioridad media (1-3 meses):

7. Programa de modernizacion de plataforma y reduccion de deuda tecnica.
8. Estrategia de desacople de integraciones legacy.

## 11) Evidencias clave (referencias)

- SIS_INTRANET.sln
- SIS_INTRANET/Web.config
- SIS_INTRANET/Views/Web.config
- SIS_INTRANET/App_Start/WebApiConfig.cs
- SIS_INTRANET/App_Start/RouteConfig.cs
- SIS_INTRANET/Filters/UsuarioAutenticadoFilter.cs
- SIS_INTRANET/Apis/RepositorioEnviosController.cs
- SIS_INTRANET/Controllers/LoginController.cs
- INTRANET.Interfaces/Helper/SSLHelper.cs

---

Nota: este informe se basa en inspeccion estatica del repositorio. La criticidad final debe consolidarse con pruebas dinamicas (DAST), pentest controlado y validacion de configuracion real de infraestructura (IIS, certificados, firewall, WAF, AD/SSO, segmentacion de red).
