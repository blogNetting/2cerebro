---
title: Flujo de desarrollo con agentes — arquitectura operable
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, arquitectura, github-actions, gh-aw, deepseek, opus, flujo]
zona: tecnico
---

El diseño del flujo completo: qué pieza cubre cada rol, qué se entregan entre ellas, cómo se reserva y se recupera el trabajo, cómo se revisa y cómo se cambia el modelo de cada rol. Cada conexión lleva la fuente que la sostiene. Evidencia en [[flujo-agentes-informe]]; puesta en marcha en [[flujo-agentes-runbook]].

**Leyenda:**
- ✔︎ = verificado por el orquestador en la documentación oficial de la pieza.
- 🧪 = no documentado de forma explícita; se confirma en la prueba de humo del runbook antes de operar.

## 1. Principios de diseño (y su origen)

| Principio | Por qué |
|---|---|
| **El tracker es la fuente de verdad**; un solo mecanismo reserva el trabajo | Patrón de Symphony (OpenAI) y de la práctica a escala ([[flujo-fase-a2-practica-a-escala]]) |
| **El traspaso es un contrato verificable**, no prosa | Deriva del plan; dotnet/runtime pasó del 38 % al 69 % por preparación ([[flujo-fase-a-practicas-reales]] P2) |
| **Primitivas nativas en lugar de una base de datos de tareas** | Beads falla en R13; GitHub Issues más Actions cubre R3 y R4 con piezas mantenidas por GitHub ([[flujo-fase-c1-spec-y-estado]]) |
| **Rondas acotadas y después, una persona** | Stripe: «at most two rounds of CI» ✔︎ |
| **Quien implementa no aprueba** | Ramp ✔︎; `CODEOWNERS` en rutas sensibles |
| **Empezar con 1–2 ejecutores** | [[flujo-fase-a-practicas-reales]] P1/P6 |
| **Cada rol tiene un modelo configurable en un único sitio** | Restricción del proyecto: quien ocupa cada rol es intercambiable |
| **No basta con pedirle al ejecutor «haz TDD»; hace falta un gate que lo aplique** | La instrucción sola falla la mayoría de las veces. En el estudio preregistrado más riguroso encontrado, solo el 41,9 % de las ejecuciones tuvo un test en rojo antes de implementar, y la condición «con instrucción de TDD» rindió **peor** en corrección que sin instrucción alguna ([Dan Luu](https://danluu.com/agentic-testing/) ✔︎, corroborado de forma independiente por [arXiv 2602.07900](https://arxiv.org/abs/2602.07900) sobre 6 modelos en SWE-bench Verified). Ver [[flujo-agentes-evidencia-empirica]] |
| **El wiki y el repo de cada proyecto no se pisan**: conocimiento reusable entre proyectos (y todo lo que conecta con vida no técnica) va al wiki; lo específico de un proyecto (entidades exactas, stack, contratos de tarea) va a `specs/` dentro de su propio repo, no se duplica en las dos partes | Corrección del usuario (2026-09-26) sobre [[patrimonial]]: mientras un proyecto está en entrevista de requisitos, sin framework spec-driven elegido, su detalle vive en la nota hub del wiki porque no hay `specs/` al que trasladarlo todavía. En el momento en que arranca el diseño formal (K1: Diseñador → repo), ese detalle se **traslada** a `specs/<proyecto>/spec.md` en el repo del proyecto, y la nota del wiki se resume a hub — no queda copiado en los dos sitios |

## 2. Roles y quién los ocupa

| Rol | Quién | Dónde corre | Modelo por defecto | Dónde se cambia el modelo |
|---|---|---|---|---|
| **Diseñador** | Opus 5.5 con la persona | Claude Code local, interactivo, suscripción Pro | `opus` | `/model` o `"model"` en `.claude/settings.json` ✔︎ |
| **Desglosador** (del diseño a issues) | El mismo, en la misma sesión | Claude Code local + `gh` | `opus` | Igual |
| **Aprobador del diseño** | Persona | PR de la especificación en GitHub | — | — |
| **Ejecutor** | Agente en GitHub Actions | Workflow gh-aw ✔︎ | DeepSeek (`deepseek-v4-pro` o `deepseek-flash`) a través del endpoint compatible con Anthropic ✔︎ | `engine.model` / `engine.env` del workflow (sección 7) |
| **Verificador determinista** | CI | GitHub Actions | — | — |
| **Revisor** | Agente en GitHub Actions | `claude-code-action` con el token de la suscripción (`claude_code_oauth_token`) ✔︎ | `opus` | `claude_args: --model …` ✔︎ |
| **Aprobador humano** | Persona | Revisión de la PR en rutas de `CODEOWNERS` | — | — |
| **Integrador** | GitHub | Merge queue y rulesets | — | — |
| **Reconciliador** | Workflow programado | GitHub Actions (cron) | Sin modelo (script con `gh`) | — |

## 3. Piezas y su papel

| Pieza | Para qué | Estado |
|---|---|---|
| **Framework spec-driven** (Spec Kit u OpenSpec; se elige en cada proyecto) | Idea → especificación, plan y tareas versionados en el repo | Maduro ([[flujo-fase-c1-spec-y-estado]] §2) |
| **GitHub Issues + sub-issues + dependencias** | Estado del trabajo: épica → tareas, `blocked-by`, historial, visibilidad | Nativo ✔︎ (`gh issue create --parent/--blocked-by`) |
| **Etiquetas de estado** | Máquina de estados visible (sección 5) | Nativo |
| **gh-aw** (GitHub Agentic Workflows) | Workflows de agente en markdown compilados a Actions: motor configurable, salidas seguras, bloqueo de issue | Oficial de GitHub ✔︎ |
| **claude-code-action** | Revisión con Opus dentro de Actions usando la suscripción | Oficial de Anthropic ✔︎ |
| **Grupos de concurrencia y `timeout-minutes` de Actions** | Reserva exclusiva por tarea y caducidad | Nativo ✔︎ |
| **Rulesets, required checks, merge queue y `CODEOWNERS`** | Puertas de integración y aprobación humana | Nativo |
| **Gates de CI** por stack | «No hay código sin tests» comprobado de verdad | [[desarrollo-agentes-f4-devsecops]] |
| **Reconciliador** (workflow con cron) | Recupera tareas abandonadas y promueve las desbloqueadas | Único pegamento propio (~50 líneas de `gh`) |

## 4. Contratos entre piezas

| # | De → a | Qué se entrega | Formato | Interfaz | Fuente |
|---|---|---|---|---|---|
| K1 | Diseñador → repo | Especificación, plan y tareas | Markdown del framework (p. ej. `specs/<feature>/spec.md`, `plan.md`, `tasks.md`) | git (PR de especificación) | [[flujo-fase-c1-spec-y-estado]] §2 |
| K2 | Persona → repo | Aprobación del diseño | Review aprobada de la PR de especificación | GitHub | Nativo |
| K3 | Desglosador → tracker | Una **épica** por *feature* y una **sub-issue por tarea**, cada una con el **contrato de tarea** (sección 6) y sus dependencias | Cuerpo de issue en markdown | `gh issue create --parent <épica> --blocked-by <n>` ✔︎ | `gh issue create --help` ✔︎ |

> **Este salto, verificado el 2026-09-25, no es una invención del proyecto: GitHub lo documenta con nombre propio.** El patrón `ResearchPlanAssignOps` de `github/gh-aw` — investigación → planificador crea issues → se asignan a un ejecutor → humano revisa y fusiona ✔︎ — es casi idéntico a K1–K5. Pero GitHub lo acota explícitamente a *«narrow, self-contained, well-scoped issues»* (su ejemplo real es limpieza de código duplicado, no features), y su **flujo recomendado tiene un humano en cada transición**; el atajo sin ese checkpoint (`assignees: copilot` directo, sin revisión) es la excepción documentada, no la norma. K3→K4 de este proyecto cae en ese atajo: hay aprobación humana del *spec* (K2), pero no de cada issue individual antes de marcarse `agente:implementar`.
>
> **Riesgo real, encontrado en el propio dogfood de GitHub, no en teoría:** su workflow `daily-spdd-spec-planner` crea issues con `assignees: copilot`, pero la asignación no siempre se materializa — issues reales (p. ej. [#61382](https://github.com/github/gh-aw/issues/61382), verificado) quedan con `assignees: []` y se **cierran solas a los 3 días como `NOT_PLANNED`, sin implementar**. Su propio `duplicate-code-detector` —el ejemplo que GitHub cita como caso real del atajo— lleva desde el 18 de septiembre de 2026 fallando y generando *«reported incomplete result»* en vez de trabajo real. El modo de fallo observado no es «la IA implementa mal un issue ambiguo» — es «el propio mecanismo de entrega falla en silencio». Es justo el fallo para el que este proyecto ya tiene respuesta (grupo de concurrencia, `timeout-minutes`, reconciliador cada 30 min con escalado a humano tras 2 intentos, §5 y §7.4) — cobertura que el propio ejemplo de referencia de GitHub no tiene: sus issues simplemente expiran.
| K4 | Tracker → ejecutor | Orden de ejecutar la tarea N | Etiqueta de un solo uso `agente:implementar` | Disparador `label_command` de gh-aw: «automatically removes that label so it can be re-applied» ✔︎ | [gh-aw triggers](https://github.github.com/gh-aw/reference/triggers/) ✔︎ |
| K5 | Ejecutor → repo | Rama con el código y los tests, y una PR que referencia la tarea | PR con `Closes #N`, prefijo `[agente]` y etiqueta `estado:en-revision` | Salida segura `create-pull-request` ✔︎ | [gh-aw safe-outputs](https://github.github.com/gh-aw/reference/safe-outputs/) ✔︎ |
| K6 | PR → CI | Ejecución de las puertas | Checks obligatorios | Evento `pull_request` y rulesets — **requiere GitHub Pro en repo privado, confirmado con la API real el 2026-09-25** (ver [[flujo-agentes-runbook]] §2) | Nativo |
| K7 | CI → revisor | PR con CI en verde | Evento | `claude-code-action` en `pull_request` o `workflow_run` | ✔︎ inputs `prompt`, `claude_args` |
| K8 | Revisor → PR | Veredicto contra el contrato | Review o comentario; si hay cambios, etiqueta `agente:rehacer` | GitHub | 🧪 que la review de la acción pueda aplicar una etiqueta |
| K9 | PR → ejecutor (rehacer) | Comentarios de la revisión | Etiqueta de un solo uso `agente:rehacer` | gh-aw `label_command` en `pull_request` + `push-to-pull-request-branch` ✔︎ | safe-outputs ✔︎ |
| K10 | PR aprobada → integración | Merge | Merge queue | Nativo | — |
| K11 | Merge → tracker | Cierre de la tarea | `Closes #N` cierra la issue | Nativo | — |
| K12 | Cierre → reconciliador | Promover lo que se ha desbloqueado | Etiquetas `estado:listo` y `agente:implementar` | Workflow con `issues: closed` + `gh issue view --json blockedBy` — **campo confirmado real y probado en vivo el 2026-09-25**: cerrar la tarea A promovió sola la tarea B | [[flujo-agentes-runbook]] §4 |

## 5. Máquina de estados de una tarea

Estados en etiquetas, visibles para cualquiera en GitHub:

```
estado:bloqueado ──(se cierran sus dependencias; reconciliador)──► estado:listo
estado:listo ──(+ agente:implementar, etiqueta de un solo uso)──► estado:en-curso
estado:en-curso ──(PR creada)──► estado:en-revision
estado:en-curso ──(timeout o fallo; reconciliador)──► estado:listo  [intento+1]
estado:en-revision ──(CI roja o el revisor pide cambios)──► estado:rehacer [ronda+1]
estado:rehacer ──(ronda ≤ 2: agente:rehacer)──► estado:en-revision
estado:rehacer / en-curso ──(ronda > 2 o intento > 2)──► estado:humano
estado:en-revision ──(CI verde + aprobación del revisor + CODEOWNERS)──► merge ──► issue cerrada
```

**Reserva (R3), en tres capas:**
1. **Concurrencia de Actions por tarea:** `concurrency: group: tarea-${{ issue.number }}` hace que haya «at most one running job or workflow in a concurrency group at any time» ✔︎.
2. **`lock-for-agent: true`** existe en gh-aw pero va bajo `on/issues`, no en la raíz — en esta versión del workflow se ha retirado por simplicidad (verificado al compilar, sección 7.1); se puede añadir combinándolo con `on.issues.types` si además del `label_command` se quiere bloquear la edición de la issue.
3. **Comprobación de idempotencia al arrancar:** si la issue ya tiene una PR abierta o `estado:en-curso`, la ejecución termina sin hacer nada. 🧪 implementable con `on.steps` de gh-aw; alternativa: la primera instrucción del prompt.
4. Endurecimiento opcional: crear la referencia `refs/heads/reserva/tarea-N` con `git push --force-with-lease=<ref>:`, que solo tiene éxito si la referencia no existe ✔︎ (documentación de git).

**Caducidad (R4):**
- `timeout-minutes` del job: «before GitHub automatically cancels it» ✔︎.
- El reconciliador, cada 30 minutos, busca issues en `estado:en-curso` sin ninguna ejecución viva (`gh run list`) y las devuelve a `estado:listo` o, tras dos intentos, a `estado:humano`.

**Paralelismo:** una tarea por grupo de concurrencia. El paralelismo total lo fija el número de runners, o un grupo global si se quiere un único ejecutor. Arranque recomendado: **1–2**.

## 6. Contrato de tarea (plantilla del cuerpo de cada sub-issue)

Es lo que evita la deriva del plan. Todo lo que el ejecutor necesita va dentro. Opus lo rellena al desglosar.

```markdown
## Objetivo
<una frase: qué comportamiento existirá cuando esto esté hecho>

## Contexto
- Especificación: specs/<feature>/spec.md §<n>
- Decisiones: docs/adr/<n>.md (si aplica)

## Alcance
- Ficheros o módulos que puede tocar: <lista>
- Fuera de alcance: <lista explícita>

## Interfaces
<firmas exactas: funciones, endpoints, tipos, con ejemplos entrada→salida>

## Criterios de aceptación (EARS)
1. Cuando <disparador>, el <sistema> deberá <respuesta>.  → test: tests/<fichero>::<test>
2. Si <condición no deseada>, entonces el <sistema> deberá <respuesta>.  → test: …

## Tests obligatorios
- Unitarios: <ficheros>. Integración: <ficheros>.
- Cada criterio tiene al menos un test que **falla sin la implementación**.
- No se modifican ni borran tests existentes (si hace falta, se para y se pide a una persona).

## Hecho cuando
- Todos los checks obligatorios en verde (sección 8).
- La PR referencia esta tarea con `Closes #<n>`.
```

## 7. Configuración de cada rol (borradores, no aplicados)

### 7.1 Ejecutor: workflow gh-aw `.github/workflows/implementar.md`

> **Compilado y verificado de verdad el 2026-09-25** con `gh aw compile` (extensión oficial `githubnext/gh-aw`, versión instalada v0.89.21) en un repo de prueba local, no publicado. El primer intento **falló**: `lock-for-agent` no va en la raíz de `on`, el propio compilador dice que va bajo `on/issues` o `on/issue_comment` — error de este documento, corregido abajo. Se retira `lock-for-agent` de esta versión (el grupo de concurrencia ya basta para la reserva exclusiva; ver nota tras el bloque). Con la corrección, compiló: `✓ implementar.md (129.9 KB), 1 succeeded, 2 warnings`.

```markdown
---
on:
  label_command:
    name: agente:implementar
    events: [issues]
concurrency:
  group: tarea-${{ github.event.issue.number }}
  cancel-in-progress: false
timeout-minutes: 60
engine:
  id: claude
  model: deepseek-v4-pro
  env:
    ANTHROPIC_BASE_URL: "https://api.deepseek.com/anthropic"
    ANTHROPIC_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
network:
  allowed: [github, api.deepseek.com, python]   # ecosistema del stack del proyecto; node/go/rust/etc. según toque
safe-outputs:
  create-pull-request:
    title-prefix: "[agente] "
    labels: [estado:en-revision]
  add-labels:
    allowed: [estado:*]
  remove-labels:
    allowed: [estado:*]
---
Implementa la tarea #${{ github.event.issue.number }} siguiendo EXACTAMENTE su contrato.
1. Si la tarea ya tiene PR abierta, termina sin hacer nada.
2. Lee AGENTS.md y los ficheros citados en «Contexto».
3. Escribe primero los tests de cada criterio y comprueba que FALLAN.
4. Implementa hasta que pasen. No toques tests existentes ni ficheros fuera de «Alcance».
5. Ejecuta los checks locales del proyecto.
6. Abre la PR con «Closes #N» y, en el cuerpo: modelo usado, enlace a esta ejecución y criterios cubiertos.
```

- **Endpoint y modelo, verificados en el fichero que de verdad ejecuta GitHub** (`implementar.lock.yml`, generado por el compilador, no escrito a mano): el paso final invoca `claude --print … --prompt-file …` con `env: { ANTHROPIC_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}, ANTHROPIC_BASE_URL: https://api.deepseek.com/anthropic }`. Así es como DeepSeek llega a consumir: cada ejecución del job hace peticiones HTTP reales a `api.deepseek.com`, autenticadas con tu clave, y DeepSeek las factura por token igual que cualquier llamada a su API.
- **Hallazgo no documentado antes: gh-aw ejecuta el agente dentro de un contenedor con cortafuegos propio** (`ghcr.io/github/gh-aw-firewall/agent`, `api-proxy` y `squid`, con SHA fijado en el propio `.lock.yml`). El `api-proxy` aplica `network.allowed` de verdad (no es solo una advertencia documental) y trae límites de presupuesto propios: `maxRuns`, `maxAiCredits`, `maxCacheMisses`. **Esto exige Docker en el runner** — trivial en runners alojados por GitHub, pero es un requisito real si se opta por un runner propio en la VM (ver §13).
- **Control de seguridad nuevo, no visto en la investigación previa:** al compilar con un secreto nuevo (`DEEPSEEK_API_KEY`), gh-aw exige aprobación explícita (`gh aw compile --approve`) antes de dejarlo pasar — control propio contra inyección de secretos no autorizados, coherente con el resto del diseño DevSecOps.
- **Ficheros protegidos.** `create-pull-request` incluye «Protected Files against supply chain attacks» y `allowed-files` ✔︎: se restringe a las rutas de código y de tests nuevos.
- **Corrección real, encontrada en la primera ejecución completa (2026-09-25), no en la documentación:** con `network.allowed` limitado a `github` y la API del modelo, el ejecutor no puede instalar nada del stack del proyecto (`pip`, `npm`, lo que sea) — el cortafuegos bloquea el registro de paquetes y el ejecutor no puede correr los tests de verdad, solo simular el resultado. **Regla general, no solo para Python:** `network.allowed` tiene que incluir el **identificador de ecosistema de gh-aw del stack del proyecto** (`python` = PyPI+conda+pythonhosted; `node` = npm/yarn/pnpm; también hay `go`, `rust`, `ruby`, `java`, `dotnet`, etc. — lista completa en [gh-aw/reference/network](https://github.github.com/gh-aw/reference/network/)), nunca dominios sueltos a mano. Es parte del checklist de puesta en marcha, [[flujo-agentes-runbook]] §2.

### 7.1 bis. Por qué el paso 3 del prompt («escribe el test y comprueba que falla») no basta por sí solo, y qué lo hace cumplirse de verdad

**Hallazgo crítico de la investigación (2026-09-25), no conocido al escribir la primera versión de este documento:** pedir «TDD» en el prompt no lo garantiza. En el estudio preregistrado más riguroso encontrado (160 ejecuciones por condición, tarea real, [danluu.com/agentic-testing](https://danluu.com/agentic-testing/) ✔︎), solo el 41,9 % de las ejecuciones con instrucción de TDD mostró un test en rojo antes de implementación sustancial, y **la condición con instrucción de TDD rindió peor en corrección que la condición sin ninguna instrucción**. Un paper académico independiente sobre 6 modelos en SWE-bench Verified llega a la misma dirección: los tests que escribe el agente funcionan como «mecanismo de *feedback* observacional», no como aserciones reales, y forzar más o menos escritura de tests por prompt apenas cambia el resultado ([arXiv 2602.07900](https://arxiv.org/abs/2602.07900) ✔︎). Detalle completo en [[flujo-agentes-evidencia-empirica]].

**Control real encontrado, no un simple check de git log:** [`nizos/tdd-guard`](https://github.com/nizos/tdd-guard) (2.352★, activo, verificado con `npm view tdd-guard` → v1.7.0 real en el registro). Es un *hook* `PreToolUse` de Claude Code que **intercepta cada `Write`/`Edit`/`MultiEdit` en tiempo real** y bloquea la acción si no hay un test en rojo que la justifique — no es una comprobación posterior, es una puerta antes de que la edición ocurra. Funciona con pytest, entre otros. Configuración verificada en la documentación oficial ([docs/installation.md](https://github.com/nizos/tdd-guard/blob/main/docs/installation.md) ✔︎):

```json
// .claude/settings.json (en el repo del proyecto, no en el runner)
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Write|Edit|MultiEdit|TodoWrite", "hooks": [{ "type": "command", "command": "tdd-guard" }] }
    ],
    "UserPromptSubmit": [{ "hooks": [{ "type": "command", "command": "tdd-guard" }] }],
    "SessionStart": [{ "matcher": "startup|resume|clear", "hooks": [{ "type": "command", "command": "tdd-guard" }] }]
  }
}
```

```bash
pip install tdd-guard-pytest   # reportero verificado en PyPI: https://pypi.org/project/tdd-guard-pytest
```

```toml
# pyproject.toml
[tool.pytest.ini_options]
tdd_guard_project_root = "/ruta/absoluta/del/repo"
```

Como es configuración estática (`settings.json`), se aplica igual en el ejecutor de gh-aw, sin sesión interactiva — `claude --print` la lee igual.

**Bypass conocido, documentado por el propio proyecto** ([docs/enforcement.md](https://github.com/nizos/tdd-guard/blob/main/docs/enforcement.md) ✔︎): un agente con permiso de ejecutar `Bash` sin restricción puede saltarse el hook editando ficheros con `echo`, `sed`, `awk` o `perl` en vez de las herramientas interceptadas. **Por eso no basta con instalarlo**: hace falta además denegar esos comandos en el `allowed-tools` del ejecutor (ya restringido por el propio `.lock.yml` de gh-aw, que lista las herramientas permitidas explícitamente — ver el fragmento de `--allowed-tools` verificado en la sección 7.1). Defensa en profundidad, no una sola barrera.

**Workflow `rehacer.md`:** el mismo motor, con disparador `label_command: agente:rehacer` en `pull_request` y salida `push-to-pull-request-branch` ✔︎. Toma los comentarios de la revisión como entrada.

### 7.2 Revisor: `.github/workflows/revisar.yml`

```yaml
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
jobs:
  revisar:
    if: github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: anthropics/claude-code-action@<SHA fijado>
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          claude_args: "--model opus --max-turns 20"
          prompt: |
            Revisa la PR contra el contrato de la tarea enlazada (Closes #N):
            - ¿Cada criterio EARS tiene un test que falla sin la implementación?
            - ¿Se han modificado o borrado tests existentes? ¿Se han tocado ficheros fuera de «Alcance»?
            - ¿La implementación cumple las interfaces exactas?
            Aprueba solo si todo se cumple. Si no, pide cambios concretos y numerados.
```

- **Token.** `CLAUDE_CODE_OAUTH_TOKEN` es «an OAuth token that authenticates with your Claude subscription, available on Pro…»; se genera con `claude setup-token` ✔︎.
- **Coste.** Opus revisa **solo con la CI en verde**, para no gastar cuota Pro en PRs que ya fallan.
- **Acción fijada por SHA**, por los compromisos de tj-actions y Trivy ([[desarrollo-agentes-investigacion]]).

### 7.3 Diseñador y desglosador (local)

- Claude Code con `opus` ✔︎, más el framework spec-driven del proyecto.
- El desglose crea issues con `gh issue create --title … --body-file contrato.md --parent <épica> --blocked-by <n>` ✔︎.
- A las tareas sin dependencias se les ponen las etiquetas `estado:listo` y `agente:implementar`; al resto, `estado:bloqueado`.

### 7.4 Reconciliador: `.github/workflows/reconciliar.yml`

Cron cada 30 minutos más el evento `issues: closed`. Script con `gh` que hace dos cosas:
1. Promueve a `estado:listo` + `agente:implementar` las issues `estado:bloqueado` cuyas dependencias están todas cerradas.
2. Devuelve a `estado:listo` las tareas `estado:en-curso` sin ninguna ejecución viva (`gh run list --workflow implementar --json`), sumando `intento:N`; con N > 2 pasan a `estado:humano`.

Es la única pieza propia del sistema. 🧪 los nombres de los campos JSON de `blockedBy`.

**Extensión para el panel del Product Owner** ([[capa-producto]] §3), corregida el 2026-09-25: **no vía `safe-outputs`** — `safe-outputs` es sintaxis exclusiva de workflows gh-aw compilados (`.md` → `.lock.yml`), y el reconciliador es un `.yml` de Actions normal, sin modelo de por medio; no hay juicio de IA que validar en un cron determinista, así que no tiene sentido convertirlo en workflow gh-aw solo para esto. La misma pasada del script llama directamente a `gh project item-edit` (refleja `estado:*` como campo `Status` en el tablero) y, semanalmente, `gh api graphql` para `create-project-status-update` (semáforo `ON_TRACK`/`AT_RISK`/`OFF_TRACK`/`COMPLETE`). Ambas llamadas usan el mismo PAT con scope `project` — secreto `GH_AW_WRITE_PROJECT_TOKEN` — porque el `GITHUB_TOKEN` por defecto no llega a Projects v2.

## 8. Puertas de CI (checks obligatorios en el ruleset de `main`)

Según el stack del proyecto ([[desarrollo-agentes-f4-devsecops]]):

| Puerta | Python | Node/TS |
|---|---|---|
| Tipos estrictos | mypy/pyright strict | `tsc --strict` |
| Tests y cobertura | pytest + coverage.py | Vitest, proveedor `v8` nativo (no `c8` aparte: desde Vitest 3.2 el proveedor `v8` integra el mismo motor, es el recomendado por defecto — [vitest.dev/guide/coverage](https://vitest.dev/guide/coverage.html) ✔︎) |
| **Cobertura del diff** | Codecov `patch` status, sin umbral global de proyecto — un número fijo alto es gameable con tests triviales (Fowler: *«high coverage numbers are too easy to reach with low quality testing»*, [martinfowler.com/bliki/TestCoverage.html](https://martinfowler.com/bliki/TestCoverage.html) ✔︎). `patch` cerca del 100 % (toda línea nueva de una tarea acotada debe estar cubierta), `project` informativo sin bloquear | Igual, mismo mecanismo |
| **Mutación del diff** | mutmut sobre los ficheros cambiados | Stryker incremental |
| **Tests intactos** | Script: si el diff modifica o borra tests existentes, falla salvo aprobación de `CODEOWNERS` | Igual |
| SAST | Semgrep CE u Opengrep, más Bandit | Semgrep CE u Opengrep |
| Secretos | gitleaks + push protection | Igual |
| Dependencias | OSV-Scanner + Dependabot | Igual |

**Reporting de cobertura: Codecov, no Coveralls** — Coveralls no tiene plan gratis para repos privados (mínimo 10 $/mes), y todo repo de Astillero es privado por decisión ya cerrada ([[decisiones]]); Codecov sí (plan Developer gratis, hasta 250 cargas/mes) — [about.codecov.io/pricing](https://about.codecov.io/pricing/) ✔︎ frente a [coveralls.io/pricing](https://coveralls.io/pricing) ✔︎. Se integra como `commit-status` nativo en el ruleset, igual que los demás gates de esta tabla — [docs.codecov.com/docs/commit-status](https://docs.codecov.com/docs/commit-status) ✔︎. **Aclaración explícita (auditoría 2026-09-25):** subir a `codecov.io` no necesita tocar `network.allowed` del ejecutor — esta tabla corre en el workflow de CI determinista, Actions normal, fuera del cortafuegos que solo envuelve `implementar.md`/`rehacer.md` (los workflows compilados por gh-aw, §7.1).

**`CODEOWNERS` con aprobación humana obligatoria:**
- `.github/**`
- `.claude/**`, `AGENTS.md`, `CLAUDE.md`
- los tests existentes
- los manifiestos de dependencias y los lockfiles
- auth, cripto y secretos
- `Dockerfile`, compose y `infra/`
- migraciones

## 9. Trazabilidad: quién hizo qué

| Pregunta | Dónde se ve |
|---|---|
| Qué agente o modelo escribió el código | Cuerpo de la PR (modelo y enlace a la ejecución, obligatorio por el prompt 🧪) + log de la ejecución de Actions + autor bot |
| Qué tarea lo originó | `Closes #N` y el historial de la issue |
| Quién revisó | Review de `claude-code-action` (modelo en `claude_args`) + reviews humanas |
| Quién aprobó el diseño | Review de la PR de especificación |
| Cuándo y cuántos intentos | Etiquetas `intento:N` y `ronda:N` + historial de ejecuciones |

**Hueco reconocido:** nadie ofrece una firma criptográfica del modelo autor ([[flujo-agentes-informe]] §7).

## 10. Fallos y recuperación

| Fallo | Qué pasa | Recuperación |
|---|---|---|
| El ejecutor se cuelga o entra en bucle | `timeout-minutes` cancela el job ✔︎ | El reconciliador la devuelve a `listo`; con más de 2 intentos, `humano` |
| Doble disparo de la misma tarea | El grupo de concurrencia deja una sola ejecución ✔︎; la segunda termina por idempotencia | — |
| CI roja | `estado:rehacer`, ronda + 1 | Máximo 2 rondas; después, `humano` |
| El revisor rechaza | Igual | Igual |
| El ejecutor toca ficheros protegidos | La salida segura lo bloquea ✔︎; `CODEOWNERS` exige aprobación | Persona |
| Cae la API de DeepSeek | Falla la ejecución | Reintento por el reconciliador; alternativa, cambiar de modelo (sección 11) |
| Se agota la cuota Pro | El revisor falla | La PR espera; se reintenta cuando se reinicia el límite |
| Conflicto de merge | La merge queue lo detecta | `agente:rehacer` con rebase |

## 11. Cambiar el modelo de un rol

| Rol | Cambio |
|---|---|
| Ejecutor → otro modelo de DeepSeek | `engine.model: deepseek-flash` |
| Ejecutor → Sonnet (API de Anthropic) | Quitar `ANTHROPIC_BASE_URL`, `ANTHROPIC_API_KEY: secrets.ANTHROPIC_API_KEY`, `model: claude-sonnet-5` |
| Ejecutor → modelo por el motor Codex | `engine.id: codex`, `OPENAI_BASE_URL`, `OPENAI_API_KEY` ✔︎. DeepSeek documenta la integración con Codex ✔︎ |
| Revisor | `claude_args: --model sonnet` o `--model fable` |
| Diseñador | `/model` en Claude Code |

La estructura del flujo no cambia en ningún caso.

## 12. Replicación a cada proyecto (Astillero)

Cómo se lleva este diseño a un repo nuevo (p. ej. [[patrimonial]]) y cómo llegan los cambios posteriores a los proyectos ya creados: [[astillero-replicacion]]. Cierra el punto 🧪 de §7.1 sobre includes remotos de gh-aw — **sí los admite** (`imports: owner/repo/path@ref`).

## 13. Alternativa sin ejecutar en Actions

Si se prefiere ejecutar en la VM:
- **CAO** (AWS Labs) con un perfil `provider: opencode_cli` y el modelo de DeepSeek ✔︎, sobre el mismo tracker, con las mismas etiquetas y contratos.
- **Coste:** hay que operar `tmux`, el servidor de CAO y la reserva con la referencia git (sección 5).
- **Riesgo:** fallos abiertos en su *dispatcher* ([[flujo-fase-c2-orquestacion-ejecucion-revision]] §2.2).

## 14. Capa de producto

Cómo el usuario dirige este motor como Product Owner de una sola persona (captura de la idea, backlog sin scoring formal, panel, bugs con el mismo contrato de tarea, versionado por checkpoint, registro de decisiones de producto): [[capa-producto]]. No es una pieza más del pipeline técnico, es quien lo dirige.

## 15. Despliegue a producción

Hasta aquí el pipeline termina en K11 (merge en `main`, issue cerrada). Esto es lo que pasa después — hueco real detectado en la auditoría del 2026-09-25, sin cubrir en ninguna nota anterior, cerrado hoy con evidencia real de operadores en solitario (40+ comentarios de dos hilos de HN, [33968378](https://news.ycombinator.com/item?id=33968378) y [43487843](https://news.ycombinator.com/item?id=43487843), más documentación oficial).

**CD automático en cada merge a `main`, sin checkpoint manual aparte.** Es el patrón dominante y sin excepción real encontrada entre 40+ operadores en solitario: *«To deploy, I just push to GitHub. A service on the server side rebuilds whenever it sees new commits»* (nicbou); Fly.io lo documenta como comportamiento por defecto de su propia action — [docs.fly.io/launch/continuous-deployment-with-github-actions](https://docs.fly.io/launch/continuous-deployment-with-github-actions/) ✔︎. No hace falta un segundo portón manual: los gates de §8 ya filtran antes del merge.

**Esto no contradice el versionado por checkpoint de [[capa-producto]] §5 — son dos ejes distintos.** `release-please` sigue generando el PR de changelog/tag que el usuario aprueba, pero eso no bloquea el despliegue: el CD corre en cada merge a `main` sea cual sea su origen. Si se quiere de verdad un checkpoint humano en el despliegue mismo (no en la versión), el mecanismo nativo es `environment: production` con *required reviewers* — soportado en repos privados con GitHub Pro (ya decidido para rulesets, sin coste adicional) — [docs.github.com/actions/reference/environments](https://docs.github.com/en/actions/reference/environments) ✔︎.

**Dónde corre la app** (evidencia real de operadores solos, no lista de mercado):

| Patrón | Cuándo | Evidencia |
|---|---|---|
| PaaS gestionado (Fly.io, Render, Railway) | Arrancar rápido, cero ops | apothegm: *«for 95% of products I recommend a Heroku-alike... outsource your infrastructure maintenance»*; caveat real: downtime documentado en Fly.io por certificado no renovado — [community.fly.io](https://community.fly.io/t/ssl-certificate-did-not-renew-automatically/4924) |
| PaaS autoalojado sobre VPS (Dokku, CapRover) | Experiencia Heroku sin ceder control ni coste de Heroku | dig1: *«Easy to set on your VPS... almost-zero-to-none management»*; Gys: 5-6 años sin incidentes |
| VPS + Docker Compose (+ Ansible) | Coste mínimo, cero lock-in — coherente con [[desarrollo-agentes-f3-git-cicd-infra]] §3.4 (Compose para &lt;5 servicios) | Mayoría de menciones en ambos hilos (Hetzner, DigitalOcean) |

Kubernetes gestionado aparece solo en minoría, ya descartado por exceso en [[desarrollo-agentes-f3-git-cicd-infra]] §2. **Sin staging permanente por defecto**: en 40+ comentarios reales nadie menciona un entorno de staging fijo — mismo argumento ya anotado para *preview environments* en [[desarrollo-agentes-f3-git-cicd-infra]] §3.4 (exceso probable a esta escala); si hace falta verificar antes de producción, el patrón real es un entorno efímero por PR, no uno permanente.

**Rollback: extiende §10 (Fallos y recuperación), no infraestructura nueva.** Revertir el commit en `main` dispara el mismo CD hacia atrás — *«Roll backs are easy, just revert»* (ransom1538, [HN 43487843](https://news.ycombinator.com/item?id=43487843)). Blue-green y feature flags no tienen evidencia de adopción real a esta escala — se descartan por ahora, no por principio.

**Migraciones de base de datos: regla dura, no opcional.** Patrón *parallel change / expand-contract* — [martinfowler.com/bliki/ParallelChange.html](https://martinfowler.com/bliki/ParallelChange.html) ✔︎. Como el ejecutor barato puede tocar el schema dentro de una tarea aislada, **expandir y contraer el schema son siempre dos tareas distintas del backlog**, nunca una — igual que cada PR es pequeño y reversible por separado (§6), aquí aplica lo mismo al schema.

**Hueco real detectado en auditoría (2026-09-25) y cerrado aquí: «cerrado» no es lo mismo que «desplegado».** El mecanismo de dependencias de §5 (`blockedBy`) solo sabe de «issue cerrada» (K11, al mergear) — con CD asíncrono y falible (§15 arriba), existe una ventana real donde la tarea de «contraer» se desbloquearía y hasta podría ejecutarse antes de que el despliegue de «expandir» esté confirmado en verde en producción. **Regla añadida:** la tarea de «expandir» no se etiqueta como dependencia satisfecha solo por `issue cerrada` — el reconciliador comprueba además el resultado real del job de CD asociado a ese merge (`gh run list` filtrado por el workflow de despliegue) antes de mover la tarea de «contraer» de `estado:bloqueado` a `estado:listo`. Mismo mecanismo que ya usa el reconciliador para caducidad (§5, «Caducidad»), un chequeo más en la misma pasada — sin pieza nueva de infraestructura.

Lo que pasa después del despliegue — monitorización, alertado, incidentes, backup, rotación de secretos, parcheo de dependencias — no es parte de este pipeline determinista, es la disciplina operativa del Product Owner: [[devops-minimo]].

## Enlaces

- [[flujo-agentes-informe]] — evidencia y descartes
- [[flujo-agentes-runbook]] — puesta en marcha y comprobación de coherencia
- [[astillero-replicacion]] — cómo se replica este diseño a cada proyecto y cómo se propagan los cambios
- [[capa-producto]] — cómo el usuario dirige este motor como Product Owner
- [[devops-minimo]] — mínimo operativo una vez la app está en producción
- [[astillero]] — proyecto
- [[_index]]
