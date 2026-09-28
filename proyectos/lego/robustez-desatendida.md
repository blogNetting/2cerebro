---
title: Lego — qué se rompe cuando no hay nadie delante
created: 2026-09-28
updated: 2026-09-28
tags: [lego, agentes, robustez, ambiguedad]
zona: tecnico
---

Cuatro cosas se rompen de forma documentada y repetida cuando un agente trabaja sin nadie mirando: los reintentos se descontrolan, las tareas se cuelgan sin avisar, varios agentes se pisan, y —la más importante para Lego— **ante una tarea incompleta el agente no pregunta: adivina**.

## 1. El hallazgo central: la tarea tiene que estar completa, porque preguntar no funciona

> «La infrasepecificación no hace principalmente que los agentes fallen; hace que **adivinen**.» — [UnderSpecBench, arXiv:2607.02294](https://arxiv.org/abs/2607.02294)

Ese paper mide cuántas veces un agente se sale de los límites cuando la instrucción está incompleta: **55,8% a 67,8% de las ejecuciones violan al menos un límite**, sobre **2.208 variantes de prompt** y **69 familias de tareas**, en cinco configuraciones de agente y modelo.

Y hay un segundo paper que mide lo contrario — si el agente sabe **cuándo preguntar**. El resultado es demoledor ([HiL-Bench, arXiv:2604.09408](https://huggingface.co/papers/2604.09408), **300 tareas** y **1.131 bloqueos**):

| Modelo | Con información completa | Con opción de preguntar | Caída |
|---|---|---|---|
| Gemini 3.1 Pro | 84,7% | **5,3%** | −79 puntos |
| Claude Opus 4.6 | 69,1% | **9,4%** | −60 puntos |
| GPT-5.3-Codex | 67,3% | **2,0%** | −65 puntos |
| GPT-5.4 | 67,3% | **1,3%** | −66 puntos |

> ### ⚠️ CORREGIDO EL 2026-09-28 — me quedé con la mitad del resumen que me convenía
>
> El mismo resumen de HiL-Bench continúa así: «el entrenamiento por refuerzo con una recompensa Ask-F1 moldeada muestra que **el juicio es entrenable**: un modelo de 32B mejora tanto la calidad de la petición de ayuda como la tasa de éxito». **Esa mitad no aparecía en el informe.**
>
> Y hay tres matices del propio paper que faltaban: la herramienta de preguntar **sí devuelve la respuesta** cuando la pregunta apunta al bloqueador correcto; cada tarea lleva **3 a 5 bloqueadores** y la métrica es **pass@3**, así que el hundimiento es en buena parte **probabilidad compuesta**, no «preguntar resta»; y su conclusión literal es que estas herramientas «**no mejoran el rendimiento de forma uniforme**».
>
> **Y preguntar sí funciona, con datos:** **Ambig-SWE** (ICLR 2026) da *«hasta un **74%** sobre las configuraciones no interactivas»* — la cifra que yo había descartado como no verificada **era real** y me equivoqué al descartarla. «Ask or Assume?» da **69,40% frente a 61,60%**, y ClarifyGPT (FSE 2024) sube Pass@1 del **70,96% al 80,80%**.
>
> **La contrapartida honesta:** el coste se multiplica por **2,1**, y un baseline simple con 1,02 preguntas por tarea bate a un andamiaje elaborado con 3,06. Y los agentes **sí preguntan** más de lo que yo decía: la disposición a hacerlo va del **31,8% al 44,5%** según el andamiaje.
>
> **Lo que queda en pie de esta sección:** el agente no pregunta por defecto, y habilitarlo sin criterio no basta. **Lo que se retira:** que preguntar «hunda el rendimiento» y que «no tenga arreglo». Detalle en [[contra-evidencia]].

Sus palabras, con el recorte que faltaba: «ningún modelo frontera recupera más que una fracción de su rendimiento con información completa cuando tiene que decidir si preguntar». El reparto de los bloqueos: **42%** información ausente, **36%** requisitos ambiguos, **22%** instrucciones contradictorias.

**Y preguntar, cuando no hay canal, es un no-op que además simula éxito.** El caso está en un pull request real, titulado sin rodeos «dejar de preguntar cosas que nadie puede responder» ([Archon #2257](https://github.com/coleam00/Archon/pull/2257)): el agente estaba instruido para parar y preguntar ante una ambigüedad, pero **un nodo no tiene canal con un humano a mitad de ejecución**, así que la pregunta se convertía en la salida del nodo, el nodo reportaba `completed`, y el flujo avanzaba **sin haber implementado nada**. Se reprodujo **tres veces**. La solución que adoptaron: el agente decide, y **registra la decisión en una sección de «desviaciones»** del informe — y sólo si falta un artefacto de entrada, escribe `BLOCKED` y termina sin tocar el código.

**Y la evidencia más caro de todas:** el informe de [Answer.AI sobre Devin](https://www.answer.ai/posts/2025-01-08-devin.html), laboratorio independiente, con denominador — **de 20 tareas, 14 fallos, 3 éxitos y 3 no concluyentes**. Su descripción del fallo, textual: «Devin pasaba días persiguiendo soluciones imposibles en vez de reconocer bloqueos fundamentales». Ejemplo concreto: le pidieron desplegar varias aplicaciones en un único despliegue de Railway, «algo que Railway no soporta», y en vez de identificar la limitación «pasó más de un día probando enfoques distintos y alucinando funciones que no existían».

**Consecuencia directa para [[crear-la-tarea]]:** la tarea **tiene que ser autosuficiente**. Todo lo que el agente necesite saber va escrito dentro, porque no va a preguntar, y si le das la opción de preguntar rinde una fracción de lo que rendiría. Y las tres categorías de bloqueo dan la lista de lo que hay que cerrar: información ausente, requisitos ambiguos, instrucciones contradictorias — que es exactamente el «dos requisitos contradictorios en las secciones 4 y 7» del que hablaba un desarrollador en [[lo-que-dice-la-comunidad]].

**El intento de arreglarlo por producto ha fallado hasta ahora.** Cursor 2.4 añadió preguntas de aclaración no bloqueantes. En su propio foro, usuarios reportan lo contrario: el agente «simplemente sigue haciendo cambios sin responder ni aclarar», y la explicación que dan es que el modelo interpreta las preguntas como **retóricas** y las trata como instrucciones implícitas ([foro de Cursor](https://forum.cursor.com/t/cursor-2-4-clarification-questions-from-the-agent/149406)).

## 2. Reintentos: el tope es pequeño, y la señal es la falta de progreso

**Cuatro implementaciones independientes, en repos distintos, coinciden en cortar a los 2–3 intentos y escalar a un humano:**

| Sistema | Tope | Qué hace al llegar |
|---|---|---|
| [issue-orchestrator](https://github.com/issue-orchestrator/issue-orchestrator/blob/main/src/issue_orchestrator/entrypoints/cli_tools/dirty_retry_budget.py) | 2 rechazos seguidos | Escribe registro `needs_human` y sale |
| [vibeflow](https://github.com/mizkun/vibeflow/commit/9e49f04c4d4844504f383243171bd2d0619f60be) | 3 intentos, cambia de agente a los 2 | `"action": "escalate"`, `needs_human` |
| [claude-code-toolkit](https://github.com/dagonet/claude-code-toolkit/blob/main/templates/general/AGENT_TEAM.md) | 3 ciclos | Notifica al usuario con lo intentado |
| [Microsoft Agent Framework](https://learn.microsoft.com/en-us/python/api/agent-framework-core/agent_framework.magenticbuilder) | `max_stall_count=3` | Suspende y pide intervención |

El razonamiento del primero está escrito en el propio código, y explica el porqué mejor que cualquier documento: dos intentos, para dar «exactamente un ciclo de “inténtalo otra vez” tras el primer fallo: margen de sobra para un “se me olvidó el `git add`”, y ningún margen para que un agente queme en silencio el tiempo máximo de la sesión».

**La señal para escalar no es el número de fallos, es la falta de progreso.** OpenHands trae un detector de atascos activado por defecto que cubre cinco patrones ([documentación](https://docs.openhands.dev/sdk/guides/agent-stuck-detector)): misma acción y misma observación **4+ veces**; misma acción con error **3+ veces**; monólogo sin avance **3+**; patrones alternos **6+ ciclos**; y errores repetidos de ventana de contexto. Compara por significado, no por identidad de evento — por eso detecta el bucle aunque el texto cambie.

**Y un caso documentado de reintento sin tope**, que conviene conocer aunque la fuente tenga trampa: el [issue #42055](https://github.com/anthropics/claude-code/issues/42055) describe un bucle de reintento sin límite superior en el auto-compactado de contexto, donde «el código incrementa un contador de fallos consecutivos pero nunca llega a comprobarlo». **Aviso:** ese issue está firmado «— A Friend», usa hashes de relleno, está fechado el 1 de abril y se cerró como no planificado — es una **propuesta satírica**, no un cambio real. Las cifras de «1.279 sesiones» y «250.000 llamadas diarias» que circulan por blogs **no aparecen en la fuente primaria**: quedan **sin verificar**.

## 3. Colgado frente a muerto: el vigilante por reloj falla en las dos direcciones

**Falso positivo — mata tareas sanas.** Claude Code tiene un vigilante de inactividad para subagentes, por defecto 600 segundos. El [issue #85265](https://github.com/anthropics/claude-code/issues/85265) lo describe con la frase más útil del informe:

> «El problema es que 600 s está dentro de la distribución normal de latencia para los niveles de razonamiento pesado... **No está detectando un flujo muerto; está guillotinando uno lento pero sano.**»

Y da tasas **con denominador**, medidas sobre el historial de transcripciones de una máquina: **9 de 325 ≈ 2,8%** en conjunto, y por modelo: fable **5 de 49 (10,2%)**, opus **4 de 158 (2,5%)**, sonnet **0 de 114**, haiku **0 de 4**. Todos los cortes a exactamente 600,0 s tras el resultado de una herramienta, y «el trabajo era viable: reanudar la tarea matada por identificador la completa con normalidad».

**Falso negativo — no se dispara cuando debería.** El [issue #50802](https://github.com/anthropics/claude-code/issues/50802) reporta el caso contrario: el vigilante no se dispara y el subagente muere con «Stream idle timeout - partial response received», sin reintento. *(Reporta «50–60% de fallo», pero **sin denominador**: no utilizable como tasa.)*

**El fallo de fondo es el mismo en los dos:** medir **tiempo** en vez de **progreso**.

**Y el colgado silencioso es real.** Del informe de la noche desatendida ([#53610](https://github.com/anthropics/claude-code/issues/53610), ya citado en [[consumir-la-tarea]]): «si el agente deja de escribir latidos, nadie se entera»; varios agentes estuvieron callados horas teniendo la cadencia escrita en su definición; y de los 5 agentes a los que se les pidió explícitamente un latido, **4 lo hicieron y uno nunca, en toda la noche**.

## 4. Concurrencia: el worktree aísla ficheros, no estado

`git worktree` da a cada agente su directorio y su rama compartiendo el mismo `.git`. **Eso no es una frontera suficiente**, y hay dos issues independientes con incidentes contados:

- [#83311](https://github.com/anthropics/claude-code/issues/83311) — el aislamiento **se aplica de forma inconsistente** dentro de un mismo lote: «**2 de 5 agentes** obtuvieron worktrees aislados correctos; los otros 3 mostraron contaminación». Un commit de un agente acabó **en la rama de otro**, contaminando su diff, y otros crearon entradas de `git stash` **en el repositorio principal**.
- [#76250](https://github.com/anthropics/claude-code/issues/76250) — el directorio de trabajo se comporta como «un único hueco mutable compartido por todo el árbol de sesión»: **~43 incidentes registrados** en una semana de uso intensivo. Casos concretos: el `git merge origin/main` del orquestador ejecutándose **dentro del worktree de un subagente vivo, sobre sus cambios sin confirmar**; y un `npm ci` corriendo dentro del worktree de otro subagente mientras su comprobación de tipos estaba en vuelo.
- [#76377](https://github.com/anthropics/claude-code/issues/76377) — fuga: «una ejecución paralela matada filtró **9 worktrees** de aislamiento a la vez, todos bloqueados por el mismo orquestador muerto».

**Y el disenso, que hay que recoger:** el hilo de Hacker News que popularizó esto ([HN 49110389](https://news.ycombinator.com/item?id=49110389), 32 puntos, 35 comentarios) no converge en la conclusión fuerte. Varios comentaristas sostienen que worktree **nunca pretendió ser** una frontera de seguridad, y que la gente lo usa para trabajo paralelo. Lo que **sí** converge es el hecho técnico: worktree aísla ficheros, no estado. Y hay una objeción sin rebatir que importa: el `.git` es escribible desde el worktree, y ahí viven los ganchos — un agente puede plantar uno que se ejecute en el commit.

**Lo que el worktree no toca:** `node_modules` (cada worktree necesita su propia instalación), los puertos, la base de datos, y los conflictos semánticos — «merge limpio, arquitectura rota»: dos agentes implementan la misma responsabilidad de forma distinta, ambos pasan tests, el merge sale limpio, y quedan dos dueños para un mismo comportamiento.

## 5. Convergencia

| Afirmación | Cuántas fuentes independientes convergen |
|---|---|
| Los agentes **adivinan** en vez de preguntar, y preguntar sin canal es un no-op | **Cinco, de cuatro tipos distintos**: dos papers (HiL-Bench, UnderSpecBench), un pull request de ingeniería (Archon #2257), un informe de laboratorio (Answer.AI/Devin) y un issue de operación real (#53610) |
| El tope de reintentos es **2–3** y luego escala a un humano | **Cuatro implementaciones** en repos distintos |
| La señal de escalada es la **falta de progreso**, no el número de fallos | Tres: vibeflow, OpenHands, guía de Anthropic |
| Los vigilantes **por tiempo** fallan en las dos direcciones | Dos issues independientes del mismo proyecto (#85265 y #50802) más el fallo de estado terminal de OpenHands (#1510) |
| El worktree aísla **ficheros, no estado** | Dos issues independientes con incidentes contados, más el hilo de HN — **con disenso explícito sobre la conclusión fuerte** |

## 6. Recomendaciones

1. **La tarea, autosuficiente y sin contradicciones.** Es la conclusión que más pesa: no hay red de seguridad del otro lado. Si falta algo, el agente adivina.
2. **Ante ambigüedad: decidir y documentar, o bloquear ruidosamente. Nunca «preguntar».** El patrón de Archon: el agente decide y lo registra en una sección de desviaciones; y sólo si falta un artefacto de entrada, escribe `BLOCKED` y **termina sin tocar el código**.
3. **Tope de reintentos pequeño y contador por tarea**, con reinicio a cero en cuanto una pasada tiene éxito.
4. **Escalar por patrón repetido, no por reloj.** Misma acción y mismo resultado 3–4 veces: parar y escribir `needs_human`.
5. **Nunca confiar en el directorio de trabajo ambiental.** Rutas absolutas y `git -C <ruta>` en cada comando.
6. **Un worktree por agente es condición necesaria, no suficiente:** hay que añadir asignación disjunta de ficheros, fusión serializada y aislamiento aparte de puertos, dependencias y base de datos.
7. **Latido propio más vigilante externo**, y por patrón además de por tiempo.

## 7. Dónde se ha buscado, y qué queda abierto

**Primarias:** issues de `anthropics/claude-code` ([#85265](https://github.com/anthropics/claude-code/issues/85265), [#83311](https://github.com/anthropics/claude-code/issues/83311), [#76250](https://github.com/anthropics/claude-code/issues/76250), [#53610](https://github.com/anthropics/claude-code/issues/53610), [#50802](https://github.com/anthropics/claude-code/issues/50802), [#76377](https://github.com/anthropics/claude-code/issues/76377), [#42055](https://github.com/anthropics/claude-code/issues/42055)), [Archon #2257](https://github.com/coleam00/Archon/pull/2257), [OpenHands #1510](https://github.com/OpenHands/software-agent-sdk/issues/1510), repos de [issue-orchestrator](https://github.com/issue-orchestrator/issue-orchestrator) y [vibeflow](https://github.com/mizkun/vibeflow). **Papers:** [HiL-Bench](https://huggingface.co/papers/2604.09408), [UnderSpecBench](https://arxiv.org/abs/2607.02294). **Comunidad:** [HN 49110389](https://news.ycombinator.com/item?id=49110389), [Answer.AI sobre Devin](https://www.answer.ai/posts/2025-01-08-devin.html), [foro de Cursor](https://forum.cursor.com/t/cursor-2-4-clarification-questions-from-the-agent/149406).

**Buscado y sin aportar nada:** `site:reddit.com` devuelve vacío desde esta vía; los libros blancos corporativos de gobernanza de agentes, genéricos y sin incidentes citables; la documentación de Devin, que no describe mecanismo de aclaración alguno.

**Sin comprobar:** las cifras del «circuit breaker» satírico (1.279 sesiones, 250.000 llamadas diarias); si ese cambio llegó a fusionarse; la tasa de fallo «50–60%» del issue #50802 (sin denominador); y **no existe ningún estudio que mida cuántos reintentos hacen falta de verdad** — todos los topes son decisiones de diseño en código, no resultados medidos.

## Enlaces

- [[crear-la-tarea]] — el formato, ahora con la exigencia de autosuficiencia
- [[consumir-la-tarea]] — el bucle y el disparador
- [[verificacion-y-oraculo]] — el oráculo
- [[lo-que-dice-la-comunidad]] — el veredicto de los que lo usan
- [[investigacion-lego]] — el informe completo
