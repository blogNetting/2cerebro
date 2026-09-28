---
title: Lego — gratis y en local: qué se puede y a qué precio
created: 2026-09-28
updated: 2026-09-28
tags: [lego, local, modelos, coste]
zona: tecnico
---

Respuesta al criterio que pusiste —**todo gratis**—, que se descompone en tres planos que no cuestan lo mismo.

## Los tres planos

| Plano | ¿Gratis? | Detalle |
|---|---|---|
| **El software** | **Sí, todo** | Agentes y motores con licencia MIT o Apache. Ninguna licencia que comprar |
| **Los modelos** | **Sí, se descargan** | Qwen, Gemma, GLM, gpt-oss y compañía son de descarga libre |
| **El hardware** | **No, y es el coste real** | Un caso con 24 GB de VRAM cuesta **unos 700 $** (una 3090 de segunda mano); quien reporta buenos resultados suele tener **64–128 GB de memoria unificada** |

**La conclusión honesta:** existe una pila enteramente gratuita en software y modelos, y **funciona**, pero con dos condiciones: un andamiaje que se ocupe de las llamadas a herramientas, y **≥24 GB de VRAM o ≥64 GB de memoria unificada**. Por debajo de eso, los reportes se vuelven mayoritariamente negativos.

## La pila mínima que sostiene la evidencia

| Pieza | Qué es | Nota |
|---|---|---|
| **`llama.cpp`** (como `llama-server`) | El motor. Expone una API compatible con la de OpenAI | Es la base de casi todo lo demás |
| **OpenCode** o **Pi** | El agente | OpenCode: 210.541★, MIT. Su plan de pago es opcional: «es completamente opcional y no lo necesitas para usar OpenCode» |
| **Qwen3.5/3.6-35B-A3B** o **GLM-4.7-Flash** cuantizado | El modelo | MoE: cabe en 24 GB porque activa una fracción de sus pesos por token |

**Y una advertencia concreta:** **Ollama, no para agentes**, salvo que se suba el contexto a mano. Su valor por defecto es **4k tokens bajo 24 GB de VRAM**, y descarta el exceso **en silencio** — su propia documentación avisa de que las tareas que requieren contexto largo, «como búsqueda web, agentes y herramientas de código, deben ponerse a 64.000 tokens como mínimo». Es la causa número uno de fallos que parecen del modelo y son de configuración.

El resto de la tabla —Cline, Codex CLI, Continue, Aider, Claude Code apuntado a local— está en piezas y coste, con sus trampas: Cline obliga a un modo compacto que **desactiva las herramientas MCP**; Claude Code contra un modelo local necesita un ajuste de cabecera o la inferencia va «90% más lenta».

## El hallazgo que confirma lo de la academia

Esto es lo importante. Un usuario midió el mismo modelo (**Qwen3.5-9B Q4**), en el mismo benchmark (**Aider Polyglot, 225 ejercicios completos**), cambiando **sólo el andamiaje**:

| Andamiaje | Resultado |
|---|---|
| Aider, tal cual | **19,11%** (43 de 225) |
| Andamiaje adaptado | **45,56%** (101 de 225) |

**Mismas pesas, mismo hardware, 2,4 veces el resultado.** Y otro caso, con un modelo de 13B, reporta pasar de «~20% a 100%» en una selección de tareas de SWE-bench **sólo con guardarraíles estructurales**, sin tocar el modelo.

Esto **converge, por un camino completamente distinto, con lo que mide la academia**: dentro del mismo modelo, cambiar el andamiaje mueve la tasa hasta **29,8 puntos porcentuales**, mientras todo el top-30 del ranking abarca 8,8 (ver [[autonomia-medida]]). Una medición viene de benchmarks revisados por pares; la otra, de un usuario en su casa con Reddit. **Dicen lo mismo.**

**Consecuencia para Lego:** la restricción de «menos piezas» no te está costando rendimiento — **te lo está dando**, porque te obliga a invertir donde la evidencia dice que está la palanca. Lo que hay que cuidar no es el número de piezas sino **cuáles**: el bucle, el aislamiento del verificador y el formato de la tarea.

**Y un detalle barato con efecto medido:** reducir el número de herramientas expuestas. Un caso reportado pasó «de 11 herramientas en el prompt de sistema a 5» y el tiempo de respuesta cayó «de ~5 minutos a ~1», mismo modelo y mismo hardware.

## La contra-evidencia: gente que lo intentó y volvió

Existe, con volumen, y hay que darla:

- El hilo **«I'm done with using local LLMs for coding»** ([1.069 votos, 865 comentarios](https://reddit.com/r/LocalLLaMA/comments/1sxqa2c/)): el autor probó «Qwen 27B y Gemma 4 31B, que se consideran los mejores modelos locales» y concluye que «la pérdida de productividad no compensa las ventajas». Su queja no es el código, es el **criterio**: «Claude parece leerme la mente en la mayoría de los casos; Qwen 27B me hace levantar la ceja mucho más a menudo».
- El comentario más votado del subreddit (3.549 votos) es una advertencia contra el entusiasmo: «cada vez que alguien dice que un modelo de 27B iguala a Opus, le pido que lo pruebe en un código que conozca de verdad. No un benchmark, no un proyecto de juguete: su código de producción».
- Un caso medido, no una opinión: **240 llamadas a herramientas**, de las que «232 pasaron sin necesidad de reemisión, y el fallo seguía ahí al final». El autor lo llama «un problema de diagnóstico, no de generación».

**El consenso, incluso entre los entusiastas:** sirve para tareas acotadas, ediciones locales y trabajo sin conexión. **No** para refactores largos de varios ficheros ni para razonamiento difícil.

## El límite técnico duro

**El flujo de parámetros de las llamadas a herramientas casi no existe en local.** Lo señala Armin Ronacher (creador de Flask): la mayoría de lo que corre en local no lo soporta, así que «sólo ves qué ediciones se están haciendo en un fichero **una vez que el modelo ha terminado de emitir la llamada entera**». La consecuencia práctica es que **no puedes abortar un comando malo antes de que termine** — y eso, en un bucle desatendido, es una diferencia de seguridad, no de comodidad.

## Recomendaciones

1. **Sí es viable hoy**, con las dos condiciones: andamiaje que gestione las herramientas, y el hardware. Por debajo de 24 GB de VRAM, los reportes son mayoritariamente negativos.
2. **Pila mínima defendible:** `llama.cpp` + OpenCode o Pi + Qwen3.5/3.6-35B-A3B o GLM-4.7-Flash cuantizado siempre que **no** se use Ollama con su contexto por defecto.
3. **No esperes paridad con un modelo frontera.** Ni los entusiastas la sostienen.
4. **Lo que más mejora el resultado es el andamiaje, no el modelo.** La diferencia de 19,11% a 45,56% con las mismas pesas es mayor que la que hay entre muchos modelos distintos.
5. **Híbrido es lo que hace la mayoría de quien reporta éxito:** local para el bucle rápido, de pago para lo difícil. Quien lo plantea como sustitución total es quien acaba escribiendo el hilo de «me rindo».

## Dónde se ha buscado, y qué queda abierto

**Reddit cubierto** —era obligatorio, y era el hueco que quedaba—: catorce hilos de r/LocalLLaMA leídos por la API JSON, no por el resumen del buscador. **Hacker News** por su API: el hilo de [forge](https://news.ycombinator.com/item?id=48192383) (687 puntos), [Ask HN: What's Your Useful Local LLM Stack?](https://news.ycombinator.com/item?id=44572043). **GitHub primario**: estrellas, fecha del último empujón y licencia de nueve repos vía `gh api`, y **el código fuente de Codex** para confirmar las banderas en vez de fiarse del README. **Documentación oficial** de OpenCode, Aider, Ollama, Cline y Continue. **Blogs independientes**: [Armin Ronacher](https://lucumr.pocoo.org/2026/5/8/local-models/), [Itay Inbar](https://itayinbarr.substack.com/p/honey-i-shrunk-the-coding-agent).

**Sin aportar nada:** [Lobsters](https://lobste.rs/t/ai) — nada sobre agentes locales de código. Y un resumen de buscador sobre Reddit que **atribuía citas a un hilo que no se pudo abrir**: descartado, no usado.

**Sin comprobar:** una discrepancia en las cifras de *forge* (el titular dice «53%→99%» sobre 18 escenarios, el README actual dice «de un dígito a 84%» sobre 26) que **no se pudo reconciliar** — se cita con cautela; la documentación de **Pi**, que cuatro hilos distintos elogian y no se verificó; la **licencia de LM Studio**, que su página de precios no declara; y el estado actual de la fuga de OpenCode, que los indicios sitúan como «mayormente corregida» pero **no se probó**.

**Y un hueco de fondo, que importa:** **no existe ningún dato con denominador sobre cuánta gente usa esto en serio.** Ni encuesta ni censo. Todo lo disponible son hilos sueltos y sus votos — **y los votos no son usuarios**. Cualquier afirmación de adopción a escala de esta categoría estaría inventada, y no se hace.

## Enlaces

- [[autonomia-medida]] — la medición académica del peso del andamiaje
- [[piezas-y-coste]] — el recuento de piezas de la opción de pago
- [[montaje-documentado]] — el montaje
- [[investigacion-lego]] — el informe completo
