# Prompts reutilizables para MCP + Claude en QA

> Colección de prompts probados, organizados por capacidad. Reemplaza los valores entre `[corchetes]` y las URLs de ejemplo por las de tu proyecto.

## 📑 Índice
- [Exploración web y generación de suites](#exploración-web-y-generación-de-suites)
- [Estrategia por niveles: smoke, regresión core, regresión completa](#estrategia-por-niveles)
- [Auto-mantenimiento de tests rotos](#auto-mantenimiento-de-tests-rotos)
- [Análisis de reportes Serenity](#análisis-de-reportes-serenity)
- [Generación de datos de prueba](#generación-de-datos-de-prueba)
- [Auditoría de accesibilidad](#auditoría-de-accesibilidad)
- [Automatización de un módulo completo](#automatización-de-un-módulo-completo)
- [Diagramas de flujo de los tests generados](#diagramas-de-flujo-de-los-tests-generados)

---

## Exploración web y generación de suites

### Prompt maestro — exploración completa
```
Abre el navegador y ve a https://tu-sitio.com/login

Realiza una exploración completa del sitio:
1. Haz login con usuario "admin" y contraseña "admin123"
2. Navega cada sección del menú principal
3. Por cada sección identifica:
   - URL de la página
   - Elementos interactivos (botones, formularios, tablas, links)
   - Selectores reales de cada elemento clave
   - Flujos posibles (crear, editar, eliminar, buscar)

Con todo lo que encuentres, genera una suite completa de regresión:
- Un .feature por cada módulo encontrado
- StepDefinitions reutilizando pasos comunes
- Una Task de Serenity por cada acción principal
- Un PageObject/UIMap por cada pantalla
- Sigue exactamente el estilo de código de mi proyecto
```

### Estrategia por fases (para sitios grandes)

**Fase 1 — Mapeo (sin generar código todavía):**
```
Ve a https://tu-sitio.com/login, haz login
y dime únicamente qué módulos y secciones tiene el sitio.
No generes código aún, solo el mapa de navegación
```

**Fase 2 — Módulo por módulo:**
```
Ahora enfócate solo en el módulo [nombre que reportó].
Explora todos sus flujos y genera el feature, steps, task y pageobject
```

### Detectar flujos críticos primero
```
Navega el sitio completo como si fueras un usuario real.
Identifica los 5 flujos más críticos para el negocio
(los que si fallan impactan más) y priorízalos.
Luego genera los tests para el flujo #1
```

### Casos borde que no se te ocurren (QA adversarial)
```
Basándote en el flujo de [nombre], actúa como un QA adversarial:
identifica todos los casos borde, condiciones de error y
escenarios negativos que un usuario malicioso o descuidado
podría ejecutar. Genera scenarios para los 5 más críticos
```

📄 Ver también: [Capacidades de Claude + MCP en QA](../../../docs/web/capacidades-mcp-qa.md)

---

## Estrategia por niveles

### Smoke — rápido, lo más crítico
```
Genera 8 smoke tests para mi proyecto que cubran
los flujos sin los cuales el sistema no funciona para nada.
Etiquétalos con @smoke. Deben ejecutarse en menos de 3 minutos
```

### Regresión core — módulo por módulo
```
Para el módulo de [nombre], genera tests de regresión
cubriendo el flujo feliz y los 3 errores más comunes.
Etiquétalos con @regression
```

### Regresión completa — máxima cobertura
```
Genera la suite completa del módulo [nombre] con todos
los casos borde, Scenario Outlines con al menos 4 filas
de datos cada uno y casos de error de red/timeout.
Etiquétalos con @full-regression
```

📄 Ver también: [Estrategia de generación de suites: 3 niveles](../../../docs/web/generacion-suites-estrategia.md)

---

## Auto-mantenimiento de tests rotos

### Caso 1 — Un test específico falló
```
Mi test LoginTest está fallando con este error:
[pega el stacktrace completo aquí]

El selector que falla es: By.id("signin2")

Ve a https://tu-sitio.com/login, inspecciona
el formulario actual y dime qué cambió.
Luego actualiza mi LoginPageObject con el selector correcto
```

### Caso 2 — Varios tests rotos después de un deploy
```
Después del último deploy fallaron estos tests:
- LoginTest
- RegisterUserTest
- CheckoutTest

Lee mis archivos de PageObjects y StepDefinitions,
luego navega https://tu-sitio.com y compara
cada selector que uso contra lo que realmente existe hoy en el DOM.
Dame un reporte de qué cambió y repara todos los selectores rotos
```

### Caso 3 — Reparación preventiva antes de un deploy
```
Mañana hacen deploy con cambios en el módulo de usuarios.
Ve a https://tu-sitio.com/users en el ambiente de staging,
compara los selectores que uso en mi UserPageObject contra
lo que ves en el DOM actual.
¿Hay algo que vaya a romperse?
```
> Este es el más poderoso — detecta los problemas antes de que exploten en producción.

### Caso 4 — Reparación masiva con reporte
```
Actúa como un QA engineer haciendo análisis de impacto.

Lee todos mis PageObjects en src/test/java/pages/
Navega cada URL que encuentres en esos archivos
Compara selector por selector contra el DOM actual

Genera un reporte con:
- Selectores que siguen funcionando ✓
- Selectores que cambiaron y necesitan update ⚠️
- Selectores que desaparecieron completamente ✗
- Código corregido para cada caso ⚠️ y ✗
```

### 💡 Pro tip — el selector más robusto
```
Al actualizar los selectores, prioriza en este orden:
1. By.id() si existe un id estable
2. By.cssSelector() con atributo data-testid si existe
3. By.cssSelector() con clase semántica
4. Evita xpath y posiciones numéricas como //div[3]
```

📄 Ver también: [Auto-mantenimiento de tests rotos](../../../docs/web/auto-mantenimiento-flujo.md)

---

## Análisis de reportes Serenity

**Dónde viven los archivos que Claude va a leer:**
```
tu-proyecto/
└── target/
    └── site/
        └── serenity/
            ├── index.html             ← reporte visual
            ├── serenity-summary.json  ← resumen ejecutivo
            └── *.json                 ← un JSON por cada test
```

### Prompt 1 — Diagnóstico general de salud de la suite
```
Lee todos los archivos JSON en target/site/serenity/

Dame un diagnóstico completo:
- Cuántos tests pasan, fallan y están pendientes
- Porcentaje de éxito general y por módulo
- Los 5 tests que fallan con más frecuencia
- Los 5 tests más lentos (en segundos)
- Tendencia: ¿la suite está mejorando o empeorando?
```

### Prompt 2 — Detección de patrones de fallo
```
Analiza los JSON de reportes en target/site/serenity/

Quiero entender POR QUÉ fallan los tests:
- Agrupa los fallos por tipo de error (NoSuchElement, Timeout, AssertionError, etc.)
- ¿Hay tests que siempre fallan en el mismo step?
- ¿Hay fallos que solo ocurren en ciertos horarios o ambientes?
- ¿Qué módulo tiene más fallos: es un problema de la app o del test?

Muestra los resultados ordenados por impacto
```

### Prompt 3 — Optimización de velocidad
```
Lee los reportes de Serenity y encuentra los cuellos de botella:

- Lista los 10 steps más lentos con su tiempo promedio
- ¿Qué tests superan los 30 segundos de ejecución?
- ¿Hay waits o sleeps explícitos que podría reemplazar
  por esperas inteligentes?
- Sugiere cómo optimizar los 3 tests más lentos
```

### Prompt 4 — Reporte ejecutivo para el equipo
```
Con base en los reportes de target/site/serenity/,
genera un reporte ejecutivo en markdown con:

## Resumen de calidad — [fecha]
- Estado general (semáforo: 🟢 🟡 🔴)
- Métricas clave esta semana
- Top 3 problemas críticos
- Comparación con semana anterior si hay datos
- Acciones recomendadas con prioridad Alta/Media/Baja

El tono debe ser para un tech lead, no técnico al 100%
```

### Prompt 5 — El más poderoso: análisis histórico
```
Tengo reportes de las últimas 4 ejecuciones en:
- target/site/serenity-run1/
- target/site/serenity-run2/
- target/site/serenity-run3/
- target/site/serenity-run4/

Compáralos y dime:
- ¿Qué tests eran estables y se volvieron inestables?
- ¿Qué tests nuevos aparecieron o desaparecieron?
- ¿La cobertura aumentó o disminuyó?
- ¿Hay tests flaky (a veces pasan, a veces fallan)?
Dame los flaky tests como prioridad — son los más peligrosos
```

📄 Ver también: [Análisis de reportes Serenity con MCP](../../../docs/web/analisis-reportes-serenity.md)

---

## Generación de datos de prueba

### Prompt 1 — CSV de usuarios colombianos
```
Genera un archivo CSV con 50 usuarios de prueba colombianos realistas.

Columnas: nombre, apellido, email, telefono, cedula, ciudad, direccion, rol

Reglas:
- Teléfonos en formato colombiano (300, 310, 311, 312, 313, 314, 315, 316, 317, 318, 319)
- Cédulas de 8 a 10 dígitos
- Ciudades reales de Colombia (Bogotá, Medellín, Cali, Barranquilla, Cartagena...)
- Emails con dominios variados (gmail, hotmail, outlook)
- Roles: admin, vendedor, cliente, supervisor (distribuidos proporcionalmente)
- Sin duplicados en email ni cédula

Guarda el archivo en: src/test/resources/testdata/usuarios.csv
```

### Prompt 2 — Datos directamente en Scenario Outline
```
Tengo este Scenario Outline en mi proyecto:

[pega tu scenario outline aquí]

Genera 10 filas de datos realistas para la tabla Examples.
Incluye:
- 6 casos válidos con datos variados
- 2 casos con datos en los límites (máximo de caracteres, mínimo)
- 2 casos inválidos para validaciones negativas

Usa datos colombianos donde aplique
```

### Prompt 3 — Datos coherentes entre sí (el más poderoso)
```
Genera 20 conjuntos de datos de prueba para el flujo de
registro y compra de mi sistema.

Cada conjunto debe ser COHERENTE internamente:
- Usuario con nombre y apellido colombiano real
- Email generado desde su nombre (ej: juan.garcia@gmail.com)
- Dirección coherente con su ciudad
- Tarjeta de crédito con número que pase validación Luhn
- Fecha de expiración futura válida

Formato: JSON guardado en src/test/resources/testdata/usuarios_compra.json

Agrupa los datos en 3 categorías:
- "validos": 12 conjuntos que deben pasar
- "limite": 4 conjuntos en el borde de las validaciones
- "invalidos": 4 conjuntos que deben fallar con mensaje específico
```

### Prompt 4 — Datos para casos borde y validaciones
```
Para el formulario de registro de mi sistema, genera datos
que prueben todos los casos borde de validación:

Campos: nombre, email, contraseña, teléfono, fecha_nacimiento

Genera datos para:
- Campo vacío (cada campo por separado)
- Solo espacios en blanco
- Máximo de caracteres permitido + 1
- Caracteres especiales: <, >, ', ", ;, --
- SQL injection básico: ' OR '1'='1
- XSS básico: <script>alert('xss')</script>
- Email sin @, sin dominio, con doble @
- Teléfono con letras, con menos dígitos, con más dígitos
- Contraseña sin mayúscula, sin número, muy corta

Guarda en src/test/resources/testdata/casos_borde.csv
con columna "caso" describiendo qué valida cada fila
```

### Prompt 5 — Datos para pruebas de carga liviana
```
Necesito 200 usuarios únicos para una prueba de carga básica.

Requisitos:
- Sin duplicados absolutamente en ningún campo único
- Distribuidos en 4 roles: 50 admin, 50 vendedor, 60 cliente, 40 supervisor
- Contraseñas que cumplan política: mínimo 8 chars, 1 mayúscula, 1 número
- Exporta en 2 formatos:
  1. CSV: src/test/resources/testdata/carga_usuarios.csv
  2. SQL INSERT: src/test/resources/testdata/carga_usuarios.sql
     (tabla: usuarios, esquema: test_db)
```

### 💡 Pro tip — regenerar datos cuando caducan
```
Lee mi archivo src/test/resources/testdata/usuarios.csv

Detecta:
- Emails que parecen ya usados varias veces en los tests
- Fechas de expiración de tarjetas que ya vencieron
- Cualquier dato que parezca obsoleto

Genera una versión actualizada del archivo con datos frescos
manteniendo la misma estructura y cantidad de filas
```

📄 Ver también: [Generación de datos de prueba con MCP](../../../docs/web/generacion-datos-prueba.md)

---

## Auditoría de accesibilidad

### Prompt 1 — Auditoría completa de una página (WCAG 2.1 AA)
```
Ve a https://tu-sitio.com/login

Realiza una auditoría completa de accesibilidad WCAG 2.1 nivel AA:

CONTRASTE Y VISUAL:
- ¿Todos los textos tienen ratio de contraste mínimo 4.5:1?
- ¿Los elementos interactivos son visualmente distinguibles?
- ¿Hay información transmitida solo por color?

NAVEGACIÓN POR TECLADO:
- ¿Todos los elementos interactivos son alcanzables con Tab?
- ¿El orden de tabulación es lógico?
- ¿Hay un indicador visible de foco en cada elemento?

SEMÁNTICA HTML:
- ¿Los inputs tienen labels asociados correctamente?
- ¿Los botones tienen texto descriptivo o aria-label?
- ¿Las imágenes tienen atributo alt?
- ¿La página tiene landmark regions (header, main, nav, footer)?
- ¿Los headings tienen jerarquía correcta (h1 → h2 → h3)?

LECTORES DE PANTALLA:
- ¿Los formularios tienen fieldset y legend donde aplica?
- ¿Los mensajes de error están asociados al campo con aria-describedby?
- ¿Los elementos dinámicos usan aria-live para anunciar cambios?

Reporta cada issue con:
- Severidad: Crítico / Advertencia / Informativo
- Elemento afectado (selector CSS)
- Descripción del problema
- Cómo corregirlo
```

### Prompt 2 — Auditoría rápida de los issues más críticos
```
Ve a https://tu-sitio.com/login

Enfócate solo en los issues CRÍTICOS de accesibilidad —
los que harían el sitio inutilizable para alguien con:
- Ceguera (usuario de lector de pantalla)
- Baja visión (usuario de alto contraste o zoom)
- Discapacidad motriz (usuario solo de teclado)

Dame máximo 10 issues ordenados por impacto,
con el elemento exacto y cómo corregirlo
```

### Prompt 3 — Generar tests de accesibilidad en Serenity
```
Basándote en la auditoría de accesibilidad de
https://tu-sitio.com/login, genera tests de Serenity BDD que:

1. Verifiquen que todos los inputs tienen labels
2. Verifiquen que los botones tienen texto accesible
3. Verifiquen que los mensajes de error son anunciados
4. Verifiquen la navegación completa con solo Tab

Crea:
- Accessibility.feature con escenarios descriptivos
- AccessibilitySteps.java con las verificaciones
- Una clase AccessibilityTask de Screenplay

Usa las mismas anotaciones y estilo de mi proyecto
```

### Prompt 4 — Auditoría de todo el sitio por secciones
```
Haz login en https://tu-sitio.com con admin/admin123
y navega cada sección del menú principal.

Por cada página que encuentres:
- URL visitada
- Top 3 issues de accesibilidad más graves
- Elementos sin aria-label o con label vacío

Al final genera un ranking de páginas:
¿cuál tiene peor accesibilidad y debe corregirse primero?
```

### Prompt 5 — Reporte ejecutivo para el equipo
```
Realiza la auditoría de accesibilidad de
https://tu-sitio.com/login

Luego genera un reporte en markdown con este formato:

## Reporte de accesibilidad — [fecha]

### Resumen ejecutivo
- Puntuación general estimada: X/100
- Issues críticos: N
- Issues de advertencia: N
- Issues informativos: N

### Issues críticos (corregir antes del próximo release)
| Elemento | Problema | Corrección sugerida |
|----------|----------|---------------------|

### Issues de advertencia (corregir en próximo sprint)
...

### Código corregido
Para cada issue crítico, muestra el HTML actual vs el HTML corregido

### Próximos pasos recomendados
...

Guarda el reporte en: src/test/resources/reports/accessibility-report.md
```

📄 Ver también: [Auditoría de accesibilidad con MCP + Playwright](../../../docs/web/auditoria-accesibilidad.md)

---

## Automatización de un módulo completo

```
Ya tienes el login funcionando. Ahora necesito que automatices
un módulo completo del sistema.

PASO 1 — EXPLORACIÓN:
1. Abre el navegador y ve a https://tu-sitio.com/config
2. Haz login con las credenciales que ya conoces
3. Identifica TODOS los módulos del menú principal
4. Dime qué módulos encontraste antes de continuar

PASO 2 — ANÁLISIS DEL MÓDULO:
Una vez identificados los módulos, enfócate en el primero
o el más importante que encuentres y:
- Navega cada sección y subsección del módulo
- Identifica todas las acciones posibles:
  * ¿Tiene tabla de datos? ¿Con qué columnas?
  * ¿Tiene botón Crear/Nuevo?
  * ¿Tiene botón Editar?
  * ¿Tiene botón Eliminar?
  * ¿Tiene buscador o filtros?
  * ¿Tiene paginación?
  * ¿Tiene exportar o importar datos?
- Identifica todos los selectores reales del DOM

PASO 3 — DETERMINA LOS FLUJOS:
Con lo que encontraste, define los flujos a automatizar:
- Flujo feliz (CRUD completo si aplica)
- Flujos negativos (campos vacíos, datos inválidos, duplicados)
- Flujos de búsqueda y filtrado
- Flujos de paginación si existe
- Casos borde específicos del módulo

PASO 4 — GENERA LA SUITE COMPLETA:
Para cada flujo identificado crea:

tasks/[Modulo]/
- Crear[Entidad].java
- Editar[Entidad].java
- Eliminar[Entidad].java
- Buscar[Entidad].java

questions/[Modulo]/
- [Entidad]CreadaExitosamente.java
- [Entidad]EnLaLista.java
- MensajeDeError.java

pages/
- [Modulo]Page.java — todos los locators del módulo

features/
- [Modulo].feature con:
  * Scenario: Crear [entidad] exitosamente
  * Scenario: Crear [entidad] con datos inválidos
  * Scenario: Editar [entidad] exitosamente
  * Scenario: Eliminar [entidad] con confirmación
  * Scenario: Buscar [entidad] existente
  * Scenario: Buscar [entidad] inexistente
  * Scenario Outline: Validaciones de campos obligatorios
    con Examples de al menos 4 filas

stepdefinitions/
- [Modulo]StepDefinitions.java — steps específicos del módulo

REGLAS ESTRICTAS:
- Navega la página real, no inventes ningún selector
- Sigue exactamente el patrón Screenplay del proyecto
- Reutiliza los Tasks y Questions que ya existen (login, etc.)
- Usa actor.attemptsTo() en todos los Tasks
- Parametriza todos los datos con {string} en los steps
- Organiza los archivos en subcarpetas por módulo
- Agrega tag @modulonombre en cada scenario para poder
  ejecutarlos independientemente
```

---

## Diagramas de flujo de los tests generados
```
Genera un diagrama visual del flujo completo de cada test
que acabas de crear en el módulo de [nombre del módulo].

Para cada escenario, quiero ver:
- Cada paso del Gherkin en orden
- Qué Task ejecuta ese paso
- Qué selector o elemento de la página interactúa
- Qué Question verifica al final

Preséntalo como un diagrama de flujo (puede ser en formato
Mermaid o como diagrama de texto estructurado) que muestre
visualmente la secuencia:

Given → When → When → Then

con cada caja indicando qué hace exactamente en el código
```
