# Decisión: Claude Desktop vs Claude Code para el proyecto móvil

## Contexto
Ya existe un proyecto de automatización **web** usando Claude Code (plugin/extensión dentro de IntelliJ), con un servidor MCP ya vinculado (Playwright).

Para el nuevo proyecto de automatización **móvil** (Appium + MCP), surge la pregunta: ¿usar Claude Desktop (app aparte) o Claude Code (el mismo que ya se usa en IntelliJ)?

## Diferencia entre ambos productos

| | Claude Desktop | Claude Code (plugin IntelliJ / CLI) |
|---|---|---|
| Qué es | Aplicación de escritorio independiente | Distinto producto, integrado al IDE o vía CLI |
| Dónde guarda la config de MCP | `%APPDATA%\Claude\claude_desktop_config.json` (Windows) | `.mcp.json` en la raíz del proyecto, o `~/.claude.json` a nivel de usuario |

Por eso, aunque ya se haya usado Claude antes (vía IntelliJ), la carpeta `%APPDATA%\Claude\` puede no existir todavía si nunca se ha instalado la app de escritorio Claude Desktop por separado — son configuraciones completamente distintas.

## Las dos opciones

### Opción A — Instalar Claude Desktop (app aparte)
- Programa independiente, separado de IntelliJ
- Se configura el servidor `appium-mcp` en su propio `claude_desktop_config.json`
- Ventaja: chat dedicado y separado del IDE, exclusivo para la automatización móvil

### Opción B — Usar Claude Code (el mismo que ya se usa en IntelliJ)
- No requiere instalar nada nuevo
- Se agrega el servidor `appium-mcp` al archivo de configuración que ya usa Claude Code (`.mcp.json` en el proyecto, o mediante el comando `claude mcp add` desde la terminal integrada de IntelliJ)
- Ventaja: todo queda centralizado en el mismo entorno que ya se usa para el proyecto web

## ✅ Decisión tomada: Opción B (Claude Code)

**Motivos:**

1. **Entorno ya funcionando** — Node.js, configuración de MCP y flujo de trabajo ya están probados y operativos con el proyecto web. No hay curva de aprendizaje de una app nueva.
2. **Menos piezas que mantener** — Instalar Claude Desktop implica un programa adicional corriendo, otra configuración que sincronizar y otro lugar donde buscar si algo falla.
3. **Consistencia entre proyectos** — Si en el futuro se necesita un flujo end-to-end que combine la suite web y la móvil (ej. empieza en la app y termina en el sitio web), tenerlos en el mismo cliente MCP lo simplifica bastante.
4. **Soporte nativo de JetBrains/IntelliJ** — Desde la versión 2025.2, IntelliJ IDEA trae un servidor MCP integrado, lo que facilita agregar servidores adicionales (como el de Appium) como una entrada más en la configuración existente.
5. **No hay conflicto real entre MCPs** — Playwright/web y Appium/móvil pueden convivir sin problema como entradas separadas dentro del mismo archivo de configuración.

## Cuándo sí conviene la Opción A (Claude Desktop aparte)

Solo tiene sentido si se quiere mantener el contexto de conversación de móvil completamente separado y "limpio" del de web. Por ejemplo:
- Si trabajan equipos distintos en cada frente (web vs móvil)
- Si preocupa mezclar el contexto de ambas conversaciones dentro de una misma sesión de chat

Para un solo QA automatizando ambos frentes, esta separación no compensa la fricción extra de mantener dos configuraciones y dos programas distintos.

## Próximo paso

Adaptar el Paso 3 del tutorial (que originalmente estaba pensado para Claude Desktop) para configurar el servidor `appium-mcp` dentro de **Claude Code**, usando `.mcp.json` en el proyecto o el comando `claude mcp add`. El resto de la instalación (`capabilities.json`, Android SDK, Appium, emulador, etc.) se mantiene exactamente igual.

👉 Ver: [Paso 3 (revisado) — Conectar appium-mcp a Claude Code](./paso3-conectar-appium-claude-code.md)

## Nota sobre el documento anterior
El documento [`paso3-conectar-appium-claude-desktop.md`](./paso3-conectar-appium-claude-desktop.md) sigue siendo válido como referencia de la **Opción A**, pero **no es la ruta que se está siguiendo** en este proyecto. Se mantiene en el repo por si en algún momento aplica el escenario de equipos separados descrito arriba.
