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

**Minutos de Actions:** en repos privados consumen la cuota del plan de GitHub. Con GitHub Free, la cuota es limitada (verificar la cifra vigente en tu cuenta con `gh api user/settings/billing/actions` antes de operar a volumen); con Pro sube. **Sin verificar la cifra exacta**, a diferencia del bloqueo de rulesets, que sí se confirmó con la llamada real de arriba.

## 2. Configuración del repositorio (por proyecto)

> **Checklist previo, obligatorio antes de la primera ejecución real.** Nace de dos fallos reales encontrados el 2026-09-25 en la primera ejecución completa contra `blogNetting/prueba-flujo-agentes` — ninguno estaba en la documentación de gh-aw, los dos costaron una ejecución entera desperdiciada. Repetir esta lista en cada proyecto nuevo, no solo en el primero:
> 1. **Permiso de PR de Actions.** Por defecto GitHub lo tiene desactivado. Sin él, ninguna PR se crea sola aunque nada esté protegido: cae directo a issue de revisión. Se activa por API, sin tocar la interfaz: `gh api -X PUT repos/<owner>/<repo>/actions/permissions/workflow -f default_workflow_permissions=write -F can_approve_pull_request_reviews=true`.
> 2. **`network.allowed` del ejecutor.** `github` + la API del modelo **no bastan**: sin el identificador de ecosistema del stack del proyecto, el cortafuegos bloquea el registro de paquetes y el ejecutor no puede instalar nada ni correr un solo test de verdad — solo puede simular el resultado, que es justo lo que no queremos. Añadir el identificador que toque (`python`, `node`, `go`, `rust`, `java`, `dotnet`…) en `network.allowed` de `implementar.md` y `rehacer.md`, **no un dominio suelto a mano**: la lista completa está en [gh-aw/reference/network](https://github.github.com/gh-aw/reference/network/) y se actualiza con la propia herramienta, un dominio copiado a mano no.

1. **Etiquetas de estado:**
   ```
   for l in estado:bloqueado estado:listo estado:en-curso estado:en-revision estado:rehacer estado:humano agente:implementar agente:rehacer; do gh label create "$l" --force; done
   ```
2. **Secretos:**
   ```
   gh secret set DEEPSEEK_API_KEY
   gh secret set CLAUDE_CODE_OAUTH_TOKEN
   gh secret set GH_AW_WRITE_PROJECT_TOKEN   # PAT scope "project" — solo si se activa el panel de PO, capa-producto.md §3
   gh secret set CODECOV_TOKEN               # solo repos privados, gate de cobertura §8
   ```
   Se introducen por teclado; nunca van al repo. Los dos últimos, añadidos el 2026-09-25 tras una auditoría de coherencia: faltaban en esta lista y el sistema fallaría en vivo sin avisar hasta el primer intento real de escribir en Projects v2 o subir cobertura. Si además se activa `astillero-update.yml` ([[astillero-mantenimiento]] §3), hace falta un tercero: `gh secret set ASTILLERO_SYNC_TOKEN` (PAT scope `workflow` — el `GITHUB_TOKEN` por defecto no puede tocar `.github/workflows/`).
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
   
   > **Cerrado el 2026-09-25, probado en vivo, no leído en la documentación:** en un repo privado del plan actual (`blogNetting`, sin campo `plan` = Free), tanto `POST /rulesets` como la protección de rama clásica devuelven `403 — "Upgrade to GitHub Pro or make this repository public to enable this feature"`. **Sin GitHub Pro (o sin hacer el repo público), no hay rulesets, ni protección de rama, ni merge queue en privado.** Dos salidas: (a) pagar GitHub Pro — 4 $/mes en la fecha de esta comprobación, confirmar precio vigente antes de decidir; (b) hacer público el repo del proyecto real — no vale para código propietario. Se recomienda (a) si el proyecto es privado.

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
- ~~El plan de GitHub (rulesets, merge queue)~~ — **cerrado**: hace falta GitHub Pro para un repo privado, confirmado con una llamada real a la API (§2). Solo queda por confirmar la cuota exacta de minutos de Actions.
- La elección entre runners alojados o propio en la VM.
- ~~Cómo se replica este runbook a un segundo proyecto sin copiar y pegar~~ — **cerrado**: [[astillero-replicacion]]. Reusable workflows para revisor y reconciliador, imports remotos de gh-aw para el ejecutor, copier para labels/`CODEOWNERS`/`AGENTS.md`/contrato de tarea. Este runbook pasa de "7 pasos manuales por proyecto" a `copier copy gh:blogNetting/astillero . --vcs-ref v0.2.2` más los secretos propios del proyecto. Versionado real desde el 2026-09-26: todas las referencias se fijan a un tag (`astillero_ref`), no a `@main` — ver [[astillero-mantenimiento]] §2.

## Invocación: skill `/astillero-proyecto` (2026-09-26)

Este runbook deja de ejecutarse a mano paso a paso — hay un skill que lo automatiza y decide él mismo si el proyecto es nuevo o ya existe (aunque sea a medias).

- **Fichero:** `~/.claude/skills/astillero-proyecto/SKILL.md` (fuera del repo del wiki, en `~/.claude/skills/`, no en `2cerebro/.claude/commands/`) — deliberado: un comando de repo solo funciona abriendo Claude Code dentro de `2cerebro`; un skill en `~/.claude/skills/` funciona igual desde la sesión de cualquier proyecto (Patrimonial incluido), en cualquier directorio.
- **Invocación:** `/astillero-proyecto`, escrito en cualquier sesión de Claude Code de esta máquina — no importa el directorio de trabajo ni el repo abierto.
- **Qué hace:** localiza si el proyecto ya existe (aunque sea a medias) cruzando el wiki con `gh repo list` — si no lo encuentra así, pregunta el nombre directamente, no asume que es nuevo sin comprobar. Comprueba 6 piezas por separado (identidad, repo, nota hub, etiquetas de estado, ecosistema, bootstrap de copier — las 4 primeras imprescindibles, las 2 últimas con default razonable), y solo pregunta o genera lo que falta de verdad. Ruta local de un proyecto nuevo, siempre fija, nunca improvisada: `~/dev/<slug>/` — mismo patrón que ya usa Astillero consigo mismo. Nunca rellena secretos ni inventa la captura de idea — eso queda siempre para el humano.
- **Sin probar en vivo todavía**: escrito hoy, pendiente de la primera invocación real (candidata: Patrimonial, en su propia sesión, no en esta).

## Enlaces

- [[flujo-agentes-arquitectura]] — diseño
- [[flujo-agentes-informe]] — evidencia
- [[astillero-replicacion]] — mecanismo de replicación a cada proyecto
- [[astillero-mantenimiento]] — propagación de actualizaciones a proyectos ya en marcha
- [[astillero]] — proyecto
- [[_index]]
