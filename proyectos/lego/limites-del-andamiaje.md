---
title: Lego — el techo del andamiaje
created: 2026-09-28
updated: 2026-09-28
tags: [lego, critica, mantenibilidad, contraevidencia]
zona: tecnico
---

La crítica más fuerte que existe contra toda esta categoría, y viene de alguien que **lo intentó de verdad** y dio marcha atrás. Es el mejor contrapeso del informe, porque ataca justo la conclusión optimista: que mejorando el andamiaje se llega a la autonomía.

## Quién lo dice

**Dex, fundador de HumanLayer**, en [«Why Software Factories Fail (or: harness engineering is not enough)»](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md) — la clave del ponente en la AI Engineer World's Fair 2026. *(El autor escribe la herramienta HumanLayer: **interés comercial declarado**, aunque su conclusión va contra su propio producto: dice que vuelvas a revisar código a mano.)*

> **Nota de cita:** el recuento de reacciones en Hacker News (394 puntos, 272 comentarios) viene de la búsqueda del subagente, pero **no he localizado el identificador del hilo**, así que no lo enlazo. El enlace al ensayo original sí está verificado y leído.

## El experimento y su resultado

En **julio de 2025** su equipo se pasó a «luces apagadas»: agentes de fondo trabajando sin nadie. Terminó, en sus palabras, con problemas enredados que los agentes no sabían resolver, caídas de servicio, usuarios enfadados, y él leyendo «código basura». **Hacia noviembre, a la tercera vez, decidieron reescribir**, y su cofundador se pasó **dos semanas** poniendo a mano los patrones en el editor.

Su tesis, literal:

> «Ninguna cantidad de ingeniería de arnés o de “loopsmaxxing” puede resolver lo que es, fundamentalmente, un problema de entrenamiento del modelo.»
>
> «**La fábrica con las luces apagadas no funciona.**»

## Por qué falla, y esto es lo importante

**Cuatro razones, y la tercera es la que más pesa:**

1. **Los modelos degradan el código con el tiempo.** «No pueden mantener ni mejorar la calidad de una base de código a lo largo del tiempo» sin dirección humana.

2. **Las recompensas del aprendizaje por refuerzo no penalizan el mal diseño.** Las tareas tipo SWE-bench puntúan sólo con `FAIL_TO_PASS` y `PASS_TO_PASS`: **el método con el que se llega al parche da igual**. Textual: «**no hay ninguna penalización por erosionar la mantenibilidad de la base de código**».

3. **El desajuste del horizonte de realimentación.** Ésta es la idea que lo cambia todo:
   > «Los tests te dan realimentación **en segundos**, pero la función de coste de una mala arquitectura se mide **en semanas, meses, quizá años**.»

   Es decir: **el oráculo existe para lo que importa poco y no existe para lo que importa mucho.** Y sin poder trazar un incidente de dentro de tres meses hasta la decisión que lo causó, no hay forma de cerrar el bucle.

4. **No hay oráculo rápido para la calidad.** Y aquí llega, **por un camino completamente independiente, al mismo argumento que [[clausura-semantica]]**:
   > «**Si un modelo pudiera distinguir de forma fiable el código bueno del malo, quizá habría escrito la versión buena desde el principio.**»

   Literalmente el mismo razonamiento que el ensayo de clausura semántica, y que la medición de Huang et al. de 2024. **Tres caminos distintos, un solo argumento.** Eso es convergencia real.

**Y un dato temporal que encaja con todo lo anterior:** las bases de código construidas por agentes empiezan a dar problemas **a los tres o seis meses**.

## El otro lado: cuándo sí funciona

La misma fuente, y hay que darlo:

- **Frontalizar el trabajo de planificación tiene un retorno enorme y medido:** «aproximadamente **una hora por adelantado reduce una revisión de 6 horas a 20 minutos**».
- **Cerca del 40% de las tareas son de un solo intento**; las medianas, un único documento de plan.
- Y su diagnóstico del cuello de botella, que es más útil que cualquier métrica: «**No tienes demasiadas PRs. Tienes demasiadas PRs malas.**»
- Su consejo final: aceptar las restricciones y moverse **2–3 veces** más rápido de forma segura, en vez de perseguir 10–100×.

Eso **no contradice** a [[crear-la-tarea]]: lo refuerza. Una hora de planificación que ahorra cinco de revisión es exactamente el argumento a favor de escribir bien la tarea, y sólo de la tarea — no de la ceremonia alrededor.

## Cómo encaja con todo lo demás

| Lo que dice esta crítica | Qué confirma de las notas anteriores |
|---|---|
| El bucle y el arnés no bastan | La conclusión 5: la verificación tiene techo |
| El oráculo rápido no cubre lo que importa | [[verificacion-y-oraculo]]: el techo del oráculo |
| «Si el modelo supiera distinguir, ya lo habría hecho bien» | [[clausura-semantica]]: el mismo argumento, otra vez |
| Degradación a los 3–6 meses | El caso medido: la ganancia «se desvanece en el monolito heredado» ([[casos-medidos]]) |
| Una hora de plan ahorra cinco de revisión | El valor de la tarea está en **acotar**, no en documentar |

**Es la pieza que faltaba para que el informe no fuera ingenuo.** Las notas anteriores concluían «escribe bien la tarea y verifica fuerte». Ésta añade el límite: **hay una parte de la calidad —la mantenibilidad— que ninguna verificación automática cubre hoy**, y sobre esa parte el humano no se puede quitar de encima. Y avisa de que el problema se manifiesta **meses después**, cuando ya nadie lo conecta con la decisión.

## Advertencia sobre la fuente

Es un **ensayo de un fundador**, con interés comercial declarado, basado en una experiencia de equipo no publicada con números. Su dato de Faros es **correlacional** y él mismo lo etiqueta así. Lo que le da peso no es su metodología —no tiene— sino tres cosas: que **documenta un fracaso propio**, que su conclusión **va contra su propio producto**, y que su argumento central **converge de forma independiente** con la medición académica y con el ensayo de clausura semántica. La marca: **testimonio cualitativo con convergencia**, no dato.

## Enlaces

- [[clausura-semantica]] — el mismo argumento del oráculo, por otro camino
- [[contra-evidencia]] — el resto de ataques a las conclusiones
- [[casos-medidos]] — los números de empresa, que encajan con la degradación a los meses
- [[crear-la-tarea]] — por qué una hora de plan vale cinco de revisión
- [[investigacion-lego]] — el informe completo
