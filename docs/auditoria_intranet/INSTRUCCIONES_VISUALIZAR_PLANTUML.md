# Instrucciones para Visualizar Diagramas PlantUML

## Cambio de Mermaid a PlantUML

Por razones de costo/restricciones en Mermaid, los diagramas han sido convertidos a **PlantUML**, que es completamente **gratuito y de código abierto**.

## Archivos PlantUML en la auditoría

- [RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md](RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md) - Diagramas de resumen ejecutivo
- [ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md](ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md) - Diagramas UML, C4, E/R

## Opción 1: VS Code (Recomendada - Más Fácil)

### 1.1 Instalar extensión PlantUML

1. Abre VS Code
2. Ve a Extensions (Ctrl+Shift+X)
3. Busca "PlantUML" (autor: jebbs)
4. Haz clic en "Install"

### 1.2 Ver diagrama en VS Code

1. Abre el archivo .md con diagrama PlantUML
2. Presiona **Alt + D** (o busca "PlantUML" en la paleta de comandos)
3. Se abrirá una ventana con vista previa de todos los diagramas

### 1.3 Exportar diagrama a imagen

1. Con el diagrama abierto, presiona clic derecho en la vista previa
2. Selecciona "Export" → "PNG" o "SVG"
3. Elige la carpeta de destino

---

## Opción 2: Editor Online (Sin Instalación)

### 2.1 PlantUML Online Viewer

1. Ve a: http://www.plantuml.com/plantuml/uml/
2. Abre el archivo .md en tu editor de texto
3. Copia todo el contenido del bloque `plantuml`
4. Pégalo en el editor online
5. El diagrama se renderiza automáticamente

### 2.2 Generar enlace compartible

- El editor online genera automáticamente URL que puedes compartir
- No requiere login

---

## Opción 3: Línea de Comandos (Para Automatización)

### 3.1 Instalar PlantUML

```bash
# En Windows (con Chocolatey)
choco install plantuml

# En macOS (con Homebrew)
brew install plantuml

# O con npm (requiere Node.js)
npm install -g plantuml
```

### 3.2 Generar imágenes

```bash
# PNG por defecto
plantuml ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md

# SVG (vector, mejor calidad)
plantuml -o output -tsvg ANEXO_DIAGRAMAS_ARQUITECTURA_PLANTUML.md

# PNG con salida en carpeta específica
plantuml -o ./imagenes RESUMEN_VISUAL_EJECUTIVO_PLANTUML.md
```

---

## Comparativa: Mermaid vs PlantUML

| Aspecto | Mermaid | PlantUML |
|--------|---------|----------|
| **Costo** | Planes pagos después de límite | 100% Gratuito |
| **Instalación** | Online solo | Online + Local + CLI |
| **Diagramas soportados** | flowchart, sequence, class, ER, C4 | flowchart, sequence, class, ER, C4 + más |
| **Comunidad** | Grande y creciente | Muy grande y establecida |
| **Actualización** | Reciente | Muy estable |

---

## Diagramas Convertidos

### Resumen Visual Ejecutivo
- ✅ Panorama de arquitectura
- ✅ Mapa ejecutivo de riesgo
- ✅ Ruta de salida a producción ASAP
- ✅ Plan progresivo de optimización

### Anexo Diagramas de Arquitectura
- ✅ UML Class Diagram (capas y dependencias)
- ✅ UML Sequence Diagram (flujo autenticación)
- ✅ UML Activity Diagram (flujo HTTP)
- ✅ C4 Context (contexto del sistema)
- ✅ C4 Container (contenedores principales)
- ✅ Índice visual del anexo
- ✅ ER Domain Transmisiones
- ✅ ER Domain Operativo/Comercial
- ✅ ER Domain Seguridad/Acceso
- ✅ ER Domain Reclamos
- ✅ ER Domain Visitas/Reservas
- ✅ ER Domain Depósito Vacíos

---

## Solución de Problemas

### PlantUML no renderiza en VS Code

1. Asegúrate de tener GraphViz instalado:
   - Windows: `choco install graphviz`
   - macOS: `brew install graphviz`
   - Linux: `apt install graphviz`

2. Reinicia VS Code

### Diagrama se ve cortado o mal formateado

- Algunos diagramas grandes necesitan más ancho: ajusta la ventana
- Prueba con el editor online para comparar

### Errores de sintaxis PlantUML

- Los bloque deben empezar con ````plantuml` y terminar con ````
- Verifica que no haya caracteres especiales sin escapar

---

## Recursos Adicionales

- **Documentación oficial**: https://plantuml.com/
- **Sintaxis C4**: https://plantuml.com/C4-Components
- **Galería de ejemplos**: https://plantuml.com/
- **GitHub**: https://github.com/plantuml/plantuml

---

**Ventaja principal:** PlantUML es una solución de código abierto y sin costo que puedes usar indefinidamente sin preocupaciones por límites de uso o pagos mensuales.
