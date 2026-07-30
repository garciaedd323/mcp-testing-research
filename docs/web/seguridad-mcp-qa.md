# Seguridad al usar Claude + MCP en testing

> Darle a un agente de IA acceso al filesystem y control del navegador es poderoso, pero no es gratis en términos de riesgo. Esta nota reúne las precauciones concretas que conviene tomar antes de conectar MCP a un proyecto real.

---

## Por qué esto merece su propia nota

Los servidores MCP usados en este repo (`filesystem`, `playwright`, `appium`) le dan a Claude capacidades que van más allá de "solo responder texto":

- **`filesystem`** — puede **leer y escribir archivos** dentro de la carpeta que se le indique.
- **`playwright`** — puede **controlar un navegador real**, incluyendo sesiones ya autenticadas, cookies, y cualquier dato visible en pantalla.
- **`appium`** — puede **controlar un dispositivo/emulador real**, incluyendo apps con datos reales si el emulador no está aislado.

Ninguno de los tres es peligroso por diseño, pero **el alcance que se le da** (qué carpeta, qué ambiente, qué cuenta) sí determina el riesgo real.

---

## 1. Credenciales — nunca en texto plano dentro del alcance de Claude

Varios documentos de este repo (`guia-mcp-qa-serenity.md`, `testing-playwright-mcp-vs-screenplay.md`) mencionan archivos como `config.json` con usuario/contraseña del ambiente probado.

**Regla práctica:**
- Usar **siempre credenciales de un ambiente de prueba/staging**, nunca de producción.
- Si el `config.json` (u otro archivo similar) contiene credenciales reales, debe estar en `.gitignore` — nunca debe llegar a un commit, y por lo tanto, tampoco debería ser algo que Claude "lea y decida reutilizar" sin que la persona lo sepa.
- Preferir variables de entorno sobre credenciales hardcodeadas en archivos que el servidor `filesystem` pueda leer libremente.

> Si Claude tiene acceso de lectura a la carpeta del proyecto, **tiene acceso a lo mismo que cualquier archivo ahí contenga** — incluidas credenciales, si están mal gestionadas. La responsabilidad de no exponerlas es la misma que ya existe para cualquier otro colaborador humano con acceso al repositorio.

---

## 2. Alcance del servidor `filesystem` — nunca apuntar a la raíz del disco

```bash
# ❌ Riesgoso — acceso a todo el disco
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\"

# ✅ Correcto — acceso solo a la carpeta del proyecto específico
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem "C:\Proyectos\mi-suite-qa"
```

La nota `mcp-filesystem-entre-proyectos.md` de este mismo repo ya cubre cómo cambiar la ruta al pasar de un proyecto a otro — la recomendación de seguridad complementaria es: **nunca dejar el alcance más amplio de lo necesario**, ni siquiera por comodidad de "no tener que reconfigurar cada vez".

---

## 3. Nunca apuntar el navegador (MCP Playwright) a producción

Es tentador usar la URL real de producción "porque ya está andando y es más representativa", pero:

- Cualquier dato que aparezca en pantalla durante la sesión (nombres de clientes reales, pedidos reales, información de vendedores reales — como en el caso de estudio de este repo) queda expuesto al agente y, por extensión, forma parte de la conversación que se procesa.
- Una exploración automática puede **disparar acciones reales** sin querer (crear un pedido de prueba en el sistema real, modificar un usuario real, etc.) si el flujo no se limita cuidadosamente a lectura/observación.

**Regla práctica:** usar siempre un ambiente de staging/QA con datos ficticios, igual que se haría con cualquier otra suite de automatización tradicional (Selenium, Cypress, Playwright puro). MCP no cambia esta regla — solo hace más fácil olvidarla, porque "se siente como una conversación", no como código de test.

---

## 4. Cuentas de prueba, no cuentas personales/reales

Al automatizar login (como en la guía de Serenity BDD de este repo), usar siempre:
- Un usuario de prueba dedicado, con permisos acotados al mínimo necesario para el flujo que se está probando.
- Nunca la cuenta personal del QA/desarrollador, ni una cuenta con permisos de administrador si el flujo no lo requiere.

---

## 5. Qué "ve" realmente Claude durante una sesión con MCP

Es importante tener claridad sobre esto: durante una sesión de exploración o generación de tests, el contenido que Claude procesa (texto de la página, estructura del DOM, resultado de comandos del filesystem) **forma parte de la conversación** con el modelo, de la misma manera que cualquier texto que se pegara manualmente en el chat.

Esto no es distinto, en esencia, a pegarle a Claude una captura de pantalla o un fragmento de código — pero como MCP lo automatiza, es más fácil que **información sensible entre a la conversación sin que la persona la revise conscientemente primero**, como sí pasaría al copiar y pegar manualmente.

**Regla práctica:** antes de iniciar una sesión de exploración larga sobre un módulo desconocido, tener claro qué tipo de datos es razonable que aparezcan en pantalla (¿hay datos personales de clientes reales? ¿información financiera?) y decidir si ese módulo específico debe probarse con datos ficticios antes de dejar que el agente lo explore libremente.

---

## 6. Checklist rápido antes de conectar MCP a un proyecto

- [ ] ¿La URL/ambiente es staging o QA, nunca producción?
- [ ] ¿Las credenciales usadas son de una cuenta de prueba, no personal ni de administrador innecesario?
- [ ] ¿El `config.json` (o equivalente) con credenciales está en `.gitignore`?
- [ ] ¿El alcance del servidor `filesystem` apunta solo a la carpeta del proyecto, no a todo el disco?
- [ ] ¿Los datos visibles en las pantallas a explorar son ficticios o ya están anonimizados?
- [ ] Si es un emulador Appium, ¿está aislado (sin cuentas reales sincronizadas, sin datos de producción del dispositivo)?

---

## 7. Por qué esto no es exclusivo de MCP

Ninguna de estas reglas es nueva o exclusiva de Claude+MCP — son las mismas precauciones que ya aplicarían con Selenium, Cypress o Playwright "tradicionales" (no apuntar a producción, no hardcodear credenciales, usar cuentas de prueba). Lo que cambia con MCP es que el punto de entrada se siente más conversacional y menos "como código", lo cual puede hacer que estas reglas básicas se relajen sin darse cuenta si no se tienen presentes explícitamente.
