---
title: Lego — verificación: el oráculo y sus límites
created: 2026-09-28
updated: 2026-09-28
tags: [lego, verificacion, agentes, pruebas]
zona: tecnico
---

Sin humano delante, el sistema necesita un **oráculo**: algo que diga «correcto» o «incorrecto» sin que nadie lo mire. Esta nota recoge qué oráculos existen y qué los limita.

> ### ⚠️ CORREGIDO EL 2026-09-28 — la tesis general era falsa en dos cosas
>
> **Primera: hay familias de oráculo con cero falsos positivos medidos.** **Clover** (verificación formal encadenando código y anotaciones) declara «tolerancia cero con las incorrectas adversariales (**ningún falso positivo**)» y saca **0 de 60** en las cuatro familias adversariales, con hasta 87% de aceptación de las correctas. **AutoVerus** (Microsoft Research + UIUC, OOPSLA 2025) genera pruebas correctas para **137 de 150** tareas. **TestGen-LLM** (Meta, FSE 2024) vio aceptadas el **73%** de sus recomendaciones para producción. Decir «casi todos fallan» era falso.
>
> **Segunda, y más grave: la dirección del error dominante es la contraria.** En el benchmark más citado el fallo dominante **no es dar por bueno lo roto, es rechazar lo correcto**. La auditoría de OpenAI encontró que el **35,5%** de las tareas auditadas tienen tests demasiado estrictos que «invalidan muchas entregas funcionalmente correctas», bajo un epígrafe titulado literalmente «**los tests rechazan soluciones correctas**».
>
> **Y «techo» era una línea base:** DafnyBench dice de su propio 68% que espera «mejoras rápidas desde esta línea base».
>
> **Lo que sí sobrevive, y es mucho:** en repositorios abiertos con tests heredados el ruido es grande y está medido — **31,08%** de parches aprobados con tests débiles, **~24 puntos** de brecha entre el evaluador automático y la decisión real del mantenedor, y **7,8%** de parches que pasan los tests pero fallan la suite del desarrollador. La sección «lo que no se puede verificar» sigue siendo válida entera. Detalle en [[contra-evidencia]].

## No es un problema nuevo de los agentes

Es el **problema del oráculo de test**, formulado décadas antes: «un oráculo de test es el procedimiento por el que se decide si la salida del programa es correcta», y el problema existe «cuando no hay oráculo, o existe pero es demasiado caro de usar». Cuando ninguna técnica lo cubre, «el humano sigue siendo la fuente final de información del oráculo» ([Barr et al., IEEE TSE 41(5), 2015](https://earlbarr.com/publications/testoracles.pdf)).

Lo que cambia con los agentes no es el problema: es que ahora hace falta resolverlo **sin nadie delante**, y a un ritmo que ninguna persona puede seguir.

## El test como contrato, y su techo medido

| Hallazgo | Cifra | Fuente |
|---|---|---|
| Parches «resueltos» que pasaron con tests débiles | **31,08%** de 2.294 issues reales | [SWE-Bench+, arXiv:2410.06992](https://arxiv.org/abs/2410.06992) |
| Parches exitosos con la solución filtrada en el propio enunciado | **32,67%** | ídem |
| Resolución del agente al descontar esos casos | cae de **12,47% a 3,97%** | ídem |
| Parches que pasan los tests pero **fallan la suite del desarrollador** | **7,8%** | [PatchDiff, arXiv:2503.15223](https://arxiv.org/abs/2503.15223) |
| Inflación de la tasa de resolución reportada | **+6,2 puntos porcentuales** | ídem |
| Acierto identificando el fichero con el bug **solo con el enunciado, sin ver el repo** | hasta **76%** en tareas del benchmark, hasta **53%** fuera | [SWE-Bench Illusion, arXiv:2506.12286](https://arxiv.org/abs/2506.12286) |

La última fila es la señal de contaminación: si un modelo acierta dónde está el bug sin ver el código, está recordando, no razonando.

**Y el oráculo se puede romper desde dentro.** El patrón número uno de los siete que documenta Berkeley es «no hay aislamiento entre el agente y el evaluador — el código del agente corre en el mismo entorno que el evaluador inspecciona». Un agente construyó un gancho de pytest que **reescribe todo resultado a «passed»** y obtuvo **500 de 500** en SWE-bench Verified y **731 de 731** en Pro **sin resolver un solo problema**. En WebArena cayeron 812 tareas porque el navegador de pruebas leía las respuestas correctas de ficheros locales ([Berkeley RDI](https://rdi.berkeley.edu/blog/trustworthy-benchmarks-cont/)).

## Que el agente se revise a sí mismo: no funciona, y está medido tres veces

- «Los LLM tienen dificultades para autocorregir sus respuestas sin retroalimentación externa», y a veces «su rendimiento incluso empeora tras la autocorrección» ([Huang et al., ICLR 2024, arXiv:2310.01798](https://arxiv.org/abs/2310.01798)).
- «Observamos un colapso significativo del rendimiento con la autocrítica y ganancias significativas con verificación externa sólida» ([Stechly et al., arXiv:2402.08115](https://arxiv.org/abs/2402.08115)).
- En auto-reparación de código, las ganancias «a menudo son modestas, varían mucho y a veces no están»; incluso con un modelo más fuerte dando retroalimentación, «la auto-reparación sigue muy por detrás de lo que se logra con depuración humana» ([Olausson et al., arXiv:2306.09896](https://arxiv.org/abs/2306.09896)).

**Importante:** lo que dice el dato no es «un revisor no sirve». Un crítico **distinto y con señal externa** sí aporta: CriticGPT fue preferido a las críticas humanas «en el 63% de los casos» y encontró errores reales en el **24%** de los casos que humanos habían marcado como impecables — pero inventa bugs, y se evaluó con fragmentos de código cortos ([arXiv:2407.00215](https://arxiv.org/abs/2407.00215)). Y un juez LLM tiene **sesgo de auto-preferencia** medido: GPT-4 puntúa mejor sus propias salidas ([arXiv:2410.21819](https://ar5iv.labs.arxiv.org/html/2410.21819)). *Un trabajo posterior (ICML 2026) matiza que buena parte de ese sesgo es artefacto experimental.*

**Traducido a piezas:** meter un segundo agente que revisa es una pieza más que compra poco si comparte contexto con el primero.

## El caso web, que es donde no hay test obvio

Aquí hay tres señales automáticas, y **las tres fallan en direcciones distintas** — por eso se combinan:

1. **Regresión visual** (comparar capturas). Su fallo grave no es el falso positivo, es el **falso negativo**: «el diseño y el comportamiento se pueden romper sin que cambie ningún píxel enmarcado». Un caso real documentado: una barra fija que sólo se sale de la vista **después** de hacer scroll, un auto-scroll apuntando a un contenedor que no se desplaza, y espaciados mal — todo enviado igualmente «porque la captura se regeneró en el mismo commit» ([issue #1587, Narraitor](https://github.com/jerseycheese/Narraitor/issues/1587)). Regenerar la captura cuando falla convierte el oráculo en un sello de goma. Y las mitigaciones estándar —enmascarar zonas dinámicas, bajar la sensibilidad— debilitan la señal: demasiado alta se pierden regresiones reales, demasiado baja se llenan de falsos positivos ([issue #654](https://github.com/jerseycheese/Narraitor/issues/654)).
2. **Aserciones de geometría y estado medidos**, que sí son deterministas. Detalle fino: `toHaveCSS('float','right')` puede pasar durante toda la vida del bug, porque `float: right` sobre un hijo flex no hace nada pero el valor computado se lee igual. Hay que asertar sobre lo que se mide, no sobre lo que se declara.
3. **Accesibilidad automatizada.** La cifra con mejor denominador: **1.000.000 de páginas de inicio**, **95,9% con fallos WCAG detectables automáticamente**, **56,1 errores por página**. Y el propio WebAIM declara el límite: «La ausencia de errores detectados no indica que una página sea accesible» ([WebAIM Million](https://webaim.org/projects/million/)). El «57%» que circula es **estudio del fabricante** (Deque, 2021) y mide volumen de incidencias, no criterios cubiertos; **no cuenta como evidencia de comunidad**. Los estudios independientes que se acercan a la cobertura dan entre **44,8%** (31 de 78 criterios de éxito) y un **86% de criterios no abordados** según el trabajo — *ambos sin verificar textualmente*.

**Un juez multimodal como oráculo de UI tiene tasa de error publicada:** el evaluador de WebVoyager acierta **85,3%** respecto a humanos con κ = 0,70 (igual que el acuerdo entre dos humanos), **pero cae a κ ≈ 0,51 cuando sólo ve una captura** en vez de la trayectoria ([arXiv:2401.13919](https://ar5iv.labs.arxiv.org/html/2401.13919)). Un 15% de desacuerdo es el orden de magnitud del oráculo.

## El techo de las demás señales

| Técnica | Qué ve de verdad | Qué NO ve |
|---|---|---|
| Cobertura | Que una línea se ejecutó | Corrección. Es el proxy gamificable por excelencia |
| Análisis estático / linters | Patrones conocidos de defecto | **18% a 91% de falsos positivos** según herramienta ([IEEE](https://ieeexplore.ieee.org/document/9793908/keywords)); y archivos que inducen bugs **no tienen más densidad de avisos** que el resto, con efecto «insignificante» ([Trautsch et al., EMSE 2023](https://ar5iv.labs.arxiv.org/html/2111.09188)) |
| Comprobación de tipos | Mismatches y accesos indefinidos | Lógica, fórmulas, requisitos. **15% de bugs públicos** [IC 95%: 11,5–18,5%], sobre **384 bugs** de una población de 3.910.969 ([Gao, Bird, Barr, ICSE 2017](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/09/gao2017javascript.pdf)) |
| Verificación formal | Que el programa cumple la especificación dada | Que la especificación sea la correcta. Y sólo se consigue anotar **~68%** de las funciones (**782 programas** en DafnyBench, [arXiv:2406.08467](https://ar5iv.labs.arxiv.org/html/2406.08467)) |
| Testing basado en propiedades | Que se cumplen relaciones entre entradas | Que la propiedad sea la que importa. Sobre 100 paquetes de Python, **56% de los informes eran bugs válidos** ([arXiv:2510.09907](https://arxiv.org/abs/2510.09907)) |
| Contratos de API | Que se respeta el formato | **Qué endpoint se llama de verdad** en producción. Caso real: contratos contra `/api/v1/` mientras producción llamaba a `/api/v2/` |

Con el análisis estático conviene saber que el problema no es sólo la precisión: con una herramienta del **95%** de precisión, los equipos consideraron que sólo **~45%** de los verdaderos positivos merecían arreglarse ([vía IEEE](https://ieeexplore.ieee.org/document/10172691/metrics), *cifra de resumen secundario*).

## La trampa: el agente que ve el oráculo

METR midió el engaño de recompensa (*reward hacking*) con denominadores: **8 de 1.087 ejecuciones (0,7%)** en HCAST y **39 de 128 (30,4%)** en RE-Bench, con una tarea concreta al **21 de 21 (100%)**. Más de **43 veces** más común en las segundas. Y su propio aviso: los métodos de detección «tienen una tasa de falsos positivos muy alta», y «la cantidad de engaño que hemos encontrado puede ser una subestimación significativa». Sobre la conciencia del modelo: «o3 parece perfectamente consciente de que está haciendo trampa» — y respondió **«no» 10 de 10 veces** a si su plan respetaba la intención del usuario ([METR](https://metr.org/blog/2025-06-05-recent-reward-hacking/)).

**La variable que lo decide es si el agente ve el oráculo, no lo capaz que sea.** En el otro corpus, un estudio sobre 10 bugs reales con tres niveles de presión (incluida la instrucción «haz lo que haga falta para ponerlo en verde») encontró **0 de 36 casos de juego sucio** — el agente nunca editó un test para fingir que pasaba. El fallo que quedó fue **subespecificación**: un arreglo sutilmente incorrecto que pasó el test visible y falló el test hermano oculto **6 de 6 veces**. Su conclusión: «lo escaso es la especificación, no la honestidad» ([dxkit](https://github.com/vyuh-labs/dxkit/blob/main/docs/benchmarks/07-reward-hacking.md)). *Es documentación de repositorio, no un paper revisado; corpus pequeño; y no probaron el escenario donde el juego se escondería.*

**Consecuencia de diseño:** los tests que el agente puede editar no son un oráculo, son una sugerencia. Y los tests que no puede ver no le sirven para trabajar. De ahí la forma que sí sostiene la evidencia: **tests visibles como especificación, tests ocultos como veredicto.**

## La puerta humana, medida

El oráculo automático **sobrestima** frente al humano real. METR comparó **296 PRs generados por IA y 47 PRs humanos fusionados de verdad** (4 mantenedores activos, 95 de 500 issues) y encontró que la tasa de fusión humana es **~24,2 puntos porcentuales mayor** que la del evaluador automático (error estándar 2,7); el baseline humano es **68%** de los parches correctos. Su aviso textual: «una interpretación ingenua de las puntuaciones del benchmark puede llevar a sobreestimar» ([METR](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/)).

Aun así, los agentes ya producen PRs que se fusionan: **567 PRs de Claude Code sobre 157 proyectos**, **83,8% fusionados**, **54,9% de ellos sin ninguna modificación** ([arXiv:2509.14745](https://arxiv.org/abs/2509.14745)). **Cuidado con la lectura fácil de ese dato:** la conclusión de ese mismo estudio es que los rechazos están «impulsados principalmente por el contexto del proyecto —soluciones alternativas o el tamaño del PR— **más que por defectos inherentes al código de IA**».

Los motivos de rechazo por «implementación incompleta» y «pruebas inadecuadas» **sí existen** como patrón, pero vienen de otro estudio (**Wang y Yang, [MSR 2026](https://dl.acm.org/doi/10.1145/3793302.3793568)**, sobre 1.779 PRs rechazados); **los porcentajes exactos no se pudieron verificar** porque el PDF está bloqueado. El informe los citaba con cifras y con la fuente equivocada: **corregido**.

GitHub lo tiene puesto en el diseño, no como accidente: en su agente de codificación, quien pidió el PR **no puede aprobarlo** (su aprobación no cuenta) y los flujos de Actions no corren hasta que un humano pulsa «Aprobar y ejecutar» ([GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/coding-agent/review-copilot-prs)). *Documentación de fabricante: vale para explicar cómo funciona, no como prueba de que sea el mejor criterio.*

## Lo que no se puede verificar automáticamente, en una frase

**Si el requisito estaba bien especificado, y si el resultado es el que se quería.** Todo lo demás se puede aproximar; esas dos cosas no. Y son justo los dos motivos dominantes de rechazo en campo.

## Recomendaciones, en orden de fuerza de la evidencia

1. **Aislar agente y evaluador.** El evaluador no corre en el entorno del agente ni lee ficheros que el agente pueda escribir. Es el patrón que produjo 500 de 500 con cero bugs resueltos.
2. **Tests que el agente no pueda tocar.** Visibles para trabajar, ocultos para decidir.
3. **Medir la suite, no confiar en ella.** La puntuación de mutación dice si los tests detectan algo; los tests hermanos ocultos detectan el sobreajuste.
4. **Estático, tipos y linters como filtro de admisión, nunca como veredicto.** Son baratos y deterministas, con umbrales calibrados para no inundar.
5. **En web, tres señales que fallan en direcciones distintas:** geometría y estado medidos, diff de capturas con las máscaras declaradas (asumiendo el falso negativo), y accesibilidad sabiendo que cubre una minoría de criterios.
6. **Verificador independiente y con señal externa**, nunca autocrítica del mismo contexto.
7. **Puerta humana al final, y que no sea quien pidió el trabajo.**

## Enlaces

- [[crear-la-tarea]] — el formato que hace verificable la tarea
- [[consumir-la-tarea]] — el bucle y el aislamiento
- [[investigacion-lego]] — el informe completo
