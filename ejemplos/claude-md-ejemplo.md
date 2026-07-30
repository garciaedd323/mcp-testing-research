# CLAUDE.md — Ejemplo real (anonimizado)

> Esta plantilla está basada en un `CLAUDE.md` real usado en un proyecto de automatización con Serenity BDD + Screenplay + MCP (ver [caso de estudio](../docs/web/caso-estudio-suite-e2e-mcp.md)). Los nombres de la empresa, apps y URLs fueron reemplazados por versiones genéricas — la estructura, las advertencias y las lecciones aprendidas son reales. Cópiala y adáptala a tu propio proyecto.

---

# CLAUDE.md — Contexto del proyecto

> Este archivo se lee automáticamente al iniciar una sesión de Claude Code en esta carpeta. Mantenerlo actualizado evita tener que re-explicar el proyecto en cada sesión nueva (ver `limitaciones-reales-mcp.md`).

## Qué es esta app

Sistema de gestión comercial con varias aplicaciones/suites dentro del mismo proyecto:
- **AppLegacy1**
- **AppLegacy2**
- **ModuloPreventa**
- **ModuloAutoventa**
- **AppV2** (con sub-apps: ModuloPreventaV2, ModuloAutoventaV2, ModuloAliadosV2)

Módulos funcionales: administración de usuarios (preventa, autoventa, mercadeo), gestión de pedidos, seguimiento a vendedores.

**Ambiente de pruebas:** `https://staging.tu-dominio.com/login` — único ambiente configurado (no hay perfiles staging/prod separados). Definido en `src/test/resources/serenity.conf` (`environments.default.webdriver.base.url`), leído vía `Constants.WEB_URL` en `OpenWeb.java`.

## Stack de automatización

- **Lenguaje:** Java (`sourceCompatibility`/`targetCompatibility` = 1.8 en `build.gradle` — el bytecode target es Java 8, aunque puede correr sobre un JDK más moderno instalado).
- **Build tool:** Gradle (`build.gradle` + `gradlew`). No hay `pom.xml`.
- **Plugin:** `net.serenity-bdd.serenity-gradle-plugin` versión `3.3.4`.
- **Framework:** Serenity BDD + Screenplay Pattern + Cucumber.
- **IDE:** IntelliJ.
- **MCP conectado:** Playwright (vía `@playwright/mcp`).

## Estructura del proyecto

```
src/main/java/com/tuempresa/app/
├── tasks/          # Screenplay Tasks
├── questions/      # Screenplay Questions
├── ui/             # Screenplay UI (locators)
├── UI/             # ⚠️ legacy, distinta capitalización — ver advertencia abajo
├── hooks/          # Hooks nuevos
└── hooksDos/       # Hooks legacy

src/test/java/com/tuempresa/app/
└── stepdefinition/

src/test/resources/features/
├── AppLegacy1/
├── AppLegacy2/
├── ModuloPreventa/
├── ModuloAutoventa/
└── AppV2/
    ├── ModuloPreventaV2/
    ├── ModuloAutoventaV2/
    └── ModuloAliadosV2/
```

Dentro de cada app, las features están organizadas por número de carpeta (`01.Login`, `02.FuerzaDeVentas`, etc.).

> ⚠️ **Ejemplo real de "trampa silenciosa" de estructura:** en el proyecto original, existían dos paquetes `ui/` y `UI/` que solo se diferenciaban por mayúscula/minúscula. En Windows (sistema de archivos case-insensitive), esto genera confusión real. **Lección:** si tu proyecto tiene algo parecido (nombres casi idénticos, distinta capitalización, carpetas legacy vs nuevas), documéntalo aquí explícitamente — Claude no lo va a adivinar solo mirando los nombres.

> ⚠️ **Ejemplo real de duplicidad de patrones:** el proyecto original tenía dos paquetes de hooks (`hooks/` nuevo y `hooksDos/` legacy), sin ninguna nota de cuál usar para código nuevo. **Lección:** si tu proyecto tiene código "viejo" y "nuevo" conviviendo, dile a Claude explícitamente cuál seguir como referencia — de lo contrario puede mezclar patrones o replicar el legacy sin darse cuenta.

## Metodología de trabajo con Claude + MCP (validada en un proyecto real)

**No pedir exploración total del sistema de una sola vez.** El enfoque que funcionó fue incremental:

1. Explorar solo el login primero.
2. Listar los módulos disponibles, sin entrar todavía.
3. Entrar a un módulo específico y observar qué contiene.
4. Escribir el flujo end-to-end deseado, con pasos muy detallados y explícitos.
5. Dejar que Claude explore cada punto del flujo (inspección real vía MCP) y genere el código.
6. Generar las pruebas **una por una** — nunca en lote. Dar el contexto y los pasos de la siguiente prueba solo después de que la anterior quede terminada.

> Con este enfoque, un proyecto real pasó de una estimación de ~7 días escribiendo la suite a mano, a ~3 días reales con Claude+MCP.

## Decisiones ya tomadas / cosas que Claude debe saber de antemano

- Algunos **dropdowns tienen múltiples `<div>` anidados** por encima del input real — si un selector para escribir en un dropdown no funciona a la primera, re-inspeccionar la estructura real en vez de asumir el selector obvio.
- Algunos **selectores basados en índices son inestables** — preferir selectores más robustos (`id`, atributos específicos) cuando estén disponibles; si no hay alternativa, confirmar visualmente que el índice sigue siendo correcto tras cualquier cambio de la página.
- **Verificar visualmente el resultado de acciones críticas**, no asumir que "el clic se registró en el código" significa que el efecto visual esperado ocurrió — Claude puede "ver por debajo" el evento disparado sin que ocurra el cambio visual correspondiente. Esta fue la limitación más importante detectada en la práctica.

## Convenciones del equipo

*(Ejemplo — reemplaza con las convenciones reales de tu equipo)*

- **Tasks:** en inglés, con sufijo `Task` (`EnterCredentialsV2Task`, `OpenSalesForceV2Task`). En la suite V2, casi todo lleva además el sufijo `V2Task`.
- **Features/Scenarios:** Gherkin en **español** (`#language:es`), con `Característica:` y `Esquema del escenario:`.
- **Tags:** en inglés/CamelCase descriptivo (`@AppV2`, `@LoginFuerzaDeVentasV2`).
- **Variables en `Examples`:** camelCase en español (`actor`, `etiqueta`, `administrador`, `sufijoVendedor`).

## Credenciales y datos de prueba

- **Usuarios de prueba configurados** (nunca cuentas reales de producción — ver `seguridad-mcp-qa.md`):
  - AppV2: usuario dedicado + código de unidad de negocio de prueba.
  - AppLegacy2: usuario dedicado propio.

- ⚠️ **Ejemplo real de deuda técnica encontrada al construir este archivo:** en el proyecto original, las credenciales estaban **hardcodeadas directamente en el código Java** de los hooks, código que además estaba commiteado al repositorio. El `.gitignore` excluía `.env`, pero ese mecanismo no protegía nada porque no se estaba usando.

  **Instrucción recomendada para Claude:** no sugerir ni generar código nuevo que siga hardcodeando credenciales — si el proyecto tiene esta misma deuda, señalarla y proponer migrar a variables de entorno en vez de replicar el patrón inseguro.

  > 💡 Este hallazgo salió a la luz precisamente **al construir este `CLAUDE.md`** — revisar el código fuente real mientras se documenta el contexto, en vez de escribir solo de memoria, es una buena práctica en sí misma (ver `limitaciones-reales-mcp.md`, sección "Hallazgo colateral").

## Cómo correr los tests

```bash
# Correr todo (son los defaultTasks — clean test aggregate ya corre así, sin argumentos)
./gradlew

# Correr un subconjunto específico
./gradlew test --tests "paquete.Clase"
./gradlew aggregate   # necesario aparte: el aggregate NO corre solo tras un --tests filtrado
```

> ⚠️ Si se corre `--tests` filtrado sin el `aggregate` manual después, el reporte de Serenity queda desactualizado (muestra resultados de una corrida anterior, no de la actual).

## Reportes

Serenity BDD, formato `single-page-html`. El reporte queda en `target/site/serenity/` — abrir `index.html`.

---

## Cómo usar esta plantilla

1. Copia este archivo como `CLAUDE.md` en la raíz de tu proyecto.
2. Reemplaza cada sección con la información real de tu proyecto (nombres de apps, URL de staging, stack técnico, convenciones reales).
3. Mientras lo llenas, **revisa el código fuente real** (config, hooks, estructura de carpetas) en vez de escribir solo de memoria — es muy probable que encuentres algo que valga la pena documentar (como pasó con la deuda de credenciales de este ejemplo).
4. Trátalo como un documento vivo — actualízalo cuando el proyecto cambie, no lo escribas una sola vez y lo olvides.
