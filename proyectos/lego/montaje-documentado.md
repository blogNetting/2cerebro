---
title: Lego — los montajes reales que existen
created: 2026-09-28
updated: 2026-09-28
tags: [lego, montaje, repos, datos]
zona: tecnico
---

> **Reescrito el 2026-09-28.** La versión anterior de esta nota contenía **un script que yo había diseñado** y que nadie ha ejecutado nunca. Se ha sustituido por **los montajes que existen, con sus ficheros reales y sus enlaces**. Lo que quede marcado como *síntesis* es mío y va etiquetado como tal.

## El montaje mínimo que existe, con sus ficheros

**[fstandhartinger/ralph-wiggum](https://github.com/fstandhartinger/ralph-wiggum)** — MIT, 300★, último empujón mayo de 2026. **Es una implementación de Ralph Wiggum, la técnica de Huntley — una de las doce que existen, y no la de más estrellas.** Se elige por lo que trae, no por ser «la»:

| Fichero | Para qué |
|---|---|
| `scripts/ralph-loop-codex.sh` | El bucle |
| `scripts/lib/spec_queue.sh` | **La cola de especificaciones** |
| `scripts/lib/circuit_breaker.sh` | **Cortacircuitos** — para el bucle cuando algo se repite |
| `scripts/lib/nr_of_tries.sh` | **Contador de reintentos** |
| `scripts/lib/response_analyzer.sh` | Analiza la respuesta del agente |
| `scripts/lib/notifications.sh` | **Notificación** (Telegram) |
| `RALPH_PROMPT.md` | El prompt que se reinyecta |
| `.claude/commands/ralph-loop.md` | El comando |
| `codex-prompts/` | Prompts por especificación |
| `TELEGRAM_SETUP.md` | El montaje de los avisos |

**Y esto responde a una pregunta que yo había respondido de memoria:** el cortacircuitos, el contador de reintentos y la notificación **ya existen como piezas de un montaje real**, no son algo que haya que inventar.

**El original, y su forma mínima**, de [Geoffrey Huntley](https://ghuntley.com/ralph/): el bucle entero es una línea.

```bash
while :; do cat PROMPT.md | claude-code ; done
```

## Los otros montajes, con lo que trae cada uno

| Montaje | Qué trae | Licencia / estado |
|---|---|---|
| **[khgs2411/flow](https://github.com/khgs2411/flow)** | **Un solo script de bash de ~63 KB, sin dependencias**, 18 comandos, **todo el estado en `PLAN.md`** | MIT |
| **[blader/taskmaster](https://github.com/blader/taskmaster)** | Un **gancho de parada** que impide que el agente termine antes de acabar. Existe precisamente porque los agentes paran antes | MIT, 524★ |
| **[mikeyobrien/ralph-orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)** | Reimplementación del bucle en Rust | MIT, 3.160★, activo |
| **[covibes/zeroshot](https://github.com/covibes/zeroshot)** | Orquestación **ejecutor–verificador** en Rust | MIT, 1.915★ |
| **[mutable-state-inc/lean-collab](https://github.com/mutable-state-inc/lean-collab)** | Memoria compartida + **bloqueos de reclamación con caducidad** + retroceso | MIT, 73★ |
| **[Dicklesworthstone/claude_code_agent_farm](https://github.com/Dicklesworthstone/claude_code_agent_farm)** | 20+ agentes en paralelo, bloqueos, monitorización por tmux | 919★ |
| **[automazeio/ccpm](https://github.com/automazeio/ccpm)** | Skill + 16 scripts. **La cola son Issues de GitHub + worktrees**, no markdown | MIT, 8.391★ |
| **[gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done)** | Contexto fresco de 200k por tarea, verificadores paralelos, commits atómicos | **ARCHIVADO** con 64.442★ |

**El más simple y el más «copiar y usar» es `khgs2411/flow`**: un fichero, sin dependencias y sin instalación.

## Las plantillas de tarea, en proyectos reales

**[majiayu000/quotabar/specs/GH55/tasks.md](https://github.com/majiayu000/quotabar/blob/main/specs/GH55/tasks.md)** — el mejor ejemplo encontrado, y **no está en la documentación de ninguna herramienta**. Sus campos, y son **más precisos que los cinco que yo había derivado**:

| Campo | Para qué |
|---|---|
| **Delivery Contract** | Rama base, política de commits, alcance, compatibilidad |
| **`Owner:`** | **Qué agente la ejecuta** |
| **`Dependencies`** | De qué depende |
| **`Covers:`** | **Qué criterios de aceptación cubre** |
| **`Done when:`** | Cuándo está terminada |
| **`Verify:`** | **Cómo se comprueba** — separado de lo anterior |
| **Handoff** | El traspaso |
| Límites explícitos | Prohibido forzar el push, filtrar errores, degradar tests o salirse del alcance |

**La separación entre `Done when` y `Verify` es la aportación**: uno dice cuándo está acabada, el otro cómo se demuestra. Yo los tenía fundidos en un campo.

**Dos más, y dicen más que la plantilla original:**

- **[WeihanLi/dotnet-exec](https://github.com/WeihanLi/dotnet-exec/blob/main/.specify/templates/tasks-template.md)** y **[nutanix-cloud-native](https://github.com/nutanix-cloud-native/cluster-api-runtime-extensions-nutanix/tree/main/.specify/templates)**: la plantilla de Spec Kit **modificada por empresas reales**. dotnet-exec añade: *«los cambios de comportamiento en el análisis, la compilación o las interfaces públicas **requieren tareas de test**»*.
- **[microsoft/kalypso-scheduler](https://github.com/microsoft/kalypso-scheduler/tree/main/specs/001-bootstrapping-script)**: Spec Kit en un repositorio de Microsoft, **y sólo tiene `spec.md` y `checklists/`. No hay `tasks.md`.** El proceso se paró en la fase de especificación.

## Lo que el montaje real NO resuelve

Lo mismo que decía la versión anterior de esta nota, pero ahora con fuente:

- **El cortacircuitos y los reintentos existen en el montaje de `fstandhartinger`, pero nadie publica su tasa de éxito.** Ver [[implementaciones-reales]].
- **La cola con ficheros no está libre de carreras.** Lo dice la propia especificación de `TASKS.md`: *«esto es mejor esfuerzo, así que dos agentes leyendo a la vez **todavía pueden competir**»*.
- **Y el límite que ninguno cubre:** la mantenibilidad no tiene oráculo rápido. Ver [[limites-del-andamiaje]].

## Síntesis mía, etiquetada

Lo que **no** es dato y es mi lectura: con lo que existe, **el montaje mínimo razonable son tres piezas** —un agente, un script de bucle y git— más los cuatro ficheros de `ralph-wiggum` (cola, estado, progreso, reglas). **No hay que inventar nada: está construido y publicado.** Lo que no está resuelto es **cuánto aguanta sin ti**, y de eso va [[etapas]].

## Enlaces

- [[etapas]] — qué está maduro y qué necesita tu mano, etapa por etapa
- [[implementaciones-reales]] — los montajes y sus fracasos
- [[crear-la-tarea]] — el formato, y las plantillas reales
