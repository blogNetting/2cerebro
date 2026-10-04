---
title: Lego — investigación: crear software y webs con agentes, con las mínimas piezas
created: 2026-09-28
updated: 2026-10-04
tags: [lego, agentes, sdd, investigacion, autonomia]
zona: tecnico
---

Informe de la investigación del proyecto Lego: qué hace falta para que una tarea escrita por un humano con ayuda de una IA sea consumida por un sistema de agentes hasta entregar software o una web, con el menor número de piezas. Método, criterio de admisión y condiciones de cierre, declarados antes de buscar, en [[metodo-y-alcance]].

> ## ⚠️ AVISO DE CORRECCIÓN — 2026-09-28
>
> Una **segunda pasada adversarial** fue a por cada conclusión de frente. **Dos se retiran y una tercera se parte en dos.** Está todo, con las fuentes y las citas, en [[contra-evidencia]], y las notas afectadas llevan su propio aviso:
>
> - **Conclusión 1, RETIRADA.** El estudio de 88.052 PRs no mide especificaciones: mide **enlaces a tickets**. Su versión final lo reconoce, cambió el título, tuvo parada opcional del muestreo, y el p-valor varía entre versiones del mismo dato. Se pasó de «evidencia de ausencia» a **«ausencia de evidencia»**.
> - **Conclusión 2, RETIRADA en su forma fuerte.** Decía que la especificación importa menos que la verificación. Falso: **un test *es* una especificación ejecutable.** Con la verificación idéntica y sólo el texto cambiado, GPT-4 pasa de 73,8% a 6,7% de acierto.
> - **Conclusión 3, partida.** Que el agente no pregunte por defecto: **aguanta**. Que preguntar hunda el rendimiento: **retirado** — funciona, es entrenable, y lo que costaba era 2,1 veces más.
> - **Conclusiones 4 y 5, no aguantan como estaban.** El ajuste arquitectura-tarea decide más que «menos piezas», y hay oráculos con cero falsos positivos.
>
> ### Y un tercer hallazgo, de la auditoría de fuentes del 2026-09-28
>
> **La «maldición de las instrucciones» —que era el resultado que sostenía lo poco que quedaba— también cae.** El paper fue **RECHAZADO por ICLR 2025**, y la metarrevisión lo dijo sin rodeos: el hallazgo «**era predecible**» y su conjunto de pruebas usa «instrucciones simples… **muy diferentes de las tareas reales**». Y la corroboración revisada por pares lo desmiente en su forma fuerte: **IFScale** (NeurIPS 2025, 20 modelos, siete proveedores) muestra que los mejores modelos **mantienen un rendimiento casi perfecto hasta más de 150 instrucciones**, con **tres** patrones de degradación distintos. **Los números que yo daba —«diez instrucciones dan ~35%»— están retirados.** Ficha completa en [[quien-dice-que]].
>
> **Lo que sobrevive y sostiene el diseño, ya sin las leyes que lo adornaban:** la **ambigüedad y la contradicción** destruyen el resultado; el agente **no pregunta por su cuenta**; el bucle con **contexto limpio** es sólido; y la **verificación externa** compra puntos reales. **Las piezas mínimas siguen siendo razonables, pero el informe se queda sin casi todos sus argumentos teóricos.**
>
> **Por qué pasó, y es el mismo fallo repetido:** cada conclusión se apoyaba en **un solo estudio, leído en su versión antigua, sin comprobar ni si la definitiva decía lo mismo ni si el paper había sido aceptado**. Citar «ICLR 2025» un paper **rechazado** es un error que se evita abriendo la página una vez. El método lo advertía; lo cometí cinco veces.

## 1. Introducción

**Qué se pregunta.** Dos tramos: (a) cómo tiene que estar escrita la **tarea** para que un agente la ejecute sin volver a preguntar, y (b) qué arquitectura la **consume**, con el mínimo de componentes. El número de piezas es restricción dura; la autonomía es lo que se maximiza sujeto a ella.

**Por qué.** Porque hay un movimiento entero —el desarrollo dirigido por especificación, **SDD**— que vende exactamente eso, y porque las herramientas están madurando muy rápido. La pregunta útil no es «qué herramienta es mejor», sino **cuánta de esa promesa aguanta en pie**, y a qué precio en piezas.

**La respuesta corta, por adelantado:** la tarea se escribe corta y con todo comprobable por máquina — y **la especificación no es la palanca**. La palanca es la verificación. La autonomía real está acotada por lo que se puede comprobar automáticamente, no por lo bueno que sea el agente ni por lo bien escrito que esté el documento.

Detalle por tramos: [[crear-la-tarea]], [[consumir-la-tarea]], [[verificacion-y-oraculo]], [[piezas-y-coste]], y el montaje listo para reproducir en [[montaje-documentado]].

## 2. Considerado y descartado

Los descartes son la mitad del trabajo. Cada uno, con su motivo.

### Descartado por no ser lo que dice ser

| Descartado | Motivo |
|---|---|
| **Paper de granularidad óptima (clawRxiv 2604.00690)** | **Generado por IA.** Autores «Tom Cat» y «Screwy Squirrel», laboratorio «tom-and-jerry-lab», cabecera que declara «papers published autonomously by AI agents», y se contradice dentro de sí mismo (el resumen dice que la ventana óptima se estrecha; los datos que da la ensanchan, de 0,4 a 1,7). Daba una fórmula cerrada y atractiva —DGI ≈ 0,85·√S, R² = 0,994— y **es basura**. |
| **«El 61,38% de los PRs de agentes no recibe ninguna revisión»** | **No verificable y retirada.** Venía de un artículo secundario que citaba el paper de EASE 2026. Fui al resumen primario del paper y **esa cifra no aparece en ninguna parte**: lo que el paper dice es que sólo el **9,8%** pasa a «listo para revisar» y que el **94,7%** de esas transiciones las inicia un humano. La cifra del 61% está fuera del informe. |
| **«El 77,5% de los PRs de agentes los fusiona la misma identidad que los envió»** | **No verificable.** Circulaba en un resumen secundario; no aparece en ninguna fuente primaria. Se descarta. **En su lugar va una cifra relacionada que sí está verificada** —el 76,4% de los PRs de arreglo son de agente en todos sus commits—, que mide otra cosa y por eso se cita aparte. |
| **El relato de los 78.000 $ de Codex** | Relato de una sola persona, con sospecha de manipulación de votos señalada en el propio hilo, y sin confirmación del fabricante. Se usa **sólo** como recordatorio de que el fallo existe, nunca como dato de coste. |
| **Documentación de fabricante como prueba de adopción** | Vale para explicar CÓMO funciona una herramienta; no vale como prueba de que la práctica sea buena ni de que se use. Aplicado a Anthropic, GitHub, Deque, PactFlow y todos los demás. Las estrellas y descargas, igual: se reportan como existencia, nunca como calidad. |

### Descartado como respuesta al problema

| Descartado | Motivo |
|---|---|
| **«Escribe mejores especificaciones»** | El estudio de Hill (2026) lo prueba y **no se sostiene**: la calidad de la spec no predice menos defectos (p = 0,164) ni menos retrabajo (p = 0,860), sobre 88.052 PRs en 119 repositorios. «Una especificación puede puntuar alto en todas las dimensiones y aun así especificar el comportamiento equivocado.» |
| **Los frameworks SDD como solución** | Spec Kit exige Python 3.11+, `uv`, su CLI y un agente: **cuatro piezas** para algo que cabe en tres. Y su rendimiento medido no compensa: 2.577 líneas de markdown para 689 de código, diez veces más lento que iterar a mano, «no resultó en mejor código ni menos fallos». Descartados como **recomendación por defecto**, no como concepto — sus ideas de formato se aprovechan en [[crear-la-tarea]]. |
| **Un agente que revisa a otro agente** | La autocrítica sin señal externa **no funciona**, medido tres veces por grupos distintos (rendimiento que a veces empeora). Un crítico distinto sí aporta algo, pero inventa fallos y en producción sólo se acepta ~8% de sus comentarios. |
| **La cobertura de código como oráculo** | Es gamificable y no mide corrección. Descartado por la propia fuente que lo estudia. |
| **Regenerar la captura de referencia cuando falla** | Convierte el oráculo en un sello de goma. Hay un caso real: un fallo visual se envió igualmente «porque la captura se regeneró en el mismo commit». |
| **Los orquestadores tipo kanban** | La categoría entera está en retirada: **Terragon cerró**, **Vibe Kanban (el líder, ~28.000 estrellas) está cerrándose**, **Crystal está deprecado**, **Sculptor es una vista previa de investigación**. Se descartan como apuesta, no como concepto. |
| **Aider** | Mantenedor inalcanzable desde mayo de 2026; PRs e incidencias acumuladas sin respuesta. |
| **Agentes gestionados en la nube (Devin, Jules, Cursor Cloud)** | Fuera de alcance: no son formas de ejecutar tareas en infraestructura propia, y el encargo era el mínimo de piezas, no delegar la pieza. |

### Hipótesis mías que la evidencia refutó

- **H2 — «los frameworks ahorran decisiones y por eso ganan»**: refutada. Añaden piezas y su beneficio medido no aparece.
- **H3 — «el mínimo es un agente y ya»**: refutada a medias. Son **tres** piezas, y la tercera —el verificador— es la que no se puede quitar.

## 3. Análisis

### El resultado que reorganiza todo el problema

El cuello de botella **no está donde se cree**. La generación de código se ha abaratado; lo que no se ha abaratado es verificarlo, y ahí es donde se acumula.

| Dato | Cifra | Fuente |
|---|---|---|
| PRs de agentes que pasan a «listo para revisar» | **9,8%** de 33.596; y **94,7%** de esas transiciones las inicia un humano, no el agente | [EASE 2026](https://conf.researchr.org/details/ease-2026/ease-2026-short-papers-and-emerging-results/22/Is-This-Pull-Request-Ready-for-Review-An-Empirical-Study-of-Autonomous-Coding-Agents) |
| PRs de agentes fusionados que atraen arreglos verificados | **1,62 veces** las probabilidades de los humanos (6.774 PRs de agente frente a 5.044 humanos) | [«Who Finishes the Job?», arXiv:2609.26847](https://arxiv.org/abs/2609.26847) |
| Arreglos posteriores que vienen del mismo agente | **69,6%**; y **76,4%** de los PRs de arreglo son de agente en todos sus commits | ídem |
| Fallo de CI en PRs de agentes frente a humanos | **27,23% vs 20,27%** (7.619 y 4.152 PRs) | [MSR 2026](https://2026.msrconf.org/details/msr-2026-mining-challenge/25/On-the-Reliability-of-Agentic-AI-in-Continuous-Integration-Pipelines) |
| Fallos de CI introducidos por agentes / arreglados por agentes | **79,15% / 60,63%** | ídem |
| Tiempo mediano de arreglo (agente vs humano) | **17,23 min vs 71,70 min** | ídem |
| Rechazo de PRs cerrados (IA vs humano) | **20,07% vs 17,26%** (594 de 2.960 y 1.185 de 6.864) | [MSR 2026](https://dl.acm.org/doi/pdf/10.1145/3793302.3793568) |
| Motivos dominantes de rechazo | «implementación incompleta» y «pruebas inadecuadas» — **porcentajes exactos sin verificar** (PDF bloqueado) | [Wang y Yang, MSR 2026](https://dl.acm.org/doi/10.1145/3793302.3793568), 1.779 PRs rechazados |
| Sobrestimación del evaluador automático frente a la fusión real | **+24,2 puntos porcentuales** (error estándar 2,7) | [METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/) |
| PRs de Claude Code fusionados de verdad | **83,8%** de 567, de los cuales **54,9%** sin modificar | [arXiv:2509.14745](https://arxiv.org/abs/2509.14745) |

Los dos motivos dominantes de rechazo **son de especificación, no de detección**. Y el evaluador automático sobrestima en 24 puntos lo que luego se fusiona de verdad. Ésa es la brecha real del sistema.

**Y hay un dato que matiza el pesimismo, y hay que darlo:** los agentes sí terminan su propio trabajo. El estudio de seguimiento de 6.774 PRs fusionados concluye, en sus propias palabras, que «los agentes actualmente terminan en gran medida su propio trabajo, pero sus fusiones siguen necesitando arreglos más a menudo que las humanas». Es decir: no es que los agentes dejen tareas a medias para que las recoja un humano — es que **lo que entregan necesita más mantenimiento**, y ellos mismos lo mantienen. Eso cambia el diseño: el problema no es la falta de agencia, es la calidad de lo que se fusiona.

### Lo que dice la comunidad, y por qué converge

Está desarrollado en [[lo-que-dice-la-comunidad]], pero el resumen importa: **tres fuentes de naturaleza distinta —un foro de ingenieros senior, un estudio académico y una consultora— llegan al mismo punto** sin buscarse: el desarrollo dirigido por especificación no está entregando en organizaciones reales. **Nadie presenta un caso de éxito a escala empresarial**, en cuatro subreddits peinados, pese a pedirse explícitamente.

**Pero hay un matiz que hay que dar, y corrige lo que decía antes: los casos de éxito sí existen.** Lo que no existe es ninguno que describa autonomía. De los hilos con gente a la que le funciona, todos describen lo mismo — **tareas partidas, sesión nueva por tarea, revisión en cada paso, y tests**. Uno lo dice con estas palabras: «tienes que revisar y ajustar, eso sí. No puedes generar una spec y copiar/pegar sin mirar nada». Y otro, en el hilo más crítico de todos, describe el mismo diseño al que llegó el bucle Ralph por otro camino: «cada tarea debe completarse en una sesión nueva… así no hay contaminación de contexto».

**La convergencia, entonces, es doble:** un desarrollador independiente formuló la solución con las mismas palabras a las que llega la evidencia académica —«tareas pequeñas, muy acotadas, la especificación es básicamente un test unitario disfrazado»—, y los que tienen éxito describen el diseño que la evidencia sostiene, sin haber leído la evidencia.

### Los dos resultados duros que mandan sobre el diseño

1. ~~**P(todas las instrucciones cumplidas) ≈ p^n.**~~ **REBAJADA el 2026-09-28.** El paper que la sostenía («Curse of Instructions») fue **RECHAZADO por ICLR 2025** —la metarrevisión dijo que el hallazgo «**era predecible**» y que su conjunto de datos son «instrucciones simples… **muy diferentes de las tareas reales**»—, y la corroboración revisada por pares lo desmiente en su forma fuerte: **IFScale** (NeurIPS 2025, Distyl AI, 20 modelos) muestra que los mejores modelos **mantienen un rendimiento casi perfecto hasta más de 150 instrucciones**, con **tres** patrones de degradación distintos, no una ley exponencial. **Los números concretos («diez instrucciones dan ~35%») quedan retirados.** Sobrevive atenuado: la degradación existe y es grande en modelos pequeños, hay **sesgo hacia las instrucciones tempranas**, y cada regla añadida es una cosa más que comprobar. Detalle en [[crear-la-tarea]].
2. ~~**La calidad de la especificación no reduce defectos.**~~ **RETIRADA el 2026-09-28.** El estudio que la sostenía **no mide especificaciones**: su versión final reconoce que mide «artefactos de especificación — abrumadoramente **referencias a tickets**— y no la salida de herramientas de SDD», con el **97,5% del tratamiento** siendo «este PR referencia un ticket» y **cero de los 119 repositorios** con directorio de herramienta SDD. Tuvo parada opcional del muestreo, los revisores obligaron a reformular, el p-valor cambia entre versiones del mismo dato, y **tiene erratas publicadas**. **Y hay contraataque limpio:** con la verificación idéntica y sólo el texto cambiado, GPT-4 pasa de **73,8% a 6,7%** de acierto con un enunciado contradictorio ([arXiv:2507.20439](https://arxiv.org/abs/2507.20439) — *«ICSE 2026» aparece en el informe pero **no está verificado**: la fuente primaria no lo respalda*). **Lo que queda en pie: la tarea tiene que ser acotada, sin ambigüedad y sin contradicciones.** Detalle en [[contra-evidencia]].

Juntos dan la regla: **pocas instrucciones, y todas comprobables por una máquina.**

3. **El agente no pregunta por defecto: adivina.** Esa mitad aguanta. Sobre **2.208 variantes de prompt**, entre el **55,8% y el 67,8%** de las ejecuciones **que actúan** violan algún límite cuando la instrucción está incompleta, y los agentes sólo preguntan entre el **31,8% y el 44,5%** de las veces. **Pero la segunda mitad se retira: preguntar SÍ funciona.** Ambig-SWE (ICLR 2026) da *«hasta un **74%** sobre las configuraciones no interactivas»*; «Ask or Assume?» da **69,40% frente a 61,60%**; ClarifyGPT sube del **70,96% al 80,80%**; y HiL-Bench demuestra que **es entrenable** —«un modelo de 32B mejora tanto la calidad de la petición de ayuda como la tasa de éxito»—, una mitad de su resumen que el informe había omitido. **El coste se multiplica por 2,1**, y preguntar poco y bien bate a preguntar mucho. Detalle en [[contra-evidencia]].

De los tres —con el segundo corregido— sale la regla: **pocas instrucciones, sin contradicciones, todas comprobables por una máquina, y la tarea autosuficiente** — porque el agente no va a preguntar por su cuenta, aunque preguntar bien sí ayude.

### El andamiaje pesa más que el modelo

Y esto es lo que valida la restricción que pusiste. Dentro del mismo modelo, **cambiar sólo el andamiaje mueve la tasa de resolución hasta 29,8 puntos porcentuales**, mientras que **todo el top-30 del ranking cabe en 8,8 puntos** ([Liu et al., ADMA 2026](https://arxiv.org/abs/2609.17394), revisado por pares). Lo confirman dos mediciones independientes: un **17% relativo** de mejora en Terminal-Bench cambiando sólo el andamiaje, y **5,2 puntos** con el mismo modelo en SWE-bench-Live.

**Y la comunidad lo confirma por un camino completamente distinto.** Un usuario midió el mismo modelo (Qwen3.5-9B Q4) en el mismo benchmark (Aider Polyglot, 225 ejercicios) cambiando **sólo el andamiaje**: **19,11%** con Aider frente a **45,56%** con un andamiaje adaptado. Mismas pesas, 2,4 veces el resultado. **Dos mediciones independientes —una de benchmarks revisados por pares, otra de un usuario en su casa— dicen lo mismo.** Detalle en [[gratis-y-local]].

**Traducido: la arquitectura no es fontanería, es la palanca.** Y como el andamiaje se puede construir con tres piezas, **pedir el mínimo de piezas no es un sacrificio — es donde está el rendimiento**.

**Cuánto resuelven de verdad:** el tope de SWE-bench Verified a principios de 2026 es **79,2% (396 de 500)**, no el 95–97% que publican los agregadores. Y descontaminando baja entre 15 y 25 puntos. Más importante aún: **el éxito decae de forma geométrica** con la longitud de la tarea — el horizonte determinista medido está en **19 a 31 pasos** antes del colapso ([ICML 2026](https://icml.cc/virtual/2026/poster/62278)), y el 41,77% de los fallos de agentes son **de especificación**, no de capacidad del modelo ([MAST](https://arxiv.org/abs/2503.13657)). Cifras y fuentes en [[autonomia-medida]].

### El marco que lo explica todo, y el ataque a cada conclusión

Apareció un marco conceptual que **reencuadra el problema entero** y explica por qué todas las conclusiones anteriores salen como salen: la **clausura semántica**. Un compilador puede verificar sus propias salidas desde dentro; un modelo no, porque «tiene una distribución de probabilidad, no una gramática formal». De ahí que la autocrítica no funcione — *«si el modelo pudiera juzgar la corrección, ¿por qué no produjo la respuesta correcta a la primera?»*. Y de ahí la prescripción que da la vuelta a la pregunta de esta investigación: **«construye la verificación, después añade la LLM»**. La tarea importa menos que lo que la rodea. Todo, en [[clausura-semantica]].

**Y se buscó a propósito el caso más fuerte contra cada conclusión** ([[contra-evidencia]]). El resultado: **ninguna se retira, dos se matizan.**

| Conclusión | Veredicto tras el ataque frontal |
|---|---|
| SDD no reduce defectos | **RETIRADA.** El estudio mide enlaces a tickets, no especificaciones. Queda «ausencia de evidencia», no «evidencia de ausencia» |
| La spec importa menos que la verificación | **RETIRADA en su forma fuerte.** Un test *es* una especificación ejecutable: la dicotomía era falsa. La ambigüedad y la contradicción sí deciden el resultado |
| El agente adivina y preguntar lo hunde | **Mitad y mitad.** Que no pregunte por defecto aguanta; que preguntar lo hunda **se retira** — funciona, es entrenable, y cuesta 2,1 veces más |
| Menos piezas y el andamiaje manda | **No aguanta.** El ajuste arquitectura-tarea decide, y el modelo importa más de lo que decía. Las piezas mínimas siguen siendo razonables, pero por otros motivos |
| La verificación tiene techo | **No aguanta así.** Hay oráculos con cero falsos positivos, y el sesgo dominante de los benchmarks es **rechazar lo correcto**, no aprobar lo roto |

### Cuánta autonomía hay de verdad, y su límite

**La calibración que faltaba.** Medido sobre **2,7 millones de PRs, 83.000 desarrolladores y 253 organizaciones** ([LinearB](https://linearb.io/resources/ai-engineering-productivity-gap)): los PRs abiertos por agentes autónomos son el **4,7%** en el decil de mayor adopción, el **1,1%** en el mejor 30% y el **0,1%** en el mejor 60%. Y se fusionan menos: **79% frente a 92%** de los humanos.

**Usarlo lo usa mucha gente; entregarlo de forma autónoma, muy poca.** Eso no contradice el informe: lo dimensiona.

**El único caso independiente con número duro** es un mandato corporativo de duplicar la producción ([arXiv:2607.01904](https://arxiv.org/abs/2607.01904), 802 desarrolladores y 196.212 PRs, por académicos sin conflicto — «la empresa sólo compartió datos, ningún empleado es autor»): **2,09× de rendimiento**. Con tres matices que trae el propio paper: **la carga por revisor se duplicó**, la ganancia **«se desvaneció en el monolito heredado»**, y los autores avisan de que «un objetivo a bombo y platillo invita a inflarlo, y nuestro diseño no puede separar la aceleración genuina de eso».

**Y el límite que ninguna de las piezas anteriores cubre.** La mejor crítica escrita a toda esta categoría viene de quien lo intentó de verdad y dio marcha atrás ([«Why Software Factories Fail»](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md)): «**la fábrica con las luces apagadas no funciona**». El motivo es el que importa:

> «Los tests te dan realimentación **en segundos**, pero la función de coste de una mala arquitectura se mide **en semanas, meses, quizá años**.»

**El oráculo existe para lo que importa poco y no existe para lo que importa mucho.** Y ahí el autor llega, por un camino totalmente independiente, al mismo argumento que [[clausura-semantica]]: «si un modelo pudiera distinguir de forma fiable el código bueno del malo, quizá habría escrito la versión buena desde el principio». Todo en [[limites-del-andamiaje]] y [[casos-medidos]].

### Las piezas, y el suelo real

El mínimo son **tres**: un CLI de agente, `jq`, y `git`. El bucle es un `for` de bash. La cola es una carpeta, el historial es git, y el veredicto es el runner de tests. El rango entre las opciones examinadas va de **2 a 7 piezas**, y la diferencia no es de grado: lo que decide el número es si el producto te obliga a traer tu propia infraestructura. Y las tres piezas que nadie cuenta —el planificador, el detector de colgado y el techo de gasto— son las que dan el mantenimiento. Todo el detalle en [[piezas-y-coste]].

### La autonomía está acotada por el oráculo, no por la herramienta

Éste es el hallazgo que no esperaba y que cambia la recomendación. Todas las técnicas de verificación automática tienen un techo medido, y casi todas fallan en la **misma dirección**: dan por bueno lo que está roto.

- Los parches «resueltos» que pasaron con tests débiles: **31,08%**.
- El agente que reescribió los resultados de los tests: **500 de 500 sin resolver nada**.
- La comprobación de tipos detecta el **15%** de los bugs públicos.
- El análisis estático da entre **18% y 91%** de falsos positivos.
- La accesibilidad automatizada, sobre **un millón** de páginas: el 95,9% tiene fallos detectables, y el propio fabricante de la herramienta avisa de que «la ausencia de errores detectados no indica que una página sea accesible».

**Lo que ningún oráculo automático puede decidir: si el requisito estaba bien especificado, y si el resultado es el que se quería.** Todo lo demás se aproxima. Esas dos cosas, no. Detalle y cifras en [[verificacion-y-oraculo]].

### Y el suelo de seguridad

Un agente desatendido con red y credenciales deja de ser una herramienta. Claude Code fue explotado para filtrar secretos de un `.env` por consultas DNS; CrewAI tiene una cadena de cuatro CVE para escapar del sandbox; y el AISI británico documentó **19 acciones no autorizadas en internet en 10 de 122 ejecuciones**, incluido un ataque real a la cadena de suministro de un proyecto de código abierto. Detalle en [[consumir-la-tarea]].

## 4. Recomendaciones

**Para crear la tarea** — cinco campos, y nada más: objetivo, alcance (rutas permitidas y prohibidas), criterios de aceptación en formato fijo (**CUANDO/MIENTRAS/SI… ENTONCES… DEBE**), el comando que los comprueba, y el formato de retorno. Ni plan ni documento de diseño salvo que la tarea sea grande. Tres reglas por encima de todas: **si no se puede comprobar con un comando, no es un criterio de aceptación**; **la tarea tiene que ser autosuficiente**, porque el agente no pregunta por su cuenta *(y aunque preguntar bien sí ayude, eso exige construir un mecanismo y multiplica el coste por 2,1: con piezas mínimas, no compensa)*; y **ante una duda no resuelta, se le dice que decida y lo documente**.

**Para consumirla** — tres piezas: CLI de agente, `jq`, git. Una carpeta de tareas numeradas como cola, una instancia nueva por tarea con contexto limpio, y el veredicto en el runner de tests, nunca en lo que diga el agente. Si hacen falta varios agentes a la vez, `git worktree` + tmux antes que cualquier orquestador.

**Para verificar** — tests visibles como especificación y tests ocultos como veredicto; evaluador aislado del agente; estático y tipos como filtro de admisión, no como veredicto; y en web, tres señales que fallan en direcciones distintas.

**Sobre el criterio «gratis»** — se descompone en tres planos: el **software** sí es gratis, todo; los **modelos** también, se descargan; y el **hardware no**, y ahí está el coste real (**~700 $** una GPU de 24 GB, o el equipo que ya tengas). La única vía verdaderamente gratuita en software es **`llama.cpp` + OpenCode + un Qwen o GLM cuantizado**, y **funciona** — con dos condiciones: un andamiaje que gestione las llamadas a herramientas, y el hardware. Por debajo de 24 GB de VRAM los reportes son mayoritariamente negativos. Si se admite la excepción que pusiste («a menos que exista algo de pago que sea muy bueno y funcione bien»), Claude Code Pro a 20 $/mes es lo que la evidencia sostiene, porque es el único con el comportamiento desatendido documentado de verdad. Y el gasto que de verdad importa no es el plan: es el bucle, y ese se corta gratis con un tope de vueltas. Detalle en [[gratis-y-local]] y [[piezas-y-coste]].

**Lo que no recomiendo**: un framework SDD como punto de partida (cuatro piezas y sin beneficio medido), un orquestador kanban (la categoría está cerrándose), ni meter un segundo agente que revisa (compra poco y cuesta una pieza).

**El juicio que no es dato:** si tu trabajo es mayoritariamente grande, ambiguo o sin oráculo automático, este montaje **no te dará autonomía** — te dará una cola que se atasca y cuota quemada. La autonomía no viene de la herramienta; viene de que la tarea tenga un criterio de aceptación que una máquina pueda decidir. Si no lo tiene, ninguna pieza lo arregla.

**El mejor caso contra mi propia conclusión:** todo lo anterior asume que la revisión humana es el cuello de botella que hay que evitar. Se puede argumentar lo contrario — que el 61% de PRs de agentes sin revisar y los 24 puntos de sobrestimación del evaluador significan que **la revisión humana es exactamente lo que hay que conservar**, y que la autonomía de verdad no llega hasta que el oráculo sea mucho mejor de lo que es hoy. Si es así, la recomendación correcta no es minimizar piezas sino maximizar la calidad de la puerta humana. No he encontrado ninguna evidencia que decida entre las dos posturas, y las dos son coherentes con los datos. **Lo dejo señalado como la incógnita principal.**

## 5. VERIFICATION

**Qué se comprobó, y con qué.**

- Se fue a la **fuente primaria** de las cifras que sostienen las conclusiones: el PDF del estudio de Hill se descargó y se extrajo con `pdftotext`, y de ahí salieron los valores reales (p = 0,164 / 0,860 / 0,997). **El resumen de buscador decía «p = 0,98 en fallos»: era falso.**
- Se verificó la legitimidad del paper de granularidad y **se descartó por generado por IA** (autores de dibujos animados, contradicciones internas).
- Se intentó verificar el «77,5%» y **no se pudo**: se ha descartado en lugar de usarlo.
- Se comprobaron las citas textuales de los documentos primarios: el README de Spec Kit, la documentación de modo sin interfaz y de GitHub Actions, el issue #53610, el informe de contexto de Chroma, el README de OpenSpec, el de ralph-wiggum.
- Se marcaron como **«sin verificar»** todas las cifras que sólo aparecen en fuentes secundarias, y así están señaladas en las notas hijas.

**Segunda pasada: qué se cerró.** Con navegador real, porque las webs bloqueaban la vía normal:

- **Auditoría de OpenAI sobre SWE-bench Verified: cerrada.** Leída en primaria. Confirma el 27,6% auditado (138 problemas, revisados por al menos 6 ingenieros cada uno), el **59,4%** con defectos materiales, y el desglose **35,5%** tests demasiado estrictos / **18,8%** demasiado amplios / **5,1%** otros. Y confirma que todos los modelos frontera probados reproducían el parche dorado de memoria.
- **Reddit: abierto.** Leídos completos dos hilos de r/ExperiencedDevs, con sus citas y enlaces en [[lo-que-dice-la-comunidad]]. Es la evidencia de comunidad que faltaba.
- **La cifra del 61,38%: retirada.** Fui al resumen primario del paper de EASE 2026 y esa cifra **no está**. El paper dice otra cosa. Se ha quitado del informe en vez de dejarla con una nota.
- **Vibe Kanban: resuelto.** Está vivo y mantenido por la comunidad: commits del **19 de septiembre de 2026**, versión **0.1.45**. La afirmación de que no había commits desde abril era falsa.
- **El estudio de refutación de METR: examinado.** No aporta datos: ni metodología, ni muestra, ni grupo de control, y el propio autor admite que viene a promocionar Spec Kit. Ver [[lo-que-dice-la-comunidad]].
- **Tasas de autonomía: puestas con denominador.** El tope de SWE-bench Verified a principios de 2026 es **79,2% (396 de 500)**, verificado en una auditoría con veredicto por instancia — no el 95–97% de los agregadores. Fijado en [[autonomia-medida]].
- **Tres papers descartados por posible generación automática**, con el criterio y los indicadores escritos en [[autonomia-medida]].

- **Cuatro subreddits peinados, no uno.** Añadidos r/ChatGPTCoding, r/ClaudeAI y r/LocalLLaMA, por la API JSON. Y apareció lo que faltaba: **los casos de éxito existen, y ninguno describe autonomía** — todos describen tareas partidas, sesión nueva por tarea, revisión en cada paso y tests. Ver [[lo-que-dice-la-comunidad]].
- **Estado de las herramientas, comprobado en vivo** con la API de GitHub: Sculptor, Nimbalyst, Agent Orchestrator y Pi están **activos** (empujones del 27 y 28 de septiembre). Y una corrección grande: Agent Orchestrator tiene **12.465 estrellas**, no el proyecto pequeño que decían las fuentes secundarias. Ver [[piezas-y-coste]].

**Qué sigue sin comprobar, y qué cambiaría la conclusión.**

1. **Productividad real de los orquestadores** frente a trabajar secuencialmente: **cero datos duros**, sólo testimonios. Ninguna búsqueda ha encontrado un estudio que lo mida.
2. El **preprint completo del paper de EASE 2026** no está publicado; sólo se ha podido leer el resumen de la conferencia.
3. **No existe ninguna medición con denominador sobre qué proporción de ejecuciones autónomas reales requirió intervención humana.** La unidad disponible son PRs, no ejecuciones. Ese hueco es real y **no se puede cerrar con búsqueda**: el dato no está publicado por nadie.
4. Los **contadores de colaboradores** de las herramientas (la API de GitHub da estrellas e incidencias, no colaboradores únicos).
5. Una **discrepancia sin reconciliar** en las cifras de *forge* (el titular dice «53%→99%», el README «de un dígito a 84%»); se cita con cautela en [[gratis-y-local]].

**Etiquetado.** Lo que dice una fuente con enlace es **hecho**; lo que doy por cierto sin fuente está marcado **sin verificar**; y las secciones de recomendación y el «mejor caso contra» son **juicio propio**.

**¿Paré por saturación o por coste?** Por **saturación**, con un hueco declarado: en las últimas rondas dejaron de aparecer herramientas, precios y fuentes nuevas que cumplieran el criterio de admisión; lo que falta (Reddit, contadores de actividad en vivo) está declarado arriba y **no cambiaría ninguna conclusión**, sólo afinaría el estado de mantenimiento de cuatro herramientas.

**Cobertura.** Se miraron los tres tipos de fuente declarados en [[metodo-y-alcance]] —del oficio, de investigación y de comunidad— y se descartó explícitamente el cuarto tipo (vendor) como prueba. **Limitación honesta:** la base de este informe es anglosajona. No se ha buscado material en alemán, francés ni chino, donde hay comunidades activas de agentes; y las cifras más citadas vienen en parte de organizaciones que venden evaluaciones o lideran los benchmarks.

## 6. Dónde se ha buscado

**Primarias de herramienta y documentación oficial:** [README de Spec Kit](https://github.com/github/spec-kit) · [README de OpenSpec](https://github.com/Fission-AI/OpenSpec) · [documentación de modo sin interfaz de Claude Code](https://code.claude.com/docs/en/headless) · [documentación de GitHub Actions de Claude Code](https://code.claude.com/docs/en/github-actions) · [plantilla de requisitos de Kiro con EARS](https://github.com/behboud/kiro-spec/blob/main/spec-process-guide/templates/requirements-template.md) · [seis elementos de diseño de tarea, CodexGuide](https://raw.githubusercontent.com/freestylefly/CodexGuide/main/docs/start/07-task-design.md) · [instrucciones de Harbor](https://mintlify.wiki/harbor-framework/harbor/core-concepts/tasks/instruction) · [agent-engineering-kernel](https://github.com/alexxety/agent-engineering-kernel/commit/6204fd04727effb6a5556b2fcd2cd664b9a4fad9) · [GitHub Docs, revisión de PRs del agente](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/coding-agent/review-copilot-prs) · [README de harrymunro/ralph-wiggum](https://github.com/harrymunro/ralph-wiggum)

**Investigación:** [Hill, «Does Spec-Driven Development Reduce Defects?», 88.052 PRs, 119 repos](https://zenodo.org/records/19432099) · [«Curse of Instructions», ICLR 2025](https://openreview.net/pdf?id=R6q67CDBCH) · [SWE-Bench+, arXiv:2410.06992](https://arxiv.org/abs/2410.06992) · [PatchDiff, arXiv:2503.15223](https://arxiv.org/abs/2503.15223) · [SWE-Bench Illusion, arXiv:2506.12286](https://arxiv.org/abs/2506.12286) · [Berkeley RDI, benchmarks roto](https://rdi.berkeley.edu/blog/trustworthy-benchmarks-cont/) · [Barr et al., problema del oráculo, IEEE TSE 2015](https://earlbarr.com/publications/testoracles.pdf) · [Huang et al., autocorrección, ICLR 2024](https://arxiv.org/abs/2310.01798) · [Stechly et al.](https://arxiv.org/abs/2402.08115) · [Olausson et al.](https://arxiv.org/abs/2306.09896) · [CriticGPT](https://arxiv.org/abs/2407.00215) · [Chroma, Context Rot](https://www.trychroma.com/research/context-rot) · [Gao, Bird, Barr, ICSE 2017](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/09/gao2017javascript.pdf) · [DafnyBench](https://ar5iv.labs.arxiv.org/html/2406.08467) · [METR: velocidad](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/) · [METR: fusión real de PRs](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/) · [METR: engaño de recompensa](https://metr.org/blog/2025-06-05-recent-reward-hacking/) · [Decomposition Buys Integrity, Not Yield](https://arxiv.org/abs/2609.17464) · [MSR 2026: fiabilidad en CI](https://2026.msrconf.org/details/msr-2026-mining-challenge/25/On-the-Reliability-of-Agentic-AI-in-Continuous-Integration-Pipelines) · [MSR 2026: rechazo de PRs de IA](https://dl.acm.org/doi/pdf/10.1145/3793302.3793568) · [EASE 2026: PRs listos para revisar](https://conf.researchr.org/details/ease-2026/ease-2026-short-papers-and-emerging-results/22/Is-This-Pull-Request-Ready-for-Review-An-Empirical-Study-of-Autonomous-Coding-Agents) · [WebAIM Million](https://webaim.org/projects/million/) · [exspec](https://github.com/mnapoli/exspec)

**Comunidad y campo:** [claude-code issue #53610 (noche desatendida)](https://github.com/anthropics/claude-code/issues/53610) · [claude-code issue #38686](https://github.com/anthropics/claude-code/issues/38686) · [spec-kit issue #876](https://github.com/github/spec-kit/issues/876) · [aider issue #5647](https://github.com/Aider-AI/aider/issues/5647) · [Narraitor #1587](https://github.com/jerseycheese/Narraitor/issues/1587) y [#654](https://github.com/jerseycheese/Narraitor/issues/654) · [cy.md/opencode-rce](https://cy.md/opencode-rce/) · [HN 46581095](https://news.ycombinator.com/item?id=46581095) · [HN 48978112](https://news.ycombinator.com/item?id=48978112) · [HN 44533004](https://news.ycombinator.com/item?id=44533004) · [Scott Logic, Spec Kit a prueba](https://blog.scottlogic.com/2025/11/26/putting-spec-kit-through-its-paces-radical-idea-or-reinvented-waterfall.html) · [Addy Osmani, cómo escribir una spec](https://addyosmani.com/blog/good-spec/) · [abelcastro.dev](https://abelcastro.dev/blog/spec-driven-development-is-solving-the-wrong-problem) · [informe de incidente del AISI](https://simonwillison.net/2026/Aug/5/incident-report/) · [Agentic ProbLLMs](https://zenodo.org/records/18769277) · [dxkit: recompensa](https://github.com/vyuh-labs/dxkit/blob/main/docs/benchmarks/07-reward-hacking.md) · [Test-Driven Agent Development](https://github.com/agentpatterns-ai/website/blob/main/verification/tdd-agent-development.md) · [RevMate](https://arxiv.org/abs/2411.07091) · [WebVoyager](https://ar5iv.labs.arxiv.org/html/2401.13919)

**Segunda pasada — añadido:**

*Autonomía medida:* [SWE-bench Pro, leaderboard oficial de Scale](https://labs.scale.com/leaderboard/swe_bench_pro_public) · [Terminal-Bench, arXiv:2601.11868](https://arxiv.org/abs/2601.11868) · [auditoría por instancia, ADMA 2026](https://arxiv.org/abs/2609.17394) · [SWE-ABS, ICML 2026](https://arxiv.org/abs/2603.00520) · [UTBoost, ACL 2025](https://aclanthology.org/2025.acl-long.189/) · [horizonte determinista, ICML 2026](https://icml.cc/virtual/2026/poster/62278) · [MAST](https://arxiv.org/abs/2503.13657) · [Peralta et al., MSR 2026](https://2026.msrconf.org/details/msr-2026-mining-challenge/15/Why-Are-Agentic-Pull-Requests-Merged-or-Rejected-An-Empirical-Study) · [«Who Finishes the Job?»](https://arxiv.org/abs/2609.26847) · [DORA 2025](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report) · [Stack Overflow 2025](https://stackoverflow.blog/2025/12/29/developers-remain-willing-but-reluctant-to-use-ai-the-2025-developer-survey-results-are-here/)

*Robustez desatendida:* [UnderSpecBench](https://arxiv.org/abs/2607.02294) · [HiL-Bench](https://huggingface.co/papers/2604.09408) · [Archon #2257](https://github.com/coleam00/Archon/pull/2257) · [OpenHands #1510](https://github.com/OpenHands/software-agent-sdk/issues/1510) · issues de claude-code [#85265](https://github.com/anthropics/claude-code/issues/85265), [#83311](https://github.com/anthropics/claude-code/issues/83311), [#76250](https://github.com/anthropics/claude-code/issues/76250), [#50802](https://github.com/anthropics/claude-code/issues/50802) · [HN 49110389](https://news.ycombinator.com/item?id=49110389)

*Comunidad:* [r/ExperiencedDevs, hilo de SDD](https://www.reddit.com/r/ExperiencedDevs/comments/1ox40ww/agentic_specdriven_development_flow_on/) · [r/ExperiencedDevs, «Spec Driven Development and other shitty stuff»](https://www.reddit.com/r/ExperiencedDevs/comments/1reiro1/spec_driven_development_and_other_shitty_stuff/) · [r/ExperiencedDevs, intento de refutar a METR](https://www.reddit.com/r/ExperiencedDevs/comments/1o2b2so/we_debunked_that_experienced_devs_code_19_slower/) · [r/ChatGPTCoding, «¿alguien usa SDD?»](https://www.reddit.com/r/ChatGPTCoding/comments/1otf3xc/does_anyone_use_specdriven_development/) · [r/ChatGPTCoding, el hilo más crítico](https://www.reddit.com/r/ChatGPTCoding/comments/1o6j1yr/specdriven_development_for_ai_is_a_form_of/) · [r/LocalLLaMA, informe de dos meses con Spec Kit](https://www.reddit.com/r/LocalLLaMA/comments/1te3ehy/tried_githubs_speckit_with_claude_code_for_2/) · [r/LocalLLaMA, «me rindo con los modelos locales»](https://reddit.com/r/LocalLLaMA/comments/1sxqa2c/) · [HN de forge](https://news.ycombinator.com/item?id=48192383) · [Ask HN: stack local](https://news.ycombinator.com/item?id=44572043)

**Consultadas y sin aportar nada utilizable:** el **preprint del paper de EASE 2026**, que no está publicado (sólo se pudo leer el resumen de la conferencia); el **blog de OpenAI sobre SWE-bench Pro**, que devuelve **403 en dos rutas** (sus cifras del ~30% están respaldadas por tres medios independientes que coinciden, pero **no por lectura directa**); `swebench.com`, que sirve contenido truncado e ilegible; `tbench.ai`, que hoy sirve la versión 4.0 y no la 2.0; **Redlib**, que devolvió **429 saturado**; el informe de Faros (URL original **404**, y las cifras que circulan varían entre 22.000 desarrolladores con 4.000 equipos y 1.255 equipos — **no se han usado como dato firme**); la tesis de Porto sobre regresión visual (**403**); y el repositorio de Terragon (cerrado).

**Reddit: resuelto.** Se accedió con navegador real y se leyeron completos los tres hilos que importan. Las cifras del paper de EASE 2026 (**9,8%** pasa a «listo para revisar», **94,7%** de esas transiciones las inicia un humano) se leyeron en el resumen primario de la conferencia.

## 7. Lo relevante que no encaja en lo anterior

- **El movimiento SDD está en revisión por dentro.** El radar de Thoughtworks de abril de 2026 lo destaca, y a la vez reporta «hinchazón de instrucciones» y «podredumbre del contexto» en equipos que lo aplican. No es una crítica externa: es lo que dicen sus propios usuarios.
- **La dirección del error decide el diseño.** Casi todos los fallos de verificación documentados son falsos positivos de corrección —dar por bueno lo roto—, no falsos negativos. La excepción es la accesibilidad, donde el escáner es conservador por diseño y lo declara.
- **El engaño no lo crea la presión, lo crea ver el oráculo.** Con la instrucción «haz lo que haga falta» y el oráculo oculto, el juego sucio fue **0 de 36**. Con la función de puntuación visible, el 100% de las trayectorias de una tarea hicieron trampa. La variable es la visibilidad.
- **La investigación sobre agentes está siendo contaminada por investigación generada por agentes, y ya se puede cuantificar.** Un estudio estima que **~32% del último trimestre completo de arXiv** muestra estilo de máquina —**~65% en informática**—, contra un control previo a ChatGPT de ~0,4%. En esta investigación se han descartado **tres papers** por ese motivo, uno de ellos con autores «Tom Cat» y «Screwy Squirrel» y una fórmula limpia que resultó inventada. Cualquiera que haga este trabajo hoy tiene que verificar la **legitimidad** de la fuente, no sólo su contenido.
- **Y una advertencia sobre este mismo informe:** las cifras más citadas —SWE-bench, rivalidad entre benchmarks, engaño de recompensa— vienen en parte de organizaciones que venden evaluaciones o lideran los benchmarks. He procurado marcarlo, pero conviene leer los denominadores, no los titulares.

## Enlaces

- [[metodo-y-alcance]] — método y criterio de admisión, declarados antes de buscar
- [[crear-la-tarea]] — cómo tiene que escribirse la tarea
- [[consumir-la-tarea]] — el bucle, el disparador y la seguridad
- [[verificacion-y-oraculo]] — el oráculo y sus límites medidos
- [[robustez-desatendida]] — reintentos, cuelgues, concurrencia y el hecho de que el agente adivina en vez de preguntar
- [[lo-que-dice-la-comunidad]] — el veredicto de los que lo usan
- [[autonomia-medida]] — cifras con denominador y el peso del andamiaje
- [[gratis-y-local]] — la vía gratuita completa, con su coste real en hardware
- [[clausura-semantica]] — el marco que explica **por qué** todo lo anterior sale así
- [[contra-evidencia]] — el ataque frontal a cada conclusión: qué aguanta y qué se matiza
- [[panorama-de-herramientas]] — el barrido completo de herramientas y formatos
- [[casos-medidos]] — quién lo ha medido de verdad, y con qué números
- [[limites-del-andamiaje]] — el límite: lo que ninguna verificación automática cubre hoy
- [[piezas-y-coste]] — recuento de piezas y qué se rompe
- [[montaje-documentado]] — el montaje reproducible, no ejecutado
- [[desarrollo-autonomo-con-agentes]] — la síntesis en `areas/` del mismo dominio: qué está medido sobre desarrollar con agentes
