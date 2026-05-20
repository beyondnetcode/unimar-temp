# Guía de Visualización - Auditoría Técnica SIS_INTRANET

Los diagramas en esta auditoría utilizan **PlantUML**, una herramienta gratuita y open-source. 

## 📋 Índice de Documentos

1. **[Resumen Ejecutivo](resumen_ejecutivo_indice.md)** - Para C-Level y decisores
2. **[Informe Técnico Principal](informe_tecnico.md)** - Análisis detallado con evidencia
3. **[Anexo de Diagramas](anexo_diagramas.md)** - Modelos arquitectónicos y E/R
4. **[Resumen Visual Ejecutivo](resumen_visual_ejecutivo.md)** - Diagrama de riesgos y roadmap

---

## 🎨 Cómo Visualizar los Diagramas PlantUML

### Opción 1: Online (RECOMENDADO - Sin instalación)

**Más rápido y más simple:**

1. Abre: https://www.plantuml.com/plantuml/uml/
2. Copia cualquier bloque de código desde `@startuml` hasta `@enduml`
3. Pégalo en el editor online
4. Haz clic en **"Submit"** o presiona **Ctrl+Enter**

**Ejemplo:**
```
@startuml
class Persona {
  +nombre : string
  +edad : int
}
@enduml
```

---

### Opción 2: VS Code (Extensión Gratuita)

**Para editar y previsualizar en tu editor:**

1. **Instala la extensión PlantUML:**
   - Abre VS Code
   - Extensiones (Ctrl+Shift+X)
   - Busca "PlantUML" por "jebbs"
   - Haz clic en "Instalar"

2. **Previsualiza diagramas:**
   - Abre cualquier `.md` con código PlantUML
   - Presiona **Alt+D** o **Ctrl+Shift+P** → "PlantUML: Preview"
   - Se abrirá un panel lateral con la visualización

---

### Opción 3: GitHub (Automático)

**Los diagramas se renderizan automáticamente:**

- Sube este repositorio a GitHub
- GitHub renderiza bloques PlantUML automáticamente en archivos `.md`
- Los colaboradores ven los diagramas sin herramientas adicionales

---

### Opción 4: Línea de Comandos (Docker)

**Para generar PNG/SVG:**

```bash
# Instala Docker primero

# Genera PNG de un archivo
docker run --rm -v C:\ruta\a\auditoria:/data plantuml /data/anexo_diagramas.md -o output

# O directamente de código PlantUML
echo "@startuml
class Test { }
@enduml" | docker run --rm -i plantuml -tpng > diagrama.png
```

---

## 📊 Diagrama de la Auditoría

```
┌─────────────────────────────────────────────────────────────┐
│           AUDITORÍA TÉCNICA SIS_INTRANET                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Resumen Ejecutivo (índice + nav)                       │
│     └─> Orientado a: C-Level, Gestores, Auditores         │
│                                                             │
│  2. Informe Técnico (análisis + evidencia)                 │
│     ├─> Arquitectura & Componentes                         │
│     ├─> Vulnerabilidades (8 issues, críticas)             │
│     ├─> Go/No-Go criteria (6 bloqueantes)                 │
│     └─> Roadmap de remediación (Fases 0-4)               │
│                                                             │
│  3. Diagrama de Diagramas (13 visualizaciones)            │
│     ├─> UML: ClassDiagram, SequenceDiagram               │
│     ├─> Arquitectura: C4 Context, C4 Container           │
│     ├─> Datos: 6 Modelos E/R (por dominio)               │
│     ├─> Procesos: Flowcharts & Gantt                     │
│     └─> Riesgos: Matriz & Timeline                       │
│                                                             │
│  4. Resumen Visual (4 gráficos ejecutivos)                │
│     ├─> Panorama arquitectónico                           │
│     ├─> Mapa de riesgo (Criticidad)                       │
│     ├─> Roadmap (Fases)                                   │
│     └─> Salida a producción (Checklist)                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ✅ Validación de Diagramas

Todos los diagramas fueron:
- ✅ Convertidos a sintaxis PlantUML válida
- ✅ Probados en el editor online PlantUML
- ✅ Documentados con notas de contexto
- ✅ Relacionados con el informe técnico

---

## 🔍 Datos Clave por Sección

| Sección | Contenido Clave | Audiencia |
|---------|-----------------|-----------|
| **Resumen Ejecutivo** | Go/No-Go criteria, Risk Summary, Timeline | C-Level, Dirección |
| **Informe Técnico** | Vulnerability Details, Architecture, Evidence | Technical Leads, Developers |
| **Anexo de Diagramas** | UML, C4, E/R Models, Data Dictionary | Architects, DBAs |
| **Visual Ejecutivo** | Risk Matrix, Roadmap, Deployment Checklist | PMO, Executives |

---

## 💡 Recomendaciones de Lectura

1. **Para Ejecutivos:** Comienza con Resumen Ejecutivo
2. **Para Técnicos:** Lee Informe + Anexo Diagramas
3. **Para PMO/Proyecto:** Consulta Resumen Visual + Roadmap
4. **Para QA/Testing:** Usa Diagrams para entender flujos críticos

---

## 📝 Notas

- Todos los diagramas son **100% PlantUML open-source**
- No requieren cuenta ni suscripción
- Compatible con sistemas Windows, Mac y Linux
- Los diagramas en `.md` pueden renderizarse en cualquier plataforma que soporte PlantUML

---

**Última actualización:** Abril 28, 2026  
**Formato:** PlantUML (Gratuito & Open-source)  
**Estado:** Auditoría Completa
