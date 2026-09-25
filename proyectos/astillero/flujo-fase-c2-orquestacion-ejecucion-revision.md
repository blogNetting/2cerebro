---
title: Flujo con agentes, fase C2: orquestación, ejecución, revisión y trazabilidad
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, orquestacion, deepseek, revision, trazabilidad, investigacion]
zona: tecnico
---

Evaluación a fondo de orquestadores, ejecutores con modelo intercambiable, bucles de revisión y trazabilidad, con sus interfaces exactas.

Informe de fase; lo ha revisado el orquestador. Síntesis en [[flujo-agentes-informe]]. Las correcciones del orquestador están en esa síntesis.

> Informe de fase, en `scratchpad/` (no forma parte del wiki todavía). Sigue el método de [[circuito-tareas-definicion]] §6 fase C: cada candidato preseleccionado en B/B2 contra C1–C13, con evidencia en contra obligatoria (C13). Siglas explicadas la primera vez.

## 1. Método

Fuentes: documentación oficial (`WebFetch`/`curl`), código fuente y specs vía `gh api`/`raw.githubusercontent.com`, metadatos e *issues* vía `gh api repos/<owner>/<repo>` y `gh api repos/<owner>/<repo>/issues`, cobertura independiente vía la API de Algolia de Hacker News (HN, el foro de noticias tecnológicas `news.ycombinator.com`) — sin `WebSearch` (agotado, según el encargo) y sin herramientas de navegador. No se instaló nada; toda verificación es lectura de fuente. Se reutiliza como punto de partida lo ya verificado en `A2-practica-a-escala.md`, `B-implementaciones.md`, `B2-inventario-corregido.md` y `orquestacion-opus-deepseek-informe.md` (esta última cubre en detalle el conector Opus↔DeepSeek, fuera de alcance aquí salvo para los candidatos de ejecución) — se cita esa procedencia en vez de repetir la verificación cuando el dato ya estaba contrastado en fuente primaria por esa fase.

Limitación declarada: por el tamaño del encargo (5 bloques × hasta 13 criterios), la profundidad no es uniforme. Los candidatos con spec o código legible (Symphony, CAO, DeepSeek Harness, OpenCode, OMA) se leyeron en la fuente que define el mecanismo exacto (README, spec, catálogo de herramientas). Los de documentación puramente de producto (Kiro Crew, Cursor) se dejan con lo ya cubierto por B/B2 y no se reabren.

## 2. Orquestación / despacho — evaluación C1–C13

**Glosario mínimo:** *claim* = reserva de una pieza de trabajo por un agente; *lease* = reserva con caducidad; *worktree* = copia de trabajo git independiente sobre el mismo repositorio; *tracker* = sistema de seguimiento de tickets/issues (Jira, Linear, GitHub Issues…); MCP = *Model Context Protocol*, el protocolo estándar para dar herramientas a un agente; OTel = OpenTelemetry, el estándar de observabilidad (métricas/logs/trazas).

### 2.1 `openai/symphony` — spec de orquestación (Elixir, referencia)

Ya cubierto en profundidad por A2 (Fuente 6). Esta fase añade lo que faltaba para C1–C13: qué *trackers* soporta de verdad y si el comando del agente puede ser otro que Codex.

- **Trackers soportados por la implementación de referencia**: el propio repo, no solo la spec, incluye carpetas de adaptador para `asana`, `github`, `gitlab`, `jira`, `linear` (`gh api repos/openai/symphony/contents/elixir/lib/symphony_elixir`, verificado 2026-09-25: https://github.com/openai/symphony/tree/main/elixir/lib/symphony_elixir). La spec (`SPEC.md`) define el `Issue Tracker Adapter` como interfaz genérica («the name `Issue` is generic... an adapter MAY map it from a ticket, card, project item, or another provider-native work object», https://raw.githubusercontent.com/openai/symphony/main/SPEC.md líneas 159-160), así que en teoría cualquier *tracker* es implementable, pero solo esos 5 existen ya escritos.
- **¿El comando del agente puede ser otro que Codex? No, en la práctica.** La spec es explícita: *"Coding-agent executable that supports the targeted Codex app-server mode"* (línea 148) y *"The Codex app-server protocol for the targeted Codex version is the source of truth for protocol schemas, message payloads, transport framing, and method names"* (líneas 952-954). `codex.command` es configurable como cadena de shell (default `codex app-server`, línea 627) pero *"The launched process MUST speak a compatible app-server protocol over stdio"* (línea 478) — es decir: se puede sustituir el binario, no el protocolo. No hay evidencia de que Claude Code, OpenCode o Pi implementen el protocolo *app-server* de Codex (`https://developers.openai.com/codex/app-server/`, citado como fuente de verdad en la propia spec, línea 987); por tanto Symphony **no** es agnóstico de arnés en la práctica, pese a que sí lo es de *tracker*.
- **C4 (claim/concurrencia):** máquina de estados explícita de 5 estados (`Unclaimed→Claimed→…→Released`, líneas 643-659), reserva atómica antes de lanzar («`claimed` and `running` checks are REQUIRED before launching any worker», línea 728).
- **C8 (fallos):** taxonomía de clases de fallo (§14.1, línea 1636) y *backoff* exponencial con fórmula exacta: `delay = min(10000 * 2^(attempt-1), agent.max_retry_backoff_ms)`, tope configurable (default 300000ms/5min) (líneas 800-801, 625).
- **C13 (evidencia):** confirmado en A2 — 27.401★ con crecimiento sin poder verificar (API de *stargazers* 404), crítica de calidad de primera mano sobre la demo oficial en HN («inscrutable agent slop», `exclipy`), estado de orquestación solo en memoria (punto único de fallo documentado por el propio autor). Sin cambios respecto a A2.

### 2.2 `awslabs/cli-agent-orchestrator` (CAO) — nueva profundización

Repo oficial de AWS Labs, 1.349★, **149 issues abiertas, último push 2026-09-25 08:43** (mismo día del informe) — el candidato de orquestación con más actividad viva de todos los evaluados (`gh api repos/awslabs/cli-agent-orchestrator`, verificado 2026-09-25). README: https://github.com/awslabs/cli-agent-orchestrator/blob/main/README.md.

- **C2 (ejecutor intercambiable, encaje DeepSeek):** soporta 12 CLI de proveedor como *worker*: Kiro CLI, **Claude Code**, Codex CLI, Antigravity CLI, Hermes, Kimi CLI, MiniMax Code, GitHub Copilot CLI, **OpenCode CLI**, Oh My Pi (OMP) CLI, Cursor CLI, Grok Build CLI (README, sección Prerequisites). El perfil de agente (`docs/agent-profile.md`) fija `provider` y `model` por perfil YAML; para un *worker* creado dinámicamente por otro agente, *"a valid `provider` in the worker profile overrides the parent's provider... the worker inherits the parent provider"* (https://raw.githubusercontent.com/awslabs/cli-agent-orchestrator/main/docs/agent-profile.md). Con `provider: opencode_cli` y `model: deepseek/...` (formato `provider/model-id` de OpenCode, ver §3.1) se obtiene DeepSeek como ejecutor dentro de CAO sin escribir adaptador nuevo.
- **C4/C5 (asignación, aislamiento):** cada agente corre como proceso CLI completo en una sesión `tmux` aislada (`cao-server` local + `cao launch`); el aislamiento es de sesión de terminal, no de contenedor — encaja con la VM sin motor de contenedores.
- **C1/C3 (división del trabajo, estado):** dos niveles de autoría de *workflow* — guiones Python (`cao_workflow`, `run_step`/`emit_output`) con validación obligatoria antes de ejecutar (`cao workflow validate`) y un **journal duradero** que permite reanudar una ejecución interrumpida; el propio validador bloquea recursos no deterministas (`random`/`time`/`datetime`/`uuid` en el cuerpo del script, porque el *resume* re-ejecuta el script entero y compara contra lo journaleado — `ReplayDivergenceError` si diverge) y exige una `recovery=` explícita por paso (`missing-recovery-policy` es error bloqueante) (https://raw.githubusercontent.com/awslabs/cli-agent-orchestrator/main/docs/workflows.md). Es el mecanismo de reintento/determinismo más explícito de todos los orquestadores evaluados en C2.
- **C6 (roles/permisos):** `role` (`supervisor`, `developer`, `reviewer`…) con `allowedTools` como *allowlist* explícita, documentado en `docs/tool-restrictions.md`.
- **C11 (GitHub):** no es nativo de GitHub — es agnóstico de forja; el enlace con GitHub lo hace el propio CLI de proveedor (p. ej. Claude Code o Copilot CLI operando sobre el repo) o *hooks*/plugins propios de CAO, no algo documentado como primera clase en el README leído.
- **C10 (coste operativo):** requiere Python ≥3.10, `tmux` ≥3.3, `uv` (gestor de paquetes Python) — sin contenedor obligatorio (hay una vía Docker opcional vía *devcontainer feature*). Cabe en la VM sin instalar motor de contenedores.
- **Evidencia en contra (C13):** issues abiertas recientes muestran fricción real de producción, no solo *feature requests*: *"StatusMonitor's view of the pane lags tmux by ~2.3s"* (#815), *"cao CLI and Web UI send no bearer, so they cannot be used against an auth-enabled server"* (#807, fallo de autenticación), *"assume_processing_on_dispatch silently disables the mid-burst processing probe"* (#814) — bugs de detección de estado del propio *dispatcher*, justo el mecanismo que C4/C8 exigen confiable (https://github.com/awslabs/cli-agent-orchestrator/issues). Sin cobertura independiente en HN encontrada (no se hizo búsqueda dedicada por presupuesto; hueco anotado en §10).

### 2.3 Claude Code — subagentes + *agent teams*

Ya documentado en detalle por B/B2 (líneas 236-244 de `B-implementaciones.md`, verificación literal en `B2` §2). Esta fase resuelve la pregunta pendiente del encargo: **¿puede un *teammate* usar un modelo no-Anthropic?**

- **No, no está documentado ni soportado.** La doc de *agent teams* solo permite fijar el modelo de un *teammate* al generarlo, y solo entre modelos Claude (mismo hallazgo que para subagentes normales, confirmado también en `orquestacion-opus-deepseek-informe.md` §2: los subagentes de Claude Code solo aceptan modelos de Anthropic, con la petición de alias personalizados cerrada `NOT_PLANNED` en https://github.com/anthropics/claude-code/issues/34821 y el enrutado por agente a otro proveedor aún abierto en https://github.com/anthropics/claude-code/issues/38698). *Agent teams* es una función distinta de los subagentes (cola compartida + *claim* con *file locking* real, ver B2 §2.1) pero **hereda la misma limitación de modelo**: no hay ningún parámetro documentado en `code.claude.com/docs/en/agent-teams` que acepte un modelo no-Claude por *teammate*.
- Consecuencia directa para el diseño: si se quiere Claude Code como arnés del ejecutor DeepSeek dentro de una topología multiagente, la única vía verificada es el patrón DeepClaude (proceso Claude Code aparte, con `ANTHROPIC_BASE_URL` apuntando al endpoint compatible de DeepSeek) — **no** se puede lograr generando un *teammate* con modelo DeepSeek dentro de la sesión de *agent teams* del orquestador.
- C13: experimental y desactivado por defecto (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), con limitaciones explícitas de la propia doc en reanudación de sesión, coordinación de tareas y comportamiento de apagado (ya citado literalmente en B2).

### 2.4 `gastownhall/gastown` (Gas Town) — nueva profundización

18.185★, creado 2025-12-16, **último push 2026-09-18** (7 días antes del informe — activo pero no tan vivo como CAO), 481 *issues* abiertas, lenguaje Go (`gh api repos/gastownhall/gastown`, verificado 2026-09-25). README: https://github.com/gastownhall/gastown/blob/main/README.md.

- **C3 (estado del trabajo) — relación con Beads:** Gas Town **no sustituye** a Beads (ya preseleccionado en B2 §7 solo por su ficha técnica, sin mención de autoría), lo **envuelve**: *"Work state stored in Beads ledger"* y *"Beads Integration 📿: Git-backed issue tracking system that stores work state as structured data"* — los *beads* (identificadores `prefijo-5carácteres`, p. ej. `gt-abc12`) son la unidad de trabajo que Gas Town orquesta; Gas Town añade encima **Convoys** (agrupación de *beads* asignada a agentes), **Molecules** (plantillas de flujo multi-paso con puntos de control) y un **Refinery** (cola de *merge* por rig, estilo Bors —fusión por lotes con verificación bisectriz—, que aísla y re-despacha las *merge requests* fallidas).
- **C5 (topología, aislamiento):** aislamiento por **git worktree** por *rig* (proyecto) y por *hook* (almacenamiento persistente del trabajo de un agente) — confirma R5/R2 con el mismo mecanismo base que el resto del ecosistema.
- **C2 (ejecutor intercambiable):** el README declara Claude Code CLI como **runtime por defecto**, pero documenta alternativas: *"Runtime Configuration... alternatives (Codex, Copilot, Gemini, Cursor)"* y en la tabla de configuración de ejemplo aparece un *runtime* `codex` con `"command": "codex"`. La lista de **presets de agente incorporados** es más amplia que la frase inicial: `claude, gemini, codex, kiro, cursor, auggie, amp, opencode, copilot, pi, omp` — incluye **OpenCode y Pi**, ambos con soporte nativo de DeepSeek (ver §3), lo que sí da una vía de ejecutor DeepSeek dentro de Gas Town sin adaptador propio.
- **C8 (fallos/recuperación):** sistema de vigilancia de tres capas — **Witness** (gestor de ciclo de vida por *rig*, detecta agentes atascados), **Deacon** (supervisor de fondo con patrullas continuas), **Dogs** (trabajadores de infraestructura despachados por el Deacon) — y **Escalation** con severidad (CRITICAL/HIGH/MEDIUM) enrutada por *bead* hacia Deacon → Mayor → Overseer si hace falta. Es el mecanismo de escalado a humano más granular de todos los candidatos de orquestación evaluados.
- **C10 (coste operativo — el punto débil):** requisitos nativos: **Go 1.26.2+**, **Beads (`bd`) 0.57.0+**, **sqlite3**, y **Dolt** (una base de datos tipo git-SQL — dependencia no trivial, no un simple binario). Hay vía Docker Compose alternativa, pero el propio README avisa: *"Native installs require the host tools below. Docker installs only require Docker Compose on the host"* — es decir, sin Docker hay que instalar Dolt igualmente. Para la VM (4 núcleos/7GB, sin motor de contenedores instalado) esto es viable pero más pesado que CAO (que solo pide Python+tmux+uv): Dolt es una base de datos con su propio proceso, no una librería.
- **Evidencia en contra (C13):** ya la trae B2 (§3.2): hilo crítico en HN *"Does Gas Town 'steal' usage from users' LLM credits?"* — cuestiona si el propio sistema de orquestación consume cuota del usuario de forma no transparente, justamente el punto ciego de C9 (coste/observabilidad) que más preocupa a esta investigación dado que Opus va por suscripción con límites no publicados.

### 2.5 Gestores de *worktree* como clase — `claude-squad`, `vibe-kanban` (brevemente)

Confirmados por B/B2, sin profundización nueva en esta fase por ser una clase de herramienta ya bien cubierta (tablero/gestor que lanza y aísla varias instancias de arnés en *worktrees*, sin mecanismo propio de *claim*/*lease* documentado, ver B2 §7).

- `smtg-ai/claude-squad`: 8.529★, **último push 2026-08-20** — más de un mes sin actividad al momento del informe (leve señal de enfriamiento, no crítica). Gestiona Claude Code/Codex/OpenCode/Amp desde terminal.
- `BloopAI/vibe-kanban`: 28.189★, push 2026-09-19 (activo). Tablero kanban para «Claude Code, Codex o cualquier agente de código».
- Encaje en el circuito: son capa de UI/orquestación ligera sobre *worktrees*, no sistemas de estado del trabajo (no cumplen R2–R4 de forma documentada) — complementan, no sustituyen, a un orquestador con *claim* real como CAO o *agent teams*.

### 2.6 Kiro — oleadas por dependencias

Sin cambios respecto a B/B2 (cita literal ya verificada): *"Kiro builds a dependency graph of the tasks in your tasks.md and groups independent tasks into waves... Waves execute sequentially; tasks within a wave execute concurrently"* (https://kiro.dev/docs/specs/). Sigue siendo el único producto con documentación oficial de paralelismo por dependencias como función nativa — encaja con el mecanismo *wave-based topological dispatch* que describe el paper académico SPOQ (§4). Producto cerrado (AWS, sin código abierto que auditar); no se puede verificar el modelo por rol ni si acepta ejecutores no-Kiro.

### 2.7 Despacho desde GitHub — `claude-code-action` y `github/gh-aw`

- **`anthropics/claude-code-action`** (8.940★, push 2026-09-24). Soporta autenticación por API directa de Anthropic, Amazon Bedrock, Google Vertex AI y Microsoft Foundry (README: https://github.com/anthropics/claude-code-action/blob/main/README.md) — **todos son backends de modelos Claude**, ningún proveedor no-Anthropic documentado. **Pero sí es técnicamente enrutable a DeepSeek** por el mismo patrón DeepClaude: `action.yml` pasa `ANTHROPIC_BASE_URL: ${{ env.ANTHROPIC_BASE_URL }}` directamente al proceso de Claude Code que ejecuta (https://raw.githubusercontent.com/anthropics/claude-code-action/main/action.yml, líneas 340-343, verificado 2026-09-25), y el input `claude_args` («Additional arguments to pass directly to Claude CLI») permite añadir `--model <nombre-en-DeepSeek>`. **No está documentado ni probado por el vendor** — es una combinación de dos mecanismos oficiales (variable de entorno + paso de argumentos), no una función soportada; se marca como inferencia propia, sin confirmar si Anthropic bloquea `ANTHROPIC_BASE_URL` en el runner alojado.
- **`github/gh-aw`** (5.183★, push 2026-09-25). «GitHub Agentic Workflows»: define flujos agénticos en Markdown+YAML, compilados a Actions estándar (`gh aw compile`); trabajos de agente **de solo lectura por defecto**, escrituras validadas vía `safe-outputs` con permisos acotados (README: https://github.com/github/gh-aw). **Motores integrados y oficialmente soportados: Copilot (por defecto), Claude Code, Codex, Gemini, Pi** (https://github.github.com/gh-aw/reference/engines/, verificado 2026-09-25). La misma página documenta **explícitamente el mecanismo de enrutado a un proveedor no nativo**, por motor: `codex→OPENAI_BASE_URL`, `claude→ANTHROPIC_BASE_URL`, `copilot→GITHUB_COPILOT_BASE_URL`, `gemini→GEMINI_API_BASE_URL`; con ejemplo YAML de `engine.env` apuntando a un proxy/router interno, y una salvaguarda documentada: al fijar esas variables, gh-aw emite automáticamente `apiProxy.modelFallback.enabled: false` para no reescribir nombres de modelo específicos de proveedor. **Sobre Pi, la propia doc dice: *"Pi supports multiple providers but requires proxy-specific tool configuration"*** — confirmación oficial de GitHub de que Pi (y por tanto DeepSeek a través de Pi) es una vía soportada, con matices de configuración. **DeepSeek Harness aparece nombrado, pero solo como integración de muestra no soportada**: *"The OpenCode, Aider, Crush, Cursor, DeepSeek Harness, Kiro, and Pydantic AI integrations in this repository are samples only. They are not officially supported by gh-aw and have no compatibility or maintenance commitment"* (`.github/workflows/shared/deepseek-harness.md`) — evidencia en contra directa para C13 sobre esa combinación concreta.
- **Copilot coding agent** (cerrado, referencia). Sin cambios respecto a B: asignación por *issue assignee*, trazabilidad nativa vía commits/logs (https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent), selección de modelo mencionada sin detalle de si incluye no-OpenAI/no-Anthropic.

### 2.8 Tabla resumen C1–C13 (orquestación/despacho)

| Candidato | C1 división | C2 rol/modelo swap | C3 estado | C4 claim/concurr. | C5 topología/aislam. | C6 revisión | C8 fallos | C10 coste VM | C11 GitHub | C13 evidencia |
|---|---|---|---|---|---|---|---|---|---|---|
| Symphony (openai) | `WORKFLOW.md` versionado | No — protocolo Codex app-server obligatorio | Efímero en memoria + *tracker* externo (5 adaptadores) | Máquina de 5 estados, *claim* explícito | Workspace por *issue*, sin aislar estado en ejecución | Vía estado de handoff del *tracker* (`Human Review`) | Taxonomía de 5 clases + *backoff* con fórmula exacta | Ligero (proceso único) | Vía adaptador (no forja fijada) | Débil: sin adopción real, estrellas sin verificar, crítica de calidad propia del vendor en HN |
| CAO (awslabs) | Guion Python versionado + validación obligatoria | **Sí** — 12 CLI, incl. OpenCode/Claude Code, `provider`/`model` por perfil | Journal duradero, *resume* determinista | Recuperación por reintento con política `recovery=` declarada | Sesión `tmux` aislada por agente | Roles + `allowedTools` | `ReplayDivergenceError`, políticas de recuperación por paso | Python+tmux+uv, sin contenedor obligatorio | No nativo (agnóstico de forja) | Activo (push mismo día), pero *issues* abiertas sobre fiabilidad del propio *dispatcher* (#815, #807, #814) |
| Claude Code *agent teams* | Lista de tareas compartida | **No** — solo modelos Claude, ni en subagentes ni en *teammates* | En sesión, sin persistencia entre reinicios documentada más allá de la lista de tareas | *Claim* con *file locking* real (verificado literal) | Sesiones independientes por *teammate* | Hooks de calidad (`TeammateIdle`, `TaskCreated`…) | No documentado en detalle | Ligero (nativo del arnés que ya se usa) | No nativo | Experimental, desactivado por defecto, limitaciones explícitas de la propia doc |
| Gas Town | Beads (épica→bead) | Parcial — presets incl. OpenCode/Pi (DeepSeek indirecto), Claude Code por defecto | Beads (persistente, git-backed) | Convoys + *hooks* por *rig* | *Worktree* por *rig* + *hook* | Refinery (cola Bors-style) | 3 capas (Witness/Deacon/Dogs) + escalado por severidad | Go+Beads+sqlite3+**Dolt** — más pesado que CAO | No nativo | Hilo HN crítico sobre consumo no transparente de cuota LLM |
| Kiro (AWS) | `requirements→design→tasks` | Cerrado, sin verificar | Estado en tiempo real, propietario | No documentado (oleadas, no *claim* entre agentes) | Oleadas por dependencias (único con doc oficial) | No detallado en la doc leída | No detallado | Producto gestionado, no autoalojado | No nativo | Producto cerrado, sin código que auditar |
| claude-code-action | *Prompt*/mención/asignación de issue | Bedrock/Vertex/Foundry oficial; DeepSeek solo por combinación no documentada de `ANTHROPIC_BASE_URL`+`claude_args` | Vía comentario/PR de GitHub | No aplica (una ejecución por disparo) | *Runner* propio del usuario | Checklists de revisión personalizables | No detallado | Ligero (Action) | **Nativo** (forja fijada) | Oficial de Anthropic, activo |
| gh-aw (github) | Markdown+YAML → Action compilada | **Sí, documentado por motor** (`ANTHROPIC_BASE_URL` etc.); Pi con DeepSeek vía proxy — oficial | `safe-outputs` validados | No aplica (una ejecución por disparo) | *Runner* de Actions, solo lectura por defecto | `safe-outputs` con permisos acotados | No detallado | Ligero (Action) | **Nativo** (forja fijada) | Oficial de GitHub, activo (push mismo día) |

## 3. Ejecutores con modelo intercambiable

### 3.1 OpenCode (`sst/opencode`, redirige a `anomalyco/opencode`)

Ya con ficha en `orquestacion-opus-deepseek-informe.md` (210k★ en esa fecha, integración oficial documentada por DeepSeek). Esta fase añade los comandos exactos.

- **Invocación no interactiva:** `opencode run "<prompt>"` (https://opencode.ai/docs/cli/). Flags relevantes: `--model`/`-m` en formato **`provider/model`** (p. ej. `deepseek/deepseek-chat`), `--agent`, `--continue`/`-c`, `--session`/`-s`, `--prompt`.
- **Cómo recibe el encargo:** argumento de línea de comandos (texto libre) o `--session <id>` para retomar una sesión anterior.
- **Proveedores:** OpenCode usa el SDK de IA (*AI SDK*) y Models.dev para más de 75 proveedores, con **DeepSeek listado como proveedor de primera clase** en la página oficial de proveedores (https://opencode.ai/docs/providers/, confirmado 2026-09-25: aparece en el índice junto a Anthropic, OpenAI, Groq, etc.). Configuración en dos pasos: `/connect` (clave de API) y el `provider`/`model` en la config de OpenCode.
- **MCP:** `opencode mcp add|list|auth|logout|debug` — gestión nativa de servidores MCP, incluida autenticación OAuth.
- **Modelos disponibles:** `opencode models [provider]` lista los modelos accesibles en formato `provider/model`, con `--refresh` para forzar recarga desde Models.dev y `--verbose` para coste/metadatos.
- **Integración con GitHub:** subcomando `opencode github run` explícitamente pensado para GitHub Actions, con `--event`/`--token`.
- **Telemetría/coste:** el flag `--verbose` de `opencode models` expone coste por modelo; no se encontró (en el tiempo disponible) un formato de salida de coste por ejecución de `opencode run` documentado aparte del habitual JSON de eventos del SDK.
- **Fallos conocidos con DeepSeek (ya en el informe previo):** un *gateway* intermediario retiró DeepSeek de su nivel gratuito (https://news.ycombinator.com/item?id=49388835) — no aplica si se usa la API directa de DeepSeek, que es la restricción del usuario.

### 3.2 DeepSeek Harness (`deepseek-ai/deepseek-harness`) — profundización nueva completa

235.580★ (verificado 2026-09-25), creado **2026-08-13**, `has_issues: false` (issues cerradas; *feedback* solo vía GitHub Discussions), 0 *issues* abiertas porque el mecanismo está apagado, no porque no haya fricción. Licencia MIT. Construido sobre **Cordis**, un marco de composición «todo es un plugin» (paper propio: *A Programming Paradigm for Spatiotemporal Composability*, https://arxiv.org/abs/2608.25512).

- **Modo headless confirmado — hallazgo central de esta fase:** el lanzador `dsh` tiene un catálogo de **perfiles**, y uno de ellos es exactamente lo que pide el encargo:

  | Comando | Propósito |
  |---|---|
  | `dsh --profile headless "job"` | Ejecuta una sesión nueva persistida, imprime la respuesta final y termina |
  | `dsh --profile acp` | Sirve clientes de automatización sobre ACP (*Agent Client Protocol*) por stdio hasta desconexión |
  | `dsh --profile sdk` / `sdk-minimal` | Sirve clientes SDK sobre JSON-RPC por stdio |
  | `dsh web` | Interfaz web (modo por defecto documentado en el README principal) |

  (Fuente: https://raw.githubusercontent.com/deepseek-ai/deepseek-harness/master/apps/cli/README.md, verificado 2026-09-25.) El README raíz solo promociona `npx @deepseek-ai/dsh web`, lo que explica por qué B2 lo dejó como «arnés de un solo agente estilo app de consumo» — **corrección de esta fase**: el modo `headless` y el modo `acp` sí existen y están documentados, solo que no en el README de aterrizaje.
- **Herramientas (catálogo verificado, `docs/tool-catalog.md`, generado automáticamente por `pnpm run verify-tool-catalog` desde los propios *plugins* en ejecución — no es documentación escrita a mano, por lo que no puede quedar desactualizada sin fallar el build):** `bash`/`pwsh` (con `run_in_background` vía un registro de *jobs*), edición de ficheros (`edit`/`read`/`write`, con política de «leer antes de escribir»), `glob`/`grep` (ripgrep empaquetado), terminales persistentes (`terminal_open/read/send/close/signal/list`), `run_code`, `exit_plan_mode` (modo plan, análogo al de Claude Code), `ask_user_question` (aprobación humana nativa) y **recursos MCP** (`list_mcp_resources`, `read_mcp_resource`) — sí soporta MCP como cliente.
- **Hallazgo no anticipado por B2 — equipo de agentes nativo:** existe un paquete experimental `@deepseek-ai/dsh-experimental-tool-agent-team` con herramientas `spawn_teammate`, `team_task_create/get/list/update`, `send_message`, `interrupt_agent`, `wait_agent` — un patrón *lead + teammates + tareas compartidas* equivalente al de *agent teams* de Claude Code, pero nativo de DeepSeek y (por construcción Cordis) potencialmente enrutable a cualquier modelo, no solo DeepSeek. Está **desactivado por defecto** en el paquete base (`dsh-base`) y marcado explícitamente `experimental`.
- **Git:** no hay herramienta de git dedicada en el catálogo; se cubre con la herramienta `bash` genérica, igual que en la mayoría de arneses de este tipo.
- **Tests:** ninguna herramienta específica de tests; se ejecutan igualmente vía `bash`/`run_code`. No hay un *gate* de tests nativo documentado (a diferencia de los *hooks* de calidad de *agent teams* de Claude Code).
- **Cobertura HN y señal de anomalía:** lanzamiento oficial con 747 puntos / 314 comentarios (https://news.ycombinator.com/item?id=49285244, 2026-08-13) — crecimiento de estrellas explicado por una cobertura de lanzamiento real y fuerte, no anómala en su origen. Cobertura posterior mucho más débil: *"Ask HN: Anyone using DeepSeek Harness (dsh) as part of a customer-facing agent?"* (5 puntos, 0 comentarios, 2026-09-19, sin respuestas — https://news.ycombinator.com/item?id=49770744) es evidencia directa en contra de adopción real para casos de agente: **la única pregunta de uso en producción encontrada se quedó sin respuesta**, cinco semanas después del lanzamiento. El resto de cobertura posterior son *plugins* de terceros de bajo impacto (`busabase-dsh-plugin`, 5 puntos; tabla de precios de modelos, 3 puntos) y un artículo de blog de terceros («What Is DeepSeek-Harness? A Complete Introduction», 3 puntos) — ninguno con datos cuantitativos propios de uso.
- **C13 conclusión revisada respecto a B2:** sigue sin evidencia de uso multiagente en producción, pero **sí** tiene evidencia primaria y verificada de que el mecanismo headless/ACP/SDK existe, con catálogo de herramientas comparable en amplitud al de Claude Code o Pi. Pasa de «candidato a vigilar, sin evaluar» a **«ejecutor técnicamente viable y bien equipado, con adopción real todavía sin confirmar»**. Es, además, el único ejecutor de los cuatro finalistas que es simultáneamente (a) el arnés oficial del proveedor del modelo elegido por restricción del usuario y (b) capaz de MCP, plan mode y aprobación humana nativos.

### 3.3 Pi (`earendil-works/pi`)

Ya cubierto por `orquestacion-opus-deepseek-informe.md` §4.2: modos *print*, JSON y RPC, SDK de TypeScript, el arnés más ligero (4 herramientas, 1.340 tokens de sistema frente a 23.132 de Claude Code, mismo benchmark de calidad — https://nqawhc.github.io/articles/harness-efficiency-not-quality/), uso diario reportado con `deepseek-v4-pro` (~10M tokens/día, https://news.ycombinator.com/item?id=48413629). Esta fase confirma metadatos: **109.260★**, creado 2025-08-09, push **2026-09-25** (mismo día, muy activo), 231 *issues* abiertas (`gh api repos/earendil-works/pi`, verificado 2026-09-25). Cobertura HN adicional encontrada ahora: hilo con 56 puntos/21 comentarios sobre un fallo real de empaquetado (*"Pi coding agent: config folder is out of place on Linux"*, https://news.ycombinator.com/item?id=..., 2026-08-17) — evidencia en contra concreta (bug de convención de directorios en Linux, la plataforma de la VM del usuario), y un hilo de 3 puntos («Pi Agent Being Merged?») que sugiere incertidumbre sobre el futuro del proyecto como entidad independiente — **sin confirmar**, solo especulación de foro.
Confirmado además en §2.7: **Pi es uno de los 5 motores oficialmente soportados por `github/gh-aw`**, con la propia documentación de GitHub afirmando que soporta múltiples proveedores mediante configuración de proxy — refuerza a Pi como el punto de unión entre «ejecutor ligero con DeepSeek nativo» y «despacho oficial desde GitHub Actions».

### 3.4 Claude Code headless apuntado al endpoint Anthropic-compatible de DeepSeek

Sin cambios de fondo respecto a `orquestacion-opus-deepseek-informe.md` §4.2 (patrón DeepClaude, `claude -p`, `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`). Límite conocido y ya citado: arnés pesado (23.132 tokens de sistema) y un nombre de modelo no reconocido cae sin avisar a `deepseek-flash` según la propia documentación de DeepSeek (https://api-docs.deepseek.com/guides/anthropic_api) — riesgo de C9 (coste/observabilidad): una ejecución podría estar usando silenciosamente un modelo distinto al configurado.

### 3.5 Tabla comparativa de ejecutores

| Ejecutor | Invocación no interactiva | Cómo recibe el encargo | Cómo reporta el resultado | MCP | Git | DeepSeek nativo | Evidencia en contra |
|---|---|---|---|---|---|---|---|
| OpenCode | `opencode run "<prompt>"` | Argumento CLI o `--session` | Salida del SDK/eventos; `--json`-like no confirmado en detalle | Sí, gestión nativa (`opencode mcp`) | Vía `bash` | **Sí**, proveedor de primera clase | *Gateway* de terceros retiró DeepSeek del nivel gratuito |
| DeepSeek Harness | `dsh --profile headless "job"` | Argumento CLI | Respuesta final impresa y salida | Sí, cliente MCP (recursos) | Vía `bash` | **Sí** (es el arnés del propio proveedor) | Adopción real sin confirmar (pregunta de uso en HN sin respuestas); *issues* cerradas (opacidad) |
| Pi | Modo *print*/JSON/RPC (README del paquete) | *Prompt* CLI o RPC | JSON estructurado o RPC | Sí (extensiones) | Vía herramienta básica | **Sí**, uso diario reportado en foro | Bug de convención de directorios en Linux; incertidumbre de futuro del proyecto (sin confirmar) |
| Claude Code → DeepSeek (DeepClaude) | `claude -p` | Argumento CLI | Igual que Claude Code nativo | Sí (tools MCP locales, confirmado en informe previo) | Vía `bash` | Sí, por endpoint Anthropic-compatible | Arnés pesado; *fallback* de modelo silencioso documentado por DeepSeek |

## 4. Bucle de revisión — opciones con evidencia

- **Rondas de CI acotadas + *backoff*:** confirmado con dos fuentes de vendor independientes en A2 (Stripe: máximo 2 rondas de CI; Symphony: *backoff* exponencial acotado configurable) — el patrón mejor contrastado de todo el circuito (A2 §4, ya citado).
- **LLM como juez (*LLM-as-a-judge*):** mecanismo nuevo visto solo en el informe de Spotify (A2), como puerta previa a la revisión humana — sin cifra de tasa de defectos post-fusión publicada por ninguna de las 4 organizaciones de A2 (laguna ya señalada en A2 §4.6).
- **Comprobación plan-vs-implementación:** `until-dev/plugins` — *"The outer loop for coding agents. Define intent in a Plan, let the agent run, and check every pull request against it"* (`gh api repos/until-dev/plugins`, verificado 2026-09-25: descripción literal del repo). **Solo 31★, push 2026-09-17** — encaja exactamente con el patrón que pide el encargo (revisión de la implementación contra el plan original, no solo contra el diff), pero con evidencia de adopción mínima; no cumple el listón de C13 («al menos dos fuentes independientes») por sí solo — se marca «prometedor sin contrastar», no recomendable como pieza principal.
- **`humanlayer/humanlayer` — evidencia en contra fuerte, corrección obligatoria de B2.** El README del repo, en el estado leído el 2026-09-25, dice literalmente: *"public issues repo for humanlayer - the code here is pretty much all deprecated - you can try the rebuild of humanlayer at https://humanlayer.com - thanks for all your support - dex"* (https://raw.githubusercontent.com/humanlayer/humanlayer/main/README.md). Último *push* al repo **2026-06-19**, más de tres meses antes de este informe. El propio proyecto se declara obsoleto y redirige a un producto de pago (`humanlayer.com`) sin código abierto verificado en esta pasada. **Se retira como candidato de código abierto para la capa de aprobación humana**; el patrón «aprobación humana como capa explícita» sigue siendo válido conceptualmente (lo confirma también SPOQ, «Human-as-an-Agent», y Co-Coder) pero necesita una implementación distinta — no hay sustituto encontrado en el tiempo disponible (hueco anotado en §10).
- **Claude Code *hooks* como puerta:** confirmado por B/B2 — código de salida 2 bloquea la acción en 33 eventos de ciclo de vida, incluidos `PreToolUse`/`PostToolUse`, `TaskCreated`/`TaskCompleted`, `TeammateIdle` (https://code.claude.com/docs/en/hooks, cita ya verificada en B2 §2.3). Mecanismo genérico de *quality gate* disponible ya en el arnés que usa el usuario, independiente del modelo del rol que se esté revisando.
- **GitHub *rulesets* y *checks* obligatorios:** un *ruleset* es una lista de reglas nombrada, aplicable a un repositorio o a varios de una organización, con hasta 75 *rulesets* por repositorio y permisos de omisión (*bypass*) configurables por rol/equipo/GitHub App (https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets, verificado 2026-09-25). El nivel organización-completa requiere plan GitHub Team o Enterprise según la misma página; el uso a nivel de un solo repositorio (el caso de este proyecto) no depende de esa restricción — **sin verificar el detalle exacto de qué nivel de plan hace falta para *rulesets* de un solo repo**, se recomienda comprobarlo en la cuenta real antes de diseñar sobre ello. `CODEOWNERS` (fichero que asigna revisores obligatorios por ruta) es una función complementaria y estable de GitHub, ya conocida, no reverificada en detalle en esta pasada por no aportar nada nuevo sobre lo ya sabido.
- **SPOQ (arXiv 2606.03115) y Co-Coder (arXiv 2606.00953):** sin cambios respecto a B — ya resumidos con cifras propias (SPOQ: validación dual reduce defectos de 0.34 a 0.20/tarea, Human-as-an-Agent los reduce a 0.03/tarea; Co-Coder compara directamente contra «Claude Code with Agent Teams» con mejoras de hasta 14.0% en *pass rate*). Ambos sin réplica independiente conocida — tratar como hipótesis fuerte, no como hecho, tal y como ya señalaba B.

## 5. Trazabilidad — opciones con evidencia

- **`open-multi-agent/open-multi-agent` (OMA) — hallazgo central de esta fase.** 6.953★, creado 2026-03-31, push **2026-09-23** (activo). Descripción del propio repo: *"Self-hosted TypeScript agent runtime with durable approvals and verifiable run records. Own it, approve it, audit it"* (`gh api repos/open-multi-agent/open-multi-agent`, verificado 2026-09-25). Mecanismo concreto verificado en el README (https://github.com/open-multi-agent/open-multi-agent/blob/main/README.md):
  - Cada ejecución, con un *journal* (diario de eventos) conectado, registra **cada bloque que el modelo vio, cada llamada a herramienta y su resultado, y cada reescritura de contexto**.
  - `verifyRun()` lee ese diario **offline, en frío**, y comprueba que cada bloque se reproduce byte a byte a partir del evento de origen nombrado; una ventana de contexto descartada (por límite de tamaño) se reporta como «no concluyente», no como fallo — es decir, prueba linaje y contenido, no que el fichero nunca se editó (matiz que el propio README declara explícitamente, evitando sobrevender la garantía).
  - Incluye un **Run Viewer** ofimático (sin servicio alojado) que reproduce el DAG (grafo dirigido acíclico) de tareas, el *waterfall* de *spans* (medición de duración anidada) y la evidencia por tarea.
  - **Aprobaciones duraderas** (`packages/core/src/approval/durable.ts`, con 16+7 casos de test documentados en el propio README) — el trabajo se suspende (`result.status?.code === 'suspended'`) hasta que un revisor decide.
  - Integración **OTel opcional** («Install the OTel package only when OMA traces should appear in the same monitoring system as the rest of your application») — no obliga a montar observabilidad externa si no se necesita, relevante para C10 en la VM.
  - **C13:** sin segunda fuente independiente de adopción encontrada en el tiempo disponible más allá del propio repo (badge de auditoría de cadena de suministro visible, pero no evidencia de uso por terceros) — se marca «documentación primaria sólida, adopción externa sin verificar».
- **Cabeceras de commit y bots (`Co-Authored-By`, identidad de sesión):** ya verificado como patrón adoptado por el propio flujo de trabajo de esta sesión (ver recordatorio de atribución activo en esta conversación); es el mecanismo más barato y ya operativo de C7 (quién hizo cada cambio), pero no cubre C9 (coste) ni C4 (verificación de integridad tipo OMA).
- **OpenTelemetry de Claude Code:** confirmado con detalle exacto de configuración (https://code.claude.com/docs/en/monitoring-usage.md, verificado 2026-09-25): variables `CLAUDE_CODE_ENABLE_TELEMETRY=1`, `OTEL_METRICS_EXPORTER`/`OTEL_LOGS_EXPORTER` (`otlp`, `prometheus`, `console`, `none`), `OTEL_EXPORTER_OTLP_ENDPOINT`/`_PROTOCOL`/`_HEADERS`. Atributos identificadores opcionales por métrica: `OTEL_METRICS_INCLUDE_SESSION_ID` (activo por defecto), `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` (activo por defecto), `OTEL_METRICS_INCLUDE_REPOSITORY` (atributos `vcs.*`, requiere v2.1.269+, desactivado por defecto). Trazas (distribución de *spans*) en beta, requieren además `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`. Contenido sensible (*prompts*, respuestas, cuerpos brutos de la API) está **desactivado por defecto** y requiere variables explícitas (`OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_RAW_API_BODIES`) — diseño consciente de privacidad, relevante para C12. Es el único de los ejecutores/orquestadores evaluados con un estándar de observabilidad (OTel) documentado con este nivel de detalle de forma nativa.
- **Cómo registra «quién/qué/qué modelo» cada pieza evaluada (resumen):**

  | Pieza | Quién | Qué modelo | Cuándo/cómo |
  |---|---|---|---|
  | Symphony | Identidad de despacho del *tracker* (opaca) | `codex_*` campos de telemetría (tokens, `rate_limits`) | Log estructurado por evento del *app-server* |
  | CAO | Perfil de agente (`name`, `role`) | Campo `provider`/`model` en el journal del *workflow* | Journal duradero + `cao workflow status` |
  | Claude Code *agent teams* | Nombre de *teammate* | Fijado al generarse (solo Claude) | Buzón (*mailbox*) + lista de tareas compartida |
  | Gas Town | Identidad de *polecat* (persistente) | *Runtime* configurado por preset | Bead + historial en Dolt/Beads |
  | OMA | Cuenta que aprueba (aprobaciones duraderas) | No es su función (agnóstico de motor) | Journal verificable offline (`verifyRun()`) |
  | Claude Code OTel | `user.account_uuid`, `session.id` | Nombre de modelo en atributos de *span*/evento | Exportación continua a un colector OTLP |

## 6. Interfaces — comandos, claves de config, variables de entorno exactas

**Orquestación**
- Symphony: config declarativa con secciones `tracker` (`kind`, `provider`, `required_labels`, `active_states`, `terminal_states`), `codex` (`command`, default `codex app-server`; `approval_policy`, `thread_sandbox`, `turn_sandbox_policy`), `agent.max_retry_backoff_ms` (default `300000`). Fuente: https://raw.githubusercontent.com/openai/symphony/main/SPEC.md §5.3.
- CAO: `cao-server` (servidor local), `cao install code_supervisor`, `cao launch --agents <perfil>`, `cao shutdown --session <nombre>|--all`, `cao workflow validate <ruta>.py`, `cao workflow run <stem> --run-id <id>`, `cao workflow status|cancel|resume <run-id>`. Perfil de agente: Markdown con *front matter* YAML (`name`, `description`, `role`, `provider`, `model`, `allowedTools`, `mcpServers`, `permissionMode`…) en `~/.aws/cli-agent-orchestrator/`. Fuentes: https://github.com/awslabs/cli-agent-orchestrator/blob/main/README.md, https://raw.githubusercontent.com/awslabs/cli-agent-orchestrator/main/docs/agent-profile.md, https://raw.githubusercontent.com/awslabs/cli-agent-orchestrator/main/docs/workflows.md.
- Claude Code *agent teams*: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (variable de entorno que activa la función). *Hooks* relevantes: `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `WorktreeCreate`/`WorktreeRemove`, `PreModelSwitch`/`PostModelSwitch` — salida con código 2 bloquea la acción (B2 §2.3, ya verificado literal).
- Gas Town: `gt install ~/gt --shell --git`, `gt sling <bead-id>`, `gt convoy`, `gt escalate`, `gt config agent set <nombre> "<comando>"` (p. ej. `gt config agent set codex-low "codex --thinking low"`). Requisitos: Go 1.26.2+, `bd` (Beads) 0.57.0+, sqlite3, Dolt. Fuente: https://raw.githubusercontent.com/gastownhall/gastown/main/README.md.
- gh-aw: YAML *front matter* con `engine: <id>` (`copilot`/`claude`/`codex`/`gemini`/`pi`), `engine.env.<VAR>` para el proxy (p. ej. `ANTHROPIC_BASE_URL`), `network.allowed` (lista de dominios permitidos, debe incluir el dominio del proxy), `safe-outputs` para escrituras validadas. Compilación: `gh aw compile`. Fuente: https://github.github.com/gh-aw/reference/engines/.
- claude-code-action: `action.yml` con `claude_args` (argumentos pasados directos a la CLI de Claude), variables de entorno pasadas al proceso incluida `ANTHROPIC_BASE_URL`, y flags de proveedor alternativo `use_bedrock`/`use_vertex`/`use_foundry`. Fuente: https://raw.githubusercontent.com/anthropics/claude-code-action/main/action.yml.

**Ejecución**
- OpenCode: `opencode run "<prompt>" --model <provider>/<model> [--agent <nombre>] [--auto]`; gestión de proveedor: `/connect` (interactivo) + `provider`/`model` en config; `opencode models [provider] [--refresh] [--verbose]`; `opencode mcp add|list|auth|logout|debug`; `opencode github run --event <e> --token <t>`. Fuente: https://opencode.ai/docs/cli/, https://opencode.ai/docs/providers/.
- DeepSeek Harness: `dsh --profile headless "<job>"` (una sesión, respuesta final, sale); `dsh --profile acp` (stdio, protocolo ACP); `dsh --profile sdk`/`sdk-minimal` (JSON-RPC por stdio); `dsh --dump-config`/`--dump-default-config`/`--dump-config-schema` para inspeccionar la config compuesta sin arrancar. Variable de entorno de instalación: `$DSH_HOME/profiles/<name>`. Fuente: https://raw.githubusercontent.com/deepseek-ai/deepseek-harness/master/apps/cli/README.md.
- Pi: modos *print*/JSON/RPC vía el paquete `coding-agent` (`packages/coding-agent/README.md`), SDK de TypeScript. Detalle exacto de flags no re-verificado en esta pasada (ya en el informe previo); pendiente si se llega a diseño de detalle (§10).
- Claude Code → DeepSeek: `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic claude -p "<prompt>"` (patrón DeepClaude, ya verificado en informe previo). Endpoint oficial de DeepSeek: https://api-docs.deepseek.com/guides/anthropic_api.

**Revisión**
- Claude Code *hooks*: `settings.json`/`.claude/settings.json`, claves por evento (`PreToolUse`, `PostToolUse`, `TaskCreated`, `TaskCompleted`, `TeammateIdle`, etc.), código de salida 2 bloquea. Fuente: https://code.claude.com/docs/en/hooks (ya citada literal en B2).
- GitHub *rulesets*: configurables por UI o API (`PUT /repos/{owner}/{repo}/rulesets`), hasta 75 por repositorio, `bypass_actors` por rol/equipo/app. Fuente: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets.
- `until-dev/plugins`: sin comandos verificados en esta pasada más allá de la descripción del repo (bajo peso de evidencia, ver §4).

**Trazabilidad**
- OMA: API en TypeScript (`packages/core`), `verifyRun()` para verificación offline; documentación de referencia en `docs/run-store.md` (almacén de ejecuciones y *leases* — coincide con el vocabulario R3/R4 de `circuito-tareas-definicion.md`), `docs/durable-approvals.md`, `docs/observability.md`. Fuente: https://github.com/open-multi-agent/open-multi-agent/blob/main/README.md.
- Claude Code OTel: `CLAUDE_CODE_ENABLE_TELEMETRY=1`, `OTEL_METRICS_EXPORTER`/`OTEL_LOGS_EXPORTER`/`OTEL_TRACES_EXPORTER` (`otlp`/`console`/`prometheus`(solo métricas)/`none`), `OTEL_EXPORTER_OTLP_ENDPOINT`/`_PROTOCOL`/`_HEADERS` (y variantes por señal `_METRICS_`/`_LOGS_`/`_TRACES_`), `OTEL_METRICS_INCLUDE_SESSION_ID`/`_ACCOUNT_UUID`/`_REPOSITORY`, `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1` (trazas). Fuente: https://code.claude.com/docs/en/monitoring-usage.md.

## 7. Evidencia en contra por candidato (obligatoria, C13)

| Candidato | Evidencia en contra | Fuente |
|---|---|---|
| Symphony | Crecimiento de estrellas sin poder verificar (API 404); crítica de calidad de primera mano sobre la demo oficial («inscrutable agent slop»); estado de orquestación solo en memoria, punto único de fallo documentado por el propio autor; protocolo obligatoriamente Codex app-server, no agnóstico de arnés pese a serlo de *tracker* | A2 (ya citado), spec §10 (esta fase) |
| CAO (awslabs) | *Issues* abiertas sobre fiabilidad del propio *dispatcher*: retardo de 2.3s en el monitor de estado (#815), fallo de autenticación de la CLI/Web UI (#807), sonda de procesamiento silenciosamente desactivada (#814) | https://github.com/awslabs/cli-agent-orchestrator/issues |
| Claude Code *agent teams* | Experimental, desactivado por defecto; limitaciones explícitas en reanudación de sesión, coordinación y apagado; ningún modelo no-Claude soportado, ni aquí ni en subagentes (petición cerrada `NOT_PLANNED`) | B2 §2.3 (ya citado); https://github.com/anthropics/claude-code/issues/34821 |
| Gas Town | Hilo crítico en HN sobre consumo no transparente de cuota LLM del usuario; requisitos de infraestructura más pesados que CAO (Dolt, base de datos dedicada) | B2 §3.2 (ya citado); README (esta fase) |
| Kiro | Producto cerrado, sin código auditable; sin verificar si acepta ejecutores no-Kiro ni modelo por rol | Ya señalado en B2 §7 |
| claude-code-action | El enrutado a DeepSeek requiere combinar dos mecanismos no pensados juntos por el vendor (`ANTHROPIC_BASE_URL` + `claude_args`), sin documentación ni prueba oficial de esa combinación | Inferencia propia sobre `action.yml`, esta fase |
| gh-aw | DeepSeek Harness solo como integración de muestra, explícitamente «not officially supported... no compatibility or maintenance commitment» | https://github.github.com/gh-aw/reference/engines/ |
| OpenCode | Un *gateway* de terceros retiró DeepSeek del nivel gratuito (no aplica si se usa la API directa de DeepSeek) | https://news.ycombinator.com/item?id=49388835 (ya en informe previo) |
| DeepSeek Harness | La única pregunta de uso real en producción encontrada en HN («customer-facing agent») quedó sin respuestas 5 semanas después del lanzamiento; *issues* de GitHub cerradas (0 abiertas, `has_issues:false`), lo que reduce la visibilidad de fricción real; sin evidencia de uso del modo *agent-team* experimental | https://news.ycombinator.com/item?id=49770744 (esta fase) |
| Pi | Bug de convención de directorios de configuración en Linux (la plataforma de la VM del usuario), 56 puntos/21 comentarios; incertidumbre especulativa sobre el futuro del proyecto («Being Merged?», sin confirmar) | https://news.ycombinator.com/item?id=49328206 (2026-08-17) |
| Claude Code → DeepSeek (DeepClaude) | Arnés pesado (23.132 tokens de sistema); *fallback* de modelo silencioso a `deepseek-flash` si el nombre no se reconoce | Informe previo, ya citado |
| `until-dev/plugins` | Solo 31★, adopción mínima, no cumple el listón de dos fuentes independientes de C13 | `gh api repos/until-dev/plugins`, esta fase |
| `humanlayer/humanlayer` | El propio repo se declara obsoleto («pretty much all deprecated»), sin *push* desde hace más de 3 meses, redirige a producto de pago sin código abierto verificado | https://raw.githubusercontent.com/humanlayer/humanlayer/main/README.md, esta fase |
| OMA | Sin segunda fuente independiente de adopción encontrada; badge de auditoría de cadena de suministro visible pero sin caso de uso de terceros documentado | esta fase |

## 8. Ranking y combinaciones coherentes de extremo a extremo

**Inferencia propia** (no hay fuente que compare estas piezas entre sí como sistema; se etiqueta como tal en todo este apartado).

Dado el conjunto de restricciones del usuario (DeepSeek para implementar, Claude por Pro para diseñar/revisar, GitHub como forja, sin contenedores instalados, sin código sin tests, 4 núcleos/7GB), las combinaciones que **conectan de verdad** — cada pieza documentada como capaz de hablar con la siguiente — son:

1. **Orquestación CAO + ejecutor OpenCode(DeepSeek)/Claude Code + revisión por *hooks* + GitHub nativo vía Actions separado.** CAO documenta explícitamente OpenCode y Claude Code como *providers*; OpenCode documenta DeepSeek como proveedor de primera clase; CAO no es nativo de GitHub, así que el tramo PR/checks necesita `gh` (CLI de GitHub) invocado desde dentro del *worker*, no un mecanismo propio de CAO — **hueco de interfaz**, no de concepto. Es la combinación con más piezas activas y documentadas en detalle (journal duradero, política de recuperación explícita) de todas las evaluadas.
2. **`gh-aw` (motor `pi`) + Pi apuntando a DeepSeek + `safe-outputs` para PR/labels + GitHub *rulesets* como *gate*.** Es la única combinación **100% documentada de extremo a extremo por los propios vendors**: gh-aw declara oficialmente que Pi soporta múltiples proveedores vía proxy, compila a Actions estándar, y sus `safe-outputs` son el mecanismo documentado para crear PRs/ramas con permisos acotados — pero corre *dentro* de GitHub Actions, no en la VM local, lo que cambia el modelo de coste/latencia respecto a lo que parece asumir la idea de partida del usuario (Opus orquestando localmente).
3. **Claude Code (sesión de Opus, suscripción Pro) como orquestador/lead local + proceso DeepClaude/OpenCode/Pi como ejecutor en *worktree* aislado + *hooks* de Claude Code como *gate* + PR a GitHub por CLI de `gh`.** Es el esqueleto ya propuesto en `orquestacion-opus-deepseek-informe.md` §5; esta fase no encuentra ningún orquestador de terceros que mejore ese esqueleto sin introducir una dependencia nueva no trivial (Dolt en Gas Town, protocolo Codex-only en Symphony, `tmux`+journal propio en CAO). **Es la combinación más coherente con la restricción real de «Opus por Pro, límites de uso»**, porque mantiene a Opus en su propia sesión de suscripción en vez de moverlo a un proceso separado que un orquestador de terceros tendría que invocar por API (rompiendo la ventaja de la suscripción).
4. **DeepSeek Harness en modo `headless`/`acp` como ejecutor «doméstico» del proveedor** — combinable con cualquiera de los tres orquestadores anteriores como una alternativa más a OpenCode/Pi/DeepClaude en el tramo ejecutor, con la ventaja de ser el arnés oficial de DeepSeek y tener MCP y aprobación humana nativos, y la desventaja de adopción real sin confirmar.
5. **OMA como capa de trazabilidad transversal**, no atada a ningún orquestador concreto: al ser agnóstico de motor, puede envolver cualquiera de las combinaciones anteriores para dar verificación offline de las ejecuciones — es la pieza que mejor resuelve C7/R9 sin acoplarse a la elección de orquestador/ejecutor.

**No coherentes / con hueco de interfaz sin resolver documentado:** Symphony (protocolo Codex-only bloquea sustituir el ejecutor por DeepSeek dentro de su propio *runner*, aunque sí lo permite como *tracker* genérico); Gas Town con Claude Code como *runtime* por defecto (habría que forzar el preset `opencode`/`pi`, no documentado como camino principal); humanlayer (repo obsoleto, no hay pieza que conectar).

## 9. Fuentes

**Con aporte:**
- https://raw.githubusercontent.com/openai/symphony/main/SPEC.md (spec completa, 2312 líneas)
- `gh api repos/openai/symphony/contents/elixir/lib/symphony_elixir` (adaptadores de *tracker*)
- https://github.com/awslabs/cli-agent-orchestrator/blob/main/README.md
- https://raw.githubusercontent.com/awslabs/cli-agent-orchestrator/main/docs/agent-profile.md
- https://raw.githubusercontent.com/awslabs/cli-agent-orchestrator/main/docs/workflows.md
- `gh api repos/awslabs/cli-agent-orchestrator` (metadatos, *issues*)
- https://raw.githubusercontent.com/gastownhall/gastown/main/README.md
- `gh api repos/gastownhall/gastown`
- https://github.com/anthropics/claude-code-action/blob/main/README.md
- https://raw.githubusercontent.com/anthropics/claude-code-action/main/action.yml
- https://github.github.com/gh-aw/reference/engines/
- https://raw.githubusercontent.com/deepseek-ai/deepseek-harness/master/README.md
- https://raw.githubusercontent.com/deepseek-ai/deepseek-harness/master/apps/cli/README.md
- https://raw.githubusercontent.com/deepseek-ai/deepseek-harness/master/docs/tool-catalog.md
- `gh api repos/deepseek-ai/deepseek-harness`
- https://hn.algolia.com/api/v1/search?query=deepseek%20harness (cobertura HN)
- https://opencode.ai/docs/cli/, https://opencode.ai/docs/providers/
- `gh api repos/earendil-works/pi`; https://hn.algolia.com/api/v1/search?query=earendil%20pi%20agent
- https://raw.githubusercontent.com/humanlayer/humanlayer/main/README.md
- `gh api repos/open-multi-agent/open-multi-agent`; https://github.com/open-multi-agent/open-multi-agent/blob/main/README.md
- https://code.claude.com/docs/en/monitoring-usage.md
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets
- `gh api repos/until-dev/plugins`
- `gh api repos/smtg-ai/claude-squad`, `gh api repos/BloopAI/vibe-kanban` (metadatos de actividad)
- Reutilizado con procedencia citada: `A2-practica-a-escala.md`, `B-implementaciones.md`, `B2-inventario-corregido.md`, `orquestacion-opus-deepseek-informe.md` (todos en esta misma carpeta/proyecto)

**Vacías (sin aporte, según regla §8 de `circuito-tareas-definicion.md`):**
- `deepseek-ai/deepseek-harness/README.md` en rama `main` — 404 (la rama por defecto real es `master`, no documentada como tal en ningún README leído; se detectó por `gh api` metadatos)
- Intento de HN Algolia por `"deepseek harness" site` no se hizo (se usó la consulta libre, suficiente)
- No se reabrió Kiro Crew (producto separado, "Running 24/7") por límite de tiempo — hueco ya anotado en B2 §8, sigue abierto
- No se investigó en detalle el paquete `coding-agent` de Pi más allá de lo ya citado en el informe previo (flags exactos de los modos *print*/JSON/RPC) — hueco nuevo, ver §10
- No se verificó si GitHub *rulesets* a nivel de un solo repositorio (no organización) requiere plan de pago — la página leída deja el punto ambiguo

## 10. Bloqueos

| Bloqueo | Efecto | Ayuda posible |
|---|---|---|
| Sin `WebSearch` (agotado, por encargo) | Cobertura de HN limitada a lo que la API de Algolia devuelve por consulta de texto libre; no se pudo contrastar con una segunda fuente independiente la adopción de OMA, CAO en foros, ni el estado exacto del hilo «Pi Agent Being Merged?» | Reabrir con cupo de `WebSearch` nuevo si el usuario lo autoriza |
| Sin herramientas de navegador | La documentación de Kiro Crew y algunas páginas de producto cerradas no se pudieron inspeccionar más allá del HTML crudo ya citado en B | Chrome real vía CDP, si se decide reabrir Kiro |
| No se profundizó en los flags exactos de los modos *print*/JSON/RPC de Pi ni en `docs/run-store.md`/`docs/durable-approvals.md` de OMA línea a línea | El diseño de interfaces (§6) tiene el nivel de detalle de CAO/DeepSeek Harness/gh-aw pero no el mismo para Pi/OMA en profundidad de fichero de configuración | Sesión dedicada si estas dos piezas entran en la preselección final |
| No se comprobó el nivel de plan de GitHub necesario para *rulesets* de un solo repositorio | Riesgo de diseñar sobre un supuesto («cabe en el plan actual») sin confirmar | Comprobar en la cuenta real de GitHub del usuario antes de la fase D |
| Límite de uso del plan Pro (heredado de fases anteriores) | El trabajo de esta fase se hizo en una sola sesión sin interrupción, pero el riesgo persiste para D | — |

## Enlaces internos

- [[circuito-tareas-definicion]] — pregunta, criterios y método de esta investigación
- [[orquestacion-opus-deepseek-informe]] — conector Opus↔DeepSeek, base de §3 de este informe
- Informes previos de esta misma carpeta de trabajo: `A-practicas-reales.md`, `A2-practica-a-escala.md`, `B-implementaciones.md`, `B2-inventario-corregido.md`

## Enlaces

- [[flujo-agentes-informe]] — síntesis de la investigación
- [[circuito-tareas-definicion]] — definición de la fase
- [[_index]]
