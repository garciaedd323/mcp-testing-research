# Caso de estudio: generación de suite E2E con Claude + MCP en un proyecto real

> Documento basado en experiencia real de configuración y uso, no en un escenario teórico. Complementa el [mapa de capacidades](./capacidades-mcp-qa.md) con evidencia concreta de qué tan bien (y en qué falla) funciona este enfoque en la práctica.

> **📊 Resultado clave:** ~7 días estimados escribiendo la suite a mano → ~3 días reales con Claude+MCP (reducción de más del 50% del tiempo).

---

## Contexto del proyecto

App de gestión comercial con varios módulos: administración de usuarios (preventa, autoventa, mercadeo), gestión de pedidos, y seguimiento a vendedores. Proyecto de automatización en Serenity BDD (Screenplay Pattern + Cucumber), con Claude Code + MCP de Playwright conectado sobre IntelliJ.

---

## Metodología: exploración incremental, no de una sola pasada

La decisión metodológica más importante de este caso fue **no pedirle a Claude que explorara todo el sistema de una vez**. En cambio, se siguió un enfoque escalonado:

1. **Explorar solo el login** — primer contacto con la app, sin avanzar más.
2. **Listar los módulos disponibles**, sin entrar todavía a ninguno.
3. **Entrar a un módulo específico** y observar qué contiene, en términos generales (qué secciones, qué acciones existen).
4. **Escribir el flujo end-to-end deseado, con pasos muy detallados y explícitos** — no un prompt abierto tipo "prueba este módulo", sino la secuencia exacta de lo que se quería validar.
5. Solo entonces, **Claude exploraba cada punto específico del flujo** (inspeccionando la página real vía MCP) y generaba el código.
6. Repetir el ciclo prueba por prueba: se daba el contexto y los pasos de la siguiente prueba **después** de que la anterior quedara terminada — nunca se pidieron varias pruebas en paralelo.

> **Por qué importa este orden:** saturar a Claude pidiéndole "explora todo el sistema y genera todo" de una sola vez es exactamente el tipo de prompt que las notas de `prompts-mcp.md` recomiendan evitar. El enfoque incremental permitió mantener el contexto acotado en cada paso, y corregir el rumbo antes de que un malentendido se propagara a 42 pruebas distintas.

---

## Qué se generó

- **42 pruebas end-to-end**, generadas una por una (no en lote).
- Estructura completa de Serenity BDD: **Tasks** (Screenplay), **Step Definitions**, y **Features** con **Scenario Outline** (esquema del escenario) para cubrir variantes de datos sin duplicar el flujo completo.

---

## Problemas técnicos que aparecieron durante la generación (y que Claude resolvió)

| Problema | Cómo se resolvió |
|---|---|
| Selectores con índices poco confiables | Requirió volver a inspeccionar la página con MCP para encontrar un selector más estable |
| Dropdowns con múltiples `<div>` superpuestos, dificultando escribir en el input real | Claude, tras re-inspeccionar, identificó la estructura real y ajustó el selector/la interacción para llegar al input correcto |

Ambos casos requirieron un ciclo de "esto no funcionó → volver a inspeccionar → ajustar" — no se resolvieron a la primera, pero sí se resolvieron sin intervención manual de código, solo dándole a Claude la oportunidad de re-observar la página.

---

## La limitación más importante detectada: "ve por debajo, no por encima"

Este es el hallazgo más valioso del caso de estudio.

Claude, apoyado en el MCP de Playwright, tiene visibilidad de lo que ocurre **a nivel de DOM/eventos** (lo que "pasa por debajo" de la página), pero no siempre corresponde a lo que un usuario real **vería en pantalla** (lo que "pasa por encima", visualmente).

Concretamente: hubo casos donde Claude **creía que había dado clic en un botón exitosamente** (el evento se disparó a nivel de código) pero **visualmente el clic no producía el efecto esperado en la interfaz** — es decir, el sistema registraba la interacción, pero el resultado visible no ocurría como se esperaría.

Esto tiene una implicación práctica directa: **la validación humana visual sigue siendo necesaria**. Hubo situaciones que se detectaron a simple vista (mirando la pantalla) que Claude, apoyado solo en la inspección técnica de la página, no detectó por sí solo.

> **Lección para cualquiera que replique este flujo:** no basta con que Claude reporte "hice clic y continué" — conviene revisar visualmente (o con un assert de estado visible en pantalla, no solo de evento disparado) que la acción tuvo el efecto esperado, especialmente en pasos críticos del flujo.

---

## Dónde Claude ayudó más allá de lo esperado

Más allá de generar código, Claude sirvió como **apoyo para entender reglas de negocio de la página que el desarrollador/QA no conocía de antemano** — es decir, al explorar la interfaz e inferir comportamientos, ayudó a clarificar cómo se suponía que debía comportarse cierta funcionalidad, no solo a automatizarla.

---

## Resumen del caso

| Aspecto | Resultado |
|---|---|
| Pruebas generadas | 42 E2E, una por una (no en lote) |
| Estructura generada | Tasks (Screenplay), Step Definitions, Features con Scenario Outline |
| Metodología clave | Exploración incremental: login → módulos → un módulo → flujo detallado → generación |
| Problemas técnicos resueltos por Claude | Selectores con índices inestables, dropdowns con `div` anidados |
| Limitación principal detectada | Diferencia entre lo que ocurre "por debajo" (DOM/eventos) y lo que se ve "por encima" (resultado visual real) |
| Valor inesperado | Aclaración de reglas de negocio de la app, no solo generación de código |
| Modelo de Claude usado | Claude Sonnet 5, vía Claude Code v2.1.220 (plan Claude Team) |
| Tiempo total vs. estimado manual | ~3 días con Claude+MCP, vs. ~7 días estimados escribiéndolo a mano (reducción de más del 50%) |

---

## Recomendaciones para quien replique este flujo

1. **No pidas exploración total de una sola vez.** El enfoque incremental (login → módulos → un módulo → flujo detallado) evitó saturar el contexto y permitió corregir el rumbo a tiempo.
2. **Da los pasos del flujo E2E de forma explícita**, no como una descripción general — cuanto más detallado el paso a paso deseado, mejor la calidad del resultado.
3. **Prevé que algunos selectores requerirán una segunda inspección** — especialmente índices y componentes con estructuras anidadas complejas (dropdowns custom).
4. **No confíes ciegamente en que "hizo clic" significa "funcionó visualmente".** Revisa manualmente los pasos críticos, o agrega asserts de estado visible, no solo de evento disparado.
5. **Aprovecha el proceso también para entender la app**, no solo para automatizarla — la exploración de Claude puede revelar comportamientos que ni el propio equipo tenía documentados.

---

## Preguntas abiertas para un siguiente caso de estudio

- ¿El problema de "ve por debajo, no por encima" se reduce usando capturas de pantalla como parte de la verificación, en vez de solo inspección de DOM?
- ¿Cómo escala este enfoque incremental cuando el número de pruebas crece de 42 a varios cientos — sigue siendo viable hacerlo una por una?
- ¿Qué tan reproducibles son estos resultados con otro modelo o versión de Claude?
