# ⚠️ ACCIÓN INMEDIATA - IIS 2008 Crisis & Go-Live Bloqueantes

**Documento de Referencia Rápida**  
**Fecha:** 2026-04-28  
**Prioridad:** P0 - CRÍTICA  
**Owner:** Infrastructure Lead + Development Lead

---

## 🔴 SITUACIÓN CRÍTICA

**IIS 2008 falla bajo carga móvil → caídas de servidor**

- **Síntoma:** Usuarios móvil reportan "Conexión rechazada"
- **Causa:** TLS 1.0/1.1 lento, connection pool débil, sin HTTP/2
- **Impacto:** 40%+ tráfico (móvil) inoperable
- **Timeline:** Resolución requerida 7-10 días máximo para Go-Live

---

## 📋 TAREAS HOY (Fase 0A - 4-6 horas)

### PARA DEV TEAM (Implementar mitigaciones temporales desde código)

**Responsable:** Dev Lead  
**Tiempo:** 4-6 horas  
**Deploy:** Hoy a staging/producción

#### Tarea 1: Connection Pool Manager
- [ ] Crear archivo: `Extranet.data/DbConnectionManager.cs`
- [ ] Código: [Copiar de ADR-0001 Sección 5.1](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md#51-problema-agotamiento-de-connection-pool)
- [ ] Objetivo: Limitar pool size, evitar agotamiento
- [ ] Test: Conexión exitosa en desarrollo

#### Tarea 2: Retry Logic en Controllers
- [ ] Modificar: `ExtranetWeb/Controllers/BaseController.cs`
- [ ] Agregar método: `ExecuteWithRetry()` con exponential backoff
- [ ] Código: [Copiar de ADR-0001 Sección 5.2](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md#52-problema-tls-handshake-timeout-en-móviles)
- [ ] Aplicar a LoginController, llamadas críticas
- [ ] Test: Reintentos funcionan en timeout simulado

#### Tarea 3: Actualizar Web.config - Compression
- [ ] Agregar en: `ExtranetWeb/Web.config` (dentro `<system.webServer>`)
- [ ] Código: [Copiar de ADR-0001 Sección 5.3](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md#53-problema-compresión-ssl-deficiente)
- [ ] Objetivo: Reducir payload 60-70% en móviles
- [ ] Test: Verificar headers Accept-Encoding/Content-Encoding

#### Tarea 4: Actualizar Web.config - Keep-Alive
- [ ] Agregar en: `ExtranetWeb/Web.config` (dentro `<system.webServer>`)
- [ ] Código: [Copiar de ADR-0001 Sección 5.4](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md#54-problema-keep-alive-incorrecto)
- [ ] Objetivo: Reutilización eficiente de conexiones
- [ ] Test: Verificar connection reuse en logs

#### Tarea 5: Aumentar Timeouts en API Calls
- [ ] Modificar: `ExtranetWeb/Util/ApiRest/AutenticacionApiRest.cs`
- [ ] Aumentar timeout: 5s → 30s
- [ ] Código: [Copiar de ADR-0001 Sección 5.5](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md#55-problema-connection-timeout-muy-corto)
- [ ] Test: Llamadas externas no fallan por timeout

#### Tarea 6: Request Throttling
- [ ] Modificar: `ExtranetWeb/Global.asax.cs`
- [ ] Agregar método: `Application_BeginRequest()` con SemaphoreSlim
- [ ] Código: [Copiar de ADR-0001 Sección 5.6](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md#56-problema-solicitudes-simultáneas-saturan-iis-2008)
- [ ] Objetivo: Limitar requests concurrentes (máx 50)
- [ ] Test: Verificar que no rechaza requests válidas

#### Integración & Testing
- [ ] [ ] Compilar solución (verificar 0 errores)
- [ ] [ ] Ejecutar unit tests locales (verde)
- [ ] [ ] Deploy a staging
- [ ] [ ] Smoke test: Login, reserva, API call → OK
- [ ] [ ] **Load test móvil simulado:** 100 usuarios concurrentes × 5 min
  - Métrica esperada: < 5% timeouts (vs actual 30-40%)
- [ ] [ ] **Monitoreo en staging 1h:** Verificar cero crashes
- [ ] [ ] **Aprobar para producción**

**Validación de éxito:**
- ✓ 40-50% reducción en timeout errors (métrica: logging + APM)
- ✓ CPU baseline igual o mejor
- ✓ Cero regresiones funcionales

---

### PARA INFRASTRUCTURE TEAM (Preparar migración IIS)

**Responsable:** Infrastructure Lead  
**Tiempo:** Paralelizar con Fase 0A (comienza hoy)  
**Deadline:** IIS 10/13 en staging Día 3, producción Día 8

#### Tarea 1: Provisión Infraestructura (Día 1-2)
- [ ] Provisionar nueva VM:
  - OS: **Windows Server 2016 (IIS 10)** o **Windows Server 2022 (IIS 13)**
  - vCPU: Mínimo 4 cores (match IIS 2008 actual)
  - RAM: Mínimo 8GB (match IIS 2008 actual)
  - Storage: Mínimo 100GB (SO + aplicación)
- [ ] Instalar roles:
  - [ ] IIS 10/13 (Web Server role)
  - [ ] .NET Framework 4.8
  - [ ] HTTP/2 support (automático en 2016+)
- [ ] Crear cuenta de servicio con permisos idénticos a IIS 2008 actual
- [ ] Conectar a red corporativa / firewall allowlist

#### Tarea 2: Instalación IIS (Día 2)
- [ ] Instalar IIS 10/13 via Server Manager
- [ ] Crear Application Pool con identidad de servicio
- [ ] Configurar binding HTTPS (copiar cert de IIS 2008)
- [ ] Habilitar TLS 1.2/1.3 en registry (automático pero validar)
- [ ] **Deshabilitar TLS 1.0/1.1** en registry:
  ```
  HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client
  DisabledByDefault = 1 (DWORD)
  Enabled = 0 (DWORD)
  
  [Similar para TLS 1.1]
  ```

#### Tarea 3: Copiar Aplicación (Día 2-3)
- [ ] Copiar binarios de aplicación (.NET 4.5 compilados) desde IIS 2008
- [ ] Copiar Web.config (con cambios Fase 0A)
- [ ] Copiar carpetas: App_Data, packages, vendors
- [ ] Verificar permisos de archivo (aplicación pool user puede leer/escribir)
- [ ] Crear logs/temp directories

#### Tarea 4: Load Testing (Día 3-5)
**En staging, simular carga real:**

- [ ] **Scenario 1: Desktop Users**
  - Tool: Apache JMeter / LoadRunner
  - Users: 100 usuarios concurrentes
  - Duration: 15 minutos
  - Expected: CPU < 70%, memory stable, cero crashes

- [ ] **Scenario 2: Mobile Users (CRÍTICO)**
  - Tool: JMeter con HTTP/2 + TLS 1.2
  - Users: 250 usuarios concurrentes (mixed 4G latency sim)
  - Duration: 15 minutos
  - TLS Profile: Simular iOS 14+ / Android 11+
  - Expected: **CERO timeouts**, CPU < 70%, HTTP/2 confirmado

- [ ] **Scenario 3: Mixed Workload**
  - Desktop: 150 usuarios
  - Mobile: 150 usuarios
  - APIs + page requests + file uploads
  - Duration: 30 minutos
  - Expected: All pass, stable 72h

- [ ] **Métrica Crítica:** Verificar HTTP/2 en use
  ```bash
  netsh http show sslcert | grep http2
  # Output should show HTTP/2 enabled
  ```

#### Tarea 5: TLS Validation (Día 3-5)
- [ ] Validar TLS 1.2/1.3 activo:
  ```bash
  openssl s_client -tls1_2 -connect server:443
  # Should connect successfully with TLS 1.2
  
  openssl s_client -tls1_3 -connect server:443
  # Should connect successfully with TLS 1.3 (si soporta)
  ```

- [ ] Deshabilitar TLS 1.0/1.1 y verificar rechazo:
  ```bash
  openssl s_client -tls1_0 -connect server:443
  # Should FAIL (alert: protocol_error)
  ```

- [ ] Validar con herramienta online:
  - Ir a: https://www.ssllabs.com/ssltest/
  - Apuntar al staging server
  - Expected: Grade A (TLS 1.2/1.3, sin TLS 1.0/1.1)

#### Tarea 6: Smoke Testing Funcional (Día 4-5)
- [ ] **Login flow:** Usuario → contraseña correcta → sesión establecida → OK
- [ ] **Reserva (móvil):** Crear reserva desde navegador móvil → OK
- [ ] **API calls:** Consumir PortalWebApi, WCF services → OK
- [ ] **Descarga de reportes:** Archivo generado exitosamente → OK
- [ ] **File upload:** Adjuntar documento → OK

#### Tarea 7: Performance Baseline (Día 5-6)
- [ ] Medir latencia P95 en staging vs IIS 2008:
  - Esperado: Reducción 6x (2500ms → 400ms)
  - Tool: APM / Application Insights
- [ ] Medir CPU bajo carga:
  - IIS 2008: ~85% en picos
  - IIS 10+: ~ 50-60% esperado
- [ ] Medir throughput (requests/seg):
  - IIS 2008: ~50 req/s
  - IIS 10+: ~500 req/s esperado (10x)

---

## 🚀 DEPLOY A PRODUCCIÓN (Día 7-8)

**Estrategia:** Blue-Green Deployment (sin downtime)

### Pre-Deployment (Día 6)
- [ ] Crear snapshot/backup de IIS 2008 actual
- [ ] Configurar Load Balancer para Blue-Green:
  ```
  Load Balancer Config:
  - Blue: IIS 2008 (actual) - 100%
  - Green: IIS 10+ (nuevo) - 0%
  ```
- [ ] Preparar runbook de rollback (instrucciones paso-a-paso)
- [ ] Notificar a usuarios: "Mantenimiento programado Día X, 2 horas ventana"

### Deployment (Día 7)
**Paso 1: Canary (10% tráfico)**
- [ ] Load Balancer: 90% Blue → 10% Green
- [ ] Monitoreo: Alertas en tiempo real (errors, latency, CPU)
- [ ] Duration: 30 minutos
- [ ] Criterio de éxito: Error rate Green ≈ Error rate Blue

**Paso 2: Gradual Rollout (50% tráfico)**
- [ ] Load Balancer: 50% Blue → 50% Green
- [ ] Monitoreo: Usuarios reportan experiencia normal?
- [ ] Duration: 60 minutos
- [ ] Criterio: No issues críticos

**Paso 3: Full Cutover (100% tráfico)**
- [ ] Load Balancer: 0% Blue → 100% Green
- [ ] Monitoreo: 24/7 observación
- [ ] Duration: Mínimo 4 horas antes de desconectar Blue
- [ ] Criterio: Operación normal confirmada

### Post-Deployment (Día 8-10)
- [ ] **Monitoreo intensivo 72 horas:**
  - [ ] CPU baseline: < 70%
  - [ ] Memory: Estable (sin memory leak)
  - [ ] Error rate: < 0.1%
  - [ ] **Mobile timeouts: CERO** ✓
  - [ ] Response time P95: < 500ms ✓
  
- [ ] **Validación móvil (real devices):**
  - [ ] iPhone XS + iOS 14: OK
  - [ ] iPhone 12 + iOS 15: OK
  - [ ] Samsung Galaxy + Android 12: OK
  - [ ] Pixel 6 + Android 13: OK
  
- [ ] **Sign-off:** Infrastructure Lead + Coordinador QA

### Rollback (Si es necesario)
- [ ] Ejecutar: `LoadBalancer.SetTraffic(Blue=100%, Green=0%)`
- [ ] Time: < 5 minutos
- [ ] Notificar users: "Servicio restaurado"
- [ ] Investigar issue en Green (staging)
- [ ] Reintentar deploy Día 9

---

## 📊 MÉTRICAS DE ÉXITO (Fase 0A + 0B)

### Fase 0A (Hoy - Mitigaciones Código)
- [ ] Deploy completado sin errores
- [ ] Timeout errors reducen 40-50% (24h observación)
- [ ] Cero regresiones funcionales
- [ ] Load test staging: 100 users × 5 min PASS

### Fase 0B (10 días - Migración IIS)
- [ ] IIS 10/13 estable en producción 72h
- [ ] **Móviles: CERO timeouts** (métrica crítica)
- [ ] CPU < 70% bajo carga máxima
- [ ] TLS 1.2/1.3 confirmado (sin 1.0/1.1)
- [ ] HTTP/2 activo (verificado con curl/netsh)
- [ ] Usuarios móvil: Satisfacción > 95%
- [ ] PCI-DSS: Conforme (TLS 1.2+)

---

## 🔗 REFERENCIAS

| Documento | Sección | Contenido |
|-----------|---------|----------|
| [ADR-0001](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md) | 5.1-5.6 | Código de mitigaciones (Fase 0A) |
| [ADR-0001](adrs/ADR_0001_IIS2008_TLS_Mobile_Crisis.md) | 6 | Plan completo migración IIS (Fase 0B) |
| [informe_tecnico.md](informe_tecnico.md) | H-00 | Descripción completa hallazgo IIS 2008 |
| [informe_tecnico.md](informe_tecnico.md) | Sección 8 | Recomendaciones P0 detalladas |

---

## 📞 ESCALACIÓN

**Problema crítico descubierto post-deploy:**
1. Contactar Infrastructure Lead
2. Activar Runbook de Rollback (< 5 min)
3. Notificar PMO
4. Root cause analysis en staging

**Preguntas técnicas:**
- Dev: Dev Lead
- Infra: Infrastructure Lead
- Escalación: PMO + Gerencia IT

---

**STATUS:** 🔴 READY TO START  
**Última actualización:** 2026-04-28 19:00  
**Next milestone:** Dev team completa Fase 0A hoy | Infra comienza Fase 0B
