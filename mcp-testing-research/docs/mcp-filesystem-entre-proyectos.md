# Cómo funciona MCP filesystem al cambiar entre proyectos

## La confusión inicial

Al trabajar en varios proyectos de QA en carpetas distintas, surge la duda: si ya configuré un servidor MCP `filesystem` para un proyecto, ¿se actualiza solo cuando cambio a otro proyecto/carpeta?

**Respuesta corta: no.** El servidor `filesystem` queda apuntando a **una ruta fija**, la que le diste al momento de crearlo. No "sigue" al proyecto activo.

## Por qué pasa esto

Cuando ejecutas:
```bash
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\del\proyecto"
```

Esa ruta queda guardada tal cual en la configuración (`.claude.json`). El servidor `filesystem` no sabe en qué proyecto estás trabajando "ahora" — solo sabe la carpeta que le diste una vez.

## Qué hacer al cambiar de proyecto

Cada vez que quieras que `filesystem` apunte a una carpeta distinta, hay que reconfigurarlo. El patrón es siempre el mismo:

**1. Elimina el servidor existente (apunta a la ruta vieja):**
```bash
claude mcp remove filesystem
```

**2. Agrégalo de nuevo con la nueva ruta:**
```bash
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\ruta\del\nuevo\proyecto"
```

**3. Verifica que quedó apuntando bien:**
```bash
claude mcp list
```

## Nota práctica

Si trabajas seguido en varios proyectos y este ir y venir se vuelve repetitivo, existe la opción de configurar MCP **por proyecto** en vez de globalmente, para no tener que quitar y agregar cada vez. Es un tema para investigar más adelante — por ahora, mientras se usa un solo servidor `filesystem` a la vez, el patrón remove → add → list es la solución más simple.

## Relacionado
- [Troubleshooting: rutas con espacios en Windows](./troubleshooting-mcp-filesystem-windows.md)
