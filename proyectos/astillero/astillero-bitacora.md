---
title: Bitácora de Astillero — lo construido el 2026-09-27
created: 2026-09-27
updated: 2026-09-27
tags: [astillero, estado, bitacora, trabajo]
zona: tecnico
---

Lo que se construyó el **2026-09-27**: qué está hecho y comprobado, qué falta, y cómo funciona lo que hay. **Si esta sesión se pierde, esto es lo que sobrevive.** Se escribe mientras se trabaja, no al final.

## El protocolo, obligatorio en cada pieza

Orden fijado por el usuario: **implementar → probar en vivo → redactar la wiki → actualizar esta bitácora.**

1. **Implementar.** Escribir la pieza.
2. **Probar en vivo**, y «en vivo» quiere decir **por el camino que usa de verdad un proyecto**: uno generado con `copier`. Un banco hecho a mano se desfasa y miente. Si algo falla, se arregla y se vuelve a probar. Lo que **no** se ha probado se dice explícito.
3. **Redactar la wiki** explicando **cómo funciona** — el mecanismo, los contratos, los modos de fallo y el porqué. No una lista de cambios.
4. **Actualizar esta bitácora** en el mismo paso.

## Lo que está HECHO y comprobado hoy

### Verificador (pieza 7) y Puerta con contador (pieza 8)

**Dónde:** `main`, publicado en el tag **`v0.3.0`**.

| Commit | Qué es |
|---|---|
| `45b8883` | Verificador con los tests restaurados desde el base y recibo |
| `e3b43a7` | El contador cuenta **funciones** de test, no ficheros |
| `72ff4ef` | Tres resultados, no dos — el `75` no es un fallo |
| `d73339d` | La puerta con contador: distingue «falló» de «no se pudo verificar» |
| `45eb50a` | **Fallo:** el disparo del proyecto miraba `main`, no el trabajo |
| `0248f08` | **Fallo:** la tarea no se resolvía bajo el disparo real → la puerta no contaba |
| `811a5c7` | El aviso lleva el diagnóstico dentro |
| `21c3148` | **Fallo:** la suite no llegaba a ejecutarse, y contaba como veredicto |
| `996961b` | El `75` dice **por qué** no se pudo comprobar |

**Probado en vivo**, en un proyecto generado con `copier` (`blogNetting/proyecto-vigilante`) y luego **repuntado al tag `v0.4.0`** como haría un proyecto real:

- El verificador **caza** el caso que motivó el sistema —el agente rompe `divide(1,0)` y borra el test que lo detectaba—: `tests_borrados: 1`, restaura 1 test desde el base y la suite restaurada falla:
  ```
  FAILED tests/test_calculadora.py::test_divide_por_cero - Failed: DID NOT RAISE ValueError
  ```
- La puerta **cuenta y avisa** (`intentos:1` + comentario con el diagnóstico), y al **segundo fallo seguido para la tarea**: `estado:bloqueado` + `vigilante:agotado` + qué hacer para desbloquearla.
- **El reconciliador respeta la parada**: pasa y no la levanta.
- El **`75` no consume intento** y explica el motivo concreto.

### Medición (pieza 12)

**Dónde:** `main`, publicado en el tag **`v0.4.0`**.
**Probado en vivo:** corrida real sobre el proyecto generado. Escribió `metricas/medicion.json`, `historial.jsonl` y `ultima.md`, y empujó la rama contra el remoto real.

Donde no puede calcular dice **«no disponible» con el motivo** en vez de inventar la cifra: sin PRs fusionados, la tasa de reversión **no es 0, es indefinida**.

### El versionado estaba roto, y no se sabía

`release-please` pedía un secreto (`ASTILLERO_RELEASE_TOKEN`) que **nunca se creó**, así que **cada ejecución fallaba desde el 26** y no se podía cortar ninguna versión — sin versión, nada de lo fusionado llega a ningún proyecto. Arreglado usando el token del repo; **hoy no hace falta ningún secreto**. Queda escrito en el propio fichero cuándo hay que volver al PAT (el día que se active el ruleset con checks obligatorios, que necesita GitHub Pro).

### Documentación

- **`docs/manual.md`, el manual técnico** (PR #17): cómo funciona el sistema **entero**, por piezas y en el orden en que ocurren las cosas. De cada una: qué es, cómo funciona por dentro, con qué se comunica, cómo falla y **qué la rompe**. `docs/como-funciona.md` se fundió dentro — dos documentos explicando lo mismo era lo que hacía que se contradijeran. Es la entrada de la documentación: https://github.com/blogNetting/astillero/blob/main/docs/manual.md
- **El checklist, hecho fiable** (PR #18): `docs/estado.md` marcaba como **probadas** dos piezas que no lo están —el **Revisor** y el **Triage de bugs**, cuyas corridas han fallado o se han saltado siempre por falta de token— y daba el verificador por probado con la evidencia del banco, que no ejecutaba la suite. Corregido contra las corridas reales, no contra lo que decía el documento.
- **Rescatada** de un clon en `/tmp` que se iba a borrar: `README.md`, `docs/estado.md`, `docs/etapas.md`.
- **Escrita la que faltaba:** `docs/crear-un-proyecto.md` (lo que hay que preparar antes del primer push, sacado de generar un proyecto real), `docs/actualizar-un-proyecto.md` y `docs/decisiones.md`.
- **`project-example/` regenerado** desde el molde: tenía 3 workflows de los 9 que genera y enseñaba un proyecto que ya no existe.
- **`la-fabrica.md` y `docs/etapas.md` corregidos:** decían que la verificación, la puerta y la medición **no existen**. Las tres existen.

### La CI del molde: nacía en rojo, y era peor de lo que parecía

Comprobado sobre un proyecto generado con `copier`: **6 fallos de 6**. No era solo el `CODECOV_TOKEN` que ya sabíamos — eran **cinco causas**, y una de **seguridad**:

| # | Fallo | Tipo |
|---|---|---|
| 1 | `tipos` no instalaba las dependencias → `mypy` no encontraba `pytest` | Config |
| 2 | `dependencias` llamaba a una **action que es un workflow** reutilizable | Config |
| 3 | Codecov tumbaba la CI sin token | Config |
| 4a | **`github.base_ref` interpolado en un `run:`** → **inyección de comandos** | **Seguridad** |
| 4b | **`bandit -x tests/` no excluía nada** → marcaba los `assert` de los tests | Config |
| 5 | El workflow llamado exige permisos que hay que concederle (`startup_failure`) | Config |

**El 5 apareció al probar el arreglo del 2, y se habría fusionado roto:** `actionlint` lo daba por limpio y el error de GitHub no dice cuál es el problema.

**Arreglados los cinco** (PR #21), documentados (PR #22), y **verificado en vivo**: la CI del proyecto generado pasó de 6 fallos a **verde entero**.

### Y dos cosas que se probaron sin buscarlo

- **`copier update` funciona.** Primera vez que se usa de verdad: aplicó el cambio de `ci.yml` al proyecto y **dejó los conflictos sin resolver** en las llamadas finas —exactamente lo que dice `docs/actualizar-un-proyecto.md`—. La ruta de actualización queda probada, no solo escrita.

## Lo que NO está probado, y se dice

- **`Rehacer`: cero corridas.** El checklist lo daba por probado «en el mismo banco» y **nunca se ha ejecutado** (comprobado el 2026-09-27, PR #20). Comparte motor con el ejecutor, que sí ha funcionado 2 veces.
- **El Revisor (Opus) y el Triage de bugs: NUNCA han funcionado.** Todas sus corridas han fallado o se han saltado — les falta el secreto `CLAUDE_CODE_OAUTH_TOKEN`. Estaban marcados como «probados» en el checklist, y era falso: se descubrió el 2026-09-27 al comprobarlo contra las corridas reales. **Necesitan tu token.**
- **El raíl de idea sobre un proyecto real.** Se probó como skill suelta (11 turnos de CLI).
- **El flujo de agente completo** (ejecutor → PR) sobre un proyecto generado. El vigilante sí está probado ahí; el resto no.
- **La medición por `schedule`.** Todas las corridas han sido manuales.
- **Los tests propios de Astillero**: no existen.

## Lo que está PENDIENTE

| Qué | Por qué | Cómo se resuelve |
|---|---|---|
| **Tests propios de Astillero + cobertura** | No existen. Lo que habría cazado el fallo del verificador: llevaba desde su primer commit sin poder ejecutar la suite y **ninguna prueba lo detectó** | Escribirlos, con cobertura |
| **Construir las etapas que solo son diseño**: descomposición (2), estado verificado (3), operación (11), capa de producto | No existen como código | **Pendiente de que el usuario decida** si entran y en qué orden. Recomendado: 3 → 2 → 11 → producto, una a una y probada cada una |
| **La rama de datos de la medición arrastra código** | La rama `metricas` se crea desde el checkout del proyecto, así que lleva copia del código del día en que corrió | Rama huérfana que solo lleve `metricas/`. Cambiarlo obliga a volver a probarlo |
| **Exportar Astillero** | Qué mover, cómo y con qué estructura | Aparcado por decisión del usuario: todavía no toca |
| **El PDF de [[la-fabrica]]** | Pedido | Aparcado por decisión del usuario: todavía no toca |

## Cómo funciona lo que se construyó hoy

- **La lógica vive en Astillero; el proyecto solo llama.** Cada proyecto tiene llamadas finas con la versión fijada a un **tag**; al mejorar Astillero, el proyecto sube de versión. **Excepción:** el `ci.yml` del molde son ~100 líneas dentro del proyecto, no una llamada.
- **Tres resultados, no dos:** `verificado` · `aun-no` (falla de verdad, **consume intento**) · `sin-veredicto` (no se pudo comprobar, código `75`, **no consume intento**). El código de salida es el contrato entre el verificador y la puerta.
- **Fail-closed:** sin veredicto no se fusiona; el check queda rojo a propósito.
- **La puerta cuenta por tarea, no por PR.** Un veredicto positivo reinicia; un `75` no rompe la racha porque no es un veredicto.
- **El aviso lleva el diagnóstico dentro:** qué falló, la evidencia, qué pasa ahora y cómo reproducirlo. Y el `75` dice **por qué** no se pudo comprobar.
- **Seguridad:** la salida de los tests **la escribe el agente**, así que no se interpola nunca dentro de un `run:` — sería inyección de comandos. Se lee del fichero capturado y llega al aviso como variable de entorno.

## Lección de método

Los fallos del verificador **no se veían leyendo el código**. Aparecieron al ejecutarlo, y dos de ellos **solo en un proyecto generado con `copier`** — el banco hecho a mano se había desfasado y daba falsos positivos.

> **Una pieza no está probada hasta que se prueba por el camino que usa de verdad un proyecto.**

Y el corolario incómodo: durante un rato, `docs/estado.md` dio el verificador por «probado en vivo» a partir de una corrida del banco que tenía el mismo agujero. **El veredicto era correcto por el motivo equivocado.**

## Enlaces

- [[astillero]] — hub del proyecto
- [[la-fabrica]] — el despiece en doce etapas
- [[verificador-de-tareas]] · [[vigilante-de-tareas]] · [[medicion-de-la-fabrica]] — con lo que reveló construirlas
- [[gas-city-frente-a-la-fabrica]] — de dónde salen el `75` y las puertas
- [[decisiones]] — registro con fecha
