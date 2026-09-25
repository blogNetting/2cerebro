---
title: Flujo con agentes, fase A2: práctica a gran escala
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, practicas, stripe, openai, ramp, investigacion]
zona: tecnico
---

Cómo trabajan con agentes OpenAI, Stripe, Ramp, Spotify y Anthropic, según sus propios informes, más la especificación de OpenAI Symphony.

Informe de fase; lo ha revisado el orquestador. Síntesis en [[flujo-agentes-informe]]. Las correcciones del orquestador están en esa síntesis.

Sesión única, sin subagentes (por encargo explícito). Fecha: 2026-09-25. Sigue la definición de [[circuito-tareas-definicion]] (6 preguntas, criterios C1–C13, «contrastado» en su §7). No repite los patrones P1–P15 de `A-practicas-reales.md`; los referencia y los matiza donde corresponde.

**Motivo de esta fase:** el orquestador detectó que la fase A no leyó los informes de primera mano de mayor peso sobre cómo organizan a escala el desarrollo con agentes OpenAI, Stripe y Anthropic, ni el spec de orquestación que OpenAI publicó (Symphony). A2 los lee en fuente primaria completa y añade cualquier otro informe organizativo comparable no cubierto.

---

## 1. Método y consultas

**Orden seguido:** lectura directa de las 4 fuentes encargadas (WebFetch; navegador Chrome real vía CDP —*Chrome DevTools Protocol*— solo para la página de OpenAI, que devuelve 403 a WebFetch/curl) → HN Algolia (`hn.algolia.com/api/v1/search` y `/items/<id>` para árboles de comentarios) para cada fuente y para hallar informes adicionales → `gh api` para el repo `openai/symphony` (README, `SPEC.md`, metadatos del repo). WebSearch no se usó (cupo agotado, según el encargo).

### Fuentes encargadas — todas con hit directo en HN Algolia

- "harness engineering codex" → story 48416264, 297 puntos, 206 comentarios (la mayor discusión de toda la investigación A+A2).
- "minions stripe coding agents" → parte 2: story 47086557 (131 pts/61 com.); parte 1: story 47110495 (93 pts/81 com.).
- "symphony openai" → story 47252045 (25 pts/6 com.); además 47257966 (4 pts/0) y 47278403 (3 pts/1) — mismo enlace, sin discusión adicional relevante.
- "multi-agent research system anthropic" → story 44272278 (35 pts/**0 comentarios**); nota aparte de Simon Willison indexada como story 44280445 (12 pts/0 comentarios).
- "effective harnesses long running agents" → story 46081704 (125 pts/37 com.).

### Búsquedas adicionales en HN Algolia (para hallar otros informes de peso comparable)

Con resultado útil: "background agents at" (→ Spotify), "Ramp background agents" (→ post oficial de Ramp vía un blog de terceros que lo cita), "why we built our background agent ramp" (→ story 46589842, 122 pts/27 com.), "spotify background coding agent" (→ stories 45840407, 45943614, 45846938).

Sin resultado útil o solo ruido (vacías o tangenciales, se listan todas): "how we use coding agents at" (solo Show HN de herramientas, ninguna práctica organizativa), "agents wrote" (anécdotas puntuales, ningún informe a escala), "unattended coding agents" (productos, no prácticas — salvo el hallazgo de Ramp vía búsqueda cruzada), "our agent workflow" (solo Show HN de productos), "PRs per week agents" (0 resultados de práctica), "engineering blog coding agents" (resultados de herramientas/ensayos individuales, no informes de organización), "Ramp engineering coding agent" (0 hits), "we tried multi agent coding and stopped" (0 hits), "AI agents did not work for us engineering" (0 hits relevantes, solo ruido de Show HN no relacionado), "our experience with autonomous coding agents failure" (0 hits), "coding agent fleet production incident" (0 hits), "abandoned AI agent coding workflow" (0 hits), "GitLab Duo agent platform team" (0 hits relevantes), "Shopify AI agent engineering blog" (0 hits relevantes), "Sourcegraph Amp production usage" (0 hits).

**Nota sobre informes negativos/de abandono:** pese a búsquedas específicas con varias formulaciones, no se encontró ningún informe de primera mano de una organización que documente haber *abandonado* un flujo de agentes a escala con datos propios. Lo que sí se encontró es evidencia en contra *dentro* de los informes positivos (ver P22, P23 y cada sección de fuente) y en los comentarios de HN. Se marca como hueco, no como ausencia de esfuerzo de búsqueda — ver §6.

### Verificación adicional

- `gh api repos/openai/symphony` (metadatos), `repos/openai/symphony/readme`, `repos/openai/symphony/contents`, y `raw.githubusercontent.com/openai/symphony/main/SPEC.md` (spec completo, 2300 líneas, leído por secciones).
- Intento de verificar si el crecimiento de estrellas de `openai/symphony` es orgánico, vía `gh api repos/openai/symphony/stargazers` con cabecera `Accept: application/vnd.github.star+json` para obtener marcas de tiempo de starring: **la API devolvió 404 en ambos intentos, con y sin la cabecera** — bloqueo técnico, no se pudo verificar. Se marca «sin verificar» en la sección correspondiente (ver Fuente 6).

---

## 2. Fuentes, extracción y citas literales

### Fuente 1 — OpenAI, «Harness engineering: Leveraging Codex in an agent-first world» (Ryan Lopopolo, 11-feb-2026)

Leída completa en español vía navegador real (CDP) — la propia página redirige a `openai.com/es-ES/...`, WebFetch da 403. [https://openai.com/index/harness-engineering/](https://openai.com/index/harness-engineering/)

- **Roles:** «Las personas dirigen. Los agentes ejecutan.» Ningún humano escribe código a mano durante 5 meses; un equipo de 3 ingenieros (crece a 7) dirige a Codex por *prompt*. Codex escribe también CI, tests, documentación, observabilidad y herramientas internas.
- **Traspaso:** el *prompt* de una tarea. El repositorio mismo es la base de conocimiento: `AGENTS.md` (~100 líneas) actúa como «mapa», no manual, apuntando a `docs/` (design-docs, exec-plans/active, exec-plans/completed, product-specs, `QUALITY_SCORE.md`, `RELIABILITY.md`, `SECURITY.md`). Cita literal: «tratamos AGENTS.md como un índice» y «los planes se consideran artefactos de primera categoría» — coincide con P10/P13 de fase A.
- **Estado del trabajo:** vive enteramente en el repo (git), sin tracker externo mencionado en ningún momento del artículo. Un agente recurrente de «doc-gardening» detecta documentación desactualizada y abre PRs de corrección; «la mayoría de ellas se pueden revisar en menos de un minuto y fusionarse automáticamente».
- **Concurrencia/aislamiento:** cada *git worktree* es arrancable de forma independiente, con su propia pila de observabilidad efímera (logs/métricas/trazas vía LogQL/PromQL) y su propio Chrome DevTools Protocol para probar la UI. Ejecuciones individuales de Codex «trabajan en una sola tarea durante más de seis horas» sin supervisión, incluso de noche.
- **Revisión:** «pedimos a Codex que revise sus propios cambios localmente, solicite revisiones adicionales específicas de agentes tanto a nivel local como en la nube, responda a cualquier comentario... y repita en un bucle hasta que todos los revisores estén satisfechos (en la práctica, esto es un bucle *Ralph Wiggum*)» — término informal tomado de un blog externo (ghuntley.com) para «reintentar hasta que pase». Cita literal clave: «El equipo humano puede revisar las pull requests, aunque no está obligado a hacerlo. Con el tiempo, hemos dirigido casi todo el esfuerzo de revisión para que se gestione de agente a agente.» Esto es **política deliberada**, no degradación pasiva — matiza P5 de fase A (ver P17).
- **Trazabilidad:** no se detalla mecanismo más allá de git; hueco explícito, igual que en fase A.
- **Números:** ~1.000.000 líneas de código en 5 meses; ~1.500 PRs abiertas y fusionadas; equipo de 3→7 ingenieros; **3,5 PR por ingeniero al día**, con rendimiento *creciente* al crecer el equipo (contraintuitivo); «aproximadamente una décima parte del tiempo» que a mano; antes dedicaban el 20% de la semana (viernes) a limpiar «basura de la IA», ahora automatizado.
- **Fallos/límites declarados por el propio autor:** el enfoque «gran AGENTS.md» falló por 4 motivos explícitos (contexto escaso, «demasiada orientación se convierte en desorientación», se degrada al instante, difícil de verificar); entropía y replicación de patrones subóptimos existentes; caveat explícito de no generalización: «este comportamiento depende mucho de la estructura y las herramientas específicas de este repositorio, y no se debe asumir que vaya a generalizarse... al menos, por ahora».
- **Tipo de evidencia:** informe de práctica de primera mano de un vendor (OpenAI) sobre su propio producto interno, con datos cuantitativos propios. Según §7, basta por la vía alternativa (fuente primaria + cifras propias).
- **Corroboración en HN (206 comentarios, la mayor de toda la investigación):** mayoritariamente escéptica, no confirmatoria. `Sarkie`: «I would never dare put that in production»; `rfw300` acusa al propio post de estar escrito por LLM con clichés, socavando el argumento de calidad; `murat124` compara la revisión humana de PRs masivas con obreros de una fábrica de vapeadores probando en 5 segundos sin inspeccionar de verdad; `apical_dendrite` plantea que esto invalida buena parte del criterio senior de ingeniería; `daxfohl` (en el hilo de «effective harnesses», mismo tema): «a dedicated testing/QA agent sounds nice but doesn't work... the more you diverge from the original dev agent's approach the less chance there is that the dev agent will get to where you want» — evidencia en contra directa de añadir un rol de agente revisor separado. Nadie en el hilo confirma cifras similares en su propia organización.
- **Confianza:** alta en que las cifras reportadas son internamente consistentes y específicas; baja en que el patrón generalice (el propio autor lo advierte) y en que la «revisión agente-a-agente» sea segura sin evidencia de tasa de defectos post-*merge* (no publicada).
- **Contrastado:** sí, por la vía alternativa de §7 (fuente primaria + datos cuantitativos propios), pero **sin corroboración independiente** de un tercero con cifras comparables — la discusión de HN es la más grande de la investigación y aun así no aporta ese segundo dato duro.

### Fuente 2 y 3 — Stripe, «Minions: Stripe's one-shot, end-to-end coding agents» (parte 1 y 2)

[Parte 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents) · [Parte 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)

- **Roles:** ingenieros invocan «minions» (agente de código, *fork* de `goose` de Block) desde Slack, CLI o web; el minion implementa de forma autónoma; un humano revisa la PR obligatoriamente. Cita literal: «Over a thousand pull requests merged each week at Stripe are completely minion-produced», y «while they're human-reviewed, they contain no human-written code».
- **Traspaso:** el hilo de Slack es la entrada más frecuente («engineers can kick off a minion directly from the thread discussing a change, and it'll be able to access the entire thread and any links»); también tickets automáticos de un sistema de detección de tests inestables (*flaky tests*), con botón de invocación de minion. Salida: PR con plantilla estándar de Stripe.
- **Estado del trabajo:** sin cola formal — invocación bajo demanda, «frequently see engineers spinning up multiple minions in parallel». Contexto centralizado vía **Toolshed**, un servidor MCP (*Model Context Protocol*, protocolo abierto para exponer herramientas/datos a un agente) interno con «more than 400 MCP tools» (part. 2: ~500) que da acceso a documentación, tickets, estado de builds y búsqueda de código (Sourcegraph).
- **Concurrencia/aislamiento:** *devboxes* — instancias EC2 de AWS pre-cargadas con el código y servicios de Stripe, arrancables en 10 segundos, estandarizadas («cattle, not pets»), aisladas de producción e internet, corriendo en entorno de QA sin datos reales de usuario. Varios minions en paralelo por ingeniero, normalmente un devbox por tarea.
- **Revisión y rehacer — el bucle más acotado numéricamente de toda la investigación:** lint local determinista (<5s) → push a CI → ejecución selectiva de un subconjunto de «more than three million» tests → autofixes automáticos donde existen → si el fallo no tiene autofix, «we send it back to the minion to try and fix» → **«We only have at most two rounds of CI. If tests fail after an initial push, we prompt the minion to fix failing tests and push a second time, but are then done»** → tras la segunda ronda, «we send the branch back to its human operator for manual scrutiny». Razón explícita: «There's a balancing act between speed and completeness here; CI runs cost tokens, compute, and time».
- **Trazabilidad:** UI web muestra «the decisions and actions the minion took», y una lista de devboxes activos por ingeniero con sus minions — sin más detalle de atribución (no se describe firma criptográfica ni bot de commit específico).
- **Números:** parte 1 (~feb-2026): >1.000 PR/semana 100% producidas por minion; >400 herramientas MCP; >3M tests. Parte 2: **>1.300 PR/semana (+30% sobre la parte 1)**; ~500 herramientas MCP; 10s de arranque de devbox; máximo 1-2 rondas de iteración antes de escalar a humano.
- **Tipo de evidencia:** vendor con datos cuantitativos propios (§7, vía alternativa).
- **Corroboración/evidencia en contra en HN (61+81 comentarios):** fuerte escepticismo sobre si la revisión humana es real: `rco8786`/`3rodents` preguntan si hay ejemplos concretos o si la revisión es sello de goma; `jimmydoe`: «Dark secret of dark factory is high quality human input... otherwise human ends up multiple shot it, and read thru the transcript to tune the input»; `iepathos`: «"1000 PRs/week" with no breakdown of complexity or value is a vanity metric... you've just created 1000 code reviews/week to waste human time rubber-stamping»; `alembic_fumes`: el ingeniero senior queda «demoted from a helmsman into a human breakwater» — coincide literalmente con el hallazgo de *burnout* de P5 en fase A. Ningún comentarista se identifica como empleado de Stripe corroborando desde dentro (a diferencia de Ramp, fuente 8).
- **Confianza:** alta en las cifras de volumen (concretas, con evolución parte1→parte2); baja-media en si el bucle de revisión humana es sustantivo (nadie de Stripe ni de HN lo confirma con datos de tasa de defectos post-*merge*).
- **Contrastado:** sí, vía alternativa de §7 (2 informes propios con cifras consistentes entre sí en el tiempo), pero sin corroboración externa independiente de un tercero con acceso real.

### Fuente 4 — Anthropic, «Effective harnesses for long-running agents»

[https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)

- **Qué es un «harness»** (arnés — la infraestructura y reglas que rodean a un agente: *prompts* de sistema, herramientas, bucles de verificación): descrito aquí como el marco que permite a un agente trabajar a través de múltiples ventanas de contexto (sesiones).
- **Roles/etapas:** SOLO DOS, secuenciales, ocupadas por el mismo agente con *prompts* iniciales distintos: un **agente inicializador** (primera sesión) monta el entorno; un **agente de codificación** (todas las sesiones siguientes) avanza incrementalmente. Cita literal: son «agentes separados en este contexto solo porque tienen *prompts* de usuario inicial distintos. El *system prompt*, el conjunto de herramientas y el arnés general del agente eran por lo demás idénticos.»
- **Traspaso/persistencia de estado:** `claude-progress.txt` (registro de progreso en texto libre), `feature_list.json` (lista estructurada de funciones con estado pasa/no pasa) y el historial de git con mensajes de commit descriptivos. Cita literal: «El hallazgo clave aquí fue encontrar una forma de que los agentes entiendan rápidamente el estado del trabajo al empezar con una ventana de contexto nueva, lo cual se logra con el archivo `claude-progress.txt` junto con el historial de git.»
- **Concurrencia: NINGUNA.** «Solo se ejecuta un agente por sesión» — refuerza directamente P1 de fase A (1-2 agentes concurrentes como límite práctico), esta vez desde un vendor, no desde foros.
- **Revisión:** autoverificación exclusivamente — sin humano ni otro agente en el bucle de este caso concreto (construcción de un clon de claude.ai como caso de estudio, no descrito como producción real). Prueba baseline extremo-a-extremo antes de nuevas funciones; usa automatización de navegador (Puppeteer MCP) para probar como lo haría un humano; «Solo marca funciones como "pasando" tras pruebas cuidadosas».
- **Trazabilidad:** git + `claude-progress.txt` + `feature_list.json`, mismo hueco que en el resto de fuentes (sin atribución verificable por terceros).
- **Números:** >200 funciones en la lista; sin cifras de tiempo, coste o PRs — es una guía metodológica, no un informe de práctica organizativa a escala; **peso evidencial menor** que las demás fuentes de A2 para el objetivo concreto de esta fase (evidencia de organización real a escala), aunque relevante para C1/C3/C8.
- **Tipo de evidencia:** documentación/guía técnica de vendor, caso de estudio interno, sin cifras de escala real. No cumple §7 (ni 2 fuentes independientes ni datos cuantitativos de uso en producción).
- **Corroboración en HN (37 comentarios):** `adidoit` señala una tensión de fondo: «el estado del arte en la construcción de arneses para agentes de larga duración es... "usar instrucciones enfáticas"» — crítica al hecho de que el control real depende de pedir con fuerza, no de garantías deterministas. `daxfohl` (citado también en Fuente 1) descarta explícitamente el patrón «agente de QA dedicado» por experiencia práctica.
- **Confianza:** media — mecanismos descritos con detalle y coherentes con otras fuentes, pero sin cifras de escala ni corroboración externa.
- **Contrastado:** NO por sí sola (sin cifras de uso a escala, un solo vendor); refuerza (no contradice) P1.

### Fuente 5 — Anthropic, «How we built our multi-agent research system» + notas de Simon Willison

[https://www.anthropic.com/engineering/built-multi-agent-research-system](https://www.anthropic.com/engineering/built-multi-agent-research-system) · [https://simonwillison.net/2025/Jun/14/multi-agent-research-system/](https://simonwillison.net/2025/Jun/14/multi-agent-research-system/)

Es sobre el sistema de *investigación* de Claude (no desarrollo de software), pero es la referencia de arquitectura orquestador-trabajadores más citada del sector y responde directamente a C5 (topología) y, sobre todo, **C9 (coste)** — el hueco de cifras de coste que fase A dejó explícitamente abierto.

- **Roles:** agente líder = orquestador, descompone la consulta y genera subagentes; subagentes = trabajadores, cada uno hace ≥3 llamadas a herramientas en paralelo y devuelve una lista de hallazgos; un agente de citas separado añade referencias al final.
- **Traspaso:** cada subagente recibe «un objetivo, un formato de salida, orientación sobre qué herramientas/fuentes usar y límites de tarea claros» — regla explícita de escalado: 1 agente / 3–10 llamadas para hechos simples; 2–4 subagentes / 10–15 llamadas cada uno para comparaciones directas; **más de 10 subagentes** para investigación compleja.
- **Concurrencia:** el líder «genera 3-5 subagentes en paralelo, no en serie»; en investigación compleja, más de 10.
- **Coste — el dato nuevo más importante de toda la fase A2:** «multi-agent systems use about 15× more tokens than chats»; «Agents typically use about 4× more tokens than chat interactions»; y, crucial para explicar el resultado, «token usage by itself explains 80% of the variance» en sus evaluaciones internas de agentes de navegación. Ganancia de rendimiento reportada: el sistema multiagente (Opus 4 líder + Sonnet 4 trabajadores) superó al agente único Opus 4 en un **90,2%** en su evaluación interna de investigación.
- **Fallos reportados en los primeros prototipos:** agentes que generaban 50 subagentes para consultas simples; búsquedas interminables de fuentes inexistentes; mala delegación → «los subagentes duplican trabajo, dejan huecos o no encuentran la información necesaria»; sesgo hacia «contenido optimizado para SEO en vez de fuentes autorizadas pero menos posicionadas» — detectado por **pruebas humanas, no por las evaluaciones automáticas** («People testing agents find edge cases that evals miss»).
- **Trazabilidad/observabilidad:** «full production tracing» de patrones de decisión y estructura de interacción **sin monitorizar el contenido de conversaciones individuales, para proteger la privacidad del usuario» — mecanismo de observabilidad distinto a todo lo visto en fase A (basado en trazas de comportamiento, no en git/atribución de autoría); también «simulaciones» en su Consola reproduciendo los *prompts* y herramientas exactos del sistema para observar al agente paso a paso.
- **Tipo de evidencia:** vendor con datos cuantitativos internos propios (§7, vía alternativa).
- **Corroboración:** la propia entrada de HN sobre el post de Anthropic tiene **0 comentarios**; toda la corroboración disponible viene de la nota independiente de Simon Willison (desarrollador y comentarista técnico reconocido, sin relación comercial con Anthropic), quien la valida sin datos propios adicionales: «OK, I'm sold on multi-agent LLM systems now» (revirtiendo su escepticismo previo), y añade dos cifras del mismo post que resalta como las más prácticas que ha visto sobre diseño de sistemas multiagente: paralelización «cut research time by up to 90% for complex queries», y mejorar las descripciones de herramientas produjo «a 40% decrease in task completion time». Es un **endoso técnico independiente, no una réplica con datos propios**.
- **Confianza:** alta en las cifras internas (multiplicador de tokens, 90,2%, 80% de varianza) por venir de una evaluación interna descrita con metodología; sin segunda organización que reporte cifras de coste comparables — el hueco de C9 se llena parcialmente, no del todo.
- **Contrastado:** parcialmente — 1 fuente primaria con datos propios + 1 análisis técnico independiente coincidente (no vendor), pero sin una segunda organización repitiendo cifras de coste similares en desarrollo de software (este caso es investigación, no código).

### Fuente 6 — OpenAI Symphony (repo, README, `SPEC.md`)

[https://github.com/openai/symphony](https://github.com/openai/symphony)

No es un informe de práctica sino una **especificación formal de orquestación** publicada por OpenAI como «el siguiente paso» tras el harness engineering de la Fuente 1: «Symphony works best in codebases that have adopted harness engineering... moving from managing coding agents to managing work that needs to get done.» Se incluye por ser la respuesta más completa y formalizada que apareció en A o A2 a C3/C4/C5/C8, aunque como evidencia de *práctica* es débil (ver más abajo).

- **Roles:** no nombra roles humanos explícitos; humanos configuran `WORKFLOW.md` (fichero versionado en el repo, con *front matter* YAML + plantilla de *prompt* — repite el patrón P10/P13 de fase A: la especificación como artefacto versionado) y un *issue tracker* externo (Linear, en el vídeo de demo) contiene las tareas. Symphony sondea el tracker y despacha un agente de código (Codex) por *issue* en un espacio de trabajo aislado; una ejecución exitosa puede terminar en un estado de traspaso definido por el flujo (p. ej. `Human Review`), **no necesariamente en `Done`** — formaliza exactamente el patrón P4 de fase A («revisión desplazada»).
- **Traspaso:** objeto `Issue` normalizado (id, identificador legible, título, descripción, prioridad, estado, etiquetas, bloqueadores, bandera `dispatchable`) obtenido del tracker, renderizado en un *prompt* vía la plantilla de `WORKFLOW.md`.
- **Dónde vive el estado — la respuesta más precisa encontrada en A+A2 a C3:** el estado de tarea vive en el tracker externo (fuente de verdad); el estado de *orquestación* (qué está reclamado, corriendo, en reintento) vive **solo en memoria del proceso orquestador**, deliberadamente sin base de datos persistente. Cita literal: «Current design is intentionally in-memory for scheduler state... It does not mean retry timers, running sessions, or live worker state survive process restart» y, tras un reinicio, «No retry timers are restored from prior process memory. No running sessions are assumed recoverable.» — **es un punto único de fallo documentado por el propio vendor**, no señalado en ninguna fuente de fase A.
- **Asignación y concurrencia:** límite global (`max_concurrent_agents`) y límites opcionales **por estado**; orden de despacho por prioridad, luego más antigua primero, luego id como desempate; mecanismo de `claim` (reserva) explícito que evita despacho duplicado — es la implementación más formal encontrada del vocabulario *claim* que define C4 en `circuito-tareas-definicion.md`, aunque **no es un *lease* con caducidad independiente**: si el proceso orquestador muere, el reclamo no expira por sí solo, se pierde junto con todo el estado (ver punto anterior).
- **Aislamiento:** un directorio de espacio de trabajo por *issue* (ruta determinista derivada del identificador saneado); gancho de configuración/limpieza por espacio de trabajo. **No se aísla estado en ejecución** (bases de datos, puertos) — el propio spec no lo aborda, confirmando que sigue siendo un hueco incluso en la especificación de orquestación más formal disponible (matiza P9 de fase A, ver P19).
- **Revisión y reintento:** reintentos con *backoff* exponencial (`10000 × 2^(intento-1)`, tope configurable, por defecto 5 min) tras salidas anómalas; tras una salida limpia, si el *issue* sigue activo en el tracker, el trabajador continúa **en el mismo hilo de conversación** con otro turno, hasta `agent.max_turns` — mecanismo explícito de continuación multi-turno de larga duración, coherente con las sesiones de >6h de la Fuente 1 y las sesiones reanudables de la Fuente 8 (Ramp).
- **Fallos y recuperación:** taxonomía explícita de 5 clases de fallo (config/flujo, espacio de trabajo, sesión de agente, tracker, observabilidad), cada una con una recuperación distinta y documentada (saltar despacho, reintentar con *backoff*, saltar el *tick* y reintentar, no tumbar el orquestador) — el tratamiento más sistemático de C8 encontrado en toda la investigación A+A2.
- **Números de uso real: ninguno.** Es una especificación + implementación de referencia, no un informe de una organización usándolo en producción.
- **Caución sobre estrellas (aplicando §7: «las estrellas miden popularidad, no uso»):** el repo tiene **27.401 estrellas y 2.837 *forks*** (verificado en vivo, `gh api repos/openai/symphony`, 2026-09-25), creado el 2026-02-26 (~7 meses) — una cifra muy alta para algo que el propio README describe como «a low-key engineering preview for testing in trusted environments» y cuyo hilo de lanzamiento en HN apenas tuvo 25 puntos y 6 comentarios. Se intentó verificar si el crecimiento de estrellas es orgánico muestreando marcas de tiempo de *stargazers* vía la API de GitHub (cabecera `Accept: application/vnd.github.star+json`); **la API devolvió 404 en ambos intentos — no se pudo verificar**. Se marca explícitamente como sospechoso de inflado (mismo patrón de alerta que el caso Uvik en `desarrollo-agentes-investigacion.md`), sin poder confirmarlo ni descartarlo.
- **Evidencia en contra directa y de primera mano sobre la calidad del propio ejemplo del vendor:** en el hilo de HN, `exclipy` revisa las specs generadas por la propia implementación de referencia de Symphony (en Elixir) y escribe: «The specs are inscrutable agent slop. I want it to tell me what it does and instead it just lists database fields... whatever GPT model wrote this is not good enough (and the fact this was published by OpenAI suggests this is not an operator skill issue).» — es un fallo de calidad reportado sobre la propia demo oficial del vendor, el tipo de «evidencia en contra» que pide C13.
- **Tipo de evidencia:** especificación/documentación técnica de vendor; sin informe de adopción real; con un reporte negativo de primera mano sobre su propio ejemplo. No cumple §7 (afirmación de vendor sobre su propio producto, sin cifras de uso, sin segunda fuente independiente que confirme adopción real).
- **Contrastado:** NO.

### Fuente 7 (nueva, no encargada) — Spotify, «1,500 PRs Later: Spotify's Journey with Our Background Coding Agent (Honk)», parte 1

[https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1](https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1) — hallada buscando «background agents at» en HN Algolia.

- **Qué es:** un agente («Honk») integrado en el sistema interno *Fleet Management* de Spotify (gestiona cambios de código a través de miles de repositorios), que sustituye scripts de transformación deterministas por *prompts* en lenguaje natural.
- **Roles:** ingenieros y gestores de producto («product managers») definen objetivos de migración por *prompt*; el agente ejecuta la transformación y genera PRs; desarrolladores humanos revisan y fusionan. Dos modos distintos: un **agente de migración** (cambios sistemáticos a escala de toda la flota) y un **agente de planificación interactivo** separado, que reúne contexto vía Slack/GitHub Enterprise antes de pasar la tarea al primero — es decir, una división planificador/ejecutor entre dos agentes, no entre modelo fuerte/barato como en P2 de fase A, sino entre modo interactivo/modo batch.
- **Traspaso:** el *prompt* de migración en sí. Cita literal: «We replaced deterministic migration scripts with an agent that takes instructions from a prompt.»
- **Estado del trabajo:** distribuido entre la infraestructura de Fleet Management (selección de repos, gestión de PR), un CLI interno que coordina la ejecución/formato/*linting*/registro, y subida de registros a **Google Cloud Platform**; las trazas de ejecución se capturan en **MLflow** (herramienta abierta de seguimiento de experimentos de aprendizaje automático, reutilizada aquí para trazar ejecuciones de agentes) — mecanismo de trazabilidad no visto en ninguna otra fuente de A ni A2.
- **Concurrencia/aislamiento:** no detallado en el post; solo reconocen la necesidad: «We need robust guardrails and sandboxing to ensure agents operate as intended.»
- **Revisión:** incluye un paso de **«LLM as a judge»** (un modelo evalúa el *diff* generado) antes de la revisión humana — mecanismo de puerta automática previa no visto en las otras fuentes de A2.
- **Números:** «more than 1,500» PRs fusionadas desde el lanzamiento (~feb-2025); «around half of Spotify's pull requests» están automatizadas por Fleet Management desde mediados de 2024 — **matiz importante: esta cifra del 50% incluye también las transformaciones deterministas antiguas, no solo el nuevo agente basado en LLM**, no hay que confundirlas; reducción de tiempo del **60–90%** frente a codificación manual; ejemplo de complejidad: un script determinista de actualización de dependencias Maven de 20.000 líneas de código sustituido por un *prompt*; «hundreds of developers» interactúan con el agente.
- **Fallos/límites reconocidos:** «Performance is a key consideration, as agents can take a long time to produce a result, and their output can be unpredictable... Beyond performance and predictability, we also have to consider safety and cost.»
- **Tipo de evidencia:** informe de vendor/organización con cifras propias (§7, vía alternativa).
- **Corroboración en HN:** muy débil — 3 historias indexadas (3pts/0, 2pts/0, 2pts/2 comentarios), esencialmente sin discusión ni corroboración externa.
- **Confianza:** media-alta en las cifras (específicas y con contexto temporal), baja en la generalización del «LLM as a judge» como filtro de calidad (sin datos de precisión del propio juez).
- **Contrastado:** sí, vía alternativa de §7 (cifras propias), pero sin corroboración externa.

### Fuente 8 (nueva, no encargada) — Ramp, «Why we built our own background agent» (Inspect)

[https://builders.ramp.com/post/why-we-built-our-background-agent](https://builders.ramp.com/post/why-we-built-our-background-agent) — hallada indirectamente (un blog de terceros intentando replicar el sistema de Ramp citaba el post oficial).

Es la fuente con **mejor corroboración independiente de toda la investigación A+A2**.

- **Roles:** gestores de producto, diseñadores E ingenieros (no solo ingenieros, a diferencia de Stripe/Spotify) inician trabajo vía Slack, interfaz web, extensión de Chrome o comentarios en PR. El agente («Inspect») implementa y verifica su propio trabajo. Cita literal: «Inspect writes the code like any other coding agent, but closes the loop on verifying its work by having all the context and tools needed to prove it, as a Ramp engineer would.» Revisión humana obligatoria antes de fusionar, con una restricción de seguridad explícita (relevante para C12): «this would allow for any user to approve their own changes. You do not want to knowingly create a vector for unreviewed code to go into the codebase» — separan expresamente quién puede *pedir* un cambio de quién puede *aprobarlo*.
- **Traspaso/estado:** cada sesión tiene «its own SQLite database» sobre **Cloudflare Durable Objects** (primitiva de cómputo con estado persistente por objeto, sin servidor) — el estado vive en un objeto duradero por sesión, ni en git ni en un tracker externo. *Prompts* de seguimiento se **encolan** explícitamente: «We chose to queue them, as we found it not only easier to manage, but also helpful for sending over thoughts on next steps while the AI is still working.»
- **Concurrencia/aislamiento:** VM aislada («sandboxed VM») por sesión sobre **Modal** (plataforma de *sandboxing* en la nube), con entorno de desarrollo completo (Vite, Postgres, Temporal, etc.). Cita literal: «There's no limit to how many sessions you can have running concurrently, and your laptop doesn't need to be involved at all.» Soporta **sub-agentes**: «a crucial tool is one that allows it to spawn sessions itself» — topología jerárquica/recursiva real, confirmada por una empresa, y no solo por un comentarista de Reddit como en P9 de fase A.
- **Revisión y rehacer:** sesiones «multiplayer» — varios humanos y el agente trabajan a la vez en la misma sesión «just as they would in a branch of code»; el agente verifica su propio trabajo antes de entregarlo (ejecuta tests, revisa telemetría y *feature flags* en *backend*; verificación visual con capturas en *frontend*); las sesiones se pueden reanudar mediante instantáneas de la VM: «When the agent is finished making changes, we take another snapshot, and restore to it later if the sandbox has exited.»
- **Trazabilidad:** atribución por *prompt* («each person's prompt that causes code changes should be attributed to them»), IDs de sesión que enlazan cambios con *prompts* originales, monitorización vía *webhooks* de GitHub, rastro de auditoría en Slack.
- **Números:** **«~30% of all pull requests merged to our frontend and backend repos are written by Inspect. It only took a couple months for us to reach this level of usage.»** Métrica de adopción en vivo: conteo de «humans prompting» en los últimos 5 minutos.
- **Tipo de evidencia:** vendor/organización con cifras propias (§7, vía alternativa).
- **Corroboración en HN — la más fuerte encontrada en A2 (122 puntos, 27 comentarios):** un comentarista que se identifica explícitamente como empleado de Ramp (`memset`) corrobora desde dentro y de forma no promocional: «I work at Ramp and have always been on the "luddite" side of AI code tools... But. This tool is scarily good. I'm seeing it "1-shot" features in a fairly sizable code base and fixes with better code and accuracy than me.» Esto es una **corroboración independiente de primera mano dentro del propio hilo**, distinta de la marketing copy del post — el caso más sólido de «contrastado» de toda la fase. Otros comentarios relevantes: `cloudking` compara con Devin (agente competidor con capacidades similares — dato cruzado entre vendors); `martypitt` plantea directamente la pregunta de coste de construir-vs-comprar (C10).
- **Confianza:** alta — cifra de adopción concreta, con corroboración independiente de un empleado dentro del mismo hilo de discusión.
- **Contrastado:** **sí, con el estándar más alto encontrado en A2** (fuente primaria con datos propios + corroboración independiente y no promocional en el mismo hilo).

---

## 3. Patrones nuevos (P16+) y relación con P1–P15 de fase A

**P16. El repositorio como única fuente de verdad de todo el conocimiento del proyecto, con un agente de mantenimiento recurrente.** OpenAI (Fuente 1) formaliza y automatiza lo que en fase A solo aparecía como relato (P10, foro; P13, estudio académico): toda la documentación de diseño, calidad, arquitectura y seguridad vive versionada en `docs/`, con linters/CI que validan su frescura y un agente de «doc-gardening» recurrente que abre PRs de corrección. Refuerza P10/P13 y añade el mecanismo de mantenimiento automático que faltaba.

**P17. Revisión agente-a-agente como política deliberada en organizaciones de alto rendimiento, no solo degradación pasiva.** P5 de fase A documentaba erosión de la revisión humana como fenómeno no planeado (quejas de *burnout*, hallazgo académico de que la mayoría de PRs de agente no reciben revisión humana). OpenAI y Stripe muestran el mismo resultado —casi ninguna revisión humana directa— pero como **decisión de diseño explícita**: el agente se revisa a sí mismo y a otros agentes primero, de forma sistemática, y el humano solo entra si el bucle automático no resuelve. Es un matiz sobre P5, no una contradicción: el resultado observable (poca revisión humana) es el mismo; el motivo difiere entre «nadie tiene tiempo de revisar» (P5) y «revisar es ahora tarea del propio sistema, por diseño» (P17). Ninguna de las 4 fuentes principales de A2 (OpenAI, Stripe, Spotify, Ramp) publica una tasa de defectos post-*fusión* que permita comparar la seguridad real de ambos motivos.

**P18. Límite numérico explícito de rondas de reintento automático antes de escalar a humano.** Stripe fija «como máximo dos rondas de CI»; la especificación de Symphony (Fuente 6) formaliza lo mismo con *backoff* exponencial acotado (`10000 × 2^(intento-1)`, tope de 5 min) y un número configurable de turnos máximos. Concreta numéricamente, con 2 fuentes de vendor independientes, el patrón «un reintento antes de escalar» que en fase A (P3) aparecía solo en el relato de una persona sin cifra verificable.

**P19. El aislamiento de estado en ejecución (no solo de código) SÍ tiene solución estándar — pero solo en organizaciones con presupuesto de infraestructura cloud dedicada.** P9 de fase A documentaba que los *worktrees* de git aíslan código pero no bases de datos/puertos, y que cada desarrollador individual improvisa su propia solución. Stripe (devboxes EC2 dedicados) y Ramp (VM *sandbox* por sesión en Modal) muestran que a escala de empresa el problema sí se resuelve de forma estandarizada y reproducible. En cambio, la propia especificación de Symphony de OpenAI —pensada para ser ligera y portable— **no lo aborda en absoluto**: solo aísla el sistema de ficheros por *issue*, sin mención de aislamiento de estado en ejecución. Conclusión: el gap de P9 es real en la capa «herramienta ligera/individual», pero está resuelto en la capa «gran empresa con infraestructura cloud propia» a base de gasto en cómputo aislado por tarea, no de coordinación más inteligente.

**P20. Primera cifra dura de coste en tokens de un sistema multiagente encontrada en A+A2.** Anthropic (Fuente 5): un sistema orquestador-trabajadores usa ~15× más tokens que un chat simple; un agente individual, ~4×; el uso de tokens explica el 80% de la varianza en sus evaluaciones internas. Llena parcialmente el hueco que fase A señaló explícitamente en su §4 («ninguna fuente de esta fase da una cifra de coste total comparable entre patrones»). Sigue siendo un hueco para *desarrollo de software* específicamente (esta cifra es de un sistema de investigación, no de codificación), y sigue sin haber una segunda organización que reporte una cifra de coste comparable.

**P21. El estado de orquestación, incluso en la especificación más formal encontrada, es efímero y constituye un punto único de fallo no resuelto.** Symphony (Fuente 6) documenta explícitamente que su estado de orquestación vive solo en memoria: si el proceso muere, se pierden todos los reintentos programados y sesiones en curso, y la recuperación depende por completo de volver a interrogar al *tracker* externo y reutilizar espacios de trabajo en disco. Ninguna fuente de fase A había documentado este fallo de forma tan explícita porque ninguna describía un orquestador formal con ese nivel de detalle. Refuerza el hueco C8 («qué pasa cuando algo falla») con un caso concreto, documentado por el propio autor del diseño.

**P22 (evidencia en contra, transversal a las 4 fuentes principales de organización — cumple el pedido explícito de informes negativos).** Ninguna de las 4 fuentes de organización con cifras (OpenAI, Stripe, Spotify, Ramp) publica su tasa de defectos, incidentes en producción o *reverts* atribuibles a código generado por agentes tras la fusión. Los comentarios más votados de HN en 3 de las 4 discusiones (`iepathos`/Stripe, `apical_dendrite`/OpenAI, `martypitt`/Ramp) señalan exactamente esta ausencia como el punto ciego más importante de los propios informes. Es el mismo hueco que fase A ya había marcado en su §4 («qué pasa cuando el humano revisor deja de revisar por completo... ningún estudio mide sus consecuencias a medio plazo de forma cuantitativa») — A2 confirma que sigue abierto en las 4 fuentes de mayor peso encontradas hasta ahora.

**P23 (evidencia en contra concreta y verificable, un solo caso pero directo).** El propio ejemplo de referencia que OpenAI publica para demostrar Symphony (specs generadas en Elixir) recibe una crítica de calidad de primera mano y verificable («inscrutable agent slop», `exclipy` en HN) — un fallo del patrón «documentación generada por agente como fuente de verdad» (P16, P10, P13) en el peor escaparate posible: la demo oficial del propio vendor que promueve el patrón.

---

## 4. Respuesta provisional actualizada a las 6 preguntas (integra fase A + A2)

### 1. Roles

Fase A no encontró un reparto «correcto» contrastado como superior a otros. A2 añade cuatro variantes reales a escala de empresa, todas con la misma estructura de fondo — **humano decide/prioriza, agente implementa y se autorrevisa, humano solo interviene en el margen** —, pero con distribución de quién puede *iniciar* trabajo cada vez más ancha: solo ingenieros (Stripe, con Codex de OpenAI dirigido por 3→7 ingenieros); ingenieros + PMs (Spotify); PMs + diseñadores + ingenieros (Ramp, el más amplio). En ningún caso encontrado en A2 hay un rol humano dedicado exclusivamente a revisar código generado por agentes a tiempo completo — la revisión humana es residual y ocurre solo cuando el bucle automático (agente revisa a agente) no resuelve. Esto no contradice la lectura de fase A sobre erosión del rol de revisión; la matiza como decisión de diseño explícita en las organizaciones de mayor rendimiento (P17), no solo como fenómeno de desgaste no planeado.

### 2. Traspaso

Se confirma con más fuerza el hallazgo de fase A: el traspaso que funciona es un **contrato verificable**, no una conversación. A2 añade que, en las organizaciones de mayor escala, ese contrato tiende a NO vivir en un *ticket* tradicional sino en uno de tres lugares: (a) el propio repositorio como base de conocimiento estructurada y mantenida por un agente (OpenAI, P16); (b) un objeto de *issue* normalizado extraído de un *tracker* externo y renderizado en *prompt* (Symphony); (c) un *prompt* efímero en Slack/CLI/web sin *tracker* formal, con el contexto reconstruido en el momento vía un servidor central de herramientas tipo MCP (Stripe, Ramp). Ninguna fuente de A2 describe el «plan como contrato ejecutable con revisión de plan» (P2/P4 de fase A) de forma tan explícita como los relatos individuales de foro — es un patrón más visible en equipos pequeños/individuales que en los 4 informes de gran empresa de A2.

### 3. Coordinación

A2 confirma con la mayor precisión encontrada hasta ahora que **git/*worktree* sigue siendo la base**, pero añade una capa de orquestación explícita por encima en las implementaciones más formales: un componente único (el orquestador) que sondea el *tracker*, aplica límites de concurrencia globales y por estado, y gestiona reclamos (*claims*) para evitar despacho duplicado (Symphony). Se confirma también el hallazgo de fase A de que el aislamiento de *estado en ejecución* (no solo de código) sigue siendo un problema abierto en las herramientas ligeras (Symphony no lo resuelve), pero A2 muestra que **sí está resuelto en las grandes empresas mediante infraestructura cloud dedicada por tarea** (devboxes de Stripe, VMs de Modal en Ramp) — matiza P9 (ver P19). Novedad de A2 no vista en fase A: el propio estado de orquestación puede ser un punto único de fallo no resuelto incluso en la especificación más formal (P21).

### 4. Topología

Se refuerza con una fuente de vendor (no solo foros) el hallazgo de fase A de que **pocos agentes concurrentes, no enjambres masivos, es la norma práctica**: Anthropic documenta explícitamente que su arnés de larga duración corre «solo un agente por sesión» (Fuente 4), y el propio `daxfohl` en HN descarta el patrón de «agente de QA dedicado» por experiencia. Donde SÍ aparece concurrencia alta y deliberada es en el nivel de *tareas independientes* paralelas (varios minions/sesiones de Ramp o Stripe corriendo a la vez, cada uno en su propio aislamiento), no en el nivel de *un mismo problema* dividido entre muchos agentes coordinándose entre sí — distinción que fase A no había hecho tan explícita. La única topología jerárquica/recursiva real confirmada por una empresa (no solo un comentario de Reddit como en P9) es la de Ramp: un agente puede generar sus propias sub-sesiones. El sistema orquestador-trabajadores de Anthropic (Fuente 5) es el caso mejor documentado de jerarquía real con datos de coste (~15× tokens) y rendimiento (+90,2%), aunque es de investigación, no de codificación.

### 5. Revisión

Sigue siendo la pregunta con más evidencia, y A2 la refuerza y la matiza a partes iguales. Se confirma con cifras nuevas y consistentes entre sí (Stripe: máx. 2 rondas de CI antes de escalar; Symphony: *backoff* acotado configurable) el hallazgo de fase A de que la revisión automática precede casi siempre a la humana. La novedad de A2 es que esto es presentado por los propios vendors como **política deliberada de alto rendimiento**, no como degradación (P17) — con la salvedad de que ninguna de las 4 organizaciones de A2 publica una tasa de defectos post-*fusión* que permita juzgar si esa política es segura (P22), exactamente la laguna que fase A ya había señalado. Aparece un mecanismo nuevo no visto en fase A: «LLM as a judge» (juez automático) como puerta previa a la revisión humana (Spotify).

### 6. Trazabilidad

Sigue siendo la pregunta peor resuelta en ambas fases combinadas. A2 no encuentra ningún mecanismo de atribución verificable por terceros (firma criptográfica, identidad de bot auditable) más allá de lo ya establecido en F2 (`Co-authored-by`, *bots* de GitHub App). Sí aparecen mecanismos de *observabilidad interna* nuevos y no vistos en fase A: trazas en MLflow reutilizado para agentes (Spotify), IDs de sesión ligados a *prompts* con atribución por persona incluso cuando varias personas colaboran en una misma sesión (Ramp), y observabilidad de patrones de decisión sin leer el contenido de las conversaciones por privacidad (Anthropic) — los tres son mecanismos operativos útiles para depurar y auditar el *comportamiento* del sistema, pero ninguno resuelve la pregunta original de C7 («qué rol, agente o modelo hizo cada cambio y quién lo aprobó») de forma verificable por alguien fuera de la propia organización.

---

## 5. Fuentes

### Encargadas, leídas en fuente primaria completa

- OpenAI, «Harness engineering: Leveraging Codex in an agent-first world» — https://openai.com/index/harness-engineering/ (leída vía navegador CDP, WebFetch da 403).
- HN, story 48416264 (297 pts/206 com.) — https://news.ycombinator.com/item?id=48416264
- Stripe, «Minions...» parte 1 — https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents
- HN story 47110495 (93 pts/81 com.) — https://news.ycombinator.com/item?id=47110495
- Stripe, «Minions...» parte 2 — https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2
- HN story 47086557 (131 pts/61 com.) — https://news.ycombinator.com/item?id=47086557
- Anthropic, «Effective harnesses for long-running agents» — https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- HN story 46081704 (125 pts/37 com.) — https://news.ycombinator.com/item?id=46081704
- Anthropic, «How we built our multi-agent research system» — https://www.anthropic.com/engineering/built-multi-agent-research-system
- HN story 44272278 (35 pts/0 com.) — https://news.ycombinator.com/item?id=44272278
- Simon Willison, notas sobre el post anterior — https://simonwillison.net/2025/Jun/14/multi-agent-research-system/
- HN story 44280445 (12 pts/0 com.) — https://news.ycombinator.com/item?id=44280445
- OpenAI Symphony, repo + README + SPEC.md — https://github.com/openai/symphony , https://raw.githubusercontent.com/openai/symphony/main/SPEC.md
- HN stories 47252045 (25 pts/6 com.), 47257966 (4 pts/0), 47278403 (3 pts/1).

### Nuevas, halladas en la búsqueda de informes comparables no cubiertos

- Spotify, «1,500 PRs Later: Spotify's Journey with Our Background Coding Agent (Part 1)» — https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1
- HN stories 45840407 (3 pts/0), 45943614 (2 pts/0), 45846938 (2 pts/2).
- Ramp, «Why we built our own background agent» (Inspect) — https://builders.ramp.com/post/why-we-built-our-background-agent
- HN story 46589842 (122 pts/27 com., con corroboración de un empleado de Ramp identificado) — https://news.ycombinator.com/item?id=46589842
- Blog de terceros que cita a Ramp como fuente oficial — https://eliot.blog/p/attempting-to-rebuild-ramps-background

### Consultadas sin aportar nada útil (todas las búsquedas HN adicionales)

"how we use coding agents at", "agents wrote", "background agents at" (parcialmente útil — dio Spotify y el hilo sobre Ramp), "unattended coding agents", "our agent workflow", "PRs per week agents", "engineering blog coding agents", "Ramp engineering coding agent", "we tried multi agent coding and stopped", "AI agents did not work for us engineering", "our experience with autonomous coding agents failure", "coding agent fleet production incident", "abandoned AI agent coding workflow", "GitLab Duo agent platform team", "Shopify AI agent engineering blog", "Sourcegraph Amp production usage" — todas sin resultado de práctica organizativa relevante, detalle en §1.

### Intentos fallidos de verificación

- `gh api repos/openai/symphony/stargazers` (con y sin cabecera `Accept: application/vnd.github.star+json`): 404 en ambos casos — no se pudo verificar si el crecimiento de estrellas del repo es orgánico.

---

## 6. Bloqueos

- **API de *stargazers* de GitHub devuelve 404** para `openai/symphony` con la cabecera de marcas de tiempo — impide verificar si sus 27.401 estrellas son un crecimiento orgánico o inflado; queda marcado «sin verificar», no descartado ni confirmado.
- **Ningún informe de abandono a escala con datos propios**, pese a 6 formulaciones de búsqueda distintas en HN Algolia. No es evidencia de que no existan — solo de que HN Algolia no los indexa con esas palabras clave; sí se encontró evidencia en contra *dentro* de los 4 informes positivos (P22, P23) y en sus discusiones de HN, que se documenta como la mejor aproximación disponible al pedido explícito de «negativo/abandono».
- **La discusión de HN sobre el post de Anthropic de investigación multiagente (Fuente 5) tiene 0 comentarios** — toda la corroboración disponible es la nota de Simon Willison, que es endoso técnico independiente pero no una réplica con datos propios de una segunda organización.
- WebSearch no se usó en absoluto en esta fase (cupo agotado, según el encargo); toda la investigación se hizo con WebFetch, `curl` a APIs públicas (HN Algolia, GitHub API), `gh api`, y el navegador Chrome real solo para la página de OpenAI (bloqueada a WebFetch/curl con 403).
- Ningún bloqueo de red o herramienta impidió completar las 6 preguntas con evidencia media o mejor; los huecos reales siguen siendo de contenido — sobre todo la ausencia sistemática de cifras de defectos/incidentes post-fusión en las 4 fuentes principales (P22), que ya fase A había señalado y A2 confirma que sigue abierta.

## Enlaces

- [[circuito-tareas-definicion]] — encargo y criterios de esta fase
- `A-practicas-reales.md` — fase A (patrones P1–P15, no repetidos aquí)
- [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] — evidencia de trazabilidad reutilizada
- [[desarrollo-agentes-investigacion]] — síntesis general, caso Uvik (estrellas infladas)
- [[sistema-desarrollo-con-agentes]] — proyecto

## Enlaces

- [[flujo-agentes-informe]] — síntesis de la investigación
- [[circuito-tareas-definicion]] — definición de la fase
- [[_index]]
