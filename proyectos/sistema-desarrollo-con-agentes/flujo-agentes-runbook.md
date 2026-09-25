---
title: Flujo de desarrollo con agentes — runbook y comprobación de coherencia
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, runbook, github-actions, gh-aw, puesta-en-marcha]
zona: tecnico
---

Pasos para poner en marcha el flujo de [[flujo-agentes-arquitectura]] en cualquier proyecto, desde un repositorio vacío, más el recorrido completo de una tarea con cada conexión verificada. **Nada de esto se ha instalado ni ejecutado todavía.**

## 1. Requisitos previos (una vez)

| Qué | Para qué | Estado en esta VM |
|---|---|---|
| Cuenta de GitHub y repositorio (privado o público) | Forja, Actions, Issues | Cuenta `blogNetting` ✔︎ |
| `gh` autenticado | Crear issues, etiquetas y rulesets | Instalado y autenticado ✔︎ |
| Extensión gh-aw (`gh extension install github/gh-aw`) | Compilar los workflows de agente | Pendiente |
| Claude Code con la suscripción Pro | Diseñador y desglosador | En uso ✔︎ |
| `claude setup-token` | Token OAuth de la suscripción para el revisor en Actions ✔︎ | Pendiente |
| Clave de la API de DeepSeek con saldo | Ejecutor | Pendiente |
| Framework spec-driven (Spec Kit u OpenSpec) | Diseño, según el proyecto | Se elige en cada proyecto |
| Herramientas de CI del stack | Puertas | Se instalan en la CI, no en la VM |

**Runners:** los alojados por GitHub no necesitan instalar nada. Un runner propio en la VM es opcional y debe ser efímero.

**Minutos de Actions:** en repos privados consumen la cuota del plan de GitHub. 🧪 comprobar la cuota de tu plan antes de operar.

## 2. Configuración del repositorio (por proyecto)

1. **Etiquetas de estado:**
   ```
   for l in estado:bloqueado estado:listo estado:en-curso estado:en-revision estado:rehacer estado:humano agente:implementar agente:rehacer; do gh label create "$l" --force; done
   ```
2. **Secretos:**
   ```
   gh secret set DEEPSEEK_API_KEY
   gh secret set CLAUDE_CODE_OAUTH_TOKEN
   ```
   Se introducen por teclado; nunca van al repo.
3. **`AGENTS.md`**: índice de unas 100 líneas que apunta a `docs/`. Es el patrón de OpenAI ([[flujo-fase-a2-practica-a-escala]] fuente 1). Incluye cómo compilar, cómo testear, las convenciones y la regla «no tocar tests existentes».
4. **`CODEOWNERS`**: la lista de rutas de [[flujo-agentes-arquitectura]] §8, con tu usuario como dueño.
5. **Workflow de CI** con las puertas del stack. Todas las acciones de terceros **fijadas por SHA**.
6. **Workflows de agente:**
   - `implementar.md` y `rehacer.md`, que se compilan con `gh aw compile` y generan los `.lock.yml` que se commitean.
   - `revisar.yml`.
   - `reconciliar.yml`.
7. **Ruleset de `main`:**
   - PR obligatoria.
   - Checks obligatorios: CI y revisor.
   - Aprobación de `CODEOWNERS`.
   - Se descartan las aprobaciones antiguas cuando llegan commits nuevos.
   - Historial lineal.
   - Merge queue.
   
   🧪 Confirmar que el plan de GitHub permite rulesets y merge queue en un repositorio privado ([[flujo-fase-c2-orquestacion-ejecucion-revision]] §10).

## 3. Operación: de la idea al merge

1. **Diseño.** Sesión de Claude Code con `opus`, usando el framework spec-driven. Resultado: `specs/<feature>/` en una rama, más una PR de especificación.
2. **Aprobación del diseño.** Tú revisas y apruebas la PR de especificación.
3. **Desglose.**
   - Opus crea la épica y una sub-issue por tarea con el contrato de [[flujo-agentes-arquitectura]] §6, usando `--parent` y `--blocked-by`.
   - Las tareas sin dependencias reciben `estado:listo` y `agente:implementar`; el resto, `estado:bloqueado`.
4. **Ejecución.** La etiqueta dispara `implementar`. DeepSeek escribe primero los tests y luego el código, y abre una PR con `Closes #N`.
5. **CI.** Pasan las puertas o se marca `estado:rehacer`.
6. **Revisión.** Con la CI en verde, Opus revisa contra el contrato. Aprueba, o pide cambios y marca `agente:rehacer`.
7. **Aprobación humana**, solo si la PR toca rutas de `CODEOWNERS`.
8. **Integración.** La merge queue integra, se cierra la issue, y el reconciliador promueve lo que se ha desbloqueado.
9. **Escalado.** Lo que acaba en `estado:humano` es tuyo: lo revisas en GitHub y decides.

## 4. Comprobación de coherencia: recorrido de una tarea

Cada salto indica la pieza que lo hace y la evidencia de que esa conexión existe.

| # | Salto | Cómo | Evidencia | Estado |
|---|---|---|---|---|
| 1 | Idea → especificación en el repo | Claude Code + Spec Kit/OpenSpec | Ambos multiarnés y con artefactos markdown ([[flujo-fase-c1-spec-y-estado]] §2) | ✔︎ |
| 2 | Especificación → épica + sub-issues con dependencias | `gh issue create --parent/--blocked-by` | Flags en `gh issue create --help` (gh 2.101.0) | ✔︎ |
| 3 | Etiqueta → dispara el workflow del ejecutor | gh-aw `label_command` | [triggers](https://github.github.com/gh-aw/reference/triggers/): quita la etiqueta y la deja lista para reaplicar | ✔︎ |
| 4 | Una sola ejecución por tarea | `concurrency.group` por issue | [Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/control-workflow-concurrency): «at most one running» | ✔︎ |
| 5 | La issue no se modifica durante la ejecución | Opcional: `lock-for-agent: true` bajo `on.issues` (no en la raíz de `on`) — se probó en la raíz y el compilador lo rechazó; retirado de la versión que sí compila, ver [[flujo-agentes-arquitectura]] §7.1 | [triggers](https://github.github.com/gh-aw/reference/triggers/) | ✔︎ corregido y verificado con `gh aw compile` |
| 6 | Workflow → Claude Code habla con DeepSeek | `engine.env.ANTHROPIC_BASE_URL` | [gh-aw engines](https://github.github.com/gh-aw/reference/engines/) + [DeepSeek Claude Code](https://api-docs.deepseek.com/quick_start/agent_integrations/claude_code) | ✔︎ (🧪 la clave en `ANTHROPIC_API_KEY`) |
| 7 | La red del agente permite llegar a DeepSeek | `network.allowed` | [gh-aw engines](https://github.github.com/gh-aw/reference/engines/): «The target domain must also appear in network.allowed» | ✔︎ |
| 8 | Ejecutor → PR con restricciones de ficheros | `safe-outputs.create-pull-request` + `allowed-files` | [safe-outputs](https://github.github.com/gh-aw/reference/safe-outputs/) | ✔︎ |
| 9 | PR → CI con checks obligatorios | Ruleset | Documentación de rulesets | ✔︎ (🧪 plan) |
| 10 | CI verde → Opus revisa con la suscripción | `claude-code-action` + `claude_code_oauth_token` + `--model opus` | [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions) + `action.yml` | ✔︎ |
| 11 | Revisión → rehacer | Etiqueta `agente:rehacer` → gh-aw sobre la PR + `push-to-pull-request-branch` | [safe-outputs](https://github.github.com/gh-aw/reference/safe-outputs/) | ✔︎ (🧪 que la acción del revisor ponga la etiqueta) |
| 12 | Colgado → cancelación | `timeout-minutes` | [workflow-syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax): «automatically cancels it» | ✔︎ |
| 13 | Cancelado → vuelve a la cola | Reconciliador (`gh run list`, `gh issue edit`) | Comandos estándar de `gh` | ✔︎ (🧪 campos JSON) |
| 14 | Merge → cierre → desbloqueo | `Closes #N` + reconciliador con `issues: closed` | Nativo | ✔︎ (🧪 campo `blockedBy` en JSON) |
| 15 | Cambio de modelo de un rol | Una clave de configuración por rol | [[flujo-agentes-arquitectura]] §11 | ✔︎ |

**Resultado:** todas las conexiones existen en la documentación oficial de su pieza. Quedan **7 puntos 🧪** de detalle de configuración; ninguno es de concepto. Se cierran en la prueba de humo.

## 5. Prueba de humo (antes de operar de verdad)

En un repositorio de prueba:
1. Una épica con tres tareas triviales: A; B, bloqueada por A; y C, que intenta modificar un test existente.
2. Comprobar:
   - que A se ejecuta y se abre su PR;
   - que un doble etiquetado de A no lanza una segunda ejecución;
   - que al cancelar A a mano el reconciliador la devuelve a `estado:listo`;
   - que B solo se lanza cuando A se cierra;
   - que C la paran las puertas y termina en `estado:humano`;
   - que Opus revisa con tu suscripción;
   - que el cambio de modelo del ejecutor se refleja en el cuerpo de la PR.
3. Medir:
   - el coste de DeepSeek por tarea (panel de uso de DeepSeek);
   - el consumo de cuota de Opus por revisión (`/usage`);
   - los minutos de Actions.

## 6. Qué queda abierto

- 6 de los 7 puntos 🧪 de §4 (el compilado real del workflow del ejecutor ya cerró el punto 6: DeepSeek/ANTHROPIC_BASE_URL, confirmado en el `.lock.yml`, 2026-09-25).
- El umbral de la cobertura del diff y de mutación, que se fija por proyecto.
- El plan de GitHub (rulesets, merge queue y minutos en privado).
- La elección entre runners alojados o propio en la VM.

## Enlaces

- [[flujo-agentes-arquitectura]] — diseño
- [[flujo-agentes-informe]] — evidencia
- [[sistema-desarrollo-con-agentes]] — proyecto
- [[_index]]
