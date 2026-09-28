---
title: Lego — contra-evidencia: qué derriba las conclusiones
created: 2026-09-28
updated: 2026-09-28
tags: [lego, contraevidencia, refutacion, correccion]
zona: tecnico
---

El ataque frontal a cada conclusión, buscado a propósito. **El resultado obliga a retirar la forma fuerte de dos de ellas y a corregir tres errores del informe.** Está escrito en el mismo sitio donde estaban las conclusiones originales, no en un apéndice, porque el informe sin esto sería falso.

## Conclusión 1 — «el SDD no reduce defectos». HAY QUE RETIRARLA

**El estudio que la sostenía no mide SDD, y lo dice él mismo.**

La versión que va a la conferencia ESEM 2026 cambió incluso el título: *«Specification Artifacts in Open-Source Pull Request Workflows Do Not Reduce Defects»*. De su propio texto final:

> «Lo que medimos son **artefactos de especificación** — abrumadoramente **referencias a tickets e incidencias** — y no la salida de herramientas de SDD, que **ningún repositorio de la muestra registra en git**.»

El desglose, del plan de revisión del propio autor:

| Fuente del «artefacto de especificación» | N | % |
|---|---:|---:|
| Incidencia `#N` con `fixes`/`closes`/`resolves` | 11.348 | 46,7% |
| URL de incidencia | 7.252 | 29,8% |
| Identificador tipo `PROJ-123` | 5.112 | 21,0% |
| Documento externo | 561 | 2,3% |
| **Sección de requisitos en el cuerpo del PR** | **24** | **0,1%** |

Y la frase que lo resume todo, del propio autor: **«el 97,5% del tratamiento es “este PR referencia una incidencia o un ticket”».**

**Auditoría del árbol de ficheros:** de los **119** repositorios que dice el paper —el manifiesto del repositorio tiene 120, y ahí me equivoqué yo—, **cero** tienen un directorio de herramienta SDD (`.specify/`, `.speckit/`, `.kiro/`). Tres los excluyen explícitamente en `.gitignore` — grafana, con el comentario *«los ficheros de Spec-kit no deberían registrarse sin un proceso de diseño mayor»*. Es decir: **donde se usan, su salida no entra en git, así que ninguna medición basada en artefactos del repositorio puede verlos — ésta incluida.**

### Otros problemas del estudio, todos documentados

- **Los revisores lo vieron.** Registro de revisión: *«condicionalmente aceptado. El revisor 26A recomendó rechazo; la metarrevisión lo anuló»*, y la condición vinculante número uno es *«reformular en torno a lo que se mide»*.
- **Parada opcional.** Del propio plan: *«Tamaño de muestra no fijado de antemano (parada opcional). Se añadieron datos buscando un efecto y se dejó de recoger cuando no apareció.»*
- **El resultado estrella se retiró del resumen:** *«el marco de “+1,4 pp, los PRs con spec introducen más defectos” se retira del resumen».*
- **El p-valor cambia entre versiones del mismo dato.** El working paper de Zenodo da **p = 0,016** para calidad→defectos; el resumen de ESEM, con los mismos 88.052 PRs, da **p = 0,164**. El informe citaba el 0,164 y enlazaba Zenodo: **mezcla de versiones**.
- **Tres registros en Zenodo en dos días**, uno titulado «…de 89.599 pull requests» y dos duplicados con 88.052.
- **Ruido de las medidas:** el rastreo de defectos SZZ mal-atribuye entre el **46% y el 71%**; la calidad se puntuó con **Claude Haiku** validado contra valoración humana en **38 PRs** (ρ = 0,42); el clasificador de IA tiene **recall del 24,2%**.
- **Autor único, «investigador independiente», preprint auto-publicado, con un libro comercial con la misma tesis.**

### Lo que sí queda en pie, y la formulación correcta

Lo defendible es mucho más débil y más preciso: **«que un PR enlace un ticket no se asocia a menos defectos»**. Nada más.

Y la corrección de fondo es de lógica: **la conclusión confundía «ausencia de evidencia» con «evidencia de ausencia».** El estudio no prueba que el SDD no funcione; prueba que **nadie lo ha medido**. Lo que sí converge desde tres tipos de fuente independientes (academia, industria, comunidad) es que **el lado que dice que funciona tampoco tiene medición**.

## Conclusión 2 — «lo que importa es la verificación, no la especificación». HAY QUE RETIRAR LA FORMA FUERTE

**Y el contraataque es limpio: dos experimentos que sólo cambian el texto del enunciado, dejando la verificación idéntica.**

**[«When Prompts Go Wrong», ICSE 2026](https://arxiv.org/abs/2507.20439)**, revisado por pares. Extienden HumanEval y MBPP mutando el enunciado (ambiguo, contradictorio, incompleto), 300 tareas, 7 modelos:

| Modelo | Enunciado original | Enunciado contradictorio |
|---|---|---|
| GPT-4, Pass@1 | **73,8%** | **6,7%** |

Y el dato que rompe la tesis de fondo — **código ejecutable pero incorrecto**: GPT-4 pasa del **24% con enunciados originales al 54%, 65% y 89%** con los mutados. **El único cambio entre condiciones es el texto.** Si la conclusión fuera cierta, el delta tendría que ser cero.

**Y el argumento lógico, que es lo que más pesa:** **un test *es* una especificación ejecutable.** «Verificar» no es una alternativa a «especificar»: es especificar en un lenguaje más estrecho. La dicotomía del informe era **falsa de raíz**. Y el ejemplo del 24%→89% muestra exactamente cuándo se rompe la equivalencia: **sin un enunciado correcto, el test hereda la ambigüedad y certifica la cosa equivocada.**

**Formulación correcta:** *la especificación fija **qué** es correcto; la verificación es el mecanismo que explota los errores de ejecución. Ninguna sustituye a la otra, y el efecto medido más grande es el de la primera.*

## Conclusión 3 — «darle la opción de preguntar lo hunde». LA MITAD ESTABA MAL**

**Me quedé con la mitad del resumen que me convenía.** El resumen de HiL-Bench que cité dice, en la frase siguiente a la que usé:

> «El entrenamiento por refuerzo con una recompensa Ask-F1 moldeada muestra que **el juicio es entrenable**: un modelo de 32B mejora tanto la calidad de la petición de ayuda como la tasa de éxito, con ganancias que se transfieren entre dominios.»

**Esa mitad no aparece en el informe.** Era del mismo resumen.

Y tres matices más del propio paper: el `ask_human()` **sí devuelve la respuesta** cuando la pregunta apunta al bloqueador correcto; cada tarea lleva **3 a 5 bloqueadores** y la métrica es **pass@3**, así que el hundimiento es en buena parte **probabilidad compuesta**, no «preguntar resta»; y su conclusión literal es que las herramientas de petición de ayuda «**no mejoran el rendimiento de forma uniforme**».

**El hundimiento del 55,8–67,8% de UnderSpecBench aplica a las ejecuciones que actúan**, no a todas — el informe lo citaba sin ese recorte. Y los agentes **sí preguntan**: la disposición a hacerlo es del **38,3%**, **44,5%** y **31,8%** según el andamiaje.

**Y la cifra del «74%» que descarté como no verificada… era real.** Viene de **Ambig-SWE, ICLR 2026** ([arXiv:2502.13069](https://arxiv.org/abs/2502.13069)): 500 incidencias de SWE-bench Verified, 6 modelos, *«hasta un 74% sobre las configuraciones no interactivas»*. **Me equivoqué al descartarla.** Otras confirmaciones: **69,40% frente a 61,60%** en «Ask or Assume?»; **ClarifyGPT** (FSE 2024) pasa Pass@1 del **70,96% al 80,80%**.

**Con la contrapartida honesta:** el coste se multiplica por **2,1** (de 1,63 $ a 3,50 $ por tarea), y un baseline interactivo simple con **1,02 preguntas** bate a un andamiaje elaborado con **3,06**. **Preguntar funciona si se pregunta poco y bien.**

**Formulación correcta:** *el agente no pregunta por defecto — eso aguanta; habilitar la pregunta sin selección no basta; el beneficio depende de preguntar poco, bien y sólo cuando hace falta, y **es entrenable**.*

## Conclusión 4 — «el andamiaje importa más que el modelo». NO AGUANTA

**El dato del 29,8% lo cité más fuerte que su fuente.** El resumen del paper dice: *«los rangos observados dentro del mismo modelo alcanzan 29,8 puntos porcentuales… aunque **este diseño observacional no identifica efectos causales del andamiaje**»*. El informe decía «cambiar **sólo** el andamiaje mueve hasta 29,8 puntos» — **eso es más de lo que dice el paper**. Y su hallazgo central es otro: los treinta primeros del ranking **no son separables estadísticamente**.

**Y el contraejemplo tiene 260 configuraciones** ([«Towards a Science of Scaling Agent Systems»](https://arxiv.org/abs/2512.08296), Google Research + DeepMind + MIT), con herramientas, prompts y **cómputo igualados**: el cambio relativo frente al agente único va de **+80,8%** en razonamiento financiero descomponible a **−70,0%** en planificación secuencial. Su conclusión: **«el ajuste entre arquitectura y tarea determina el éxito»**.

**Y lo más incómodo: Anthropic, en su propio blog, niega la tesis.** *«Actualizar a Claude Sonnet 4 es una ganancia de rendimiento mayor que duplicar el presupuesto de tokens»*, y **«el uso de tokens por sí solo explica el 80% de la varianza»**. Es documentación de fabricante —no vale como prueba— pero contradice literalmente la afirmación.

**Formulación correcta:** *añadir agentes o pasos no es una palanca fiable; el óptimo es finito y depende del ajuste arquitectura-tarea; y subir de modelo suele rendir más por euro que añadir orquestación.* Nótese que esto **sigue apoyando el diseño de piezas mínimas** — pero ya no por el motivo que yo daba.

## Conclusión 5 — «casi todos los oráculos fallan dando por bueno lo roto». NO AGUANTA ASÍ

**Hay familias de oráculo con cero falsos positivos medidos:**

- **Clover** ([arXiv:2310.17807](https://arxiv.org/abs/2310.17807)): *«una tasa de aceptación prometedora (hasta el 87%) para instancias correctas, manteniendo **tolerancia cero con las incorrectas adversariales (ningún falso positivo)**»* — y **0 de 60** en cada una de las cuatro familias adversariales.
- **AutoVerus** (Microsoft Research + UIUC, **OOPSLA 2025**): genera pruebas correctas para **137 de 150** tareas.
- **TestGen-LLM** (Meta, **FSE 2024 Industry**): *«73% de sus recomendaciones aceptadas para producción»*.

**Y el error de dirección es el más grave:** en el benchmark más citado, el modo de fallo dominante **no es dar por bueno lo roto, es rechazar lo correcto**. La auditoría de OpenAI encontró que **el 35,5% de las tareas auditadas tienen tests demasiado estrictos que «invalidan muchas entregas funcionalmente correctas»** — bajo un epígrafe titulado literalmente **«los tests rechazan soluciones correctas»**.

**Y «techo» era una línea base.** DafnyBench dice de su propio 68%: *«esperamos que DafnyBench permita mejoras rápidas desde esta línea base»*.

**Formulación correcta:** *el oráculo tiene techo **en los repositorios abiertos con los tests heredados** — ahí el ruido es grande y está medido (31,08% de parches aprobados con tests débiles; ~24 puntos de brecha entre el evaluador automático y el mantenedor real). Pero hay familias de verificación con falsos positivos cero, y el sesgo dominante de los benchmarks es rechazar lo correcto.*

## Tres errores del propio informe, corregidos

1. **Los motivos de rechazo del 32,66% y 14,98% no son de donde decía.** No están en el estudio de los 567 PRs ([arXiv:2509.14745](https://arxiv.org/abs/2509.14745)) — su conclusión literal es que los rechazos están *«impulsados principalmente por el contexto del proyecto… **más que por defectos inherentes al código de IA**»*. La fuente correcta es **Wang y Yang, MSR 2026** ([DOI 10.1145/3793302.3793568](https://doi.org/10.1145/3793302.3793568)), con **1.779 PRs rechazados** analizados, que sí dice que los PRs de IA se rechazan más por implementación incompleta y pruebas inadecuadas. **Los porcentajes exactos no se pudieron verificar**, así que salen del informe.
2. **El p-valor mezclaba versiones** del mismo estudio (0,016 frente a 0,164). Corregido arriba.
3. **La tabla de HiL-Bench omitía su recorte** (dominio SWE, «ejecuciones que actúan») y **la mitad del resumen sobre el refuerzo**. Corregido arriba.

## Veredicto final

| Conclusión | Veredicto tras el ataque |
|---|---|
| 1. El SDD no reduce defectos | **RETIRADA.** El estudio mide enlaces a tickets, no spec. Queda «ausencia de evidencia», no «evidencia de ausencia» |
| 2. La spec importa menos que la verificación | **RETIRADA en su forma fuerte.** Un test *es* una especificación; la dicotomía era falsa |
| 3. El agente no pregunta / preguntar lo hunde | **Primera mitad en pie, segunda retirada.** Preguntar funciona si se pregunta bien, y es entrenable |
| 4. Menos piezas y el andamiaje manda | **No aguanta.** El ajuste arquitectura-tarea decide; el modelo importa más de lo que decía |
| 5. Casi todos los oráculos fallan | **No aguanta.** Hay oráculos con cero falsos positivos, y el sesgo dominante es rechazar lo correcto |

**Lo que sobrevive de todo el informe, y no es poco:** la tarea debe ser autosuficiente y acotada; la verificación externa compra puntos reales; el bucle con contexto limpio es sólido; y las piezas mínimas siguen siendo razonables — **aunque ya no por los motivos que yo daba**. Lo que se cae es la arquitectura teórica con la que lo justificaba.

**La lección de método, que es la parte que más vale:** las dos conclusiones que más me gustaban eran las que se apoyaban en **un solo estudio**, leído en su versión antigua, sin comprobar si la versión final decía lo mismo. Cuando el barrido adversarial fue a por ellas de frente, se cayó.

## Dónde se ha buscado

Texto completo de los papers citados, **el repositorio entero del estudio de Hill** (`REVISION-PLAN.md`, `main.tex`, `CORRECTIONS.md` — la pieza más decisiva), el registro de revisión de ESEM, arXiv, APIs de Zenodo, GitHub, Crossref, OpenAlex, HN Algolia, y `r.jina.ai` y Wayback para saltar los 403 de openai.com y Cloudflare.

**Bloqueado o sin aportar:** `openreview.net` (verificación anti-robots: **no se pudo verificar «Curse of Instructions»**, que el informe cita); Semantic Scholar (429); Reddit (JSON inválido en tres rutas — **la pata de comunidad de estas conclusiones es sólo Hacker News**); Boehm y Basili (de pago: **la cita textual no se consiguió**); y DORA 2025, cuya web sólo da el marco sin cifras.

**Un dato que hay que dejar de citar:** el famoso «**87,5% de los defectos vienen de requisitos**» **no tiene fuente primaria**. Es folclore, y circula por todas partes.

**Sin comprobar:** el «aceptado condicional» de ESEM es autodeclarado por el autor —las actas no están publicadas y la conferencia es la semana que viene—; las citas marcadas como leídas por subagente no se comprobaron byte a byte; y **no existe un experimento que varíe factorialmente calidad de especificación × fuerza de verificación**, así que la conclusión 2 se refuta por conjunción de dos literaturas, no por un experimento directo.

## Enlaces

- [[investigacion-lego]] — el informe, ahora con estas correcciones
- [[crear-la-tarea]] — donde estaba la conclusión 1, corregida
- [[verificacion-y-oraculo]] — donde estaba la conclusión 5, corregida
- [[robustez-desatendida]] — donde estaba la conclusión 3, corregida
- [[limites-del-andamiaje]] — la crítica que sí sobrevive
