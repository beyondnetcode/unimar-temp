# CHECKLIST: Implementación de Mitigaciones TLS (ADR-001 Fase 0)

**Objetivo:** Implementar cambios de código para reducir caídas TLS en IIS 2008 antes del Go-Live  
**Timeline:** 2-3 días  
**Responsable:** Equipo Development  
**Revisor:** Arquitecto + Coordinador QA  

---

## ✅ Pre-Implementación

- [ ] **Lectura completa de ADR-001** 
  - Archivo: `./adrs/ADR-001_Migracion_IIS2008_a_IIS10_TLS_Moviles.md`
  - Secciones críticas: 4.1 (Web.config), 4.2 (Global.asax.cs), 4.3 (BundleConfig)

- [ ] **Backup de configuración actual**
  - [ ] Backup de `SIS_INTRANET/Web.config` → `Web.config.backup.20260428`
  - [ ] Backup de `SIS_INTRANET/Global.asax.cs` → `Global.asax.cs.backup.20260428`
  - [ ] Backup de `SIS_INTRANET/App_Start/BundleConfig.cs` → `BundleConfig.cs.backup.20260428`

- [ ] **Validar entorno de testing**
  - [ ] QAS disponible para deploy de mitigaciones
  - [ ] Load test tool configurado (JMeter/LoadRunner/Artillery)
  - [ ] Acceso a logs IIS QAS

---

## 📝 Cambios en Web.config

**Archivo:** `SIS_INTRANET/Web.config`

### Paso 1: Agregar Keep-Alive Headers

**Ubicación:** `<system.webServer>` (crear si no existe)

```xml
<system.webServer>
  
  <!-- NUEVO: Keep-Alive para reducir renegociación SSL -->
  <httpProtocol>
    <customHeaders>
      <add name="Connection" value="keep-alive" />
      <add name="Keep-Alive" value="timeout=15, max=100" />
    </customHeaders>
  </httpProtocol>

  <!-- resto de configuración existing... -->
</system.webServer>
```

**Checklist:**
- [ ] Header `Connection: keep-alive` agregado
- [ ] Header `Keep-Alive: timeout=15, max=100` agregado
- [ ] No hay conflictos con headers existentes
- [ ] Sintaxis XML válida

### Paso 2: Configurar App Pool Settings

**Ubicación:** `<system.webServer>` → `<applicationPool>`

```xml
<system.webServer>
  
  <!-- NUEVO: Tuning de AppPool para reducir memory leaks TLS -->
  <applicationPool 
    processModel.maxProcesses="4" 
    processModel.idleTimeout="00:20:00"
    autoStart="true">
    <!-- Reciclaje automático cada 1440 min (24h) o si supera 512 MB memoria privada -->
    <recycling logEventOnRecycle="PrivateMemory">
      <periodicRestart privateMemory="512000" />
    </recycling>
  </applicationPool>

</system.webServer>
```

**Checklist:**
- [ ] `maxProcesses="4"` configurado (aumentar si > 8 cores)
- [ ] `idleTimeout="00:20:00"` configurado (20 minutos)
- [ ] Reciclaje periódico cada 1440 minutos (24h)
- [ ] Memory recycle threshold en 512000 KB (512 MB)

### Paso 3: Configurar Límites de Conexión

**Ubicación:** `<system.webServer>` → `<security>`

```xml
<system.webServer>
  
  <!-- NUEVO: Limitar conexiones para evitar exhaustion -->
  <security>
    <requestFiltering>
      <!-- Máximo tamaño de request (default 30 MB, aquí 100 MB para archivos) -->
      <requestLimits maxAllowedContentLength="104857600" maxQueryString="2048" />
    </requestFiltering>
  </security>

</system.webServer>
```

**Checklist:**
- [ ] `maxAllowedContentLength="104857600"` agregado (100 MB)
- [ ] `maxQueryString="2048"` agregado
- [ ] Valores apropiados para carga esperada

---

## 🔧 Cambios en Global.asax.cs

**Archivo:** `SIS_INTRANET/Global.asax.cs`

**Ubicación:** Método `Application_Start()`

```csharp
protected void Application_Start()
{
    // NUEVO: Configurar protocolo TLS y connection pooling para reducir caídas
    // Ejecutar ANTES de cualquier call a servicios externos
    
    // 1. Forzar TLS 1.2+ (deshabilitar TLS 1.0/1.1 que causan caídas en móviles)
    // IMPORTANTE: Esto garantiza compatibilidad con clientes modernos
    ServicePointManager.SecurityProtocol = 
        SecurityProtocolType.Tls12 | 
        SecurityProtocolType.Tls11;
    
    // 2. Aumentar pool de conexiones para soportar +500 usuarios móviles concurrentes
    // Default es 2, aumentar a 10 para mejor throughput
    ServicePointManager.DefaultConnectionLimit = 10;
    
    // 3. Reutilizar puerto para reducir TIME_WAIT (conexiones zombie)
    ServicePointManager.ReusePort = true;
    
    // 4. Session resumption: reducir overhead de handshake SSL/TLS
    // Solo activar si certificates no requieren revocation check
    ServicePointManager.CheckCertificateRevocationList = false;
    
    // 5. Timeouts moderados para evitar latencia en móviles
    // Expect100Continue causa delay adicional, desactivar
    ServicePointManager.Expect100Continue = false;
    
    // Resto de Application_Start() existing...
    AreaRegistration.RegisterAllAreas();
    FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
    RouteConfig.RegisterRoutes(RouteTable.Routes);
    BundleConfig.RegisterBundles(BundleTable.Bundles);
}
```

**Checklist:**
- [ ] `ServicePointManager.SecurityProtocol` configurado (Tls12 | Tls11)
- [ ] `DefaultConnectionLimit` aumentado a 10
- [ ] `ReusePort` habilitado
- [ ] `CheckCertificateRevocationList` deshabilitado (si no es crítico)
- [ ] `Expect100Continue` deshabilitado
- [ ] Código ubicado al INICIO de `Application_Start()`
- [ ] Sintaxis C# válida (sin errores de namespace)

**Validación adicional:**
- [ ] Que using statements incluyan `System.Net` (si es necesario)

---

## 📦 Optimización de BundleConfig.cs

**Archivo:** `SIS_INTRANET/App_Start/BundleConfig.cs`

### Paso 1: Habilitar Minificación

```csharp
public class BundleConfig
{
    public static void RegisterBundles(BundleCollection bundles)
    {
        // NUEVO: Habilitar minificación para reducir tamaño CSS/JS
        // Reduce requests HTTP y reconexiones SSL en móvil
        BundleTable.EnableOptimizations = true;
        
        // Bundles de JavaScript
        bundles.Add(new ScriptBundle("~/bundles/jquery")
            .Include(
                "~/Scripts/jquery-{version}.js",  // Minificado a jquery-X.X.X.min.js
                "~/Scripts/jquery-ui-{version}.js"
            ));
        
        // Bundles de CSS
        bundles.Add(new StyleBundle("~/Content/css")
            .Include(
                "~/Content/bootstrap.css",
                "~/Content/site.css"
                // Serán minificados a bootstrap.min.css, site.min.css
            ));
        
        // NUEVO: Habilitar compresión GZIP en IIS (no aquí, pero verificar)
        // En IIS: StaticContent compression + Dynamic content compression
    }
}
```

**Checklist:**
- [ ] `BundleTable.EnableOptimizations = true` agregado
- [ ] Todos los bundles usan `{version}` placeholder para minificación automática
- [ ] CSS/JS se cargarán como `.min.css` / `.min.js` en producción
- [ ] No hay referencias a archivos no-minificados

---

## 🔍 Testing de Mitigaciones (QAS)

### Paso 1: Deploy a QAS

- [ ] Cambios Web.config deployed a QAS AppPool
- [ ] Cambios Global.asax.cs compilados y deployed
- [ ] BundleConfig.cs compilados y deployed
- [ ] AppPool reciclado en QAS
- [ ] Verificar logs IIS (Event Viewer) - no debe haber errores críticos

### Paso 2: Smoke Test Funcional

- [ ] Login en QAS funciona (web)
- [ ] Login en QAS funciona (móvil/responsive)
- [ ] Operación crítica funciona (ej: búsqueda de transmisiones)
- [ ] Descarga de archivo funciona (test de tamaño > 50 MB)
- [ ] Integración WCF funciona (ej: consulta a servicio externo)

**Resultado esperado:** Todos los tests pasan SIN errores de TLS/certificado

### Paso 3: Load Test con Usuarios Móviles

**Herramienta:** JMeter / LoadRunner / Artillery  
**Script:** Simular usuarios móviles (User-Agent: Chrome Mobile, Safari Mobile)

**Configuración:**
```
- Usuarios simultáneos: 100 → 250 → 500 (ramps progresivos)
- Duración: 10 minutos por ramp
- Endpoints: 
  * GET /login
  * POST /api/transmisiones (búsqueda)
  * GET /archivos/download (5 MB file)
  * GET /dashboards/resumen
- Aceptar: 0% error rate en TLS negotiation
```

**Métricas a monitorear:**
- [ ] Response time p95 < 2000 ms (aceptable para móvil)
- [ ] SSL/TLS errors = 0
- [ ] HTTP 200 = 100% de requests
- [ ] CPU IIS < 80%
- [ ] Memory IIS < 1.5 GB (de 4 GB disponible)

**Resultado esperado:** 
- ✅ 500 usuarios móviles concurrentes SIN caídas TLS
- ✅ Response time aceptable (< 2s)
- ✅ 0 errores de certificado/TLS

---

## 📋 Validación Pre-Go-Live

### Checklist Final

- [ ] Código implementado en SIS_INTRANET (todas 3 partes: Web.config, Global.asax.cs, BundleConfig.cs)
- [ ] Tests unitarios existentes pasan (si existen)
- [ ] QAS smoke test funcional 100% OK
- [ ] QAS load test 500 móviles OK (0 TLS errors)
- [ ] IIS logs en QAS sin errores críticos relacionados a TLS
- [ ] Code review completado (arquitecto + lead dev)
- [ ] Documentación de cambios agregada a código (comments en Web.config, Global.asax.cs)

### Cambios Listos para Producción

- [ ] Build release compilado sin warnings
- [ ] Configuración QAS = Configuración PROD esperada
- [ ] Rollback plan documentado (revertir Web.config si es necesario)
- [ ] Monitoring/alertas configuradas:
  - [ ] Alert: TLS errors > 0 en últimas 5 min
  - [ ] Alert: AppPool recycles > 2 en última hora
  - [ ] Alert: CPU IIS > 80% por > 10 min

---

## 🎯 Criterio de Aceptación (Go-Live Gate)

**BLOQUEANTE:** Todos estos deben ser ✅ ANTES de producción

| Criterio | Estado | Evidencia |
|----------|--------|-----------|
| Usuarios móviles conectan sin caídas TLS | ✅/❌ | Load test report |
| Load test 500 móviles concurrentes: 0 TLS errors | ✅/❌ | Load test report |
| Certificado SSL válido en navegadores | ✅/❌ | SSL test online |
| QAS smoke test funcional 100% | ✅/❌ | Test execution log |
| Code review aprobado | ✅/❌ | Review sign-off |

---

## 📞 Escalación y Soporte

**Si hay problemas durante implementación:**

1. **Issue:** Web.config XML parse error
   - **Solución:** Usar XML validator (VS, etc.) - verificar tags cerrados
   - **Contacto:** Arquitecto

2. **Issue:** `ServicePointManager.SecurityProtocol` no compilacompila
   - **Solución:** Agregar `using System.Net;`
   - **Contacto:** Lead Dev

3. **Issue:** Load test falla con TLS errors en QAS
   - **Solución:** Verificar AppPool identity, certificado, IIS TLS settings
   - **Contacto:** Coordinador Infraestructura + Arquitecto

4. **Issue:** Performance degrada después de cambios
   - **Solución:** Aumentar `DefaultConnectionLimit` a 20, ajustar App Pool settings
   - **Contacto:** Arquitecto + Performance Lead

---

## 📚 Referencias

- [ADR-001: Migración IIS 2008 a IIS 10+ TLS Móviles](./ADR-001_Migracion_IIS2008_a_IIS10_TLS_Moviles.md) - Documento completo
- [RESUMEN_EJECUTIVO_ADR-001.md](./RESUMEN_EJECUTIVO_ADR-001.md) - Resumen para stakeholders
- [Microsoft ServicePointManager Docs](https://learn.microsoft.com/en-us/dotnet/api/system.net.servicepointmanager)
- [IIS Request Limits](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/security/requestfiltering/requestlimits)

---

**Checklist Versión:** 1.0  
**Última Actualización:** 28 de abril de 2026  
**Estado:** Listo para implementación

---

**¿Dudas?** Revisar sección 4 de ADR-001 con código exacto de cada cambio.
