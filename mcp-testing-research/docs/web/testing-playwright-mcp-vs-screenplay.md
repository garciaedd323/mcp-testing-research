# Testing con Playwright MCP: qué es, cómo se estructura y cuándo usarlo

Este documento resume el análisis de un proyecto de pruebas automatizadas basado en **Playwright MCP** (Model Context Protocol), y lo compara con un enfoque de automatización tradicional basado en **Screenplay Pattern + Cucumber (Features) + JUnit**.

---

## 1. ¿Qué es Playwright MCP?

Es un enfoque de testing donde, en lugar de escribir código de automatización tradicional, se le dan **instrucciones en lenguaje natural** a un agente de IA (por ejemplo, dentro de Cursor, VS Code o Claude Code). El agente usa herramientas MCP para controlar un navegador real: hace clics, llena formularios, navega y toma capturas de pantalla, como si fuera una persona probando la app manualmente.

---

## 2. Estructura de un proyecto Playwright MCP

```
mi-proyecto-testing/
├── config.json              # Configuración del ambiente y credenciales
├── mcp-test-prompt.md        # Instrucciones paso a paso para el agente
├── run-mcp-tests.js          # Script informativo (lee config.json)
├── save-results.js           # Guarda resultados JSON y dispara el reporte HTML
├── generate-html-report.js   # Genera el reporte HTML final
├── utils/
│   └── test-helpers.js       # Funciones de apoyo reutilizables
├── screenshots/               # Capturas de cada paso
├── results/                   # Resultados de cada ejecución en JSON
└── reports/                   # Reportes HTML generados
```

### Descripción de cada archivo

| Archivo | Para qué sirve | Analogía |
|---|---|---|
| `config.json` | Guarda URL, usuario, contraseña, timeout y si se deben tomar capturas | La ficha con la dirección y la llave que le das a alguien para que entre a revisar tu casa |
| `mcp-test-prompt.md` (y variantes) | Guion en lenguaje natural con los pasos exactos que debe seguir el agente | Una receta de cocina: mismos utensilios, pasos distintos según qué quieras probar |
| `run-mcp-tests.js` | Solo imprime en consola un resumen de la configuración cargada | Un checklist que te recuerda qué vas a hacer, no ejecuta nada por sí mismo |
| `save-results.js` | Guarda los resultados de una corrida en `results/*.json` y dispara automáticamente el reporte HTML | El archivador que guarda tus notas de la prueba ordenadamente |
| `generate-html-report.js` | Convierte un JSON de resultados en un reporte visual HTML | Convierte notas crudas en un informe con gráficos y estadísticas |
| `utils/test-helpers.js` | Funciones de apoyo: calcula % de éxito, crea carpetas necesarias | Utilería interna reutilizada por `save-results.js` |
| `screenshots/` | Capturas tomadas durante cada paso de la prueba | Evidencia visual de que cada paso funcionó |
| `results/` | Historial de ejecuciones en formato JSON (pasos, duración, éxito/fallo) | La bitácora o libro de actas de cada corrida |
| `reports/` | Reportes HTML finales, listos para compartir | El informe visual final para mostrarle a alguien sin que tenga que leer JSON |

---

## 3. Flujo de ejecución

```
config.json  +  mcp-test-prompt.md
        │
        ▼
Agente + Playwright MCP  (ejecuta cada paso en el navegador)
        │
        ├──────────────┐
        ▼              ▼
  screenshots/     results/*.json
                        │
                        ▼
                save-results.js
                        │
                        ▼
            generate-html-report.js
                        │
                        ▼
                 reports/*.html
```

**Pasos para implementarlo en un proyecto nuevo:**

1. **Requisitos previos**: Node.js instalado, y un editor con soporte de agente MCP (Cursor, VS Code, Claude Code) con el **servidor MCP de Playwright** habilitado.
2. **Crear la estructura de carpetas** mostrada arriba.
3. **Escribir `config.json`** con las credenciales y datos del ambiente a probar.
4. **Escribir `mcp-test-prompt.md`** describiendo, paso a paso y en lenguaje natural, lo que el agente debe hacer en la app real (nombres exactos de campos y botones tal como aparecen en pantalla).
5. **Copiar los scripts de apoyo** (`save-results.js`, `generate-html-report.js`, `utils/test-helpers.js`) — son genéricos y no dependen de la app específica.
6. **Ejecutar la prueba**: pegar el contenido del `.md` en el chat del agente en modo Agent; el agente lee `config.json`, navega y toma capturas.
7. **Guardar resultados**: `node save-results.js '<json-de-resultados>'` — esto genera automáticamente el reporte HTML.
8. **Revisar** `screenshots/`, `results/` y `reports/`.
9. **Iterar**: si el agente falla en un paso, se ajusta la descripción del `.md` para que sea más precisa.

---

## 4. Comparación: chat directo vs. archivos versionados

Ambos métodos usan el mismo motor (un agente de IA con MCP de Playwright controlando el navegador). Lo que cambia es todo lo que envuelve a ese motor.

| | Chat directo (pegar pasos en el chat) | Archivos versionados (`config.json` + `.md` + scripts) |
|---|---|---|
| Persistencia | Los pasos se pierden al cerrar el chat | Quedan escritos y versionados en el repo |
| Repetibilidad | Hay que reescribir/recordar los pasos cada vez | Se ejecuta la misma prueba las veces que se quiera |
| Historial | No hay registro de corridas anteriores | `results/` guarda un historial comparable en el tiempo |
| Compartible | Vive solo en la conversación de una persona | `reports/*.html` se puede enviar a cualquiera |
| Multi-ambiente | Hay que repetir credenciales de palabra | `config.json` separa credenciales de instrucciones, reusable en dev/staging/producción |

**Cuándo usar cada uno:**
- **Chat directo** → exploración rápida, un bug puntual, "revisa si esto funciona ahora".
- **Archivos versionados** → pruebas de regresión repetibles, algo auditable, algo que el equipo también ejecutará.

---

## 5. Playwright MCP vs. Screenplay Pattern + Features + JUnit

Un proyecto con **Screenplay Pattern, Cucumber (Features en Gherkin) y JUnit** es un nivel distinto (y más maduro/profesional) de automatización, distinto tanto del chat directo como del enfoque de archivos `.md` + agente MCP.

| | Playwright MCP (`.md` + agente IA) | Screenplay + Features + JUnit |
|---|---|---|
| ¿Qué ejecuta la prueba? | Un agente de IA interpretando el `.md` en cada corrida | Código Java compilado (Actores, Tasks, Questions), sin IA en tiempo de ejecución |
| ¿Es determinista? | No del todo — la IA puede interpretar la pantalla distinto cada vez | Sí, mismo resultado siempre que la app no cambie |
| ¿Corre en CI/CD sin intervención? | Difícil — necesita un agente MCP activo | Sí, es su caso de uso natural (Jenkins, GitHub Actions, etc.) |
| Velocidad | Más lento (la IA "piensa" cada paso) | Rápido, es solo código ejecutándose |
| Dónde vive el conocimiento | En texto en lenguaje natural | En código versionado, reutilizable y refactorizable |
| Rol de la IA | Es el actor que ejecuta la prueba | Es un asistente de desarrollo para escribir/depurar el código de las pruebas |

### Cómo se combinan bien

Un uso razonable es usar el **MCP de Playwright como herramienta de exploración**: pedirle al agente que navegue la app en vivo para descubrir selectores, validar un flujo o detectar cambios en el DOM — y luego **traducir eso a código Screenplay real** (Tasks, Questions, step definitions) que sí queda dentro de la suite de JUnit. El MCP funciona ahí como un "explorador de apoyo", no como el motor final de la prueba en producción/CI.

---

## 6. Notas de seguridad

- El `config.json` normalmente contiene **credenciales reales** (usuario y contraseña del ambiente probado). Nunca debe subirse a un repositorio público tal cual — usar `.gitignore` para excluirlo, o reemplazar los valores sensibles por variables de entorno antes de commitear.
- Las carpetas `screenshots/`, `results/` y `reports/` suelen excluirse del control de versiones (salvo un `.gitkeep` para mantener la estructura), ya que son artefactos generados en cada corrida.
