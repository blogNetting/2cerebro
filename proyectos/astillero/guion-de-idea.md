---
title: El raíl común — de idea a especificación sin variar
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, especificacion, determinismo, diseno]
zona: tecnico
---

Cómo se consigue que **la misma idea produzca siempre las mismas preguntas y la misma forma de resultado**, para poder pulirla en vez de empezar de cero cada vez. Es el raíl sobre el que corren las etapas 0 y 1. Contexto: [[la-fabrica]].

## El problema, y por qué no se arregla pidiendo «pregunta mejor»

- Los modelos **reconocen la ambigüedad pero por defecto responden igual**: no preguntan. Y la calidad de las preguntas **se hunde cuando hay varias ambigüedades juntas**.
- Así que sin raíl, cada ejecución pregunta lo que le parece — o no pregunta. **Y nada es comparable entre ejecuciones, con lo que no se puede mejorar.**

## La solución, que está medida

En un estudio sobre pipelines de agentes, la planificación en **texto libre** resultó ser *«la mayor fuente de varianza residual»*. Y **validarla contra un esquema fijo antes de invocar la siguiente pieza eliminó el efecto por completo: índice de determinismo 1,000.**

**El determinismo no viene de mejores preguntas. Viene de que la salida tenga forma fija y se valide antes de usarla.**

Precio medido, y hay que asumirlo: añadir estructura **reduce** la consistencia literal entre ejecuciones (una medida de similitud cae de 0,745 a 0,535). Se cambia consistencia por verificabilidad — y aquí compensa, porque lo que se gana es **poder pulir**.

## Fase 1 — Las cinco preguntas, siempre las mismas

En este orden, de lo general a lo específico, **una por turno**:

| # | Pregunta | Por qué está |
|---|---|---|
| 1 | **¿Qué problema resuelve y para quién?** | La misión. Sin esto no hay criterio para decir si algo sobra |
| 2 | **¿Qué NO es?** | El no-alcance. Es lo que impide que el agente rellene los huecos por su cuenta |
| 3 | **¿Cómo sabrás que funciona?** | Los criterios de aceptación — la pieza con efecto medido |
| 4 | **¿Qué restricciones no estándar hay?** | Lo **único** que está medido que un fichero de contexto ayuda a especificar |
| 5 | **¿Qué te da más miedo que salga mal?** | El fallo a prevenir, declarado antes de construir |

**Regla dura: una sola por turno.** Con varias juntas, la calidad de las respuestas se hunde.

## Fase 2 — La salida, con forma fija

Esto es lo que se escribe al terminar. **Sin campo vacío**: si no se sabe, se dice «no lo sé» — que es información, no un hueco.

```yaml
mision: <una frase, qué problema y para quién>
usuario: <quién lo usa>
no_es:                          # el no-alcance, explícito
  - <lo que este proyecto NO hace>
criterios_aceptacion:
  - id: CA-1
    dado: <precondición>
    cuando: <disparador>
    entonces: <postcondición>
    indefinido: <qué pasa si no se cumple la precondición>
restricciones_no_estandar:      # lo único con efecto medido
  - <p. ej. "usa esta herramienta concreta", "no toques esta carpeta">
riesgos:
  - <qué tiene que no pasar>
```

**El formato de los criterios no es decorativo.** Precondición / disparador / postcondición / indefinido es el formato que dio **+9,8 puntos de detección de bugs** en el único estudio que lo midió contra alternativas.

## Fase 3 — Validación antes de seguir

Se comprueba contra el esquema **antes** de descomponer en tareas:

- ¿Están los cinco campos?
- ¿Cada criterio tiene **las cuatro partes**? Un criterio sin «indefinido» está incompleto.
- ¿El no-alcance tiene al menos una entrada? Un no-alcance vacío significa que no se pensó.

**Si falta algo, no se avanza: se vuelve a preguntar.** Ahí está el raíl — no en la fase 1, en la 3.

## Cómo se pule

El guion es un fichero. **Se edita y todas las ejecuciones siguientes usan la versión editada.** Eso es lo que hace que mejore con el uso en vez de depender de la inspiración del modelo.

**Y la advertencia que acompaña a todo el diseño:** está medido que de **49 reglas añadidas a un proceso, 39 no mejoraron nada** y tres empeoraron el resultado. Si se añade una pregunta nueva, tiene que traer su motivo — y si al cabo de unas cuantas ejecuciones no aporta nada, se quita.

## Lo que hay que asumir

- **El formato de los criterios es el único medido.** Que se llame «dado/cuando/entonces» o «precondición/disparador» es indiferente: **nadie ha medido que un vocabulario sea mejor que otro**. Lo que importa es que **estén las cuatro partes**, no cómo se llamen.
- **Esto reduce la variedad a propósito.** Dos ejecuciones darán resultados más parecidos **no porque el modelo sea más consistente, sino porque la salida tiene una forma fija**. Si algún día hace falta variedad creativa, este no es el sitio.

## Enlaces

- [[la-fabrica]] · [[verificador-de-tareas]] · [[medicion-de-la-fabrica]]
- [[desarrollo-autonomo-con-agentes]] — la evidencia
