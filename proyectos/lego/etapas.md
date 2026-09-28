---
title: Lego — las etapas: qué está maduro y qué necesita tu mano
created: 2026-09-28
updated: 2026-09-28
tags: [lego, etapas, veredicto, datos]
zona: tecnico
---

El proceso descompuesto en seis etapas. En cada una: **qué se puede automatizar hoy con piezas que existen y están publicadas**, y **qué va a requerir intervención humana**. Todo con el dato y su enlace. Las etiquetas son **DATO** (medido o documentado, con enlace) o **LECTURA** (mía, marcada).

## Resumen en una tabla

| Etapa | ¿Automatizable hoy? | Lo que lo impide |
|---|---|---|
| **1. Escribir la tarea** | **Parcial** | El formato está resuelto. **Decidir qué construir no** |
| **2. Encolar y disparar** | **Sí, con reservas** | El planificador no está garantizado y falla en silencio |
| **3. Ejecutar el bucle** | **Parcial** | Funciona por tandas cortas. La noche entera, no |
| **4. Verificar** | **Parcial** | Hay oráculos buenos para código, **ninguno rápido para mantenibilidad** |
| **5. Aislar** | **Sí, pero hay que montarlo** | El aislamiento por worktree **no cubre el estado** |
| **6. Cerrar** | **No** | La autonomía real medida es del **4,7%** |

---

## Etapa 1 — Escribir la tarea

### Lo maduro

**El formato está resuelto y publicado.** No hay que inventarlo:

- **Los criterios de aceptación en forma fija** (CUANDO/MIENTRAS/SI… ENTONCES… DEBE) vienen de **EARS**, de Alistair Mavin en Rolls-Royce (IEEE RE 2009), y lo usa Kiro. [Guía práctica](https://www.modernrequirements.com/blogs/ears-notation-the-practical-guide/).
- **Los campos exactos**, sacados de un proyecto real: [`quotabar/specs/GH55/tasks.md`](https://github.com/majiayu000/quotabar/blob/main/specs/GH55/tasks.md) — con **`Owner`** (el agente), **`Covers`** (qué criterios cubre), **`Done when`** (cuándo está terminada) y **`Verify`** (cómo se comprueba), separados.
- **Plantillas modificadas por empresas**: [dotnet-exec](https://github.com/WeihanLi/dotnet-exec/blob/main/.specify/templates/tasks-template.md), [nutanix](https://github.com/nutanix-cloud-native/cluster-api-runtime-extensions-nutanix/tree/main/.specify/templates).

**DATO — y la ambigüedad decide el resultado, medido:** con la verificación idéntica y **sólo el texto cambiado**, GPT-4 pasa de **73,8% a 6,7%** de acierto cuando el enunciado se vuelve contradictorio; y el código **ejecutable pero incorrecto** pasa del **24% al 89%** ([arXiv:2507.20439](https://arxiv.org/abs/2507.20439) — *«ICSE 2026» no verificado*).

**DATO — el reparto de lo que falta:** de los bloqueos de tarea, **42% información ausente, 36% requisitos ambiguos, 22% instrucciones contradictorias** ([HiL-Bench](https://huggingface.co/papers/2604.09408), 300 tareas y 1.131 bloqueos).

**DATO — el retorno de hacerlo bien:** *«aproximadamente **una hora por adelantado reduce una revisión de 6 horas a 20 minutos**»* ([HumanLayer](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md)).

### Lo que necesita tu mano

**Decidir qué construir.** No hay herramienta que lo haga:

- **DATO:** la guía oficial de Claude Code lo pone en su lista de cuándo **no** usar el agente: *«decisiones de arquitectura críticas — **el agente acelera, no decide**»*.
- **DATO:** *«cerca del **40% de las tareas son de un solo intento**»* ([HumanLayer](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md)) — **el 60% restante necesita que alguien decida antes**.
- **DATO:** el caso real que contó un desarrollador — una especificación con **dos requisitos contradictorios en las secciones 4 y 7**. *«El agente estaba haciendo exactamente lo que se le dijo — es que le dijeron dos cosas distintas»* ([[lo-que-dice-la-comunidad]]).

**LECTURA:** esta es la etapa donde tu mano vale más. **Una hora aquí ahorra cinco allí, y es el único sitio donde el dato dice que el retorno es de orden de magnitud.**

---

## Etapa 2 — Encolar y disparar

### Lo maduro

**Existe y está implementado:**

- **El repositorio como cola.** Implementado en [`ralph-wiggum`](https://github.com/fstandhartinger/ralph-wiggum) con `spec_queue.sh`; en [`beads`](https://github.com/gastownhall/beads) como **grafo de dependencias**; en [`ccpm`](https://github.com/automazeio/ccpm) como **Issues de GitHub + worktrees**.
- **El modo sin interfaz**, documentado: `claude -p` procesa, ejecuta y sale, **sin leer nunca de la entrada estándar y sin preguntar permisos** ([documentación oficial](https://code.claude.com/docs/en/headless)).
- **GitHub Actions** con la acción oficial, que en modo automatización **no espera a que nadie le mencione** ([documentación](https://code.claude.com/docs/en/github-actions)).
- **`cron`**, que ya tienes.

### Lo que no funciona bien, y está medido

**DATO — el planificador de GitHub no está garantizado:** *«en repositorios públicos, desactiva la programación **tras 60 días sin actividad**»*, y los flujos programados son «mejor esfuerzo», con retrasos de **10 a 30 minutos** en horas punta ([documentación](https://code.claude.com/docs/en/github-actions)).

**DATO — y falla en silencio:** un fallo de cron **no deja registro ni notificación**. No hay ejecución fallida: hay silencio, que es **indistinguible del éxito** ([[consumir-la-tarea]]).

**DATO — medido en un caso real:** el cron de sesión **se saltó 7 disparos consecutivos sin error**, y el primer disparo llegó **3,5 horas tarde** ([issue #53610](https://github.com/anthropics/claude-code/issues/53610)).

**DATO — otra trampa:** *«GitHub no dispara flujos en commits hechos con el `GITHUB_TOKEN` por defecto»* — es una medida anti-bucle, y **rompe el auto-fusionado en silencio** ([documentación](https://code.claude.com/docs/en/github-actions)).

### Lo que necesita tu mano

**DATO:** en el caso real, el operador tuvo que **añadir 40 entradas de permisos a mano** a la mañana siguiente, y el flujo se quedó **tres horas parado** por un fichero de estado obsoleto que nadie limpió ([issue #53610](https://github.com/anthropics/claude-code/issues/53610)).

**LECTURA, marcada:** esta etapa es automatizable, pero **el vigilante hay que ponerlo tú**. Un planificador que falla en silencio necesita que alguien mire que sigue vivo.

---

## Etapa 3 — Ejecutar el bucle

### Lo maduro

**El patrón central está resuelto, y es contraintuitivo: contexto limpio por tarea.**

- **DATO — la medición:** **18 modelos** de cuatro fabricantes, **8 longitudes de entrada** y 11 posiciones por configuración: *«los modelos no usan su contexto de forma uniforme; su rendimiento se vuelve cada vez menos fiable **a medida que crece la entrada**»*. Y con **prompts enfocados de ~300 tokens frente a historiales de ~113.000, todos rinden mejor** ([Chroma](https://www.trychroma.com/research/context-rot)).
- **DATO — la convergencia:** la comunidad llegó al mismo diseño sin leer la medición: *«cada tarea debe completarse **en una sesión nueva**. Así no hay contaminación de contexto»* ([[lo-que-dice-la-comunidad]]), y **GSD** lo industrializa con **contexto fresco de 200.000 tokens por tarea** ([[panorama-de-herramientas]]).
- **DATO — que funciona:** el único caso independiente con número duro da **2,09×** de rendimiento sobre **802 desarrolladores y 196.212 PRs**, medido por académicos con la empresa excluida del análisis ([arXiv:2607.01904](https://arxiv.org/abs/2607.01904)). **Con sus matices**: la carga por revisor **se duplicó** y el efecto **se desvaneció en el monolito heredado**.
- **DATO — las piezas de seguridad existen:** [`ralph-wiggum`](https://github.com/fstandhartinger/ralph-wiggum) trae **cortacircuitos, contador de reintentos, analizador de respuesta y notificación**. No hay que inventarlas.

### Lo que no funciona bien

**DATO — la operación desatendida sostenida falla, documentada paso a paso:** nueve fallos distintos en una noche real, con el operador despierto a las **03:30 y 04:30**. Su resultado: *«de 12 horas esperadas de progreso desatendido, **menos de la mitad**, a un coste extremo para el usuario»* ([issue #53610](https://github.com/anthropics/claude-code/issues/53610)).

**DATO — los vigilantes por tiempo matan tareas sanas:** **9 de 325 ejecuciones (2,8%)** se cortaron estando vivas; por modelo, **5 de 49 (10,2%)** en uno. La frase del informe: *«no está detectando un flujo muerto; está **guillotinando uno lento pero sano**»* ([issue #85265](https://github.com/anthropics/claude-code/issues/85265)).

**DATO — la contaminación entre agentes es real:** **2 de 5 agentes** acabaron operando sobre el repositorio principal, y un commit de uno terminó **en la rama de otro** ([issue #83311](https://github.com/anthropics/claude-code/issues/83311)); **~43 incidentes** registrados en una semana ([issue #76250](https://github.com/anthropics/claude-code/issues/76250)).

**DATO — y el techo lo admite su creador:** Geoffrey Huntley, sobre su propio invento: *«te despertarás con **una base de código rota que no compila** de vez en cuando»*; *«**no usaría Ralph en una base de código existente ni de broma**»*; *«quien venda que una herramienta hace el 100% del trabajo sin un ingeniero **está vendiendo humo**»* ([ghuntley.com/ralph](https://ghuntley.com/ralph/)).

**DATO — y lo más importante: nadie publica la tasa de éxito.** Ningún montaje encontrado dice cuántas tareas salieron bien de cuántas. Lo más cercano es el «llegarás al 90%» de Huntley, **que es una expectativa, no una medición**.

### Lo que necesita tu mano

**LECTURA, marcada:** hoy la vigilancia nocturna es humana. El dato que lo sostiene no es una opinión: **es que el cron se salta disparos sin avisar, los vigilantes matan lo sano y los agentes se contaminan entre sí** — y las tres cosas están documentadas con incidentes contados.

---

## Etapa 4 — Verificar

### Lo maduro

**Aquí sí hay piezas que funcionan de verdad, y con medición:**

- **DATO — verificación formal con cero falsos positivos:** **CLOVER** *«mantiene tolerancia cero con las incorrectas adversariales — **ningún falso positivo**»*, con **0 de 60** en cada una de las cuatro familias adversariales, [publicado en LNCS 2024](https://arxiv.org/abs/2310.17807) (Stanford + VMware Research).
- **DATO — y genera pruebas, no sólo las comprueba:** **AutoVerus** prueba correctamente **137 de 150** tareas, frente a 67 del GPT-4o directo, [OOPSLA 2025](https://arxiv.org/abs/2409.13082) (Microsoft Research + UIUC).
- **DATO — y aceptación industrial:** **TestGen-LLM** de Meta, con el **73% de sus recomendaciones aceptadas para producción** ([FSE 2024](https://arxiv.org/abs/2402.09171)).
- **DATO — y para el caso web:** [exspec](https://github.com/mnapoli/exspec) ejecuta especificaciones en texto plano **en un navegador real**, sin código pegamento.

### Lo que no funciona bien

**DATO — los tests aprueban lo roto:** **31,08%** de parches aprobados lo son con tests débiles, y **32,67%** con la solución filtrada en el enunciado; al descontarlos, la resolución cae del **12,47% al 3,97%** ([SWE-bench+, arXiv:2410.06992](https://arxiv.org/abs/2410.06992)).

**DATO — pero el error va en las DOS direcciones, y esto corrige mi informe:** **35,5%** de las tareas auditadas tienen tests **demasiado estrictos** que *«invalidan muchas entregas funcionalmente correctas»*, bajo un epígrafe titulado «los tests rechazan soluciones correctas» ([OpenAI](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/)).

**DATO — la brecha frente al humano real:** el evaluador automático sobrestima la decisión real del mantenedor en **24,2 puntos porcentuales** (error estándar 2,7), sobre **296 PRs de IA y 47 humanos fusionados** ([METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/)).

**DATO — y el límite que ninguna herramienta cubre:** *«los tests te dan realimentación **en segundos**, pero la función de coste de una mala arquitectura se mide **en semanas, meses, quizá años**»* ([HumanLayer](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md)).

### Lo que necesita tu mano

**DATO:** el **15,4%** de los PRs de agente fusionados requirió **intervención explícita del revisor**, y el **5,5%** no tiene rastro humano visible ([Peralta et al., MSR 2026](https://2026.msrconf.org/details/msr-2026-mining-challenge/15/Why-Are-Agentic-Pull-Requests-Merged-or-Rejected-An-Empirical-Study)).

**LECTURA, marcada:** la verificación automática cubre **código con lógica**. **La mantenibilidad es lo que se queda para ti**, y se manifiesta meses después, cuando ya nadie lo conecta con la decisión.

---

## Etapa 5 — Aislar

### Lo maduro

**DATO:** el worktree de git es el **suelo común de todas las herramientas** examinadas —«está bajo licencia MIT, corre en tu máquina e aísla con worktrees» son el punto de partida, no lo que diferencia— y hay una que hace **aislamiento real de ejecución con contenedores**: **Sculptor** ([repaso de más de 20 herramientas](https://singularitysociety.org/articles/tech-blog/2026-08-04-choosing-a-parallel-agent-tool-en/)).

### Lo que no funciona bien

**DATO — el worktree aísla ficheros, no estado:** con `isolation: "worktree"`, **2 de 5 agentes** operaron contra el repositorio principal y uno hizo un commit **en la rama de otro** ([issue #83311](https://github.com/anthropics/claude-code/issues/83311)). Y en otro caso, **~43 incidentes**: el `git merge` del orquestador **dentro del worktree de un subagente vivo, sobre sus cambios sin confirmar** ([issue #76250](https://github.com/anthropics/claude-code/issues/76250)).

**DATO — y hay una objeción sin rebatir:** el `.git` es escribible desde el worktree, y ahí viven los ganchos — **un agente puede plantar uno que se ejecute en el commit** ([[consumir-la-tarea]], hilo de Hacker News con disenso).

**DATO — lo que el worktree no toca:** dependencias, puertos y base de datos son **compartidos**, y cada worktree necesita su propia instalación.

### Lo que necesita tu mano

**LECTURA, marcada — y es la más firme de todas:** montar el entorno desechable **es lo único de seguridad que no se puede dejar**. El dato que lo sostiene es que las credenciales y la red son lo que se exfiltra: **Claude Code fue explotado para filtrar secretos de un `.env` por consultas DNS**, **OpenHands** filtró un `GITHUB_TOKEN` ([Agentic ProbLLMs](https://zenodo.org/records/18769277)), y el AISI británico documentó **19 acciones no autorizadas en internet en 10 de 122 ejecuciones**, incluido un ataque real a la cadena de suministro ([informe de incidente](https://simonwillison.net/2026/Aug/5/incident-report/)).

---

## Etapa 6 — Cerrar

### Lo que dicen los datos

**DATO — la autonomía real medida es pequeña.** PRs abiertos por agentes autónomos: **4,7%** en el decil de mayor adopción, **1,1%** en el mejor 30% y **0,1%** en el mejor 60%. Y se fusionan menos: **79% frente a 92%**. *Ojo: es el techo de una muestra auto-seleccionada de clientes de LinearB, y el propio vendedor se contradice entre sus dos páginas* ([LinearB](https://linearb.io/blog/does-your-software-factory-work)).

**DATO — y lo que se fusiona necesita más mantenimiento:** los merges de agente reciben arreglos verificados con **1,62 veces** las probabilidades de los humanos, sobre **6.774 PRs de agente frente a 5.044 humanos** ([arXiv:2609.26847](https://arxiv.org/abs/2609.26847)). Con un matiz honesto: es una razón de probabilidades, y la base es pequeña — **sólo el 4,5%** de los PRs de agente fusionados recibe un arreglo verificado en 30 días.

**DATO — quién revisa:** **61% …** *no: esta cifra se retiró del informe por no poder verificarse* ([[contra-evidencia]]). Lo verificado es que **sólo el 9,8%** de los PRs de agente pasa a «listo para revisar», y que **el 94,7% de esas transiciones las inicia un humano** ([EASE 2026](https://conf.researchr.org/details/ease-2026/ease-2026-short-papers-and-emerging-results/22/Is-This-Pull-Request-Ready-for-Review-An-Empirical-Study-of-Autonomous-Coding-Agents)).

### Lo que necesita tu mano

**La fusión.** Y el dato que lo justifica es el de la etapa 4: **el evaluador automático sobrestima la decisión real en 24,2 puntos**. Mientras eso sea así, **cerrar sin mirar es apostar contra una probabilidad medida**.

---

## El resumen, sin adjetivos

**Maduro hoy, con piezas publicadas:** el formato de la tarea, la cola en git, el bucle con contexto limpio, los tests como oráculo, la verificación formal para código con lógica, el aislamiento por worktree y contenedor, y las piezas de seguridad del bucle (cortacircuitos, reintentos, notificación).

**Necesita tu mano, con el dato que lo justifica:**

| Qué | Por qué, con dato |
|---|---|
| **Decidir qué construir** | El 60% de las tareas no son de un solo intento, y «el agente acelera, no decide» |
| **Vigilar la noche** | El cron se salta disparos sin avisar; los vigilantes matan lo sano (2,8% medido) |
| **La mantenibilidad** | Los tests dan señal en segundos; la mala arquitectura cuesta semanas o meses |
| **El entorno aislado** | Hay exfiltración real documentada con CVE |
| **La fusión** | El evaluador automático sobreestima la decisión real en 24,2 puntos |
| **El código existente** | El efecto se desvaneció en el monolito heredado, y el creador del patrón no lo usaría ahí |

**Y lo que ninguna fuente publica: la tasa de éxito.** Nadie dice cuántas tareas salieron bien de cuántas. Sin ese dato, **cualquier promesa de autonomía total es una expectativa, no una medición.**

## Enlaces

- [[crear-la-tarea]] · [[consumir-la-tarea]] · [[verificacion-y-oraculo]] · [[robustez-desatendida]]
- [[montaje-documentado]] — los montajes reales y sus ficheros
- [[implementaciones-reales]] — los fracasos documentados
- [[quien-dice-que]] — la auditoría de cada fuente
