---
title: Lego — quién dice qué, y quién lo respalda
created: 2026-09-28
updated: 2026-09-28
tags: [lego, fuentes, auditoria, verificacion]
zona: tecnico
---

Auditoría de fuentes. Para cada afirmación que sostiene algo: **quién lo dice, quién es esa persona u organización, qué intereses tiene, y si he podido comprobarlo en la fuente primaria.** Nació de que el informe anterior citaba «ICLR 2025» un paper que estaba **rechazado** — un error que se evita abriendo la página una vez.

## Cómo leer la columna de verificación

- **TEXTO COMPLETO** — he leído el cuerpo del paper o la página entera.
- **RESUMEN** — sólo el resumen o la página de la conferencia.
- **NO VERIFICADO** — no pude llegar a la fuente primaria, y digo por qué.

## Los cuatro pilares, uno por uno

### 1. «P(todas las instrucciones) ≈ p^n» — el paper estaba RECHAZADO

| | |
|---|---|
| **Qué afirma** | Que el cumplimiento de todas las instrucciones decae como la potencia del número de ellas |
| **Quién lo dice** | Keno Harada, Yudai Yamazaki, Masachika Taniguchi, Takeshi Kojima, Yusuke Iwasawa, **Yutaka Matsuo** |
| **Quiénes son** | Grupo de la **Universidad de Tokio**. Matsuo es catedrático conocido en IA en Japón; Iwasawa y Kojima también publican en el área. **No son cualquiera** |
| **Dónde** | [OpenReview R6q67CDBCH](https://openreview.net/forum?id=R6q67CDBCH) — **submitted to ICLR 2025, NO publicado** |
| **Estado real** | **«Decision: Reject»**, 22 de enero de 2025. Lo leí en la propia página |
| **Qué dijo la metarrevisión** | *«Dudas de que ManyIFEval sea una **extensión directa** de benchmarks existentes… el conjunto se centra en instrucciones **simples y verificables binariamente, muy diferentes de las tareas reales**… el método de autorrefinado no es novedoso»*, y —textual— *«los revisores argumentaron que la “maldición de las instrucciones” **era predecible** (con lo que también estoy de acuerdo)»* |
| **Verificación** | **TEXTO COMPLETO** (página de OpenReview, decisión y metarrevisión) |
| **Veredicto** | **Mi cita estaba mal.** Escribir «ICLR 2025» y enlazar un PDF de OpenReview se lee como aceptado. **No lo estaba** |

**Y la corroboración revisada por pares lo desmiente en su forma fuerte:**

| | |
|---|---|
| **Qué afirma** | Que los modelos aguantan **muchas** instrucciones: los mejores mantienen rendimiento casi perfecto **hasta más de 150** |
| **Quién lo dice** | Daniel Jaroslawicz, Brendan Whiting, Parth Shah, Karime Maamari |
| **Quiénes son** | **[Distyl AI](https://distyl.ai)** — una empresa que vende soluciones de IA a empresas. **Interés comercial, aunque este resultado concreto no les favorece** |
| **Dónde** | [arXiv:2507.11538](https://arxiv.org/abs/2507.11538) — **NeurIPS 2025** |
| **Cifras** | **20 modelos, siete proveedores, hasta 500 instrucciones.** gemini-2.5-pro: 100% (10 instr.) → 84,8% (250) → **68,9% (500)**. claude-3.7-sonnet: 100% → 72,9% → 52,7%. gpt-4o: 94% → 22,2% → **15,4%** |
| **Tres patrones** | **Umbral** (los de razonamiento), **lineal**, y **exponencial** — éste último **es el de los modelos pequeños** |
| **Verificación** | **TEXTO COMPLETO** (arXiv HTML, tabla de resultados por modelo) |
| **Veredicto** | **Mi recomendación de «muy pocas instrucciones» era exagerada.** Los números concretos («diez dan ~35%») **quedan retirados** |

**Y el paper rechazado trae una mitigación que el informe no mencionaba:** el autorrefinado sube GPT-4o del **15% al 31%** y Claude 3.5 Sonnet del **44% al 58%**; y basta con decirle al modelo que no las está siguiendo.

### 2. «El SDD no reduce defectos» — el estudio mide otra cosa

| | |
|---|---|
| **Quién lo dice** | **Brenn Hill**, «investigador independiente», **autor único** |
| **Qué intereses** | Tiene un **libro comercial con la misma tesis** (*The Delivery Gap*). El paper es un **preprint auto-publicado**, no está en arXiv, y la conferencia (ESEIW 2026) **aún no se ha celebrado** |
| **Cómo lo publicó** | **Tres registros en Zenodo en dos días**, uno con otro número de PRs (89.599) y dos duplicados con 88.052 |
| **Qué mide de verdad** | Su versión final reconoce que mide «artefactos de especificación — abrumadoramente **referencias a tickets**— y no la salida de herramientas de SDD». **El 97,5% del tratamiento es «este PR referencia un ticket»** |
| **Auditoría de repos** | **Cero de 120** repositorios tienen directorio de herramienta SDD; tres los excluyen en `.gitignore` |
| **Metodología** | **Parada opcional** del muestreo («se añadieron datos buscando un efecto»), p-valor que **cambia entre versiones** (0,016 frente a 0,164), SZZ con **46–71% de mala atribución**, calidad puntuada por **Claude Haiku** validado en **38 PRs** |
| **Verificación** | **TEXTO COMPLETO** del PDF y **del repositorio entero** del autor (`REVISION-PLAN.md`, `main.tex`) |
| **Veredicto** | **Conclusión retirada.** Pasa de «evidencia de ausencia» a **«ausencia de evidencia»** |

### 3. «El agente no pregunta y preguntar lo hunde» — me quedé con media fuente

| | |
|---|---|
| **HiL-Bench** | [arXiv:2604.09408](https://arxiv.org/abs/2604.09408), **12 autores**, preprint. Su resumen dice **las dos cosas**: que los modelos caen al decidir si preguntar, **y** que «el entrenamiento por refuerzo… muestra que el juicio **es entrenable**: un modelo de 32B mejora la petición de ayuda y la tasa de éxito». **El informe citaba sólo la primera.** |
| **UnderSpecBench** | [arXiv:2607.02294](https://arxiv.org/abs/2607.02294). El «55,8–67,8%» aplica a las ejecuciones **que actúan**, no a todas — recorte que faltaba |
| **Ambig-SWE** | [arXiv:2502.13069](https://arxiv.org/abs/2502.13069), **ICLR 2026**. El «hasta 74%» **era real**: yo lo había descartado como no verificable y **me equivoqué al descartarlo** |
| **Veredicto** | **La mitad que quedaba se retira.** Preguntar funciona; el coste se multiplica por 2,1 |

### 4. «El andamiaje importa más que el modelo» — cité más de lo que dice la fuente

| | |
|---|---|
| **El 29,8%** | [arXiv:2609.17394](https://arxiv.org/abs/2609.17394), ADMA 2026. El resumen del propio paper dice: *«este diseño observacional **no identifica efectos causales** del andamiaje»* |
| **El contraejemplo** | [arXiv:2512.08296](https://arxiv.org/abs/2512.08296) — **Google Research + DeepMind + MIT**, **260 configuraciones** con herramientas, prompts y **cómputo igualados**: de **+80,8%** en razonamiento financiero descomponible a **−70,0%** en planificación secuencial |
| **Y el fabricante niega mi tesis** | Anthropic, en su propio blog: *«actualizar a Claude Sonnet 4 es una ganancia mayor que duplicar el presupuesto de tokens»* y «el uso de tokens por sí solo explica el 80% de la varianza». **Es fabricante, no vale como prueba, pero contradice lo que yo decía** |
| **Veredicto** | **Conclusión retirada.** El ajuste arquitectura-tarea decide |

### 5. «Casi todos los oráculos fallan» — falso, y el error va en dirección contraria

| | |
|---|---|
| **Contra: hay oráculos con cero falsos positivos** | **CLOVER** [arXiv:2310.17807](https://arxiv.org/abs/2310.17807): «tolerancia cero con las incorrectas adversariales», **0 de 60** en cuatro familias. **AutoVerus** (Microsoft Research + UIUC, **OOPSLA 2025**): 137 de 150. **TestGen-LLM** (Meta, **FSE 2024**): 73% de recomendaciones aceptadas |
| **Y el error va al revés** | La auditoría de OpenAI encontró que el **35,5%** de las tareas auditadas tienen tests demasiado estrictos que «**invalidan muchas entregas funcionalmente correctas**», bajo el epígrafe «**los tests rechazan soluciones correctas**» |
| **Verificación** | Auditoría de OpenAI leída **entera con navegador real**; el resto, en resumen o cuerpo según el caso |
| **Veredicto** | **Rebajada.** El ruido en repositorios abiertos con tests heredados sí es grande y está medido; la generalización no |

## Los segundos pilares, y quién los firma

| Afirmación | Quién la firma | Quiénes son / interés | Verificación |
|---|---|---|---|
| Los PRs de agentes reciben arreglos **1,62×** más que los humanos; **69,6%** los hace el mismo agente | [«Who Finishes the Job?», arXiv:2609.26847](https://arxiv.org/abs/2609.26847) | Preprint. Verificado con anotadores humanos y juez LLM (κ = 0,78) | RESUMEN |
| **2,09×** de rendimiento tras un mandato corporativo | [arXiv:2607.01904](https://arxiv.org/abs/2607.01904) — Hao He, Yegor Denisov-Blanch, Sanmi Koyejo, **Bogdan Vasilescu** | **Académicos de Stanford/Carnegie Mellon**, y la salvaguarda está declarada: «la empresa sólo compartió datos; **ningún empleado es autor**» | RESUMEN. **El paper más sólido del informe** |
| Devin fusiona el **43,0%** (512/1.355) | [arXiv:2607.21832](https://arxiv.org/abs/2607.21832) — Polytechnique Montréal | **Independiente**, y contradice el «67%» del fabricante | RESUMEN |
| **4,7%** de PRs de agentes autónomos en el decil superior | [LinearB](https://linearb.io/resources/ai-engineering-productivity-gap) | **Vende plataforma de métricas.** Publica denominador y admite que es correlacional | Página primaria |
| El «**~74%**» de preguntar | [Ambig-SWE, ICLR 2026](https://arxiv.org/abs/2502.13069) | Revisado por pares. **Yo lo había descartado por error** | RESUMEN |
| La **ambigüedad** hunde el resultado (73,8% → 6,7%) | [ICSE 2026, arXiv:2507.20439](https://arxiv.org/abs/2507.20439) | Revisado por pares | RESUMEN |
| El **41,77%** de fallos de agentes son de especificación | [MAST, arXiv:2503.13657](https://arxiv.org/abs/2503.13657) — 13 autores de **Berkeley y Stanford**, incl. Ion Stoica, Matei Zaharia, Dan Klein | **No consta revisado por pares** (sólo arXiv v3). **Y el 41,77% NO está en el resumen**: lo cité de una fuente secundaria. **Sin verificar**, y además MAST estudia sistemas **multiagente**, no agentes sueltos |
| «La fábrica con las luces apagadas no funciona» | **Dex, fundador de HumanLayer** | **Vende herramientas de agentes**, aunque su conclusión va contra su producto. Testimonio, **sin números publicados** | Página primaria |
| Las cifras de Salesforce (+79%), Stripe, Spotify, Uber | Los propios equipos | **Autoinformes sin denominador.** Los hilos de HN sobre Salesforce y Spotify están **vacíos** (4 y 2 puntos) | Páginas primarias |

## Lo que NO se ha podido verificar, y por qué

- **La cita textual de Boehm y Basili (2001)**: de pago. El famoso «**87,5% de los defectos vienen de requisitos**» que circula por todas partes **no tiene fuente primaria** — es folclore, y hay que dejar de citarlo.
- **El preprint del paper de EASE 2026**: no publicado; sólo el resumen.
- **Las cifras de Uber y Cursor**: sólo fuentes secundarias.
- **El «aceptado condicional» del estudio de Hill en ESEM**: **autodeclarado por el autor**; las actas no están publicadas y la conferencia aún no se ha celebrado.
- **El intervalo de confianza del 19% de METR**: no está en la fuente primaria.
- **Reddit en profundidad**: la API JSON funcionó para unos hilos y no para otros. **La pata de comunidad de tres conclusiones es sólo Hacker News.**

## La lección, y es la misma cinco veces

**Cinco conclusiones, cinco fuentes de un solo estudio cada una, y ninguna comprobada contra su versión final ni contra su estado de aceptación.** No fue un error de búsqueda: fue no abrir la página de la conferencia. El método que seguí lo advertía explícitamente —«una sola fuente no confirma nada»— y lo incumplí en bucle.

Lo que cambia a partir de aquí: **ninguna afirmación entra sin dos fuentes independientes y sin comprobar el estado de publicación.**

## Enlaces

- [[contra-evidencia]] — el ataque a las cinco conclusiones
- [[investigacion-lego]] — el informe, con su aviso de corrección
- [[crear-la-tarea]] — donde estaban los dos primeros pilares
