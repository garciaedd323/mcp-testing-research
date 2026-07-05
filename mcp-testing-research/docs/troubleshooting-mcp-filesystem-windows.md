# Troubleshooting: "Failed to connect" al agregar un servidor MCP filesystem

## Contexto
Al configurar un servidor MCP de tipo `filesystem` con Claude Code (CLI) en Windows, para darle acceso a una carpeta local de pruebas de QA.

## El error

Comando usado (primer intento):
```
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem C:\Users\EdwardJaimeGarciaCue\Documents\Archivos del escritorio\PruebasQA\PruebaMcp
```

El comando se ejecutó sin error aparente y modificó `.claude.json`, pero al correr:
```
claude mcp list
```

El servidor aparecía como:
```
filesystem: ... - ✗ Failed to connect
```

Mientras que otros servidores (Gmail, Google Drive) sí mostraban conexión exitosa.

## Causa

La ruta de la carpeta contiene **espacios** (`Archivos del escritorio`) y no estaba entre comillas. Sin comillas, la terminal/`npx` interpreta la ruta como varios argumentos separados en lugar de una sola ruta, por lo que el servidor no encuentra el directorio y falla al iniciar.

## Solución

1. Eliminar el servidor mal configurado:
```
claude mcp remove filesystem
```

2. Volver a agregarlo, esta vez con la ruta **entre comillas dobles**:
```
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\Users\EdwardJaimeGarciaCue\Documents\Archivos del escritorio\PruebasQA\PruebaMcp"
```

3. Verificar de nuevo:
```
claude mcp list
```

## Lección general

> Si una ruta de Windows contiene espacios (por ejemplo, cualquier carpeta con "Archivos del escritorio", "Program Files", "Mis documentos", etc.), **siempre** debe ir entre comillas dobles al pasarla como argumento en la terminal — sin importar qué comando sea.

## Nota aparte

Después de este fix, los servidores de Google (Gmail, Drive, Calendar) aparecieron como "Needs authentication". Esto es un tema separado (expiración de sesión de autenticación OAuth) y no está relacionado con el error de `filesystem`.

## Fuentes
- [Claude Code CLI reference](https://code.claude.com/docs/en/cli-reference)

## Relacionado
- [Cómo funciona MCP filesystem al cambiar entre proyectos](./mcp-filesystem-entre-proyectos.md)
