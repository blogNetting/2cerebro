---
title: Linear + JEV frente a Gas City — comparativa para desarrollo de software
created: 2026-10-04
updated: 2026-10-04
tags: [gas-city, linear, jev, typesafe, orquestacion, costes, investigacion]
zona: tecnico
---

¿Linear + «System One»/JEV mejora a Gas City para desarrollar software — más barato, más rápido, mejor? Informe de la investigación del 2026-10-04. Cada afirmación con fuente lleva su enlace y la cita textual que la respalda; lo que no la tiene va marcado.

## 1. Introducción: qué se pregunta y por qué

La pregunta de partida era si la combinación de **Linear** (gestor de issues con agentes) y **JEV** (el «System One Model» de TypeSafe) mejora a **Gas City** (el orquestador multiagente que ya tienes montado, ver [[gas-city-frente-a-la-fabrica]]) en eficiencia, coste, velocidad y calidad para desarrollo de software.

Antes de buscar, el criterio de admisión se fijó sobre el problema, no sobre los nombres: entra lo que (a) sirva de verdad para *producir software con agentes*, (b) tenga evidencia de uso real con cifras, y (c) se pueda comparar en las cuatro varas. Y se escribieron tres hipótesis rivales antes de evaluarlas:

- **H1** — Linear + JEV mejora a Gas City.
- **H2** — No son la misma categoría; no compiten.
- **H3** — Lo mejor no es ninguna de las dos.

**Conclusión adelantada, porque es lo que sostiene todo lo demás: H2. La pregunta, tal como está formulada, compara cosas de tres categorías distintas, y una de las tres —JEV— no puede escribir código en absoluto.** Eso no cierra el interés de la pregunta: JEV sí tiene un hueco real como *capa de decisión* barata dentro de un bucle de agentes como el que Gas City ejecuta, y ahí hay evidencia medida. Pero no puede «sustituir» a Gas City porque no hace lo que Gas City hace.

## 2. Considerado y descartado

| Qué | Motivo del descarte |
|---|---|
| **Sustituir Gas City entero por Linear + JEV** | Imposible por categoría: JEV **no genera texto ni código** (*«No text generation, no parsing»*, [docs.typesafe.ai](https://docs.typesafe.ai/)), y Linear es un gestor de issues en la nube, no un orquestador local de agentes. Ninguno de los dos ocupa el puesto del orquestador. |
| **Las cifras de cabecera de JEV** — «193,6x más rápido, 444,6x más barato» | Son de los *workflows propios* de TypeSafe y la propia empresa las etiqueta como cotas optimistas. Un test independiente sobre 78 casos encontró ventajas de velocidad de solo **2–3,6x** y divergencia en la precisión. El fabricante documentando su producto no es evidencia de comunidad ([typesafe.ai](https://typesafe.ai/), [crítica agregada](https://m.huxiu.com/article/4893173.html?type=text)). |
| **«Cero alucinaciones» como propiedad de calidad** | Es una garantía de *formato*, no de contenido: JEV puede elegir la opción equivocada. Cita de la comunidad: *«It can still forward a billing query to the dev department incorrectly»* ([HN 49717558](https://news.ycombinator.com/item?id=49717558), StevenWaterman). |
| **Que JEV sea una arquitectura nueva** | Consenso de la comunidad en el hilo de lanzamiento: es un **clasificador con cabezas de probabilidad calibradas**, no una técnica nueva — *«basically a zero-shot classifier that can accept raw text… as an input»* ([HN 49717558](https://news.ycombinator.com/item?id=49717558), petesergeant). El mérito real es el empaquetado, no la invención. |
| **El propio artículo de Yegge como fuente del montaje** | Ya descartado en [[gas-city-frente-a-la-fabrica]]; aquí se usa la plataforma, no el artículo. |

## 3. Análisis: qué es cada cosa y en qué es buena

### 3.1. Las tres son categorías distintas

| | **Gas City** | **Linear** | **JEV (TypeSafe)** |
|---|---|---|---|
| Qué es | Orquestador multiagente de código (self-hosted, MIT) | Gestor de issues + plataforma de agentes alojada (SaaS) | Modelo de decisión (clasificador con probabilidades calibradas) |
| ¿Escribe código? | **Sí**, vía agentes Claude Code que orquesta | Sí, vía «coding sessions» alojadas | **No** — decisión, no generación |
| Dónde corre | Tu máquina (tmux) | Nube de Linear | API de TypeSafe |
| Autoalojable | **Sí** | **No** (cloud-only; reportado por terceros) | **No** |
| Precio | Software gratis + tus tokens de modelo | Gratis / $10-usuario (Basic) / $16-usuario (Business), anual + créditos IA | $0,042/M tokens de entrada, salida gratis (early access) |
| Evidencia de calidad propia | Dashboard del propio proyecto: CI 45 %, 0 PRs de agente fusionados, 493 PRs abiertos | Informes de fallos de comunidad (UI-locked, fallos MCP, ediciones demasiado amplias) | Test independiente: rápido, pero 2–3,6x no 193x |

Fuentes de la tabla: Gas City → [[gas-city-frente-a-la-fabrica]] §3.5–3.7 (artefacto primario, `gh api` + dashboard del proyecto). Linear → [linear.app/pricing](https://linear.app/pricing), [linear.app/docs/ai-credits](https://linear.app/docs/ai-credits), [changelog 2026-08-20](https://linear.app/changelog/2026-08-20-coding-environments). JEV → [typesafe.ai](https://typesafe.ai/), [docs.typesafe.ai](https://docs.typesafe.ai/).

### 3.2. JEV: dónde encaja de verdad (que no es «escribir software»)

La documentación oficial es tajante sobre el alcance: *«Send state and typed questions; get structured answers your code can use directly»* y *«No text generation, no parsing»* ([docs.typesafe.ai](https://docs.typesafe.ai/)). Ofrece tres primitivas — **Choice**, **Score** y **Noul** (sí/no con probabilidad) — y devuelve distribuciones de probabilidad, no texto. Las preguntas deben ser atómicas: *«questions should be atomic — one specific, well-scoped thing»* ([docs.typesafe.ai](https://docs.typesafe.ai/)).

Eso limita su uso a **decidir dentro de un bucle**, y ahí la comunidad ya lo ha probado con números concretos sobre Claude Code (que es justo la pieza que Gas City orquesta):

| Herramienta de comunidad | Qué hace | Cifra medida por su autor | Enlace |
|---|---|---|---|
| `jev-effort` | Elige el esfuerzo de razonamiento por paso, **sin romper el prompt cache** | En un task: **$2,00 → $0,32** manteniendo el pass rate; 99,1 % de cache hits en una sesión real | [github.com/ifoster01/jev-effort](https://github.com/ifoster01/jev-effort) |
| `jev-use` | Manda a JEV los pasos que no necesitan texto | **p50 ~230 ms** y **~$0,02 por 1.000 juicios** | [github.com/shitianfang/jev-use](https://github.com/shitianfang/jev-use) |
| `jev-enforce` | Comprueba cada respuesta y edición contra `CLAUDE.md`/`AGENTS.md` | **~350 ms por chequeo**, ~3 céntimos/1.000, ~93 % precisión/recall a umbral 0,7 | [npmjs.com/package/jev-enforce](https://www.npmjs.com/package/jev-enforce) |

La propia TypeSafe reconoce la dificultad en código: su CEO, en el hilo de HN, *«the hard part for coding is actually state engineering (e.g. getting your dependencies in context)»*, y añade *«we haven't even tried it yet»* ([HN 49717558](https://news.ycombinator.com/item?id=49717558), CompleteSkeptic). Es decir: **el propio fabricante aún no demuestra JEV escribiendo software**, y la comunidad que lo ha integrado no reporta que ahorre tokens de forma concluyente — *«I have no real proof it saves me tokens, or is more accurate»* ([HN 49717558](https://news.ycombinator.com/item?id=49717558), ramon156).

**Dato de contraste importante para ti:** un test independiente situó la consistencia de clasificación de JEV **a la par de DeepSeek V4.1 Flash**, con ~0,32 s de latencia mediana ([cobertura agregada](https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/)). DeepSeek V4.1 Flash ya lo usas ([[orquestacion-modelos-y-costes]]), a $0,15–$0,30/M de entrada. Es decir: **la ventaja de JEV como capa de decisión es real pero no única** — su valor específico no es «más barato que lo que ya tienes», sino la *calibración de probabilidad* y las primitivas tipadas.

### 3.3. Linear: es un gestor de issues, no un orquestador — y su alternativa real es Beads

Linear se define como *«the product development system for teams and agents»* ([linear.app](https://linear.app/)), y su pieza de agente son las **coding sessions**: *«Linear Agent can now set up, run, and test your code before returning its work»* ([changelog 2026-08-20](https://linear.app/changelog/2026-08-20-coding-environments)). Es decir: Linear **planifica y aloja agentes**, no orquesta sesiones locales.

Por eso su comparación natural dentro de tu montaje **no es Gas City entero, sino Beads** — la pieza que en Gas City hace de almacén de trabajo ([[gas-city-frente-a-la-fabrica]] §3.3). Las diferencias que deciden:

- **Autoalojamiento:** Linear es cloud-only, sin opción on-prem (reportado por terceros, [utilo.io](https://utilo.io/blog/linear-review-2026-project-management)); Beads es local y git-backed. Para tu perfil esto no es neutro: manda el trabajo de tus proyectos a la nube de un tercero.
- **Coste recurrente:** $10–16 por usuario y mes, **solo anual**, más créditos IA aparte: tokens *«at the provider's published rates, with no markup»* **más** *«sandbox runtime: Charged at $0.25 per 20-minute block»* ([linear.app/docs/ai-credits](https://linear.app/docs/ai-credits)). Los créditos son un saldo prepago que **caduca a los 12 meses** y no se reembolsa ([linear.app/docs/ai-credits](https://linear.app/docs/ai-credits)).
- **Fiabilidad reportada por comunidad:** el agente *«only works inside Linear's web interface»* — sin terminal ni API externa (análisis de terceros, [frr.dev](https://www.frr.dev/posts/agentic-experience-data-driven-cli-design-llm/)); llamadas MCP que fallan *«intermittently… during long sessions»* y se leen como un rechazo del usuario que no ocurrió ([GitHub anthropics/claude-code #51674](https://github.com/anthropics/claude-code/issues/51674)); y agentes en la nube que editan *«~18 files when only 1–2 expected»* ([foro de Cursor](https://forum.cursor.com/t/linear-cursor-cloud-agent-sonnet-4-5-makes-overly-broad-edits-touches-18-files-when-only-1-2-expected/165633/4)). El propio ingeniero de Linear reconoce en su blog que en tareas mal acotadas el agente *«lands the correct fix about a third of the time»* ([linear.app/now](https://linear.app/now/linear-agent-bug-fix)).

### 3.3.bis. ¿Está Beads superado? Sí en fricción, no en lo que Linear ofrece

La objeción que más circula (recibida de un tercero el 2026-10-04) es que Beads *«estaba bien cuando no había otra cosa, pero ya está solucionadísimo»*, con Linear como reemplazo. Conviene separarlo, porque es verdad a medias.

- **Beads sí tiene fricción real:** el post *«Beads Is Dead. Long Live the Linear CLI»* documenta un daemon que falla con *«DATABASE MISMATCH DETECTED»*, errores de sincronización *«ghost sync»* y **seis pasos de fricción por sesión**; su autor lo cambia por **Claude Code Tasks** + **Linear CLI** ([frr.dev](https://www.frr.dev/posts/beads-is-dead/)). **Cautela:** ese post está traducido a PT/FR/ES — parecen varias fuentes y es una sola.
- **La categoría se ha poblado:** hay alternativas agent-native, algunas con migración desde Beads ya escrita — **kata** (`kata import --source-format beads`), **ticket**, **ergo**, **OpenTasks**; [Thoughtworks Radar](https://www.thoughtworks.com/pt-br/radar/tools/beads) reconoce la categoría y nombra a Beads pionero, no difunto.
- **Pero Linear no es el reemplazo de esa categoría.** Beads no es un tablero: es un **grafo de dependencias con *claims* atómicos para agentes concurrentes**, local y git-backed — de ahí el orden *«with no central scheduler»* por aristas `needs` ([[gas-city-frente-a-la-fabrica]] §3.3). Linear es un tracker **humano en la nube**: tiene bloqueos y MCP, pero no da lease/fencing, ni detección de «ready», ni seguridad de concurrencia multiagente entre sesiones.

**Conclusión de este apartado:** quien quiera *lo mejor para agentes*, el sustituto de Beads está en **kata/ergo/OpenTasks** (agent-native), no en Linear; Linear es mejor en el eje **humano/SaaS**. Y una consecuencia operativa verificada: **«portea todo» no lleva Gas City a Linear** — el import de Linear trae *issues*, no orquestación, y Gas City corre sobre Beads sin backend Linear (solo `dolt+bd` o `file`, [[gas-city-instalacion-y-modelos]] §3). Migrar el tracking a Linear **es salir de Gas City**, no configurarlo por dentro. Importa de Asana, Jira, Shortcut, GitHub, Trello, GitLab, Pivotal y otro workspace de Linear, más CSV ([linear.app/switch/migration-guide](https://linear.app/switch/migration-guide)).


### 3.4. La respuesta, punto por punto

| Tu pregunta | Respuesta |
|---|---|
| ¿Mejora a Gas City? | **No como sustituto** — categorías distintas y JEV no escribe código. Como *añadido*, JEV sí puede aportar en routing/guardarraíles, pero eso es sumar a tu montaje, no cambiarlo. |
| ¿Más barato? | **No para el código.** JEV abarata *una decisión* ($0,02/1.000 juicios), pero tú ya tienes DeepSeek a precio casi de saldo. Linear **añade** un coste recurrente por asiento + créditos que caducan. |
| ¿Más rápido? | JEV acelera **la decisión** (p50 ~230 ms), no el **desarrollo**. El cuello de botella del código es el agente y la verificación, no el clasificador. |
| ¿Mejor? | **Sin evidencia.** No existe ningún benchmark cabeza a cabeza Gas City vs Linear+JEV. Lo más cercano: Linear Agent con fallos de fiabilidad documentados y JEV sin demostrar aún uso en código por su propio fabricante. |

### 3.5. El mejor caso **contra** mi conclusión

Si Linear, con entornos gestionados y sin operación que mantener, diera a un desarrollador solo un camino más fiable que el Gas City autoalojado — cuyas métricas propias son malas (CI 45 %, 0 PRs de agente fusionados, [[gas-city-frente-a-la-fabrica]] §3.7) — la balanza podría inclinarse hacia **quitarse la operación de encima** aun pagando por asiento. **Lo que lo zanjaría: tasas de defectos por PR de un usuario real en producción, en cualquiera de los dos.** No he encontrado ninguna para ninguno de los dos. Sin eso, el caso no se sostiene más que el contrario.

## 4. Recomendaciones

Ordenadas por relación entre coste y lo que aportan.

1. **No cambiar Gas City por Linear + JEV.** No ocupan su puesto. Cualquier sustitución sería por otra cosa, no por esta combinación.
2. **Si algo de JEV vale la pena, es como capa de decisión añadida, no como motor.** Las tres herramientas de §3.2 se instalan *sobre Claude Code*, que es lo que Gas City orquesta: son complemento, no reemplazo. La más interesante es `jev-effort`, porque ataca un problema que este wiki ya tenía fichado — la pérdida de prompt cache al enrutar entre modelos ([[orquestacion-modelos-y-costes]] §4.2) — y su autor reporta que **no la rompe**.
3. **Antes de pagar JEV, medir.** Su ventaja como clasificador está **a la par de DeepSeek V4.1 Flash**, que ya tienes. Lo único que JEV aporta que DeepSeek no es *probabilidad calibrada nativa* + primitivas tipadas. Si tu caso no necesita eso explícitamente, no hay motivo para introducir un proveedor más.
4. **Linear solo si el objetivo es trabajar en equipo con un tablero SaaS asumido.** Para uso individual es coste recurrente por algo (tracking) que Beads ya te da local, más una nube de terceros a la que mandas el trabajo. Si algún día lo pruebas, que sea como *capa de planificación* junto a agentes locales, no como fuente de código.
5. **No adoptar nada por las cifras de cabecera de JEV.** 193x/444x es marketing auto-reportado; las mediciones independientes dan 2–3,6x.

## 5. Dónde se ha buscado

| Fuente | Tipo | Resultado |
|---|---|---|
| [typesafe.ai](https://typesafe.ai/) y [docs.typesafe.ai](https://docs.typesafe.ai/) | Artefacto primario (fabricante) | **Lo que decide.** De aquí sale que JEV no genera texto ni código y las tres primitivas |
| [Blog de anuncio de JEV](https://typesafe.ai/blog/introducing-system-one-models-and-jev) | Artefacto primario (fabricante) | Cifras de cabecera y avisos de sesgo del propio autor |
| [HN, hilo de lanzamiento 49717558](https://news.ycombinator.com/item?id=49717558) (1.989 pts, 520 comentarios) | Comunidad independiente | **Lo más útil después de la doc.** Novedad, límites, uso en código |
| [antirez (Salvatore Sanfilippo)](https://www.infoq.cn/article/POjWf9P5wCYjQaB39jD6) — 116K vistas | Comunidad independiente | La crítica más citada: *«Jev might have some very narrow use cases, but the hype around it shows that most people… can't tell what's important»* |
| [TechCrunch](https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/) | Prensa técnica | Test independiente: paridad con DeepSeek V4.1 Flash en clasificación |
| [linear.app/pricing](https://linear.app/pricing), [docs/ai-credits](https://linear.app/docs/ai-credits), [changelog](https://linear.app/changelog/2026-08-20-coding-environments) | Artefacto primario (fabricante) | Precios, créditos, qué factura cada sesión |
| [HN Algolia — «Linear Agent»](https://hn.algolia.com/api/v1/search?query=Linear%20Agent&tags=story) | Comunidad | Los posts de Linear Agent tienen **2–9 puntos**: sin validación de comunidad fuerte |
| [frr.dev](https://www.frr.dev/posts/agentic-experience-data-driven-cli-design-llm/), [GitHub #51674](https://github.com/anthropics/claude-code/issues/51674), [foro Cursor](https://forum.cursor.com/t/linear-cursor-cloud-agent-sonnet-4-5-makes-overly-broad-edits-touches-18-files-when-only-1-2-expected/165633/4) | Comunidad | Fallos de fiabilidad del agente de Linear |
| [github.com/ifoster01/jev-effort](https://github.com/ifoster01/jev-effort), [shitianfang/jev-use](https://github.com/shitianfang/jev-use), [npm jev-enforce](https://www.npmjs.com/package/jev-enforce) | Herramientas reales de comunidad | Único uso de JEV con números de ahorro |
| [[gas-city-frente-a-la-fabrica]] y [[orquestacion-modelos-y-costes]] | Investigación previa del wiki (artefacto primario) | Base de Gas City y del reparto de modelos |
| Búsqueda «Linear + Jev replace orchestrator» | — | **No aportó nada.** Refuerza que nadie combina las dos cosas con ese fin |

**Cobertura:** de las fuentes que existen, miré las primarias de los dos fabricantes, el hilo grande de comunidad de JEV y evidencia de fallos de Linear. **No miré** los foros internos de Linear, ni he podido ejecutar ninguno de los dos (JEV está en early access). Lo que queda sin mirar es, otra vez, donde estaría la prueba de calidad en producción.

## 6. Verificación y límites

- **Qué se comprobó:** las citas de la documentación de TypeSafe y de Linear se tomaron de las páginas primarias en vivo; las de comunidad, del hilo de HN y de los repos, con su enlace. Las cifras de Gas City vienen del artefacto primario ya verificado en [[gas-city-frente-a-la-fabrica]].
- **Qué es hecho, qué es supuesto y qué es juicio.** Hechos con URL y cita: §3.1–3.3. Supuesto sin fuente: que Linear siga siendo cloud-only (reportado por terceros, no verificado en doc de Linear). Juicio mío: que JEV valga solo como capa añadida (§4).
- **«No existe» donde no puedo probarlo:** sobre un benchmark Gas City vs Linear+JEV digo **«no he encontrado ninguno»**, no «no existe».
- **Un porcentaje sin denominador, evitado:** las cifras de mejora de JEV (193x/444x) y de comunidad (2–3,6x) se citan con su origen (medición propia vs test independiente sobre 78 casos), no como un número suelto.

## 7. Lo relevante que no encaja

- **El propio fabricante de JEV duda de su uso en código:** *«we haven't even tried it yet»* es la confesión más útil del hilo, y desmonta cualquier plan de sustituir un orquestador de código por JEV hoy.
- **La pregunta correcta no es «Linear o Gas City», sino «¿quiero planificación SaaS y alojada, o local y autoalojada?».** Es la decisión de fondo que arrastra todas las demás.
- **`jev-effort` merece una prueba aislada** si en algún momento mides coste por tarea en tu montaje: es la única herramienta que reporta ahorro tangible (~6x en un task) resolviendo a la vez el problema de cache que este wiki ya tenía identificado.

## Enlaces

- [[gas-city-frente-a-la-fabrica]] — los mecanismos y las métricas reales de Gas City
- [[orquestacion-modelos-y-costes]] — precios por modelo y el problema de cache al enrutar
- [[gas-city-instalacion-y-modelos]] — el reparto Opus/DeepSeek ya aplicado en tu montaje
- [[verificacion-externa-agentes]] — por qué un verificador barato solo cuenta si está fuera del alcance del agente
- [[_index]]
