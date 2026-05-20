# Architecture Decision Records (ADRs) - SIS_INTRANET Auditoría

**Carpeta de Decisiones Arquitectónicas Críticas**

Estos documentos registran decisiones técnicas importantes tomadas durante la auditoría y en la ruta hacia Go-Live.

---

## 📋 Índice de ADRs

### ADR-0001: Migración Urgente IIS 2008 → IIS 10/13 por Crisis de TLS en Móviles

**Status:** CRITICAL - PHASE 0 BLOCKER  
**Date:** 2026-04-28  
**Severity:** P0 (Production Outage)  
**Owner:** Infrastructure Team + Dev Team

**Resumen:**
- **Problema:** IIS 2008 (EOL 2015) falla en negociación TLS con dispositivos móviles → caídas de servidor
- **Impacto:** 40%+ usuarios (móvil) no pueden acceder; aplicación inoperable
- **Decisión:** Migrar a IIS 10 (Win 2016) o IIS 13 (Win 2022) en 7-10 días
- **Estrategia Híbrida:**
  - Fase 0A (Hoy): Mitigaciones desde código (.NET) → reducción 40-50% incidentes
  - Fase 0B (10 días): Migración infraestructura → solución permanente 100%
- **Bloqueante:** SÍ - NO-GO sin completar
- **Archivos:** [ADR_0001_IIS2008_TLS_Mobile_Crisis.md](ADR_0001_IIS2008_TLS_Mobile_Crisis.md)

**Acciones Inmediatas:**
1. Implementar mitigaciones código hoy (4-6 horas)
2. Iniciar provisión de IIS 10/13 en staging (paralelo)
3. Load testing en staging (Día 3-5)
4. Rollout a producción (Día 7-8)
5. Validación 72h (Día 8-10)

**Criterios de Éxito:**
- ✓ TLS 1.2/1.3 activo (no 1.0/1.1)
- ✓ HTTP/2 operativo
- ✓ Cero timeouts móvil en 72h
- ✓ CPU < 70% bajo carga
- ✓ Usuarios móvil reportan "conexión exitosa"

**Referencias:**
- Sección 4 (H-00) en [informe_tecnico.md](../informe_tecnico.md)
- Sección 7.1 en [informe_tecnico.md](../informe_tecnico.md) - Bloqueantes Go/No-Go
- [README_VISUALIZACION.md](../README_VISUALIZACION.md) - Contexto de auditoría

---

## 📝 Formato Estándar de ADR

Todos los ADRs en esta carpeta siguen el formato:

```
# ADR-XXXX: [Título Decisión]

Status: PROPOSED|ACCEPTED|DEPRECATED|SUPERSEDED
Date: YYYY-MM-DD
Author: [Rol/Nombre]
Severity: P0|P1|P2|P3
Target Resolution: [Timeline]

## 1. Resumen Ejecutivo
## 2. Contexto Técnico
## 3. Opciones Consideradas
## 4. Decisión Recomendada
## 5. Soluciones / Implementación
## 6. Consecuencias y Beneficios
## 7. Aprobación
## 8. Changelog
```

---

## 🗺️ Roadmap de ADRs Planeados

| ADR | Título | Estado | Target | Owner |
|-----|--------|--------|--------|-------|
| 0001 | IIS 2008 → IIS 10/13 Migración | CRITICAL | 2026-05-08 | Infra |
| 0002 | Estrategia de Secretos y Vault | Pending | 2026-04-29 | SecOps |
| 0003 | Standardización de Authn/Authz | Pending | 2026-05-15 | DevLead |
| 0004 | Modernización de Stack (.NET) | Pending | 2026-06-30 | Arch |
| 0005 | Modelo de Datos Canonical | Pending | 2026-07-15 | DBA |

---

## 🔗 Navegación Rápida

**Contexto General:**
- [Informe Técnico Principal](../informe_tecnico.md)
- [Resumen Ejecutivo](../resumen_ejecutivo_indice.md)
- [Anexo de Diagramas](../anexo_diagramas.md)

**Infraestructura & Deployment:**
- [ADR-0001: IIS 2008 Crisis](ADR_0001_IIS2008_TLS_Mobile_Crisis.md) - Bloqueante P0

**Seguridad (Planeado):**
- ADR-0002: Estrategia de Secretos
- ADR-0003: Autenticación/Autorización Estándar

**Arquitectura (Planeado):**
- ADR-0004: Modernización de Framework
- ADR-0005: Modelo de Datos

---

## ✅ Proceso de Aprobación

Cada ADR requiere:

1. **Documentación:** Completar template
2. **Revisión Técnica:** Dev Lead + Arquitecto
3. **Revisión Operacional:** Ops Lead (si aplica)
4. **Aprobación Ejecutiva:** PMO/Gerencia (si P0/P1)
5. **Comunicación:** Notificar a stakeholders
6. **Seguimiento:** Status updates cada 48h (P0) o semanal (P1+)

---

## 📞 Contacto y Escalación

**Para ADRs Críticos (P0):**
- Coordinador Infraestructura: [contacto]
- Dev Lead: [contacto]
- Escalación: Gerencia IT

**Para ADRs Operacionales (P1):**
- Technical Lead: [contacto]

**Para ADRs Estratégicos (P2+):**
- Solution Architect: [contacto]

---

**Última actualización:** 2026-04-28  
**Próxima revisión:** 2026-04-29 (ADR-0001 progress)
