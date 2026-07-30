# Appium Inspector vs capabilities.json: ¿son lo mismo?

## La confusión

Si ya tienes las capacidades (capabilities) configuradas y funcionando en **Appium Inspector** — es decir, ya probaste que conecta bien con tu emulador y tu app — surge la duda: ¿ese archivo se reutiliza directamente para el servidor MCP, o hay que crear uno nuevo?

**Respuesta corta: hay que crear un archivo físico nuevo.**

Aunque el contenido (las capacidades) sea el mismo, son dos cosas distintas en la práctica.

## La diferencia

| | Appium Inspector | Servidor MCP |
|---|---|---|
| Dónde vive la info | Dentro de la propia app, en un campo de texto o en "sesiones guardadas" internas de Inspector | Un archivo `.json` físico en tu disco |
| ¿Se puede reutilizar directamente? | No — Inspector no lo guarda como un archivo `.json` accesible en una carpeta que tú elijas | Necesita la ruta exacta a un archivo `.json` |
| ¿Lo puedes copiar? | Sí, el contenido | Sí — pegas el mismo JSON en el archivo nuevo |

## Por qué no sirve "tal cual"

Appium Inspector guarda las capacidades internamente (en la configuración de la app, como sesiones guardadas dentro de su propia interfaz), **no como un archivo `.json` en una carpeta que tú puedas apuntar** desde el servidor MCP.

Es decir: aunque tengas las capacidades configuradas y funcionando en Inspector, no hay un archivo en disco que el MCP pueda leer directamente desde ahí.

## Qué hacer entonces

1. Si ya tienes las capacidades funcionando en Appium Inspector, copia ese mismo JSON que usaste ahí
2. Pégalo en un archivo **nuevo, físico**, en la ruta definida para el proyecto: `C:\mcp-mobile\capabilities.json`
3. Ese archivo nuevo es al que apunta la variable `CAPABILITIES_CONFIG` en la configuración del MCP

## Lección general

> Una herramienta con interfaz gráfica (como Appium Inspector) que "recuerda" tu configuración no necesariamente la expone como un archivo que otra herramienta pueda leer. Cuando dos herramientas necesitan compartir configuración, hay que verificar si realmente comparten el mismo archivo o si solo comparten el mismo *contenido* copiado a mano.

## Relacionado
- [Paso 3 (revisado) — Conectar appium-mcp a Claude Code](./paso3-conectar-appium-claude-code.md)
