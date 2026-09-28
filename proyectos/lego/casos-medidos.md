---
title: Lego — casos medidos: quién entrega software con agentes, y con qué números
created: 2026-09-28
updated: 2026-09-28
tags: [lego, casos, empresa, medicion]
zona: tecnico
---

¿Existe alguien que haya montado «escribo una tarea → los agentes la ejecutan hasta entregar software» y lo haya **medido**? Sí, existen casos. Y el conjunto da una conclusión que calibra todo lo anterior: **usarlo lo usa mucha gente; entregarlo de forma autónoma, muy poca.**

Etiquetas: **[IND]** tercero independiente · **[AUTO]** el propio equipo, sin auditoría · **[VEND]** fabricante vendiendo.

> ### ⚠️ CORREGIDO EL 2026-09-28 — tres cifras estaban mal, y una de ellas no existía
>
> La auditoría fue a la página primaria de cada empresa. Resultado:
>
> - **La cifra de LinearB no es la que yo daba.** El **4,7%** no es un dato de la industria: es **el techo del decil superior de «organizaciones élite» dentro de una muestra auto-seleccionada de clientes de LinearB**. Su propio texto dice que incluso en el percentil 90, «la parte autónoma de la cadena abre **1 de cada 20 PRs**». Y el vendedor **se contradice a sí mismo**: una página dice «2,7 M de PRs, 253 organizaciones» y otra «8,1 M de PRs, 4.800 equipos» — **dos universos distintos**.
> - **Dos cifras de Spotify no están en la fuente.** Ni «**1.000 PRs cada 10 días**» ni «**del año a la semana para el 70% de la flota**» aparecen en el artículo primario. Lo que dice es **1.500+ PRs en total** y que **«alrededor de la mitad»** de los PRs de Spotify están automatizados. **Las dos cifras que yo citaba: sin verificar.**
> - **La cifra de Anthropic está mal atribuida.** La página que yo enlazaba **no contiene el «90% del código»**. Procede del director financiero en una declaración de mayo de 2026, y un medio lo reporta como rango **70–90%**.
>
> **Y dos añadidos:** StrongDM **no publica ninguna métrica de resultado** —la web sólo tiene el eslogan de los 1.000 $/día en tokens— y en el hilo de Hacker News un **exempleado afirma que la empresa fue vendida y el CTO se marchó a una consultora**. Y el informe citaba «los dos estudios de METR»: **sólo se pudo verificar uno**.
>


## El único caso independiente con número duro

**Un mandato corporativo de duplicar la producción, en una empresa B2B anonimizada** ([arXiv:2607.01904](https://arxiv.org/abs/2607.01904), julio 2026). Autores académicos —Hao He, Yegor Denisov-Blanch, Sanmi Koyejo, Bogdan Vasilescu— y una salvaguarda que le da peso:

> «El papel de la empresa se limitó a compartir datos; no participó en el análisis ni en la redacción, y **ningún empleado de la empresa es autor**.»

**Qué midió:** panel de **802 desarrolladores** y **196.212 PRs** entre enero de 2024 y abril de 2026, con diferencia en diferencias escalonada. Resultado: las PRs por desarrollador pasaron de **21,2 a 44,3 al mes**, es decir **2,09 veces** la base previa al mandato.

**Y trae el matiz incómodo, que es la mitad del valor del caso:**

- **La carga por revisor se duplicó** (×2,0).
- La revisión humana cayó al **68%**, y la automática pasó a superarla (**84%**).
- La ganancia estaba «**concentrada en código nuevo y no separable entre generaciones de modelo**», y **«se desvaneció en el monolito heredado»**.
- Y el propio paper avisa: «un objetivo emitido a bombo y platillo invita a inflarlo, y **nuestro diseño no puede separar del todo la aceleración genuina de eso**» — porque la empresa se puso el conteo de PRs como métrica oficial.

Traducido: **funciona donde el trabajo es nuevo y acotado, y no funciona en el código viejo y enredado.** Que es exactamente lo que llevamos viendo en las diez notas anteriores.

## Cuánta autonomía hay de verdad

El dato que más corrige la intuición, **una vez leído con precisión** ([LinearB](https://linearb.io/blog/does-your-software-factory-work), 2,7 M de PRs, 83.000 desarrolladores, 253 organizaciones). **Ojo: el 4,7% no es la industria —es el techo del decil superior de «organizaciones élite», en una muestra auto-seleccionada de clientes de LinearB**:

| Grupo | PRs abiertos por agentes autónomos |
|---|---|
| Decil superior de «élite» | **4,7%** — y su propio texto dice «**1 de cada 20 PRs**» |
| Mejor 30% | **1,1%** |
| Mejor 60% | **0,1%** |

Y los PRs de agente se fusionan menos: **79% frente a 92%** de los humanos en el decil superior, y **37% frente a 81%** en el mejor 60%. LinearB vende plataforma —**interés comercial declarado**— pero publica el denominador y admite que los datos son correlacionales.

**En resumen: la autonomía real medida, incluso en los equipos punteros, está por debajo del 5% de las PRs.** Uber reporta un 11% de PRs abiertos por agentes *(charla, no fuente primaria de Uber: sin verificar)*.

## La tasa de fusión, por agente, con denominador

**[9.428 PRs de agentes en 489 repositorios** Python de más de 100 estrellas](https://arxiv.org/abs/2607.21832) (Polytechnique Montréal, independiente), merge ajustado:

| Agente | Tasa de fusión | Crudo |
|---|---|---|
| Claude Code | **84,3%** | 166/219 |
| Codex | 73,5% | 5.451/6.057 |
| Cursor | 63,9% | 291/541 |
| Copilot | 59,6% | 874/1.259 |
| **Devin** | **43,0%** | **512/1.355** |

**Devin —el producto que vende exactamente el discurso «ticket a PR»— saca el peor resultado, y muy por debajo de su propio «67%» autoreportado.** Es el caso más claro de cifra de fabricante desmentida por medición independiente en todo el informe.

## Los casos de empresa, y por qué casi ninguno vale como prueba

| Caso | Cifra | Por qué no es prueba |
|---|---|---|
| **[Stripe Minions](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents)** | «**más de mil PRs fusionados por semana**… sin código escrito por humanos» | **[AUTO]** Sin denominador. La comunidad lo calculó: ~3.000–3.500 ingenieros → **menos de 1 PR por ingeniero y semana**, y lo llamó «métrica de vanidad» ([HN](https://news.ycombinator.com/item?id=47110495)) |
| **[Spotify Honk](https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1)** | **Lo que dice de verdad:** **1.500+ PRs** en total, y «alrededor de la mitad» de los PRs de Spotify automatizados. **Las cifras que yo citaba —«1.000 cada 10 días» y «70% de la flota»— NO están en la fuente** | **[AUTO]** Sin metodología |
| **[Salesforce](https://www.salesforce.com/news/stories/how-engineering-became-agentic/)** | «PRs fusionados por desarrollador **+79%**», «salida total **+151,3%**» | **[AUTO]** No publica denominador ni define qué es su «Effective Output» |
| **Uber** | 11% de PRs abiertos por agentes; presupuesto anual de IA agotado en **4 meses** | **[AUTO]**, y no se pudo leer fuente primaria de Uber |

**Y el caso que mejor encarna lo que buscas, sin números:** **[StrongDM, «la fábrica de software»](https://factory.strongdm.ai/)** — «el código no debe ser escrito por humanos», «el código no debe ser revisado por humanos», escenarios de aceptación guardados **fuera del código** (holdout), miles por hora, y **1.000 $/día/ingeniero en tokens**. Es el flujo más puro que existe. Y **no publica ninguna métrica de resultado**: ni tasa de éxito, ni defectos, ni coste por PR. [Simon Willison](https://simonwillison.net/2026/Feb/7/software-factory/) lo enmarca como «un atisbo de un futuro posible», no como una medición.

## La contra-evidencia contra el propio uso

**[Anthropic, ensayo aleatorizado sobre formación de habilidades](https://www.anthropic.com/research/AI-assistance-coding-skills)** ([arXiv:2601.20245](https://arxiv.org/abs/2601.20245)) — **52 ingenieros**, tarea con una librería concreta:

> «los participantes del grupo con IA puntuaron **un 17% menos**»; «el grupo con IA sacó **50% de media en el cuestionario, frente al 67% del grupo que programó a mano**» (d = 0,738; p = 0,01).

Y por patrón de uso, que es lo más útil: **delegar (n=4) e iterar (n=4) dieron menos del 40%**; **consultar conceptos (n=7), más del 65%**. Los autores anticipan que el efecto será «más pronunciado» con productos agénticos.

**Dos cautelas:** es Anthropic estudiando su propio producto **sin declarar conflicto de interés**, y son mayoritariamente juniors. Pero la reacción de la comunidad fue el hilo más grande de todos los que ha visto esta investigación (**482 puntos, 347 comentarios** en Hacker News).

## Los huecos que dejó la búsqueda

1. **Nadie publica coste por PR fusionado.** Uber admite que no lo puede calcular; StrongDM da 1.000 $/día/ingeniero (≈20.000 $/mes) sin resultado asociado. **Es la métrica que falta en toda la industria.**
2. **No existe ni un solo caso de reversión medida.** Se buscó explícitamente y **no hay ningún equipo que haya publicado cuántas tareas hizo, qué se perdió y por qué lo retiró**. Los únicos «fracasos» documentados son predicciones de analista (Gartner), fraudes (Builder.ai, cuyo «Neural Network» eran 700 ingenieros humanos) o incidentes sueltos (Replit borró una base de datos de producción).
3. Los hilos de Hacker News sobre **Salesforce y Spotify están vacíos** — 4 puntos y 2 puntos. Los casos más citados por los agregadores **no generaron discusión técnica**.

## Qué sostienen los datos, en tres frases

1. **El flujo funciona donde la tarea es acotada y verificable** — migraciones, arreglos, funcionalidades periféricas. El único caso independiente da **2,09×**, y a la vez avisa de que el efecto **se desvanece en el código heredado**.
2. **La autonomía real medida es pequeña**: por debajo del **5%** de las PRs incluso en el decil superior de adopción.
3. **Fusionar no es sinónimo de éxito**: los merges de agentes necesitan reparación posterior con **1,62 veces** las probabilidades de los humanos, y el producto que más vende ese discurso es el que peor mide.

**El mejor caso contra esta lectura**, sin matizar: Stripe corre >1.000 PRs semanales sin una línea humana, con entorno aislado y revisión real; Spotify movió migraciones de un año a una semana para el 70% de su flota; y el mandato 2× está medido por académicos sin conflicto. No es humo. Pero ninguna de esas cifras describe el **100%** del trabajo: describen la parte acotada y verificable. El resto sigue igual.

## Dónde se ha buscado

Papers leídos a texto completo con `pdftotext`: [arXiv:2607.01904](https://arxiv.org/abs/2607.01904) y [arXiv:2607.21832](https://arxiv.org/abs/2607.21832). Páginas primarias de [Stripe](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents), [Salesforce](https://www.salesforce.com/news/stories/how-engineering-became-agentic/), [Anthropic](https://www.anthropic.com/research/AI-assistance-coding-skills), [LinearB](https://linearb.io/resources/ai-engineering-productivity-gap), [Cognition](https://cognition.com/blog/devin-annual-performance-review-2025), [GitClear](https://www.gitclear.com/ai_assistant_code_quality_2025_research), [StrongDM](https://factory.strongdm.ai/), [METR](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/). Reacción de comunidad por la **API de Hacker News** y **Redlib**.

**Sin aportar nada:** los hilos vacíos de Salesforce y Spotify; las búsquedas de «empresa que revirtió su rollout agéntico con números» (**cero resultados**); y la crítica metodológica publicada a las cifras de Salesforce o LinearB, que **no existe** más allá de foros.

**Sin verificar:** el intervalo de confianza numérico del 19% de METR (no está en la fuente primaria); las cifras de Uber y de Cursor (sólo secundarias); los denominadores de Salesforce y Stripe; y si el 2,09× está inflado por el propio mandato —los autores dicen que no pueden separarlo.

**Una incoherencia detectada en una fuente que se cita mucho:** GitClear publica un titular de «4 veces más clonación de código» que **no cuadra con sus propias cifras** (8,3% → 12,3%, aproximadamente 1,5×). Queda señalado y **no se usa como dato**.

## Enlaces

- [[autonomia-medida]] — las tasas de resolución por benchmark
- [[lo-que-dice-la-comunidad]] — el veredicto de los desarrolladores
- [[contra-evidencia]] — el ataque a las conclusiones
- [[investigacion-lego]] — el informe completo
