---
title: La fábrica — despiece por etapas
created: 2026-09-27
updated: 2026-09-28
tags: [astillero, agentes, fabrica, diseno, autonomia]
zona: tecnico
---

Despiece de Astillero en sus doce etapas, con dos ejes: **quién tiene que actuar** en cada una y **si puede operar sola hoy**. Sirve para separar lo que ya se puede automatizar de lo que no, y para saber dónde está el trabajo que queda.

La evidencia que sostiene cada decisión, con papers y cifras: [[desarrollo-autonomo-con-agentes]]. El método de investigación: [[metodo-de-investigacion]].

## Aviso sobre la base de evidencia — leer antes que nada

**El concepto tiene ocho meses de vida y nadie ha publicado un solo resultado medido.** Conviene saberlo antes de fiarse de ninguna cifra:

- **Nadie publica métricas de resultado.** StrongDM (el caso de referencia mundial) no publica tasas de defectos, ni comparativas de coste, ni resultados de producción. La «Agent Factory» de GitHub publica **conteos de workflows, no resultados**. Uber y OpenAI publican números de **adopción** (uso, carga sobre sistemas), **no de calidad del software resultante**. No hay un solo benchmark de una fábrica completa.
- **La única evidencia empírica encontrada es negativa.** Dex Horthy (HumanLayer) lo intentó y le explotó: *«en julio de 2025 pasamos a luces apagadas»* → la web cayó, los usuarios protestaron, y en noviembre **reescribieron desde cero**. Su diagnóstico: *«ninguna cantidad de ingeniería de arnés o de bucles puede resolver lo que es fundamentalmente un problema de entrenamiento del modelo»*, y **«no hay penalización por erosionar la mantenibilidad»** — los tests dan señal en segundos, pero el coste de la mala arquitectura se mide en meses.
- **Casi todo el que promueve el concepto vende algo.** StrongDM **se vendió en 2026** y su fundador montó **Diffusion**, una consultora para construir fábricas. Hay tres cursos de pago arrancando entre septiembre y noviembre de 2026. Un comentario de HN lo resume: *«la web tiene cero benchmarks, cero tasas de defectos, cero comparativas de coste, cero resultados de producción. La única métrica que ofrecen es "gasta más dinero"»*.
- **El término es más viejo de lo que parece y el actual es nuevo.** «Software factory» viene de una conferencia de la **OTAN de 1968** — la misma que dio «ingeniería del software». Lo nuevo es aplicarlo a agentes: Dan Shapiro lo definió en **enero de 2026** («un caja negra que convierte especificaciones en software», con «oscura» tomado de la fábrica de Fanuc, que está a oscuras porque no hay humanos).

**Consecuencia para este diseño:** el despiece de abajo se apoya en lo que **sí** está medido (los modos de fallo, la verificación, el contexto, la descomposición) — no en la promesa de la fábrica. Las etapas están ordenadas por lo que puede sostener cada una por separado, y por eso el despiece sirve aunque el concepto entero no tenga resultados publicados.

## Por qué despiezarla así

El objetivo del sistema es que **el trabajo ocurra sin que nadie lo conduzca**. «La fábrica» es el nombre que le dan las fuentes — Dan Shapiro la llama así en el nivel 5 de su escalera de autonomía; StrongDM llama así a su sistema; GitHub tiene su propia «Agent Factory». No es una metáfora mía.

Despiezarla con esos dos ejes sale de una observación incómoda: **una etapa puede estar técnicamente resuelta y aun así necesitar tu vista.** Automatizar lo que está maduro pero requiere criterio humano produce basura más rápido. Por eso el eje de madurez, solo, engaña.

## Las doce etapas

| # | Etapa | Qué hace | Quién actúa | ¿Sola hoy? |
|---|---|---|---|---|
| 0 | **Idea** | Convierte lo que quieres en algo escribible | **Tú** | ❌ necesita cabeza |
| 1 | **Especificación** | Contrato: precondición / postcondición / criterios de aceptación | Máquina | ✅ con criterios |
| 2 | **Descomposición** | Parte en tareas por **dependencia de estado** | Máquina | ⚠️ falta el criterio |
| 3 | **Estado** | Dónde vive qué está hecho y qué está verificado | Máquina | ✅ |
| 4 | **Despacho** | Coge la siguiente tarea y arranca el agente | Máquina | ✅ |
| 5 | **Ejecución** | El agente escribe test y código, aislado | Máquina | ✅ |
| 6 | **Autocomprobación** | El agente corre sus pruebas y corrige | Máquina | ✅ |
| 7 | **Verificación** | **Algo distinto del agente** decide si está bien | Máquina | ✅ **construido y probado** (2026-09-27) |
| 8 | **Puerta** | Pasa, reintenta o bloquea | Máquina → **Tú** al bloquear | ✅ **construida y probada** (2026-09-27) |
| 9 | **Aprobación y merge** | Rutas protegidas → tú; el resto, solo | Máquina → **Tú** en excepciones | ⚠️ **no bloquea de verdad**: sin ruleset (falta GitHub Pro en repo privado) ningún check es obligatorio, así que se puede fusionar en rojo. Detalle en [[astillero-bitacora]] |
| 10 | **Despliegue** | Publicar y poder volver atrás | Máquina | ❌ sin destino decidido |
| 11 | **Operación** | Que no se caiga en silencio | Máquina | ⚠️ diseñado, sin estrenar |
| 12 | **Medición** | Saber si la fábrica funciona o solo va rápido | Máquina → **Tú** leyendo | ✅ **construida y probada** (2026-09-27) |

**Apareces en cuatro de doce** (0, 8, 9, 12). De esos cuatro, **tres son avisos, no trabajo**: solo el 0 es tu tiempo de verdad.

## Las cinco etapas que ya están maduras

**1. Especificación.** Lo que la hace funcionar no es escribir mucho: es que exista un **contrato verificable**. El formato con efecto medido: precondición, postcondición y comportamiento indefinido, escrito **antes** de los tests — dio +9,8 puntos de detección de bugs (denominador no publicado en el resumen del paper: **dato sin denominador, pendiente de verificar en el cuerpo**). Y el ciclo completo (constitución → especificar → plan → tareas → implementar → verificar) está en la práctica común.

**Herramienta para esta etapa — sin recomendación sostenible.** Se compararon cuatro (Spec Kit, OpenSpec, BMAD, Kiro) contra un control sin spec, sobre **50 tickets** (10 por categoría) con revisión ciega. Los resultados: fusión del **80-84 % con spec frente al 72 % sin spec**. **Pero el propio estudio declara una banda de ruido de ±10 puntos y dice que las opciones dentro de esa banda no se ordenan** — y la diferencia entre el mejor (OpenSpec, 84 %) y Spec Kit (80 %) es de **4 puntos, o sea 2 tickets**. La diferencia de deriva que se citaba (2,4 % frente a 12,5 %) son **~1 suceso frente a ~5** sobre 40 casos. **Ninguna de las dos permite elegir.** Y el estudio es de un vendor y de una sola pasada.

**Lo único que sí se sostiene de esa comparación:** que **usar alguna especificación escrita supera a no usar ninguna** (80-84 % frente a 72 %, 12 puntos, justo en el borde de la banda), y que **en arreglos pequeños el control gana** (90 %). La elección entre las cuatro **no está decidida por datos**.

**3. Estado.** GitHub Issues más un campo propio. La única regla dura: **«cerrado» y «verificado» son cosas distintas y las escribe quien corresponda**, no quien hizo el trabajo. Despiece de las piezas 3 y 8: [[estado-de-verificacion]].

**4. Despacho y 5. Ejecución.** `gh-aw` los cubre: compila un flujo escrito en Markdown a un workflow de Actions, arranca un recinto aislado por tarea, y **la clave del modelo la tiene un proxy, no el agente** — el agente la usa pero no puede leerla. El límite se pone con permisos, no con instrucciones.

**6. Autocomprobación.** El agente corre sus pruebas. Es útil, pero **no es un control**: está medido que un agente satura sus propios tests, y que autocorregirse sin un oráculo externo **empeora** el resultado.

## Las dos etapas que eran el agujero — **cerradas el 2026-09-27**

> **Actualización (2026-09-27, tarde).** Esta sección se escribió cuando las etapas 7 y 12 eran los dos huecos del sistema. **Ya no lo son**: el verificador, la puerta y la medición están **construidos, probados en vivo y publicados en el tag `v0.4.0`**. Lo que sigue es el diseño que se decidió entonces, y sigue valiendo como despiece — pero **no como lista de lo que falta**. Estado real: [[astillero-bitacora]].
>
> Lo que **sí** queda de estos dos huecos es la parte que nunca se cerró: **medir la intención** (reconstruir el problema desde el cambio, sin ver el enunciado, y comprobar que reconcilia). El verificador mide «pasa», no «es lo que se pidió».
>
> **Y un bug encontrado el 2026-09-28, después de darla por cerrada:** el paso final del verificador salía en rojo aunque el veredicto fuera «verificado» — la última línea del script devolvía código 1 cuando no había nada que explicar. Arreglado, probado en vivo dos veces (fuera del runner y sobre una PR de prueba real) y **fusionado en `main`** (PR #28, commit `141fed2`). No cambia el diseño; era un fallo de una línea de shell. Falta cortar versión para que llegue a un proyecto real, que hoy sigue fijado a un tag anterior. Detalle en [[verificador-de-tareas]] y [[astillero-bitacora]].

**7. Verificación.** Es la que decide todo lo demás. Despiece de la pieza: [[verificador-de-tareas]] (el mecanismo) y [[recibo-de-verificacion]] (lo que queda escrito, 7b). **Corrección importante sobre lo que dije al principio:** propuse *ocultar* los tests al agente, y la evidencia dice que **ocultar mueve el agujero, no lo cierra**. Está medido: con tests invisibles, **más del 80 % de las ejecuciones especulan sobre un evaluador imaginado**, y en **el 10-25 % de los casos ese razonamiento desvía el trabajo de lo pedido y aun así cobra como correcto** — un fallo que además se vuelve **invisible** para quien solo mira el verde. La posición que sí está respaldada empíricamente es **solo lectura**: *«restaura el rendimiento legítimo a la vez que impide los intentos de modificar los tests»*.

La receta que sí tiene cifras de producción detrás, y que es la de los dos sistemas que funcionan (SWE-bench Pro V2 y el benchmark de Octomind):

1. **El verificador no corre en el entorno del agente.** Se captura el cambio del agente y se aplica sobre una **imagen limpia**, y el verificador se ejecuta ahí.
2. **Los tests se restauran desde la base antes de puntuar** — se sobrescribe lo que el agente haya hecho con la suite, y se ejecutan exactamente esos.
3. **Solo lectura sobre los tests durante el trabajo**, no ocultamiento.
4. **Y además de medir «pasa», medir «es lo que se pidió»** — el único mecanismo que verifica contra la intención y no contra el test (reconstruir el problema desde el parche, sin ver el enunciado, y comprobar que reconcilia).

**El cuello de botella es el oráculo, no el criterio.** Está medido que **345 parches erróneos pasaban en verde** en SWE-bench, afectando al 40,9 % de SWE-bench Lite. Y OpenAI **retiró su propia recomendación** de adoptar un benchmark de tests ocultos porque exigían cosas no derivables del enunciado. Ni el mutation testing ni las pruebas por propiedades lo arreglan: el estudio más directo encuentra efecto **marginal**, y el cara a cara da **empate** (68,75 % frente a 68,75 %).

**Lo que GitHub da de serie, verificado:** una regla que **impide empujar** a rutas concretas como `tests/`; exigir que el check verde venga de **una app concreta** (protege contra falsificar el resultado); **workflows obligatorios desde otro repositorio** que el agente no controla; y **separar quién aporta código de quién ejecuta el CI**. Lo que **no existe**: un permiso por ruta — no se puede decir «esta app escribe en los tests pero no en el código».

**12. Medición.** Sin ella no puedes distinguir «va bien» de «va rápido». Despiece de la pieza: [[medicion-de-la-fabrica]]. Los cuatro números: **reversión, defectos escapados, tiempo de revisión, coste por tarea**. Las referencias medidas existen (reversión de Codex 6,1 % frente a humano 11,5 %; tiempo mediano de revisión +441,5 % con adopción alta de IA), pero **nadie publica un sistema que mida esto para agentes**. Es diseño, no copia.

## Lo que está a medias, y por qué se deja

- **2. Descomposición** — el criterio existe y está medido (por dependencia de estado, no por tamaño; cada unidad dentro de 20-30 K tokens; grafo re-ejecutable). Falta escribirlo como regla operativa.
- ~~**8. Puerta** — falta el vigilante~~ — **hecho**: contador de intentos por tarea, bloqueo al segundo fallo y aviso con el diagnóstico dentro. Despiece: [[vigilante-de-tareas]]. Probado en vivo el 2026-09-27.
- **11. Operación** — diseñado en [[devops-minimo]], sin estrenar.

## Lo que se aparca a propósito

**10. Despliegue.** Diseñado (CD en el merge, rollback revirtiendo, migraciones en dos tareas), pero **sin decidir dónde ni cómo se despliega**. No se cierra hasta que exista esa decisión.

## Ciclo de vida infinito — regla permanente

Este diseño **no se cierra nunca**. El estado del arte cambió tres veces durante esta misma investigación (un framework pasó a modo mantenimiento, un proyecto que di por descartado tenía benchmark, dos mediciones que creía firmes resultaron discutibles). Por tanto:

- **La revisión periódica que `AGENTS.md` ya exige para las skills y para el contenido investigado se aplica también a este diseño.** En cada `/lint`, o al tocar cualquiera de estas etapas: repetir la búsqueda de forma sistemática —no de memoria, no los mismos nombres ya conocidos— comprobar qué sigue en pie, y actualizar.
- **Lo que hay que vigilar especialmente**: si aparece la primera fábrica con **resultados medidos publicados** (hoy no existe ninguna); si algún framework cubre por fin la verificación; y si los umbrales prestados de otros dominios (5 reintentos, 3 intentos, 1-2 semanas) se sustituyen por cifras propias medidas aquí.
- **Y una advertencia para esa revisión:** el diseño tiende a crecer. Se midió que de 49 reglas añadidas a un proceso, **39 no mejoraron nada** y tres empeoraron el resultado. Cada añadido futuro tiene que traer su medición.

## Enlaces

- [[astillero]] · [[flujo-agentes-arquitectura]] · [[devops-minimo]]
- [[desarrollo-autonomo-con-agentes]] — la evidencia medida
- [[verificacion-externa-agentes]] — síntesis del principio de la etapa 7, el agujero del oráculo
- [[gas-city-frente-a-la-fabrica]] — contraste de las etapas 7, 8 y 12 contra Gas City: qué mecanismo se puede reutilizar y qué es marketing sin cifras
- [[verificacion-sin-oraculo-informe]] — el informe que respalda la etapa 7: cómo se implementa una verificación que el agente no puede tocar, con los ocho mecanismos y el agujero de cada uno

**Despiece de las etapas que tienen nota propia** (cada una desarrolla la fila correspondiente de la tabla de arriba):

- [[estado-de-verificacion]] — piezas 3 y 8: «cerrado» y «verificado», y quién puede escribir cada uno
- [[verificador-de-tareas]] — pieza 7: cómo se decide sin que el agente influya, y las cuatro piezas de GitHub por debajo
- [[recibo-de-verificacion]] — pieza 7b: qué queda escrito al verificar, y por qué un recibo que falta significa «no verificado»
- [[vigilante-de-tareas]] — pieza 8: cuándo una tarea pasa, se reintenta o se bloquea
- [[medicion-de-la-fabrica]] — pieza 12: las cinco de DORA más lo que hay que añadir
- [[github-pro-en-astillero]] — la puerta que falta en la etapa 9: qué habilita GitHub Pro y qué contratos dependen de él
