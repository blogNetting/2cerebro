---
title: Flujo de desarrollo con agentes — informe de investigación
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, orquestacion, estado-del-trabajo, revision, trazabilidad, investigacion]
zona: tecnico
---

Cómo se organiza de verdad el trabajo con agentes, desde el diseño hasta el código revisado. Qué funciona según la evidencia, qué se descarta, y qué flujo se recomienda. Diseño operable en [[flujo-agentes-arquitectura]] y puesta en marcha en [[flujo-agentes-runbook]].

## 1. Introducción

Pregunta, alcance, criterios (C1–C13), requisitos del estado del trabajo (R1–R13) y método: [[circuito-tareas-definicion]].

**Fases e informes:**
- A. Prácticas reales: [[flujo-fase-a-practicas-reales]].
- A2. Práctica a gran escala: [[flujo-fase-a2-practica-a-escala]].
- B. Inventario de herramientas: [[flujo-fase-b-implementaciones]].
- B2. Corrección del inventario: [[flujo-fase-b2-inventario-corregido]].
- C1. Especificación y estado del trabajo: [[flujo-fase-c1-spec-y-estado]].
- C2. Orquestación, ejecución, revisión y trazabilidad: [[flujo-fase-c2-orquestacion-ejecucion-revision]].

Los subagentes trabajaron en Sonnet 5. La revisión, la verificación y esta síntesis las ha hecho el orquestador (Opus 5.5).

**Leyenda:**
- ✔︎ = dato verificado por el orquestador en fuente primaria.
- **Propuesta mía** = inferencia de diseño, no hallazgo.

## 2. Correcciones del orquestador a los informes de fase

| Informe | Error | Corrección |
|---|---|---|
| F1 (anulado) y B | Presentaban como literal de la documentación de subagentes de Claude Code la frase «No explicit task queue… Coordination is implicit». **No existe** en la página | Retirada. Lo cierto: los subagentes no comparten cola; los *agent teams* sí ✔︎ |
| Este informe (arquitectura, 2026-09-25) | El primer YAML de `implementar.md` tenía `lock-for-agent` en la raíz de `on`, cosa que no describe ningún ejemplo de la documentación de gh-aw que se leyó — nadie lo probó a compilar antes de escribirlo | Corregido al compilar de verdad con `gh aw compile` (extensión oficial instalada, repo de prueba local): el compilador rechazó la clave y dio la ubicación correcta (`on/issues`); la versión corregida sí compila, ver [[flujo-agentes-arquitectura]] §7.1 |
| F1 (anulado) | Afirmaba que Claude Code no tiene reserva de tareas | Falso: *agent teams* tiene «Task claiming uses file locking» ✔︎ |
| B | Decía haber clasificado todo repo de 500★ o más y faltaban Symphony, Backlog.md, Vibe Kanban, Claude Squad, Gas Town y BMAD | Completado en B2 |
| B2 | Paráfrasis de Codex Cloud presentada como cita literal | Corregida en B2 |
| B | Justificaba Beads por la fama del autor | Retirado; Beads se evalúa solo por evidencia |
| B2 | Dejaba `deepseek-ai/deepseek-harness` «a vigilar» | Evaluado en C2: tiene modo *headless* ✔︎ |
| A2 | En «roles» atribuía a Stripe «Codex dirigido por 3→7 ingenieros», que es del equipo de OpenAI | Stripe usa *minions*, basados en goose ✔︎ |
| Previo | Benchmark de Uvik con fecha de ejecución futura | Excluido ([[desarrollo-agentes-investigacion]]) |

## 3. Considerado y descartado

| Opción | Motivo | Evidencia |
|---|---|---|
| **Beads como almacén del estado** | Cumple sobre el papel la mayoría de R1–R13, pero falla en R13 (coste operativo) con fallos abiertos y sin negar por los mantenedores:<br>- TTL de reserva «accepted but inert» ([#5681](https://github.com/gastownhall/beads/issues/5681))<br>- demonios `dolt sql-server` huérfanos a ~38 % de CPU y ~2 GB ([#4282](https://github.com/gastownhall/beads/issues/4282))<br>- bloqueo que congela la herramienta ([#3415](https://github.com/gastownhall/beads/issues/3415))<br>- datos obsoletos entre agentes concurrentes ([#4746](https://github.com/gastownhall/beads/issues/4746))<br>- corrupción del journal ([#4521](https://github.com/gastownhall/beads/issues/4521), [#6064](https://github.com/gastownhall/beads/issues/6064))<br>- testimonio de abandono tras 6 meses en producción ([HN 84 pts](https://news.ycombinator.com/item?id=46487580)) | Todas las issues abiertas ✔︎ |
| Claude Code *agent teams* como almacén | Solo modelos Claude (incompatible con ejecutor intercambiable); estado solo local, «never uploaded»; experimental; tareas que «can lag» sin recuperación automática | [agent-teams](https://code.claude.com/docs/en/agent-teams) ✔︎ |
| OpenAI Symphony tal cual | El ejecutor tiene que hablar el protocolo *app-server* de Codex («MUST speak a compatible app-server protocol») ✔︎; estado de orquestación en memoria; «engineering preview»; sin adopción documentada | [SPEC.md](https://github.com/openai/symphony/blob/main/SPEC.md) ✔︎ |
| Gas Town | Envuelve Beads y hereda su coste (Dolt); hilo crítico sobre consumo opaco de cuota | [[flujo-fase-c2-orquestacion-ejecucion-revision]] §2.4 |
| humanlayer como capa de aprobación | El propio repo: «the code here is pretty much all deprecated» | [README](https://github.com/humanlayer/humanlayer) ✔︎ |
| wedow/ticket | Sin ningún mecanismo de concurrencia (cero `flock` en el script) | [[flujo-fase-c1-spec-y-estado]] §3.1 |
| Task Master | Sin commits desde 2026-04-23 | [[flujo-fase-c1-spec-y-estado]] §4 |
| Enjambres y muchos agentes concurrentes como punto de partida | En un enjambre, el 54,5 % de los commits fue coordinación ✔︎; los practicantes convergen en 1–2 agentes a la vez; DORA 2025 muestra que la IA empeora la estabilidad | [kiankyars](https://kiankyars.github.io/machine_learning/2026/02/12/sqlite.html) ✔︎, [[flujo-fase-a-practicas-reales]] P1/P6 |
| Revisión solo agente-a-agente, sin datos | Ninguna de las cuatro organizaciones con cifras (OpenAI, Stripe, Spotify, Ramp) publica su tasa de defectos tras el merge | [[flujo-fase-a2-practica-a-escala]] P22 |
| affaan-m/ECC, ruvnet/ruflo, oh-my-codex | Estrellas sin explicar por evidencia independiente | [[flujo-fase-b-implementaciones]] §6, B2 |
| CrewAI, AutoGen, MetaGPT, LangGraph | Generalistas, no pensados para diseño → PR → revisión | B, B2 |

## 4. Análisis: las seis preguntas

### 4.1 Roles

- **Práctica a escala.** «Las personas dirigen. Los agentes ejecutan» (OpenAI). En Stripe hay más de 1.300 PRs por semana «completely minion-produced, human-reviewed» ✔︎. En Ramp, alrededor del 30 % de las PRs las escribe su agente ✔︎. En Spotify, más de 1.500 PRs ✔︎.
- **Reparto por modelo.** El reparto planificador fuerte → implementador barato existe en la práctica, con un fallo documentado: la **deriva del plan** (el implementador reinterpreta lo que se dejó en prosa) ([Reddit](https://www.reddit.com/r/ClaudeAI/comments/1v7h42s/plan_drift_between_opus_5_planning_and_sonnet_5/)). Anthropic lo empaqueta con el alias `opusplan` ✔︎. No hay benchmark que demuestre que repartir por fases mejora frente a usar un solo modelo: es un hueco de evidencia.
- **Separación de poderes.** Quien pide un cambio no puede aprobarlo: «this would allow for any user to approve their own changes» (Ramp) ✔︎.

### 4.2 Traspaso

- **Contrato verificable, no prosa.** Interfaces exactas, criterios de aceptación y casos límite. Lo sostienen el hilo de la deriva del plan, [until-dev](https://github.com/until-dev/plugins) y la preparación en dotnet/runtime: del 38 % al 69 % de éxito solo por documentar el repo ✔︎.
- **Especificación versionada en el repo** como artefacto de primera clase: coinciden el foro, el estudio académico [arXiv 2607.10856](https://arxiv.org/abs/2607.10856) y OpenAI (`AGENTS.md` como índice y `docs/` como base de conocimiento).
- **Formato.** Los frameworks spec-driven (Spec Kit, OpenSpec, BMAD, Kiro) generan specs, planes y tareas en markdown. Kiro usa la notación **EARS**: plantillas de requisito del tipo «Cuando <disparador>, el <sistema> deberá <respuesta>».

### 4.3 Coordinación

- **Git y el tracker son la base.** Symphony formaliza el patrón: el tracker es la fuente de verdad, un único orquestador reserva el trabajo y un espacio de trabajo aislado por issue ✔︎.
- **Hueco confirmado.** Ningún sistema ligero y maduro resuelve a la vez la reserva atómica (R3) y la caducidad de reservas abandonadas (R4). Beads los tiene, pero con R13 en contra. GitHub Issues no: la asignación admite hasta 10 personas, no es exclusiva ✔︎.
- **Primitivas nativas que sí lo resuelven, verificadas en su documentación:**
  - Los **grupos de concurrencia de GitHub Actions**: «there can be at most one running job or workflow in a concurrency group at any time» ✔︎.
  - **`timeout-minutes`**: «The maximum number of minutes to let a job run before GitHub automatically cancels it» ✔︎.
  - **`git push --force-with-lease=<ref>:`**, que con valor esperado vacío exige que «the named ref must not already exist» ✔︎.
  
  **Propuesta mía:** usarlas como reserva y caducidad en lugar de una base de datos de tareas.

### 4.4 Topología

- **Uno o dos agentes simultáneos** en la práctica individual.
- **Paralelismo sobre tareas independientes**, cada una aislada (Stripe, Ramp). No sobre un mismo problema repartido entre muchos agentes.
- **Orquestador y trabajadores** multiplica el coste por unas 15 veces frente a un chat ✔︎ (Anthropic, en investigación, no en código).
- **Co-Coder:** partir el trabajo según las dependencias del código mejora hasta un 14 % frente a los *agent teams* de Claude Code ✔︎ ([arXiv 2606.00953](https://arxiv.org/abs/2606.00953)).

### 4.5 Revisión

- **Rondas acotadas antes de escalar a una persona.** Stripe: «at most two rounds of CI» ✔︎. Symphony: *backoff* exponencial con tope.
- **Qué hay en la práctica.** Un modelo que evalúa el diff antes que la persona (Spotify, «LLM as a judge») ✔︎. Revisar el plan en lugar del código (P4). La mayoría de PRs de agente no reciben revisión humana ([arXiv 2605.02273](https://arxiv.org/abs/2605.02273)) ✔︎.
- **Tests.** Los escritos por IA dejan sobrevivir más mutantes a igual cobertura ([[desarrollo-agentes-f4-devsecops]]).

### 4.6 Trazabilidad

Mecanismos verificados, sin firma criptográfica del modelo que hizo cada cambio (sigue siendo un hueco):
- Identidad de bot y *trailers* en el commit: `Co-Authored-By: <modelo>` y `Claude-Session` ✔︎ ([[desarrollo-agentes-investigacion]]).
- OpenTelemetry de Claude Code.
- Registro de ejecución verificable sin conexión en open-multi-agent.
- Logs de las ejecuciones de Actions.

## 5. Recomendaciones

1. **Flujo recomendado: nativo de GitHub y dirigido por eventos**, detallado en [[flujo-agentes-arquitectura]]:

   | Qué | Cómo |
   |---|---|
   | **Diseño** | Opus 5.5 en Claude Code local, con la suscripción Pro. Framework spec-driven elegido en cada proyecto |
   | **Traspaso** | Issues de GitHub con contrato (sub-issues y dependencias nativas ✔︎) |
   | **Reserva y caducidad** | Grupos de concurrencia y `timeout-minutes` de Actions ✔︎, más un reconciliador periódico |
   | **Ejecución** | DeepSeek dentro de GitHub Actions mediante gh-aw o claude-code-action, con el endpoint compatible con Anthropic ✔︎ |
   | **Revisión** | Gates deterministas, Opus en Actions con tu suscripción (`claude setup-token` ✔︎) y aprobación humana en rutas sensibles |
   | **Integración** | Merge queue |
   | **Trazabilidad** | *Trailers*, bot y logs de ejecución |

2. **Alternativa local** (sin Actions para ejecutar): CAO de AWS Labs, que admite OpenCode con el modelo que se configure ✔︎, sobre el mismo almacén de estado.
3. **No adoptar Beads** mientras sigan abiertos sus fallos de R13. Si se cierran, reevaluar: resuelve R10 (economía de contexto) y R11 (memoria del proyecto) mejor que las demás opciones.
4. **Empezar con 1–2 ejecutores**; aumentarlos solo si la estabilidad (fallos tras el merge, trabajo rehecho) no empeora.

## 6. Dónde se ha buscado

- **Documentación oficial:** Claude Code (subagents, agent-teams, hooks, model-config, costs, best-practices, github-actions, monitoring), GitHub (Actions concurrency y workflow-syntax, issues, sub-issues, dependencies, rulesets), gh-aw (engines, triggers, safe-outputs, frontmatter), DeepSeek (Anthropic API, Responses API, pricing, kv_cache), Kiro, OpenCode, Codex, CAO, Symphony SPEC.md y git push.
- **Informes de práctica:** OpenAI, Stripe (partes 1 y 2), Ramp, Spotify, Anthropic (dos posts de ingeniería), Eventual, experimento del enjambre SQLite.
- **Estudios:** arXiv 2601.15195, 2605.02273, 2601.21194, 2603.15911, 2607.10856, 2509.14745, 2606.03115 (SPOQ) y 2606.00953 (Co-Coder).
- **Comunidad:** Hacker News (API de Algolia; decenas de consultas, listadas en cada informe) y Reddit mediante el Chrome real (r/ExperiencedDevs, r/ClaudeAI, r/ChatGPTCoding).
- **GitHub:** más de 600 repos barridos; los de 500★ o más, clasificados (B y B2).
- **Sin aporte o bloqueado:**
  - Búsqueda web agotada (200/200).
  - API de arXiv con 406 por `curl` (sí funciona vía WebFetch).
  - API de *stargazers* con 404, que impidió verificar la inflación de estrellas por fechas.
  - Transcripción de Karpathy con 429.
  - Página de modelos de Kiro con 404.
  - r/ClaudeCode y r/LocalLLaMA no recorridos en la fase A.

## 7. Otros

- **Huecos que la investigación no ha podido cerrar:**
  - Tasa de defectos tras el merge del código de agentes (nadie la publica).
  - Comparación controlada entre patrones.
  - Firma verificable del modelo autor.
  - Adopción real de `deepseek-harness` (la única pregunta de uso en producción en HN quedó sin respuesta ✔︎).
- **Riesgo de coste.** Opus revisando en Actions consume tu cuota Pro, y Anthropic no publica sus límites. Por eso la revisión de Opus va **después** de que la CI esté en verde (sección 5).

## 8. Qué se hace así en la práctica, qué no, y qué encaja con lo que habíamos hablado

| Idea de partida | Qué dice la práctica | Encaje |
|---|---|---|
| Opus diseña y genera el trabajo | Se hace (planificador fuerte, `opusplan`). Condición: contrato verificable, o aparece la deriva del plan | ✅ con condición |
| Un modelo más barato implementa | Existe en la práctica y es técnicamente viable (endpoint de DeepSeek compatible con Anthropic y Codex ✔︎). **No hay evidencia de que mejore la calidad**; se justifica por coste, que es tu motivo | ✅ por coste |
| Algo guarda el trabajo pendiente entre medias | Sí: el tracker como fuente de verdad (Symphony) o la especificación en el repo (OpenAI). Los trackers específicos para agentes (Beads) tienen más fallos de los que prometen | ✅, pero con GitHub Issues y primitivas de Actions, no con Beads |
| Enjambre de ejecutores | No es lo que funciona: 1–2 concurrentes y paralelismo sobre tareas independientes aisladas | ⚠️ empezar con 1–2 |
| El resultado se revisa | Sí, con rondas acotadas y revisión humana en lo sensible. La revisión solo entre agentes existe (OpenAI), pero sin datos de defectos | ✅ con gates deterministas y aprobación humana |

## Enlaces

- [[flujo-agentes-arquitectura]] — diseño operable
- [[flujo-agentes-runbook]] — puesta en marcha paso a paso y comprobación de coherencia
- [[circuito-tareas-definicion]] — definición de la fase
- [[sistema-desarrollo-con-agentes]] — proyecto
- [[_index]]
