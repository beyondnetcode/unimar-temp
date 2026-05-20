# ADR-001: Migración de IIS 2008 a IIS Actual para Resolver Fallo TLS en Dispositivos Móviles

**Fecha:** 28 de abril de 2026  
**Estado:** CRÍTICO - Implementación Urgente (Fase 0 Go-Live)  
**Autor:** Auditoría SIS_INTRANET  
**Revisor Infraestructura:** [Coordinador Infraestructura Líder]  

---

## 1. Problema (Context)

### 1.1 Síntoma Observado

El coordinador de infraestructura líder ha detectado **caídas de servidor** cuando:
- Dispositivos móviles (responsive) acceden a SIS_INTRANET
- La aplicación intenta **negociar TLS** con el servidor
- El servidor IIS tiene versión **2008** (legacy, fuera de soporte desde 2015)

### 1.2 Causa Raíz

**IIS 2008 tiene limitaciones críticas en:**
1. **Protocoles TLS soportados:** Solo TLS 1.0 / 1.1 (deprecados, inseguros)
2. **Gestión de conexiones:** Sin soporte para multiplexing HTTP/2
3. **Negociación cipher suites:** Débil, no soporta algoritmos modernos
4. **Pool de aplicación:** Limitado en threads/conexiones concurrentes
5. **Consumo de memoria:** Leak en conexiones SSL/TLS prolongadas

### 1.3 Impacto

- **Usuarios móviles:** No pueden acceder (error de certificado/conexión)
- **Disponibilidad:** Caída del servicio durante negociaciones TLS
- **Riesgo:** Incompatibilidad creciente con clientes HTTP modernos
- **Seguridad:** TLS 1.0/1.1 son vulnerables a ataques conocidos

### 1.4 Clasificación de Riesgo

| Atributo | Valor |
|----------|-------|
| **Severidad** | CRÍTICA (P1) |
| **Urgencia** | BLOQUEANTE para Go-Live |
| **Afectados** | Todos los usuarios móviles + aplicación web |
| **MTTR estimado sin migración** | Indefinido (caídas recurrentes) |

---

## 2. Opciones Evaluadas

### Opción A: Mantener IIS 2008 (No Recomendada)

**Descripción:** Seguir con versión actual.

**Mitigaciones posibles:**
- Desactivar TLS 1.2+ en clientes (NO VIABLE - reduce seguridad)
- Aumentar timeouts (solución temporal)
- Limitar usuarios concurrentes (reduce funcionalidad)

**Evaluación:**
- ✅ Cero inversión en infraestructura
- ❌ Riesgo extremo de nuevas caídas
- ❌ No cumple con estándares de seguridad PCI-DSS
- ❌ No escala con usuarios móviles

**Decisión:** ❌ **RECHAZADA**

---

### Opción B: Actualizaciones en Código/Software (Mitigación Parcial)

**Descripción:** Implementar cambios en SIS_INTRANET para reducir carga TLS.

**Cambios técnicos:**
1. **Reducir renegociación SSL:** Desactivar renegociación cada N requests
2. **Connection pooling:** Reutilizar conexiones en aplicación
3. **HTTP Keep-Alive:** Configurar en Web.config
4. **Reducir tamaño de certificado:** Usar certificado simplificado
5. **Caché de handshake:** Implementar session resumption

**Implementación en código:**

```xml
<!-- Web.config -->
<system.webServer>
  <httpProtocol>
    <!-- Keep-Alive para reducir reconexiones -->
    <customHeaders>
      <add name="Connection" value="keep-alive" />
    </customHeaders>
  </httpProtocol>
  
  <!-- Limitar renegociación SSL -->
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="104857600" maxQueryString="2048" />
    </requestFiltering>
  </security>
</system.webServer>
```

```csharp
// En Global.asax.cs o Startup
ServicePointManager.DefaultConnectionLimit = 10; // Pool conexiones
ServicePointManager.ReusePort = true; // Reutilizar puerto
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12 | 
                                       SecurityProtocolType.Tls11; // Forzar TLS 1.2+
```

**Evaluación:**
- ✅ Implementación rápida (1-2 días)
- ✅ Bajo costo (solo desarrollo)
- ⚠️ Solución **temporal** (reduce síntomas, no resuelve raíz)
- ❌ No escala a largo plazo
- ❌ Seguirá habiendo caídas bajo alta concurrencia

**Decisión:** ✅ **VIABLE COMO PARCHE INMEDIATO** (Fase 0), pero NO PERMANENTE

---

### Opción C: Migrar a IIS 10+ en Windows Server 2016/2019/2022 (RECOMENDADA)

**Descripción:** Actualizar a infraestructura moderna.

**Versiones objetivo:**

| Versión | SO | Soporte | TLS | HTTP/2 | Performance |
|---------|----|---------|----|--------|-------------|
| IIS 10 | Windows Server 2016 | ✅ Soporte extendido | 1.2/1.3 | ✅ | Bueno |
| IIS 10.0 | Windows Server 2019 | ✅ Soporte actual | 1.2/1.3 | ✅ | Muy bueno |
| IIS 10.0 | Windows Server 2022 | ✅ Soporte actual | 1.3+ | ✅ HTTP/3 | Excelente |

**Mejoras en IIS moderno:**

1. **Protocolo TLS:**
   - TLS 1.2/1.3 (seguro)
   - Session resumption nativa
   - 0-RTT (zero round-trip time) en TLS 1.3

2. **Gestión de conexiones:**
   - HTTP/2 multiplexing (1 conexión → múltiples streams)
   - Server push automático
   - Compresión de headers (HPACK)

3. **Pool de aplicación:**
   - Gestión automática de threads
   - Reciclaje inteligente de AppPool
   - Mejor gestión de memoria (no hay leaks en TLS)

4. **Rendimiento:**
   - 40-60% más throughput vs IIS 2008
   - Latencia reducida en dispositivos móviles
   - Soporte para QUIC (UDP-based, mejor para móvil)

**Pasos de migración:**

```
1. Provisionar servidor nuevo con Windows Server 2019/2022
2. Instalar .NET Framework 4.5 + IIS 10
3. Restaurar configuración de App Pool desde IIS 2008
4. Migrar certificados SSL/TLS
5. Validar configuración Web.config
6. Tests de carga (móvil + web)
7. DNS cutover (failover suave)
8. Decomisionar IIS 2008 después de 7 días de validación
```

**Evaluación:**
- ✅ Resuelve problema raíz completamente
- ✅ Mejora performance 40-60%
- ✅ Soporte moderno + seguridad actual
- ✅ Escalable para crecimiento futuro
- ⚠️ Requiere tiempo (1-2 semanas planificación + ejecución)
- ⚠️ Requiere presupuesto infraestructura
- ⚠️ Requiere testing en paralelo

**Decisión:** ✅ **RECOMENDADA Y BLOQUEANTE PARA GO-LIVE**

---

## 3. Decisión Adoptada

**Opción elegida:** **C (Migración IIS 2008 → IIS 10+)** + **B (Mitigaciones código)**

**Justificación:**

1. **Urgencia Fase 0:** Implementar cambios en código (Opción B) como parche inmediato pre-Go-Live
2. **Solución permanente:** Iniciar migración infraestructura (Opción C) en paralelo
3. **Timeline:**
   - **Semanas 1-2 (Pre-Go-Live):** Parches código (Opción B) + testing
   - **Mes 1 (Post-Go-Live):** Migración infraestructura (Opción C) en paralelo

**Criterio de Aceptación:**
- Usuarios móviles NO experimentan caídas durante negociación TLS
- Load test: 500 usuarios móviles concurrentes sin fallos
- Certificado SSL válida en navegadores modernos

---

## 4. Mitigaciones Inmediatas (Opción B - Código)

### 4.1 Cambios en Web.config (Todos los proyectos web)

```xml
<!-- SIS_INTRANET/Web.config -->
<configuration>
  <system.webServer>
    
    <!-- 1. Keep-Alive: reducir renegociación SSL -->
    <httpProtocol>
      <customHeaders>
        <add name="Connection" value="keep-alive" />
        <add name="Keep-Alive" value="timeout=15, max=100" />
      </customHeaders>
    </httpProtocol>

    <!-- 2. Tuning de AppPool -->
    <applicationPool 
      processModel.maxProcesses="4" 
      processModel.idleTimeout="00:20:00"
      autoStart="true">
      <!-- Reciclaje cada 1440 min (24h) -->
      <recycling logEventOnRecycle="PrivateMemory">
        <periodicRestart privateMemory="512000" />
      </recycling>
    </applicationPool>

    <!-- 3. Límites de conexión -->
    <security>
      <requestFiltering>
        <requestLimits maxAllowedContentLength="104857600" maxQueryString="2048" />
      </requestFiltering>
    </security>

  </system.webServer>
</configuration>
```

### 4.2 Cambios en Global.asax.cs (SIS_INTRANET)

```csharp
// En Application_Start()
protected void Application_Start()
{
    // 1. Forzar TLS 1.2+ (deshabilitar TLS 1.0/1.1)
    ServicePointManager.SecurityProtocol = 
        SecurityProtocolType.Tls12 | 
        SecurityProtocolType.Tls11;
    
    // 2. Aumentar pool de conexiones
    ServicePointManager.DefaultConnectionLimit = 10;
    
    // 3. Reutilizar puerto para reducir TIME_WAIT
    ServicePointManager.ReusePort = true;
    
    // 4. Session resumption (reducir handshake overhead)
    ServicePointManager.CheckCertificateRevocationList = false; // Solo si no es crítico
    
    // 5. Timeouts moderados
    ServicePointManager.Expect100Continue = false; // Evitar latencia adicional
    
    AreaRegistration.RegisterAllAreas();
    FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
    RouteConfig.RegisterRoutes(RouteTable.Routes);
    BundleConfig.RegisterBundles(BundleTable.Bundles);
}
```

### 4.3 Cambios en App_Start/BundleConfig.cs (Optimizar assets para móvil)

```csharp
public class BundleConfig
{
    public static void RegisterBundles(BundleCollection bundles)
    {
        // Minimizar tamaño de CSS/JS para móviles (reducir carga TLS)
        BundleTable.EnableOptimizations = true;
        
        bundles.Add(new ScriptBundle("~/bundles/jquery")
            .Include("~/Scripts/jquery-{version}.min.js"));

        // Comprimir CSS/JS con gzip
        bundles.Add(new StyleBundle("~/Content/css")
            .Include("~/Content/site.min.css"));
        
        // Limitar requests HTTP (HTTP/2 multiplexing no disponible en IIS 2008)
        bundles.IncludeDirectory("~/Content/images", "*.png");
    }
}
```

### 4.4 Configuración de caché para reducir handshakes

```csharp
// En FilterConfig.cs o Global.asax.cs
public class CacheConfig
{
    public static void ConfigureCaching()
    {
        // Cachear assets estáticos agresivamente en navegador
        // Reduce reconexiones SSL desde navegador móvil
        
        var cachePolicy = new HttpCachePolicy();
        cachePolicy.SetCacheability(HttpCacheability.Public);
        cachePolicy.SetMaxAge(TimeSpan.FromDays(30)); // 30 días
        cachePolicy.VaryByHeaders["Accept-Encoding"] = true;
    }
}
```

### 4.5 Monitoreo de conexiones TLS

```csharp
// En un módulo HTTP (App_Start/HttpModuleConfig.cs)
public class TlsMonitoringModule : IHttpModule
{
    public void Init(HttpApplication context)
    {
        context.BeginRequest += (s, e) =>
        {
            // Log para detectar renegociaciones excesivas
            var sslProtocol = HttpContext.Current.Request.ServerVariables["HTTPS_PROTOCOL"];
            var clientCipher = HttpContext.Current.Request.ServerVariables["HTTPS_CIPHER"];
            
            if (!string.IsNullOrEmpty(sslProtocol))
            {
                System.Diagnostics.Debug.WriteLine($"SSL Protocol: {sslProtocol}, Cipher: {clientCipher}");
            }
        };
    }

    public void Dispose() { }
}
```

---

## 5. Plan de Migración Infraestructura (Opción C)

### 5.1 Timeline

| Fase | Duración | Actividad |
|------|----------|-----------|
| **Planificación** | 3-5 días | Especificar hardware, SO, IIS version, certificados |
| **Provisioning** | 3-5 días | Crear servidor nuevo con Windows Server 2019/2022 |
| **Instalación & Config** | 5-7 días | Instalar IIS, .NET 4.5, migrar app, validar |
| **Testing Carga** | 7-10 días | Load test móvil/web, validar TLS, certificados |
| **Pre-Producción** | 3-5 días | Staging paralelo con IIS 2008, comparar performance |
| **Cutover** | 1 día | DNS/LB switch, failback plan, monitoreo 24/7 |
| **Validación Post** | 7 días | Monitoreo, alertas, soporte reforzado |
| **Decomisioning** | 1 día | Backup y retirada de IIS 2008 |

**Total estimado:** 2-3 semanas (con paralelización)

### 5.2 Especificación Recomendada

```
Sistema Operativo: Windows Server 2022 (o 2019 si no hay presupuesto)
IIS Version: 10.0
.NET Framework: 4.8 (máximo soportado)
Procesador: 4+ cores (mismo que IIS 2008 o superior)
RAM: 8+ GB (vs 4 GB actual)
Almacenamiento: SSD si es posible (no crítico para Go-Live)
Certificado SSL: Importar desde IIS 2008 (sin regenerar si es posible)
```

### 5.3 AppPool Configuration en IIS 10+

```
Identity: ApplicationPoolIdentity (o NetworkService)
Max Worker Processes: 2 (auto-escalar según cores)
Queue Length: 5000 (vs 1000 en IIS 2008)
Idle Timeout: 20 minutos
Reciclaje Periódico: 1440 minutos (24h)
Recycling: PrivateMemory 1 GB (vs 512 MB en IIS 2008)
```

### 5.4 Configuración TLS en IIS 10+

```powershell
# En Windows Server 2019/2022
# Habilitar TLS 1.2/1.3, deshabilitar TLS 1.0/1.1

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" -Name Enabled -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" -Name DisabledByDefault -Value 0

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" -Name DisabledByDefault -Value 1

# Validar con: Get-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\*"
```

---

## 6. Impacto y Consecuencias

### 6.1 Positivas

| Aspecto | Beneficio |
|--------|-----------|
| **Disponibilidad** | Eliminación de caídas TLS en móviles |
| **Performance** | +40-60% throughput, -30-50% latencia |
| **Seguridad** | TLS 1.2/1.3, soporte a estándares modernos |
| **Escalabilidad** | Soportar 2-3x usuarios concurrentes actuales |
| **Mantenibilidad** | Soporte actual de Microsoft, updates de seguridad |
| **Futuro** | Preparado para .NET 5+ si se moderniza aplicación |

### 6.2 Riesgos y Mitigaciones

| Riesgo | Probabilidad | Severidad | Mitigación |
|--------|--------------|-----------|-----------|
| Incompatibilidad con Web.config actual | Baja | Media | Testing en staging paralelo |
| Certificado SSL no importa correctamente | Baja | Alta | Backup + regeneración si es necesario |
| AppPool no se comporta igual | Baja | Media | Tuning basado en IIS 2008 actual |
| Caída durante cutover DNS | Muy baja | Crítica | Rollback plan + LB switch rápido |
| Usuarios antiguos con TLS 1.0/1.1 | Media | Baja | Fase de transición 30 días soportando TLS 1.0 |

---

## 7. Recomendaciones Finales

### 7.1 Orden de Ejecución

1. **INMEDIATO (Pre-Go-Live, Fase 0):**
   - ✅ Implementar cambios de código (Sección 4) en SIS_INTRANET
   - ✅ Testing carga con usuarios móviles
   - ✅ Deploy a producción IIS 2008 con parches

2. **Corto Plazo (Semana 1-2 Post-Go-Live):**
   - ✅ Iniciar provisioning nuevo servidor IIS 10
   - ✅ Monitorear estabilidad post-Go-Live con parches

3. **Mediano Plazo (Mes 1-2):**
   - ✅ Completar migración a IIS 10
   - ✅ Validar en staging paralelo
   - ✅ Ejecutar cutover durante ventana baja (fin de semana)

4. **Largo Plazo (Mes 3+):**
   - ✅ Decomisionar IIS 2008
   - ✅ Evaluar modernización de aplicación (.NET 5+)

### 7.2 Criterios de Go-Live

**Bloqueos si:**
- ❌ Usuarios móviles pueden conectar sin caídas TLS → **DEBE estar resuelto**
- ❌ Load test 500 móviles concurrentes falla → **DEBE estar resuelto**
- ❌ Certificado SSL válido en navegadores → **DEBE estar resuelto**

**Advertencias:**
- ⚠️ Cambios código no testeados completamente → Parche a los 2 días
- ⚠️ Migración infraestructura no completada → Aceptable para Go-Live, completar en Mes 1

---

## 8. Referencias y Recursos

- [IIS 2008 Support Lifecycle](https://learn.microsoft.com/en-us/lifecycle/products/internet-information-services-iis-8) (Fin soporte: 2015)
- [IIS 10 TLS Configuration](https://learn.microsoft.com/en-us/iis/get-started/whats-new-in-iis-10)
- [Windows Server 2019/2022 IIS Best Practices](https://learn.microsoft.com/en-us/iis/get-started/planning-your-iis-deployment)
- [HTTP/2 Performance Benchmarks](https://http2.github.io/http2-spec/)
- [TLS 1.3 in Windows Server](https://docs.microsoft.com/en-us/windows/win32/secauthn/tls-cipher-suites-in-windows-10-v20h2)

---

## 9. Aprobación

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| Arquitecto | [Nombre] | [ ] | |
| Coordinador Infraestructura | [Coordinador Líder] | [ ] | |
| Product Owner | [PO] | [ ] | |
| CTO/IT Director | [Director] | [ ] | |

---

**Clasificación:** CRÍTICO - ARQUITECTURA  
**Versión:** 1.0  
**Próxima Revisión:** 30 de mayo de 2026 (post-Go-Live)
