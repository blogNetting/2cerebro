---
title: El verificador — pieza 7 de la fábrica
created: 2026-09-27
updated: 2026-09-28
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

## Lo que reveló construirlo (2026-09-27)

Todo esto salió **ejecutándolo**, no leyéndolo. Los tres primeros son fallos que estuvieron vivos y ninguno se veía en el código.

**1. El disparo tiene que ser `pull_request`, nunca `workflow_run`.**
La plantilla del proyecto disparaba con `workflow_run` («cuando la CI termine»). En ese evento GitHub define **`GITHUB_REF` como la rama por defecto** y `GITHUB_SHA` como su último commit. Como el verificador hace el checkout sin fijar `ref:`, **miraba `main` en vez del trabajo**. Y el base se derivaba de la rama del propio agente, así que «restaurar los tests» reponía la versión que había dejado el agente. Efecto combinado: **el caso que esta pieza existe para cazar habría salido VERIFICADO en verde.** Con `pull_request`, el checkout coge el commit del trabajo y `pull_request.base.sha` da el base correcto sin deducir nada.

**2. Hay que instalar las dependencias antes de ejecutar la suite.** El verificador lanzaba `python -m pytest -q` sobre un runner limpio, donde pytest no está: `No module named pytest`, código 1. **La suite no llegaba a ejecutarse nunca** — y como el código era 1, el sistema lo contaba como *veredicto*, gastando intento. En cualquier proyecto real, **toda tarea habría quedado bloqueada al segundo intento, siempre**. Hace falta un paso de instalación propio (`install_command`).

**3. «No arrancó» no es «no pasa»: eso es el `75`.** Si el comando de test no llega a ejecutarse —falta un módulo, no existe el binario, código 127— **no hay medición**, y sin medición no hay veredicto. Va por el camino del `75`, que no consume intento. Se limita a **firmas inequívocas** a propósito: adivinar por el texto del fallo en general clasificaría un test real que fallara parecido como «no se pudo comprobar», y eso sería **aprobar trabajo roto en silencio** — el fallo opuesto y peor.

**4. La salida de los tests la escribe el agente, y no se interpola nunca dentro de un `run:`.** Él redacta los tests; su salida es texto que no se controla. Meterla en un script de shell con llaves dobles sería **inyección de comandos**. Se compone leyendo del fichero capturado en tiempo de ejecución, y llega al aviso como variable de entorno.

**5. Y la lección de método, que vale para todas las piezas.** Los tres primeros fallos **no aparecieron en el banco de pruebas hecho a mano** — que se había desfasado respecto al molde y daba un falso positivo. Aparecieron al probarlo **en un proyecto generado con `copier`**, que es por donde pasa un proyecto de verdad.

**6. Un veredicto correcto puede seguir saliendo en rojo — encontrado el 2026-09-28, un día después de dar la pieza por cerrada.** La última orden del paso `clasificar` era `[ -n "$motivo" ] && printf 'Motivo: %s\n' "$motivo"`. Con veredicto limpio, `$motivo` queda vacío a propósito — no hay nada que explicar —, la comprobación `[ -n ... ]` da **falso** (código 1), y al ser la **última orden del paso**, `bash -e` (el shell por defecto de un `run:` de Actions) toma ese código como el resultado del paso entero. Encontrado leyendo el registro real: `Veredicto: verificado (código 0)` seguido de `Process completed with exit code 1`. **Cada PR que verificaba bien salía con el check en rojo igual** — el mismo tipo de mentira que el punto 1 de arriba, pero al revés: aquí el veredicto era correcto y el check mentía. Arreglado cambiando el `&&` final por un `if`; probado fuera del runner (exit 1 antes, exit 0 después) y en vivo, apuntando temporalmente un proyecto de prueba a la rama del arreglo. `actionlint` no lo detecta — es lógica de shell, no sintaxis YAML. **PR #28, fusionada en `main` el 2026-09-28** (commit `141fed2`). Falta cortar versión para que un proyecto real la reciba.

> **Una pieza no está probada hasta que se prueba por el camino que usa de verdad un proyecto.**

Corolario incómodo y anotado: durante unas horas este documento y `estado.md` dieron el verificador por «probado en vivo» a partir de una corrida del banco que **tenía el mismo agujero**. El veredicto era correcto **por el motivo equivocado**. Por eso el paso de redactar la wiki va **después** de probar: lo que se escribe antes de construir es diseño, no descripción del funcionamiento.

## Lo que hay que asumir

- **El cuello de botella es el oráculo, no el criterio.** Está medido que **345 parches erróneos pasaban en verde** en SWE-bench (40,9 % de un subconjunto entero — denominador declarado por el paper). El verde mentía porque el test era insuficiente.
- **Ni el mutation testing ni las pruebas por propiedades lo arreglan**: el estudio más directo encuentra efecto **marginal**, y el cara a cara da **empate** (68,75 % frente a 68,75 %, 16 problemas).
- **Nadie lo tiene montado en producción.** No existe ningún caso publicado de una empresa que use tests ocultos o solo-lectura como puerta de merge. Todo lo real con cifras son benchmarks. **Esta pieza se construye sin nadie delante.**

## Enlaces

- [[la-fabrica]] · [[recibo-de-verificacion]] · [[estado-de-verificacion]] · [[vigilante-de-tareas]]
- [[desarrollo-autonomo-con-agentes]] — la evidencia
- [[gas-city-frente-a-la-fabrica]] — el constructo `check` de Gas City: «el paso está hecho cuando lo dice tu script, no cuando lo dice el agente», con presupuesto de intentos. Aporta el patrón, no el aislamiento
- [[verificacion-sin-oraculo-informe]] — el informe que respalda esta pieza: los ocho mecanismos para que el agente no toque los tests, cada uno con su agujero, y por qué **ocultar** no es la respuesta y **solo lectura** sí
