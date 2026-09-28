---
title: Lego — el panorama completo de herramientas
created: 2026-09-28
updated: 2026-09-28
tags: [lego, herramientas, panorama]
zona: tecnico
---

Barrido sistemático, abriendo los índices enteros en vez de citar el top-3. El resultado incómodo: **son más de treinta frameworks de especificación**, y la mayoría no aporta nada que no esté ya en las tres piezas.

## Capa 1 — El agente (el runtime)

Es la pieza que ejecuta. Verificados en vivo el 2026-09-28 con la API de GitHub:

| Herramienta | Licencia | Estrellas | Último empujón | Piezas |
|---|---|---|---|---|
| **Claude Code** | Cerrado, plan de pago | — | continuo | CLI + cuenta |
| **[OpenCode](https://github.com/sst/opencode)** | MIT | 210.541 | hoy | CLI + clave |
| **[Codex CLI](https://github.com/openai/codex)** | Apache-2.0 | 126.918 | hoy | CLI + cuenta + `bubblewrap` |
| **[Pi](https://github.com/earendil-works/pi)** | MIT | 110.007 | hoy | CLI + modelo |
| **[Cline](https://github.com/cline/cline)** | Apache-2.0 | 69.477 | activo | extensión de editor |
| **[Aider](https://github.com/Aider-AI/aider)** | Apache-2.0 | 49.232 | **22-may-2026** | uv + Python + CLI + clave |
| **[Continue](https://github.com/continuedev/continue)** | Apache-2.0 | 36.052 | activo | extensión + ficheros |

**Aider es el único con el mantenimiento roto**, confirmado por la fecha del último empujón.

## Capa 2 — El formato de la tarea

Aquí es donde está el ruido, y donde conviene ser claro: **no hay un estándar, hay una propuesta y un montón de variantes.**

### La propuesta más cercana a un estándar

**[`TASKS.md`](https://github.com/tasksmd/tasks.md)** — «una especificación ligera para colas de tareas de agentes — el compañero de `AGENTS.md`». Su división del trabajo es exactamente la correcta:

> «AGENTS.md dice a los agentes **cómo** trabajar. TASKS.md les dice **en qué** trabajar.»

Sus campos: prioridad P0–P3, casillas markdown, e **ID, Tags, Details, Files, Acceptance, Plan, Blocked by, Blocked, Parent, Research, Last-enriched, Estimate, Verification, Risk**, más cinco de pre-registro de hipótesis.

**Coincide casi exactamente con los cinco campos que yo había derivado por separado** (objetivo, alcance, criterios, comprobación, retorno) — con `Files` como alcance, `Acceptance` como criterios y `Verification` como la comprobación. **Convergencia independiente.**

**Pero la cautela es obligatoria: tiene 11 estrellas y 3 bifurcaciones.** Es un proyecto personal de Fyodor Ivanischev, no un estándar adoptado. Lo honesto es decir: dos caminos independientes llegaron a la misma forma de fichero, y ninguno de los dos tiene adopción masiva.

Y su propia limitación declarada refuerza lo que dice [[robustez-desatendida]]: «esto es **mejor esfuerzo**, así que dos agentes leyendo a la vez **todavía pueden competir**» — el bloqueo entre agentes concurrentes no está resuelto.

### Los frameworks que imponen un formato

| Framework | Estrellas | Formato | Piezas |
|---|---|---|---|
| **[GitHub Spec Kit](https://github.com/github/spec-kit)** | >115.000 | `constitution.md` + `spec.md` + `plan.md` + `tasks.md`; requisitos numerados `FR-001`; marcadores `[NEEDS CLARIFICATION]` | Python 3.11 + uv + CLI + agente |
| **[BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD)** | ~49.500 | 12+ agentes con nombre propio y 34+ flujos | alto |
| **[OpenSpec](https://github.com/Fission-AI/OpenSpec)** | ~56.000 | `changes/` con `proposal.md`, `specs/`, `design.md`, `tasks.md`; deltas AÑADIDO/MODIFICADO/ELIMINADO | Node 20 + CLI |
| **[Kiro](https://kiro.dev)** (AWS) | ~3.700 | `requirements.md` en notación EARS + `design.md` + `tasks.md` | IDE propietario; gratis 50 créditos/mes, Pro 20 $/mes |
| **[Task Master](https://github.com/eyaltoledano/claude-task-master)** | ~27.700 | `tasks.json` desde un PRD en texto | Node; MIT + Commons Clause |
| **[spec-workflow-mcp](https://github.com/Pimzino/spec-workflow-mcp)** | ~4.200 | `.spec-workflow/` con requisitos, diseño, tareas y registro de aprobaciones | GPL-3.0 |
| **[ai-dev-tasks](https://github.com/snarktank/ai-dev-tasks)** | ~7.800 | dos ficheros markdown de prompt | **ninguna. No instala nada** |
| **[Tessl](https://tessl.io)** | — | registro de **10.000+ especificaciones** de librerías externas | cuenta; en beta |
| **[Cursor Plan Mode](https://cursor.com)** | — | plan revisable dentro del editor | **cero** |
| **[GSD](https://github.com/gsd-build/get-shit-done)** | 48.000–61.000 | ver abajo | ocho ficheros de estado, siete roles |

**Y así hasta más de treinta**, según el recuento de [SoftwareSeni](https://www.softwareseni.com/the-30-plus-framework-landscape-navigating-spec-driven-development-options-in-2026/), que además dice que de esos treinta **sólo seis merecen evaluación seria** y que el resto son «un dolor de cabeza genuino».

**La recomendación más útil de todo ese catálogo**, y coincide con la conclusión del informe: la configuración mínima que proponen es «`AGENTS.md` con `ai-dev-tasks`: gratis, portable, **y no instala nada**».

## Lo que faltaba no eran herramientas: eran categorías

Un barrido sobre ~640 entradas (el 100% de las listas curadas y los leaderboards por estrellas, ~9% de un universo de ~7.400 repos) encontró **cinco categorías enteras** que no estaban en el mapa:

| Categoría | La pieza que la representa | Estado |
|---|---|---|
| **Protocolos de transporte de tarea** | **[ACP, Agent Client Protocol](https://github.com/agentclientprotocol/agent-client-protocol)** — «un protocolo para conectar cualquier editor con cualquier agente». Adoptado por **JetBrains** en diciembre de 2025 | 4.347★, activo hoy |
| **Gestor de dependencias de configuración de agente** | **[APM](https://github.com/microsoft/apm)** de Microsoft — «piensa en `package.json`… pero para la configuración de agentes» | 3.914★ |
| **Rastreador de incidencias con grafo de dependencias para agentes** | **[Beads](https://github.com/gastownhall/beads)** — «proporciona memoria persistente y estructurada para agentes de código. **Reemplaza los planes markdown desordenados por un grafo con dependencias**» | **27.485★**, activo hoy, 1.282 incidencias |
| **Gestor de espacio de trabajo multiagente** | **[Gastown](https://github.com/gastownhall/gastown)** — 18.207★ y las discusiones de comunidad **más ruidosas del sector** (403, 354 y 219 puntos en HN), incluida una controversia abierta sobre si consume créditos del usuario | 18.207★ |
| **Lenguaje de especificación formal con validador** | **[Allium](https://github.com/juxt/allium)** — «el lenguaje de especificación que te contesta»: comprueba las specs, detecta callejones sin salida y **genera los tests a partir de los comportamientos formales** | 499★ |

**Y un hallazgo negativo que importa:** el artículo que prometía enumerar «más de 30 frameworks» **no enumera 30** — nombra once y discute siete. La lista real de **~56** está en [`Engineering4AI/awesome-spec-driven-development`](https://github.com/Engineering4AI/awesome-spec-driven-development), y [`awesome-openspec`](https://github.com/wearetechnative/awesome-openspec) lista **112 entradas de las que ~60 son variantes del mismo OpenSpec**. Eso no es un ecosistema rico: **es saturación de derivados.**

**Dos trampas detectadas en las propias listas:**

- **Estrellas ≠ uso, demostrado en vivo:** **OpenAI Symphony** tiene **27.455★** y su README lo llama «vista previa de ingeniería discreta para entornos de confianza». Su entrada más votada en Hacker News tiene **2 puntos**. Es el caso más claro de la regla: el dato de existencia y el de adopción se contradicen abiertamente.
- **Las listas curadas de este sector tienen enlaces caducados de forma sistemática** (dan por bueno `Aider` en una URL muerta y `OpenCode` en un repositorio que no es). Conviene verificar cada URL contra la API de GitHub, que es lo que se ha hecho aquí.

**Y una categoría que se declara vacía, con cautela:** no apareció **ninguna herramienta seria para escribir una tarea en lenguaje natural fuera del ámbito del código** y que un sistema de agentes la ejecute con verificación. Lo más cercano son frameworks genéricos sin concepto de criterio de aceptación. *Cautela: podría ser un fallo de vocabulario en la búsqueda, no un hecho sobre el mundo — no se sostiene como hallazgo firme.*

## Capa 3 — La orquestación

Ver [[piezas-y-coste]] para el recuento. Lo nuevo aquí:

- **[GSD (Get Shit Done)](https://mintlify.wiki/gsd-build/get-shit-done/agents/overview)** — es el hallazgo grande. Convierte el patrón Ralph en producto con **contexto fresco de 200.000 tokens por tarea** («la tarea 50 tiene la misma calidad que la 1»), investigadores, planificadores, ejecutores y verificadores en paralelo, **verificación hacia atrás desde el objetivo**, detectores de código hueco, **commits atómicos** (uno por tarea, lo que permite `git bisect` para localizar la tarea que rompió algo) y un canal de seguridad con escaneo de inyección de prompts y de secretos.
  - **Roles:** orquestador (que «nunca hace trabajo pesado»), planificador, ejecutor, verificador, depurador, comprobador de planes y cuatro investigadores en paralelo.
  - **Ficheros:** `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, `CONTEXT.md`, `PLAN.md`, `SUMMARY.md`, `RESEARCH.md`.
  - **Su mecanismo de retorno estructurado** coincide con lo que recomienda [[robustez-desatendida]]: el agente devuelve `PLANNING COMPLETE`, `CHECKPOINT REACHED` o `PLANNING BLOCKED`, y ante desbordamiento de contexto devuelve un punto de continuación y el orquestador lanza una instancia nueva.
  - **El aviso:** una evaluación independiente lo puntúa **2/5**, señalando más de un 90% de solapamiento conceptual con patrones ya existentes y **afirmaciones de marketing sin verificar** («usado por Amazon, Google, Shopify y Webflow», sin prueba).

**Lo que GSD demuestra:** el patrón que recomiendo se puede industrializar. **Lo que cuesta:** ocho ficheros de estado y siete roles, frente a mis tres piezas y dos ficheros.

## Capa 4 — La verificación

- **[exspec](https://github.com/mnapoli/exspec)** — especificaciones en texto plano ejecutadas en un navegador real por una IA, sin código pegamento. Es la respuesta al caso web.
- **[quinny](https://socket.dev/pypi/package/quinny/overview/0.2.1)** — compila criterios de aceptación a pytest o node:test.
- **[archiet-microcodegen](https://pypi.org/project/archiet-microcodegen/)** — el compilador determinista: PRD entra, aplicación FastAPI sale, cero LLM. **0 y 4 estrellas: nadie lo usa.** Ver [[clausura-semantica]] para el porqué.

## Considerado y descartado en este barrido

| Descartado | Motivo |
|---|---|
| **El resto de los 30+ frameworks** | No aportan mecanismo nuevo: reempaquetan los mismos cuatro ficheros. Su valor diferencial es la ceremonia |
| **`archiet-microcodegen` como opción** | Idea limpia, **adopción cero** (0 y 4 estrellas). Sólo cubre lo plantillado |
| **Tessl como herramienta de tarea** | Su parte abierta es un registro de dependencias, no un sistema para escribir tus tareas. El framework propio sigue en beta cerrada |
| **Task Master** | Su propia crítica documentada: «PRD vago entra, tareas vagas salen» — no arregla el problema de la calidad de la especificación |
| **`TASKS.md` como estándar** | Es una **propuesta** con 11 estrellas. Se cita por la convergencia de formato, no como autoridad |
| **Los frameworks como categoría** | Treinta opciones para algo que cabe en un fichero markdown es, en palabras del propio catálogo, «un dolor de cabeza genuino». Y **ninguno tiene medición de eficacia**, salvo el benchmark de coste por funcionalidad |

## Lo que sigue faltando, y lo digo

**La medición de la categoría.** Existe **un solo** benchmark público de coste —[RanTheBuilder](https://ranthebuilder.cloud/blog/i-tested-three-spec-driven-ai-tools-here-s-my-honest-take/), ~75–200 $ por funcionalidad— y **ninguno mide eficacia**. Nadie ha publicado: «con este framework se entregaron N funcionalidades con M defectos, contra estas otras con este otro framework». Y su propio autor avisa de que las diferencias entre los tres que probó son «ruido».

**Nadie ha medido tampoco si GSD —u otro orquestador— bate al bucle simple.** Hay evaluaciones de calidad de código, y hay una puntuación de 2/5, pero **ninguna comparación a ciegas de resultado final**.

## Dónde se ha buscado

El catálogo de [más de 30 frameworks](https://www.softwareseni.com/the-30-plus-framework-landscape-navigating-spec-driven-development-options-in-2026/) enumerado entero · la [lista de 9 plantillas](https://securityboulevard.com/2026/06/9-prd-and-spec-templates-built-for-ai-coding-agents/) enumerada entera · `gh api` para estrellas, licencia y fecha del último empujón de cada repositorio · documentación primaria de Spec Kit, OpenSpec, Kiro, GSD, `TASKS.md` y archiet-microcodegen · Reddit por la API JSON y DuckDuckGo por el navegador real, **porque el presupuesto de búsqueda de la sesión se agotó (200/200)**.

**Sin aportar nada:** los listados agregadores de «mejores herramientas», que repiten los mismos nombres sin datos.

## Enlaces

- [[piezas-y-coste]] — el recuento de piezas y qué se rompe
- [[crear-la-tarea]] — el formato, y su convergencia con `TASKS.md`
- [[clausura-semantica]] — por qué el compilador determinista no se usa
- [[investigacion-lego]] — el informe completo
