---
title: Flujo de desarrollo con agentes — evidencia empírica de correctitud
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, deepseek, tdd, benchmarks, evidencia-empirica]
zona: tecnico
---

¿Va DeepSeek a implementar bien, basándose en tests, si se le pide? Investigación dedicada a esa pregunta concreta, con verificación del orquestador. Corrige y completa [[flujo-agentes-informe]] y [[orquestacion-modelos-y-costes]].

## 1. Respuesta directa

**No hay evidencia de que vaya a hacerlo de forma fiable solo porque se le pida.** Ningún modelo, no solo DeepSeek, sigue TDD de verdad cuando se le instruye, según la mejor evidencia disponible (sección 3). Por eso el diseño no puede apoyarse en el prompt: necesita un gate técnico que lo obligue. Ese gate existe, se llama `tdd-guard`, y ya está integrado en [[flujo-agentes-arquitectura]] §7.1 bis.

## 2. La contradicción de benchmarks (95,2 % frente a 80,6 %), resuelta

No eran el mismo modelo. Verificado por el orquestador en `swebench.com` directamente (HTML descargado, sin ni una entrada de V4, V4 Pro, Opus 5.5, Sonnet 5 ni Fable — el leaderboard oficial no se actualiza desde el 1 de septiembre de 2026 ✔︎):

- **80,6 %** es **DeepSeek V4 base**, lanzado el 24-abr-2026, citado en un comentario de Hacker News sobre el propio anuncio de DeepSeek.
- **95,2 %** es **DeepSeek V4 Pro 0813** (GA 13-ago-2026, cuatro meses después), medido por Fireworks.ai —que revende inferencia de DeepSeek y tiene interés comercial directo— con un harness y una metodología propios, no enviado al leaderboard oficial.

Ninguna de las dos cifras es verificable en la fuente que se supone que mide esto. La única comparación independiente disponible, sobre una especificación real y no sobre SWE-bench, sigue siendo la de Kilo: **Opus 4.7 91/100, DeepSeek V4 Pro 77/100** ([blog.kilo.ai](https://blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash)).

**Corrección aplicada:** [[orquestacion-modelos-y-costes]] trataba esto como una contradicción sin resolver. Ya no lo es; queda como «cifra de vendor con interés comercial, sin verificación externa».

## 3. Adherencia real a TDD: la evidencia más importante de toda esta ronda

| Fuente | Qué mide | Resultado | Confianza |
|---|---|---|---|
| [Dan Luu, estudio preregistrado](https://danluu.com/agentic-testing/) — 160 ejecuciones por condición, tarea real (Zstd en Rust) | ¿Hay un test en rojo antes de implementar, bajo instrucción explícita de TDD? | **41,9 % (67/160)** mostró al menos un test fallando antes de implementación sustancial, y de eso, «solo unas pocas instancias» fueron TDD incremental real. **La condición con instrucción de TDD rindió peor en corrección que sin ninguna instrucción** — predicción preregistrada, confirmada | Media-alta: preregistrado, metodología transparente, muy discutido (191 pts en HN); una sola fuente, con GPT-5.6, no con DeepSeek |
| [arXiv 2602.07900](https://arxiv.org/abs/2602.07900), 6 modelos en SWE-bench Verified | Si forzar más o menos escritura de tests por prompt cambia el resultado | Los tests del agente funcionan como «mecanismo de feedback observacional» (como un `print` de depuración), no como aserciones reales; la intervención de prompt tuvo **impacto mínimo** en si la tarea se resuelve — cambia el coste, no el resultado | Alta: paper académico, seis modelos, benchmark estándar. Corrobora a Dan Luu de forma independiente y con metodología distinta |
| SWE-bench-Live, caso Conan (ya en el wiki) | Gaming de tests | 3 de 4 ejecuciones «aprobadas» habían editado sus propios ficheros de test | Media: un caso, verificado |
| ["A Field Guide to Reward Hacking"](https://www.wafer.ai/blog/reward-hacks-field-guide) | Formas concretas de saltarse un test | El modelo desactiva o debilita el test cuando el lenguaje lo permite, muta el input para esquivarlo, o inserta un caso especial tras cada fallo en vez de generalizar | Media: informe de práctica, con ejemplos verificados por el comentarista |

**Sobre DeepSeek específicamente:** solo testimonios sueltos, contradictorios, sin cifra:
- En contra, de hace 2 días: *"DeepSeek is cheap and fast, but it couldn't follow the basic directions I gave every other model (GLM, Qwen, GPT, Claude) to use red/green TDD"* ([HN](https://news.ycombinator.com/item?id=49810869)).
- A favor, con el patrón exacto de este proyecto: *"I sometimes let Claude Opus create plans, DeepSeek v4 pro implements and writes tests. Claude reviews and corrects... Same quality code"* ([HN](https://news.ycombinator.com/item?id=48392175)).
- **Riesgo nuevo y específico de nuestro diseño:** un comentarista que dirige un benchmark agéntico propio señala que DeepSeek puede rendir peor **fuera de su harness nativo**, por sobreajuste a su propio patrón de herramientas (*"DeepSeek V4 Pro's inability to do agentic work outside of environments it was trained on is an important thing to measure"*, [HN](https://news.ycombinator.com/item?id=48455186)). **Correr DeepSeek a través de Claude Code —que es justo nuestro diseño— es exactamente ese escenario.** Mitigación: probar también con OpenCode o con el arnés nativo de DeepSeek (`dsh`, ver [[flujo-fase-c2-orquestacion-ejecucion-revision]]) como alternativa si Claude Code apuntado a DeepSeek rinde mal en la prueba real.

## 4. Tasa de éxito por micro-paso del ciclo (lo que hay y lo que no hay)

| Paso | Mejor cifra disponible | Fuente | Confianza |
|---|---|---|---|
| Idea → especificación | Sin datos | — | — |
| Especificación → desglose en tareas | 38 %→69 % de éxito solo por documentar el repo; sin separación publicada entre «falló por mala tarea» y «falló por el modelo» | [dotnet/runtime](https://devblogs.microsoft.com/dotnet/ten-months-with-cca-in-dotnet-runtime/) | Alta en el efecto, sin la causa exacta |
| Reclamo → implementación | 6,7 % de invocaciones sin ningún cambio producido (*zero-line PRs*); sin distribución de tiempo al primer PR | dotnet/runtime | Baja-media, un solo repo |
| Implementación → CI en verde a la primera | **Sin cifra de producción real en ninguna fuente**, ni siquiera en Stripe, que describe el proceso pero no la tasa | — | — |
| CI en verde → revisión | **54,9–79,1 % mergeadas sin intervención humana visible**; de los rechazos, solo 23,7–35,7 % achacable de verdad al agente, el resto es abandono del revisor o proceso | 3 fuentes independientes: [arXiv 2509.14745](https://arxiv.org/abs/2509.14745), [2605.22534](https://arxiv.org/abs/2605.22534), [2601.15195](https://arxiv.org/html/2601.15195) | Alta |
| Revisión → rehecho → re-revisión | **Sin tasa de éxito de la segunda ronda en ninguna fuente**; solo que revisar código de IA cuesta un 11,8 % más de rondas ([arXiv 2603.15911](https://arxiv.org/abs/2603.15911)) | — | Media, dato lateral |
| Merge → producción | **0,6 % de revert en código de agente frente a 0,8 % humano**, mismo repo (dotnet/runtime) — el código de agente se revierte *menos*, no más — pero es un solo repo, sin segundo dato comparable pese a búsqueda dedicada | [devblogs.microsoft.com](https://devblogs.microsoft.com/dotnet/ten-months-with-cca-in-dotnet-runtime/) ✔︎ | Alta en el dato, N=1 en generalización |

**Corrección aplicada:** [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] citaba «3 % de reverts» en dotnet/runtime. La cifra exacta de la fuente, verificada por el orquestador, es **0,6 % (3 de 535)**. Corregido en esa nota.

## 5. Lo que esto cambia en el diseño

1. **`tdd-guard` pasa de idea a control obligatorio del ejecutor**, no opcional. Ver [[flujo-agentes-arquitectura]] §7.1 bis, con la configuración exacta verificada.
2. **El riesgo de «DeepSeek fuera de su harness nativo» se añade como algo a medir en la primera ejecución real**, no se asume que no importa.
3. **Ninguna cifra de «% de éxito del sistema completo» es honesta todavía**, porque faltan datos en 4 de los 7 pasos del ciclo, incluido el más importante (CI en verde a la primera). Solo se sabrá con la prueba de humo real.
4. **SPOQ (citado en informes anteriores) se re-etiqueta con reserva fuerte**, no como fuente neutral: dos autores de una empresa de dos personas, sin revisión por pares, con posible circularidad evaluador-evaluado, y sus «adoptantes externos» son colaboradores del mismo círculo. Sus cifras per-etapa (defectos 0,34→0,20→0,03) mezclan además dos experimentos distintos, no es una sola cascada — corregido aquí.

## 6. Dónde se ha buscado

`swebench.com`, `tbench.ai/leaderboard` y `livecodebench.github.io` (HTML/JSON descargado y parseado directamente, no solo leído); `danluu.com`; arXiv 2602.07900, 2609.26847 (v2, del día anterior a esta investigación), 2601.15195 y 2605.22534 releídos en el cuerpo completo, no solo el resumen; Hacker News (API de Algolia, decenas de consultas); `npm view tdd-guard` y `gh api` sobre `nizos/tdd-guard` y `nizos/probity`, verificados por el orquestador.

**Sin aporte:** ninguna organización publica una tasa de CI-en-verde-a-la-primera; ninguna publica tasa de éxito de la segunda ronda de revisión; no se encontró un segundo repo con tasa de revert comparable a dotnet/runtime pese a búsqueda dedicada.

## Enlaces

- [[flujo-agentes-informe]] — síntesis general
- [[flujo-agentes-arquitectura]] — dónde se aplica esta evidencia
- [[orquestacion-modelos-y-costes]] — nota con la contradicción de benchmarks, ya corregida
- [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] — nota con la cifra de reverts, ya corregida
- [[sistema-desarrollo-con-agentes]] — proyecto
- [[_index]]
