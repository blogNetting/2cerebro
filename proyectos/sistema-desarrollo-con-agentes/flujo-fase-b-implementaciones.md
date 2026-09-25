---
title: Flujo con agentes, fase B: herramientas
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, herramientas, inventario, investigacion]
zona: tecnico
---

Inventario de herramientas, productos y funciones nativas de los arneses que implementan la organización del trabajo con agentes.

Informe de fase; lo ha revisado el orquestador. Síntesis en [[flujo-agentes-informe]]. Las correcciones del orquestador están en esa síntesis.

Barrido sistemático hecho en sesión única, sin subagentes ni navegador (por encargo explícito de esta subtarea), con cupo de WebSearch agotado: se usó `gh` (CLI oficial de GitHub), `WebFetch` (lector de páginas) y `curl` contra APIs públicas. Fecha: 2026-09-25. Sigue [[circuito-tareas-definicion]] (6 preguntas Q1–Q6, criterios C1–C13, definición de «contrastado» en su §7).

**Reutilización del material anulado:** se reutilizaron como materia prima bruta (re-verificados aquí) `sweep.tsv` y `F1-descubrimiento.md` de la sesión F1, cuyo resultado quedó anulado por sesgo y huecos. Se corrigieron explícitamente sus dos defectos conocidos: (1) el hallazgo de *agent teams* de Claude Code, que F1 no vio, se documenta en la §3 con la doc oficial; (2) se añadieron por `gh api` directo los repos de alta adopción que F1 mencionó en prosa pero no dejó en el barrido bruto reproducible (CrewAI, AutoGen/ag2, MetaGPT, OpenHands, goose, Temporal, Hatchet, OpenCode, Codex, Gemini CLI, Copilot CLI, claude-task-master, Archon, beads, cli-agent-orchestrator, LangGraph, AutoGPT, aider, Rivet, TanStack Workflow, dbos durable-swarm).

---

## 1. Método y consultas

### GitHub (`gh search repos "<consulta>" --sort stars --limit 50 --json fullName,stargazersCount,pushedAt,createdAt,description`)

**Heredadas de F1 (reutilizadas, re-verificadas por fecha de sesión):** agent harness, agent swarm coding, agent task tracker, agent workflow durable, agentic software engineering, agents kanban, ai code review agent, ai software engineer, autonomous coding agent, background agents, claude code agents parallel, claude code orchestrator, coding agent review, coding agent task, coding agents orchestration, git worktree agents, headless coding agent, issue tracker agents, multi-agent coding, parallel coding agents, spec-driven development, task management ai agents. Ninguna vacía (4–40 resultados cada una).

**Nuevas en esta fase**, del encargo explícito ("linear agent", "codex orchestration", "agent team coding", "subagents", "coding agent supervisor", "ticket to pull request", "issue to pull request agent", "SWE agent", "agent manager worktree", "orchestrate claude code", "swarm coding", "issues agent autonomous") más mineo de listas curadas ("awesome claude code", "awesome ai agents orchestration", "awesome multi agent", "awesome spec driven development", "claude code agent teams", "agent worktree parallel", "autonomous pull request agent", "kanban ai agents"):

| Consulta | Resultados |
|---|---|
| linear agent | 50 |
| codex orchestration | 50 |
| agent team coding | 3 |
| subagents | 50 |
| coding agent supervisor | 9 |
| ticket to pull request | 4 |
| **issue to pull request agent** | **0 (vacía)** |
| SWE agent | 50 |
| **agent manager worktree** | **0 (vacía)** |
| orchestrate claude code | 50 |
| swarm coding | 50 |
| **issues agent autonomous** | **0 (vacía)** |
| awesome claude code | 50 |
| **awesome ai agents orchestration** | **0 (vacía)** |
| awesome multi agent | 32 |
| awesome spec driven development | 5 |
| claude code agent teams | 50 |
| **agent worktree parallel** | **0 (vacía)** |
| autonomous pull request agent | 1 |
| kanban ai agents | 4 |

Cinco consultas devolvieron cero resultados; se listan tal cual pide el encargo, no se ocultan.

**`gh api repos/<owner>/<repo>`** puntual sobre 40 repos nombrados explícitamente (grandes frameworks/vendors que no salían con las consultas de texto libre): `crewAIInc/crewAI`, `microsoft/autogen`, `ag2ai/ag2`, `FoundationAgents/MetaGPT`, `OpenHands/OpenHands`, `aaif-goose/goose` (= `block/goose`), `temporalio/temporal`, `hatchet-dev/hatchet`, `sst/opencode` (redirige a `anomalyco/opencode`), `openai/codex`, `google-gemini/gemini-cli`, `github/copilot-cli`, `eyaltoledano/claude-task-master`, `coleam00/Archon`, `gastownhall/beads`, `awslabs/cli-agent-orchestrator`, `langchain-ai/langgraph`, `Significant-Gravitas/AutoGPT`, `Aider-AI/aider`, `princeton-nlp/SWE-agent`, `rivet-dev/rivet`, `TanStack/workflow`, `dbos-inc/durable-swarm`, más 18 repos de la cola larga cuya fecha de creación/actividad hacía falta para el control de anomalías (§6).

Consolidado en un único TSV, deduplicado por `fullName` (quedándose con el máximo de estrellas visto por si una consulta reportaba un número desactualizado en caché): **1.115 filas brutas → 1.092 repos únicos → 155 con ≥500★**.

### Control de estrellas infladas
`gh api -H "Accept: application/vnd.github.star+json" repos/<owner>/<repo>/stargazers` — **endpoint bloqueado en este entorno: devuelve 404 en todas las páginas y para cualquier repo probado**, incluido un repo de control bien conocido (`github/spec-kit`). Confirmado con `-i` (headers completos, HTTP/2.0 404, sin mensaje de scope insuficiente). Se documenta como bloqueo de herramienta en §8, y se sustituyó por una combinación de: velocidad de crecimiento (estrellas / meses desde `created_at`), ratio *forks*/estrellas y *subscribers*/estrellas (`gh api repos/<repo>`), estado de Issues (`has_issues`, nº de issues abiertas), y cobertura independiente en Hacker News (`curl "https://hn.algolia.com/api/v1/search?query=<término>&tags=story"`).

### Funciones nativas de arneses
`WebFetch` sobre documentación oficial de Claude Code (`code.claude.com/docs/en/agent-teams` ya guardado en HTML por la sesión anterior, más `sub-agents`, `hooks`, `agent-sdk/overview`), Codex (`learn.chatgpt.com/docs/cloud`), GitHub Copilot coding agent (`docs.github.com`), Cursor (`cursor.com/docs/background-agent`), OpenCode (`opencode.ai/docs/`, `opencode.ai/docs/agents/`), Gemini CLI (ficheros `.md` crudos del repo vía `raw.githubusercontent.com`, dado que `geminicli.com` devolvió 404 en la ruta probada), Amp (`ampcode.com/manual`), Kiro (`kiro.dev/docs/specs/`, HTML crudo vía `curl`). Para Claude Code *agent teams* se usó también el HTML ya guardado (`cc-agent-teams.html`), parseado con `python3`/`re` porque el HTML crudo de Mintlify no se renderiza con grep simple (contenido embebido en el DOM final, no en JS de arranque).

### Académico
`export.arxiv.org/api/query` vía `WebFetch` (no vía `curl` directo, que sigue devolviendo 406 en este entorno tal como reportó F1) — **funcionó** cuando se accedió a través de `WebFetch`, a diferencia del intento por `curl` puro. Cada paper encontrado se verificó además en su página `arxiv.org/abs/<id>` directa.

---

## 2. Tabla consolidada — todo repo ≥500★ encontrado, clasificado

70 relevantes / 85 irrelevantes, sobre 155 repos únicos con ≥500★. Ordenada por relevancia y luego por estrellas descendente. "Preguntas (§1)" remite a las seis preguntas de [[circuito-tareas-definicion]] (Q1 roles/división, Q2 roles y reparto, Q3 coordinación/estado, Q4 topología, Q5 revisión, Q6 trazabilidad — aquí se anota la más aplicable; muchos repos tocan varias). El motivo de exclusión de cada irrelevante evita que quede «caído silenciosamente».

| ★ | Repo | Clasificación | Motivo | Preguntas (§1) |
|---|---|---|---|---|
| 138.822 | [github/spec-kit](https://github.com/github/spec-kit) | **Relevante** | Toolkit oficial de GitHub para /specify /plan /tasks /implement; genera tasks.md verificable | Q1 |
| 89.128 | [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | **Relevante** | Plataforma de agentes de código (ex-OpenDevin); incluye resolución de issues y modo cloud | Q4,Q5 |
| 70.602 | [FoundationAgents/MetaGPT](https://github.com/FoundationAgents/MetaGPT) | **Relevante** | Simula roles PM/arquitecto/ingeniero/QA sobre una tarea en lenguaje natural, cubre todo el circuito | Q1,Q2,Q3,Q4,Q5 |
| 70.280 | [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) | **Relevante** | Spec-driven development (SDD) para asistentes de código | Q1 |
| 64.458 | [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) | **Relevante** | Sistema de meta-prompting/context engineering dirigido por spec para agentes autónomos | Q1 |
| 61.150 | [microsoft/autogen](https://github.com/microsoft/autogen) | **Relevante** | Framework de programación agéntica de Microsoft Research, orquestación genérica | Q3,Q4 |
| 59.004 | [crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) | **Relevante** | Framework de orquestación de agentes con roles (*role-playing*) explícitos, generalista | Q2,Q3 |
| 42.253 | [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | **Relevante** | Infra de bajo nivel (grafo) para construir agentes resilientes, usada para orquestar multiagente | Q3,Q4 |
| 29.760 | [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents) | **Relevante** | Implementación de LangChain del patrón "deep agents" (planificador+subagentes+FS), inspirado en Claude Code | Q2,Q4 |
| 28.086 | [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) | **Relevante** | Sistema de gestión de tareas con IA para Cursor/Windsurf/otros | Q1,Q3 |
| 27.410 | [gastownhall/beads](https://github.com/gastownhall/beads) | **Relevante** | Grafo de tareas con dependencias persistido en git, memoria de tareas entre sesiones de agente | Q2,Q3 |
| 27.085 | [xai-org/grok-build](https://github.com/xai-org/grok-build) | **Relevante** | CLI de agente de código de xAI (Grok) (ver §4 vendors y §6 anomalías) | — |
| 23.550 | [coleam00/Archon](https://github.com/coleam00/Archon) | **Relevante** | "Harness builder" para hacer el código con IA determinista y repetible | Q1 |
| 23.286 | [temporalio/temporal](https://github.com/temporalio/temporal) | **Relevante** | Motor de ejecución duradera de propósito general, usado ampliamente para orquestar agentes como workflows | Q3,Q8 |
| 22.401 | [winfunc/opcode](https://github.com/winfunc/opcode) | **Relevante** | GUI/Toolkit para Claude Code: crea agentes personalizados, gestiona sesiones interactivas de CC | Q4,Q7 |
| 20.402 | [SWE-agent/SWE-agent](https://github.com/SWE-agent/SWE-agent) | **Relevante** | Toma un issue de GitHub y lo arregla automáticamente; agente único, referencia de línea base para revisión (issue→PR) | Q5 |
| 20.402 | [princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent) | **Relevante** | Nombre de org original antes de transferirse a `SWE-agent/SWE-agent`; mismo repo/hallazgo | Q5 |
| 16.641 | [HKUDS/DeepCode](https://github.com/HKUDS/DeepCode) | **Relevante** | "Agent Harness & Loop Engineering & Multi-Agent Orchestration" explícito | Q3,Q4 |
| 15.239 | [yc-software/qm](https://github.com/yc-software/qm) | **Relevante** | "Multiplayer agent harness for work" (Y Combinator), en Slack y web — crecimiento muy rápido pero verificado (§6) | Q3,Q4 |
| 11.071 | [aden-hive/hive](https://github.com/aden-hive/hive) | **Relevante** | Harness multiagente para IA en producción — crecimiento muy rápido, ver §6 | Q3,Q4 |
| 10.221 | [omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent) | **Relevante** | Meta-harness que orquesta Claude Code/Codex/Cursor/Pi; políticas, sandboxing y colaboración en tiempo real — producto de Databricks, ver §6 | Q3,Q4 |
| 8.003 | [hatchet-dev/hatchet](https://github.com/hatchet-dev/hatchet) | **Relevante** | Motor de orquestación para tareas en segundo plano, agentes de IA y workflows duraderos con colas y reintentos | Q3,Q8 |
| 7.960 | [SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) | **Relevante** | Versión minimalista de SWE-agent, mismo patrón issue→PR | Q5 |
| 7.781 | [gsd-build/gsd-2](https://github.com/gsd-build/gsd-2) | **Relevante** | Sistema de meta-prompting/context engineering SDD para trabajo autónomo largo | Q1 |
| 6.170 | [rivet-dev/rivet](https://github.com/rivet-dev/rivet) | **Relevante** | Primitivas de "actores" con estado: agentes de IA, apps colaborativas, ejecución duradera | Q3,Q4 |
| 5.446 | [buildermethods/agent-os](https://github.com/buildermethods/agent-os) | **Relevante** | Sistema para inyectar estándares del repo y escribir mejores specs para SDD | Q1 |
| 4.955 | [ag2ai/ag2](https://github.com/ag2ai/ag2) | **Relevante** | Continuación comunitaria de AutoGen tras fork; activo donde el original no lo está | Q3,Q4 |
| 4.548 | [phodal/auto-dev](https://github.com/phodal/auto-dev) | **Relevante** | Plataforma de desarrollo multiagente en Kotlin Multiplatform | Q2,Q3,Q4 |
| 4.359 | [Doriandarko/maestro](https://github.com/Doriandarko/maestro) | **Relevante** | Framework temprano (2024) para que Claude Opus orqueste subagentes | Q2,Q4 |
| 4.294 | [Pimzino/spec-workflow-mcp](https://github.com/Pimzino/spec-workflow-mcp) | **Relevante** | Servidor MCP que da herramientas de workflow SDD estructurado a agentes | Q1 |
| 3.948 | [matt1398/claude-devtools](https://github.com/matt1398/claude-devtools) | **Relevante** | DevTools para Claude Code: logs de sesión, llamadas a herramientas, uso de tokens, subagentes | Q7,Q9 |
| 3.858 | [Pimzino/claude-code-spec-workflow](https://github.com/Pimzino/claude-code-spec-workflow) | **Relevante** | Workflow automatizado para Claude Code: Requisitos → Diseño → Tareas | Q1 |
| 3.762 | [nicobailon/pi-subagents](https://github.com/nicobailon/pi-subagents) | **Relevante** | Extensión de Pi para delegación asíncrona de subagentes | Q4 |
| 3.746 | [gemini-cli-extensions/conductor](https://github.com/gemini-cli-extensions/conductor) | **Relevante** | Plugin oficial de Google (extensión Gemini CLI) para SDD | Q1 |
| 3.683 | [gotalab/cc-sdd](https://github.com/gotalab/cc-sdd) | **Relevante** | Harness SDD mínimo multi-arnés (CC, Codex, Cursor, Copilot, Windsurf, OpenCode, Gemini CLI, Antigravity) | Q1 |
| 3.280 | [ColeMurray/background-agents](https://github.com/ColeMurray/background-agents) | **Relevante** | Sistema de código de agentes en segundo plano | Q3,Q4 |
| 2.600 | [HarnessRouter/harnessrouter](https://github.com/HarnessRouter/harnessrouter) | **Relevante** | Interfaz unificada autoalojada para múltiples arneses de agentes | Q4 |
| 1.656 | [patoles/agent-flow](https://github.com/patoles/agent-flow) | **Relevante** | Visualización en tiempo real de la orquestación de agentes de Claude Code | Q4,Q7 |
| 1.642 | [spec-kitty/spec-kitty](https://github.com/spec-kitty/spec-kitty) | **Relevante** | SDD con kanban, git worktrees y auto-merge declarados explícitamente | Q1,Q3,Q4 |
| 1.606 | [donvito/codex-astra-luna-orchestrator](https://github.com/donvito/codex-astra-luna-orchestrator) | **Relevante** | Patrón orquestador (Astra/Sol) + subagentes (Luna) dentro de Codex | Q2,Q4 |
| 1.553 | [zhnt/loushang](https://github.com/zhnt/loushang) | **Relevante** | Harness de IA para workflows de código: orquestación multi-modelo, sesiones con estado | Q3 |
| 1.451 | [Danau5tin/multi-agent-coding-system](https://github.com/Danau5tin/multi-agent-coding-system) | **Relevante** | Orquestador + agentes explorador/programador; #13 en Terminal-Bench de Stanford | Q2,Q3,Q4 |
| 1.424 | [yohey-w/multi-agent-shogun](https://github.com/yohey-w/multi-agent-shogun) | **Relevante** | Topología jerárquica explícita (shogun→karo→ashigaru) sobre tmux para Claude Code | Q4 |
| 1.349 | [awslabs/cli-agent-orchestrator](https://github.com/awslabs/cli-agent-orchestrator) | **Relevante** | Orquestación multiagente oficial de AWS Labs para CLIs de código en tmux aisladas | Q3,Q4 |
| 1.299 | [gastownhall/gascity](https://github.com/gastownhall/gascity) | **Relevante** | SDK "orchestration-builder" para workflows de código multiagente | Q3,Q4 |
| 1.262 | [open-gsd/gsd-pi](https://github.com/open-gsd/gsd-pi) | **Relevante** | Fork activo de gsd-2 (posible señal de fricción/desacuerdo en la comunidad, sin verificar la causa) | Q1 |
| 1.244 | [Gentleman-Programming/agent-teams-lite](https://github.com/Gentleman-Programming/agent-teams-lite) | **Relevante** | SDD con orquestador + 9 subagentes especializados | Q1,Q2 |
| 1.215 | [tintinweb/pi-subagents](https://github.com/tintinweb/pi-subagents) | **Relevante** | Subagentes/orquestación de workflow para Pi: ejecución paralela, fleet view | Q4,Q7 |
| 1.047 | [context-labs/whip](https://github.com/context-labs/whip) | **Relevante** | Harness en Go: subagentes en segundo plano, modelos enrutables por proveedor | Q4 |
| 1.014 | [hoangsonww/Claude-Code-Agent-Monitor](https://github.com/hoangsonww/Claude-Code-Agent-Monitor) | **Relevante** | Dashboard en tiempo real para CC/Codex: sesiones, tablero kanban, orquestación de subagentes | Q7 |
| 1.012 | [Human-Agent-Society/CORAL](https://github.com/Human-Agent-Society/CORAL) | **Relevante** | Ejecuta Claude Code/OpenCode/Codex con calificación comparativa | Q6 |
| 974 | [zhu1090093659/spec_driven_develop](https://github.com/zhu1090093659/spec_driven_develop) | **Relevante** | Workflow SDD: planificación arquitectura-primero, descomposición de tareas, integración GitHub | Q1 |
| 972 | [cabloy/cabloy](https://github.com/cabloy/cabloy) | **Relevante** | Framework Node.js fullstack con SDD para desarrollo trazable | Q1,Q7 |
| 904 | [JohnRiceML/clawport-ui](https://github.com/JohnRiceML/clawport-ui) | **Relevante** | Centro de mando para "Claude Code agent teams" (función nativa, ver §3) | Q3,Q7 |
| 825 | [cyrusagents/cyrus](https://github.com/cyrusagents/cyrus) | **Relevante** | Agente de fondo de Claude Code para Linear/Slack/GitHub/GitLab; soporta Codex | Q2,Q3 |
| 768 | [Ark0N/Codeman](https://github.com/Ark0N/Codeman) | **Relevante** | Mission control autoalojado multi-arnés 24/7, ve cada subagente en vivo | Q3,Q4,Q7 |
| 744 | [kaochenlong/spectra-app](https://github.com/kaochenlong/spectra-app) | **Relevante** | App+CLI+skills SDD multi-arnés | Q1 |
| 732 | [DenisSergeevitch/repo-task-proof-loop](https://github.com/DenisSergeevitch/repo-task-proof-loop) | **Relevante** | Skill dirigida por spec con generación de subagentes | Q1 |
| 710 | [HazAT/pi-interactive-subagents](https://github.com/HazAT/pi-interactive-subagents) | **Relevante** | Subagentes interactivos para Pi en terminales cmux | Q4 |
| 686 | [shotgun-sh/shotgun](https://github.com/shotgun-sh/shotgun) | **Relevante** | SDD: specs conscientes del repo | Q1 |
| 642 | [langtalks/swe-agent](https://github.com/langtalks/swe-agent) | **Relevante** | Multiagente investigador+desarrollador con LangGraph | Q2,Q3 |
| 629 | [Cjbuilds/Codex-Orchestration](https://github.com/Cjbuilds/Codex-Orchestration) | **Relevante** | Asigna cualquier modelo a cualquier rol dentro de Codex | Q2 |
| 629 | [mouredev/hello-sdd](https://github.com/mouredev/hello-sdd) | **Relevante** | Curso educativo sobre SDD (material de referencia, no herramienta) | Q1 |
| 621 | [onevcat/Prowl](https://github.com/onevcat/Prowl) | **Relevante** | Orquestador nativo de macOS para agentes de código | Q4 |
| 603 | [SWE-agent/SWE-ReX](https://github.com/SWE-agent/SWE-ReX) | **Relevante** | Ejecución en sandbox para agentes, masivamente paralela | Q4 |
| 550 | [tzachbon/smart-ralph](https://github.com/tzachbon/smart-ralph) | **Relevante** | SDD con compactación, patrón de bucle "Ralph Wiggum" | Q1,Q8 |
| 538 | [QuintinShaw/pi-dynamic-workflows](https://github.com/QuintinShaw/pi-dynamic-workflows) | **Relevante** | Workflows estilo CC para Pi: modelo real, aislamiento worktree, contabilidad de coste | Q3,Q4,Q9 |
| 535 | [ObservedObserver/async-code](https://github.com/ObservedObserver/async-code) | **Relevante** | CC/Codex para tareas en paralelo, UI estilo Codex | Q3,Q4 |
| 535 | [tommy0103/obelisk](https://github.com/tommy0103/obelisk) | **Relevante** | Historial de sesión/subagente/workflow consultable | Q7 |
| 506 | [qiz029/dscode](https://github.com/qiz029/dscode) | **Relevante** | Harness de agente sobre DeepSeek (ejecutor relevante para el rol de implementación con DeepSeek, no aporta patrón de organización propio) | — |
| 267.138 | [affaan-m/ECC](https://github.com/affaan-m/ECC) | Irrelevante | Descripción genérica, sin cobertura HN encontrada, crecimiento extremo — no se trata como candidato hasta verificarse (§6) | — |
| 209.939 | [anomalyco/opencode](https://github.com/anomalyco/opencode) | Irrelevante | Agente open source multi-modelo (antes sst/opencode); tiene subagentes con modelo por rol — ver §3 | — |
| 187.544 | [Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) | Irrelevante | Plataforma de agente autónomo generalista, no específica de desarrollo en equipo | — |
| 126.387 | [openai/codex](https://github.com/openai/codex) | Irrelevante | CLI de agente de código, ejecutor único (Codex Cloud sí tiene paralelismo, documentado aparte en §3) | — |
| 107.158 | [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) | Irrelevante | Sin subagentes/orquestación multiagente nativa documentada (§3) | — |
| 82.959 | [bytedance/deer-flow](https://github.com/bytedance/deer-flow) | Irrelevante | "SuperAgent harness" ampliable con subagentes, pero no documenta patrón de roles/coordinación para desarrollo | — |
| 77.587 | [shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) | Irrelevante | Material educativo que reconstruye Claude Code, no una herramienta de organización | — |
| 73.236 | [ruvnet/ruflo](https://github.com/ruvnet/ruflo) | Irrelevante | Cobertura HN independiente casi nula (máx. 3 puntos, un post pregunta si alguien lo usa) frente a 73k★ — ver §6 | — |
| 69.273 | [cline/cline](https://github.com/cline/cline) | Irrelevante | Agente IDE/SDK/CLI de un solo agente, sin patrón multi-rol | — |
| 54.634 | [aaif-goose/goose](https://github.com/aaif-goose/goose) | Irrelevante | Agente de Block/Square, un solo agente por sesión | — |
| 54.586 | [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) | Irrelevante | Lista curada (meta-fuente), no herramienta | — |
| 49.178 | [Aider-AI/aider](https://github.com/Aider-AI/aider) | Irrelevante | Pair programming en terminal, agente único | — |
| 47.111 | [zhayujie/CowAgent](https://github.com/zhayujie/CowAgent) | Irrelevante | Asistente generalista autoevolutivo, sin patrón de coordinación | — |
| 40.104 | [tinyhumansai/openhuman](https://github.com/tinyhumansai/openhuman) | Irrelevante | Harness de agente único, generalista | — |
| 39.933 | [wshobson/agents](https://github.com/wshobson/agents) | Irrelevante | Marketplace de plugins/subagentes (colección, no patrón propio) | — |
| 25.314 | [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | Irrelevante | Colección de +100 prompts de subagentes temáticos | — |
| 23.078 | [micro/go-micro](https://github.com/micro/go-micro) | Irrelevante | Framework de microservicios en Go; "agent" no se refiere a IA — falso positivo | — |
| 15.859 | [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness) | Irrelevante | Arnés de agente personal único ("Ohmo"), no multiagente | — |
| 14.803 | [mindfold-ai/Trellis](https://github.com/mindfold-ai/Trellis) | Irrelevante | Descripción genérica, sin cobertura externa encontrada | — |
| 11.209 | [github/copilot-cli](https://github.com/github/copilot-cli) | Irrelevante | Da acceso al "coding agent" cloud (función nativa, ver §3) | — |
| 8.362 | [strands-agents/harness-sdk](https://github.com/strands-agents/harness-sdk) | Irrelevante | SDK genérico de AWS para agentes de producción, no específico de organización de desarrollo | — |
| 7.928 | [chaitanyagiri/munder-difflin](https://github.com/chaitanyagiri/munder-difflin) | Irrelevante | Adopción mínima, sin evidencia más allá de su propia descripción | — |
| 6.738 | [zai-org/ZCode](https://github.com/zai-org/ZCode) | Irrelevante | Harness de agente único de Z.ai (producto de vendor) | — |
| 6.235 | [VoltAgent/awesome-codex-subagents](https://github.com/VoltAgent/awesome-codex-subagents) | Irrelevante | Ídem, para Codex | — |
| 5.958 | [truefoundry/trueforge](https://github.com/truefoundry/trueforge) | Irrelevante | Runtime genérico LLM→agente | — |
| 5.649 | [vastsa/PI-Desktop](https://github.com/vastsa/PI-Desktop) | Irrelevante | App de escritorio para un solo agente Pi | — |
| 5.217 | [kodu-ai/claude-coder](https://github.com/kodu-ai/claude-coder) | Irrelevante | Extensión VSCode, agente único | — |
| 5.152 | [FailproofAI/failproofai](https://github.com/FailproofAI/failproofai) | Irrelevante | Observabilidad de arneses (tangencial a C9, no a roles/coordinación) | — |
| 5.121 | [Waishnav/devspace](https://github.com/Waishnav/devspace) | Irrelevante | Harness mínimo sobre MCP, agente único | — |
| 4.964 | [campfirein/byterover-cli](https://github.com/campfirein/byterover-cli) | Irrelevante | Capa de memoria portable, no roles/coordinación | — |
| 4.517 | [ai-boost/awesome-harness-engineering](https://github.com/ai-boost/awesome-harness-engineering) | Irrelevante | Lista curada (meta-fuente) | — |
| 4.273 | [lintsinghua/claude-code-book](https://github.com/lintsinghua/claude-code-book) | Irrelevante | Libro sobre arquitectura de Claude Code, referencia | — |
| 3.630 | [code-yeongyu/lazycodex](https://github.com/code-yeongyu/lazycodex) | Irrelevante | Harness único con memoria de proyecto, sin patrón multiagente | — |
| 3.540 | [davepoon/buildwithclaude](https://github.com/davepoon/buildwithclaude) | Irrelevante | Hub/directorio, índice no herramienta | — |
| 3.513 | [onecli/onecli](https://github.com/onecli/onecli) | Irrelevante | "Un agente por empleado", agente individual | — |
| 3.153 | [peteromallet/desloppify](https://github.com/peteromallet/desloppify) | Irrelevante | Harness de un solo propósito (calidad de código) | — |
| 3.060 | [NVlabs/SoL-Pi](https://github.com/NVlabs/SoL-Pi) | Irrelevante | Investigación NVIDIA sobre escalado de bucles, no herramienta de organización | — |
| 3.046 | [wesammustafa/Claude-Code-Everything-You-Need-to-Know](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know) | Irrelevante | Guía educativa, no herramienta | — |
| 2.978 | [QwenLM/Qwen-MM-Plugins](https://github.com/QwenLM/Qwen-MM-Plugins) | Irrelevante | Plugin multimodal, no organización de roles | — |
| 2.799 | [visa/visa-vulnerability-agentic-harness](https://github.com/visa/visa-vulnerability-agentic-harness) | Irrelevante | Agente interno de Visa, dominio de seguridad específico | — |
| 2.681 | [zubair-trabzada/ai-marketing-claude](https://github.com/zubair-trabzada/ai-marketing-claude) | Irrelevante | Subagentes de marketing, dominio ajeno | — |
| 2.644 | [centminmod/my-claude-code-setup](https://github.com/centminmod/my-claude-code-setup) | Irrelevante | Plantilla de configuración personal | — |
| 2.642 | [rohitg00/awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) | Irrelevante | Colección/índice | — |
| 2.380 | [google-antigravity/antigravity-cli](https://github.com/google-antigravity/antigravity-cli) | Irrelevante | CLI del harness Antigravity de Google, agente único (ver producto vendor §4) | — |
| 2.350 | [DenisSergeevitch/agents-best-practices](https://github.com/DenisSergeevitch/agents-best-practices) | Irrelevante | Guía de buenas prácticas, no herramienta | — |
| 2.269 | [0xSteph/pentest-ai-agents](https://github.com/0xSteph/pentest-ai-agents) | Irrelevante | Subagentes de pentesting, dominio ajeno | — |
| 2.261 | [Infisical/agent-vault](https://github.com/Infisical/agent-vault) | Irrelevante | Vault de credenciales, infra de seguridad | — |
| 2.115 | [peteromallet/dataclaw](https://github.com/peteromallet/dataclaw) | Irrelevante | Publica historial de chat como dataset HF | — |
| 2.077 | [maxritter/pilot-shell](https://github.com/maxritter/pilot-shell) | Irrelevante | Guía de "context engineering", agente único | — |
| 2.043 | [iannuttall/claude-agents](https://github.com/iannuttall/claude-agents) | Irrelevante | Colección personal de subagentes | — |
| 1.982 | [deusyu/translate-book](https://github.com/deusyu/translate-book) | Irrelevante | Subagentes de traducción, dominio ajeno | — |
| 1.982 | [glittercowboy/taches-cc-resources](https://github.com/glittercowboy/taches-cc-resources) | Irrelevante | Colección personal de recursos | — |
| 1.981 | [AntigmaLabs/ante](https://github.com/AntigmaLabs/ante) | Irrelevante | Harness tipo Claude Code, agente único | — |
| 1.893 | [unreallabsai/unreal-agent](https://github.com/unreallabsai/unreal-agent) | Irrelevante | Harness async-first, agente único | — |
| 1.835 | [ShenSeanChen/waku-agent](https://github.com/ShenSeanChen/waku-agent) | Irrelevante | Harness local-first, agente único | — |
| 1.802 | [Picrew/awesome-agent-harness](https://github.com/Picrew/awesome-agent-harness) | Irrelevante | Lista curada (meta-fuente) | — |
| 1.690 | [kyegomez/awesome-multi-agent-papers](https://github.com/kyegomez/awesome-multi-agent-papers) | Irrelevante | Lista curada de papers (meta-fuente académica) | — |
| 1.689 | [lst97/claude-code-sub-agents](https://github.com/lst97/claude-code-sub-agents) | Irrelevante | Colección de subagentes, uso personal | — |
| 1.581 | [langchain-ai/deepagentsjs](https://github.com/langchain-ai/deepagentsjs) | Irrelevante | Versión JS de deepagents, mismo patrón ya cubierto | — |
| 1.548 | [achimala/dream-loop](https://github.com/achimala/dream-loop) | Irrelevante | Subagente para gráficos 3D, dominio ajeno | — |
| 1.460 | [evo-hq/evo](https://github.com/evo-hq/evo) | Irrelevante | Investigación de benchmarks, no organización de desarrollo con roles | — |
| 1.450 | [exoharness/exo](https://github.com/exoharness/exo) | Irrelevante | Agente autoeditable/recursivo, no roles/coordinación | — |
| 1.290 | [Spielewoy/autoprompt-skill](https://github.com/Spielewoy/autoprompt-skill) | Irrelevante | Skill única, no patrón de organización | — |
| 1.149 | [JuliusBrussee/cavekit](https://github.com/JuliusBrussee/cavekit) | Irrelevante | Plugin SDD congelado | — |
| 1.076 | [Gentleman-Programming/gentle-shell](https://github.com/Gentleman-Programming/gentle-shell) | Irrelevante | Solapa con otros candidatos Pi ya cubiertos | — |
| 1.069 | [vstorm-co/pydantic-deepagents](https://github.com/vstorm-co/pydantic-deepagents) | Irrelevante | Reimplementación de menor adopción que deepagents/langchain | — |
| 1.055 | [shyamsaktawat/OpenAlpha_Evolve](https://github.com/shyamsaktawat/OpenAlpha_Evolve) | Irrelevante | Programación evolutiva autónoma, no roles/coordinación | — |
| 1.012 | [tensorlakeai/tensorlake](https://github.com/tensorlakeai/tensorlake) | Irrelevante | Runtime serverless genérico | — |
| 1.006 | [0xfurai/claude-code-subagents](https://github.com/0xfurai/claude-code-subagents) | Irrelevante | Colección de +100 subagentes | — |
| 968 | [iamfakeguru/agent-md](https://github.com/iamfakeguru/agent-md) | Irrelevante | Fichero de directivas estático, no proceso de coordinación | — |
| 951 | [ccplugins/awesome-claude-code-plugins](https://github.com/ccplugins/awesome-claude-code-plugins) | Irrelevante | Lista curada | — |
| 938 | [data-goblin/power-bi-agentic-development](https://github.com/data-goblin/power-bi-agentic-development) | Irrelevante | Skills de Power BI, dominio ajeno | — |
| 885 | [augmentcode/augment-swebench-agent](https://github.com/augmentcode/augment-swebench-agent) | Irrelevante | Implementación de referencia SWE-bench, agente único (relevante como benchmark C13) | — |
| 829 | [qwen-code-dev-bot/oh-my-cli](https://github.com/qwen-code-dev-bot/oh-my-cli) | Irrelevante | CLI mínima, agente único | — |
| 818 | [alfredxw/denova](https://github.com/alfredxw/denova) | Irrelevante | Escritura creativa/RPG, dominio ajeno | — |
| 787 | [SWE-bench/SWE-smith](https://github.com/SWE-bench/SWE-smith) | Irrelevante | Infraestructura de datos de entrenamiento, no patrón de organización | — |
| 730 | [kingbootoshi/cartographer](https://github.com/kingbootoshi/cartographer) | Irrelevante | Documentación de código con subagentes paralelos, no coordinación de trabajo | — |
| 720 | [soumatheusgomes/vibe-coding-toolkit](https://github.com/soumatheusgomes/vibe-coding-toolkit) | Irrelevante | Toolkit de plugins/prompts curado | — |
| 684 | [shinpr/claude-code-workflows](https://github.com/shinpr/claude-code-workflows) | Irrelevante | Alcance limitado a exploración/aprobación, no coordinación multiagente | — |
| 683 | [devoxx/DevoxxGenieIDEAPlugin](https://github.com/devoxx/DevoxxGenieIDEAPlugin) | Irrelevante | Plugin IntelliJ para LLMs locales, asistente único | — |
| 664 | [ThibautBaissac/rails_ai_agents](https://github.com/ThibautBaissac/rails_ai_agents) | Irrelevante | Skills específicos de Rails, dominio de framework | — |
| 629 | [Decade-qiu/CookHero](https://github.com/Decade-qiu/CookHero) | Irrelevante | App de cocina, fuera de dominio | — |
| 589 | [seulee26/mckinsey-pptx](https://github.com/seulee26/mckinsey-pptx) | Irrelevante | Generador de PPTX, dominio ajeno | — |
| 530 | [RoggeOhta/awesome-codex-cli](https://github.com/RoggeOhta/awesome-codex-cli) | Irrelevante | Lista curada | — |
| 517 | [jqueryscript/awesome-claude-code](https://github.com/jqueryscript/awesome-claude-code) | Irrelevante | Lista curada | — |

**Nota de fusión de organizaciones:** `sst/opencode` fue transferido a `anomalyco/opencode` (redirect 301 confirmado por `gh api`); `princeton-nlp/SWE-agent` fue transferido a `SWE-agent/SWE-agent` — ambos casos son un único repo contado una vez a efectos de conteo total, listado dos veces en la tabla porque las dos rutas de nombre aparecieron en distintas búsquedas y se prefirió no ocultar ninguna referencia encontrada.

---

## 3. Funciones nativas de los arneses de agentes

Términos: **arnés** (*harness*) = programa que aloja el bucle agente↔herramientas↔modelo (Claude Code, Codex CLI, etc.). **Subagente** = agente auxiliar invocado por el agente principal dentro de la misma sesión, con su propio contexto. **Claim** = reserva de una tarea por un agente. **Lease** = reserva con caducidad.

### Claude Code (Anthropic)

**Subagentes** ([code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)):
- Delegación automática ("Claude automatically delegates tasks based on the task description... the `description` field in subagent configurations") o explícita (`@mención`, `--agent <name>`).
- **Sin cola de tareas ni claim/lease explícitos: "No explicit task queue or claim/lease mechanism. Coordination is implicit"** — cita literal de la doc, tal como ya había encontrado F1.
- Límite de concurrencia: 20 subagentes simultáneos por defecto (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`), profundidad de anidación 3 niveles (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`).
- **Modelo por subagente**: campo `model` acepta alias (`sonnet`, `opus`, `haiku`, `fable`), ID completo (`claude-opus-5-5`, `claude-sonnet-5`) o `inherit`. **Solo modelos Claude** aparecen documentados; no hay mención de modelos de terceros (DeepSeek, GPT, etc.) en este mecanismo nativo.
- Retorno: subagentes en primer plano bloquean la conversación; en segundo plano notifican al terminar; son reanudables vía `SendMessage`.

**Agent teams** ([code.claude.com/docs/en/agent-teams](https://code.claude.com/docs/en/agent-teams), **experimental y desactivado por defecto**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`):
- Arquitectura: **lead** (coordina) + **teammates** (sesiones independientes de Claude Code, cada una en su propio *context window*) + **lista de tareas compartida** + **buzón (mailbox)** de mensajería directa entre agentes.
- Tareas: "Tasks have three states: pending, in progress, and completed... can also depend on other tasks: a pending task with unresolved dependencies cannot be claimed until those dependencies are completed."
- **Claim con lock de fichero**: *"Task claiming uses file locking to prevent race conditions when multiple teammates try to claim the same task simultaneously."* Dos modos de asignación: el lead asigna explícitamente, o el teammate hace *self-claim* de la siguiente tarea no bloqueada al terminar la suya.
- **Hooks de calidad**: `TeammateIdle`, `TaskCreated`, `TaskCompleted` — salir con código 2 bloquea la acción y devuelve feedback al agente.
- Persistencia: config del equipo en `~/.claude/teams/{team-name}/config.json` (se borra al terminar la sesión); lista de tareas en `~/.claude/tasks/{team-name}/` (persiste localmente, nunca se sube, sobrevive a la reanudación).
- Permisos: los teammates heredan el modo de permisos del lead (salvo `dontAsk`); las peticiones de permiso de un teammate aparecen en la sesión del lead.
- **Modelo por teammate**: se fija al generarse el teammate y no cambia después salvo relanzarlo; se puede nombrar una definición de subagente reutilizable (rol) al invocar. Igual que en subagentes, solo se documentan modelos Claude.
- Limitación explícita de la doc: *"Agent teams have known limitations around session resumption, task coordination, and shutdown behavior."*

**Hooks** ([code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks)): 33 eventos de ciclo de vida, incluidos `PreToolUse`/`PostToolUse`, `SubagentStart`/`SubagentStop`, `TaskCreated`/`TaskCompleted`, `TeammateIdle`, `WorktreeCreate`/`WorktreeRemove`, `PreModelSwitch`/`PostModelSwitch`. El código de salida 2 bloquea la acción en los eventos que lo permiten (tabla explícita en la doc); es el mecanismo genérico para *quality gates* automáticos (relevante a C6).

**Agent SDK** ([platform.claude.com/docs/en/agent-sdk/overview](https://platform.claude.com/docs/en/agent-sdk/overview), redirige desde `code.claude.com/docs/en/agent-sdk/overview`): librería que embebe el binario de Claude Code (Python/TypeScript) con las mismas capacidades (hooks, subagentes, MCP, permisos, sesiones). Existe además **Managed Agents**: "a hosted agent harness that runs the agent loop, with sessions in an Anthropic-managed cloud sandbox or a self-hosted sandbox on your own infrastructure" — producto separado, vía API/CLI `ant`/REST. Ni el SDK ni Managed Agents documentan en esta página soporte de modelos no-Anthropic.

### OpenAI Codex

- **Codex CLI** ([github.com/openai/codex](https://github.com/openai/codex)): agente de terminal ligero, ejecutor único, sin gestión de tareas/multiagente nativa documentada.
- **Codex Cloud** ([learn.chatgpt.com/docs/cloud](https://learn.chatgpt.com/docs/cloud)): **paralelismo real** — *"Give longer tasks dedicated environments and let them continue while you work on something else"*; cada tarea corre en un **entorno cloud aislado y dedicado**. Integración directa: *"Start work in Codex cloud from GitHub pull requests, GitLab merge requests and issues, Linear issues, or Slack channels."* Revisión: *"Review the summary and diff, request a follow-up, or open a pull request when the result is ready"* (revisión humana antes de abrir PR). Trazabilidad vía logs de tarea. **No se menciona swap a modelos no-OpenAI.**

### Gemini CLI (Google)

- Sin `subagents.md` en el repo de documentación (`google-gemini/gemini-cli/docs/cli/`, listado completo verificado por `gh api repos/.../contents/docs/cli`) — **confirma la ausencia de subagentes nativos** que ya había encontrado F1.
- **Model routing** ([docs/cli/model-routing.md](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/model-routing.md)): fallback automático entre modelos por turno/sesión; incluye *"Local Model Routing (Experimental)"* con un modelo **Gemma** local para decidir el enrutado — es el único indicio de modelo no-propietario, y solo como clasificador interno, no como ejecutor de rol.
- **git-worktrees** ([docs/cli/git-worktrees.md](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/git-worktrees.md)): aislamiento de sesiones paralelas vía `--worktree`, sin número máximo documentado y **sin mecanismo de coordinación/lock**: *"You are responsible for cleaning up your worktrees manually."*

### OpenCode (SST → Anomaly Co.)

- **Agentes primarios vs. subagentes** ([opencode.ai/docs/](https://opencode.ai/docs/), [opencode.ai/docs/agents/](https://opencode.ai/docs/agents/)): *"Primary agents are the main assistants you interact with directly"* (Build, Plan); *"Subagents are specialized assistants that primary agents can invoke for specific tasks"*, automática o por `@mención`.
- **Modelo por agente, incluidos proveedores no-vendor**: *"Use the model config to override the model for this agent... The model ID in your OpenCode config uses the format `provider/model-id`"* — es el **único arnés confirmado en esta fase que permite explícitamente asignar un modelo distinto por rol, con cualquier proveedor configurado (incluye modelos abiertos/DeepSeek vía API compatible)**, relevante directamente para la restricción del usuario de usar DeepSeek en el rol de implementación.
- **Coordinación**: herramienta `Task` con `permission.task` para restringir, por patrón glob, qué subagentes puede invocar cada agente — no hay cola de tareas persistente ni claim/lease documentados.

### Cursor — Background/Cloud Agents

[cursor.com/docs/background-agent](https://cursor.com/docs/background-agent): disparo desde iOS/Web/Desktop/Slack/GitHub o Bitbucket/Linear/API; *"You can run as many agents as you want in parallel"* en VMs aisladas. *"Agents are visible to members of the Cursor team they were started under"* (trazabilidad de equipo). Salida en PR "merge-ready" en rama separada. **Sin mecanismo de claim/lease ni resolución de conflictos entre agentes concurrentes documentado.** *"Cloud Agents use a curated selection of models"* — selección curada, sin confirmar si incluye modelos no-OpenAI/no-Anthropic.

### GitHub Copilot coding agent

[docs.github.com — about coding agent](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent): asignación por *"selecting 'Copilot' as the assignee"* en un issue. *"Working on GitHub adds transparency, with every step happening in a commit and being viewable in logs"* (trazabilidad nativa vía commits/logs). *"Depending on how you start your Copilot cloud agent task, you may be able to select the model used"* — selección de modelo posible mencionada, sin detallar si incluye modelos no-OpenAI. **La doc no aborda qué pasa si dos issues/agentes tocan los mismos ficheros** (ya lo había notado F1; se confirma en esta re-verificación, incluida la página de mejores prácticas que devolvió 404 al intentar profundizar).

### Kiro (AWS)

[kiro.dev/docs/specs/](https://kiro.dev/docs/specs/): flujo de tres fases **Requisitos → Diseño → Tareas** (`requirements.md`/`design.md`/`tasks.md`), con seguimiento de estado en tiempo real. **Ejecución en oleadas**: *"Kiro builds a dependency graph of the tasks in your tasks.md and groups independent tasks into waves: Wave 1 - all tasks with no dependencies... Wave 2... Waves execute sequentially; tasks within a wave execute concurrently."* — coincide conceptualmente con el "wave-based topological dispatch" del paper académico SPOQ (§5), dato relevante a Q3/Q4. Kiro tiene además un producto separado, **"Crew"** ("Running 24/7", modo autónomo), no explorado en profundidad por límite de tiempo — hueco anotado en §8.

### Amp (Sourcegraph)

[ampcode.com/manual](https://ampcode.com/manual): agentes en segundo plano ("Orbs: Machines that Amp creates per thread"), continuidad multi-dispositivo (*"Amp is the same agent and the same threads everywhere"*), **multi-modelo declarado**: *"Multi-Model: GPT-5.6, Claude Fable 5.1, fast models... Amp uses them all, for what each model is best at"*. **No se encontró documentación explícita de subagentes con nombre propio (p. ej. "Oracle"), task tracking formal ni bucle de revisión estructurado** en la página consultada — posible hueco de documentación más que ausencia de la función; marcado como sin verificar en profundidad.

---

## 4. Productos de vendors

| Vendor | Producto | Qué organiza | Fuente |
|---|---|---|---|
| Anthropic | Claude Code (CLI, subagentes, *agent teams* experimental), Agent SDK, Managed Agents | Ver §3 | [code.claude.com/docs](https://code.claude.com/docs/en/agent-teams), [platform.claude.com](https://platform.claude.com/docs/en/agent-sdk/overview) |
| OpenAI | Codex CLI + Codex Cloud | Paralelismo real en entornos cloud aislados, integración GitHub/GitLab/Linear/Slack | [learn.chatgpt.com/docs/cloud](https://learn.chatgpt.com/docs/cloud) |
| Google | Gemini CLI, Antigravity (`google-antigravity/antigravity-cli`, 2.380★), extensión oficial `gemini-cli-extensions/conductor` (SDD) | Sin subagentes nativos en Gemini CLI; Antigravity es agente único orientado a IDE/terminal (sin verificar en profundidad — hueco, §8) | [github.com/google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli), [github.com/google-antigravity/antigravity-cli](https://github.com/google-antigravity/antigravity-cli) |
| GitHub (Microsoft) | Copilot coding agent, Copilot CLI, Issues/sub-issues/Projects v2 (funciones nativas de la forja, ya documentadas en la fase de definición previa/F1) | Asignación por *issue assignee*, trazabilidad vía commits | [docs.github.com](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) |
| Microsoft (Research) | AutoGen / continuación comunitaria AG2 | Framework de programación agéntica genérico | [github.com/microsoft/autogen](https://github.com/microsoft/autogen) |
| AWS | `awslabs/cli-agent-orchestrator` (oficial, org AWS Labs), Kiro (specs + Crew), Strands Agents (`strands-agents/harness-sdk`) | Orquestación multiagente de CLIs vía tmux; SDD con ejecución en oleadas; SDK genérico de agentes | [github.com/awslabs/cli-agent-orchestrator](https://github.com/awslabs/cli-agent-orchestrator), [kiro.dev/docs/specs](https://kiro.dev/docs/specs/) |
| Databricks | Omnigent (`omnigent-ai/omnigent`) — anunciado en el blog oficial de Databricks, no solo repo comunitario | Meta-harness multiagente: políticas, sandboxing, colaboración en tiempo real | [databricks.com/blog/introducing-omnigent...](https://www.databricks.com/blog/introducing-omnigent-meta-harness-combine-control-and-share-your-agents) (vía HN Algolia, título y URL confirmados; contenido del blog no releído directamente — **sin verificar el detalle interno**, solo la existencia y autoría del anuncio) |
| xAI | `xai-org/grok-build` — CLI de agente de código, anunciado en [x.ai/news/grok-build-cli](https://x.ai/news/grok-build-cli) (vía HN, 100 puntos) | Agente de terminal, capacidad de orquestación declarada en la descripción del repo, sin detalle propio verificado en profundidad | [github.com/xai-org/grok-build](https://github.com/xai-org/grok-build) |
| Sourcegraph | Amp | Ver §3 | [ampcode.com/manual](https://ampcode.com/manual) |
| Y Combinator | `yc-software/qm` — "multiplayer agent harness for work", alojado en `qm.ycombinator.com` | Coordinación multiagente vía Slack/web (producto propio de YC, no solo inversión) | [github.com/yc-software/qm](https://github.com/yc-software/qm), cobertura HN 682 puntos |
| Temporal Technologies / Hatchet / Rivet | Temporal, Hatchet, Rivet | Motores de ejecución duradera de propósito general reutilizados para orquestar agentes | Ver tabla §2 |

**Hueco declarado:** no se investigó en profundidad Cursor Enterprise, Windsurf (Cognition), Devin/Devin Review (Cognition, cerrado, sin repo público — `gh api repos/cognition-labs/devin` devuelve 404 según F1, no re-verificado en esta fase), ni Replit Agent, ni el ecosistema completo de GitHub Copilot Workspace — por límite de tiempo de la subtarea, no por juicio de irrelevancia. Ver §8.

---

## 5. Sistemas académicos verificados

Verificados cada uno en su página `arxiv.org/abs/<id>` directa (no solo vía la API), tal como exige la definición de la fase:

### SPOQ — Specialist Orchestrated Queuing for Multi-Agent Software Engineering
[arxiv.org/abs/2606.03115](https://arxiv.org/abs/2606.03115) — Royce Carbowitz y Dheeraj Kumar, enviado 2 jun 2026, cs.SE/cs.MA. Tres mecanismos: **(1)** *wave-based topological dispatch* — oleadas de ejecución paralela calculadas del grafo de dependencias (ratio 1.03–1.11 respecto al óptimo, *speedup* hasta 14.3×); **(2)** *dual validation gates*, antes (planificación) y después (código) de ejecutar cada tarea, reduce defectos de 0.34 a 0.20 por tarea; **(3)** *Human-as-an-Agent* — un humano participa en la descomposición y puede ser consultado durante la ejecución, reduce defectos residuales a 0.03/tarea. Jerarquía de tres niveles (Opus trabajador, Sonnet revisor, Haiku investigador). **Tipo de evidencia: estudio con benchmark propio de los autores, 55 páginas, incluye estudio longitudinal y réplica en modelo *open-weights* — sin réplica independiente conocida todavía.**

### Co-Coder — When Parallelism Pays Off: Cohesion-Aware Task Partitioning for Multi-Agent Coding
[arxiv.org/abs/2606.00953](https://arxiv.org/abs/2606.00953) — Xu Yang, Lunyiu Nie, Ethan Chandra, Stanislav Gannutin, Fangru Lin, Swarat Chaudhuri, enviado 31 may 2026, cs.LG/cs.MA. Formaliza la orquestación multiagente como un problema de partición de grafos (coste de comunicación entre agentes vs. ganancia de paralelismo); construye grafos de dependencias por análisis estático, aísla ficheros "hub" estructurales, particiona por detección de comunidades. **Sobre 28 tareas reales (DevEval, CodeProjectEval), compara explícitamente contra "Claude Code with Agent Teams"** como baseline — relevante directamente para esta investigación —, con mejoras de hasta 14.0% en *pass rate*, 2.10× de *speedup* y 35% menos coste de API en los proyectos más densos en dependencias. **Tipo de evidencia: estudio con benchmark propio, sin réplica independiente conocida.**

### Descartado por falta de verificación
"Survey on Multi-Agent LLM Frameworks with Semantic Memory Integration and Dynamic DAG Orchestration" (citado por F1 como de ICIRCA 2026, vía Semantic Scholar) — la búsqueda en `export.arxiv.org/api/query` con los términos del título no devolvió ningún resultado (`totalResults=0`). No se encontró en arXiv; puede ser un paper de una conferencia (ICIRCA) no indexado ahí. **Se excluye de la lista de candidatos citables** por no poder verificarse en una página `abs` propia, conforme a la regla de evidencia.

---

## 6. Anomalías de estrellas

**Bloqueo de herramienta primero:** el método de muestreo por `starred_at` (`gh api -H "Accept: application/vnd.github.star+json" repos/.../stargazers`) **no funcionó en este entorno**: devuelve 404 en todas las páginas probadas, para cualquier repo incluido un repo de control muy conocido (`github/spec-kit`), con headers HTTP completos verificados (`gh api -i`) que no muestran carencia de scope. Se registra como bloqueo (§8) y se sustituyó por señales indirectas: velocidad de crecimiento, ratio *forks*/★ y *subscribers*/★, estado de Issues, y cobertura independiente en HN.

| Repo | ★ | Creado | ★/mes aprox. | Forks | Subs | Issues abiertas | Cobertura HN independiente | Veredicto |
|---|---|---|---|---|---|---|---|---|
| [affaan-m/ECC](https://github.com/affaan-m/ECC) | 267.144 | 2026-01-18 (~8,2 meses) | **~32.600** | 39.918 | 1.344 | 232 (ratio issues/★ = 0,09%, muy por debajo de repos comparables como `anomalyco/opencode` con 2,97%) | **Ninguna encontrada** en varias consultas a HN Algolia ("ECC agent harness" y variantes) | **Anomalía fuerte, sin explicación encontrada.** Es el repo con más estrellas de todo el barrido — más que `anomalyco/opencode` (el agente de código open source más popular conocido) — de una cuenta individual con descripción genérica de buzzwords, sin cobertura de prensa/comunidad independiente detectable. No se recomienda como candidato hasta que se explique. |
| [ruvnet/ruflo](https://github.com/ruvnet/ruflo) | 73.237 | 2025-06-02 (~15,8 meses) | ~4.600 | 8.695 | 1.014 | Máximo encontrado: **3 puntos** ("Ruflo: An agent meta-harness for Claude Code and Codex"); un segundo post literalmente pregunta *"Is any one using ruflo?"* (1 punto) | **Discrepancia fuerte entre estrellas (73k) y discurso independiente (casi nulo).** El autor tiene otros proyectos de "swarm"/"flow" con reputación mixta en la comunidad (no verificado en esta fase con más detalle). |
| [xai-org/grok-build](https://github.com/xai-org/grok-build) | 27.088 | 2026-07-14 (~2,3 meses) | ~11.800 | 5.101 | 0 (Issues **desactivadas**, `has_issues:false`) | **Fuerte y verificada**: anuncio oficial [x.ai/news/grok-build-cli](https://x.ai/news/grok-build-cli) (100 pts HN), cobertura técnica independiente ("What xAI's Grok build CLI sends to xAI: A wire-level analysis", 539 pts), post "Grok Build is open source" con 590 pts | **Crecimiento rápido pero explicado y verificado**: lanzamiento oficial de una empresa grande (xAI), con análisis técnico independiente. No se trata como anomalía. |
| [yc-software/qm](https://github.com/yc-software/qm) | 15.239 | 2026-07-29 (~1,9 meses) | ~8.000 | 1.866 | 575 | **Fuerte y verificada**: "qm – Multiplayer agent harness for work", **682 puntos** en HN, alojado en `qm.ycombinator.com` | **Legítimo**: producto propio de Y Combinator con viralidad orgánica documentada en HN. No es anomalía. |
| [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) | 64.458 | 2025-12-14 (~9,4 meses) | ~6.900 | 5.443 | **0** (`has_issues:true`, es decir, activas pero sin ninguna abierta en 9 meses con 64k★ y sin *push* desde 2026-05-31) | Real: 473 puntos en HN ("Get Shit Done: A meta-prompting...") | **Señal débil, sin explicación clara**: cobertura HN real, pero cero issues abiertas en un repo popular abandonado hace 4 meses es estadísticamente raro. No se afirma inflado — se marca "sin verificar la causa". |
| [omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent) | 10.221 | 2026-06-11 (~3,5 meses) | ~2.900 | 1.624 | 1.472 | Verificada: anuncio en el blog oficial de Databricks | **Explicado**: es un lanzamiento de producto de una empresa grande (Databricks), con issues activas proporcionales — no es anomalía. |
| [aden-hive/hive](https://github.com/aden-hive/hive) | 11.071 | 2026-01-12 (~8,4 meses) | ~1.300 | 5.657 | 1.355 | No buscada en profundidad por límite de tiempo | Crecimiento rápido pero issues/forks proporcionales a las estrellas (ratio saludable) — **sin verificar, prioridad baja** |
| [mindfold-ai/Trellis](https://github.com/mindfold-ai/Trellis) | 14.803 | 2026-01-26 (~8 meses) | ~1.850 | 842 | 72 | No se encontró cobertura relevante en HN | Descripción genérica ("el mejor agent harness"), sin verificar — **prioridad baja, no se investigó a fondo** |

**Conclusión del control:** de los 6 crecimientos rápidos investigados con datos independientes, **2 quedan como anomalía sin explicar** (`affaan-m/ECC`, `ruvnet/ruflo`), **3 quedan explicados y verificados como legítimos** (`xai-org/grok-build`, `yc-software/qm`, `omnigent-ai/omnigent`), y **1 queda con una señal débil no concluyente** (`gsd-build/get-shit-done`). Ninguna acusación se hace sin los datos que la acompañan; ninguno de los dos casos anómalos se usa como candidato en la preselección de §7.

---

## 7. Propuesta de preselección para evaluación profunda (Fase C)

Neutral respecto a la hipótesis del usuario. Criterio de entrada: evidencia de mantenimiento/adopción real (vendor grande, señal HN, o cifras propias de un paper) **y** que cubra una parte del circuito (Q1–Q6) no trivialmente redundante con otro ya elegido.

| # | Candidato | Por qué entra |
|---|---|---|
| 1 | **Claude Code — subagentes + *agent teams*** (nativo) | Es el arnés que ya usa el usuario (Pro); documentación oficial confirma carencias explícitas en coordinación en subagentes (Q3/Q4) y aporta un mecanismo de *claim* con *file locking* real en *agent teams* — línea base obligada, con la limitación de que es **experimental** y solo declara modelos Claude por rol |
| 2 | **GitHub Copilot coding agent + Issues/sub-issues/Projects v2** | Única combinación 100% nativa de la forja fijada (GitHub); trazabilidad casi gratis vía commits/logs; encaje directo con C11 |
| 3 | **spec-kit (GitHub oficial)** | Con diferencia el proyecto más adoptado (138.822★); estándar de facto para Q1 (división/especificación), no resuelve coordinación ni ejecución |
| 4 | **OpenCode** | Único arnés que documenta explícitamente modelo-por-rol con **cualquier proveedor** (`provider/model-id`), lo que encaja directamente con la restricción del usuario de usar DeepSeek para implementar — dato que ni Claude Code ni Gemini CLI ofrecen de forma nativa |
| 5 | **Beads** (`gastownhall/beads`) | Grafo de tareas con dependencias persistido en git, entre sesiones — resuelve Q3 (estado del trabajo) de forma independiente del arnés/modelo, por un autor reconocido (Steve Yegge, según cobertura de F1 vía su blog) |
| 6 | **awslabs/cli-agent-orchestrator** | Único orquestador multiagente *oficial de un vendor grande* que coordina explícitamente varios arneses de CLI (incluido Claude Code) en sesiones aisladas — encaja con la restricción "ejecutor intercambiable" |
| 7 | **Kiro (AWS) — specs con ejecución en oleadas** | Único sistema con documentación oficial de un mecanismo de paralelismo por dependencias (*waves*) equivalente al que describe el paper SPOQ — aporta evidencia de que el patrón "oleadas topológicas" ya está en producto, no solo en papers |
| 8 | **SPOQ** (paper) | Única fuente, académica o no, que reporta cifras propias sobre las 4 partes del circuito (oleadas, validación dual, humano-como-agente) — sin réplica independiente, tratar como hipótesis fuerte, no como hecho |
| 9 | **Co-Coder** (paper) | Compara directamente contra "Claude Code with Agent Teams" con cifras propias — evidencia académica más reciente y más pertinente a la pregunta exacta del usuario que SPOQ |
| 10 (señal débil, incluir con reserva) | **Temporal o Hatchet** (uno de los dos, a decidir en C por coste operativo en la VM de 4 núcleos/7GB) | Único enfoque con años de madurez en producción *fuera* del mundo IA — referencia de cuánto cuesta operativamente resolver bien Q3/Q4 de forma genérica; a verificar si cabe sin motor de contenedores |

**Excluidos explícitamente y por qué:**
- **affaan-m/ECC y ruvnet/ruflo**: estrellas no explicadas por evidencia independiente (§6) — no se preseleccionan hasta que se aclare.
- **CrewAI, AutoGen/ag2, MetaGPT, AutoGPT, LangGraph**: frameworks generalistas de agentes, no nacidos para el circuito código→PR→revisión sobre GitHub; se anota su existencia (son los más citados del espacio "multiagente" en general) pero no se preseleccionan como primera línea porque cualquier adopción exigiría construir encima toda la capa de Q1/Q5/Q6/Q7 específica de software.
- **Codex Cloud y Cursor Background Agents**: paralelismo real y bien documentado, pero cerrados/vendor-lock, y ninguno de los dos documenta *claim*/lease ni resolución de conflictos entre agentes concurrentes — se anotan como referencia de "estado del arte comercial" (relevante para comparar C9/C10) pero no como base para el procedimiento replicable que pide el entregable final.
- **Devin (Cognition)**: cerrado, sin repo público, evidencia 100% de vendor — no evaluable con el mismo rigor.
- **La mayoría de la cola larga de la tabla §2** (colecciones de subagentes temáticos, listas curadas, harnesses de un solo agente sin patrón de coordinación): descartados por motivo individual ya anotado en la tabla, no por falta de estrellas.
- **Amp**: documentación consultada no deja claro su mecanismo de task-tracking/subagentes con nombre propio — se deja fuera de la preselección hasta poder profundizar (hueco declarado, no descarte de fondo).

---

## 8. Bloqueos

| Bloqueo | Efecto | Cómo se sorteó / qué queda pendiente |
|---|---|---|
| `gh api .../stargazers` con header `star+json` devuelve 404 en todas las páginas y repos probados, incluido un repo de control | No se pudo hacer el muestreo de fechas de `starred_at` que pide el método | Sustituido por velocidad de crecimiento + ratios forks/subs/issues + cobertura HN (§6); si se reabre este bloqueo, repetir el control con el método original |
| WebSearch agotado (heredado de fases anteriores) | Sin búsqueda web general | `gh`, `WebFetch` y `curl` cubrieron todo lo necesario en esta fase |
| `curl export.arxiv.org/api/query` devuelve 406 | — | **Resuelto parcialmente**: el mismo endpoint funcionó a través de `WebFetch` en esta sesión (a diferencia de lo reportado por F1 con `curl` puro) — anotar esta diferencia para sesiones futuras |
| No se navegó (Chrome/CDP prohibido en esta subtarea) | Reddit/X no consultados aquí (ya cubiertos en fase A) | No aplica a esta fase |
| Amp: documentación pública no detalla subagentes/task-tracking | Amp queda con cobertura incompleta en §3 | Pendiente de una lectura más profunda de `ampcode.com` (p. ej. la página "Oracle" o "subagents" si existe) en fase C si se preselecciona |
| Google Antigravity, Cursor Enterprise, Windsurf, Replit Agent, GitHub Copilot Workspace | No investigados en profundidad | Por límite de tiempo de la subtarea, no por juicio de irrelevancia — hueco declarado para fase C |
| "Survey on Multi-Agent LLM Frameworks..." (ICIRCA 2026) | No verificable en arXiv | Descartado de la lista citable (§5); podría existir en las actas de la conferencia, no accesible con las herramientas de esta subtarea |
| Databricks blog de Omnigent no releído directamente (solo título/URL vía HN) | El detalle interno del producto queda sin verificar | Pendiente de `WebFetch` directo a `databricks.com/blog/...` si Omnigent se preselecciona en fase C |

---

## Enlaces

- [[circuito-tareas-definicion]] — define las 6 preguntas, C1–C13 y las reglas de evidencia que sigue este documento
- [[sistema-desarrollo-con-agentes]] — proyecto
- [[desarrollo-agentes-investigacion]] — investigación previa del ciclo completo
-

## Enlaces

- [[flujo-agentes-informe]] — síntesis de la investigación
- [[circuito-tareas-definicion]] — definición de la fase
- [[_index]]
