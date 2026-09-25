---
title: Flujo con agentes, fase A: prácticas reales
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, practicas, investigacion]
zona: tecnico
---

Quince patrones de cómo se organiza de verdad el trabajo con agentes (foros, estudios, equipos), con su evidencia y si están contrastados.

Informe de fase; lo ha revisado el orquestador. Síntesis en [[flujo-agentes-informe]]. Las correcciones del orquestador están en esa síntesis.

Investigación hecha en sesión única (sin subagentes, por encargo explícito). Fecha: 2026-09-25.

Objetivo: cómo se organiza REALMENTE el trabajo de desarrollo con agentes de IA, desde el diseño hasta el código revisado — partiendo de la práctica, no de las herramientas. Sigue la definición de [[circuito-tareas-definicion]] (6 preguntas, criterios C1–C13, definición de "contrastado" en su sección 7).

**Nota de reutilización:** [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] ya cubrió con evidencia sólida: el caso dotnet/runtime (878 PRs Copilot coding agent, 10 meses), el dataset AIDev/AIDev-pop y sus 4 papers derivados sobre merge rate y conflictos, el post de Cognition "Don't Build Multi-Agents", y los mecanismos de atribución de commits (`Co-authored-by`, GitHub App bots). No se repite esa evidencia aquí; se cita por referencia donde aplica a las 6 preguntas y se re-verifica solo lo que se reutiliza de forma literal (ver nota en cada cita).

---

## 1. Método y consultas

**Orden seguido:** HN Algolia (muchas variantes) → lectura de hilos con más señal → WebFetch de blogs de equipos/vendors citados en esos hilos → arXiv (página de búsqueda web, no API — bloqueada) → Reddit vía navegador Chrome real (`old.reddit.com`/`reddit.com`, `browser_navigate` + `browser_evaluate`, sin login). WebSearch no se usó (cupo agotado, según el encargo).

### HN Algolia (`hn.algolia.com/api/v1/search`, tags=story, hitsPerPage 6–8)

Consultas con resultado útil: "how we use coding agents", "agents in production workflow", "planner implementer agents", "agent code review process", "parallel agents experience", "spec driven development agent", "orchestrator worker agents coding", "code review bottleneck AI generated PRs", "human in the loop agent coding", "task queue for coding agents", "issue tracker for AI agents beads", "worktree parallel agents merge conflicts".

Consultas SIN resultado útil (vacías o solo ruido irrelevante): "cheap model implement expensive model plan", "Copilot internal engineering workflow report", "AI pair programming roles", "engineering team AI agent adoption report" (solo un resultado de producto, no de práctica), "until-dev plan review" (0 hits), "Sourcegraph Amp internal dogfood" (0 hits), "how we dogfood our own coding agent" (0 hits relevantes), "spec kit GitHub team practice" (0 hits), "Shopify AI agents engineering" (0 hits relevantes), "Replit agent internal workflow" (0 hits relevantes), "Canva engineering AI coding agents" (0 hits relevantes), "we let AI write all our code experience report" (0 hits relevantes), "multi-agent coding failed" (solo 2 resultados tangenciales), "AI teammate workflow" (0 resultados de práctica real, solo productos).

### arXiv (`arxiv.org/search/?query=...&searchtype=all`, verificado cada paper en `arxiv.org/abs/<id>`)

Consultas con resultado: "multi-agent LLM software engineering workflow roles" (mayormente marcos conceptuales, poco empírico — descartados la mayoría), "empirical study developers AI coding agent workflow practice", "how developers use AI coding agents in industry survey", "case study software team adopting autonomous coding agents", "human oversight AI generated pull requests study", "interview study developers coding agents software engineering practice".

Todas las consultas devolvieron algo; ninguna vacía. Se priorizaron papers empíricos (encuestas, entrevistas, minería de datos reales) sobre propuestas de arquitectura (MetaGPT, ChatCollab, NOMAD, etc. — descartados por no ser evidencia de práctica real, ver sección 2).

### Reddit (Chrome real vía CDP, sin login)

Subreddits recorridos: r/ExperiencedDevs, r/ClaudeAI, r/ChatGPTCoding. Búsquedas con resultado útil: "AI agents team workflow" (r/ExperiencedDevs), "planner implementer model" (r/ClaudeAI), "abandoned multi-agent workflow" (r/ChatGPTCoding). No se buscó en r/ClaudeCode ni r/LocalLLaMA por límite de tiempo — hueco, ver sección 6.

---

## 2. Catálogo de patrones de organización observados en la práctica

Para cada patrón: qué es, qué preguntas de la sección 1 de [[circuito-tareas-definicion]] responde, quién lo usa, resultados/números reportados, fallos/abandono reportados, tipo de evidencia, nº de fuentes independientes, confianza, y si cumple la definición de "contrastado" (§7 del documento de definición: ≥2 fuentes independientes, o 1 fuente primaria con datos cuantitativos propios).

### P1. Sesión única, un humano con control total, sin delegar en otros humanos ni en enjambres de agentes

- **Preguntas que responde:** roles (todos los roles los ocupa un único agente + un único humano), topología (uno).
- **Quién lo usa:** desarrollador solo, 10 años como único dev de su empresa, describe haber sustituido devs externos por Claude Code porque el tiempo de dar contexto a un humano nuevo supera el ahorro. [Reddit r/ExperiencedDevs, hilo "How do you cope with multi agent workflows?"](https://www.reddit.com/r/ExperiencedDevs/comments/1ten4yg/how_do_you_cope_with_multi_agent_workflows/)
- **Resultado reportado:** con el plan Claude Code Max 5x, rara vez llega al límite de 5h; la mayor parte del tiempo la dedica a arquitectura, plan y revisión, no a escribir prompts en paralelo — cifra: "reach 50% max [uso de cuota]" — **sin verificar** (autorreporte, sin cifra externa).
- **Fallos/abandono reportado:** el propio hilo es una reacción contra la narrativa de "10x agentes en paralelo"; varios comentaristas independientes confirman que solo gestionan 1–2 sesiones simultáneas de forma sostenible (`mechkbfan`: "I can basically do two at moment"). Otro comentario (`originalchronoguy`) describe haber montado una demo de 10 agentes concurrentes tipo "equipo de 4-5 ingenieros + QA + BA" y que cuesta de 1.200 a 3.500 $ por función según velocidad exigida — presentado como advertencia, no como recomendación.
- **Tipo de evidencia:** hilo de práctica de primera mano en foro técnico (Reddit), múltiples autores independientes coincidiendo.
- **Fuentes independientes:** 1 hilo, pero con ≥5 comentaristas distintos coincidiendo en el mismo límite práctico (1-2 agentes concurrentes) sin conocerse entre sí.
- **Confianza:** media — coincidencia de varios profesionales anónimos en un foro, sin cifras duras, pero consistente con P2/P8 (mismo límite práctico reportado también en r/ClaudeAI).
- **Contrastado:** sí, por convergencia de múltiples fuentes independientes anónimas apuntando en la misma dirección (aunque cada una individualmente es débil).

### P2. Modelo fuerte planifica / modelo más barato o distinto implementa, con el plan convertido en "contrato" ejecutable

- **Preguntas que responde:** roles (planificador ≠ implementador, ambos modelos), traspaso (qué debe contener el plan para no perderse en la implementación).
- **Quién lo usa:** patrón discutido activamente por varios usuarios de Claude Code con Opus 5 planificando y Sonnet 5 implementando — exactamente la hipótesis de partida del usuario. [Reddit r/ClaudeAI, "Plan drift between Opus 5 (planning) and Sonnet 5 (implementation)"](https://www.reddit.com/r/ClaudeAI/comments/1v7h42s/plan_drift_between_opus_5_planning_and_sonnet_5/).
- **Fallo reportado y consenso de solución:** "plan drift" — el modelo implementador reinterpreta partes del plan dejadas como prosa/intención en vez de contrato explícito, y Claude Code vuelve a generar un plan ad hoc al empezar cada fase si el plan original no se referencia literalmente. Solución convergente de varios comentaristas independientes: (a) el plan debe ser una lista de cambios concretos verificables ("create X, change Y so it does Z, done when tests pass"), no objetivos en prosa; (b) especificar interfaces exactas, nombres de fichero/función, ejemplos entrada→salida y casos límite explícitos; (c) un paso "antes de empezar" donde el modelo implementador repite las restricciones con sus propias palabras, para detectar malentendidos antes de escribir código. Un comentarista (`gregerw`) describe un flujo de 4 fases ya estabilizado: **research → plan → implement → verify**, con el modelo fuerte (Opus o "fable") en research/plan/verify y el barato (Sonnet) solo en implement.
- **Herramienta de terceros que materializa justo este patrón** (evidencia independiente, no citada por nadie del hilo): [`until-dev/plugins`](https://github.com/until-dev/plugins) — "Define intent in a Plan, let the agent run, and check every pull request against it." Verificado en vivo vía API de GitHub: 31★, creado 2026-07-30, último push 2026-09-17 — proyecto real y activo, aunque pequeño.
- **Tipo de evidencia:** hilo de práctica de primera mano con múltiples autores convergiendo en la misma solución de forma independiente + una herramienta open-source real que confirma el patrón sin haber sido mencionada en el hilo.
- **Fuentes independientes:** ≥4 comentaristas distintos + 1 herramienta open source independiente = 2 tipos de evidencia distintos apuntando a lo mismo.
- **Confianza:** alta en que el fallo ("plan drift") es real y recurrente; media-alta en que la solución de "plan como contrato" lo resuelve (autorreportado, sin medición antes/después).
- **Contrastado:** sí — cumple la definición de §7 (≥2 fuentes independientes).

### P3. Pipeline personal con "lanes": diseñador → implementador → revisor → integrador, todos agentes, con escalado a humano solo en excepciones

- **Preguntas que responde:** las 6: roles (5 tipos de agente + 1 humano solo para excepciones), traspaso (tarea de beads con estado), coordinación (issue tracker + lock, worktree por tarea), topología (varias "lanes" paralelas dentro de una sesión), revisión (agente revisor con un reintento antes de PR; CI como segundo gate), trazabilidad (parcial — no se detalla explícitamente).
- **Quién lo usa:** un desarrollador independiente, comentario detallado en HN. [HN, comentario de `404softwarelabs` en "Ask HN: What happens to code review process when using LLMs?"](https://news.ycombinator.com/item?id=49462511)
- **Flujo descrito literalmente:** beads para trackear tareas → una sesión de Claude Code con varias "lanes" dinámicas, cada lane: toma tarea lista → agente diseñador (si hay cambios de UI) → agente implementador (en git worktree) → agente revisor → si rechaza, un reintento de implementación → si aprueba, crea PR → cuando CI está verde (tests, auditoría de vulnerabilidades) marca "lane-verified" → un agente "lander" separado rebasa contra master, asegura que master sigue verde, mergea y despliega, y devuelve la tarea al pipeline si el conflicto es complejo o el CI queda rojo tras el rebase.
- **Rol humano declarado:** "no miro el código, no verifico tareas individuales"; solo atiende tareas marcadas "needs my attention" (decisiones, configuración manual) y revisa periódicamente cómo funciona el producto en producción, reportando bugs con herramienta propia.
- **Resultado reportado:** "50-100 PRs al día" según su capacidad de resolver preguntas — **sin verificar**, autorreporte de una sola persona sin cifra externa ni repo público citado. Cuello de botella declarado: su propia atención a las preguntas escaladas, y la CI (tuvo que montar un servidor de CI dedicado).
- **Tipo de evidencia:** práctica reportada de primera mano, sin corroboración cuantitativa externa.
- **Fuentes independientes:** 1.
- **Confianza:** baja-media — detallado y coherente internamente, pero autorreporte anónimo de una persona sin repo/cifra verificable; la cifra de 50-100 PR/día es extraordinaria y no se puede contrastar.
- **Contrastado:** NO según §7 — una sola fuente, sin datos verificables externamente. Se documenta como patrón interesante, no como práctica probada.

### P4. Revisión desplazada un nivel de abstracción: "plan review" en vez de "code review"

- **Preguntas que responde:** revisión (qué se revisa y por quién), traspaso (el plan como artefacto central de revisión, no el código).
- **Quién lo usa:** equipo real de 5 desarrolladores. [HN, comentario de `dylanratcliffe`](https://news.ycombinator.com/item?id=49483931).
- **Descripción:** en vez de revisar el código generado, el equipo revisa el plan de implementación entre pares (una capa de abstracción más arriba) y luego un bucle automático compara la implementación final contra el plan aprobado y verifica que se siguió. Permite entender "cómo" se resuelve algo (p. ej. "vamos a hacer esta migración de BD así") sin leer el SQL generado línea a línea. Han construido una herramienta propia para esto: [`until-dev/plugins`](https://github.com/until-dev/plugins) (ver P2 — mismo repo, confirma independientemente el patrón).
- **Resultado/fallo reportado:** ninguno explícito de fallo; se presenta como solución operativa a la tensión "generar funciones más rápido de lo que se puede revisar tradicionalmente y aun así entender la arquitectura".
- **Tipo de evidencia:** práctica reportada de primera mano de un equipo real (no un individuo) + herramienta open source propia verificable en GitHub.
- **Fuentes independientes:** 1 relato + 1 artefacto de software verificable de forma independiente (el repo existe, tiene actividad reciente) = evidencia primaria con verificación cruzada parcial.
- **Confianza:** media — un solo equipo, pero con verificación cruzada de que la herramienta existe y hace lo descrito.
- **Contrastado:** parcialmente — no llega a 2 fuentes independientes narrando el mismo patrón, pero sí tiene una fuente primaria con "datos" (el repo real, su descripción y actividad) que corrobora el relato. Se marca **medio**, no descartado.

### P5. Revisión delegada por completo a un agente, sin lectura humana — patrón de degradación, no recomendado

- **Preguntas que responde:** revisión (quién revisa realmente).
- **Quién lo usa:** reportado como práctica real (no deseable) por al menos 2 fuentes independientes:
  1. [Reddit r/ExperiencedDevs](https://www.reddit.com/r/ExperiencedDevs/comments/1tw5622/): "Asked my manager to review a PR I made last Thursday. He sent me a markdown file of a review his Codex had done on my PR. He didn't even read it, didn't look at my code at all either." (`Leather-Rice5025`); otro usuario en el mismo hilo: "We just have codex review the prs. Skips this [el desgaste]" (`orbital_trace`).
  2. Confirmación académica independiente: el estudio ["These Aren't the Reviews You're Looking For" (arXiv 2605.02273, mayo 2026)](https://arxiv.org/abs/2605.02273), sobre el dataset AIDev (GitHub), encuentra que **la mayoría de PRs generadas por IA no reciben ninguna revisión**, y cuando la reciben, predomina la "interacción mediada por automatización" (agente revisando a agente) sobre la revisión humana directa; la participación humana, cuando existe, se expresa más como "dirigir al agente" que como evaluación independiente del código.
- **Resultado reportado:** en el hilo de Reddit se enlaza explícitamente con burnout del equipo ("So so burnt out!", `FunTooth3`) — el bucle "pide a Claude que revise el ticket → implementa → verifica → PR → pide a Claude que revise el MR → verifica" descrito por `79215185-1feb-44c6` en el mismo hilo se presenta como fuente de agotamiento, no de alivio, especialmente en código de bajo nivel que el modelo no conoce bien.
- **Tipo de evidencia:** anécdotas de práctica de primera mano (Reddit) + estudio académico independiente sobre datos reales de GitHub — dos tipos de evidencia distintos, mismo hallazgo.
- **Fuentes independientes:** 2 (Reddit + paper), sin relación entre sí.
- **Confianza:** alta en que el fenómeno ocurre y es común; el paper da la única cifra poblacional real de "cuánta revisión humana hay de verdad", aunque su resumen (obtenido en esta sesión) no incluyó el porcentaje exacto — **sin verificar** la cifra precisa, solo la dirección del hallazgo.
- **Contrastado:** sí (2 fuentes independientes, un tipo con datos poblacionales).

### P6. Enjambre de agentes homogéneo/heterogéneo con partición estricta por módulo y coalescer, sin rol de revisión formal

- **Preguntas que responde:** las 6, con topología como foco: 1 agente bootstrap + 6 agentes trabajadores (2×Claude, 2×Codex, 2×Gemini) corriendo en continuo + 1 agente "coalescer" (Gemini) que debía deduplicar y resolver divergencia entre implementaciones paralelas.
- **Quién lo usa:** experimento individual publicado como blog técnico: reimplementar SQLite en Rust con un "small swarm". [Blog, "Building SQLite with a small swarm", 12-feb-2026](https://kiankyars.github.io/machine_learning/2026/02/12/sqlite.html) (106 puntos / 120 comentarios en HN, discusión activa).
- **Coordinación:** git como mecanismo central — cada agente hace pull de main, reclama UNA tarea acotada vía fichero de lock (`current_tasks`), implementa localmente, testea contra sqlite3 real como oráculo, actualiza documentos de estado compartido (`PROGRESS.md`, directorio de notas) y hace push.
- **Resultado cuantitativo (fuente primaria con cifras propias):** ~19.000 líneas de Rust en 154 commits en 3 días (10–12 feb 2026); 282 tests unitarios generados por los agentes; 64 consultas SQLLogicTest pasando (CRUD, JOIN, GROUP BY). Codex agotó el 100% de su cuota semanal; Claude usó el 70%.
- **Fallo reportado explícitamente por el propio autor:** **54,5% de los commits (84 de 154) fueron overhead de coordinación** (lock/claim/stale-lock/release) — es decir, más de la mitad del esfuerzo de commits no fue trabajo útil sino gestión de la propia coordinación. El agente "coalescer" solo se ejecutó una vez, al final del proyecto, y Gemini no logró completar la deduplicación. No se implementó concurrencia real; quedaron búsquedas de freelist lineales y clones de buffer redundantes sin optimizar.
- **Tipo de evidencia:** informe de práctica primario con datos cuantitativos propios de un experimento real (no vendor-claim, autor independiente).
- **Fuentes independientes:** 1 fuente primaria con datos cuantitativos propios — cumple el criterio alternativo de §7 ("una fuente primaria con datos cuantitativos" basta si no hay 2 independientes).
- **Confianza:** media — un solo experimento, de 3 días, sin repetición ni comparación con un baseline sin swarm; pero las cifras son propias, verificables (repo/commits) y la cifra de "54,5% overhead de coordinación" es exactamente el tipo de evidencia en contra que pide C13 y que rara vez se reporta.
- **Contrastado:** sí, según la vía alternativa de §7 (fuente primaria + datos cuantitativos), con confianza media, no alta.

### P7. Modelo "human-led, agent-assisted" con fases y checkpoints de commit, sin tablero centralizado

- **Preguntas que responde:** roles (humano retiene arquitectura/revisión/integración; agente hace implementación/tests/boilerplate), traspaso (hash de commit como "testigo" entre fases: *"I've committed this with git hash X. Let's continue to phase two"*), coordinación (git worktrees, sin tablero de tareas centralizado — coordinación conversacional), topología (3-5 pestañas de terminal en paralelo, agente único dentro de cada fase), revisión (Greptile para revisión automática de PR + revisión conversacional humana del diff).
- **Quién lo usa:** Eventual (antes daft.ai), empresa con producto real (Daft, motor de datos distribuido). [Blog de empresa, "How We Use AI Coding Agents", eventual.ai, ~dic-2025](https://www.eventual.ai/blog/how-we-use-ai-coding-agents) (también indexado en HN con [46278056](https://news.ycombinator.com/item?id=46278056) y [46316161](https://news.ycombinator.com/item?id=46316161)).
- **Resultado reportado:** equipos usando 3-5 pestañas de Claude Code en paralelo dicen haber resuelto un issue de CI "de un año de antigüedad" con este método — **sin verificar** cifra ni caso concreto, es una afirmación cualitativa del propio equipo.
- **Fallo/límite reportado explícitamente:** el modelo falla en código que toca varios sistemas a la vez (ejemplos de auth dados), y en concurrencia — cita literal: un agente escribió un test que pasaba usando `sleep(5)`, degradando la UX real en producción. También citan el riesgo de "a wall of changes you can't track" (una pared de cambios imposible de rastrear) si no se fuerza el checkpoint por fases con commits descriptivos.
- **Tipo de evidencia:** informe de práctica de una empresa real con producto, sin cifras cuantitativas duras (a diferencia de dotnet/runtime en F2).
- **Fuentes independientes:** 1.
- **Confianza:** media — vendor/empresa hablando de su propio proceso interno, sin verificación externa, pero es información operativa concreta (no marketing de producto de terceros) y coincide en varios puntos (worktrees, checkpoints por fase, revisión conversacional) con P2, P3 y P9.
- **Contrastado:** NO por sí solo (1 fuente, sin datos cuantitativos propios) — se apoya en su coincidencia con otros patrones para ganar credibilidad, pero individualmente queda en "reportado, no contrastado".

### P8. Dogfooding heterogéneo por equipo, sin coordinación formal — Anthropic sobre sí misma

- **Preguntas que responde:** roles y topología, de forma fragmentaria por equipo (infraestructura de datos, diseño de producto, seguridad, inferencia, ingeniería de producto, marketing de crecimiento, legal); NO responde bien coordinación/traspaso/revisión/trazabilidad de forma explícita.
- **Quién lo usa:** Anthropic, sobre el uso interno de Claude Code en sus propios equipos. [Claude.com/blog, "How Anthropic teams use Claude Code", 24-jul-2025 (redirigido desde anthropic.com/news)](https://claude.com/blog/how-anthropic-teams-use-claude-code).
- **Patrones concretos citados:** Seguridad sigue "design doc → pseudocode → test-driven development → check-ins periódicos"; Marketing de crecimiento usa explícitamente "dos sub-agentes especializados" (única mención textual de topología multi-agente explícita en todo el post); el resto de equipos usa un agente por tarea con checkpoints humanos.
- **Resultado reportado:** cifras dispersas y no comparables entre sí — "3x más rápido" en debugging de seguridad, "80% de reducción" en tiempo de documentación en el equipo de inferencia, "20 minutos ahorrados" en un incidente de Kubernetes. Todas son autorreportadas, sin metodología de medición explicada.
- **Vacíos reconocidos por el propio análisis de esta sesión (no por Anthropic):** el post NO detalla estructuras formales de asignación de tareas, cómo se rastrea el estado/progreso entre traspasos, cómo se resuelven conflictos o salidas divergentes de agentes, tasas de error/rechazo/rework, ni procesos de revisión de seguridad/cumplimiento para salidas automatizadas.
- **Tipo de evidencia:** afirmación de un vendor sobre su propio producto/uso interno — según §7, **no basta por sí sola** como "contrastado".
- **Fuentes independientes:** 1.
- **Confianza:** media en que describe usos reales (detalle concreto por equipo, no genérico); baja en que sea representativo de "cómo coordinar" — el post evita sistemáticamente las preguntas 3, 5 y 6 de la sección 1.
- **Contrastado:** NO — vendor-claim único, sin cifras verificables ni corroboración externa.

### P9. Aislamiento de código con worktrees, pero SIN aislamiento de estado en ejecución (BD, puertos) — gap identificado por la propia comunidad

- **Preguntas que responde:** coordinación (qué falla al evitar choques cuando varios agentes corren en paralelo sobre el mismo repo).
- **Quién lo usa:** discusión práctica activa. [Reddit r/ChatGPTCoding, "If you run multiple AI agents on the same repo, how do you stop them stepping on each other?"](https://www.reddit.com/r/ChatGPTCoding/comments/1vx0hb6/if_you_run_multiple_ai_agents_on_the_same_repo/).
- **Problema descrito:** git worktrees resuelven la colisión de código, pero no la de estado en ejecución — dos agentes no pueden testear contra la misma base de datos ni la misma instancia corriendo. Soluciones caseras descritas por 3 comentaristas independientes: nombre de proyecto docker-compose y puertos derivados del nombre de la rama (`COMPOSE_PROJECT_NAME=$(basename $PWD)`, offset de puertos por worktree), clonado de la BD de desarrollo por worktree con numeración incremental, o aislamiento completo en VMs (descartado por sobrecoste de gestión frente al beneficio, según un comentarista).
- **Patrón adicional en el mismo hilo (jerarquía):** un comentarista (`philip_laureano`) describe un agente "command deck" que planifica, crea specs y despacha trabajo a agentes de implementación que leen instrucciones de un sistema de tickets y un "sistema de memoria compartida" que el command deck puede "espiar" — ejemplo concreto de topología jerárquica (1 planificador → N implementadores) fuera de los vendors ya cubiertos en F2.
- **Tipo de evidencia:** discusión de práctica de primera mano, sin cifras, múltiples autores coincidiendo en el diagnóstico del problema (worktrees ≠ aislamiento completo) aunque con soluciones distintas.
- **Fuentes independientes:** 1 hilo, ≥4 comentaristas independientes coincidiendo en el diagnóstico.
- **Confianza:** media — el diagnóstico (worktrees no aíslan estado de ejecución) es consistente y no contradicho en ningún sitio de la investigación; las soluciones son caseras y no estandarizadas.
- **Contrastado:** parcialmente — el problema sí (varias fuentes independientes coincidentes), la solución no (cada quien monta la suya).

### P10. Especificación versionada en el repo sustituye al tracker como lugar donde "vive" el trabajo — con un riesgo señalado explícitamente

- **Preguntas que responde:** coordinación (dónde vive el estado real del trabajo).
- **Quién lo usa:** equipo pequeño real (~10 devs), organizado en 3 "swimlanes" autoorganizadas, con ficheros de spec por fase del proyecto como fuente de verdad compartida en vez de registrar todo en Jira/Linear. [Reddit r/ClaudeAI, "Project management in the AI era (JIRA / Linear)"](https://www.reddit.com/r/ClaudeAI/comments/1vkqoaj/project_management_in_the_ai_era_jira_linear/).
- **Variantes reportadas por otros equipos en el mismo hilo:** un dev remoto solo usa Linear como tablero de comunicación con PM/QA (no como especificación); otro equipo (`pestkranker`) commitea las specs al repo y las revisa vía pull request de GitHub junto con el código, de modo que el traspaso de trabajo entre desarrolladores pasa por una reunión o un canal escrito (Slack, GitHub Issues) explícito.
- **Riesgo señalado explícitamente (por un comentarista, `SSShken`, sin responder si les pasa a ellos):** *"the tickets did not stop being useful, they stopped being the place the work is described. That now lives in a session nobody else can read and that disappears when it ends."* — el riesgo real no es la herramienta de tracking, sino que la descripción viva del trabajo quede atrapada en una sesión de agente no compartida y efímera si no se fuerza su volcado a un artefacto persistente (spec en el repo, ticket, o handoff escrito).
- **Tipo de evidencia:** discusión de práctica, varios equipos reales describiendo variantes del mismo problema, sin cifras.
- **Fuentes independientes:** 1 hilo con ≥4 equipos/personas distintas aportando su propia variante.
- **Confianza:** media.
- **Contrastado:** parcialmente — el patrón "spec en repo como fuente de verdad" tiene 2+ relatos independientes coincidentes (cumple §7); el riesgo señalado es una única voz sin verificación cuantitativa, se trata como hipótesis razonable, no como hallazgo firme.

### P11. Testing más voluminoso pero no necesariamente mejor cuando el autor es un agente (hallazgo académico cuantitativo)

- **Preguntas que responde:** revisión / calidad del traspaso agente→humano.
- **Fuente primaria:** [Milanese, Salzano, Spina et al., "Human-Agent versus Human Pull Requests: A Testing-Focused Characterization and Comparison", arXiv 2601.21194, 29-ene-2026](https://arxiv.org/abs/2601.21194), sobre el dataset AIDev (6.582 PRs humano-agente vs. 3.122 PRs humanas).
- **Cifras:** tasa de inclusión de tests comparable (42,9% HAPR vs. 40,0% HPR) pero **casi el doble de ratio test-código en líneas** en las PR de agentes; los agentes tienen mucha más probabilidad de AÑADIR tests nuevos que de modificar los existentes (OR=1,79) frente a humanos, que priorizan modificar tests ya existentes; diferencias en "test smells" (patrones de mala calidad en tests) estadísticamente significativas pero con tamaño de efecto insignificante — es decir, calidad comparable pese a la diferencia de volumen.
- **Tipo de evidencia:** estudio académico con minería de datos reales de GitHub.
- **Fuentes independientes:** 1 (pero es un estudio con datos cuantitativos propios sobre miles de PRs reales — cumple §7 por la vía alternativa).
- **Confianza:** alta en las cifras reportadas (metodología de minería de datos verificable), media en su generalización fuera del dataset AIDev.
- **Contrastado:** sí (vía alternativa de §7).

### P12. Revisión híbrida humano-agente: el agente escala, el humano da contexto — con coste de calidad si se adopta sin criterio

- **Fuente primaria:** [Zhong, Noei, Zou, Adams, "Human-AI Synergy in Agentic Code Review", arXiv 2603.15911, 16-mar-2026](https://arxiv.org/abs/2603.15911) — 278.790 conversaciones de revisión en 300 repos open-source reales.
- **Hallazgo:** los humanos aportan más feedback de "entendimiento, testing y transferencia de conocimiento" que los agentes; cuando se revisa código generado por IA, los humanos intercambian un 11,8% más de rondas que en código humano; las sugerencias de agentes de revisión se adoptan con mucha menos frecuencia que las de revisores humanos, y más de la mitad de las sugerencias de IA rechazadas eran incorrectas o resueltas de otra forma por el desarrollador; cuando SÍ se adoptan sugerencias de IA, producen aumentos significativamente mayores de complejidad y tamaño de código que las sugerencias humanas adoptadas.
- **Tipo de evidencia:** estudio académico con datos reales a gran escala.
- **Confianza:** alta (tamaño de muestra grande, metodología de minería de repos).
- **Contrastado:** sí (fuente primaria con datos cuantitativos propios, §7).

### P13. Estudio de prácticas de equipos que CONSTRUYEN agentes (no solo los usan): "los cuellos de botella se desplazan, no desaparecen"

- **Fuente primaria:** [Lyu, Williams, Shi et al., "How Do Practitioners Build SE Agents? Insights from a Mixed-Methods Study", arXiv 2607.10856, jul-2026](https://arxiv.org/abs/2607.10856) — 20 entrevistas semiestructuradas + encuesta a 80 desarrolladores.
- **Hallazgo central:** conforme baja el coste de implementación, emerge trabajo nuevo alrededor de la revisión de código y la evaluación de agentes; identifican un flujo de 7 etapas y 5 cambios de proceso mayores, entre ellos el desarrollo "guiado por evaluación" (definir cómo se va a evaluar antes de iterar) y — coincidiendo directamente con P10 — **"las especificaciones se convierten en artefactos de primera clase que los equipos testean y versionan junto al código"**. Identifican 6 retos críticos y 12 prácticas correspondientes; entre los retos: métricas de evaluación poco fiables, brechas de comprensión (la complejidad del código supera lo que el desarrollador entiende) y cambios de comportamiento inesperados por actualizaciones de modelo del lado del proveedor.
- **Tipo de evidencia:** estudio académico mixto (entrevistas + encuesta).
- **Confianza:** alta en la dirección del hallazgo (coincide independientemente con P10, sobre specs versionadas), media en las cifras exactas (no se extrajeron números de la encuesta en esta sesión).
- **Contrastado:** sí — coincide con P10 (Reddit) de forma totalmente independiente: 2 fuentes de tipo distinto (foro de práctica + estudio académico) llegando al mismo hallazgo sobre specs como artefacto central.

### P14. Aceptación real de PRs de agente en open source con un solo vendor (Claude Code) — cifra de referencia para el "traspaso" agente→humano

- **Fuente primaria:** [Watanabe, Li, Kashiwa et al., "On the Use of Agentic Coding: An Empirical Study of Pull Requests on GitHub", arXiv 2509.14745, sep-2025, revisado feb-2026](https://arxiv.org/abs/2509.14745) — 567 PRs generadas con Claude Code en 157 proyectos open source diversos.
- **Cifras:** 83,8% de las PRs se aceptan y mergean; de esas, 54,9% se integran SIN modificación humana alguna; el 45,1% restante requiere revisión/cambio humano, sobre todo en corrección de bugs, documentación y adecuación a estándares específicos del proyecto.
- **Tipo de evidencia:** estudio académico con datos reales de GitHub, específico de un solo agente/vendor (a diferencia del dataset AIDev multi-vendor ya citado en F2).
- **Confianza:** alta.
- **Contrastado:** sí (§7, fuente primaria con datos cuantitativos).

### P15 (reutilizado de F2, re-etiquetado aquí solo por completitud de las 6 preguntas — no se repite evidencia). Issue→PR asíncrono vía agente cloud del vendor de Git hosting

Ver [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] §3.1, §3.3, §3.5: patrón dominante por adopción (Copilot coding agent, Codex Cloud, Jules, GitLab Duo), evidencia dotnet/runtime (67,9% merge rate, 878 PRs/10 meses), confianza alta ya establecida allí. Se reutiliza aquí solo para no dejar un hueco en el catálogo, sin volver a verificar.

---

## 3. Respuesta provisional a las 6 preguntas, según la práctica

### 1. Roles

No hay un reparto único. La evidencia muestra un espectro:

- **Todo en un agente + un humano** (P1): frecuente entre desarrolladores independientes o equipos muy pequeños; los propios practicantes desaconsejan escalar a "enjambres" sin necesidad real.
- **Planificador (modelo fuerte) / implementador (modelo barato o distinto)**, ambos IA (P2): patrón activamente discutido y con solución de "plan como contrato" convergente entre varias fuentes independientes — es el más parecido a la hipótesis original del usuario, y el más contrastado de los patrones "modelo dual".
- **Varios roles de agente encadenados** (diseñador → implementador → revisor → integrador/"lander"), con el humano solo en excepciones (P3): descrito, pero solo por una fuente sin verificación externa — no contrastado.
- **Humano retiene arquitectura, revisión final e integración; agente hace implementación/tests** (P7, P8): patrón "human-led, agent-assisted" reportado por dos empresas reales (Eventual, y fragmentariamente Anthropic), consistente entre sí pero cada una es una sola fuente.
- **Jerarquía explícita con un agente "orquestador"** (P9, patrón "command deck"): existe en la práctica individual, sin adopción medible.

**Lectura combinada:** no hay un reparto de roles "correcto" contrastado como superior a los demás; lo que sí está contrastado (≥2 fuentes independientes en P2, P5, P10/P13, P11, P12, P14) es que, sea cual sea el reparto, **el rol de revisión humana tiende a erosionarse o desplazarse** (a "revisar el plan" en vez del código — P4 — o a no revisar en absoluto — P5) más que a mantenerse igual que en un flujo sin IA.

### 2. Traspaso

El patrón más contrastado no es una herramienta sino una propiedad del artefacto: **el traspaso funciona cuando es un contrato verificable (interfaces exactas, criterios de aceptación explícitos, casos límite) y falla cuando es prosa de intención** ("plan drift", P2). Esto coincide con el hallazgo ya establecido en F2 sobre dotnet/runtime ("preparation matters more than the model"). Además, dos fuentes independientes de tipo distinto (P10, foro; P13, estudio académico) coinciden en que **la especificación tiende a convertirse en un artefacto versionado en el propio repo**, no en un documento aparte — desplazando parcialmente al issue/ticket tradicional como "lugar donde vive la descripción del trabajo", aunque el ticket se mantiene para visibilidad de progreso ante personas no técnicas (P10).

### 3. Coordinación

Git (commits, worktrees, ramas) es el mecanismo de coordinación de facto en todos los patrones observados, no solo en los vendors ya cubiertos en F2. Dos hallazgos propios de esta fase, no vistos en F2:

- **El coste de la propia coordinación puede ser mayor que el trabajo útil**: en P6, el 54,5% de los commits fueron overhead de coordinación (locks, reclamos, liberaciones) — la evidencia en contra más fuerte encontrada en esta fase contra la idea de que "más agentes en paralelo = más rendimiento".
- **Los worktrees aíslan código, no estado en ejecución** (P9): bases de datos, puertos, instancias corriendo siguen siendo un punto de colisión no resuelto de forma estándar; cada equipo/persona improvisa su propia solución (nombres de proyecto docker-compose derivados de la rama, clonado de BD, o VMs completas si se puede pagar el coste operativo).

### 4. Topología

La práctica reportada contradice la narrativa de "enjambres masivos" como estado del arte:

- Los propios practicantes activos en foros técnicos convergen, de forma independiente, en **1-2 agentes concurrentes como límite práctico sostenible** (P1), citando fatiga de atención y coste como razones, no solo capacidad técnica.
- Cuando SÍ se prueban enjambres más grandes (P6: 6 agentes trabajadores + coalescer), el propio autor documenta que más de la mitad del esfuerzo de commits fue overhead de coordinación y que el rol de "reconciliación" (coalescer) fracasó en completarse.
- La postura de Cognition/Devin contra el paralelismo de subagentes sin trace compartido (ya en F2) se ve reforzada aquí por evidencia independiente de práctica de terceros, no solo por el vendor-claim original.
- La topología jerárquica (1 orquestador → N implementadores) existe en la práctica individual (P9) pero sin evidencia de escala más allá de un solo usuario.

### 5. Revisión

Es la pregunta con más evidencia contrastada de toda la fase:

- **La mayoría de PRs de agente no reciben revisión humana en absoluto**, y cuando la reciben, tiende a ser agente-revisa-a-agente ("automation-mediated") en vez de evaluación humana independiente (P5, arXiv 2605.02273) — coincide con anécdotas de Reddit del mismo periodo.
- Cuando SÍ hay revisión humana de código generado por IA, requiere **más rondas** que revisar código humano (+11,8%, P12) y las sugerencias de agentes revisores se aceptan mucho menos que las humanas.
- Una práctica real (no hipotética) para escapar de este cuello de botella es **desplazar la revisión al nivel del plan** en vez del código línea a línea (P4), con una herramienta open source real (aunque pequeña, 31★) que lo materializa.
- Con un solo vendor bien acotado (Claude Code en open source), el 54,9% de PRs entran sin ningún cambio humano y el 45,1% restante sí necesita intervención, sobre todo en bugs y estándares del proyecto (P14) — una cifra de referencia útil y contrastada para calibrar cuánta "revisión humana por PR" es realista esperar.

### 6. Trazabilidad

Esta fase no añade hallazgos nuevos más allá de lo ya establecido en F2 (identidad de bot vía GitHub App, trailers `Co-authored-by`/`Co-Authored-By`, enlace a sesión completa). Ningún patrón de práctica encontrado aquí (P1-P14) describe un mecanismo de trazabilidad propio distinto al ya documentado; varios (P3, P6) mencionan documentos de estado (`PROGRESS.md`, notas) que funcionan como bitácora pero no como trazabilidad verificable por un tercero. Se mantiene como hueco no resuelto por la práctica — ver sección 4.

---

## 4. Lo que la práctica NO resuelve bien / huecos

- **Aislamiento de estado en ejecución entre agentes paralelos** (bases de datos, puertos, servicios corriendo): ningún patrón encontrado lo resuelve de forma estandarizada; cada equipo improvisa (P9).
- **Coste real de la coordinación misma**: solo un patrón (P6) lo midió explícitamente y encontró que superaba el 50% del esfuerzo — sugiere que es un coste sistemáticamente subestimado en el resto de relatos, que no lo miden.
- **Trazabilidad más allá de lo ya cubierto en F2**: ningún patrón de esta fase aporta un mecanismo nuevo; sigue dependiendo por completo de confiar en el vendor (GitHub, Anthropic) como intermediario, sin firma criptográfica verificable por terceros (hueco ya señalado en F2, confirmado que sigue abierto).
- **Cifras de coste total (tokens + tiempo humano) por patrón**: ninguna fuente de esta fase da una cifra de coste total comparable entre patrones (P3 da PRs/día pero no coste; P6 da consumo de cuota pero no coste en $; P1 da % de cuota Max 5x pero sin traducir a horas humanas).
- **Qué pasa cuando el humano revisor deja de revisar por completo** (P5) a escala: hay anécdotas y un hallazgo poblacional de que ocurre, pero ningún estudio mide sus consecuencias a medio plazo (bugs en producción, deuda técnica) de forma cuantitativa — es la brecha más señalada cualitativamente (quejas de burnout, "AI slop") pero menos medida cuantitativamente.
- **Comparación directa y controlada entre patrones** (p. ej. P2 vs. P7 vs. P15 en la misma tarea): no existe en ninguna fuente encontrada; toda la evidencia es de un solo patrón a la vez, nunca comparativa.

---

## 5. Fuentes

### Usadas y citadas (con URL en la línea correspondiente de las secciones 2-4)

- Reddit (r/ExperiencedDevs, r/ClaudeAI, r/ChatGPTCoding), vía Chrome real (CDP), 6 hilos completos leídos.
- Hacker News (`hn.algolia.com` API): 1 hilo de comentario detallado citado literalmente (404softwarelabs), 1 comentario de `dylanratcliffe`.
- Blog "Building SQLite with a small swarm" (kiankyars.github.io).
- Blog de empresa Eventual/daft.ai ("How We Use AI Coding Agents").
- Blog de Anthropic/Claude.com ("How Anthropic teams use Claude Code").
- Repositorio GitHub `until-dev/plugins`, verificado en vivo vía API de GitHub.
- 6 papers de arXiv verificados en `arxiv.org/abs/<id>`: 2601.21194, 2509.14745, 2603.15911, 2605.02273, 2607.10856, 2607.21997.

### Consultadas sin aportar nada útil / descartadas

- Búsquedas HN: "cheap model implement expensive model plan", "Copilot internal engineering workflow report", "AI pair programming roles", "until-dev plan review", "Sourcegraph Amp internal dogfood", "how we dogfood our own coding agent", "spec kit GitHub team practice", "Shopify AI agents engineering", "Replit agent internal workflow", "Canva engineering AI coding agents", "we let AI write all our code experience report", "multi-agent coding failed" (solo resultados tangenciales) — ver detalle en sección 1.
- arXiv: consulta genérica "multi-agent LLM software engineering workflow roles" devolvió mayoritariamente marcos conceptuales/propuestas de arquitectura (MetaGPT, ChatCollab, NOMAD, GUISpector, Shapley-Coop, papers sobre "personalidad y emoción en equipos multi-agente", "estereotipos lingüísticos") descartados por no ser evidencia de práctica real, no por ser de mala calidad.
- Ask HN "May be a basic question, but how can I use AI well?" y "Ask HN: Multi-agent workflows in production; where people using 1000s of agents?": revisados, sin contenido sustancial adicional al ya cubierto.
- "Show HN: Cq – Stack Overflow for AI coding agents" (225pts/103 comentarios): no se profundizó por límite de tiempo — es candidato para ampliar en una fase posterior si hiciera falta más evidencia sobre memoria compartida entre agentes (no cubierto por las 6 preguntas de raíz).

---

## 6. Bloqueos

- No se buscó en r/ClaudeCode ni r/LocalLLaMA (mencionados como sugeridos en el encargo) por límite de tiempo dentro de esta sesión única sin subagentes — hueco declarado, no crítico porque r/ClaudeAI y r/ChatGPTCoding ya aportaron señal suficiente para contrastar los patrones principales.
- La API de arXiv sigue bloqueada (406) desde esta máquina — igual que en F2, se usó la página de búsqueda web (`arxiv.org/search/?query=...`), que funciona.
- WebSearch no se usó en absoluto en esta fase (cupo ya reportado como agotado en el encargo); toda la investigación se hizo con WebFetch, `curl` a APIs públicas (HN Algolia, GitHub API), y el navegador Chrome real para Reddit.
- Ningún bloqueo de red o de herramienta impidió completar las 6 preguntas con al menos evidencia "media" o mejor; los huecos reales son de contenido (sección 4), no de acceso.

## Enlaces

- [[circuito-tareas-definicion]] — encargo y criterios de esta fase
- [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] — evidencia reutilizada (dotnet/runtime, AIDev, Cognition, trazabilidad)
- [[desarrollo-agentes-investigacion]] — síntesis general
- [[sistema-desarrollo-con-agentes]] — proyecto

## Enlaces

- [[flujo-agentes-informe]] — síntesis de la investigación
- [[circuito-tareas-definicion]] — definición de la fase
- [[_index]]
