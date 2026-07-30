# Limitaciones reales de Claude + MCP en testing

> Catálogo vivo de limitaciones encontradas en la práctica — no en la teoría. Se va llenando a medida que se documentan más casos, igual que el [caso de estudio](./caso-estudio-suite-e2e-mcp.md). Si encuentras una limitación nueva usando este enfoque, agrégala aquí con la misma estructura: qué pasa, por qué pasa, y qué se puede hacer al respecto.

---

## 1. Falta de contexto persistente entre sesiones

### Qué pasa

Cada conversación con Claude es, por defecto, **una sesión aislada**. Si se trabaja en un proyecto de automatización durante varios días, y cada día se abre un chat nuevo, Claude no recuerda automáticamente lo que se decidió, exploró, o construyó el día anterior — hay que volver a darle ese contexto.

> **Analogía:** es como contratar a un consultor externo brillante, pero que **sufre de amnesia total cada vez que sale de la oficina**. Cada mañana que vuelve, hay que volver a explicarle en qué proyecto está trabajando, qué se decidió ayer, y por qué. El consultor es igual de capaz cada día — el problema es que no hay continuidad entre sesiones si nadie se encarga de re-informarlo.

Esto se nota especialmente en el flujo de este repo: si la exploración incremental de un módulo (login → módulos → flujo detallado, como en el caso de estudio) se corta a mitad de camino y se retoma al día siguiente en un chat nuevo, ese progreso de "entendimiento del proyecto" no viaja solo — hay que volver a dárselo.

### Por qué pasa

Claude no tiene memoria automática entre conversaciones separadas salvo que exista un mecanismo explícito diseñado para eso. No es un descuido ni un error — es una decisión de diseño de cómo funcionan los modelos de lenguaje en general: cada conversación procesa el texto que tiene disponible en esa sesión, sin acceso implícito a sesiones anteriores.

### Qué se puede hacer al respecto

**Opción 1 — `CLAUDE.md` en el proyecto (Claude Code)**

Claude Code tiene un mecanismo pensado específicamente para esto: un archivo `CLAUDE.md` en la raíz del proyecto, que se lee automáticamente al iniciar una sesión ahí. Se puede usar para guardar contexto persistente:

```markdown
# CLAUDE.md — Contexto del proyecto

## Qué es esta app
Sistema de gestión comercial: administración de usuarios (preventa, autoventa,
mercadeo), gestión de pedidos, seguimiento a vendedores.

## Stack de automatización
Serenity BDD + Screenplay Pattern + Cucumber. Tests en `src/test/java/...`

## Decisiones ya tomadas
- Los dropdowns del módulo de pedidos tienen varios <div> anidados —
  ver docs/web/[nota de selectores] antes de escribir un nuevo selector ahí.
- Las pruebas se generan una por una, nunca en lote (ver metodología del
  caso de estudio).

## Convenciones del equipo
- Nombres de Tasks en inglés, nombres de Features en español.
- Cada escenario nuevo debe usar datos de la cuenta de prueba `qa_test_01`,
  nunca cuentas reales (ver seguridad-mcp-qa.md).
```

> ⚠️ El comportamiento exacto de `CLAUDE.md` puede variar según la versión de Claude Code — conviene confirmar el detalle actualizado en la documentación oficial (`docs.claude.com`) antes de asumir un comportamiento específico no probado.

**Opción 2 — Memoria entre chats (claude.ai)**

Existe una función de memoria que permite que Claude busque información de conversaciones pasadas dentro de claude.ai, si está activada en la configuración. Es un mecanismo distinto al de Claude Code, pensado para el cliente de chat, no para sesiones de terminal/IDE.

**Opción 3 — Archivo de contexto manual (funciona en cualquier cliente)**

Si no se quiere depender de una función específica de un producto, la alternativa más simple y portable es mantener un archivo de contexto propio (parecido al ejemplo de `CLAUDE.md` de arriba) y **pegarlo o referenciarlo manualmente al inicio de cada sesión nueva** — más trabajo manual, pero funciona igual en cualquier cliente MCP, no solo en Claude Code.

### Tabla comparativa de las 3 opciones

| Opción | Automático | Requiere Claude Code específicamente | Esfuerzo |
|---|---|---|---|
| `CLAUDE.md` | Sí, se lee solo | Sí | Bajo (una vez escrito) |
| Memoria de claude.ai | Parcial, según configuración | No (es de claude.ai) | Bajo |
| Archivo de contexto manual | No, hay que pegarlo cada vez | No, funciona en cualquier cliente | Medio (repetitivo) |

---

## 2. [Pendiente de documentar] Reproducibilidad entre sesiones

¿El mismo prompt, sobre el mismo módulo, produce resultados consistentes en distintas sesiones/días? Si has notado variaciones (mismo flujo, distinto código generado, distintos selectores elegidos), documentarlo aquí con ejemplos concretos ayudaría a calibrar expectativas de otros equipos.

---

## 3. [Pendiente de documentar] Consumo de tokens / costo en exploraciones largas

¿Cuánto "cuesta" en tokens una sesión típica de exploración incremental de un módulo grande? ¿Hay un punto en el que conviene cortar la sesión y empezar una nueva por límites prácticos, más allá de lo que ya se resolvió con `CLAUDE.md`?

---

## 4. Ver también: la limitación ya documentada en el caso de estudio

La limitación de que Claude puede "creer" que una acción funcionó a nivel de código sin que el resultado sea visible en pantalla ya está documentada a fondo en el [caso de estudio](./caso-estudio-suite-e2e-mcp.md#la-limitación-más-importante-detectada-ve-por-debajo-no-por-encima) — no se repite aquí para no duplicar contenido, pero es, en efecto, parte de este mismo catálogo de limitaciones reales.

---

## Cómo agregar una limitación nueva a este catálogo

1. Copiar la estructura de la sección 1 (Qué pasa / Por qué pasa / Qué se puede hacer al respecto).
2. Si no hay solución conocida todavía, está bien dejarlo como "sin solución clara por ahora" — es más honesto que forzar una respuesta.
3. Si la limitación ya se resolvió en otra nota del repo, enlazarla en vez de duplicar contenido (como se hizo con la sección 4 de arriba).
