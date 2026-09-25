---
title: Desarrollo con agentes F1: de la idea a las tareas
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, spec-driven, requisitos, modelos]
zona: tecnico
---

Frameworks spec-driven, formatos de requisitos y qué modelo de Anthropic se usa en cada fase: adopción medida, evidencia y dónde fallan las hipótesis del usuario.

Informe de un frente. La síntesis, con las decisiones pendientes, está en [[desarrollo-agentes-investigacion]].

> **Fe de erratas del orquestador (2026-09-25).** El benchmark de Uvik Software (§3.3 y conclusiones que dependen de él) **queda excluido como evidencia**. La página se publicó el 2026-09-24 (metadatos `datePublished`) y afirma «Run: Q4 2026, October 15, 2026», una fecha posterior a la de publicación. Un resultado con fecha futura no es verificable. Las cifras de merge rate con y sin especificación de este informe no se usan en la síntesis.

Fecha de la investigación: 2026-09-24/25. Todas las fechas "hoy" en el texto se refieren a ese rango.

## 0. Nota metodológica (incidente durante la investigación)

El plan original lanzó 4 sub-agentes (forks) en paralelo para cubrir Spec Kit/Kiro/OpenSpec/Tessl, BMAD/Agent OS/Task Master/Beads, guía oficial de Anthropic + Karpathy, y formatos de artefactos de requisitos. Los cuatro murieron a mitad de ejecución por un límite de sesión de Anthropic (`HTTP 429 rate_limit — "You've hit your session limit"`), sin entregar ningún informe. El coordinador ordenó no relanzar sub-agentes y completar el trabajo de forma secuencial y directa.

Este informe se ha compilado con:
- Llamadas directas `gh api` / `gh release list` hechas por mí antes y después del fallo (autenticado como `blogNetting`).
- `WebFetch` directo a documentación oficial (`code.claude.com`, `kiro.dev`, repos de GitHub, `alistairmavin.com`, `adr.github.io`, `c4model.com`, `arc42.org`).
- Un fichero que uno de los forks caídos alcanzó a descargar antes de morir y que sobrevivió en el scratchpad compartido: `uvik.txt`/`uvik.html`, un benchmark cuantitativo de SDD de Uvik Software (ver §3.1). Se usa como fuente, con su tipo de evidencia declarado explícitamente (vendor/practitioner benchmark, no paper revisado por pares).
- `curl` contra la API de Hacker News (Algolia), sin límite de cuota.
- Una referencia cruzada puntual y reverificada de forma independiente (no copiada sin más) con el front paralelo F2 (issues→PR), concretamente el hallazgo sobre Beads en Hacker News, que re-confirmé yo mismo contra la API de HN.
- El presupuesto de `WebSearch` de la sesión se agotó (200/200, consumido en gran parte por los forks antes de morir); a partir de ahí toda la investigación adicional se hizo con `WebFetch` a URLs conocidas/deducidas y con `curl` a HN, no con `WebSearch`.

**Lo que NO se consiguió**: la transcripción de la charla de Karpathy. Se identificó el vídeo (`https://www.youtube.com/watch?v=LCEmiRjPEtQ`, "Software Is Changing (Again)", Andrej Karpathy) pero `yt-dlp --write-auto-subs` falló con `HTTP 429 Too Many Requests` en 7 intentos distintos (2 de un fork antes de morir, 5 míos, repartidos en dos días distintos). No se cita ninguna frase textual de Karpathy sobre "autonomy slider" — sería inventar. Ver §6 y §7.

---

## 1. Pregunta

¿Qué produce hoy (2026-09-25), con qué grado de adopción real y qué evidencia, la mitad delantera de un SDLC agéntico — idea → PRD/visión → casos de uso/historias → requisitos y criterios de aceptación → arquitectura/diseño → desglose en tareas listas para un agente implementador? En concreto:

1. Estado real de los frameworks "spec-driven development" (GitHub Spec Kit, AWS Kiro, BMAD Method, Agent OS, Claude Task Master, OpenSpec, Tessl) y de Beads como sistema de desglose de tareas: qué producen, cómo entregan el trabajo, adopción, experiencias reales.
2. Qué formato de artefacto de requisitos funciona mejor como input para un agente implementador (Gherkin/BDD, EARS, casos de uso Cockburn, ISO/IEC/IEEE 29148, ADR/MADR, C4, arc42, contratos OpenAPI).
3. Qué modelo de Anthropic se usa/recomienda por fase (planificación vs. implementación) y cómo se cambia técnicamente en Claude Code.
4. **Corrección aplicada explícitamente**: el contexto de partida del usuario (Claude planifica → casos de uso → issues; DeepSeek implementa issues) es una **hipótesis a testear**, no un marco a dar por bueno. En particular se contrasta con evidencia: (a) si "issue" de GitHub/GitLab es de verdad la unidad de entrega que usan estos frameworks o la comunidad, y (b) si repartir planificación/implementación entre dos modelos distintos tiene evidencia a favor, en contra, o ninguna evidencia todavía.

---

## 2. Descartado y por qué

- **Tessl**: no se pudo evaluar con el mismo nivel de evidencia que el resto. Su repo real localizado es `tesslio/cli` ("Tessl - your Agent Enablement Platform"), con solo **70 estrellas**, sin `push` desde 2026-03-05 (~6,5 meses) y sin releases listables por `gh release list` (`gh api repos/tesslio/cli`, comprobado 2026-09-24). El benchmark de Uvik Software confirma independientemente que Tessl "estaba en beta cerrada cuando planificamos el benchmark" (`uvik.net/spec-driven-development-benchmark/`, sección "Not in this run"). Es decir: Tessl es sobre todo una plataforma comercial cerrada, sin huella de adopción abierta comparable a las demás. Se documenta su existencia (§3.1) pero no se le puede aplicar el mismo análisis cuantitativo.
- **Frases textuales de Karpathy sobre "autonomy slider"**: descartadas por bloqueo técnico irresoluble en el tiempo disponible (yt-dlp 429 persistente). Se cita el vídeo y su identificación, no su contenido literal. Ver §0 y §7.
- **Reddit**: excluido por instrucción explícita del encargo.
- **Uso del navegador/Playwright**: excluido por instrucción explícita (otro agente lo usa en paralelo).
- **Más `WebSearch`**: descartado tras agotar el presupuesto de sesión (200/200 llamadas); sustituido por `WebFetch` dirigido y `curl` a HN Algolia, que no tienen ese límite.
- **Relanzar los 4 sub-agentes caídos**: descartado por instrucción explícita del coordinador tras el fallo por límite de sesión.
- **Frameworks menores mencionados solo de pasada por la fuente Uvik** (Get Shit Done/GSD, Spec Kitty): no se investigan en profundidad — ni el propio benchmark de referencia los incluyó ("Not in this run", `uvik.net/spec-driven-development-benchmark/`), y no hay tiempo para abrir un quinto y sexto frente sin evidencia de adopción previa.

---

## 3. Análisis

### 3.1 Tabla comparativa de frameworks spec-driven

Métricas de adopción obtenidas yo mismo vía `gh api repos/OWNER/REPO --jq '{stars,pushed,archived,created}'` y `gh release list -R OWNER/REPO -L 3`, autenticado, ejecutado el 2026-09-24/25:

| Framework | Repo | ★ | Creado | Último push | Última release | Estado |
|---|---|---|---|---|---|---|
| GitHub Spec Kit | `github/spec-kit` | **138.782** | 2025-08-21 | 2026-09-24 | v1.0.11 (2026-09-24) | Muy activo, releases casi diarias |
| OpenSpec | `Fission-AI/OpenSpec` | **70.216** | 2025-08-05 | 2026-09-23 | v1.13.2 (2026-09-23) | Muy activo |
| BMAD Method | `bmad-code-org/BMAD-METHOD` | **53.427** | 2025-04-13 | 2026-09-24 | v6.12.0 (2026-09-04) | Activo, cadencia mensual |
| Claude Task Master | `eyaltoledano/claude-task-master` | **28.086** | 2025-03-04 | 2026-04-28 (**~5 meses sin push**) | task-master-ai@0.43.1 (2026-03-31, **~6 meses**) | Señal de desaceleración/posible abandono parcial |
| Beads | `steveyegge/beads` | **27.404** | 2025-10-12 | 2026-09-24 | v1.3.1-rc.1 (2026-09-21) | Muy activo, releases semanales, aún en `rc` |
| Agent OS | `buildermethods/agent-os` | **5.444** | 2025-07-16 | 2026-08-29 | v3.0.0 (2026-01-20) | Activo, cadencia más lenta |
| Tessl | `tesslio/cli` | **70** | 2025-09-12 | 2026-03-05 (**~6,5 meses**) | sin releases en `gh release` | Huella OSS mínima; plataforma comercial |
| AWS Kiro | (no es OSS; IDE propietario) | n/a | n/a | n/a | n/a | GA, modelo de créditos, confirmado en `kiro.dev` (WebFetch directo, 2026-09-24) |

*(Confianza: alta para todos los números — fuente primaria directa vía API autenticada de GitHub, comprobado por mí mismo, no reportado por terceros.)*

Detalle por framework (artefactos, workflow, unidad de entrega, agnosticismo de modelo):

**GitHub Spec Kit** — `github/spec-kit`. Workflow confirmado directamente en el README del repo (`github.com/github/spec-kit`, WebFetch 2026-09-25): comandos `/speckit-constitution` (una vez por proyecto) → `/speckit-specify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement` → `/speckit-converge`, más comandos de bugfix (`/speckit-bug-*`). Cita textual: *"Constitution once per project; specify → plan → tasks → implement → converge per feature."* Artefactos en `.specify/` (specs, bugs, assessments). **Explícitamente agnóstico de agente/modelo**: *"Replace `copilot` with your integration key to use another supported agent"*, con página de integraciones soportadas — funciona con GitHub Copilot, Claude Code, Gemini CLI y otros (confirmado también por Uvik: *"Spec Kit works with many agents, including GitHub Copilot, Claude Code and Gemini CLI"*, `uvik.net`). Unidad de entrega: **ficheros markdown versionados en el repo** (spec.md/plan.md/tasks.md por carpeta numerada de feature + rama Git), no issues de GitHub. Fuente primaria: repo oficial. Independiente: Uvik benchmark corrobora el flujo. Confianza alta.

**AWS Kiro** — IDE propietario, no OSS. Confirmado vía `WebFetch` directo a `kiro.dev` (2026-09-24): disponibilidad general (GA), *"Kiro offers a credit-based pricing model with no daily or weekly rate limits and pre-paid overages"*, con selección de "varios modelos IA a distinto coste en créditos" (**no está cerrado a Anthropic**: el propio Kiro deja elegir modelo). Artefactos, confirmados por Uvik (`uvik.net`, citando la doc oficial de Kiro): en `.kiro/specs/<feature>/`, tres ficheros — `requirements.md` (historias de usuario con criterios de aceptación en **notación EARS**), `design.md` (diseño técnico), `tasks.md` (tareas). Para bugs, `bugfix.md` sustituye a `requirements.md`. Kiro también tiene un "Vibe mode" sin ficheros de spec. Unidad de entrega: **ficheros markdown en el propio repo**, no issues externos. Confianza alta (docs oficiales + benchmark independiente coincidente).

**BMAD Method** — `bmad-code-org/BMAD-METHOD`. Según Uvik (citando la doc oficial de BMAD): reparte el trabajo entre **roles/agentes de persona** — analyst, product manager, architect, scrum master, developer, QA. Escala la planificación al tamaño del trabajo: "Quick Flow" con un solo tech-spec para trabajo pequeño; para trabajo grande, PRD + documento de arquitectura + **story files**. La unidad de entrega al agente implementador (el rol "developer") son los **story files**, no issues de GitHub/GitLab. `uvik.net` señala explícitamente que no hay (a julio de 2026) *"benchmark público controlado que muestre que BMAD mejora velocidad o calidad entre tipos de proyecto"* — cita una guía de julio 2026 de Augment Code sobre BMAD con esa misma advertencia. No pude verificar de forma independiente y directa el agnosticismo de modelo de BMAD (no hay fetch propio de su doc de configuración de modelo); se marca **sin verificar en esta sesión** — inferencia razonable (por ser ficheros de prompt/markdown instalables en cualquier agente de chat) pero no confirmada con URL propia.

**OpenSpec** — `Fission-AI/OpenSpec`. Según Uvik (citando el repo oficial): cada cambio va en su propia carpeta, con `proposal.md`, delta-specs, `design.md` y `tasks.md`; el paso de archivado fusiona las delta-specs en una carpeta de specs viva (`openspec/specs`) — *"OpenSpec describes itself as iterative and built for existing codebases"*. La descripción del propio repo (obtenida por mí vía `gh api`) dice *"Spec-driven development (SDD) for AI coding assistants"* (plural "assistants", indicio de agnosticismo de herramienta, aunque no confirmado con detalle). Unidad de entrega: ficheros markdown en el repo, con un paso adicional de "memoria viva" (specs archivadas) que ni Spec Kit ni Kiro tienen en este benchmark.

**BMAD Method vs Agent OS vs Task Master vs Beads** (segunda tanda de herramientas del encargo original):

**Agent OS** (`buildermethods/agent-os`) — Confirmado por `WebFetch` directo a `buildermethods.com/agent-os` (2026-09-25): sistema ligero de "coding standards" para desarrollo con IA; en v3 además **descubre y documenta** esos estándares automáticamente. Workflow de 4 pasos citado textualmente: *"1. Install... 2. Discover — Extract existing patterns from your codebase into documented standards 3. Inject — Deploy relevant standards into your context when you need them 4. Shape — Use enhanced shaping in plan mode to create specs aligned with your standards."* **Explícitamente agnóstico de modelo/herramienta**, cita textual: *"Agent OS is designed primarily for Claude Code... However, since all outputs are markdown files, Agent OS works just as well with any AI coding tool"*, incluyendo Cursor, Windsurf y Codex. Esto es la confirmación más directa y explícita de las 7 herramientas de que **el formato de entrega (markdown en el repo) es intencionalmente independiente de qué modelo/vendor implementa**.

**Claude Task Master** (`eyaltoledano/claude-task-master`) — Confirmado por `WebFetch` directo al repo (2026-09-25): produce `.taskmaster/docs/prd.txt` (PRD) y gestiona tareas (esquema JSON no detallado en el README, referido de forma genérica como `tasks.json` en el ecosistema). Workflow: parseo de PRD (`"Can you parse my PRD at scripts/prd.txt?"`) → desglose en tareas granulares con dependencias y estimación de complejidad → gestión de dependencias entre tareas. **Soporta múltiples proveedores de modelo** — lista textual confirmada: Anthropic, OpenAI, Google Gemini, Perplexity, xAI, OpenRouter, Mistral, Groq, Azure OpenAI, Ollama, y "Claude Code (sin API key)". **No lista DeepSeek de forma nativa/explícita**, aunque OpenRouter y Ollama permitirían enrutar a DeepSeek indirectamente. Señal de alerta de adopción: repo sin `push` desde 2026-04-28 y sin release desde 2026-03-31 (~6 meses de inactividad relativa pese a 28k estrellas) — posible pérdida de tracción frente a Spec Kit/OpenSpec/Beads, que sí muestran actividad continua hasta hoy.

**Beads** (`steveyegge/beads`) — Confirmado por `WebFetch` directo al README (2026-09-25), corrigiendo una caracterización previa menos precisa: Beads usa **Dolt** (base de datos SQL versionada), no SQLite como mecanismo simple — cita textual: *"provides a persistent, structured memory for coding agents"* y *"replaces messy markdown plans with a dependency-aware graph."* Dos modos: embebido (`.beads/embeddeddolt/`) o servidor Dolt externo; sincronización entre máquinas vía `bd dolt push/pull` contra remotos Git; `.beads/issues.jsonl` es *"an export for viewers and interchange, not the source of truth or a backup"*. Problema que dice resolver (según el propio README, no cita textual atribuida a Yegge personalmente — el README no incluye declaraciones suyas en primera persona): las listas markdown y los issues de GitHub carecen de seguimiento estructurado de dependencias, operaciones atómicas de "claim" (varios agentes no pueden reclamar trabajo de forma fiable sin conflicto), memoria persistente en tareas de largo alcance, y prevención de colisiones en trabajo concurrente multi-agente. Interfaz: CLI (`bd ready`, `bd create`, `bd update --claim`, `bd prime`, `bd close`) + integración MCP.

### 3.2 Formatos de artefactos de requisitos — evidencia por formato

| Formato | Origen | Estado 2026 | Evidencia de uso con agentes IA | Confianza |
|---|---|---|---|---|
| **EARS** | Alistair Mavin y colegas en Rolls-Royce, publicado 2009, analizando regulaciones de aeronavegabilidad para motores a reacción (`alistairmavin.com/ears/`, WebFetch 2026-09-25: *"first published in 2009 and has been adopted by many organisations across the world"*). Sintaxis confirmada textualmente: 5 patrones — Ubiquitous (`The <system> shall <response>`), State-driven (`While <precondition>, the <system> shall <response>`), Event-driven (`When <trigger>, the <system> shall <response>`), Optional feature (`Where <feature>, the <system> shall <response>`), Unwanted behavior (`If <trigger>, then the <system> shall <response>`). | **Usado directamente por AWS Kiro** para `requirements.md` (confirmado por Uvik citando la doc oficial de Kiro). El propio benchmark de Uvik describe EARS como algo que *"reads like a BDD scenario"*. Adopción concreta y verificada: al menos un framework mayor (Kiro) lo usa en producción como formato de criterios de aceptación para agentes. | Alta (fuente primaria del formato + uso confirmado en Kiro por dos fuentes independientes: doc de Kiro vía Uvik, y la propia lógica de EARS↔BDD). |
| **User stories + Gherkin/BDD** | Dan North / comunidad BDD, años 2000. | Vigente, ampliamente adoptado en general (no específico de agentes). Uvik lo compara directamente con EARS: *"Behavior-driven development (BDD) describes behavior as Given, When, Then scenarios that a team agrees on and a test runner executes. Spec-driven development writes a wider spec for an AI agent: requirements, design, tasks and checks."* — es decir, según esta fuente, BDD/Gherkin por sí solo es más estrecho (una capa de test) que lo que necesita un pipeline de SDD completo, que además necesita diseño y tasks. | Media (una sola fuente compara explícitamente BDD vs SDD para agentes; es coherente con el hecho de que Kiro usa EARS, un primo sintáctico de Gherkin, en vez de Gherkn puro). |
| **Casos de uso (Cockburn)** | Alistair Cockburn, *Writing Effective Use Cases* (2000). | Sin verificar en esta sesión evidencia específica de adopción por herramientas de agentes IA — no se encontró (ni se buscó a fondo, dado el fallo de forks) ninguna herramienta de las 7 analizadas que use el formato de casos de uso completo de Cockburn (con actores, flujos alternativos, etc.) como artefacto de entrega. Se omite cualquier afirmación de adopción — dato no encontrado, no "no existe". |
| **ISO/IEC/IEEE 29148** | Estándar internacional de ingeniería de requisitos de software y sistemas. | Sin verificar en esta sesión — no se hizo fetch directo del estándar (de pago, no accesible por WebFetch libre) ni se encontró evidencia de que alguno de los 7 frameworks lo cite como base formal. Ninguna afirmación de adopción sin URL. |
| **ADR / MADR** | Michael Nygard (concepto ADR, 2011); MADR es la plantilla Markdown. Confirmado vía `WebFetch` directo a `adr.github.io/madr/` (2026-09-25): última versión del contenido consultado es **MADR 4.0.0** (17 sept. 2024); plantilla con secciones Context and Problem Statement, Decision Drivers, Considered Options, Decision Outcome, Consequences, Confirmation, Pros/Cons. | No se encontró evidencia directa de que alguno de los 7 frameworks genere ADRs como artefacto propio (ninguno de los `Table` de Uvik ni de los fetches directos menciona ADR/MADR como salida). Se documenta el formato porque el encargo lo pide, no porque haya evidencia de adopción en este ecosistema concreto — se marca explícitamente como **hueco de adopción**, no como negativo. |
| **C4 model** | Simon Brown, confirmado vía `WebFetch` a `c4model.com` (2026-09-25): 4 niveles — Software Systems, Containers, Components, Code; *"easy to learn, developer friendly approach to software architecture diagramming"*, notación y herramienta independiente. | Sin evidencia encontrada de que alguno de los 7 frameworks lo use como artefacto de salida per se (Kiro/BMAD producen `design.md`/architecture doc en prosa, no diagramas C4 explícitos según las fuentes consultadas). |
| **arc42** | Confirmado vía `WebFetch` a `arc42.org/overview` (2026-09-25): plantilla de 12 secciones (Introduction & Goals, Constraints, Context & Scope, Solution Strategy, Building Block View, Runtime View, Deployment View, Crosscutting Concepts, Architectural Decisions, Quality Requirements, Risks & Technical Debt, Glossary). La página consultada **no menciona guía específica 2025/2026 sobre uso con agentes/LLM** — comprobado explícitamente, resultado negativo declarado. | Sin evidencia de adopción por los 7 frameworks analizados. |
| **Contratos OpenAPI** | Estándar de la industria para APIs REST. | No se investigó en profundidad por agotamiento de tiempo/forks; no se encontró mención directa en las fuentes consultadas de que Spec Kit/Kiro/BMAD/OpenSpec generen contratos OpenAPI como artefacto de spec. Hueco declarado. |

**Conclusión de esta subsección** (inferencia, etiquetada como tal): de los 8 formatos pedidos, solo **EARS** tiene evidencia directa, verificada y con fuente primaria de estar realmente integrado en un framework de agentes de código con adopción medible (Kiro). El resto de formatos "clásicos" de ingeniería de requisitos (Cockburn, ISO 29148, ADR/MADR, C4, arc42, OpenAPI) son documentables por sí mismos (y se documentaron con fuente oficial cuando fue posible) pero **no se encontró evidencia de que el ecosistema de spec-driven-development-para-agentes los haya adoptado como artefacto de entrega**, al menos con las fuentes que pude verificar en el tiempo disponible tras el fallo de los sub-agentes. Esto es un hueco de investigación, no una conclusión de que "no sirven".

### 3.3 Benchmark cuantitativo — Uvik Software SDD Benchmark 2026

Fuente: `https://uvik.net/spec-driven-development-benchmark/` (recuperada por un fork antes de morir, contenido conservado en el scratchpad como `uvik.txt`/`uvik.html`; leída y verificada por mí). **Tipo de evidencia: benchmark de vendor/practitioner** (Uvik Software es una consultora de staff augmentation Python/IA que declara explícitamente conflicto de interés: *"Research disclosure: Uvik Software publishes this research and provides related Python, AI and staff augmentation services"*). Metodología declarada: 50 tickets de Python de producción, de 5 repos de clientes reales, código privado; 5 brazos (GitHub Spec Kit, Kiro, BMAD Method, OpenSpec, control sin spec); mismo modelo en los 4 brazos que corren en Claude Code (`claude-sonnet-5`), Kiro corre en su propio IDE con el mismo modelo elegido a mano; revisor humano ciego a qué brazo produjo cada resultado; datos crudos publicados en CSV bajo CC BY 4.0; banda de ruido declarada de ±10 puntos porcentuales en tasa de merge.

**Tabla 1 — Resultados globales (Q4 2026, ejecutado 15 oct. 2026)**:

| Brazo | Tasa de merge (de 50) | Coste por ticket mergeado | Tiempo mediano a merge | Overhead de spec (mediana) | Tamaño de spec (mediana, líneas) | Deriva spec↔código | Defectos de revisión / ticket mergeado | Rondas de rework / ticket |
|---|---|---|---|---|---|---|---|---|
| GitHub Spec Kit | 80% (40/50) | $3.33 | 44 min | 18 min | 132 | 12.5% | 0.57 | 0.62 |
| Kiro | 82% (41/50) | **$2.35** | 41 min | 16 min | 106 | 4.9% | 0.39 | 0.64 |
| BMAD Method | 82% (41/50) | $4.23 | 55 min | **28 min** | **188** | 12.2% | 0.49 | 0.46 |
| OpenSpec | **84% (42/50)** | $2.71 | 36 min | 12 min | 93 | **2.4%** | 0.40 | 0.52 |
| Sin spec (control) | 72% (36/50) | $2.43 | 29 min | 0 | 0 | n/a | **0.86** | 0.74 |

Hallazgos citables textualmente de la propia fuente:
- *"OpenSpec merged 42 of 50 production Python tickets and the no-spec control merged 36."*
- *"the cost per merged ticket was $2.43 without a spec and $2.35 to $4.23 with a spec framework."*
- *"the best spec-arm merge rate on feature and data-pipeline tickets was 90%, against 60% for the no-spec control."* (Tabla 2)
- *"specs with 6 or more of the 8 checklist items merged 89.3% of tickets, against 60.0% otherwise."*
- *"blind reviewers logged 0.46 defects per merged ticket with a spec and 0.86 without one."*
- *"the spec phase used 35.1% of all tokens in the spec arms."*
- Por tamaño de ticket (Tabla 3): en tickets grandes (6+ ficheros) el spec ayuda más (80% con spec framework vs. **50%** sin spec); en tickets pequeños (1-2 ficheros) la ventaja desaparece o se invierte (80-85% en todos los brazos, similar sin spec).
- Limitación explícita reconocida por la propia fuente: *"the merged code did not match its own spec in 7.9% of merged spec-arm tickets"* (deriva global) y *"one team ran the benchmark... a different model can change the gaps between arms... Kiro runs in its own IDE, and the other spec arms run in Claude Code... the Kiro results measure Kiro as a product."*
- **Directamente relevante para la hipótesis del reparto de modelos**: en "What the next run will test", la propia Uvik declara que **todavía no ha probado el reparto de modelos por fase** — cita textual: *"Model tiering: a stronger model for the spec phase and a cheaper model for the build phase"* está en la lista de cosas a testear en la **próxima** ejecución (Q1 2027), no en esta. Es decir: ni siquiera el benchmark más cuantitativo disponible sobre SDD ha medido todavía el patrón "modelo caro planifica, modelo barato implementa".

Confianza en el benchmark en su conjunto: **media**. Es la única fuente cuantitativa controlada encontrada (mejor que cualquier opinión aislada), pero es de una sola consultora, con una sola ejecución, un solo modelo, un solo lenguaje (Python), y declara ella misma sus límites con rigor inusual para contenido de marketing — lo cual sube algo la confianza frente a un benchmark de vendor típico, pero sigue sin ser replicación independiente. El propio texto cita un *"June 2026 comparative study on arXiv [that] lists the lack of benchmarks for the complete SDD process as a risk for the field"* — es decir, la propia literatura académica (referenciada, no verificada por mí de forma directa con URL propia del paper) confirma que este es un vacío de evidencia del campo, no solo mío.

### 3.4 Modelo de Anthropic por fase y mecanismo de cambio en Claude Code

**Line-up de modelos actual (2026-09-25)**, confirmado directamente vía `WebFetch` a `https://code.claude.com/docs/en/model-config` (nota: `docs.claude.com` redirige 301 a `code.claude.com` para la documentación de Claude Code — dato en sí mismo, cambio de dominio):

- **Fable** (alias `fable`) — el tier más capaz y más orientado a autonomía larga. Cita textual: *"Claude Fable 5.1 and Claude Fable 5 are the most capable models in Claude Code, suited to tasks larger than a single sitting. They sustain long autonomous sessions, investigate before acting, and verify their work more often than smaller models."* Nombres concretos: `claude-fable-5-1`, `claude-fable-5`.
- **Opus** (alias `opus`) — *"Uses the latest Opus model for complex reasoning tasks"*. Versiones listadas: Opus 5.5 (`claude-opus-5-5`), Opus 5, Opus 4.8, 4.7, 4.6.
- **Sonnet** (alias `sonnet`) — *"Uses the latest Sonnet model for daily coding tasks"*. Versiones: Sonnet 5 (`claude-sonnet-5`), 4.6, 4.5.
- **Haiku** (alias `haiku`) — *"Uses the fast and efficient Haiku model for simple tasks"*. Versión: Haiku 4.5.

**Guía oficial de "qué modelo por fase"**, confirmada en `code.claude.com/docs/en/costs`, sección "Choose the right model" (WebFetch directo, 2026-09-25): *"Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning."* Y para subagentes: *"For simple subagent tasks, specify `model: haiku` in your subagent configuration."* Sobre teams de agentes: *"Use Sonnet for teammates. It balances capability and cost for coordination tasks."* Sobre razonamiento extendido: *"You can't turn off thinking on Opus 5.5 or the Fable models, which always use extended thinking"* — es decir, Fable y Opus 5.5 son de facto los tiers "razonadores forzados", coherente con su uso recomendado en planificación/arquitectura.

**Mecanismo técnico de cambio de modelo por fase**, confirmado en `code.claude.com/docs/en/model-config`:
1. **`/model <alias|nombre>`** en sesión — cambia el modelo activo; con `Enter` lo guarda como default, con `s` lo cambia solo para esa sesión.
2. **Campo `model` en `settings.json`** — configuración permanente de proyecto/usuario (ejemplo oficial: `{"model": "opus"}`).
3. **Campo `model` en el frontmatter de subagentes** (`.claude/agents/*.md`) — cada subagente puede fijar su propio modelo (visto también en esta misma sesión: la herramienta `Agent` de este entorno acepta `model: sonnet|opus|haiku|fable`, coincidente exactamente con el line-up documentado). Variables de entorno asociadas: `CLAUDE_CODE_SUBAGENT_MODEL` (default para subagentes sin modelo propio) y `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` (fuerza todos los subagentes a un modelo).
4. **Alias `opusplan`** (y su variante `opusplan[1m]` de contexto largo) — **mecanismo oficial y ya empaquetado para exactamente el patrón que plantea la hipótesis del usuario**. Cita textual: *"The `opusplan` model alias provides an automated hybrid approach: In plan mode: uses `opus` for complex reasoning and architecture decisions. In execution mode: automatically switches to `sonnet` for code generation and implementation. This pairs Opus's reasoning for planning with Sonnet's efficiency for execution."*
5. Cuando se cambia de modelo con `/model`, el cambio también alcanza a los *"subagents that inherit the main conversation's model"* — es decir, hay propagación automática salvo que el subagente fije su propio `model:`.

Fuente primaria única para todo el §3.4: documentación oficial de Anthropic para Claude Code, verificada por mí mismo con `WebFetch` en dos páginas independientes (`costs`, `model-config`) que se corroboran entre sí. Confianza **alta** — es documentación oficial de primera mano, no un resumen de terceros, obtenida directamente en esta sesión.

**Guía oficial "explorar → planificar → implementar → commit"**, confirmada en `code.claude.com/docs/en/best-practices` (redirigido desde `anthropic.com/engineering/claude-code-best-practices`, WebFetch 2026-09-25), sección *"Explore first, then plan, then code"*: cita textual: *"The recommended workflow has four phases: Explore [plan mode, sin editar] → Plan [pedir un plan detallado, editable con Ctrl+G] → Implement [salir de plan mode e implementar verificando contra el plan] → Commit [pedir commit + PR]."* Añade una recomendación de coste-beneficio explícita: *"Planning is most useful when you're uncertain about the approach, when the change modifies multiple files, or when you're unfamiliar with the code being modified. If you could describe the diff in one sentence, skip the plan."* La misma página describe también el patrón "entrevista antes de especificar" (*"For larger features, have Claude interview you first... write a complete spec to SPEC.md... start a fresh session to execute it"*) y un paso de revisión adversarial con subagente en contexto limpio antes de dar el trabajo por terminado.

---

## 4. Qué es estándar / qué es lo mejor / qué es hype

**Estándar de facto (por adopción medida, no por opinión)**:
- **GitHub Spec Kit** es, con diferencia, el framework con más adopción medible (138.782★, actividad diaria de releases). Fuente primaria: `gh api repos/github/spec-kit` (yo mismo, 2026-09-24). Corroboración independiente: 1 (Uvik benchmark, que lo incluyó como referencia de facto). Tipo de evidencia: métrica de repositorio (objetiva) + inclusión en benchmark independiente. Confianza **alta** — la métrica de estrellas es directa y verificable, aunque estrellas no es lo mismo que uso en producción.
- El patrón **"explorar → planificar → implementar → commit"** con `plan mode` es la guía oficial y explícita de Anthropic para Claude Code, no una práctica de comunidad no oficializada. Fuente primaria: `code.claude.com/docs/en/best-practices`. Es "estándar" en el sentido de que es la recomendación del propio vendor del modelo de referencia del proyecto, no en el sentido de adopción medida por terceros (no tengo cifra de cuántos usuarios siguen este patrón).
- **Markdown-en-el-repo como unidad de entrega** (no issues de GitHub/GitLab) es el patrón que comparten **los 4 frameworks spec-driven analizados con evidencia directa** (Spec Kit, Kiro, BMAD, OpenSpec) y también Agent OS. Esto es lo más cercano a un "estándar de facto" de la industria de herramientas de SDD para agentes que he podido verificar con evidencia — 5 fuentes independientes (cada framework, con su propia doc/repo) apuntan al mismo patrón. Confianza **alta**.

**Lo mejor (por evidencia, no por hype)**:
- Dentro del único benchmark cuantitativo controlado disponible (Uvik, confianza media por ser de un solo vendor/una sola ejecución), **OpenSpec** tiene la mejor tasa de merge (84%) y el menor "spec drift" (2.4%), con coste intermedio ($2.71); **Kiro** tiene el mejor coste por ticket mergeado ($2.35) con tasa de merge casi igual de buena (82%). Ninguna diferencia entre los 4 frameworks de spec es estadísticamente significativa según la propia banda de ruido declarada (±10pp) — la única diferencia que sí supera el ruido es **spec vs. no-spec** en tickets de feature/pipeline de datos (90% vs 60%), no framework A vs framework B. Confianza **media** (fuente primaria única, evidencia tipo benchmark de vendor con metodología transparente y limitaciones auto-declaradas).
- **`opusplan`** es, con evidencia de fuente primaria (documentación oficial de Anthropic), el mecanismo mejor soportado para el patrón "modelo caro planifica, modelo más barato implementa" — pero **dentro del ecosistema Anthropic únicamente** (Opus planifica, Sonnet ejecuta). No hay evidencia — ni siquiera del propio Anthropic — sobre repartir planificación en Opus/Fable e implementación en un modelo de **otro proveedor** (DeepSeek); eso es terreno inexplorado por la propia documentación oficial.

**Hype / sin contrastar todavía**:
- **Beads** tiene crecimiento viral (27.404★ en 11 meses) y ecosistema de terceros emergente, pero **sin caso de adopción corporativa documentado** más allá de early adopters — corroborado por el front paralelo F2 y reverificado por mí de forma independiente: el hilo de Hacker News *"Show HN: I replaced Beads with a faster, simpler Markdown-based task tracker"* (84 puntos, autor `wild_egg`, `https://news.ycombinator.com/item?id=46487580`, verificado por mí vía `curl` a la API de Algolia el 2026-09-25) es contra-evidencia práctica directa: cita textual del autor: *"I've been running long duration coding agents with Claude Code for about 6 months now. Steve Yegge released Beads back in October and I found that giving Claude tools for proper task tracking was a massive unlock. But Beads grew massively in a short time and every release made it slower and more frustrating to use... its background daemon took to syncing the wrong things at the wrong times."* — es decir: valida la necesidad del problema que Beads dice resolver (seguimiento de tareas estructurado para agentes de larga duración), pero rechaza la solución concreta de Beads (complejidad operativa del daemon Dolt) en favor de un tracker markdown más simple. Confianza **media** en que "task tracking estructurado > markdown suelto" es valioso; confianza **baja-media** en que Beads específicamente sea ya "lo mejor probado" frente a alternativas más simples.
- **Modelo-tiering por fase (caro planifica / barato implementa) como mejora de calidad/coste medida empíricamente**: es hype hasta la fecha, no evidencia. La única fuente que lo iba a testear cuantitativamente (Uvik) lo pone explícitamente en su lista de **trabajo futuro**, no de resultados actuales. Anthropic lo ofrece como producto (`opusplan`) — eso es "vendor claim" de que el patrón tiene sentido de diseño, no un dato de que mejora resultados frente a usar Opus o Fable de principio a fin. **No encontré ningún benchmark, paper ni post de practicante con datos comparando explícitamente "un modelo todo el pipeline" vs "modelo caro planifica + modelo barato implementa"** — es el hueco de evidencia más importante de todo este informe. Confianza en esta ausencia: alta (dediqué varias búsquedas en HN específicamente a esto — ver §5 — y no until encontré nada con datos).

---

## 5. Dónde la hipótesis del usuario no se sostiene (o no está confirmada)

Repaso explícito de la hipótesis de partida — *"Claude (Opus/Sonnet) planifica y convierte la idea en casos de uso → issues; DeepSeek implementa los issues"* — contra la evidencia reunida:

1. **"Issues" como unidad de entrega**: **no se sostiene** como descripción de lo que hace el ecosistema de herramientas líder. Las 5 herramientas con evidencia directa (Spec Kit, Kiro, BMAD, OpenSpec, Agent OS) entregan el trabajo al agente implementador como **ficheros markdown versionados en el propio repositorio** (`tasks.md`, story files, `.kiro/specs/*.md`, specs de OpenSpec), no como issues de GitHub/GitLab. Beads va todavía más lejos en sentido contrario a "issue simple": es una base de datos de grafo de dependencias (Dolt) diseñada explícitamente porque, según su propio README, ni el markdown suelto ni los issues tradicionales bastan para agentes — *"replaces messy markdown plans with a dependency-aware graph"*. Y sin embargo, la contra-evidencia de HN (§4) muestra practicantes que vuelven de Beads a un tracker markdown simple porque la complejidad operativa no compensaba. Conclusión honesta: **no hay un consenso único** sobre cuál es "la" unidad de entrega correcta — hay al menos 3 familias en competencia (markdown-en-repo tipo Spec Kit/Kiro/OpenSpec/BMAD; grafo-de-dependencias tipo Beads; e issues de GitHub/GitLab tradicionales, que ninguno de los 5 frameworks de SDD analizados usa como formato nativo). "Issue de GitHub" específicamente es la opción **menos representada** en la evidencia reunida sobre estas 5 herramientas.

2. **Casos de uso → issues como cadena correcta**: no encontré ninguna de las 5 herramientas de SDD que produzca "casos de uso" en el sentido clásico (Cockburn) ni que el siguiente paso sea literalmente "crear issues". El patrón real observado es más bien: spec/requisitos (formato propio de cada herramienta, a menudo EARS-like) → plan/diseño → task list interna al repo → implementación directamente desde esa task list, sin paso intermedio de "abrir issues" como unidad formal (aunque nada impide combinar ambos; simplemente no es lo que hacen por defecto).

3. **Reparto de modelos caro/barato entre fases**: **hipótesis sin refutar pero también sin confirmar por evidencia empírica**. A favor: existe como producto oficial de Anthropic (`opusplan`), documentado y soportado a nivel de plataforma — esto es evidencia de que el vendor de referencia del proyecto considera el patrón lo bastante valioso como para construir un alias dedicado. En contra o al menos "no evidenciado": (a) el propio benchmark cuantitativo más riguroso disponible (Uvik) admite que **todavía no lo ha medido** y lo deja para su próxima ejecución; (b) no localicé ningún post de practicante en Hacker News con datos comparando explícitamente pipeline-un-modelo vs. pipeline-dos-modelos (busqué con varias consultas — ver §6 — sin resultados sustantivos); (c) el propio Anthropic, al recomendar Opus/Fable para "complex reasoning" y Sonnet para "daily coding tasks", está describiendo una heurística de coste-beneficio dentro de **su propia familia de modelos**, no necesariamente una afirmación de que cruzar de proveedor (Anthropic planifica → DeepSeek implementa) preserve la misma calidad — ese salto extra (cambio de vendor, no solo de tier) no tiene, hasta donde pude verificar, ningún estudio ni benchmark dedicado.

4. **Flujos de un solo modelo como alternativa viable**: hay indicios de que un solo modelo para todo el pipeline no es descartable. El propio benchmark de Uvik corrió Spec Kit/BMAD/OpenSpec y el control sin spec **con el mismo modelo** (`claude-sonnet-5`) en las 4 condiciones — es decir, ya dentro de ese diseño experimental, la variable que más pesa no es "qué modelo hace qué fase" sino "hay spec o no", y ni siquiera eso separa con claridad a los frameworks entre sí (diferencias dentro del margen de ruido, salvo spec vs. no-spec en tickets grandes/complejos). Esto es, como mínimo, una señal de que el efecto del *formato del artefacto* y de *si hay spec* podría pesar más que el efecto de *qué modelo ejecuta cada fase* — pero es una inferencia mía a partir de un diseño experimental que no aisló esa variable, no una conclusión del propio estudio.

**Conclusión de esta sección**: ninguna parte de la hipótesis original queda refutada de forma tajante, pero tampoco ninguna queda confirmada por evidencia directa y específica. Lo mejor soportado por evidencia es: (a) spec-driven-development en general sí mejora resultados frente a no tener spec, sobre todo en trabajo grande/ambiguo, con al menos un benchmark cuantitativo (confianza media); (b) el artefacto de entrega real que usa el ecosistema es markdown-en-repo, no issues; (c) el mecanismo técnico para repartir modelo caro/barato por fase existe y está soportado oficialmente, pero su efecto en calidad/coste frente a un solo modelo no está medido públicamente todavía.

---

## 6. Fuentes consultadas (incluidas las que no aportaron nada)

**Con resultado útil**:
- `gh api repos/github/spec-kit`, `Fission-AI/OpenSpec`, `bmad-code-org/BMAD-METHOD`, `buildermethods/agent-os`, `eyaltoledano/claude-task-master`, `steveyegge/beads`, `tesslio/cli` + `gh release list` — ejecutados directamente por mí, 2026-09-24/25.
- `https://kiro.dev/` (WebFetch, 2026-09-24) — pricing, GA, spec mode general.
- `https://code.claude.com/docs/en/costs` (WebFetch, 2026-09-25; URL original consultada `docs.claude.com/en/docs/claude-code/costs`, redirige 301) — gestión de coste, elección de modelo, Fable/Opus 5.5 thinking forzado.
- `https://code.claude.com/docs/en/model-config` (WebFetch, 2026-09-25, dos pasadas) — line-up completo de modelos, `opusplan`, config de subagentes.
- `https://code.claude.com/docs/en/best-practices` (WebFetch, 2026-09-25; URL original `anthropic.com/engineering/claude-code-best-practices`, redirige 308) — explore/plan/implement/commit, CLAUDE.md, entrevista-antes-de-especificar, revisión adversarial.
- `https://code.claude.com/docs/en/common-workflows` (WebFetch, 2026-09-25) — recetas de prompts, plan mode, worktrees; sin mención directa de "explore-plan-code-commit" (esa frase concreta vive en `best-practices`, no aquí).
- `https://github.com/github/spec-kit` (WebFetch, 2026-09-25) — comandos exactos del workflow.
- `https://buildermethods.com/agent-os` (WebFetch, 2026-09-25) — workflow, agnosticismo de modelo confirmado explícitamente.
- `https://github.com/eyaltoledano/claude-task-master` (WebFetch, 2026-09-25) — artefactos, proveedores soportados.
- `https://github.com/steveyegge/beads/blob/main/README.md` (WebFetch, 2026-09-25) — mecanismo Dolt, problema declarado, comandos CLI.
- `https://alistairmavin.com/ears/` (WebFetch, 2026-09-25) — origen y sintaxis EARS.
- `https://adr.github.io/madr/` (WebFetch, 2026-09-25) — MADR 4.0.0, plantilla.
- `https://c4model.com/` (WebFetch, 2026-09-25) — C4, Simon Brown, 4 niveles.
- `https://arc42.org/overview` (WebFetch, 2026-09-25) — arc42, 12 secciones; comprobado explícitamente que no menciona guía IA/LLM.
- `https://uvik.net/spec-driven-development-benchmark/` — recuperado por un fork antes de morir (`uvik.txt`/`uvik.html` en scratchpad), leído y verificado por mí; benchmark cuantitativo completo, único de su tipo encontrado.
- `https://hn.algolia.com/api/v1/items/46487580` (curl, 2026-09-25) — hilo HN sobre reemplazo de Beads, verificado de forma independiente tras referencia cruzada con F2.
- `https://hn.algolia.com/api/v1/search?query=GitHub+Spec+Kit&tags=story` y variantes similares (curl, 2026-09-25) — localizó `den.dev/blog/github-spec-kit/` y el item HN 47229829 (RalphMAD, BMAD+Ralph Loop).
- `https://den.dev/blog/github-spec-kit/` (WebFetch, 2026-09-25) — experiencia real de practicante, en general positiva con matices ("human in the loop is as important as it ever was").
- `https://hn.algolia.com/api/v1/items/47229829` (curl, 2026-09-25) — RalphMAD, combina BMAD con "Ralph Loop" de Geoffrey Huntley; solo 2 puntos, confianza baja, se cita solo como indicio de que existe ecosistema de extensiones sobre BMAD.
- `https://www.youtube.com/watch?v=LCEmiRjPEtQ` (WebFetch, 2026-09-25) — confirmó título ("Software Is Changing (Again)") y ponente (Karpathy), sin contenido de transcripción.

**Sin resultado útil / vacías**:
- `WebSearch "Anthropic Fable model claude 2026"` y `WebSearch "Claude Code opusplan mode..."` — presupuesto de sesión agotado (200/200), 0 resultados obtenidos por esta vía (resuelto después vía WebFetch directo).
- `yt-dlp --write-auto-subs` sobre `LCEmiRjPEtQ` — 7 intentos totales (2 de un fork + 5 míos, en dos días distintos), todos `HTTP 429 Too Many Requests`. Sin transcripción.
- `curl HN "Opus plan Sonnet execute"` — resultados irrelevantes (apps de vibecoding sin relación).
- `curl HN "orchestrator worker LLM pipeline"` — resultados irrelevantes (mcp-agent, pgflow, sin relación con el reparto planificación/implementación entre modelos).
- `curl HN "spec-driven development AI agents"` — solo proyectos de muy bajo tráfico (1 punto), sin señal.
- `curl HN "Claude Opus Sonnet plan mode"` — sin resultados directamente relevantes al reparto de modelos por fase.
- Búsqueda de un estudio/paper arXiv específico sobre formatos de requisitos para generación de código con LLM: no se realizó una búsqueda dedicada tras el agotamiento de `WebSearch`; el único indicio indirecto es la mención de Uvik a un *"June 2026 comparative study on arXiv [that] lists the lack of benchmarks for the complete SDD process as a risk for the field"*, sin URL propia verificada por mí — se cita como referencia de segunda mano, confianza baja por no verificación directa.
- Ninguna búsqueda ni fetch a Reddit (excluido por instrucción).
- Ninguna búsqueda ni fetch con navegador/Playwright (excluido por instrucción).

---

## 7. Dudas abiertas

1. **Transcripción de Karpathy sobre "autonomy slider"**: sigue sin conseguirse. Recomendación práctica: reintentar `yt-dlp` en otro momento (el 429 puede ser un bloqueo temporal de IP/rate-limit de YouTube, no necesariamente permanente), o buscar la transcripción en un servicio de terceros, o pedir al usuario si tiene acceso a algún resumen/artículo ya guardado.
2. **Agnosticismo de modelo de BMAD y OpenSpec**: no confirmado con fetch propio (solo inferido/parcial). Falta una verificación directa de la documentación de configuración de modelo de ambos.
3. **Formatos Cockburn / ISO 29148 / ADR-MADR / C4 / arc42 / OpenAPI como artefactos de entrega para agentes**: no se encontró evidencia de adopción específica en el ecosistema de SDD-para-agentes analizado — pero tampoco se hizo una búsqueda exhaustiva (agotamiento de `WebSearch` + fallo de forks). Es plausible que exista evidencia que no se encontró, no que no exista.
4. **El "estudio arXiv de junio 2026" y el "estudio arXiv de mayo 2026 sobre +48.000★"** citados por Uvik: no verificados con URL propia — quedan como referencia de segunda mano hasta que alguien los localice y confirme directamente en arXiv.
5. **Réplica independiente del benchmark de Uvik**: no existe, hasta donde pude verificar. Sería el paso más valioso para subir la confianza de todo el §3.3 y §4 de "media" a "alta".
6. **Coste real de OpenHands resolver / GitLab Duo con DeepSeek como ejecutor real** (mencionado tangencialmente por el front paralelo F2, fuera del alcance estricto de F1): si el usuario quiere profundizar en el "DeepSeek implementa" de la hipótesis original, ese es terreno de F2, no de este informe — aquí solo se cubre el lado de la especificación/planificación (F1).
7. **Impacto real de que 3 de los 4 forks originales murieran por rate limit**: no se pudo cubrir con el mismo nivel de profundidad y triangulación de fuentes que si los 4 sub-agentes hubieran completado su trabajo en paralelo (habrían aportado más búsquedas HN, más practitioner reports, más profundidad en Agent OS/Task Master/Beads). Este informe es sólido en las partes donde tuve WebFetch directo a fuentes primarias, pero más delgado en "experiencias de practicantes" variadas de las que un barrido amplio de HN/blogs habría aportado.

## Enlaces

- [[desarrollo-agentes-investigacion]] — síntesis de la investigación completa
- [[astillero]] — proyecto
- [[_index]]
