---
title: El verificador — pieza 7 de la fábrica
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, verificacion, diseno, seguridad]
zona: tecnico
---

Cómo se decide si una tarea está bien hecha, sin que el agente que la hizo pueda influir. Es la pieza que decide todas las demás y **el primer agujero de la fábrica**. Evidencia: [[desarrollo-autonomo-con-agentes]]. Contexto: [[la-fabrica]].

## El principio, y la corrección que costó aprender

**El verificador no puede ser el mismo sistema que el agente puede modificar.** Cuando el agente controla su propio evaluador, el verde deja de ser una señal.

**Y no vale ocultarlo sin más** — esto lo di por hecho al principio y la evidencia lo desmiente: con tests invisibles, **más del 80 % de las ejecuciones especulan sobre un evaluador imaginado**, y en **el 10-25 % de los casos ese razonamiento desvía el trabajo de lo pedido y aun así se cobra como correcto**. El fallo se vuelve **invisible**, que es peor que un test manipulado. Lo medido es que la posición correcta es **solo lectura**: *«restaura el rendimiento legítimo a la vez que impide los intentos de modificar los tests»*.

## Cómo funciona — cuatro pasos

**1. El agente no corre en el mismo sitio que el verificador.**
Del entorno del agente se captura **su cambio** (el diff), y el verificador lo aplica sobre una **imagen limpia**. El verificador nunca se ejecuta dentro del recinto del agente.

**2. Los tests se restauran desde la base antes de puntuar.**
Se sobrescribe lo que el agente haya hecho con la suite con la versión del commit base, y se ejecutan exactamente esos. Se restauran también los ficheros **borrados** — un agente puede hacer pasar la suite borrando tests.

**3. Solo lectura sobre los tests mientras el agente trabaja.**
No ocultamiento. Con la regla de rutas (abajo) ni siquiera puede empujar cambios que los toquen.

**4. Además de medir «pasa», medir «es lo que se pidió».**
Se reconstruye el problema **a partir del cambio y sin ver el enunciado**, y se comprueba que reconcilia con él. Es el único mecanismo medido que verifica contra la **intención** y no contra el test. Ganancia medida: **+7,0 %** de acierto (GPT-5-mini) y **+3,6 %** (MiniMax-2.5) — ambos sobre el mismo arnés, y las dos etapas aportan por separado.

## Las cuatro piezas de GitHub que se usan por debajo

Todas verificadas en su documentación:

| Pieza | Qué hace | Qué deja abierto |
|---|---|---|
| **Regla de rutas restringidas** (push ruleset) | **Impide empujar** commits que toquen `tests/`. Sintaxis tipo glob, hasta 200 rutas | Solo en GitHub Team+ y repos privados. **Rechaza el push entero**: si el mismo commit toca `tests/` y `src/`, se cae todo |
| **Check requerido con app esperada** | Exige que el verde venga de **una app concreta** — protege contra falsificar el resultado | Protege la **fuente**, no la **integridad de los tests** |
| **Workflow obligatorio desde otro repositorio** | El workflow que valida vive en un repo que el agente no controla | Hay que mantenerlo aparte; si alguien lo debilita, se debilita en todos |
| **Protección de ejecución** (reglas de actor) | **Separa quién aporta código de quién ejecuta el CI** | **No impide leer** los tests, solo quién dispara el CI |

**Y lo que no existe:** un permiso por ruta. **No se puede decir «esta app escribe en los tests pero no en el código».** Los permisos son por repositorio, no por ruta — por eso hay que sacar los tests del repo **o** usar la regla de rutas.

## Lo que hay que asumir

- **El cuello de botella es el oráculo, no el criterio.** Está medido que **345 parches erróneos pasaban en verde** en SWE-bench (40,9 % de un subconjunto entero — denominador declarado por el paper). El verde mentía porque el test era insuficiente.
- **Ni el mutation testing ni las pruebas por propiedades lo arreglan**: el estudio más directo encuentra efecto **marginal**, y el cara a cara da **empate** (68,75 % frente a 68,75 %, 16 problemas).
- **Nadie lo tiene montado en producción.** No existe ningún caso publicado de una empresa que use tests ocultos o solo-lectura como puerta de merge. Todo lo real con cifras son benchmarks. **Esta pieza se construye sin nadie delante.**

## Enlaces

- [[la-fabrica]] · [[recibo-de-verificacion]] · [[estado-de-verificacion]] · [[vigilante-de-tareas]]
- [[desarrollo-autonomo-con-agentes]] — la evidencia
- [[gas-city-frente-a-la-fabrica]] — el constructo `check` de Gas City: «el paso está hecho cuando lo dice tu script, no cuando lo dice el agente», con presupuesto de intentos. Aporta el patrón, no el aislamiento
