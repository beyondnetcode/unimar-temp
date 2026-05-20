# Inventario y Plan de Remediación de Librerías Externas (Intranet)

Este documento detalla el análisis de las librerías externas detectadas en la solución `SIS_INTRANET`, sus vulnerabilidades, nivel de criticidad y las recomendaciones para su remediación progresiva.

## 1. Inventario Base Detectado

| Librería | Versión | Problema / Vulnerabilidad | Criticidad |
| :--- | :--- | :--- | :--- |
| **Microsoft.AspNet.Mvc** | 4.0.20710.0 | Versión de 2012 - End of Support desde 2016. Múltiples CVEs no parchados (XSS, CSRF). | 🔴 CRÍTICA |
| **Microsoft.AspNet.Razor** | 2.0.20710.0 | Versión de 2012 - Fuera de soporte. Vulnerabilidades en template rendering. | 🔴 CRÍTICA |
| **Microsoft.AspNet.WebApi** | 4.0.20710.0 | Versión de 2012 - End of Support. Vulnerabilidades de serialización (CVE-2016-4050). | 🔴 CRÍTICA |
| **Microsoft.AspNet.WebApi.Client** | 4.0.20710.0 | Versión de 2012 - Vulnerabilidades de seguridad conocidas sin parches. | 🔴 CRÍTICA |
| **Microsoft.AspNet.WebApi.Core** | 4.0.20710.0 | Versión de 2012 - Múltiples vulnerabilidades de inyección. | 🔴 CRÍTICA |
| **Microsoft.AspNet.WebApi.WebHost** | 4.0.20710.0 | Versión de 2012 - Vulnerabilidades en routing y ejecución. | 🔴 CRÍTICA |
| **Microsoft.AspNet.WebPages** | 2.0.20710.0 | Versión de 2012 - Vulnerabilidades de XSS y ejecución remota. | 🔴 CRÍTICA |
| **ClosedXML_Excel** | 1.0.0 | Versión de 2013 - Muy antigua. Problemas con XLSX grandes y posibles vulnerabilidades sin parches. | 🔴 ALTA |
| **DocumentFormat.OpenXml** | 2.7.2 | Versión de 2013 - Obsoleta. Vulnerabilidades de procesamiento XML. Recomendado 2.20+. | 🔴 ALTA |
| **Newtonsoft.Json** | 5.0.4 | Versión de 2013 - Múltiples CVEs resueltos en 12.0+. Vulnerabilidades de deserialización. | 🔴 ALTA |
| **SharpZipLib** | 0.86.0 | Versión de 2009 - OBSOLETA. Vulnerabilidades en compresión ZIP (CVE-2018-1000606). | 🔴 ALTA |
| **Antlr** | 3.4.1.9004 | Versión de 2012 - Sin soporte oficial. Vulnerabilidades en análisis de gramática. | ⚠️ MEDIA |
| **Microsoft.AspNet.Web.Optimization** | 1.1.3 | Versión de 2013 - Sin actualizaciones. Problemas con minificación y bundling. | ⚠️ MEDIA |
| **Microsoft.Net.Http** | 2.0.20710.0 | Versión de 2012 - Sin soporte. Problemas de certificados SSL/TLS. | ⚠️ MEDIA |
| **Microsoft.Web.Infrastructure** | 1.0.0.0 | Versión de 2011 - Muy antigua. Componente legacy, sin vulnerabilidades críticas recientes. | ⚠️ MEDIA |
| **NPOI** | 2.3.0 | Versión de 2017 - Desactualizada. Problemas de rendimiento y compatibilidad. | ⚠️ MEDIA |
| **PagedList** | 1.17.0.0 | Versión de 2013 - Muy antigua pero es utilidad simple sin vulnerabilidades críticas. | ⚠️ MEDIA |
| **PagedList.Mvc** | 4.5.0.0 | Versión de 2013 - Complemento de paginación. | ⚠️ MEDIA |
| **WebGrease** | 1.5.2 | Versión de 2013 - Sin mantenimiento. Riesgo bajo. | ⚠️ MEDIA |

---

## 2. Resumen de Criticidad

| Nivel | Cantidad | Librerías |
| :---: | :---: | :--- |
| 🔴 **CRÍTICA** | 6 | Microsoft.AspNet.* (Mvc, Razor, WebApi*2, WebPages) |
| 🔴 **ALTA** | 4 | ClosedXML_Excel, DocumentFormat.OpenXml, Newtonsoft.Json, SharpZipLib |
| ⚠️ **MEDIA** | 10 | Antlr, Web.Optimization, Microsoft.Net.Http, Web.Infrastructure, NPOI, PagedList*, WebGrease |

---

## 3. Plan Rápido de Actualización por Oleadas

### Oleada 1: Parches Críticos de Seguridad (Inmediato - Bloqueante / Post-Arranque)
**Objetivo:** Mitigar las vulnerabilidades más críticas de componentes utilitarios sin cambiar el framework principal.
- **Newtonsoft.Json**: Actualizar de 5.0.4 a la versión 13.0+ (corrige vulnerabilidades de inyección y deserialización).
- **SharpZipLib**: Reemplazar la librería obsoleta de 2009 por la biblioteca nativa `System.IO.Compression` o actualizar a la versión moderna en caso de uso intensivo de APIs antiguas.
- **DocumentFormat.OpenXml**: Actualizar a la versión 2.20+ para prevenir ataques de entidades XML (XXE) y procesamiento malicioso.

### Oleada 2: Core y Rendimiento (Plazo Medio)
**Objetivo:** Mitigar problemas de procesamiento de archivos grandes y deuda técnica operativa.
- **ClosedXML_Excel / NPOI**: Homologar el uso de librerías de Excel. Actualizar NPOI a su rama actual (2.6.x) o sustituir ClosedXML antiguo por EPPlus (verificando licencia) o ClosedXML moderno.
- **PagedList**: Actualizar a forks modernos (como `X.PagedList`) o reemplazar con lógica de paginación nativa/IQueryable de base de datos.

### Oleada 3: Migración de Framework Base (Plazo Largo)
**Objetivo:** Salir del soporte finalizado de Microsoft.
- **ASP.NET MVC 4 / WebApi 4**: Migrar el proyecto a **ASP.NET MVC 5.3+ y WebApi 2** (.NET Framework 4.8), o bien iniciar el proyecto de reescritura progresiva hacia **.NET 6/8**. Esto solucionará todas las dependencias `Microsoft.AspNet.*` críticas actuales.

---

## 4. Matriz Riesgo vs Esfuerzo

| Componente a Actualizar | Riesgo de No Actuar | Esfuerzo de Actualización | Prioridad de Ejecución |
| :--- | :---: | :---: | :---: |
| **Newtonsoft.Json** | Alto | Bajo | 1 - Inmediata |
| **SharpZipLib** | Alto (CVE) | Bajo-Medio | 2 - Inmediata |
| **DocumentFormat.OpenXml** | Alto | Bajo | 3 - Corto Plazo |
| **ClosedXML_Excel / NPOI** | Alto | Medio | 4 - Corto Plazo |
| **ASP.NET MVC 4 a MVC 5** | Crítico | Alto | 5 - Medio Plazo |
| **Migración global a .NET 8**| Crítico | Muy Alto | 6 - Largo Plazo |

---

## 5. Matriz Priorizada de Remediación (Formato CSV)

```csv
Libreria,VersionActual,VersionRecomendada,Criticidad,Esfuerzo,Oleada,AccionRecomendada
Newtonsoft.Json,5.0.4,13.0.x,Alta,Bajo,1,Actualizar vía NuGet para parchear deserialización
SharpZipLib,0.86.0,System.IO.Compression,Alta,Bajo-Medio,1,Remover librería obsoleta y usar la nativa de .NET
DocumentFormat.OpenXml,2.7.2,2.20+,Alta,Bajo,1,Actualizar vía NuGet para prevención XXE
ClosedXML_Excel,1.0.0,Alternativa Moderna,Alta,Medio,2,Sustituir por OpenXML nativo o ClosedXML actualizado
NPOI,2.3.0,2.6+,Media,Medio,2,Actualizar para mejorar performance
Microsoft.AspNet.Mvc,4.0.20710.0,5.2.9+,Critica,Alto,3,Migrar proyecto a MVC 5 y .NET Framework 4.8
Microsoft.AspNet.WebApi,4.0.20710.0,5.2.9+,Critica,Alto,3,Migrar proyecto a WebAPI 2
Microsoft.AspNet.Razor,2.0.20710.0,3.x,Critica,Alto,3,Se actualiza junto con MVC 5
PagedList,1.17.0.0,X.PagedList,Media,Bajo,2,Sustituir por fork moderno
```

---

## 6. Script para Inventario Automático en CI

Puedes utilizar el mismo script de automatización sugerido en la Extranet para auditar el proyecto de Intranet y detectar dependencias obsoletas en el pipeline de DevOps (basado en `packages.config` y `.csproj`).
