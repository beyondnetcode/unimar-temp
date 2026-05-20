# ADR-0002: Estrategia de Actualización Segura de Framework y Librerías (Max Compatibility)

**Status:** APPROVED  
**Date:** 2026-05-11  
**Author:** ALBERTO ARROYO  
**Assigned Developer:** Edwin Aztuvilca  
**Severity:** Technical Debt / Security Remediation  

---

## 1. RESUMEN EJECUTIVO

**Problema:**
El ecosistema web de Unimar opera sobre un stack tecnológico de 2012 (.NET 4.5, MVC 4) con múltiples vulnerabilidades críticas en dependencias clave (Newtonsoft, SharpZipLib, jQuery). La acumulación de deuda técnica representa un riesgo de seguridad inmediato.

**Decisión:**
Implementar una estrategia de actualización **"In-Place"**. Esta estrategia define el **techo máximo de versión** que garantiza compatibilidad regresiva a nivel de código y API, evitando una reescritura completa de las soluciones hacia .NET moderno (que queda reservado para fases posteriores de descarte/sustitución).

---

## 2. CONTEXTO Y LÍMITES DE COMPATIBILIDAD

### 2.1 Restricciones de Diseño
- **Sin Reescritura de Vistas:** No se permite forzar el cambio de Bootstrap 3 a 5 o migración a SPAs en este ciclo.
- **Mantener Runtime .NET:** El objetivo es el runtime final del .NET Framework tradicional (v4.8) que no requiere reestructuración de SDK styles.
- **Retrocompatibilidad Operativa:** La actualización debe permitir correr los aplicativos actuales en el servidor de destino sin modificar la lógica de negocio principal.

---

## 3. MATRIZ CONSOLIDADA DE VERSIONES SEGURO / TARGET

Esta tabla consolida el inventario base de las soluciones (Intranet y Extranet) y define el horizonte de compatibilidad para el plan de remediación.

### 3.1 Core Platform y Framework base

| Componente | Versión Actual | **Target Recomendado** | Impacto / Estrategia |
| :--- | :---: | :---: | :--- |
| **.NET Framework** | 4.5 | **4.8** | **Mínimo.** Permite TLS 1.2 nativo y APIs modernas de Framework. |
| **ASP.NET MVC** | 4.0.20710.0 | **5.2.9 / 5.3+** | **Bajo.** Requiere actualización de NuGet y ajuste de web.config. |
| **ASP.NET WebAPI** | 4.0 | **WebAPI 2 (5.2.9)**| **Bajo.** Se actualiza en conjunto con el stack MVC. |
| **ASP.NET Razor** | 2.0 | **3.x** | **Transparente.** Se actualiza junto con MVC. |
| **IIS / Windows OS**| 2008 / 2008 R2 | **10 / Windows Server 2022**| **Nulo en código.** Obligatorio para solventar crisis de TLS. |

### 3.2 Dependencias de Backend (NuGet / Assemblies)

| Librería | Versión Actual | **Target Recomendado** | Notas de Compatibilidad / Mitigación |
| :--- | :---: | :---: | :--- |
| **Newtonsoft.Json** | 4.5.6 / 5.0.4 | **13.0.x** | Compatible. Parchea vulnerabilidades de deserialización. Requiere `bindingRedirects`. |
| **log4net** | 2.0.8 / 1.2.10 | **2.0.15+** | Compatible. Homologar todas las referencias DLL y NuGet en una sola versión. |
| **SharpZipLib** | 0.86.0 | **System.IO.Compression**| **Cambio Mínimo.** Reemplazar por librería nativa de .NET 4.5+ para compresión ZIP estándar. |
| **DocumentFormat.OpenXml**| 2.7.2 | **2.20+** | Compatible. Corrige procesamiento malicioso de XML (XXE). |
| **NPOI** | 2.3.0 | **2.6.x** | Compatible. Mejora rendimiento de manipulación de Excel. |
| **EPPlus** | Desconocida (DLL)| **4.5.3.3** | **Límite legal.** Última versión bajo licencia LGPL (gratis). Versiones 5+ requieren licencia comercial. |
| **iTextSharp** | 10.0.0.0 | **5.5.x** | **Mantener rama.** No migrar a iText 7 (cambia licenciamiento a AGPL y APIs). |

### 3.3 Dependencias de Frontend (JavaScript / CSS)

| Librería | Versión Actual | **Target Recomendado** | Notas de Compatibilidad / Mitigación |
| :--- | :---: | :---: | :--- |
| **jQuery** | 2.2.4 | **3.7.1** | **Medio.** Utilizar obligatoriamente el plugin `jQuery Migrate` para no romper scripts legacy. |
| **Bootstrap** | 3.3.6 | **3.4.1** | Compatible. Última revisión de la rama 3.x que corrige XSS sin forzar reescritura de clases HTML. |
| **Moment.js** | 2.13.0 | **2.30.x** | Compatible. Parches finales antes de evaluar reemplazo (Day.js). |
| **DataTables** | 1.10.19 | **1.13.x / 2.0** | Compatible. Versiones actuales mantienen retrocompatibilidad fuerte con el ecosistema anterior. |

---

## 4. ESTRATEGIA DE EJECUCIÓN (OLEADAS)

Para garantizar la estabilidad, la ejecución se agrupa por el riesgo de introducir regression bugs:

### Oleada 1: Seguridad Crítica (Bajo Impacto Funcional)
1. Actualizar **Newtonsoft.Json** a 13.x y forzar bindings.
2. Unificar **log4net** y mover todas las DLLs locales a su paquete NuGet respectivo.
3. Subir **DocumentFormat.OpenXml** a 2.20+.

### Oleada 2: Seguridad de Frontend y Limpieza
1. Integrar `jQuery Migrate` y subir **jQuery** a la serie 3.x estable.
2. Parchear **Bootstrap** a la versión 3.4.1.
3. Rastrear DLLs desconocidas (EPPlus, iText) y reemplazarlas por paquetes oficiales NuGet bloqueados en versiones compatibles.

### Oleada 3: Salto de Framework Base
1. Actualizar Target Framework de los proyectos (`.csproj`) a **4.8**.
2. Ejecutar update de paquete **Microsoft.AspNet.Mvc** a la versión 5.2.9+.
3. Recompilar y correr Smoke Tests E2E sobre el nuevo IIS 10/13.

---

## 5. CONSECUENCIAS

### Positivas (+)
- **Eliminación de CVEs Críticos:** Mitigación de las brechas principales identificadas en el reporte OWASP consolidado.
- **Soporte Extendido:** Salida del soporte finalizado de MVC 4, pasando a una rama con soporte extendido de Microsoft vía .NET Framework 4.8.
- **Menor Costo:** 90% más barato y rápido que una reescritura a .NET 8/Core.

### Riesgos y Mitigaciones (-)
- **Breaking Changes Sutiles en Newtonsoft:** Se mitiga configurando correctamente los validadores de tipo y verificando serialización de fechas en `Web.config`.
- **Conflictos de Bindings:** Se resuelve corriendo el comando `Update-Package -reinstall` a nivel de proyecto para regenerar las redirecciones automáticas de ensamblados en los archivos de configuración.

---

## 6. APROBACIÓN TÉCNICA

| Rol | Nombre | Firma | Fecha |
| :--- | :--- | :---: | :--- |
| **Lead Architect** | - | _ _ _ | 2026-05-11 |
| **Dev Team Lead** | - | _ _ _ | 2026-05-11 |
| **Developer** | Edwin Aztuvilca | _SIGNED_ | 2026-05-11 |
| **Seguridad Info** | - | _ _ _ | 2026-05-11 |

---

## 7. CHANGELOG

| Versión | Fecha | Autor | Descripción de cambios |
| :--- | :--- | :--- | :--- |
| 1.0 | 2026-05-11 | ALBERTO ARROYO | Creación del ADR consolidando los reportes de Intranet y Extranet. |
