---
title: Flujo con agentes, fase B2: inventario corregido
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, herramientas, inventario, investigacion]
zona: tecnico
---

Corrección de la fase B: cita inventada retirada, inventario completado por varias vías y preselección rehecha con evidencia.

Informe de fase; lo ha revisado el orquestador. Síntesis en [[flujo-agentes-informe]]. Las correcciones del orquestador están en esa síntesis.

Subtarea de revisión encargada por el orquestador sobre la fase B (`B-implementaciones.md`). Sesión única, sin subagentes, sin navegador. Herramientas: `gh` (CLI oficial de GitHub), `WebFetch`/`curl` contra documentación y APIs públicas (GitHub GraphQL/REST, HN Algolia, arXiv). Fecha: 2026-09-25. Sigue [[circuito-tareas-definicion]] (6 preguntas Q1–Q6, criterios C1–C13, definición de «contrastado» en su §7).

El orquestador encontró dos fallos en B: **(1)** una cita literal fabricada de la doc de subagentes de Claude Code, y **(2)** un inventario incompleto pese a que B afirmaba haber clasificado «todo proyecto de ≥500★ encontrado». Este documento corrige ambos y recalcula la preselección.

---

## 1. Método

### 1.1 Verificación de citas (encargo 1)

Cada cita literal de B §3 se re-verificó contra el contenido crudo de la página fuente, no contra un resumen de modelo:

- Para páginas Mintlify (`code.claude.com`, `platform.claude.com`) se usó el sufijo `.md` que sirve el markdown fuente sin renderizar (`curl -s -L "<url>.md"`), evitando el problema que B mencionaba de «contenido embebido en el DOM final».
- Para páginas sin variante `.md` (Codex Cloud, Cursor, GitHub Copilot, Kiro, Amp) se descargó el HTML crudo con `curl` y se buscó cada fragmento citado con `python3`/`re` (evitando `grep`/`ugrep`, que fallaba con patrones largos en UTF-8).
- Para Gemini CLI se repitió el método de B: `raw.githubusercontent.com` sobre los `.md` del propio repo.
- Para el paper Co-Coder se releyó `arxiv.org/abs/2606.00953` directamente (B ya lo había hecho; aquí se re-verificó en persona la frase citada).

### 1.2 Búsqueda ampliada del inventario (encargo 2)

B usaba una sola estrategia (`gh search repos "<consulta de texto libre>" --sort stars`), que **no encuentra un repo si su descripción no contiene ninguna de las palabras buscadas** — el fallo metodológico exacto que señaló el orquestador. Aquí se añadieron tres estrategias complementarias, documentadas con sus consultas exactas (incluidas las vacías):

**(i) Búsqueda por topic de GitHub** — `gh search repos --topic <t> --sort stars --limit 40/100 --json fullName,stargazersCount`. Topics consultados: `ai-agents`, `coding-agent`, `claude-code`, `codex`, `multi-agent`, `agentic-coding`, `agent-orchestration`, `spec-driven-development`, `task-management`, `kanban`, `llm-agents`, `autonomous-agents`, `swe-agent`, y la combinación `developer-tools`+`ai`. Ninguna consulta devolvió cero resultados; cada una devolvió entre 13 (`swe-agent`) y 100 (varias). Total bruto: 14 topics × hasta 100 filas.

**(ii) Minado de listas curadas.** Se identificó y minó por completo `andyrewlee/awesome-agent-orchestrators` (README vía `gh api repos/andyrewlee/awesome-agent-orchestrators/readme --jq .content | base64 -d`): 234 enlaces `github.com/<owner>/<repo>` extraídos con regex, consultados en bloque por GraphQL (`gh api graphql` con alias `r0..r29` por lote de 30, para minimizar llamadas) para obtener estrellas, forks, issues abiertas y fecha de creación de los 234 en una sola pasada. Se identificaron además, pero **no se minaron con el mismo detalle por límite de tiempo** (ver §5): `RyanAlberts/best-of-Agent-Harnesses` (167 arneses catalogados) y `e2b-dev/awesome-ai-agents` (30.162★). `hesreallyhim/awesome-claude-code` y `ai-boost/awesome-harness-engineering` ya estaban clasificados por B como meta-fuentes, no se re-minaron.

**(iii) Repos y productos mencionados en fase A y A2.** `A-practicas-reales.md` no menciona ningún repo de GitHub relevante (grepeado con `github\.com/[A-Za-z0-9_.-]+/[A-Za-z0-9_.-]+`; el único hit, `until-dev/plugins`, es una cita ajena al tema). `A2-practica-a-escala.md` menciona explícitamente `openai/symphony` (ya cubierto en detalle por A2, pero **ausente de la tabla ≥500★ de B** — hueco confirmado) y ningún otro repo con nombre propio no mencionado ya por B.

**(iv) Ocho candidatos nombrados explícitamente por el orquestador** — `gh api repos/<owner>/<repo>` uno a uno: `openai/symphony`, `MrLesk/Backlog.md`, `BloopAI/vibe-kanban`, `smtg-ai/claude-squad`, `gastownhall/gastown`, `coleam00/Archon`, `OpenHands/OpenHands`, `bmad-code-org/BMAD-METHOD`.

Todas las llamadas de estrellas/forks/issues se hicieron con `gh api graphql` en lotes (20–30 repos por consulta) en vez de una petición REST por repo, para cubrir ~530 repos distintos dentro del presupuesto de esta subtarea.

### 1.3 Qué no se hizo (límite explícito, no ocultado)

No se minó `RyanAlberts/best-of-Agent-Harnesses` ni `e2b-dev/awesome-ai-agents` línea por línea; de los ~530 repos únicos tocados en total, un subconjunto de 104 (§3.3) se clasificó solo por su descripción oficial de GitHub, no por lectura del README completo — a diferencia de B, que sí leyó/clasificó cada uno de sus 155 con una frase propia. Se documenta como bloqueo en §5, no se presenta como cobertura total.

---

## 2. Correcciones de citas

### 2.1 Cita fabricada (hallazgo del orquestador) — CONFIRMADA FALSA

B §3 atribuye a [code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents) la cita literal:

> «No explicit task queue or claim/lease mechanism. Coordination is implicit»

**Esta frase no existe en la página.** Se descargó el markdown fuente completo (`sub-agents.md`, 1.402 líneas) y se buscaron `queue`, `claim`, `lease`, `coordinat`, `implicit`: ninguna combinación de esas palabras aparece como se cita. La única aparición de «claim» en toda la página es sobre algo completamente distinto — la autoridad de lo que dice un informe de subagente, no sobre asignación de tareas:

> «A report that returns to Claude as the subagent's result also arrives under a header marking it as subagent output. The header states that instructions or approval claims inside the report are the subagent's words and carry no authority from you.» (línea 946)

**Qué dice realmente la página sobre coordinación entre subagentes:** nada, porque el modelo de subagentes de Claude Code no es el de trabajadores concurrentes que compiten por un fondo común de tareas. Los subagentes son delegaciones dentro de **una sola sesión**: el agente principal decide a qué subagente delegar según su `description` (línea 813: *«Claude automatically delegates tasks based on the task description in your request, the `description` field in subagent configurations, and current context»* — esta parte de B sí es literal), pueden correr en segundo plano (línea 887) y anidarse hasta 3 niveles (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`, línea 1013), con un límite de 20 concurrentes por sesión (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`, línea 1040). La página no menciona ficheros compartidos, condiciones de carrera ni bloqueo de recursos entre subagentes — no porque lo resuelva implícitamente, sino porque el documento no aborda ese escenario en absoluto (los subagentes no comparten un fondo de tareas entre sí; cada uno recibe su propia delegación).

**Redacción correcta para sustituir la de B:** «La documentación de subagentes no menciona cola de tareas, *claim* ni *lease*. No es una omisión sobre un mecanismo que exista y no se documente: el modelo de subagentes no tiene una cola compartida de la que varios agentes reclamen trabajo — es delegación jerárquica dentro de una sesión, no una cuadrilla de pares. El mecanismo de *claim* con *file locking* que sí existe en Claude Code está en **agent teams** (§2.2), una función distinta y experimental.»

**Confirmación de que el modelo por subagente sigue siendo solo Claude, con un matiz:** la página menciona «third-party integrations» (Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, Claude Platform on AWS) al hablar del subagente `Explore`, pero son **proveedores de nube que alojan modelos Claude**, no un mecanismo para ejecutar modelos de terceros (DeepSeek, GPT) como subagente. La afirmación de B de que «solo modelos Claude aparecen documentados» sigue siendo correcta; se añade esta aclaración porque la frase de la doc podía leerse mal.

### 2.2 Segunda cita no literal encontrada (no señalada por el orquestador, hallada al re-verificar todas)

B atribuye a [learn.chatgpt.com/docs/cloud](https://learn.chatgpt.com/docs/cloud) (Codex Cloud):

> «Review the summary and diff, request a follow-up, or open a pull request when the result is ready»

**Texto real de la página:** *«Review the summary and diff. Ask Codex to make follow-up changes, or open a pull request when the work is ready.»*

Es una paráfrasis presentada entre comillas como cita literal — mismo defecto que el hallazgo del orquestador, aunque de menor gravedad (el sentido no cambia). Corregido aquí con el texto exacto.

También se detectó un truncamiento menor sin marcar con elipsis en la misma sección de B: la cita *«Start work in Codex cloud from GitHub pull requests, GitLab merge requests and issues, Linear issues, or Slack channels.»* corta la frase real, que continúa *«...or Slack channels **and threads**.»* — el contenido no cambia, pero falta la elipsis que exige el estilo de evidencia de `circuito-tareas-definicion.md` §8.

### 2.3 Resto de citas de B §3 — re-verificadas, todas literales

Se comprobó cada cita entre comillas de B §3 contra la fuente cruda (no un resumen). Resultado, con la página fuente descargada para cada una:

| Cita en B | Fuente | Verificación |
|---|---|---|
| Agent teams: «Tasks have three states: pending, in progress, and completed...» | `agent-teams.md` línea 186 | **Literal** |
| Agent teams: «Task claiming uses file locking to prevent race conditions...» | `agent-teams.md` línea 195 | **Literal** |
| Agent teams: «Agent teams have known limitations around session resumption...» | `agent-teams.md` línea 10 | **Literal** |
| Agent SDK: «a hosted agent harness that runs the agent loop, with sessions in an Anthropic-managed cloud sandbox or a self-hosted sandbox...» | `agent-sdk/overview.md` | **Literal** |
| Codex Cloud: «Give longer tasks dedicated environments and let them continue while you work on something else» | `codex-cloud.html` | **Literal** |
| Gemini CLI: «Local Model Routing (Experimental)» / Gemma local | `model-routing.md` línea 29–32 | **Literal** |
| Gemini CLI: «You are responsible for cleaning up your worktrees manually.» | `git-worktrees.md` línea 75 | **Literal** |
| OpenCode: «Primary agents are the main assistants you interact with directly» | `opencode.ai/docs/agents/` | **Literal** |
| OpenCode: «Subagents are specialized assistants that primary agents can invoke...» | ídem | **Literal** |
| OpenCode: «Use the model config to override the model for this agent...» + «...format `provider/model-id`» | ídem (dos frases distintas de la misma página, unidas con elipsis correctamente) | **Literal**, las dos partes |
| Cursor: «You can run as many agents as you want in parallel» | `cursor.com/docs/background-agent` | **Literal** |
| Cursor: «Agents are visible to members of the Cursor team they were started under» | ídem | **Literal** |
| Cursor: «Cloud Agents use a curated selection of models» | ídem | **Literal** |
| GitHub Copilot: «...selecting "Copilot" as the assignee» | `docs.github.com` (HTML-entity `&quot;`, contenido idéntico) | **Literal** |
| GitHub Copilot: «Working on GitHub adds transparency, with every step happening in a commit...» | ídem (envuelto en `<strong>`, texto plano idéntico) | **Literal** |
| GitHub Copilot: «Depending on how you start your Copilot cloud agent task, you may be able to select the model used» | ídem | **Literal** |
| Kiro: «Kiro builds a dependency graph of the tasks in your tasks.md and groups independent tasks into waves: Wave 1...Wave 2...Waves execute sequentially; tasks within a wave execute concurrently.» | `kiro.dev/docs/specs/` | **Literal** (elipsis correcta sobre texto intermedio real) |
| Amp: «Orbs: Machines that Amp creates per thread» | `ampcode.com/manual` | **Literal** |
| Amp: «Amp is the same agent and the same threads everywhere» | ídem | **Literal** |
| Amp: «Multi-Model: GPT-5.6, Claude Fable 5.1, fast models... Amp uses them all...» | ídem | **Literal** |
| Co-Coder (paper): compara contra «Claude Code with Agent Teams» | `arxiv.org/abs/2606.00953`, releído en persona | **Literal**, confirmado en el abstract |

**Conclusión de §2:** de 22 citas verificadas en B §3, **1 es fabricada** (subagentes/queue-claim-lease) y **1 es una paráfrasis no marcada como tal** (Codex Cloud/«request a follow-up»); las 20 restantes son literales, incluidas las de mayor peso para la preselección (agent teams, OpenCode, Kiro, Cursor, Copilot).

---

## 3. Adenda — repos ≥500★ relevantes no incluidos en la tabla de B

### 3.1 Los ocho nombrados por el orquestador

| Repo | ★ | ¿Ya estaba en B? |
|---|---|---|
| `coleam00/Archon` | 23.550 | **Sí** — fila 75 de la tabla de B, clasificado Relevante. No es un hueco. |
| `OpenHands/OpenHands` | 89.130 | **Sí** — fila 64 de la tabla de B, clasificado Relevante. No es un hueco. |
| `openai/symphony` | 27.401 | **No.** Hueco confirmado: A2 le dedica una sección entera (Fuente 6) pero nunca entra en la tabla ≥500★ de B. |
| `MrLesk/Backlog.md` | 6.840 | **No.** Hueco confirmado. |
| `BloopAI/vibe-kanban` | 28.189 | **No.** Hueco confirmado. |
| `smtg-ai/claude-squad` | 8.529 | **No.** Hueco confirmado. |
| `gastownhall/gastown` | 18.185 | **No.** Hueco confirmado (`gastownhall/beads` y `gastownhall/gascity`, de la misma organización, sí estaban; este repo distinto no). |
| `bmad-code-org/BMAD-METHOD` | 53.444 | **No.** Hueco confirmado. |

Dos de los ocho nombrados por el orquestador ya estaban cubiertos; se deja constancia para no sobrecorregir. Los otros seis son huecos reales y confirman el fallo metodológico señalado.

### 3.2 Adenda — relevantes (nuevos, ≥500★, no en B), por estrellas

Motivo abreviado; preguntas Q1–Q6 de [[circuito-tareas-definicion]] §1. Métricas vía `gh api graphql` el 2026-09-25.

| ★ | Repo | Motivo | Preguntas |
|---|---|---|---|
| 235.578 | [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) | Arnés oficial de DeepSeek («Everything is a Plugin»), lanzado 2026-08-13. Lanzamiento oficial con 747 puntos en HN ([story](https://hn.algolia.com/api/v1/search?query=deepseek%20harness)) — crecimiento explicado, no anómalo. `has_issues:false` (mismo patrón que `openai/symphony`, preview cerrada). **Directamente relevante a la restricción del usuario de usar DeepSeek para implementar**, pero sin evidencia encontrada de que resuelva el circuito multiagente Q1–Q6 más allá de ser un arnés de un solo agente estilo app de consumo — pendiente de profundizar en fase C | — (candidato «a vigilar», ver §4) |
| 77.843 | [stablyai/orca](https://github.com/stablyai/orca) | «ADE for working with a fleet of parallel agents. Run any coding agent with your own subscription» — encaja directo con «ejecutor intercambiable». Crecimiento rápido (6 meses) sin cobertura HN independiente encontrada pese a búsqueda específica; ratios forks/★ (6,5%) e issues/★ (8,7%) saludables, a diferencia de `affaan-m/ECC` — **señal mixta, sin verificar del todo** | Q4, Q5 |
| 53.444 | [bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | «Breakthrough Method for Agile AI Driven Development» — metodología de roles ágiles para desarrollo con agentes, enfoque distinto al de spec-kit (roles/ceremonias vs. comandos /specify /plan /tasks). Ratios sanos, sin anomalía | Q1, Q2 |
| 28.189 | [BloopAI/vibe-kanban](https://github.com/BloopAI/vibe-kanban) | Tablero kanban específico para gestionar agentes de código (Claude Code, Codex, cualquiera); 195 puntos en HN, bien corroborado | Q3, Q4 |
| 27.401 | [openai/symphony](https://github.com/openai/symphony) | Ver A2 Fuente 6: spec de orquestación más formal encontrada en toda la investigación (claim, *backoff*, taxonomía de fallos), pero sin adopción real documentada y con crecimiento de estrellas sin verificar (§6 de A2, bloqueo de API repetido aquí) | Q3, Q4, Q8 |
| 27.399 | [manaflow-ai/cmux](https://github.com/manaflow-ai/cmux) | Terminal (basado en Ghostty) con pestañas para multitarea de agentes de código en paralelo | Q4 |
| 23.521 | [pingdotgg/t3code](https://github.com/pingdotgg/t3code) | «Agent harness control surface»: controla Claude Code, Codex, Cursor, Grok Build, OpenCode y Antigravity desde móvil/web/escritorio — capa de control multi-arnés | Q4 |
| 18.531 | [getpaseo/paseo](https://github.com/getpaseo/paseo) | Orquesta múltiples agentes de código desde escritorio y móvil | Q3, Q4 |
| 18.265 | [andrewyng/openworker](https://github.com/andrewyng/openworker) | Agnóstico de modelo (BYO API key para cualquier proveedor, incluido Ollama local); bucle de revisión explícito: *«the fixer is never the only checker»*; acciones registradas y gobernadas — encaja con Q5/Q6/Q12 y con «ejecutor intercambiable» | Q5, Q6, Q12 |
| 18.185 | [gastownhall/gastown](https://github.com/gastownhall/gastown) | Gestor de espacio de trabajo multiagente, de la misma organización que `beads`/`gascity` (ya en B); 403+354+253+219 puntos en HN en distintos hilos, la mejor cobertura independiente encontrada en la adenda — incluye un hilo crítico *(«Does Gas Town 'steal' usage from users' LLM credits?»)*, evidencia en contra relevante para C12/C13 | Q3, Q4 |
| 14.614 | [superset-sh/superset](https://github.com/superset-sh/superset) | Orquesta 100+ agentes de código en paralelo con la propia suscripción; YC (P26), 108+96 puntos en HN — bien corroborado | Q4, Q5 |
| 14.570 | [AndyMik90/Aperant](https://github.com/AndyMik90/Aperant) | Codificación autónoma multi-sesión | Q3, Q4 |
| 12.362 | [Untrivial-ai/agent-orchestrator](https://github.com/Untrivial-ai/agent-orchestrator) | Supervisa equipos de agentes de código de planificación a *merge*, con cualquier arnés (+25) — cobertura amplia de Q1–Q5 | Q1–Q5 |
| 11.606 | [humanlayer/humanlayer](https://github.com/humanlayer/humanlayer) | Capa de aprobación humana («human-in-the-loop») para que agentes de código resuelvan problemas en bases de código complejas — mecanismo de revisión distinto (aprobación como servicio) a todo lo ya cubierto por B | Q5 |
| 10.977 | [google/ax](https://github.com/google/ax) | «Google's open agentic orchestration runtime» — motor de orquestación genérico, análogo a LangGraph/Temporal ya excluidos por B como infraestructura generalista | Q3 |
| 10.763 | [langchain-ai/open-swe](https://github.com/langchain-ai/open-swe) | Agente de código asíncrono open-source de LangChain; `has_issues:false` (repo aún en preview) | Q3, Q5 |
| 10.550 | [openchamber/openchamber](https://github.com/openchamber/openchamber) | Entorno de desarrollo agéntico sobre OpenCode | Q4 |
| 9.639 | [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) | Implementación nº2 (de 4 encontradas ≥500★) del patrón «Ralph Wiggum» ya documentado en B (`tzachbon/smart-ralph`) y A2 (OpenAI, Stripe) — confirma la replicación amplia del patrón de reintento acotado | Q8 |
| 8.940 | [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action) | **Oficial de Anthropic**, no cubierto por B §3 (funciones nativas): GitHub Action que activa Claude Code por mención `@claude`, asignación de issue o *prompt* explícito; corre en el *runner* propio del usuario; salida como comentario/PR con seguimiento visual de progreso — hueco de documentación nativa en B, relevante a C11 | Q1, Q5, Q6 |
| 8.529 | [smtg-ai/claude-squad](https://github.com/smtg-ai/claude-squad) | Gestiona múltiples instancias de Claude Code/Codex/OpenCode/Amp desde terminal | Q4 |
| 6.953 | [open-multi-agent/open-multi-agent](https://github.com/open-multi-agent/open-multi-agent) | «Self-hosted TypeScript agent runtime with durable approvals and verifiable run records. Own it, approve it, audit it.» — mecanismo de aprobación + registro auditable, encaje directo con Q5/Q6 (la pregunta peor resuelta según A2 §4.6) | Q5, Q6 |
| 6.840 | [MrLesk/Backlog.md](https://github.com/MrLesk/Backlog.md) | Gestor de tareas Markdown-nativo en el propio repo git, para humanos y agentes; 254 puntos en HN | Q1, Q3 |
| 6.095 | [Q00/ouroboros](https://github.com/Q00/ouroboros) | «Agent OS»: evaluación por etapas con puerta de entrevista, bucle de evolución con presupuesto; 14 arneses soportados (Claude Code, Codex, Gemini CLI, OpenCode, Copilot, Kiro...) | Q3, Q5, Q8 |
| 5.830 | [generalaction/emdash](https://github.com/generalaction/emdash) | YC (W26); ejecuta múltiples agentes de código en paralelo, cualquier proveedor | Q4 |
| 5.592 | [21st-dev/1code](https://github.com/21st-dev/1code) | Capa de orquestación para agentes de código (Claude Code, Codex) | Q3, Q4 |
| 5.183 | [github/gh-aw](https://github.com/github/gh-aw) | **Oficial de GitHub**, no cubierto por B §3: «GitHub Agentic Workflows» — define automatización agéntica en Markdown+YAML, la compila a GitHub Actions estándar; motores soportados: Copilot, Claude Code, Codex, Gemini, Pi; trabajos de agente de solo lectura por defecto, escrituras validadas vía `safe-outputs` con permisos acotados — hueco de documentación nativa en B, relevante a C11/C12 | Q1, Q3, Q12 |
| 3.524 | [SeemSeam/claude_codex_bridge](https://github.com/SeemSeam/claude_codex_bridge) | Espacio de trabajo CLI multiagente visible, mezcla Codex/Claude/Gemini/Kimi/Qwen/Cursor/Copilot/Pi/OpenCode | Q4 |
| 3.221 | [AutoMaker-Org/automaker](https://github.com/AutoMaker-Org/automaker) | Dirige agentes con varios proveedores (Claude/Codex/Copilot/Cursor/Gemini/OpenCode) sobre Claude Agent SDK | Q4 |
| 3.158 | [mikeyobrien/ralph-orchestrator](https://github.com/mikeyobrien/ralph-orchestrator) | Implementación nº3 del patrón Ralph Wiggum | Q8 |
| 2.976 | [michaelshimeles/ralphy](https://github.com/michaelshimeles/ralphy) | Implementación nº4 del patrón Ralph Wiggum (bash, multi-arnés: Claude Code, Codex, OpenCode, Cursor, Qwen, Droid) | Q8 |
| 2.813 | [teaql/teaql-agent-kit](https://github.com/teaql/teaql-agent-kit) | «A model-mediated harness for reliable agentic software development» — descripción vaga, prioridad baja, sin profundizar | Q1 |
| 2.448 | [subsy/ralph-tui](https://github.com/subsy/ralph-tui) | Implementación nº5 del patrón Ralph Wiggum, con TUI | Q8 |
| 2.355 | [AgentsMesh/AgentsMesh](https://github.com/AgentsMesh/AgentsMesh) | Ejecuta ~100 agentes de código en máquinas propias, con planificación/aislamiento | Q3, Q4 |
| 2.222 | [zeronsh/zeron](https://github.com/zeronsh/zeron) | Plano de control nativo para Claude Code, Codex, Cursor, Devin y otros | Q4 |
| 2.195 | [amElnagdy/delegate-skills](https://github.com/amElnagdy/delegate-skills) | «Delegate a coding task to a separate coding agent CLI, review the diff, land the commit yourself — one per implementer» — patrón explícito de revisión humana por delegación, pequeño pero directamente en el punto de Q5 | Q2, Q5 |
| 2.166 | [777genius/agent-teams-ai](https://github.com/777genius/agent-teams-ai) | Kanban + agentes que se asignan y revisan entre sí + 300+ modelos/200+ proveedores LLM — encaja fuerte con «ejecutor intercambiable» y revisión agente-a-agente (P17 de A2) | Q3, Q5, Q6 |
| 2.121 | [Orkas-AI/Orkas](https://github.com/Orkas-AI/Orkas) | LLM comandante dirige subagentes especialistas sobre CLIs instaladas (Claude Code, Codex, OpenCode, OpenClaw, Hermes) | Q2, Q4 |
| 2.089 | [google-github-actions/run-gemini-cli](https://github.com/google-github-actions/run-gemini-cli) | **Oficial de Google**, Action para invocar Gemini CLI en GitHub Actions — mismo hueco de documentación nativa que los dos anteriores | Q1, Q11 |
| 2.035 | [coder/xum](https://github.com/coder/xum) | App de escritorio para desarrollo agéntico aislado en paralelo | Q4, Q5 |
| 1.910 | [the-open-engine/zeroshot](https://github.com/the-open-engine/zeroshot) | «Independent executor–verifier orchestration for software changes» — patrón explícito de dos roles separados ejecutor/verificador | Q2, Q5 |

### 3.3 Adenda — irrelevantes, clasificados con motivo (todas ≥500★ encontradas y no en B)

**Ecosistema «OpenClaw»** (agentes personales/de automatización general de escritorio, no específicos de desarrollo en equipo — misma categoría de exclusión que B aplicó a AutoGPT/goose/cline):

| ★ | Repo | Motivo |
|---|---|---|
| 390.467 | [openclaw/openclaw](https://github.com/openclaw/openclaw) | «The AI that really does things. Any OS. Any Platform.» — agente de automatización de propósito general, no de desarrollo en equipo. Cobertura HN masiva y bien corroborada (1.349, 1.099, 802, 667, 518, 514, 511, 397 puntos en hilos distintos), incluidas restricciones de Anthropic/Google por uso indebido de suscripciones y una CVE de escalada de privilegios — **no es una anomalía de estrellas, es un producto real y controvertido**, pero fuera del alcance Q1–Q6 |
| 248.814 | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | «The agent that grows with you» — agente personal generalista |
| 32.878 | [zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw) | Infraestructura de asistente personal autónomo, ecosistema OpenClaw |
| 30.846 | [nanocoai/nanoclaw](https://github.com/nanocoai/nanoclaw) | Alternativa ligera a OpenClaw para mensajería (WhatsApp/Telegram/Slack/Discord/Gmail) |
| 30.013 | [sipeed/picoclaw](https://github.com/sipeed/picoclaw) | Ecosistema OpenClaw, automatización general |
| 22.538 | [NVIDIA/NemoClaw](https://github.com/NVIDIA/NemoClaw) | Ejecuta agentes tipo Hermes/OpenClaw en un sandbox NVIDIA, no específico de código |
| 12.632 | [nearai/ironclaw](https://github.com/nearai/ironclaw) | «Agent OS focused on privacy, security and extensibility», ecosistema OpenClaw |
| 8.103 | [nullclaw/nullclaw](https://github.com/nullclaw/nullclaw) | Infraestructura de asistente autónomo, ecosistema OpenClaw |
| 6.069 | [netease-youdao/LobsterAI](https://github.com/netease-youdao/LobsterAI) | Construido explícitamente sobre OpenClaw, agente de escritorio generalista |
| 5.542 | [HKUDS/ClawTeam](https://github.com/HKUDS/ClawTeam) | «Agent Swarm Intelligence», ecosistema OpenClaw |
| 3.493 | [aiming-lab/MetaClaw](https://github.com/aiming-lab/MetaClaw) | Agente personal autoevolutivo, ecosistema OpenClaw |
| 2.499 | [snarktank/antfarm](https://github.com/snarktank/antfarm) | «Build your agent team in OpenClaw with one command» — construido sobre OpenClaw, no independiente |
| 2.229 | [tnm/zclaw](https://github.com/tnm/zclaw) | Asistente personal en un ESP32 (microcontrolador), completamente fuera de dominio |

**Anomalías de estrellas nuevas** (mismo control que B §6 — velocidad de crecimiento, ratio forks/issues, cobertura HN):

| ★ | Repo | Creado | Forks | Issues abiertas | Cobertura HN | Veredicto |
|---|---|---|---|---|---|---|
| 33.362 | [Yeachan-Heo/oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex) | 2026-02-02 (~7,7 meses) | 2.542 | **1** (issues activadas) | No buscada específicamente | **Anomalía fuerte**: 33k★/2,5k forks con una sola issue abierta en 7,7 meses es el mismo patrón de alerta que `gsd-build/get-shit-done` en B §6, pero más extremo. No se preselecciona |
| 40.677 | [herdrdev/herdr](https://github.com/herdrdev/herdr) | 2026-03-27 (~6 meses) | 3.109 | 334 | Búsqueda «herdr coding agent» en HN: máximo 5 puntos, ningún hilo propio encontrado | **Señal débil, no concluyente** (igual que `gsd-build` en B §6): ratios forks/issues saludables, pero sin corroboración HN pese a 40k★. No se preselecciona hasta aclarar |

**Otras exclusiones puntuales:**

| ★ | Repo | Motivo |
|---|---|---|
| 17.967 | [rowboatlabs/rowboat](https://github.com/rowboatlabs/rowboat) | Framework multiagente genérico («AI coworker with memory and collaboration»), no nacido para el circuito código→PR→revisión — misma razón que B usó para excluir CrewAI/AutoGen |
| 17.548 | [leon-ai/leon](https://github.com/leon-ai/leon) | Asistente personal preexistente desde 2019, falso positivo del topic `ai-agents`; no relacionado con desarrollo de software |
| 18.210 | [RightNow-AI/openfang](https://github.com/RightNow-AI/openfang) | «Open-source Agent Operating System» genérico, sin evidencia de patrón específico de desarrollo en equipo |
| 10.887 | [accomplish-ai/coworker](https://github.com/accomplish-ai/coworker) | README: «This project is no longer supported» — proyecto abandonado, evidencia en contra directa de C13 |
| 10.113 | [cloudflare/cloudflare-os](https://github.com/cloudflare/cloudflare-os) | Espacio de trabajo de agentes genérico (documentos, apps), no específico de código |
| 83.177 | [paperclipai/paperclip](https://github.com/paperclipai/paperclip) | «Manage agents at work» — descripción demasiado vaga para confirmar relevancia al circuito de código; sin profundizar |
| 51.323 | [multica-ai/multica](https://github.com/multica-ai/multica) | «Make humans and AI agents work as one team» — plataforma de trabajo en equipo genérica, no específica de código |
| 48.561 | [HKUDS/nanobot](https://github.com/HKUDS/nanobot) | Framework de agente personal, generalista |
| 33.666 | [Pythagora-io/gpt-pilot](https://github.com/Pythagora-io/gpt-pilot) | Agente único («the first real AI developer»), sin *push* desde 2026-06 (~3 meses de inactividad); sin patrón multiagente |
| 15.658 | [plandex-ai/plandex](https://github.com/plandex-ai/plandex) | Agente de terminal único, sin orquestación multiagente |
| 7.708 | [sweepai/sweep](https://github.com/sweepai/sweep) | Asistente de código para JetBrains, agente único, sin *push* desde 2025-09 (~1 año, posible abandono) |

**Cola larga sin profundizar** (104 repos, clasificados solo por su descripción oficial de GitHub — RAG/scraping/memoria/observabilidad de LLM, gestión de proyectos genérica sin IA, guías/tutoriales/libros, agentes de un solo propósito ajeno al desarrollo de software, listas curadas adicionales, falsos positivos de topic). Se listan todos, sin ocultar ninguno, con su descripción oficial como motivo:

| ★ | Repo | Descripción (motivo) |
|---|---|---|
| 184.485 | [firecrawl/firecrawl](https://github.com/firecrawl/firecrawl) | The web data API to search, scrape, and interact at scale. 🔥 |
| 147.034 | [langchain-ai/langchain](https://github.com/langchain-ai/langchain) | The agent engineering platform. |
| 145.698 | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | Makes your AI agent think like the laziest senior dev in the room. |
| 121.288 | [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) | Turn any codebase into a queryable knowledge graph (skill, no orquestación) |
| 116.235 | [browser-use/browser-use](https://github.com/browser-use/browser-use) | Agents that use the browser. |
| 94.659 | [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) | Persistent context across sessions — capa de memoria, no de coordinación |
| 91.284 | [infiniflow/ragflow](https://github.com/infiniflow/ragflow) | Motor RAG |
| 84.237 | [unclecode/crawl4ai](https://github.com/unclecode/crawl4ai) | Web crawler/scraper para LLMs |
| 78.613 | [dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide) | Guía educativa |
| 76.920 | [AppFlowy-IO/AppFlowy](https://github.com/AppFlowy-IO/AppFlowy) | Alternativa a Notion, gestión de proyectos genérica |
| 75.643 | [microsoft/ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners) | Curso educativo |
| 75.611 | [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) | Lista curada |
| 74.259 | [thedaviddias/Front-End-Checklist](https://github.com/thedaviddias/Front-End-Checklist) | Checklist, no herramienta de orquestación |
| 71.713 | [daytonaio/daytona](https://github.com/daytonaio/daytona) | Infraestructura de sandboxing genérica |
| 70.036 | [diegosouzapw/OmniRoute](https://github.com/diegosouzapw/OmniRoute) | *Gateway* de modelos (enrutador de API), no orquestador de tareas |
| 69.394 | [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) | Descripción ininteligible, sin evidencia de patrón relevante |
| 68.436 | [openinterpreter/openinterpreter](https://github.com/openinterpreter/openinterpreter) | Agente único para modelos abiertos |
| 68.305 | [asgeirtj/system_prompts_leaks](https://github.com/asgeirtj/system_prompts_leaks) | Filtraciones de *system prompts*, material de referencia |
| 66.449 | [Mintplex-Labs/anything-llm](https://github.com/Mintplex-Labs/anything-llm) | Agente local genérico |
| 66.323 | [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) | Guía de buenas prácticas |
| 65.976 | [mem0ai/mem0](https://github.com/mem0ai/mem0) | Capa de memoria, no de coordinación |
| 59.866 | [makeplane/plane](https://github.com/makeplane/plane) | Alternativa a Jira, gestión de proyectos sin IA específica |
| 57.026 | [rohitg00/ai-engineering-from-scratch](https://github.com/rohitg00/ai-engineering-from-scratch) | Material educativo |
| 50.890 | [bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book) | Libro |
| 47.984 | [GitHubDaily/GitHubDaily](https://github.com/GitHubDaily/GitHubDaily) | Boletín de noticias de GitHub |
| 46.887 | [sickn33/agentic-awesome-skills](https://github.com/sickn33/agentic-awesome-skills) | Catálogo de *skills*, no orquestador |
| 43.911 | [MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search) | Framework de búsqueda de empleo, dominio ajeno |
| 43.176 | [chatanywhere/GPT_API_free](https://github.com/chatanywhere/GPT_API_free) | Proxy de API gratuito, dominio ajeno |
| 42.335 | [agno-agi/agno](https://github.com/agno-agi/agno) | Plataforma de agentes genérica |
| 41.035 | [Hmbown/Codewhale](https://github.com/Hmbown/Codewhale) | Agente de terminal único |
| 40.278 | [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) | Tutor personalizado, dominio ajeno |
| 36.020 | [continuedev/continue](https://github.com/continuedev/continue) | Agente de código único (extensión IDE) |
| 35.702 | [esengine/DeepSeek-Reasonix](https://github.com/esengine/DeepSeek-Reasonix) | Agente de terminal único sobre DeepSeek |
| 33.892 | [TabbyML/tabby](https://github.com/TabbyML/tabby) | Autocompletado de código autoalojado |
| 30.313 | [ComposioHQ/composio](https://github.com/ComposioHQ/composio) | Plataforma de *tooling*/autenticación para agentes |
| 30.162 | [e2b-dev/awesome-ai-agents](https://github.com/e2b-dev/awesome-ai-agents) | Lista curada (no minada en detalle, ver §5) |
| 28.489 | [yamadashy/repomix](https://github.com/yamadashy/repomix) | Empaqueta un repo para dárselo a un LLM, utilidad puntual |
| 28.125 | [QwenLM/qwen-code](https://github.com/QwenLM/qwen-code) | Agente de terminal único |
| 27.317 | [Fosowl/agenticSeek](https://github.com/Fosowl/agenticSeek) | Agente autónomo generalista local |
| 27.112 | [OthmanAdi/planning-with-files](https://github.com/OthmanAdi/planning-with-files) | Planificación persistente en fichero para un agente, no multiagente |
| 24.943 | [vxcontrol/pentagi](https://github.com/vxcontrol/pentagi) | Agente de *pentesting*, dominio ajeno |
| 24.379 | [NirDiamant/GenAI_Agents](https://github.com/NirDiamant/GenAI_Agents) | Tutoriales |
| 21.094 | [wekan/wekan](https://github.com/wekan/wekan) | Kanban genérico sin IA |
| 20.120 | [1jehuang/jcode](https://github.com/1jehuang/jcode) | «The most RAM efficient harness» — agente único; posible referencia de coste operativo (C10) pero sin patrón multiagente |
| 20.074 | [dailydotdev/daily](https://github.com/dailydotdev/daily) | Agregador de noticias, dominio ajeno |
| 17.689 | [TransformerOptimus/SuperAGI](https://github.com/TransformerOptimus/SuperAGI) | Framework de agente autónomo generalista |
| 16.203 | [opf/openproject](https://github.com/opf/openproject) | Gestión de proyectos genérica sin IA |
| 15.885 | [treeverse/dvc](https://github.com/treeverse/dvc) | Versionado de datos/ML, dominio ajeno |
| 15.259 | [theonedev/onedev](https://github.com/theonedev/onedev) | Plataforma de desarrollo genérica sin foco en agentes |
| 14.527 | [aiming-lab/AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw) | Investigación autónoma, dominio ajeno |
| 14.318 | [tonhowtf/omniget](https://github.com/tonhowtf/omniget) | Descargador de cursos/vídeos, dominio ajeno |
| 14.301 | [fathah/hermes-desktop](https://github.com/fathah/hermes-desktop) | Cliente de escritorio de Hermes, agente personal |
| 12.680 | [The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge](https://github.com/The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge) | Tutorial |
| 12.582 | [plankanban/planka](https://github.com/plankanban/planka) | Kanban genérico sin IA |
| 11.657 | [Leantime/leantime](https://github.com/Leantime/leantime) | Gestión de proyectos genérica sin IA |
| 10.110 | [invoiceninja/invoiceninja](https://github.com/invoiceninja/invoiceninja) | Facturación, dominio ajeno |
| 9.883 | [kanboard/kanboard](https://github.com/kanboard/kanboard) | Kanban genérico sin IA |
| 9.808 | [GetBindu/Bindu](https://github.com/GetBindu/Bindu) | Capa de identidad/pagos para agentes, no de coordinación de código |
| 9.187 | [usekaneo/kaneo](https://github.com/usekaneo/kaneo) | Gestión de proyectos genérica sin IA |
| 8.229 | [basecamp/fizzy](https://github.com/basecamp/fizzy) | Kanban genérico sin IA |
| 8.011 | [YaoApp/yao](https://github.com/YaoApp/yao) | Motor de apps *low-code*; tablero de tareas genérico, no específico de código |
| 7.971 | [RayVentura/ShortGPT](https://github.com/RayVentura/ShortGPT) | Automatización de vídeo, dominio ajeno |
| 7.958 | [Upsonic/Upsonic](https://github.com/Upsonic/Upsonic) | Framework de agentes genérico |
| 7.435 | [algorithmicsuperintelligence/openevolve](https://github.com/algorithmicsuperintelligence/openevolve) | Programación evolutiva, no coordinación de equipo |
| 6.579 | [julep-ai/julep](https://github.com/julep-ai/julep) | Motor de ejecución duradera genérico, análogo a Temporal/Hatchet ya excluidos por B como infraestructura generalista |
| 6.561 | [airweave-ai/airweave](https://github.com/airweave-ai/airweave) | Capa de recuperación de contexto, no de coordinación |
| 6.322 | [KunAgent/Kun](https://github.com/KunAgent/Kun) | Espacio de trabajo de agente personal generalista |
| 6.267 | [kucherenko/jscpd](https://github.com/kucherenko/jscpd) | Detector de copia/pega, herramienta puntual |
| 5.997 | [google/agents-cli](https://github.com/google/agents-cli) | CLI para desplegar agentes en Google Cloud, no orquestador del circuito de código |
| 5.966 | [aiwaves-cn/agents](https://github.com/aiwaves-cn/agents) | Framework de investigación de agentes autoevolutivos |
| 5.938 | [spinabot/brigade](https://github.com/spinabot/brigade) | «Personal intelligence», generalista |
| 5.837 | [rllm-org/rllm](https://github.com/rllm-org/rllm) | RL para LLMs, dominio de investigación |
| 5.812 | [Klavis-AI/klavis](https://github.com/Klavis-AI/klavis) | Plataforma de integración MCP, infraestructura genérica |
| 5.376 | [cloudflare/vibesdk](https://github.com/cloudflare/vibesdk) | Plataforma de *vibe coding*, no orquestación multiagente |
| 4.616 | [nexu-io/html-video](https://github.com/nexu-io/html-video) | Generación de vídeo, utilidad puntual |
| 4.035 | [CommandCodeAI/command-code](https://github.com/CommandCodeAI/command-code) | Agente único |
| 3.783 | [tutti-os/tutti](https://github.com/tutti-os/tutti) | Descripción vaga, sin evidencia de patrón relevante |
| 3.714 | [JetBrains/go-modern-guidelines](https://github.com/JetBrains/go-modern-guidelines) | Guía de estilo para agentes, no herramienta |
| 3.671 | [Windy3f3f3f3f/how-claude-code-works](https://github.com/Windy3f3f3f3f/how-claude-code-works) | Material educativo |
| 3.481 | [superagent-ai/grok-cli](https://github.com/superagent-ai/grok-cli) | Agente de terminal único |
| 3.366 | [fuxicodex/Fuxi](https://github.com/fuxicodex/Fuxi) | Agente de terminal único |
| 3.179 | [ccch1mneyyy/dsh-TUI](https://github.com/ccch1mneyyy/dsh-TUI) | Plugin de TUI, utilidad puntual |
| 2.792 | [openlit/openlit](https://github.com/openlit/openlit) | Observabilidad de LLM, tangencial a C9, no a coordinación |
| 2.720 | [cocoindex-io/cocoindex-code](https://github.com/cocoindex-io/cocoindex-code) | Motor de búsqueda de código, utilidad puntual |
| 2.708 | [Windy3f3f3f3f/claude-code-from-scratch](https://github.com/Windy3f3f3f3f/claude-code-from-scratch) | Material educativo |
| 2.623 | [itayinbarr/little-coder](https://github.com/itayinbarr/little-coder) | Arnés para LLMs pequeños, agente único |
| 2.570 | [CoderLuii/HolyClaude](https://github.com/CoderLuii/HolyClaude) | Estación de trabajo personal, sin patrón de coordinación documentado |
| 2.543 | [webfuse-com/awesome-autoresearch](https://github.com/webfuse-com/awesome-autoresearch) | Lista curada |
| 2.501 | [softaworks/agent-toolkit](https://github.com/softaworks/agent-toolkit) | Colección de *skills* |
| 2.476 | [semanser/codel](https://github.com/semanser/codel) | Agente autónomo generalista |
| 2.290 | [Sahir619/fable-method](https://github.com/Sahir619/fable-method) | Destilado de técnicas de un modelo concreto, no herramienta de coordinación |
| 2.170 | [OpenCoworkAI/open-cowork](https://github.com/OpenCoworkAI/open-cowork) | App de escritorio de instalación de arneses, no orquestador propio |
| 2.090 | [4thfever/cultivation-world-simulator](https://github.com/4thfever/cultivation-world-simulator) | Simulador de videojuego, dominio ajeno |
| 1.907 | [melih-unsal/DemoGPT](https://github.com/melih-unsal/DemoGPT) | Generador de apps agénticas, dominio ajeno |
| 1.701 | [mims-harvard/ToolUniverse](https://github.com/mims-harvard/ToolUniverse) | Herramientas científicas, dominio ajeno |
| 1.637 | [jina-ai/langchain-serve](https://github.com/jina-ai/langchain-serve) | Infraestructura de despliegue genérica |
| 1.636 | [HumanSignal/Adala](https://github.com/HumanSignal/Adala) | Etiquetado de datos, dominio ajeno |
| 1.617 | [ray-r-ren/agent-apprenticeship](https://github.com/ray-r-ren/agent-apprenticeship) | Ecosistema de aprendizaje de agentes, no específico de código |
| 1.541 | [WecoAI/aideml](https://github.com/WecoAI/aideml) | Agente de ingeniería de ML, dominio ajeno |
| 1.472 | [ghostwright/phantom](https://github.com/ghostwright/phantom) | «AI co-worker with its own computer», generalista |
| 1.466 | [najmuzzaman-mohammad/gawkbot](https://github.com/najmuzzaman-mohammad/gawkbot) | Bot de automatización de tareas menores, dominio ajeno |
| 1.379 | [tmgthb/Autonomous-Agents](https://github.com/tmgthb/Autonomous-Agents) | Lista curada de papers |
| 567 | [bhouston/mycoder](https://github.com/bhouston/mycoder) | Agente de código único |
| 551 | [QuantaAlpha/RepoMaster](https://github.com/QuantaAlpha/RepoMaster) | Agente único, alternativa a Claude Code |

---

## 4. Preselección corregida

### 4.1 Corrección de método (encargo c)

La preselección de B justificaba el candidato **Beads** parcialmente con «por un autor reconocido (Steve Yegge...)». Eso no es evidencia según C13 (madurez y evidencia) ni según la regla de `circuito-tareas-definicion.md` §7 («las estrellas miden popularidad, no uso»; nada dice que la fama del autor sustituya evidencia de uso). Se retira esa frase; Beads se rejustifica solo por lo que hace (grafo de tareas con dependencias persistido en git, sobrevive a sesiones).

### 4.2 Preselección corregida (12 candidatos, objetivo 8–12)

Cobertura exigida: spec/descomposición, estado del trabajo/coordinación, orquestación/despacho de ejecutores con modelo intercambiable, topología, bucle de revisión, trazabilidad.

| # | Candidato | Cubre | Por qué entra (evidencia, no reputación) |
|---|---|---|---|
| 1 | **Claude Code — subagentes + agent teams** (nativo) | Topología, coordinación | Arnés que ya usa el usuario (Pro); *claim* con *file locking* real y verificado en agent teams (§2.3); documentación confirma que subagentes NO tienen cola compartida (corregido en §2.1, ya no por cita falsa sino por ausencia real del mecanismo) — línea base obligada, experimental |
| 2 | **GitHub Copilot coding agent + Issues/Projects v2** | Trazabilidad, encaje GitHub | Única combinación 100% nativa de GitHub (forja fijada); trazabilidad vía commits/logs verificada literalmente (§2.3) |
| 3 | **spec-kit** (GitHub oficial) | Spec/descomposición | Más adoptado con diferencia (138.822★); estándar de facto para /specify /plan /tasks |
| 4 | **OpenCode** | Orquestación con modelo intercambiable | Único arnés que documenta modelo-por-rol con **cualquier proveedor** (`provider/model-id`, verificado literal en §2.3) — encaje directo con la restricción DeepSeek |
| 5 | **awslabs/cli-agent-orchestrator** | Orquestación/despacho | Único orquestador oficial de un vendor grande que coordina varios arneses de CLI en sesiones aisladas |
| 6 | **OpenAI Symphony** (`openai/symphony`) | Coordinación, fallos | Spec de orquestación más formal encontrada en A+A2+B2: *claim* explícito, *backoff*, taxonomía de 5 clases de fallo. **Entra con reserva fuerte**: sin adopción real documentada, estado de orquestación en memoria (punto único de fallo documentado por el propio autor), crecimiento de estrellas sin poder verificar (API de *stargazers* bloqueada, repetido en B y A2) |
| 7 | **Beads** (`gastownhall/beads`) | Estado del trabajo | Grafo de tareas con dependencias persistido en git, entre sesiones — justificación **solo** técnica tras retirar la mención al autor |
| 8 | **Kiro** (AWS) | Topología (oleadas) | Único producto con documentación oficial de paralelismo por dependencias (*waves*), equivalente en producto real al patrón que describe el paper SPOQ |
| 9 | **humanlayer/humanlayer** (nuevo, adenda) | Bucle de revisión | Aprobación humana como capa explícita («human-in-the-loop»); mecanismo de revisión que ninguno de los candidatos anteriores cubre de forma dedicada |
| 10 | **open-multi-agent/open-multi-agent** (nuevo, adenda) | Trazabilidad | «Durable approvals and verifiable run records» — la pregunta peor resuelta en A+A2 (Q6, confirmado en A2 §4.6) tenía cero candidatos dedicados en la preselección original de B; este cubre ese hueco concreto |
| 11 | **SPOQ** (paper) | Topología, revisión | Único estudio con cifras propias sobre oleadas + validación dual + humano-como-agente; sin réplica independiente, tratar como hipótesis fuerte |
| 12 | **Co-Coder** (paper) | Topología | Compara directamente contra «Claude Code with Agent Teams» con cifras propias (verificado literal en el abstract, §2.3) — evidencia académica más pertinente a la pregunta exacta del usuario |

### 4.3 Exclusiones explícitas y por qué (corregidas y ampliadas)

- **affaan-m/ECC, ruvnet/ruflo** (B) y **Yeachan-Heo/oh-my-codex** (nuevo, §3.3): estrellas no explicadas por evidencia independiente — no se preseleccionan hasta aclararse.
- **herdrdev/herdr** (nuevo): señal débil no concluyente (§3.3) — mismo tratamiento que `gsd-build/get-shit-done` en B.
- **CrewAI, AutoGen/ag2, MetaGPT, AutoGPT, LangGraph, google/ax, rowboatlabs/rowboat, multica-ai/multica** (B + nuevos de la adenda): frameworks generalistas de agentes, no nacidos para el circuito código→PR→revisión sobre GitHub.
- **openclaw/openclaw y todo su ecosistema** (nuevo, §3.3): pese a ser, con 390.467★, **más popular que cualquier repo de la tabla de B incluido spec-kit**, queda excluido por alcance, no por adopción — es un agente de automatización general de escritorio, no un patrón de organización de equipo de desarrollo. Se deja constancia explícita para que quede claro que no es un olvido por tamaño.
- **Codex Cloud y Cursor Background Agents** (B, reafirmado tras la adenda): paralelismo real y bien documentado, pero cerrados y sin *claim*/*lease* documentado — sin cambios respecto a B.
- **Devin** (B, reafirmado): cerrado, sin repo público.
- **deepseek-ai/deepseek-harness** (nuevo): **no se excluye por descarte, se deja como candidato «a vigilar»** fuera de la preselección de 12. Es directamente relevante a la restricción de usar DeepSeek para implementar, con lanzamiento oficial bien corroborado (747 pts en HN) — pero en el tiempo disponible no se encontró evidencia de que resuelva un patrón multiagente Q1–Q6 más allá de ser un arnés de un solo agente; recomendado como primer punto de profundización si la fase C tiene margen.
- **Implementaciones repetidas del patrón Ralph Wiggum** (`frankbria/ralph-claude-code`, `mikeyobrien/ralph-orchestrator`, `michaelshimeles/ralphy`, `subsy/ralph-tui`, todas nuevas en la adenda, más `tzachbon/smart-ralph` ya en B): se documenta que el patrón está replicado de forma independiente en 5 repos ≥500★ más el propio reporte de OpenAI/Stripe en A2 — evidencia de que el patrón «reintento acotado antes de escalar» está bien contrastado — pero no se añade ninguna implementación individual a la preselección por redundancia entre sí.
- **La mayoría de la adenda relevante de §3.2** (Untrivial-ai/agent-orchestrator, getpaseo/paseo, superset-sh/superset, stablyai/orca, 21st-dev/1code, generalaction/emdash, AgentsMesh/AgentsMesh, zeronsh/zeron, Orkas-AI/Orkas, pingdotgg/t3code, manaflow-ai/cmux, AndyMik90/Aperant, openchamber/openchamber, andrewyng/openworker, AutoMaker-Org/automaker, coder/xum, SeemSeam/claude_codex_bridge, 777genius/agent-teams-ai, Q00/ouroboros, bmad-code-org/BMAD-METHOD, MrLesk/Backlog.md, BloopAI/vibe-kanban, gastownhall/gastown, smtg-ai/claude-squad, teaql/teaql-agent-kit, amElnagdy/delegate-skills, the-open-engine/zeroshot): confirman que el patrón «capa de despacho multi-arnés con modelo/proveedor intercambiable» está ampliamente replicado (decenas de implementaciones ≥500★ en 2026), reforzando la restricción de «ejecutor intercambiable» del usuario como algo que el mercado ya da por sentado — pero son redundantes funcionalmente con OpenCode/`awslabs/cli-agent-orchestrator` ya preseleccionados. No se añaden individualmente; quedan documentados en §3.2 para fase C si se necesita un segundo ejemplo del mismo patrón.
- **Amp**: sin cambios respecto a B — documentación pública insuficiente sobre *task-tracking* con nombre propio, hueco declarado.
- **anthropics/claude-code-action, github/gh-aw, google-github-actions/run-gemini-cli** (nuevos, nativos, §3.2): no entran en la preselección de 12 porque no son un *patrón* nuevo sino una vía de activación (GitHub Actions) de arneses ya preseleccionados (Claude Code, Gemini CLI) — pero se marca como **corrección obligatoria de B §3**, que documentó funciones nativas de arneses sin cubrir su integración oficial vía Actions, relevante directamente a C11.

---

## 5. Bloqueos

| Bloqueo | Efecto | Cómo se sorteó / qué queda pendiente |
|---|---|---|
| `RyanAlberts/best-of-Agent-Harnesses` (167 arneses catalogados) y `e2b-dev/awesome-ai-agents` no se minaron línea por línea | Posible cobertura adicional no capturada | Identificados y clasificados como meta-fuente en la cola larga (§3.3); pendiente de minado si fase C lo requiere |
| 104 repos de la cola larga (§3.3) clasificados solo por su descripción oficial de GitHub, no por lectura de README | Menor profundidad que el resto del inventario (y que B, que sí escribió una frase propia por repo) | Ninguno de los 104 mostró, por su descripción, relación con Q1–Q6; si la fase C necesita descartar esta posibilidad con más rigor, requiere una pasada adicional |
| API de *stargazers* de GitHub (`Accept: application/vnd.github.star+json`) sigue devolviendo 404 (confirmado de nuevo aquí sobre `openai/symphony` y `deepseek-ai/deepseek-harness`) | No se puede verificar si el crecimiento de estrellas de los candidatos más grandes de la adenda es orgánico | Sustituido por el mismo método indirecto de B §6 (velocidad, ratios, cobertura HN); repetir con el método original si se reabre el bloqueo |
| `deepseek-ai/deepseek-harness` no se investigó en profundidad más allá de metadatos + cobertura HN | Es el candidato más directamente ligado a la restricción DeepSeek del usuario y queda solo como «a vigilar», no evaluado | Recomendado como primer punto de fase C si hay margen de tiempo |
| WebSearch agotado (heredado) | Sin búsqueda web general | `gh`, `WebFetch`, `curl` y GraphQL cubrieron el encargo |
| No se navegó (Chrome/CDP no autorizado en esta subtarea) | N/A — no se necesitó para verificar ninguna de las páginas de este encargo (todas accesibles por `curl`/`WebFetch`) | — |

---

## Enlaces

- [[circuito-tareas-definicion]] — encargo y criterios que sigue este documento
- `B-implementaciones.md` — documento corregido aquí
- `A2-practica-a-escala.md` — origen del hueco de `openai/symphony`
- [[astillero]] — proyecto
- [[desarrollo-agentes-investigacion]] — investigación previa del ciclo completo
-

## Enlaces

- [[flujo-agentes-informe]] — síntesis de la investigación
- [[circuito-tareas-definicion]] — definición de la fase
- [[_index]]
