---
title: Gas City — qué mecanismos sirven y qué es marketing
created: 2026-09-27
updated: 2026-10-04
tags: [gas-city, yegge, verificacion, puerta, auditoria, investigacion]
zona: tecnico
---

Investigación del artículo de Steve Yegge [*Welcome to Gas City*](https://steve-yegge.medium.com/welcome-to-gas-city-57f564bb3607) (24-abr-2026, 4.613 palabras) y de la plataforma que describe, contra los tres problemas que aparecen en cualquier sistema multiagente serio: **verificar** que el trabajo está bien, **impedir** que se fusione sin pasar por una puerta, y **medir** si el conjunto va bien o solo va rápido.

## 1. Introducción: qué se pregunta y por qué

La pregunta es si Gas City aporta algo construible. El criterio, declarado antes de buscar: **entra lo que documente mecánica de un sistema que corre de verdad, con detalle suficiente para comparar**; no entra visión sin mecanismo, ni lo que ya estuviera en el wiki. Y el principio que sirve de vara de medir, recogido en [[verificacion-externa-agentes]]: la verificación solo cuenta si la posee algo **distinto del agente y fuera de su alcance de escritura**.

Conclusión adelantada, porque es lo que sostiene el resto: **sí sirve, pero no donde el artículo dice.** El artículo es la parte menos aprovechable; la plataforma que hay debajo tiene tres mecanismos concretos y ninguno aparece mencionado en el artículo.

## 2. Considerado y descartado

| Qué | Motivo del descarte |
|---|---|
| **La tesis del artículo** (las «dark factories», comerse el SaaS, el «de-SaaSer») | Es encuadre de negocio sin una sola cifra de resultado. Se usa para situar el vocabulario, no entra en el diseño. Motivo: ningún número de calidad de software en todo el texto. |
| **El claim de auditoría** — *«The forensics and auditing capabilities of Gas City are unparalleled, because of MEOW and Dolt»* y *«That's your SOC2 story, sitting right there in the database, already written»* | **Contradicho por la fuente primaria.** En el índice completo de la documentación (`docs.gascity.com/llms.txt`) **no hay ninguna página** de auditoría, forense, verificación, recibos ni medición. Y el propio tracker del proyecto documenta el registro de auditoría destruyéndose: #6523 *«four formulas close beads with `--notes`, destroying rulings and measurements at seven sites»*; #5846 *«no ownership check, no CAS, no audit»*; #6444 *«no casualty record; session.stopped reads as a clean drain»*; #6222 sobrescritura silenciosa por leer-modificar-escribir fuera de transacción. |
| **La claim de fiabilidad del artículo** — *«Reliability, friends, is a dial»*, «más rondas de revisión, más jueces» | Sin un solo número ni denominador. Lo único medido que el proyecto publica sobre sí mismo es su propio sobrecoste: issue #3924 *«Recover orchestration overhead of ~25% of formula-run wall clock: measured 3h35m profile»*. No entra como dato de diseño. |
| **La promesa de «la comunidad ha hablado»** — *«The community has spoken: This is The Way»*, *«over two thousand members on the Discord»* | La tracción en GitHub dice otra cosa (tabla en §3.5): 1.309★ frente a 18.185★ de Gas Town. No sostiene la adopción que sugiere. |
| **El mecanismo de fiabilidad que el artículo *sí* propone**: equipos de 2-3 agentes vigilándose — *«You should always have at least two or three working together on a little crew»*, *«much like a second hash function, dramatically decreases the chance of some sort of collision»* | Es exactamente el mecanismo que la evidencia **rechaza**: autocorregirse sin oráculo externo empeora el resultado, y el techo de la revisión por LLM medido está en ~50-60 % con el «techo de intención» como fallo dominante ([[verificacion-externa-agentes]]). Se descarta como base de fiabilidad. **No se descarta como pieza**: sirve si va *terminada* por un script, ver §3.1. |

## 3. Análisis detallado

### 3.1. El hallazgo principal: el bucle de verificación (`check`)

La plataforma tiene un constructo de verificación de primera clase, y **el artículo no lo menciona en ningún momento**. Cita literal de la [especificación de fórmulas v2](https://docs.gascity.com/reference/specs/formula-spec-v2):

> `[steps.check]` wraps a step in an inline run/check verification loop: after each iteration closes, **the orchestrator runs the configured script**. Exit 0 closes the step; any other nonzero exit is a "not yet" verdict that spawns the next iteration while budget remains, and exhaustion closes the step as failed.

Y la formulación del principio, en la [guía de fórmulas](https://docs.gascity.com/guides/understanding-formulas):

> `check` is for work you can verify: the step is done **when your script says so, not when the agent says so**.

Eso es, palabra por palabra, el principio que sostiene [[verificacion-externa-agentes]]. Lo que se aprovecha es el **patrón**, no la garantía — ver §3.4.

### 3.2. El tercer estado ya está implementado, con nombre y convención

El diseño de un recibo de verificación con tres resultados —`PASSED | FAILED | NO_VERIFICABLE`— venía justificado por analogía con otros campos. **Gas City lo tiene implementado**, con convención numérica y presupuesto propio. Cita textual de la misma especificación:

> A check script reports two different things with a nonzero exit: "the thing I verify is not true yet" (a verdict), and "I could not reach the infrastructure I need in order to tell" (a blind read). **Only the first should cost an attempt.** A blind read that is counted as a verdict spends the step's budget on a question the script never answered.

| Estado | Salida | ¿Consume intento? | Consecuencia |
|---|---|---|---|
| Paso | `0` | n/a | `PASSED` |
| Infraestructura inalcanzable | `75` (`EX_TEMPFAIL`, de `sysexits.h`) | **No** — reintento sin coste | `NO_VERIFICABLE` |
| Cualquier otro distinto de cero | — | Sí — el veredicto es «aún no» | `FAILED` → nueva iteración |

Tres cosas se ganan, y son concretas: el tercer estado **deja de ser una analogía importada y pasa a tener precedente en software**, con nombre y convención; se puede **adoptar `75` en vez de inventarse un código** (no es arbitrario: es la convención que el propio proyecto ya usaba, y el sesgo documentado es deliberado — *«exit 75 is the signal a script declares deliberately, and the one to write against»*); y un detalle fino que merece copiarse: **el presupuesto de infraestructura es un presupuesto aparte**, *«so a script that exits 75 forever still terminates; it just does not burn the semantic budget on the way there»*.

### 3.3. Puertas y presupuestos

| Mecanismo | Cómo funciona | Cita textual | Reserva |
|---|---|---|---|
| **Puerta** | Un *gate* es una primitiva: `[steps.gate]` sintetiza un bead hermano de tipo `gate` y una arista `blocks` | *«the step stays blocked until the gate bead is closed»* | **Declarado pero sin consumidor en ejecución**: *«The gate `type` vocabulary and the `waits_for` mode distinction have no runtime consumer»*. Es diseño a copiar, no función en la que apoyarse |
| **Contador de intentos y bloqueo** | `max_attempts` (≥ 1) y `on_exhausted: hard_fail \| soft_fail`, tanto en `check` como en `retry` | *«Total attempts including the first»* | Real y en uso |
| **Orden sin planificador central** | Aristas `needs` bloqueantes sobre beads | *«a bead with an open blocker is invisible to agents until that blocker closes — which is how ordering happens with no central scheduler»* | Real. Depende de Beads, cuyos defectos están catalogados (el TTL de *claim* hardcodeado e inerte, #5681) |
| **Durabilidad del trabajo** | El orquestador lee el progreso del almacén de beads, no de las sesiones | *«The loop closes through shared state, which is why work survives a crash on either side»* | Real |

### 3.4. Lo que Gas City **no** da, y es justo el agujero

Aquí está la frontera del hallazgo, y conviene no maquillarla.

- **Los roles son prompts, no barreras.** Cita textual de [How Gas City Works](https://docs.gascity.com/getting-started/how-gas-city-works): *«a "reviewer" or a "planner" is nothing more than the prompt you wrote for it»*, y *«the orchestrator hardcodes zero roles — no built-in "manager" or "reviewer."»*
- **La plataforma declara que no es un recinto.** Cita textual de [Command Execution Trust Boundaries](https://docs.gascity.com/reference/trust-boundaries): *«Gas City intentionally runs operator-configured commands. Those commands are a feature, not a sandbox.»* Y el propio artículo lo asume: *«You will need to wire it to your own sandboxing, MCP servers, and so on.»*
- **Conclusión precisa, sin pasarme:** Gas City aporta **separación por configuración** (agente distinto, ámbito declarado, script ejecutado por el orquestador) pero **no separación forzada**. No he encontrado en la documentación ninguna afirmación de que el agente no pueda escribir el script de verificación ni la fórmula que lo invoca. **Hueco de búsqueda declarado:** no revisé el código de `gastownhall/gascity-packs`, solo su documentación y el volcado del tracker.

Y un detalle operativo que se deduce de la documentación, **marcado como deducción mía, no como vulnerabilidad documentada**: la precedencia de capas de fórmulas hace que *«FormulasDir is a rig-local formula directory — the highest-priority formula layer, above city pack formulas»*. Es decir, **una fórmula dentro del rig pisa por nombre a la del pack de la ciudad**. Si el agente escribe en el rig, la ruta que define *cómo se verifica* puede redefinirse desde dentro. Es una capa que conviene blindar explícitamente en vez de heredar su precedencia.

### 3.5. Adopción real: los números, con su denominador

| Medida | Gas City | Gas Town | Beads |
|---|---|---|---|
| Estrellas (`gh api`, 2026-09-29) | **1.309** | 18.185 | 27.404 |
| Forks | 430 | 1.675 | 1.855 |
| Issues abiertas | **1.364** | 481 | 1.293 |
| Creado | 2026-02-22 | 2025-12-16 | 2025-10-12 |
| Última versión | v1.4.2 (2026-09-18) | — | — |

Gas City tiene **el 7,2 % de las estrellas de Gas Town** y **más issues abiertas que estrellas**. El hilo de HN del lanzamiento que analizamos: **33 puntos**, frente a 403, 354, 253 y 219 de los cuatro hilos de Gas Town. No es «la comunidad ha hablado»: es un proyecto con una fracción de la tracción de sus predecesores.

### 3.6. La hipótesis rival que hay que tomar en serio: adoptar la plataforma o construir lo propio

Es el mejor caso **contra** la conclusión de este informe, y no se despacha en una línea.

**A favor de adoptar:** es MIT, está vivo (push hoy mismo), y ya trae —verificado en su documentación— el bucle de verificación, el tercer estado, las puertas, los presupuestos de intentos, el grafo de trabajo durable, agentes con identidad y ámbito declarados, bus de eventos, reutilización por pack/fórmula, el rastro en Dolt, la limpieza de secretos del entorno (*«remove inherited environment variables whose keys look secret-bearing»*, con `TOKEN`, `PASSWORD`, `SECRET`, `API_KEY`…) y una tabla de fronteras de confianza que marca los campos de texto libre como *«Untrusted data — do not concatenate into shell commands»*. Construir eso a mano es mucho trabajo.

**En contra, con fuente:**
- **No cierra el agujero, lo muda de sitio.** El problema es el oráculo, y Gas City explícitamente no lo da (*«wire it to your own sandboxing»*). Adoptar la plataforma no resuelve el problema que motiva construir nada.
- **Madurez y criticidad.** El tracker son 1.364 issues abiertas, y las recientes son fontanería: *«transcript discovery hard-codes ~/.claude/projects and ignores CLAUDE_CONFIG_DIR — every wake silently starts a new conversation and orphans the old transcript»* (#6677), *«one rig's or pack's config-load failure silently stops every bead-backed order in the city»* (#6656).
- **Voz de la comunidad, no del fabricante.** Un usuario con experiencia real: *«it was a little too nondeterministic for production code»*, y — lo más incómodo para el caso de adopción — *«if you use Fable, slap in an epic planning document, and ask it to run a workflow… it's almost as good as gastown/gascity but **far more predictable**»* ([HN 48840185](https://news.ycombinator.com/item?id=48840185), jaggederest). Otro: *«I did switch over to using Gascity, but it does still seem to have quite a few troubles»* ([HN 47695269](https://news.ycombinator.com/item?id=47695269), kvanbeek). Y la crítica estructural: *«I really fundamentally do not understand what problem Gas City solves that is not already solved by normal subagent orchestration patterns… Why do we need hundreds of thousands of lines of opaque Go code to accomplish any of this?»* ([HN 48015406](https://news.ycombinator.com/item?id=48015406), thurn). Aviso de encuadre: *«These deployments can be for anything»* es del artículo; el propio repo se describe más estrechamente como *«Orchestration-builder SDK for multi-agent coding workflows»*.
- **Peso operativo**: exige `tmux`, `jq`, `git`, **`dolt` ≥ 2.1.0**, **`bd` (Beads) ≥ 1.0.4**, `flock` y `gh`. Dolt es un proceso de base de datos, no una librería. **Mitigación verificada el 2026-09-29:** `GC_BEADS=file` evita dolt y bd ([[gas-city-instalacion-y-modelos]] §3).

### 3.7. El proyecto sí publica métricas — y dicen lo contrario de lo que sugiere el artículo

Existe [`gastownhall/gascity-project-dashboard`](https://github.com/gastownhall/gascity-project-dashboard), un repo que corre un workflow semanal, guarda los datos en `data/gascity/snapshots.jsonl` y publica el resultado en la wiki del proyecto. **Hay 21 semanas publicadas, la última del 2026-09-20**, con 41 métricas.

Lo que publican no es calidad del software que Gas City produce, sino **la salud de su propio repositorio**: **nadie publica métricas del software que sale de la fábrica; este publica métricas de sí mismo, y es el único de los casos conocidos que se molesta en hacerlo.** Y esas métricas, leídas en serie, son la mejor evidencia empírica del modo de fallo que este tipo de sistemas produce:

| Métrica (última semana, 2026-09-20) | Valor | Qué dice |
|---|---|---|
| `agent_merged_prs_30d` | **0 PRs** | Cero PRs de agente fusionados en 30 días |
| `agent_pr_human_review_coverage_30d` | **`None`** | **Definieron exactamente la métrica de «tasa de intervención humana»** y **no la pueden calcular**: la columna existe, el valor es nulo |
| `unique_reviewers_30d` / `reviewer_concentration_factor_30d` | **1 / 1** | Un solo revisor para todo el proyecto, concentración máxima |
| `ci_success_rate_7d` | **45,1 %** (67 fallidos de 122 commits) | El denominador es del propio snapshot; la resta cuadra (67/122 = 54,9 % de fallo) |
| `open_prs` / `stale_prs_14d` | **493 / 242** | La mitad de los PRs abiertos llevan más de dos semanas parados |
| `external_pr_merge_median_hours_30d` | **382,78 h** (~16 días, 63 PRs) | Tiempo mediano hasta fusionar un PR de fuera |
| `open_issues` / `stale_issues_30d` | **1.289 / 468** | |
| `bug_issues_created_30d` | **119** | |
| `release_downloads_total` | **7.876** | Uso real, no solo interés: hay gente ejecutándolo |

Y la serie temporal de las 21 semanas es la parte que importa, porque **no es un mal día, es una tendencia**:

| Semana | CI en verde | PRs de agente fusionados | PRs abiertos |
|---|---|---|---|
| 2026-05-24 | 75,2 % | 2 | 154 |
| 2026-06-14 | 71,1 % | 0 | 160 |
| 2026-07-12 | 43,3 % | 0 | 267 |
| 2026-08-06 | 65,0 % | 0 | 346 |
| 2026-08-23 | 40,9 % | 0 | 442 |
| 2026-09-20 | 45,1 % | 0 | **493** |

En cuatro meses: el CI en verde baja del 75 % al 45 %, los PRs de agente fusionados se quedan en **cero y no se recuperan**, y la cola de PRs abiertos se multiplica por 3,2 (154 → 493) con **un solo revisor**. Eso es, medida por el propio autor del concepto, **la cola creciendo mientras el oráculo no escala**.

Dos cautelas para no sobreleer: los 0 PRs de agente pueden significar «no se etiquetan como de agente», no «no los hay»; y estas cifras son del repositorio del propio proyecto, no de las fábricas de sus usuarios. Dicho eso, **es el único conjunto de datos en serie que existe sobre un sistema de este tipo, y lo publica él**.

## 4. Recomendaciones

Ordenadas por relación entre lo que cuesta y lo que aporta.

1. **Adoptar el tercer estado con la convención `75`.** Es lo único que cambia una decisión ya escrita: `NO_VERIFICABLE` ya no se justifica solo con analogías de otros campos, sino con una implementación real en un orquestador, y el recibo debe declarar **la salida concreta** que produjo cada estado, no solo el estado. Copiar también el presupuesto de infraestructura separado.
2. **El patrón de revisión terminada por script.** Gas City aporta la forma exacta: carriles de revisión en abanico (LLM, acotados), un sintetizador que los junta, un paso de arreglo, y **el bucle que repite hasta que pasa un veredicto por script** — *«gates the re-review with a `[steps.check]` that runs an artifact validator»*. Los carriles LLM generan hallazgos, pero **la condición de salida es un script, nunca otro LLM**.
3. **Copiar la lista de métricas de su fuente más parecida, en vez de inventarla.** El dashboard semanal de Gas City publica 11 métricas que sirven casi tal cual, con la lección de diseño que vale más que la lista: **cada métrica va con su `source` y su `unit` declaradas en el propio dato**, así el número no se puede citar sin su procedencia. Y la señal que hay que vigilar de cerca: `agent_pr_human_review_coverage_30d` — la tasa de intervención humana, que aquí existe como columna y **vale `None`**.
4. **Blindar la capa de verificación contra escritura desde el rig.** Consecuencia directa de la precedencia de `formulas_dir` (§3.4): la fórmula y el script de verificación deben vivir fuera del alcance de escritura del ejecutor, y eso hay que declararlo como invariante con su comprobación, no como supuesto.
5. **Robar dos controles de seguridad concretos**: la limpieza de variables de entorno heredadas por nombre sospechoso (`TOKEN`, `PASSWORD`, `SECRET`, `API_KEY`, `CREDENTIAL`, `OAUTH`…) y la tabla de fronteras que marca los campos de texto libre como datos no confiables que no se concatenan a la shell.
6. **Añadir a la vigilancia periódica una señal concreta y barata**: el dashboard semanal de Gas City. Es la única serie temporal pública que existe sobre un sistema de este tipo, se actualiza sola y tarda un minuto en leerse.
7. **No cambiar nada del diseño por el artículo.** Sus dos claims grandes —auditoría «sin parangón» y «la fiabilidad es un dial»— no están respaldados ni por su producto ni por su comunidad.

## 5. Dónde se ha buscado

| Fuente | Tipo | Resultado |
|---|---|---|
| [Artículo, *Welcome to Gas City*](https://steve-yegge.medium.com/welcome-to-gas-city-57f564bb3607) | Autor (primaria de la claim) | Leído completo, 4.613 palabras. `WebFetch` → **403**; obtenido con navegador real por CDP (`/navegador-cdp`) |
| `docs.gascity.com/llms.txt` y 7 páginas de documentación | Artefacto primario del proyecto | **La fuente que decide.** De aquí salen `check`, `gate`, los presupuestos y la distinción veredicto/infraestructura |
| `gh api orgs/gastownhall/repos` — los 12 repos de la organización | Artefacto primario | **Lo que más aportó después de la documentación.** Aquí aparecieron el dashboard de métricas, el stack de OpenTelemetry y el protocolo de federación (§7) |
| [`gascity-project-dashboard`](https://github.com/gastownhall/gascity-project-dashboard): README, `scripts/project_dashboard.py` y `data/gascity/snapshots.jsonl` (21 semanas) | Artefacto primario | Las métricas semanales de §3.7. Comprobada la aritmética del propio snapshot (67 fallidos de 122 = 54,9 % ↔ 45,1 % de éxito) |
| `gh api repos/gastownhall/gascity` (+ `/releases`) y el clon del repo (4.378 ficheros Go) | Artefacto primario | Estrellas, forks, issues, fechas, versión; y los mecanismos que ninguna documentación describe ([[gas-city-instalacion-y-modelos]]) |
| Tracker: `search/issues` para `verification`, `audit trail`, `oracle`, `sandbox`, `false positive`; y 25 issues recientes | Artefacto primario | Contra-evidencia del claim de auditoría (#6523, #5846, #6444, #6222) y el sobrecoste medido (#3924) |
| HN vía API de Algolia: historias y 388 comentarios con «gascity» | Comunidad independiente | Voz de usuarios reales y críticas estructurales (§3.6) |
| [DoltHub, *A Week In Gas Town*](https://www.dolthub.com/blog/2026-03-24-a-week-in-gas-town/) (Tim Sehn, 2026-03-24) | Comunidad independiente | Cinco días de uso real: *«The whole experience cost me $3,000»*, polecats *«merging directly to master»*, y *«Humans, code review at your own risk»*. **Es Gas Town** |
| [`gastownhall/wasteland`](https://github.com/gastownhall/wasteland) · [`gascity-otel`](https://github.com/gastownhall/gascity-otel) · [`gascity-packs`](https://github.com/gastownhall/gascity-packs) | Artefacto primario, **no leídos en profundidad** | Pistas abiertas: federación entre fábricas (abandonada 2026-07-07), el stack de observabilidad (abandonado el día que nació) y los 19 packs donde viven las fórmulas de revisión. **Hueco declarado** |
| Discord de gastownhall.ai | Comunidad | **No consultado.** Requiere cuenta; la afirmación de «two thousand members» no la he podido verificar |

**Cobertura:** de las fuentes que existen sobre Gas City, miré las dos primarias (documentación y tracker) y la comunidad de HN. No miré el código de los packs ni el Discord. Lo que queda sin mirar es precisamente donde estaría la prueba de si el verificador está aislado.

## 6. Verificación y límites de este informe

- **Qué se comprobó mecánicamente:** las 10 citas de la documentación y 8 de las de comunidad se verificaron con búsqueda literal sobre los ficheros descargados. Dos fallaron en la primera pasada por saltos de línea y `**negritas**` en el original, y se localizaron y releyeron antes de citarlas. Una cita que había atribuido al artículo resultó ser de un comentario de HN y se ha reatribuido. La aritmética del dashboard se comprobó contra su propio denominador.
- **Corrección de un error mío, encontrado en una segunda pasada:** una primera versión afirmaba que Gas City era «un caso más» de que nadie publica métricas. **Era falso** — publica 21 semanas de métricas semanales; lo que no publica es calidad del software que sale de la fábrica. La lección de método: **el primer barrido miró los repos que el artículo y la documentación nombran, y no la organización entera del proyecto**, que es donde estaban el dashboard, el stack de observabilidad y el protocolo de federación.
- **Qué es hecho, qué es supuesto y qué es juicio.** Hechos con URL y cita: todo lo de §3.1 a §3.5. Supuesto sin fuente: que el agente no puede escribir la capa de fórmulas de la ciudad — la documentación no lo dice ni a favor ni en contra. Juicio mío, etiquetado: que la precedencia de `formulas_dir` sea un riesgo explotable (§3.4) y la recomendación de no adoptar la plataforma (§3.6).
- **El mejor caso contra la conclusión del informe:** si los carriles de revisión de Gas City resultaran producir código correcto en la práctica, la estimación sobre el techo de la revisión por LLM (~50-60 %) sería demasiado pesimista. **Lo que lo zanjaría: tasas de defectos de un usuario en producción. No he encontrado ninguna.**
- **«No existe» donde no puedo probarlo:** sobre el aislamiento del verificador digo **«no he encontrado evidencia de que exista»**, no «no existe». La búsqueda se hizo en el índice completo de la documentación y en el tracker; el código de los packs quedó sin revisar.
- **Un límite del artículo como fuente:** su autor declara que **no escribió Gas City** (*«I did not write Gas City. It was created by Julian Knutsen and Chris Sells»*) y a la vez afirma ser su defensor más firme (*«I am all-in with Gas City»*). Es a la vez parte interesada y fuente secundaria del producto de otros, lo que agrava el problema de citar sus claims como si fueran del proyecto.

## 7. Lo relevante que no encaja en lo anterior

- **Yegge pone su propio techo de seguridad:** *«I wouldn't use it in situations where you could physically hurt people, e.g. in medical or navigation systems. Not in 2026.»* Es el autor del concepto acotando su propio claim de fiabilidad.
- **El caso que el artículo usa para vender fiabilidad es el que menos la prueba.** Su ejemplo de dos agentes revisándose es moderar imágenes de jugadores de su juego, y él mismo lo justifica por ser de bajo riesgo: *«It's super low volume, low-stakes, not the end of the world if the agent crew messes up»*. Es un caso de uso real y honesto, pero no sostiene nada sobre calidad de software.
- **Una crítica al *claim* de fiabilidad que viene de la propia comunidad:** sobre el audit trail, un usuario responde a la frase de Yegge («cómo traigo la IA a mi empresa y paso una auditoría») con *«The important audit at my company is conducted by the FDA… "I told an AI-mayor in the form of a cartoon fox what to do" is not among the answers they want to hear»* ([HN 47770999](https://news.ycombinator.com/item?id=47770999)). Es el problema de fondo: un rastro que un tercero competente pueda reconstruir sin haber estado.
- **El único informe de uso en producción que he encontrado es positivo y de otra cosa:** *«i have a 512gb ram m3 ultra mac studio setup with a gas city that runs one of my companies»* ([HN 49532605](https://news.ycombinator.com/item?id=49532605)) — describe inferencia local corriendo junto a la «agent city», con la prueba de revisión *«caught 6 of 6 planted P1 defects, zero false positives»*. Es el único dato de calidad con denominador explícito (6 de 6 defectos plantados) que aparece, y **es de un modelo local, no de Gas City**: Gas City es el entorno donde corre, no la pieza evaluada.
- **Sigue sin existir un informe de Gas City (no Gas Town) en producción con métricas.** Lo más parecido son las tres fuentes de comunidad de §5, y ninguna mide calidad del software producido.
- **Tres artefactos de la organización que quedan como pistas abiertas, no como hallazgos:**
  - [`gastownhall/wasteland`](https://github.com/gastownhall/wasteland) — 89★, 29 forks, **sin push desde 2026-07-07**. *«Wasteland — federation protocol for Gas Towns»*. Es el reparto de trabajo entre fábricas distintas, y está a medio hacer.
  - [`gastownhall/gascity-otel`](https://github.com/gastownhall/gascity-otel) — 6★, **creado y abandonado el mismo día (2026-03-13)**. *«OpenTelemetry observability stack for Gas City — VictoriaMetrics + VictoriaLogs + Grafana»*. Es lo más parecido a una pieza de medición que alguien ha empezado.
  - [`gastownhall/gascity-packs`](https://github.com/gastownhall/gascity-packs) — 88★, **19 packs publicados**. Ahí viven `build-basic-review` y `fix-loop-base`, las dos fórmulas que materializan el patrón de §3.1. **Es el hueco declarado de esta investigación.**
- **El artículo es, medido, el mayor canal de entrada del proyecto que promociona.** Entre los referrers del snapshot del 2026-09-20, `steve-yegge.medium.com` aparece con **212 visitas / 73 únicas**, por detrás de Google (1.437) y github.com (955), y por delante de Bing (92) y chatgpt.com (80). No es una hipérbole del autor: la relación entre sus posts y la tracción del proyecto es medible — y eso mismo obliga a tratar sus claims sobre el producto como material promocional.

## Enlaces

- [[gas-city-instalacion-y-modelos]] — cómo se instala, qué packs se escogen, el reparto de modelos y qué hay que apagar
- [[gas-city-traje-a-medida]] — Gas City como montaje para desarrollar más rápido
- [[gas-city-con-2cerebro]] — cómo usarlo junto con el wiki para crear y desarrollar aplicaciones
- [[verificacion-externa-agentes]] — el principio que Gas City formula igual y no garantiza
- [[verificacion-sin-oraculo-informe]] — los mecanismos concretos para que el agente no toque los tests: qué está probado y qué solo propuesto
- [[metodo-de-investigacion]] — el método con el que se hizo este barrido
- [[linear-y-jev-frente-a-gas-city]] — si Linear + JEV (TypeSafe) mejoran a Gas City: no lo sustituyen, y dónde encaja JEV
