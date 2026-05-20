# Índice de Decisiones Arquitectónicas (ADRs)

Los ADRs (Architecture Decision Records) documentan decisiones técnicas críticas con contexto, opciones evaluadas, decisión y consecuencias.

## ADRs en el Proyecto

### ADR-001: Migración de IIS 2008 a IIS 10+ para Resolver Fallo TLS en Dispositivos Móviles

- **Fecha:** 28 de abril de 2026
- **Estado:** CRÍTICO - Implementación Urgente (Fase 0 Go-Live)
- **Severidad:** P1 (Bloqueante)
- **Archivo:** [ADR-001_Migracion_IIS2008_a_IIS10_TLS_Moviles.md](./ADR-001_Migracion_IIS2008_a_IIS10_TLS_Moviles.md)

**Documentos relacionados:**
- [RESUMEN_EJECUTIVO_ADR-001.md](./RESUMEN_EJECUTIVO_ADR-001.md) - Resumen 1 página para directivos
- [CHECKLIST_IMPLEMENTACION_ADR-001.md](./CHECKLIST_IMPLEMENTACION_ADR-001.md) - Guía paso-a-paso para development

**Resumen:**
El coordinador de infraestructura reportó caídas de servidor cuando dispositivos móviles negocian TLS con IIS 2008. Esta versión legacy (2008, fuera de soporte desde 2015) tiene limitaciones críticas en protocolos TLS, gestión de conexiones y performance.

**Decisión:**
- **Inmediato (Fase 0):** Implementar mitigaciones en código (Web.config + Global.asax.cs) para reducir renegociación TLS en IIS 2008
- **Corto plazo (Mes 1):** Migrar a IIS 10+ (Windows Server 2019/2022) con TLS 1.2/1.3

**Impacto:**
- ✅ Elimina caídas en usuarios móviles
- ✅ +40-60% performance
- ✅ Soporte moderno y seguridad actual
- ⚠️ Requiere provisioning infraestructura

**Criterios de Go-Live:**
- Cambios código implementados y testeados (load test 500 móviles)
- Usuarios móviles pueden conectar sin caídas TLS
- IIS 10+ en provisionamiento (cutover en Mes 1)

---

## Cómo Usar Este Índice

1. **Para entender decisiones críticas:** Lee los ADRs en orden de severidad
2. **Para referencias técnicas:** Cada ADR contiene secciones de mitigación, opciones evaluadas y evaluación
3. **Para planificación:** Los ADRs indican bloqueantes para Go-Live vs. items diferibles
4. **Para actualización:** Nuevos ADRs deben ser creados para decisiones arquitectónicas mayores durante ejecución

---

## Guía de Creación de Nuevos ADRs

Si necesitas documentar una decisión arquitectónica durante la ejecución:

### Estructura Recomendada

```markdown
# ADR-XXX: Título de la Decisión

**Fecha:** [fecha]
**Estado:** [PROPUESTA | APROBADA | CRITICADA | DEFERIDA | REEMPLAZADA | SUPERADA]
**Autor:** [nombre]
**Revisor:** [nombre]

## 1. Problema (Context)
- Síntoma observado
- Causa raíz
- Impacto
- Clasificación de riesgo

## 2. Opciones Evaluadas
### Opción A: [nombre]
- Descripción
- Evaluación (pros/contras)
- Decisión: ✅/❌

### Opción B: [nombre]
- [...]

### Opción C: [RECOMENDADA]
- [...]

## 3. Decisión Adoptada
- Opción elegida y justificación
- Criterios de aceptación

## 4. Mitigaciones
- [acciones específicas]

## 5. Impacto y Consecuencias
- Positivas
- Riesgos y mitigaciones

## 6. Recomendaciones Finales
- Orden de ejecución
- Criterios de Go-Live

## 7. Referencias
- [enlaces útiles]

## 8. Aprobación
- [tabla de firmas]
```

---

## Historial de ADRs

| ID | Título | Fecha | Estado | Severidad |
|----|--------|-------|--------|-----------|
| ADR-001 | Migración IIS 2008 → IIS 10+ TLS Móviles | 2026-04-28 | CRÍTICO | P1 |

---

**Mantenedor:** Arquitectura de SIS_INTRANET  
**Próxima Revisión:** 30 de mayo de 2026 (post-Go-Live)
