# Inventario y Plan de Remediación de Librerías Externas (Extranet)

Este documento consolida el inventario de librerías externas detectadas en la solución Extranet, evaluando su nivel de obsolescencia, criticidad y planteando un plan estructurado para su actualización y control continuo.

## 1. Inventario Base Detectado

| Librería | Versión detectada | Descripción de problema | Criticidad |
| :--- | :--- | :--- | :--- |
| **Microsoft.AspNet.Mvc** | 4.0.20710.0 | Stack ASP.NET MVC 4 muy antigua; alto riesgo de vulnerabilidades históricas sin mitigaciones actuales. | Crítica |
| **Microsoft.AspNet.WebApi / Core / WebHost / Client** | 4.0.20710.0 | Web API 4 legado; mantenimiento y hardening limitados frente a estándares actuales. | Crítica |
| **Microsoft.AspNet.Razor** | 2.0.20710.0 y 2.0.30506.0 | Motor de vistas antiguo; superficie de riesgo por componentes legacy. | Alta |
| **Microsoft.AspNet.WebPages** | 2.0.20710.0 y 2.0.30506.0 | Dependencia legacy del stack MVC 4; deuda técnica y seguridad acumulada. | Alta |
| **Newtonsoft.Json** | 4.5.6 (NuGet) y 6.0.0.0 (DLL) | Versiones muy antiguas; historial amplio de fallas corregidas en ramas más nuevas. | Alta |
| **Microsoft.Net.Http** | 2.0.20710.0 | Componente antiguo de transporte HTTP; riesgo por obsolescencia. | Media |
| **Microsoft.Web.Infrastructure** | 1.0.0.0 | Versión muy antigua y con poco valor hoy; componente legacy. | Media |
| **log4net** | 2.0.8 (NuGet) y 1.2.10.0 (DLL) | Coexisten dos líneas antiguas; posible inconsistencia y exposición por versiones desfasadas. | Alta |
| **jQuery** | 2.2.4 | Rama antigua con múltiples issues históricas de XSS/prototype pollution en ecosistemas asociados. | Alta |
| **Bootstrap** | 3.3.6 | Versión vieja; varias correcciones de seguridad y compatibilidad en versiones posteriores. | Alta |
| **DataTables** | 1.10.19 | Desactualizada respecto a ramas actuales; riesgo medio por plugins auxiliares legacy. | Media |
| **Moment.js** | 2.13.0 | Librería en mantenimiento mínimo y versión antigua; riesgo por deuda y bugs no priorizados. | Media |
| **Chart.js** | 2.1.4 | Muy antigua; mejoras de seguridad y estabilidad en ramas posteriores no aplicadas. | Media |
| **Font Awesome** | 4.6.3 | Muy antigua; riesgo bajo-medio (principalmente mantenimiento/compatibilidad). | Media |
| **Bootbox** | 5.1.1 | No crítica por sí sola, pero antigua frente a últimas revisiones del ecosistema Bootstrap/jQuery. | Media |
| **JSZip** | 3.1.3 | Antigua; versiones nuevas corrigen defectos y endurecen comportamiento en parsing. | Media |
| **bootstrap-datetimepicker** | 4.15.35 | Plugin legacy sin evolución activa fuerte; depende de jQuery/moment antiguos. | Media |
| **select2** | 4.0.3 | Desactualizada; posibles problemas de compatibilidad y fixes faltantes. | Media |
| **iTextSharp** | Ensamblado 10.0.0.0 | Referencia por DLL local, difícil trazabilidad de parcheo/licencia exacta. | Alta |
| **EPPlus** | No declarada (DLL local) | Sin versionado explícito; riesgo de seguridad y licenciamiento no verificable. | Alta |
| **Microsoft.Office.Interop.Word** | 14.0.0.0 | Dependencia COM antigua; problemas de despliegue/seguridad en servidor. | Alta |
| **Microsoft.ReportViewer.WinForms** | 11.0.0.0 | Muy antiguo; dependencia legacy de runtime/reporting. | Media |
| **jqGrid, fuelux, gritter, flot, ace, etc.** | N/A | No versionado en metadatos. Requiere auditoría de código o migración a alternativas modernas. | Media |

---

## 2. Plan Rápido de Actualización por Oleadas

Para mitigar los riesgos sin desestabilizar por completo el sistema en producción, se propone la siguiente estrategia por oleadas (Waves):

### Oleada 1: Critical Security Patches & Core Backend (Plazo Corto)
**Objetivo:** Cerrar vulnerabilidades severas de back-end y unificar versiones críticas con bajo impacto visual en el cliente.
- **Newtonsoft.Json**: Actualizar a la versión 13.x estable. Eliminar referencias cruzadas y forzar `bindingRedirect` en el `web.config` si es necesario.
- **log4net**: Homologar a la última versión (2.0.15+). Reemplazar la DLL local por el paquete de NuGet para centralizar el versionado.
- **iTextSharp / EPPlus**: Reemplazar DLLs locales por paquetes NuGet. Evaluar licenciamiento de iTextSharp (AGPL) o migrar a iText 7 / alternativas abiertas (ej. PdfSharp). EPPlus debe moverse a versión 4.x (última Open Source) o 5+ si se tiene licencia comercial.

### Oleada 2: Frontend Security & Obsolete Plugins (Plazo Medio)
**Objetivo:** Reducir superficie de ataque XSS (Cross-Site Scripting) y modernizar el stack de cliente.
- **jQuery**: Actualizar a la serie 3.x (ej. 3.7.1). Requerirá el uso de `jQuery Migrate` temporalmente para identificar código roto.
- **Moment.js**: Reemplazar con alternativas modernas y ligeras (ej. Day.js, date-fns) o aplicar parche de seguridad si está disponible para rama 2.x.
- **DataTables, select2, JSZip**: Subir a versiones actuales que soporten jQuery 3.x.

### Oleada 3: Core MVC Framework (Plazo Medio-Largo)
**Objetivo:** Salir de la deuda técnica extrema en el framework base de .NET.
- **ASP.NET MVC y WebAPI**: Migrar el proyecto de MVC 4 a MVC 5 (ASP.NET 4.8). Esta es una migración "in-place" que no requiere reescribir a .NET Core todavía, pero permite aprovechar parches de seguridad recientes del runtime de .NET Framework.
- **Dependencias COM**: Remover `Microsoft.Office.Interop.Word` y reemplazar con SDKs como OpenXML SDK para evitar bloqueos en el IIS por automatización de Office.

### Oleada 4: Modernización de Interfaz (Plazo Largo)
**Objetivo:** Refactorizar el diseño legacy.
- **Bootstrap 3.3.x a 5.x**: Migración compleja que requerirá reescribir plantillas (Razor views).
- **Plugins Visuales (Bootbox, Chart.js, FuelUX, Ace)**: Reemplazar dependencias muertas por componentes de interfaz modernos nativos.

---

## 3. Matriz Riesgo vs Esfuerzo

| Componente a Actualizar | Riesgo de No Actuar | Esfuerzo de Actualización | Prioridad de Ejecución |
| :--- | :---: | :---: | :---: |
| **Newtonsoft.Json** | Alto | Bajo | 1 - Inmediata |
| **log4net (unificar versión)** | Medio | Bajo | 2 - Inmediata |
| **DLLs Locales (iTextSharp, EPPlus)** | Alto | Medio | 3 - Corto Plazo |
| **jQuery + Migrate** | Alto | Medio | 4 - Corto Plazo |
| **ASP.NET MVC 4 a MVC 5** | Crítico | Alto | 5 - Medio Plazo |
| **Office Interop a OpenXML** | Alto | Alto | 6 - Medio Plazo |
| **Bootstrap 3 a 5** | Medio | Muy Alto | 7 - Largo Plazo |
| **Moment.js a Day.js** | Medio | Medio | 8 - Largo Plazo |

---

## 4. Matriz Priorizada de Remediación (Formato CSV)

```csv
Libreria,VersionActual,VersionRecomendada,Criticidad,Esfuerzo,Oleada,AccionRecomendada
Newtonsoft.Json,4.5.6/6.0.0.0,13.0.x,Alta,Bajo,1,Actualizar vía NuGet y unificar bindings
log4net,1.2.10/2.0.8,2.0.15+,Alta,Bajo,1,Unificar a única versión vía NuGet
iTextSharp,10.0.0.0 (DLL),5.5.x o migrar,Alta,Medio,1,Reemplazar DLL por NuGet. Revisar licencia AGPL
EPPlus,No declarada (DLL),4.5.3 (Free) o 6.x (Comercial),Alta,Medio,1,Auditar uso y mover a NuGet
jQuery,2.2.4,3.7.x,Alta,Medio,2,Actualizar e integrar jQuery Migrate
Microsoft.AspNet.Mvc,4.0.20710.0,5.2.9,Critica,Alto,3,Migrar proyecto a MVC 5 y .NET Framework 4.8
Microsoft.AspNet.WebApi,4.0.20710.0,5.2.9,Critica,Alto,3,Migrar proyecto a WebAPI 2
Microsoft.Office.Interop.Word,14.0.0.0,N/A,Alta,Alto,3,Reemplazar por OpenXML SDK
Bootstrap,3.3.6,5.3.x,Alta,Muy Alto,4,Reescribir plantillas y vistas UI
Moment.js,2.13.0,Day.js o date-fns,Media,Medio,4,Refactorizar uso de fechas en JS
```

---

## 5. Script para Inventario Automático en CI

El siguiente script en PowerShell (compatible con Pipelines de Azure DevOps, Jenkins, GitLab CI o GitHub Actions) inspecciona de manera repetible los archivos `packages.config` y `.csproj` para generar un inventario automatizado que previene el ingreso de librerías locales (DLLs sin NuGet) y avisa sobre dependencias obsoletas.

**Archivo sugerido: `scripts/ci-inventory.ps1`**

```powershell
<#
.SYNOPSIS
    Genera un inventario automatizado de las librerías NuGet y reporta DLLs locales.
#>

param (
    [string]$SolutionPath = ".\",
    [string]$OutputPath = ".\inventario_dependencias.csv"
)

Write-Host "Iniciando escaneo de dependencias en $SolutionPath..." -ForegroundColor Cyan

$inventory = @()

# 1. Escanear packages.config para NuGet
$packageFiles = Get-ChildItem -Path $SolutionPath -Filter "packages.config" -Recurse
foreach ($file in $packageFiles) {
    [xml]$xml = Get-Content $file.FullName
    foreach ($pkg in $xml.packages.package) {
        $inventory += [PSCustomObject]@{
            Proyecto = $file.Directory.Name
            Libreria = $pkg.id
            Version = $pkg.version
            Origen = "NuGet (packages.config)"
        }
    }
}

# 2. Escanear archivos .csproj buscando referencias a DLLs locales (HintPaths que no apunten a \packages\)
$projectFiles = Get-ChildItem -Path $SolutionPath -Filter "*.csproj" -Recurse
foreach ($proj in $projectFiles) {
    [xml]$xmlProj = Get-Content $proj.FullName
    
    # Manejar el namespace de MSBuild si existe
    $ns = New-Object System.Xml.XmlNamespaceManager($xmlProj.NameTable)
    $ns.AddNamespace("msb", "http://schemas.microsoft.com/developer/msbuild/2003")

    $references = $xmlProj.SelectNodes("//msb:Reference | //Reference", $ns)
    foreach ($ref in $references) {
        $hintPath = $ref.HintPath
        if ($hintPath -and $hintPath -notmatch "\\packages\\") {
            $inventory += [PSCustomObject]@{
                Proyecto = $proj.Directory.Name
                Libreria = $ref.Include.Split(',')[0]
                Version = "Desconocida/Local"
                Origen = "DLL Local ($hintPath)"
            }
        }
    }
}

# 3. Escanear package.json para librerías de Frontend (si aplica)
$npmFiles = Get-ChildItem -Path $SolutionPath -Filter "package.json" -Recurse -Exclude "node_modules"
foreach ($npm in $npmFiles) {
    $json = Get-Content $npm.FullName | ConvertFrom-Json
    if ($json.dependencies) {
        $json.dependencies.psobject.properties | ForEach-Object {
            $inventory += [PSCustomObject]@{
                Proyecto = $npm.Directory.Name
                Libreria = $_.Name
                Version = $_.Value
                Origen = "NPM (package.json)"
            }
        }
    }
}

# 4. Exportar y mostrar advertencias
if ($inventory.Count -gt 0) {
    $inventory | Export-Csv -Path $OutputPath -NoTypeInformation -Encoding UTF8
    Write-Host "Inventario guardado en: $OutputPath" -ForegroundColor Green
    
    $localDlls = $inventory | Where-Object { $_.Origen -like "DLL Local*" }
    if ($localDlls.Count -gt 0) {
        Write-Host "`n[ADVERTENCIA] Se detectaron referencias a DLLs locales. Deberían migrarse a NuGet:" -ForegroundColor Yellow
        $localDlls | Format-Table Proyecto, Libreria, Origen
    }
} else {
    Write-Host "No se encontraron dependencias." -ForegroundColor Yellow
}
```

### Instrucciones para CI:
1. Guardar el código anterior como `ci-inventory.ps1`.
2. Ejecutarlo en cada compilación (Build Step) del CI/CD para generar `inventario_dependencias.csv`.
3. Adicionar una regla para que falle el Build si se detectan nuevas `DLL Local` o si versiones críticas no cumplen un umbral.
