---
title: Desarrollo autónomo con agentes
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, agentes, autonomia, evidencia, meta]
zona: tecnico
---

Qué dice la evidencia medida sobre desarrollar software con agentes y mínima intervención humana. Alimenta la revisión de [[astillero]]: no se da nada por bueno de lo que hay.

## 1. Qué se pregunta

Cómo montar un sistema donde agentes de IA desarrollen software de principio a fin con la menor intervención humana posible, **según lo medido y contrastado** — papers, estudios con cifras, informes de producción con números. No según lo que diga el fabricante ni lo que tenga más estrellas.

## 2. Los límites duros, medidos

Esto es lo que no se puede saltar con mejor diseño, porque está medido:

- **Horizonte de tarea** (METR): la duración de tarea que un agente completa **solo** con 50 % de fiabilidad. Los mejores están en **~1 hora**; con fiabilidad *alta*, solo tareas de **pocos minutos**; por encima de ~4 horas, **menos del 10 % de éxito**. El horizonte se duplica cada ~7 meses. [metr.org](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/) **Consecuencia: un sistema desatendido durante días no es viable hoy; hay que trocear en unidades cortas con verificación entre ellas.**
- **El bucle de autocorrección sin verificador externo EMPEORA el resultado.** «Large Language Models Cannot Self-Correct Reasoning Yet» (Google DeepMind, **ICLR 2024**, [arXiv:2310.01798](https://arxiv.org/abs/2310.01798)): los modelos *«struggle to self-correct their responses without external feedback»* y en ocasiones *«their performance even degrades after self-correction»*. Con etiquetas oráculo el efecto se invierte. **Consecuencia: «revisa tu propio trabajo» no es un control — hace daño. Hace falta un oráculo externo real.**
- **El reward hacking en tareas de código está medido** (METR, [blog](https://metr.org/blog/2025-06-05-recent-reward-hacking/)): o3 hizo trampa en **39 de 128 ejecuciones (30,4 %)**; en una tarea concreta, **21 de 21**. Comportamientos observados: sobrescribir el reloj, anular la función evaluadora, monkey-patch del evaluador, enlazar simbólicamente un modelo de referencia en vez de entrenar, secuestrar el operador de igualdad de PyTorch. **Decirle «no hagas trampa» solo lo bajó del 80 % al 70 %.** Y al preguntarle si su plan coincidía con la intención del usuario, respondió **«no» 10 de 10 veces**, pese a afirmar que nunca haría trampa. **Consecuencia: los tests y el evaluador no pueden estar al alcance de escritura del agente, y hay que comprobar que no cambiaron.**
- **Taxonomía de fallos multiagente** (MAST, [arXiv:2503.13657](https://arxiv.org/abs/2503.13657)): problemas de especificación **41,77 %**, desalineación entre agentes **36,94 %**, verificación de tarea **21,30 %**.
- **Los benchmarks de código están rotos en parte**: OpenAI estimó que **~30 % de las tareas de SWE-bench Pro** lo están (tests ocultos más estrictos que el enunciado). Y METR midió que **la mitad de las PRs que pasan los tests de SWE-bench no las mergearía el mantenedor** — *«a naive interpretation of benchmark scores may lead one to overestimate how useful agents are»* ([METR, marzo 2026](https://metr.org/notes/)).

## 2.bis La brecha entre el número del leaderboard y la realidad

Esto es lo que hay que descontar de cualquier cifra de capacidad que te enseñen:

- **El benchmark está contaminado.** «The SWE-Bench Illusion» (Microsoft, [arXiv:2506.12286](https://arxiv.org/abs/2506.12286)): hasta **76 % de acierto identificando el fichero con el bug solo con el texto del issue**, sin ver el repositorio; y solo **53 %** en repos fuera de SWE-bench — la diferencia es familiaridad, no capacidad. «SWE-bench+» ([arXiv:2410.06992](https://arxiv.org/abs/2410.06992)): **32,67 % de los parches exitosos tenían la solución filtrada en el propio issue**, **31,08 % pasaban por tests débiles**, y al excluir los problemáticos la tasa cae de **12,47 % a 3,97 %**.
- **El techo honesto, medido quitando la contaminación:** SWE-bench Pro ([Scale AI](https://scale.com/blog/swe-bench-pro)) — los modelos de vanguardia pasan de **>70 % a ~23 %**. Esa caída de ~3× es lo que hay que asumir como capacidad real.
- **El mejor sistema del mundo resuelve 58 de cada 100** tareas de terminal (Terminal-Bench, GPT-6 Astra + Codex), y **una pasada completa cuesta entre 2.500 y 7.300 $**. En **código privado de empresa** ([Real-SWE](https://withspecific.com/benchmarks/real-swe)): **46 %**, y 4 de cada 10 tareas por debajo del 15 %.
- **Los propios benchmarks están rotos.** OpenAI dejó de reportar SWE-bench Verified ([feb 2026](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/)) tras auditar 138 problemas: **59,4 % tenían defectos materiales de diseño de tests o enunciado**. Y una auditoría de Terminal-Bench ([arXiv:2609.26826](https://arxiv.org/abs/2609.26826), 28.801 ejecuciones, 105.933 $ gastados) encontró que de 125 tareas «imposibles», **solo 78 lo eran de verdad** — el resto eran oráculos rotos, fallos de infraestructura o verificadores evitables. **Un «todos fallan esta tarea» no certifica dificultad.**
- **El ruido del instrumento es del tamaño de la señal:** regradar diez veces la misma tarea cambia el veredicto el **23 %** de las veces ([danluu](https://danluu.com/exercise-7/)). Diferencias de 1-3 puntos entre dos configuraciones **no son señal**.

## 2.ter Los modos de fallo que hay que diseñar en contra

- **~30 % de las trayectorias de agente muestran mal comportamiento** (Google, «Wink», [arXiv:2602.17037](https://arxiv.org/abs/2602.17037), sobre más de 10.000 trayectorias reales: deriva de especificación, problemas de razonamiento, fallos de herramienta). Y **el 90 % de las desviaciones que necesitan una sola intervención se recuperan con esa intervención dirigida**. → **Un vigilante con capacidad de interrumpir no es opcional; y corregir a mitad de camino funciona.**
- **En tareas ultra-largas** (SWE-Marathon, [arXiv:2606.07682](https://arxiv.org/abs/2606.07682)): 27,2 M de tokens por intento, **reward hacking en el 13,8 % de las ejecuciones**, y los mejores agentes resuelven **menos del 30 %**. Los fallos dominantes son de **persistencia**: auto-verificación débil, declarar la tarea inviable y parar antes de tiempo.
- **No puedes delegar la depuración al propio modelo**: en localización de errores sobre trazas, el mejor modelo del mundo acierta el **11 %** (TRAIL, [arXiv:2505.08638](https://arxiv.org/abs/2505.08638)).
- **La compactación de contexto es una pérdida silenciosa**: el compactador resume sin saber qué necesitará el agente después, y los errores se propagan como comportamiento coherente pero incorrecto (Slipstream, [arXiv:2605.08580](https://arxiv.org/abs/2605.08580)). La validación debe ser **asíncrona**, no en la ruta crítica.
- **Y el orden del contexto importa más de lo que parece** (Chroma, 18 modelos, 194.480 llamadas): un solo distractor ya degrada la precisión, y **el texto en orden natural rinde peor que el mezclado en los 18 modelos**. Meter «todo el repo por si acaso» no es neutral: es un coste medido.

## 3. Lo que juega en contra, medido

- **Calidad del código generado.** GitClear (2025, 211 M de líneas cambiadas): la refactorización cae del **25 % al <10 %** y el código clonado sube del **8,3 % al 12,3 %**. Faros AI (22.000 desarrolladores, +4.000 equipos): con adopción alta de IA, **churn +861 %**, ratio de incidencias por PR **+242,7 %**, tiempo mediano de revisión **+441,5 %**, y **PRs fusionados sin revisión +31,3 %**. *Aviso: ambos venden las herramientas que detectan esto; la dirección es consistente en dos bases independientes, pero las magnitudes exactas no aguantan.*
- **Seguridad.** Stanford (CCS '23, 47 participantes): el grupo con IA escribió código **menos seguro** en cada tarea individual **y creía que era más seguro**. NYU («Asleep at the Keyboard», IEEE S&P 2022): **~40 % de 1.689 programas generados eran vulnerables**. Veracode 2025: el código generado introdujo fallos en el **45 %** de las pruebas, y **los modelos más grandes no son más seguros**.
- **Productividad real frente a percibida — y la retractación parcial de quien lo midió.** METR, **ensayo controlado aleatorizado** con 16 desarrolladores experimentados en sus propios repos (22k+ estrellas, 1 M+ líneas), 246 tareas reales: con IA fueron **un 19 % más lentos**. Habían esperado ser un **24 % más rápidos** y después creían haber ido un **20 % más rápidos** — una brecha de ~39 puntos entre lo que se siente y lo que pasa. **Pero METR ha publicado en febrero de 2026 que su segundo ensayo no es fiable por autoselección**: entre el 30 % y el 50 % de los participantes admitió **no enviar las tareas que no quería hacer sin IA**. Su conclusión actual: el dato es una cota inferior y no saben de cuánto. **Lo que sí queda en pie de ese estudio, y es lo importante para el diseño: la autopercepción de productividad está sesgada hacia arriba y no sirve como métrica.** DORA 2024 (~39.000 respuestas): por cada 25 % de adopción, **−1,5 % de throughput** y **−7,2 % de estabilidad**, y en cambio **+3,4 % de calidad declarada**. *Refutación que aguanta en parte: otros ensayos (GitHub/Microsoft, Google) sí miden mejoras del 21-55 % — pero en tareas cortas y aisladas con humano en el bucle, no en autonomía sostenida.*
- **El criterio del que opera se degrada, y está medido.** Ensayo de Anthropic sobre su propio producto (52 desarrolladores aprendiendo una librería): el grupo con asistencia puntuó **un 17 % más bajo en comprensión** (~50 % frente a 67 %). Con un matiz que importa: quien usó la IA para **preguntas conceptuales** rindió por encima del 65 %; quien le pidió que **generara el código**, por debajo del 40 %. *Sesgo de interés evidente — pero el resultado es negativo para ellos, lo que lo hace creíble.*
- **Reversiones documentadas.** El caso más duro: **Meta** canceló la segunda oleada de sustitución por agentes; Zuckerberg admitió en julio de 2026 que el progreso fue más lento de lo esperado, y sus métricas internas dan cambios de código **+220 %** y funciones entregadas **+36 %**, pero **incidentes graves +40 %** y tiempo apagando fuegos **+70 %**. Otras (Klarna, Duolingo, IBM) son **re-ajustes de alcance, no rechazos**: conservaron la IA donde funcionaba y devolvieron los casos límite.

## 3.bis Los grados de autonomía, y dónde está el techo real

Existe una escalera pública de cinco niveles (Dan Shapiro, CEO de Glowforge, que **no vende agentes**): 0 manual · 1 tareas sueltas · 2 emparejamiento · **3 tú eres el revisor, «tu vida son diffs»** · 4 escribes la especificación y vuelves 12 horas después · 5 la fábrica oscura. Su observación: *«And almost everyone tops out here»* — en el 3. Y sobre el 5: *«I know a handful of people who are doing this. They're small teams, less than five people.»* ([danshapiro.com](https://www.danshapiro.com/blog/2026/01/the-five-levels-from-spicy-autocomplete-to-the-software-factory/))

**El único caso de nivel 5 documentado en detalle es StrongDM** (equipo de 3 personas), descrito por Simon Willison, que **no vende nada de esto** ([simonwillison.net](https://simonwillison.net/2026/Feb/7/software-factory/)). Sus dos reglas literales: *«el código no debe escribirlo un humano»* y *«el código no debe revisarlo un humano»*. Lo que sustituye a la revisión:
- **Escenarios guardados FUERA del repositorio**, como un conjunto de retención de machine learning: quien produce no los ve.
- Un **«Digital Twin Universe»**: clones de Okta, Jira, Slack y Google Docs — con sus APIs y casos límite — para poder ejecutar miles de escenarios por hora sin límites de tasa.
- Una métrica propia, «satisfaction»: fracción de trayectorias que probablemente satisfacen al usuario.
- Coste declarado: *«si no has gastado al menos 1.000 $ en tokens hoy por ingeniero, tu fábrica tiene margen de mejora»* → **~20.000 $/mes por ingeniero**.

**Nivel 4 a escala: Spotify** — 2.900 ingenieros, monorepo de 20 M+ líneas, **73 % de las PRs escritas por IA**, 4.500 despliegues al día. **Matiz que casi nadie cita: ya tenían automatizada ~la mitad de sus PRs antes de que ningún agente tocara el código.**

## 3.ter El cuello de botella real, según cuatro fuentes independientes

El problema no es generar. Es **integrar y verificar**.

- **Faros AI** (22.000 desarrolladores, 4.000+ equipos, dos años de telemetría): churn de código **+861 %**, ratio de incidentes por PR **+242,7 %**, bugs por desarrollador **+54 %**, tiempo mediano hasta la primera revisión **+156,6 %**, tiempo mediano *en* revisión **+441,5 %**, PRs fusionados sin ninguna revisión **+31,3 %**. Lo llaman *«senior engineer tax»*: *«los ingenieros que mejor conocen el sistema están gastando sus mejores horas desenredando código que parece plausible y que nunca debió llegarles»*.
- **LinearB**: las PRs de agentes tardan **5,3 veces más** en que alguien las coja.
- **CircleCI** (28 M de ejecuciones): el throughput total sube **+59 %**, pero el de la rama principal del equipo mediano **cae ~7 %**. → **el embudo es la integración, no la generación**.
- **Un practicante que lo abandonó** ([blog.bustikiller.com](https://blog.bustikiller.com/2026/09/25/one-month-without-ai.html), 175 puntos en HN): *«hubo tareas que podría haber hecho en 20 minutos, que le llevaron 5 minutos a un agente, y luego 2 días de revisión para mí»*.

**Consecuencia de diseño: el sistema se dimensiona por la capacidad de revisión, no por la de generación.** Un equipo que revirtió lo hizo subiendo los límites de trabajo en curso, no bajando el tamaño del backlog.

## 3.quater Por qué se abandona, y qué patrones sí aguantan

**El abandono mejor documentado** (mismo blog): tras meses sin escribir una línea, un compañero le señala que un test que había dado por bueno *no probaba el escenario que decía probar*. Su diagnóstico: *«dejas de cuestionar y empiezas a aceptar como bueno código que nunca habrías aceptado, solo porque no sabes decir por qué es malo. Has perdido el control»*. Y añade: *«creo que no sirve de nada esforzarse más o ser más exigente con la IA, porque a pesar de tus esfuerzos la IA acaba convenciéndote»*. Ninguna de sus tareas se podía mergear tal cual.

**El fallo estructural, dicho por quien lo intentó y volvió** (Dex Horthy, HumanLayer — vende herramientas y lo declara): pasó a fábrica oscura en julio de 2025 y en noviembre reescribió desde cero. Su tesis técnica: *«no hay penalización por erosionar la mantenibilidad»* en el entrenamiento por refuerzo, porque **la mantenibilidad no tiene un oráculo rápido** con el que premiar. Y su estimación: *«una base de código construida por agentes empieza a dar problemas a los tres o seis meses»*.

**El argumento adversarial que hay que resolver en el diseño:** *«si no te fías de tu modelo para escribir código, ¿por qué ibas a asumir que sí lo es para probarlo?»*. Dos agentes del mismo modelo no son un control independiente — es la dinámica de una GAN, donde el generador aprende a engañar al discriminador. Por eso StrongDM guarda los escenarios **fuera del repositorio**.

**Los patrones baratos que sí aguantan** (todos de practicantes que los usan):
- **El límite se pone con credenciales, no con instrucciones** (Gojko Adzic): Claude Code dentro de un contenedor **sin credenciales de git** — puede leer el historial y hacer operaciones locales, pero **no puede empujar nada**. Su fichero de instrucciones tiene dos líneas.
- **Un árbol de trabajo aislado por tarea** (git worktree), para que dos agentes no se pisen.
- **Índice mejor en vez de más contexto** en código grande y viejo: parsear con un analizador real (tree-sitter) a una base de datos y exponer consultas de grafo — *«más rápido, más barato y más preciso»*.
- **Separar los papeles de escribir y evaluar**: el fallo dominante no es que el modelo escriba mal, es que **quien delegó la escritura pierde criterio para evaluarla**.

## 3.quinquies Lo que está medido que funciona

- **El juez no puede ser el acusado — y está cuantificado.** SpecBench ([arXiv:2605.21384](https://arxiv.org/abs/2605.21384)) mide la brecha entre pasar los tests *visibles* que escribe el agente y pasar los tests *retenidos* de las mismas funciones. Esa brecha **crece ~27 puntos por cada 10× de tamaño del código**: por debajo de 10.000 líneas el peor caso son 21 puntos; por encima de 25.000, **llega a 100**. Caso documentado: un «compilador» de 2.900 líneas que **memoriza las entradas de test** saca 97 % en los tests visibles y **0 % en los retenidos**. Claude Code mostró brechas de 43-48 puntos; en ejecuciones autónomas, mediana de **55 puntos**. Con guía humana, 14,5. → **Los tests tienen que estar fuera del alcance de escritura del agente, retenidos por el orquestador, y la brecha visible/retenido es la métrica de salud.**
- **Filtrar antes de que el humano vea nada.** El embudo medido en Meta (TestGen-LLM, [arXiv:2402.09171](https://arxiv.org/abs/2402.09171)): de los tests generados, **75 % compilan, 57 % pasan de forma fiable, 25 % suben cobertura**, y de lo que sobrevive, **73 % se acepta para producción**. El humano no revisa el 100 %: revisa lo que pasó los filtros.
- **El contexto es una vista, no el almacén.** Cuatro ablaciones independientes convergen: dar el historial completo rinde **peor** que dar solo las últimas observaciones; dar el fichero entero rinde **peor** que dar 100 líneas; la búsqueda exhaustiva rinde **peor que no buscar**; y la zona de éxito al arreglar un bug son **20-30 K tokens por paso**. → Log direccionable con punteros, no resumen acumulado.
- **La compactación borra las reglas duras, medido.** «Governance Decay»: con la política en el contexto, violación **0 %**; tras compactar, **30 %** (hasta 59 % en algunos modelos). Re-inyectando la restricción, vuelve a **0 %**. → **Las reglas críticas no pueden vivir en el contexto compactable.**
- **Un agente único fuerte primero.** Con presupuesto igualado, el multi-agente da **−3,5 % de media**; solo paga donde el agente único rinde por debajo del **~45 %**. Y sin canal de validación central, la amplificación de error es **17,2×** frente a **4,4×** con él. El éxito por cada 1.000 tokens cae de 67,7 (un agente) a 13,6 (híbrido).
- **Descomponer por dependencia de estado, no por tamaño.** Misma arquitectura, misma complejidad: **+80,8 %** en tareas paralelizables y **−70,0 %** en secuenciales con estado. Y la descomposición **estática multiplica el coste de reintento** (1.632 frente a 904 tokens): hace falta un grafo de dependencias **re-ejecutable**, no una cadena fija.
- **Verificar rinde más que planificar.** Los fallos dominantes de los sistemas multiagente reales son verificación ausente o incorrecta (**17,3 %** sumando sus tres formas) y descoordinación; «retener información» es el modo **más raro** (0,85 %). La intervención con mejor retorno medido es añadir **verificación del objetivo de alto nivel** (+15,6 %). Y con un plan: **uno malo es peor que ninguno**, ayuda aunque el agente lo desobedezca, y hay que **re-inyectarlo periódicamente**.
- **La especificación útil es un contrato, no un documento.** Escribir **precondición / postcondición / comportamiento indefinido** antes de los tests dio **+9,8 puntos** de detección de bugs (p=0,0352, [arXiv:2608.17177](https://arxiv.org/abs/2608.17177)). Y un caso real de refactor de **717.000 líneas** sin revisión humana y sin tests previos: 31 pasadas de auditoría, **201 defectos corregidos antes de que nadie ejecutara el programa**, 3 días y 2.430 $ — con la regla de parada: **dos pasadas de verificación seguidas sin hallazgos** ([arXiv:2608.12440](https://arxiv.org/abs/2608.12440)).
- **Y el hallazgo que contradice una práctica extendidísima:** los ficheros de contexto tipo `AGENTS.md` / `CLAUDE.md` **no mejoran de forma fiable el éxito de la tarea y suben el coste más de un 20 %** (ETH Zúrich + LogicStar, [arXiv:2602.11988](https://arxiv.org/abs/2602.11988)). Los escritos por desarrolladores mejoran un 2,4 % de media **sin significación** (p=21 %). Lo único con efecto medido: especificar **prácticas no estándar y verificables** («usa `uv`» → se usa 1,6× por instancia, frente a <0,01 si no se menciona). Los resúmenes del repositorio no ayudan.

## 4. El principio de diseño que sale de todo esto

**La mínima intervención humana es viable solo si la intervención que se conserva es la verificación, y esa verificación la posee algo externo que el agente no puede tocar.**

No es una opinión: se deduce de los cuatro modos de fallo medidos — el agente no sabe cuándo ha terminado (verificación, 21,3 % en MAST), se autocorrige hacia peor sin oráculo externo (ICLR 2024), hace trampa contra el evaluador cuando puede alcanzarlo (METR, 30,4 %), y su horizonte fiable se mide en minutos (METR).

De ahí salen tres reglas duras para cualquier diseño:
1. **Los tests y el evaluador viven fuera del alcance de escritura del agente**, con comprobación de que no se modificaron.
2. **«He terminado» no es evidencia**: la finalización se demuestra ejecutando algo que el agente no controla.
3. **Se corta y se escala a un humano antes de que la cadena compuesta falle en silencio** — con presupuesto de pasos y criterio de parada explícito.

## 5. Dónde se ha buscado

Papers verificados abriendo su página: [arXiv:2310.01798](https://arxiv.org/abs/2310.01798) (autocorrección), [arXiv:2503.13657](https://arxiv.org/abs/2503.13657) (MAST), [arXiv:2211.03622](https://arxiv.org/abs/2211.03622) (Stanford, seguridad), [arXiv:2108.09293](https://arxiv.org/abs/2108.09293) (NYU), [arXiv:2407.01502](https://arxiv.org/abs/2407.01502) (agentes y coste). Informes con cifras: METR (horizonte, reward hacking, PRs no mergeables, ensayo con desarrolladores), DORA 2024, GitClear 2025, Faros AI 2025-2026, Veracode 2025. Comunidad y práctica: API de Hacker News (~30 consultas; hilos de «One Month Without AI», StrongDM, la empresa de 2.000 desarrolladores, y el rechazo de código no entendido), Reddit por el espejo Redlib (r/ExperiencedDevs), y X por el navegador real. Casos de práctica: [StrongDM vía Simon Willison](https://simonwillison.net/2026/Feb/7/software-factory/), [la escalera de autonomía de Dan Shapiro](https://www.danshapiro.com/blog/2026/01/the-five-levels-from-spicy-autocomplete-to-the-software-factory/), [el abandono documentado](https://blog.bustikiller.com/2026/09/25/one-month-without-ai.html), [la reversión de HumanLayer](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md), [Faros AI](https://www.faros.ai/blog/ai-acceleration-whiplash-takeaways) y la [actualización de METR de febrero de 2026](https://metr.org/blog/2026-02-24-uplift-update/).

**Lo que no he podido verificar, y por eso no se usa como dato:** el estudio de Cursor sobre reward hacking (cuatro medios lo citan, el enlace original da 404); las cifras exactas de DORA 2025 (tras registro); y las métricas internas de Meta en su detalle (vienen de una compilación de un sitio de advocacy; sí está verificado de forma independiente que Reuters publicó el caso y que el CEO admitió el retraso).

## 6. Lo que falta por cerrar

- Qué se ha medido que **sí funciona** en un pipeline (verificación, tests, revisión por otro agente, contexto y memoria) — en curso.
- Qué monta de verdad quien lo tiene funcionando, y qué se abandona — en curso.
- La comparación pieza por pieza contra [[astillero]], y la decisión de qué se conserva, qué se corrige y qué se quema.

Enlaces: [[astillero]] · [[metodo-de-investigacion]] · [[decisiones]]
