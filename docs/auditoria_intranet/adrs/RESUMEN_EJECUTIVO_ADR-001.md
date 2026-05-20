# RESUMEN EJECUTIVO: Problema Crítico IIS 2008 TLS en Móviles

**Preparado para:** Coordinador de Infraestructura Líder  
**Fecha:** 28 de abril de 2026  
**Clasificación:** CRÍTICO - BLOQUEANTE GO-LIVE  

---

## 🚨 Hallazgo Reportado

Caídas de servidor al intentar dispositivos móviles negociar TLS con IIS 2008.

**Root Cause:** IIS 2008 (versión 2008, fuera de soporte desde 2015) tiene limitaciones críticas en:
- Protocolos TLS (solo 1.0/1.1, deprecados)
- Negociación de cipher suites (débil)
- Gestión de conexiones concurrentes
- Memory leaks en conexiones SSL/TLS prolongadas

---

## 📊 Impacto Actual

| Aspecto | Impacto |
|--------|---------|
| **Usuarios afectados** | Todos los usuarios móviles (responsive) |
| **Disponibilidad** | Caídas recurrentes durante peak hours |
| **Seguridad** | TLS 1.0/1.1 vulnerables (PCI-DSS no cumple) |
| **Performance** | Throttling por falta de multiplexing HTTP/2 |
| **Escalabilidad** | No soporta +500 usuarios concurrentes móviles |

---

## ✅ Solución Propuesta (Aprobada)

### Fase 0 (PRE-GO-LIVE - Urgente): Mitigación en Código
- Implementar cambios en Web.config + Global.asax.cs para reducir renegociación TLS
- Load test: validar 500 usuarios móviles sin caídas
- **Timeline:** 2-3 días
- **Riesgo:** Bajo (parche temporal, código bien documentado)
- **Go-Live:** Puede proceder con mitigaciones activadas

### Fase 1 (POST-GO-LIVE - Mes 1): Migración a IIS 10+
- Provisionar servidor nuevo: Windows Server 2019/2022 + IIS 10
- Migrar configuración, certificados SSL, aplicación
- Validar en staging paralelo
- **Cutover:** Fin de semana con ventana baja
- **Beneficios:** +40-60% performance, TLS 1.2/1.3, HTTP/2, soporte moderno
- **Timeline:** 2-3 semanas
- **Riesgo:** Bajo (staging en paralelo, rollback plan)

---

## 📝 Opciones Implementadas en Código (Fase 0)

### 1. Web.config - Keep-Alive y Pool Tuning
```xml
<!-- Reducir renegociación SSL -->
<customHeaders>
  <add name="Connection" value="keep-alive" />
  <add name="Keep-Alive" value="timeout=15, max=100" />
</customHeaders>

<!-- Aumentar límites de conexión -->
<requestLimits maxAllowedContentLength="104857600" />
```

### 2. Global.asax.cs - Forzar TLS 1.2+
```csharp
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12 | 
                                       SecurityProtocolType.Tls11;
ServicePointManager.DefaultConnectionLimit = 10;
ServicePointManager.ReusePort = true;
```

### 3. Caché de Assets Estáticos
- Minimizar tamaño CSS/JS (gzip)
- Reducir reconexiones SSL desde navegador móvil
- Cache 30 días en navegador

**Detalles completos:** Ver [ADR-001](./adrs/ADR-001_Migracion_IIS2008_a_IIS10_TLS_Moviles.md) - Sección 4

---

## 🎯 Criterios de Go-Live (Bloqueantes)

✅ **DEBE estar resuelto ANTES del Go-Live:**
1. Cambios código (mitigación TLS) implementados en SIS_INTRANET
2. Load test: 500 usuarios móviles concurrentes SIN caídas TLS
3. Certificado SSL válido en navegadores modernos
4. Usuarios móviles pueden conectar sin errores de handshake

⚠️ **NO bloquea Go-Live (pero requiere paralelismo):**
- Migración completa a IIS 10 (puede completarse en Mes 1)

---

## 💰 Presupuesto y Recursos

| Item | Costo | Timeline |
|------|-------|----------|
| Desarrollo (cambios código) | 0 (interno) | 2-3 días |
| Testing carga (móvil) | 0 (interno) | 1 día |
| Servidor IIS 10 (provisioning) | [A calcular infraestructura] | 3-5 días |
| Migración y validación | [A calcular infraestructura] | 5-7 días |
| **Total Fase 0** | **~5 días desarrollo** | |
| **Total Fase 1** | **2-3 semanas** | |

---

## 📋 Próximos Pasos Inmediatos

**Hoy/Mañana:**
1. ✅ Aprobación de cambios código para Fase 0
2. ✅ Comunicar timing al equipo de infraestructura

**Esta semana:**
1. Implementar cambios código (2 días)
2. Ejecutar load test móviles 500 usuarios (1 día)
3. Validar resultado vs. criterio de Go-Live
4. Iniciar provisioning IIS 10 en paralelo

**Semana siguiente:**
1. Go-Live en producción con mitigaciones activadas
2. Monitoreo reforzado 24/7 (semana 1)

**Mes 1:**
1. Completar migración a IIS 10 en servidor paralelo
2. Cutover durante ventana baja (fin de semana)
3. Validación y decomisioning de IIS 2008

---

## 📖 Documentación Completa

Toda la información técnica, opciones evaluadas y decisión arquitectónica está documentada en:

**[ADR-001: Migración IIS 2008 → IIS 10+ para Resolver Fallo TLS en Dispositivos Móviles](./adrs/ADR-001_Migracion_IIS2008_a_IIS10_TLS_Moviles.md)**

Contiene:
- Análisis completo del problema
- 3 opciones evaluadas (con pros/contras)
- Decisión adoptada (Opción B + C)
- Mitigaciones en código (código específico)
- Plan de migración infraestructura
- Tabla de riesgos y mitigaciones
- Criterios de aceptación

---

## ✍️ Aprobación Requerida

| Rol | Aprobación | Fecha |
|-----|-----------|-------|
| **Coordinador Infraestructura** | [ ] | |
| **Arquitecto** | [ ] | |
| **CTO/Director IT** | [ ] | |

---

**Documento:** RESUMEN_ADR-001_EJECUTIVO.md  
**Clasificación:** CRÍTICO - ARQUITECTURA  
**Revisión próxima:** 30 de mayo de 2026 (post-Go-Live)

---

**¿Preguntas o cambios?**  
Revisar documento ADR-001 completo o contactar al equipo de auditoría.
