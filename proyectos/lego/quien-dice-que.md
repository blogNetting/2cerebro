---
title: Lego — quién dice qué, y quién lo respalda
created: 2026-09-28
updated: 2026-09-28
tags: [lego, fuentes, auditoria, verificacion]
zona: tecnico
---

Auditoría de las 11 fuentes que sostienen el informe: **quién firma cada cosa, quiénes son, qué intereses tienen, si está revisado por pares, y si el texto completo respalda la cifra.** Nació de que el informe citaba «ICLR 2025» un paper **rechazado** — un error que se evita abriendo la página una vez.

**Niveles de lectura:** **[cuerpo]** = texto completo · **[resumen]** = sólo el abstract · **[no leído]** = no se pudo llegar.

## Tabla maestra

| Fuente | Quién firma | Revisado por pares | Interés | ¿Respalda la cifra? |
|---|---|---|---|---|
| **Curse of Instructions** | Harada, Yamazaki, Taniguchi, Kojima, Iwasawa, Matsuo | **NO — RECHAZADO en ICLR 2025** | — | Sólo resumen; **sin afiliación en el paper** |
| **IFScale** ([2507.11538](https://arxiv.org/abs/2507.11538)) | Jaroslawicz, Whiting, Shah, Maamari — **Distyl AI** | **Sí — NeurIPS 2025** | Empresa de IA para empresas | **Sí** [cuerpo]. 20 modelos, 7 proveedores |
| **HiL-Bench** ([2604.09408](https://arxiv.org/abs/2604.09408)) | 12 autores, **Scale.AI** | No (preprint v1→v4) | **Fuerte: venden evaluación y datos de RL** | pass@3 y Ask-F1 ✓, **pero el «humano» es un LLM congelado** |
| **UnderSpecBench** ([2607.02294](https://arxiv.org/abs/2607.02294)) | Ji, Zhang, Xu, Tian, Li, Gao, Wang, Cheung — **HKUST + Tongji** | No (v1 única) | No | ✓ **pero sólo en «acted runs»**, y el resumen omite el matiz |
| **Ambig-SWE** ([2502.13069](https://arxiv.org/abs/2502.13069)) | Vijayvargiya, Zhou, Yerukola, Sap, **Neubig** — **CMU LTI** | **Sí — ICLR 2026** | No | El 74% ✓ en prosa, **sin tabla que lo derive, y añadido en v2** |
| **Scaling Agent Systems** ([2512.08296](https://arxiv.org/abs/2512.08296)) | Kim, Gu, Park… — **Google Research + DeepMind + MIT** | No (v1→v3) | Autores de Google | **Sí, pero SWE-bench con sólo 20 instancias y ±20 pp** |
| **When Prompts Go Wrong** ([2507.20439](https://arxiv.org/abs/2507.20439)) | Larbi, Akli, Papadakis, Bouyousfi, Cordy, **Sarro** (UCL), Le Traon | **NO VERIFICADO — «ICSE» no aparece en el PDF** | No | 73,8→6,7 ✓ [cuerpo] |
| **Hill / ESEM 2026** ([Zenodo](https://zenodo.org/records/19432099)) | **Brenn Hill**, autor único | **Sí — aceptado, ESEM 2026**, confirmado en el programa oficial | Sin financiación declarada | 97,5% ✓; **el paper dice 119 repos, no 120** |
| **Who Finishes the Job** ([2609.26847](https://arxiv.org/abs/2609.26847)) | Takerngsaksiri, Duong, Barnett — **Deakin University** | No — enviado a EMSE | Declara no tener | 1,62× ✓ **pero es una razón de probabilidades sobre una base del 4,5%** |
| **CLOVER** ([2310.17807](https://arxiv.org/abs/2310.17807)) | Sun, Sheng, Padon, Barrett — **Stanford + VMware Research** | **Sí — LNCS 2024** | Autor en laboratorio de empresa | «Cero falsos positivos» ✓; **«60 adversariales» mal: son 240** |
| **AutoVerus** ([2409.13082](https://arxiv.org/abs/2409.13082)) | Yang, Li, Misu, Yao + 9 de **Microsoft Research** | **Sí — OOPSLA 2025** | **Construyen Verus, y evalúan Verus** | 137/150 ✓ [cuerpo] |
| **El mandato 2×** ([2607.01904](https://arxiv.org/abs/2607.01904)) | He, Agarwal, Denisov-Blanch, Azaletskiy, **Koyejo**, **Vasilescu** — **CMU + Stanford** | No (preprint) | Empresa cliente anónima | 802 / 196.212 / 2,09× ✓ [cuerpo] |

## Las correcciones que salieron de auditar

**1. La «maldición de las instrucciones» está RECHAZADA, y su afiliación la inventé yo.**
[OpenReview](https://openreview.net/forum?id=R6q67CDBCH) dice **«Decision: Reject»**, 22 de enero de 2025. La metarrevisión: *«ManyIFEval se ve como una **extensión trivial** de IFEval, sin novedad significativa»* y —textual— *«los revisores argumentaron que la “maldición de las instrucciones” **era predecible** (con lo que también estoy de acuerdo)»*.
**No existe versión en arXiv** (búsqueda de dos frases: cero resultados). Y **el PDF es anónimo con metadatos vacíos: la afiliación no consta en el paper.** Yo escribí «Universidad de Tokio» — es **inferencia por historial de publicación, no un dato del paper**, y así queda corregido.

**2. La afirmación «ICSE 2026» de *When Prompts Go Wrong* no está verificada.**
La cadena «ICSE» **tiene cero ocurrencias en todo el PDF**, el campo de comentarios de arXiv está vacío y sólo hay una versión. **No es que sea falso: es que la fuente primaria no lo respalda.** Queda como **sin verificar**.

**3. HiL-Bench lo firma Scale.AI, y su «humano» no es humano.**
Los doce autores firman con correo `@scale.com`. Scale **vende servicios de evaluación de agentes y datos de refuerzo**, y el paper presenta un benchmark *y* un modelo entrenado con su propia recompensa — **sin declaración de conflicto en el PDF**. Y el detalle que más cambia la lectura: **`ask_human()` lo responde un Llama-3.3-70B congelado**, no una persona. Mide preguntar a un simulador.

**4. El dato de SWE-bench del paper de Google se apoya en 20 instancias.**
Las 260 configuraciones son ciertas, pero la celda de SWE-bench Verified usa **20 instancias** (de 500, con semilla fija) y un intervalo de confianza de **±20 puntos porcentuales**. La conclusión «todas las arquitecturas multiagente degradan» se sostiene sobre eso. **Y las cifras de cabecera cambiaron entre versiones: v1 tenía 180 configuraciones, cuatro benchmarks y R² = 0,513; v3 tiene 260, seis y R² = 0,373.**

**5. El estudio de Hill está aceptado de verdad — y tiene erratas publicadas.**
El **programa oficial de ESEIW 2026** lo lista como charla de 15 minutos el lunes 5 de octubre, track *Software Engineering in Practice*. Su ORCID es real y no tiene empleo declarado. **Pero tiene un [`CORRECTIONS.md`](https://github.com/brennhill/delivery-gap-research/blob/main/CORRECTIONS.md) con dos erratas**: un fallo de pandas **eliminó 2.891 de 25.209 PRs (11,5%)** de una comparación descriptiva, y la Tabla 1 pasó de 17.973 a 17.094. El autor sostiene que ninguna conclusión cambia — y **deja marcado que la versión de Zenodo sigue sin corregir**, que es la que yo citaba.
**Y el título aceptado es otro** del que figura en el repositorio: el programa dice *«Does Spec-Driven Development Reduce Defects? An Empirical Test of Industry Claims Across 119 Open-Source Repositories»*. **119, no 120** —yo escribí 120, que es lo que tiene el manifiesto, no lo que dice el paper.

**6. El «cero falsos positivos» de CLOVER y su muestra.**
Es real **[cuerpo]**: *«no hay falsos positivos (ningún ejemplo incorrecto pasa las 6 comprobaciones)»*. Pero **las «60 muestras adversariales» están mal contadas**: son **60 programas a mano, cada uno con 4 variantes incorrectas — 240 ejemplos adversariales**. El 60 es el denominador por categoría. Y **la propia versión del paper estrechó la afirmación**: en v1 decía «tolerancia cero con las **incorrectas**», en v4 pasó a «tolerancia cero con las **adversariales incorrectas**».

**7. Los demás matices que hay que llevar puestos.**
- **UnderSpecBench**: el resumen pierde el calificador; el cuerpo dice «**acted runs**». Y la metadata de arXiv **omite a un autor** que sí firma el PDF.
- **Ambig-SWE**: el 74% es **relativo** a la condición no interactiva, aparece sólo en prosa, **y se añadió en la v2** —la v1 no lo tenía—. Y un modelo se evaluó sobre **100 de 500** instancias.
- **Who Finishes the Job**: el 1,62× es una **razón de probabilidades de arreglos *verificados***, y la base es pequeña — **sólo el 4,5%** de los PRs de agente fusionados recibe un arreglo verificado en 30 días.
- **AutoVerus**: los autores **construyen Verus** y evalúan con Verus-Bench. El propio texto se protege diciendo que no se solapan con el equipo que creó la mayoría de las tareas.

## Las fuentes de empresa: quién las firma y qué venden

**Todas las cifras verificadas contra su página primaria.** Y el patrón es constante: **casi ninguna publica denominador, y casi todas las firma quien vende.**

| Empresa | La cifra | Quién firma | Qué vende | Denominador |
|---|---|---|---|---|
| **LinearB** | 4,7% de PRs de agentes | **Andrew Zigler, ingeniero de marketing** | Analítica de ingeniería | Sí, pero **auto-seleccionado**, y el vendedor **se contradice**: 2,7 M de PRs en una página, 8,1 M en otra |
| **Salesforce** | +79% PRs/dev, +151,3% «salida» | **El presidente de ingeniería** | Agentforce | **No.** El +151,3% usa una *«puntuación de salida efectiva» propietaria definida por Salesforce** |
| **Stripe** | >1.000 PRs/semana sin código humano | Un ingeniero | Stripe | **No.** Ni total, ni porcentaje, ni base |
| **Spotify** | 1.500+ PRs, «la mitad» automatizados | Dos ingenieros | Spotify | Parcial — **y las cifras que yo citaba no existen en la fuente** |
| **Cognition / Devin** | «67% de PRs fusionados» | **El propio vendedor** | Devin | **No.** Y la medición independiente da **3 de 20 tareas** (Answer.AI) |
| **Anthropic** | «90% del código» | Declaración de su **director financiero** | Claude Code | No. Un medio lo reporta como **70–90%** |
| **Faros AI** | Incidents/PR +242,7%, bugs/dev +54% | **«Faros Research», sin autor** | Analítica de ingeniería | Sí, «telemetría, no encuestas», pero **de su propia plataforma**; su FAQ admite que «puede no generalizar» |
| **METR** | 19% más lento (RCT) | Becker, Rush, Barnes, Rein | **Nada — es un evaluador** | **Sí, ensayo aleatorizado con pantallas grabadas.** Y su financiación es la más limpia: donaciones, «**METR no ha aceptado financiación de empresas de IA**», aunque acepta **tokens gratis** |
| **DORA 2025** | Encuesta a ~5.000 profesionales | DeBellis, Storer, Harvey, Beane... | **Presentado por Google Cloud** | **Es encuesta auto-reportada, no medición.** Con investigación de **GitHub, GitLab y Workhelix** — todos vendedores de herramientas de IA |
| **StrongDM** | «1.000 $/día/ingeniero en tokens» | Su **cofundador y CTO** | Acceso privilegiado | **No publica NINGUNA métrica de resultado.** Y en HN un exempleado afirma que **la empresa fue vendida y el CTO se marchó** |

## Las fuentes de comunidad: votos reales, y la inversión que descubrí

**Comprobados con la API de Hacker News y la API JSON de Reddit.**

| Hilo | Puntos **reales** | Comentarios | Qué dice de verdad |
|---|---:|---:|---|
| [«Why Software Factories Fail»](https://news.ycombinator.com/item?id=49023019) | **394** | 272 | Escepticismo informado. El autor del análisis admite en el hilo que **no encontró datos de resultado de StrongDM** |
| [«Spec-Driven Development: The Waterfall Strikes Back»](https://news.ycombinator.com/item?id=45935763) | **225** | 191 | **El hilo más sustancial sobre SDD — y no lo había mirado** |
| [«Understanding SDD: Kiro, Spec-Kit, Tessl»](https://news.ycombinator.com/item?id=45610996) (Martin Fowler) | 128 | 32 | Análisis de referencia |
| «Spotify has merged 1500 agent created PRs» | **2** | 2 | **Vacío.** El caso más citado por los agregadores no generó discusión |
| [«¿Alguien usa SDD?»](https://www.reddit.com/r/ChatGPTCoding/comments/1otf3xc/does_anyone_use_specdriven_development/) | **78** | 94 | **El más votado sobre SDD, y está A FAVOR** |
| [«La masturbación técnica»](https://www.reddit.com/r/ChatGPTCoding/comments/1o6j1yr/specdriven_development_for_ai_is_a_form_of/) | **66** | 87 | **Su comentario principal (40 pts) REBATE al autor** |
| [«Spec Driven Development and other shitty stuff»](https://www.reddit.com/r/ExperiencedDevs/comments/1reiro1/spec_driven_development_and_other_shitty_stuff/) | **7** | 39 | Pequeño y dividido |
| [«Agentic, Spec-driven…»](https://www.reddit.com/r/ExperiencedDevs/comments/1ox40ww/agentic_specdriven_development_flow_on/) | **15** | 99 | Ambivalente |

**La consecuencia:** la afirmación «la comunidad rechaza el SDD» **es falsa**, y era la columna vertebral de [[lo-que-dice-la-comunidad]]. Elegí los hilos que decían lo que esperaba. Lo que hay es una **discusión dividida**.

## Lo que no se pudo comprobar, y dónde se intentó

- **La afiliación de «Curse of Instructions»**: PDF anónimo con metadatos vacíos; foro sin ella; `api2.openreview.net` con reto anti-robots; **no existe versión en arXiv** (dos búsquedas de frase completa, cero resultados), OpenAlex cero, DBLP con reto, DuckDuckGo con CAPTCHA.
- **«ICSE 2026» de *When Prompts Go Wrong***: sin comentario de venue en arXiv, sin la cadena en el PDF. **Tendría que confirmarlo el programa de ICSE 2026, y no se consultó.**
- **Las actas de ESEM 2026**: DROPS devuelve 404 y 403. El paper está aceptado y **programado**, las actas no están publicadas.
- **La tabla que produce el 74% de Ambig-SWE**: no existe; la cifra sólo aparece en prosa.
- **La cita textual de Boehm y Basili (2001)**: de pago. Y el famoso «**87,5% de los defectos vienen de requisitos**» **no tiene fuente primaria**: es folclore, y hay que dejar de citarlo.

## La lección, y es la misma cinco veces

**Cinco conclusiones, cada una sobre un solo estudio, y ninguna comprobada contra su versión final, su estado de aceptación o su texto completo.** No fue un fallo de búsqueda: fue **no abrir la página de la conferencia una vez**. Citar «ICLR 2025» un paper rechazado se evita con un clic.

**Y hubo un segundo modo de fallo, peor que el primero: elegir la evidencia que me convenía.** La nota de comunidad afirmaba que «la comunidad rechaza el SDD» apoyándose en dos hilos de **7 y 15 puntos**, mientras el hilo más votado del tema —**78 puntos**— decía lo contrario, y el comentario principal del hilo que usé como prueba **rebatía a su propio autor**. **Eso no es un error de verificación: es sesgo de selección, y no lo detecta ninguna comprobación de fuentes.** Sólo lo detecta ir a por la evidencia contraria, que es exactamente lo que no hice hasta que me lo pediste.

**Y la auditoría encontró algo que va más allá de mis errores: casi todas las fuentes tienen un interés, un conflicto o una versión vieja.** Casi la mitad de los papers están firmados por empresas que venden lo que evalúan. Las fuentes de empresa, **todas sin excepción**, las firma quien vende. El único bloque limpio —académicos sin conflicto, financiación declarada, denominador grande y salvaguardas explícitas— es el **mandato 2× de CMU y Stanford**, y aun así tiene sus matices.

**Regla a partir de aquí:** ninguna afirmación entra sin **dos fuentes independientes**, sin **comprobar el estado de publicación**, sin **decir quién firma y qué vende**, y sin **buscar activamente el caso contrario antes de escribirla**.

## Enlaces

- [[contra-evidencia]] — el ataque a las cinco conclusiones
- [[investigacion-lego]] — el informe, con su aviso de corrección
- [[crear-la-tarea]] — donde estaban los dos primeros pilares
