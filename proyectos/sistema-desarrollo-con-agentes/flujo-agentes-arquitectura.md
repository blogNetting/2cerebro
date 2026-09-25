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
| K4 | Tracker → ejecutor | Orden de ejecutar la tarea N | Etiqueta de un solo uso `agente:implementar` | Disparador `label_command` de gh-aw: «automatically removes that label so it can be re-applied» ✔︎ | [gh-aw triggers](https://github.github.com/gh-aw/reference/triggers/) ✔︎ |
| K5 | Ejecutor → repo | Rama con el código y los tests, y una PR que referencia la tarea | PR con `Closes #N`, prefijo `[agente]` y etiqueta `estado:en-revision` | Salida segura `create-pull-request` ✔︎ | [gh-aw safe-outputs](https://github.github.com/gh-aw/reference/safe-outputs/) ✔︎ |
| K6 | PR → CI | Ejecución de las puertas | Checks obligatorios | Evento `pull_request` y rulesets | Nativo |
| K7 | CI → revisor | PR con CI en verde | Evento | `claude-code-action` en `pull_request` o `workflow_run` | ✔︎ inputs `prompt`, `claude_args` |
| K8 | Revisor → PR | Veredicto contra el contrato | Review o comentario; si hay cambios, etiqueta `agente:rehacer` | GitHub | 🧪 que la review de la acción pueda aplicar una etiqueta |
| K9 | PR → ejecutor (rehacer) | Comentarios de la revisión | Etiqueta de un solo uso `agente:rehacer` | gh-aw `label_command` en `pull_request` + `push-to-pull-request-branch` ✔︎ | safe-outputs ✔︎ |
| K10 | PR aprobada → integración | Merge | Merge queue | Nativo | — |
| K11 | Merge → tracker | Cierre de la tarea | `Closes #N` cierra la issue | Nativo | — |
| K12 | Cierre → reconciliador | Promover lo que se ha desbloqueado | Etiquetas `estado:listo` y `agente:implementar` | Workflow con `issues: closed` + `gh issue view --json` 🧪 (nombre exacto del campo `blockedBy`) | [[flujo-fase-c1-spec-y-estado]] §5 |

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
2. **`lock-for-agent: true`** de gh-aw bloquea la issue durante la ejecución ✔︎.
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

Todas las claves están tomadas de la documentación de gh-aw ✔︎ (engines, triggers, safe-outputs, frontmatter). Los valores concretos se validan con `gh aw compile` en la prueba de humo 🧪.

```markdown
---
on:
  label_command:
    name: agente:implementar
    events: [issues]
  lock-for-agent: true
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
  allowed: [github.com, api.deepseek.com]
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

- **Endpoint y modelo.** DeepSeek documenta `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic` para Claude Code ✔︎. gh-aw pasa el modelo al proveedor «verbatim» cuando `ANTHROPIC_BASE_URL` está fijado ✔︎.
- **Clave.** DeepSeek admite la cabecera `x-api-key` («Fully Supported») ✔︎, así que su clave va en `ANTHROPIC_API_KEY` 🧪.
- **Ficheros protegidos.** `create-pull-request` incluye «Protected Files against supply chain attacks» y `allowed-files` ✔︎: se restringe a las rutas de código y de tests nuevos.

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

## 8. Puertas de CI (checks obligatorios en el ruleset de `main`)

Según el stack del proyecto ([[desarrollo-agentes-f4-devsecops]]):

| Puerta | Python | Node/TS |
|---|---|---|
| Tipos estrictos | mypy/pyright strict | `tsc --strict` |
| Tests y cobertura | pytest + coverage | vitest + c8 |
| **Cobertura del diff** | diff-cover (umbral por proyecto) | diff-cover sobre lcov |
| **Mutación del diff** | mutmut sobre los ficheros cambiados | Stryker incremental |
| **Tests intactos** | Script: si el diff modifica o borra tests existentes, falla salvo aprobación de `CODEOWNERS` | Igual |
| SAST | Semgrep CE u Opengrep, más Bandit | Semgrep CE u Opengrep |
| Secretos | gitleaks + push protection | Igual |
| Dependencias | OSV-Scanner + Dependabot | Igual |

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

## 12. Alternativa sin ejecutar en Actions

Si se prefiere ejecutar en la VM:
- **CAO** (AWS Labs) con un perfil `provider: opencode_cli` y el modelo de DeepSeek ✔︎, sobre el mismo tracker, con las mismas etiquetas y contratos.
- **Coste:** hay que operar `tmux`, el servidor de CAO y la reserva con la referencia git (sección 5).
- **Riesgo:** fallos abiertos en su *dispatcher* ([[flujo-fase-c2-orquestacion-ejecucion-revision]] §2.2).

## Enlaces

- [[flujo-agentes-informe]] — evidencia y descartes
- [[flujo-agentes-runbook]] — puesta en marcha y comprobación de coherencia
- [[sistema-desarrollo-con-agentes]] — proyecto
- [[_index]]
