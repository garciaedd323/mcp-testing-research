# Guía de configuración MCP para QA Engineers
### Serenity BDD + Playwright + IntelliJ IDEA

> Guía práctica basada en experiencia real de configuración.  
> Al final de esta guía tendrás Claude Code integrado en IntelliJ con acceso a tu proyecto y capacidad de navegar la web automáticamente.

---

## Requisitos previos

Antes de comenzar, asegúrate de tener instalado:

- IntelliJ IDEA (cualquier edición)
- Node.js (verifica con `node --version` en tu terminal)
- Java 17 o superior
- Un proyecto Serenity BDD existente

Si no tienes Node.js, descárgalo desde [nodejs.org](https://nodejs.org) e instala la versión LTS.

---

## Paso 1 — Instalar Claude Code

Abre una terminal (cmd o PowerShell en Windows) y ejecuta:

```bash
npm install -g @anthropic-ai/claude-code
```

Verifica que quedó instalado correctamente:

```bash
claude --version
```

---

## Paso 2 — Plugin en IntelliJ IDEA

1. Ve a `Settings → Plugins → Marketplace`
2. Busca **"claude mcp"**
3. Instala **"MCP Servers for AI Assist..."** (by meanmail.dev)
4. Reinicia IntelliJ cuando lo solicite

> También encontrarás `Claude Code [Beta]` en `Settings → Tools` — es la integración oficial de Anthropic con JetBrains y se configura automáticamente al instalar Claude Code.

---

## Paso 3 — Agregar el servidor MCP de filesystem

El servidor `filesystem` le da a Claude acceso directo a los archivos de tu proyecto. Sin él, Claude trabaja sin contexto.

```bash
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\a\tu\proyecto"
```

**Reemplaza** `C:\ruta\a\tu\proyecto` con la ruta real de tu proyecto. Por ejemplo:

```bash
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\Users\TuUsuario\IdeaProjects\ProyectoWebQA"
```

> **Importante en Windows:** Si tu ruta tiene espacios, ponla siempre entre comillas dobles.

---

## Paso 4 — Agregar el servidor MCP de Playwright

El servidor `playwright` le da a Claude control real del navegador para navegar páginas, inspeccionar el DOM y tomar selectores reales.

```bash
claude mcp add playwright npx -- -y @playwright/mcp@latest
```

---

## Paso 5 — Verificar la configuración

```bash
claude mcp list
```

Deberías ver algo así:

```
filesystem: npx -y @modelcontextprotocol/server-filesystem C:\...\ProyectoWebQA - √ Connected
playwright: npx -y @playwright/mcp@latest - √ Connected
```

Si alguno aparece con `× Failed to connect`, revisa la sección de solución de problemas al final de esta guía.

---

## Paso 6 — Trabajar con múltiples proyectos

Si trabajas con varios proyectos simultáneamente, puedes nombrar los servidores diferente:

```bash
claude mcp add proyecto-a npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\ProyectoA"
claude mcp add proyecto-b npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\ProyectoB"
```

Para eliminar un servidor existente:

```bash
claude mcp remove filesystem
```

---

## Uso básico con Serenity BDD

Una vez configurado, abre Claude Code en IntelliJ y prueba estos prompts:

### Diagnóstico inicial del proyecto
```
Lee la estructura completa de mi proyecto y dime:
- Qué tecnologías y frameworks detectas
- Cómo está organizado (capas, paquetes, módulos)
- Qué flujos ya tienen tests y cuáles no
- Qué me recomiendas automatizar primero
```

### Exploración y generación automática de tests
```
Abre el navegador y ve a https://tu-aplicacion.com/login
Haz login con usuario "admin" y contraseña "admin123"

Navega el módulo de [nombre del módulo] e identifica:
- Todas las acciones posibles (crear, editar, eliminar, buscar)
- Todos los selectores reales del DOM

Con lo que encuentres, genera la suite completa de Serenity BDD:
- Feature con escenarios positivos y negativos
- StepDefinitions reutilizando pasos existentes
- Tasks de Screenplay con actor.attemptsTo()
- PageObject con los selectores reales

Sigue exactamente el estilo de código de mi proyecto
```

### Reparación de tests rotos
```
Mi test [NombreTest] está fallando con este error:
[pega el stacktrace aquí]

Ve a la URL del test, inspecciona el DOM actual
y actualiza los selectores rotos en mi PageObject
```

### Análisis de reportes Serenity
```
Lee los archivos JSON en target/site/serenity/ y dime:
- Cuántos tests pasan, fallan y están pendientes
- Los 5 tests que fallan con más frecuencia
- Los 5 tests más lentos
- Qué patrones de fallo detectas
```

---

## Solución de problemas comunes

### `filesystem: × Failed to connect`

El problema más común en Windows son los espacios en la ruta. Solución:

```bash
claude mcp remove filesystem
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\sin espacios\proyecto"
```

O mueve el proyecto a una ruta sin espacios: `C:\QA\MiProyecto`

### `MCP server filesystem already exists`

El servidor ya está registrado. Elimínalo primero:

```bash
claude mcp remove filesystem
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\tu\nueva\ruta"
```

### Claude dice "no tengo herramienta de navegador"

Significa que el servidor de Playwright no está agregado o no está conectado:

```bash
claude mcp add playwright npx -- -y @playwright/mcp@latest
```

Luego abre una **sesión nueva** de Claude Code — las sesiones existentes no detectan servidores agregados después de iniciadas.

### `Needs authentication` en Gmail, Drive o Calendar

Estos son servidores opcionales de Google. Para autenticarlos, ejecuta `claude` en la terminal e intenta usar uno de esos servicios — te pedirá autenticarte con tu cuenta de Google. No son necesarios para el trabajo de QA automatizado.

---

## Referencia rápida de comandos

| Acción | Comando |
|--------|---------|
| Agregar servidor filesystem | `claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta"` |
| Agregar servidor Playwright | `claude mcp add playwright npx -- -y @playwright/mcp@latest` |
| Ver todos los servidores | `claude mcp list` |
| Eliminar un servidor | `claude mcp remove [nombre]` |
| Abrir Claude Code en terminal | `claude` |
| Ver versión instalada | `claude --version` |

---

## Capacidades principales de MCP para QA

Una vez configurado, puedes aprovechar estas capacidades:

- **Exploración web automática** — Claude navega tu aplicación real e identifica selectores sin que copies nada manualmente
- **Generación de suites completas** — Describe el flujo en lenguaje natural y genera feature, steps, tasks y page objects
- **Auto-mantenimiento** — Cuando la UI cambia y los tests se rompen, Claude navega la página actual y repara los selectores
- **Análisis de reportes** — Lee los JSON de Serenity y detecta patrones de fallo, tests lentos y gaps de cobertura
- **Generación de datos de prueba** — Crea CSVs con datos realistas y coherentes para tus Scenario Outlines
- **Auditoría de accesibilidad** — Verifica criterios WCAG 2.1 navegando la aplicación real
- **Testing cross-browser** — Configura múltiples servidores Playwright para Chrome, Firefox y Safari en paralelo

---

*Guía generada desde experiencia real de configuración — Edward Jaime García · 2026*
