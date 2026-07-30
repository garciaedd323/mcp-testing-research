# MCP para Testing Automatizado

> Investigación en curso sobre el Model Context Protocol (MCP) aplicado a QA y pruebas automatizadas — web y móvil.

## 📌 ¿Qué es esto?

Este repositorio recopila investigación, notas, enlaces y prompts sobre cómo usar **MCP + Claude** para automatizar tareas de testing: exploración web, generación de suites, mantenimiento de tests, análisis de reportes, pruebas móviles, y más.

---

## 🚀 Instalación rápida (empieza aquí)

> Configuración probada en Windows con IntelliJ IDEA + Serenity BDD.

### Requisitos previos

- IntelliJ IDEA (cualquier edición)
- Node.js — verifica con `node --version` ([descargar](https://nodejs.org))
- Java 17 o superior
- Un proyecto Serenity BDD existente

### 1 — Instalar Claude Code

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

### 2 — Plugin en IntelliJ

`Settings → Plugins → Marketplace` → buscar **"claude mcp"** → instalar **"MCP Servers for AI Assist..."** (by meanmail.dev) → reiniciar IntelliJ.

> También verás `Claude Code [Beta]` en `Settings → Tools` — es la integración oficial de Anthropic, se configura automáticamente.

### 3 — Agregar servidor filesystem

```bash
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\a\tu\proyecto"
```

> **Windows:** Si la ruta tiene espacios, ponla siempre entre comillas dobles.

### 4 — Agregar servidor Playwright

```bash
claude mcp add playwright npx -- -y @playwright/mcp@latest
```

### 5 — Verificar

```bash
claude mcp list
```

Resultado esperado:

```
filesystem: npx -y @modelcontextprotocol/server-filesystem C:\...\ProyectoWebQA - √ Connected
playwright: npx -y @playwright/mcp@latest - √ Connected
```

### Referencia rápida de comandos

| Acción | Comando |
|--------|---------|
| Agregar filesystem | `claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta"` |
| Agregar Playwright | `claude mcp add playwright npx -- -y @playwright/mcp@latest` |
| Ver servidores | `claude mcp list` |
| Eliminar servidor | `claude mcp remove [nombre]` |
| Abrir Claude Code | `claude` |
| Ver versión | `claude --version` |

📖 **¿Quieres la guía completa y detallada?** — [`docs/web/guia-mcp-qa-serenity.md`](./docs/web/guia-mcp-qa-serenity.md) tiene el paso a paso extendido (agregar/eliminar servidores adicionales, prompts de diagnóstico inicial del proyecto, prompts de exploración y generación automática de tests) más allá de lo resumido arriba.

---

## 🌐 Testing Web

### Mapa de capacidades

```mermaid
mindmap
  root((Claude + MCP<br/>Agente QA autónomo))
    Exploración y creación
      Exploración web automática
      Generación de suites
      Auto-mantenimiento
    Análisis e integración
      Análisis de reportes
      Integración CI/CD
    Ejecución y cobertura
      Generación de datos
      Auditoría accesibilidad
      Testing cross-browser
```

### Documentación

| Documento | Contenido |
|---|---|
| [`docs/web/que-es-mcp-analogia.md`](./docs/web/que-es-mcp-analogia.md) | 👉 Empieza aquí si eres nuevo en MCP |
| [`docs/web/caso-estudio-suite-e2e-mcp.md`](./docs/web/caso-estudio-suite-e2e-mcp.md) | 🟢 **Caso real:** 42 pruebas E2E generadas con Claude+MCP en un proyecto de gestión comercial (Serenity/Screenplay) — metodología de exploración incremental, problemas técnicos resueltos, y la limitación clave detectada (Claude puede "creer" que un clic funcionó a nivel de código sin que el resultado sea visible en pantalla) |
| [`docs/web/capacidades-mcp-qa.md`](./docs/web/capacidades-mcp-qa.md) | Resumen de las 8 capacidades de Claude + MCP en QA |
| [`docs/web/auditoria-accesibilidad.md`](./docs/web/auditoria-accesibilidad.md) | 🟡 Conceptual — Auditoría a11y automática con Playwright |
| [`docs/web/auto-mantenimiento-flujo.md`](./docs/web/auto-mantenimiento-flujo.md) | 🟡 Conceptual — Cómo se detectan y reparan tests rotos |
| [`docs/web/testing-crossbrowser.md`](./docs/web/testing-crossbrowser.md) | 🟡 Conceptual — Ejecución en Chrome, Firefox y Safari en paralelo |
| [`docs/web/generacion-suites-estrategia.md`](./docs/web/generacion-suites-estrategia.md) | 🟡 Conceptual — Estrategia de 3 niveles: smoke, regresión core, completa |
| [`docs/web/analisis-reportes-serenity.md`](./docs/web/analisis-reportes-serenity.md) | 🟡 Conceptual — Análisis de patrones de fallo y cobertura |
| [`docs/web/generacion-datos-prueba.md`](./docs/web/generacion-datos-prueba.md) | 🟡 Conceptual — Generación de datos de prueba realistas |
| [`docs/web/testing-playwright-mcp-vs-screenplay.md`](./docs/web/testing-playwright-mcp-vs-screenplay.md) | Playwright MCP (agente IA + `.md`) vs Screenplay Pattern + Cucumber + JUnit: estructura de un proyecto MCP, chat directo vs archivos versionados, comparación con automatización tradicional, y cómo combinar ambos (MCP como exploración, Screenplay como código final en CI/CD) |
| [`docs/web/troubleshooting-mcp-filesystem-windows.md`](./docs/web/troubleshooting-mcp-filesystem-windows.md) | 🟢 Error real resuelto: rutas con espacios en Windows |
| [`docs/web/mcp-filesystem-entre-proyectos.md`](./docs/web/mcp-filesystem-entre-proyectos.md) | 🟢 Cómo cambiar la ruta de MCP al pasar de un proyecto a otro |

> 🟢 = validado con experiencia real · 🟡 = conceptual, pendiente de ejecutarse en un proyecto real · sin marca = análisis/comparativa

**Prompts:** [`recursos/prompts/web/prompts-mcp.md`](./recursos/prompts/web/prompts-mcp.md) 🎯
**Enlaces curados:** [`recursos/enlaces/mcp-testing.md`](./recursos/enlaces/mcp-testing.md) *(plantilla en construcción, se irá llenando)*

---

## 📱 Testing Móvil

> En investigación activa — Android + Windows por ahora.

### Documentación

| Documento | Contenido |
|---|---|
| [`docs/movil/instalacion-entorno-android.md`](./docs/movil/instalacion-entorno-android.md) | 👉 Empieza aquí: instalación completa del entorno (Node, JDK, Android Studio, Appium, cliente MCP) |
| [`docs/movil/decision-claude-desktop-vs-code.md`](./docs/movil/decision-claude-desktop-vs-code.md) | Decisión: por qué se usa Claude Code y no Claude Desktop |
| [`docs/movil/paso3-conectar-appium-claude-code.md`](./docs/movil/paso3-conectar-appium-claude-code.md) | ✅ Paso 3 vigente: conectar appium-mcp a Claude Code |
| ↳ [`docs/movil/appium-inspector-vs-capabilities-json.md`](./docs/movil/appium-inspector-vs-capabilities-json.md) | Aclaración: por qué no se reutiliza el capabilities de Inspector directamente |
| ↳ [`docs/movil/troubleshooting-appium-mcp-failed.md`](./docs/movil/troubleshooting-appium-mcp-failed.md) | Error real resuelto: appium-mcp aparece como "failed" |
| ↳ [`docs/movil/paso3-conectar-appium-claude-desktop.md`](./docs/movil/paso3-conectar-appium-claude-desktop.md) | ⚠️ Versión alternativa no usada (Claude Desktop) |

**Prompts:** [`recursos/prompts/movil/`](./recursos/prompts/movil) *(aún vacío, se irá llenando)*

---

## 🛠️ Solución de problemas comunes

### `filesystem: × Failed to connect`
Rutas con espacios en Windows. Solución:
```bash
claude mcp remove filesystem
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\sin espacios"
```

### `MCP server filesystem already exists`
El servidor ya está registrado con otra ruta. Elimínalo primero:
```bash
claude mcp remove filesystem
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\nueva\ruta"
```

### Claude dice "no tengo herramienta de navegador"
Falta el servidor de Playwright. Agrégalo y abre una sesión nueva:
```bash
claude mcp add playwright npx -- -y @playwright/mcp@latest
```

### `Needs authentication` en Gmail, Drive o Calendar
Son servidores opcionales de Google, no necesarios para QA. Para autenticarlos, ejecuta `claude` en terminal e intenta usar uno de esos servicios.

---

## 📂 Estructura del repositorio

```
mcp-testing-research/
├── docs/
│   ├── web/       ← documentación de MCP para testing web
│   └── movil/     ← documentación de MCP para testing móvil
├── notas/         ← notas de investigación en bruto, ideas sueltas
├── recursos/
│   ├── enlaces/       ← enlaces externos curados, por categoría
│   ├── prompts/
│   │   ├── web/       ← prompts reutilizables para testing web
│   │   └── movil/     ← prompts reutilizables para testing móvil
│   └── capturas/      ← capturas de pantalla relevantes (diagramas, UI)
└── ejemplos/      ← código o configuraciones de ejemplo
```

## 🚧 Estado

Proyecto en investigación activa. Ver [`notas/`](./notas) para el progreso más reciente.

> 📝 Para agregar una nota nueva, copia [`notas/_plantilla.md`](./notas/_plantilla.md) como punto de partida (contexto, hallazgos, preguntas abiertas, fuentes).

## 📄 Licencia

Este contenido se comparte bajo [MIT License](./LICENSE) (o cambia según prefieras: CC-BY para contenido no-código).
