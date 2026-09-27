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

### La 10: el ejecutor, y un fallo que lo mataba entero

Al probar el flujo de agente completo por primera vez sobre un proyecto generado, el ejecutor **murió en la activación**:

```
ERR_API: Failed to process runtime import for
.github/workflows/blogNetting/astillero/.github/workflows/shared/implementar-core.md
```

**Qué pasaba, leído del registro, no supuesto:** el fichero tenía **dos** imports del núcleo compartido. El de compilación (`.github/aw/imports/.../implementar-core.md`) **funcionaba** — es la copia cacheada al compilar. El otro, un macro de import **en tiempo de ejecución**, se resolvía a una **ruta local inexistente** y tumbaba todo.

**Y era redundante por diseño:** el research dice que los imports remotos de gh-aw **se resuelven al compilar** y el `.lock.yml` queda congelado con lo que había en el ref — que es justo el objetivo, fijar una versión, no traer la última en cada ejecución. El macro de ejecución iba en contra de eso.

**Corregido** en las plantillas `implementar` y `rehacer`, con el motivo escrito al lado para que no se vuelva a añadir. **Pendiente de fusionar** hasta que el reejecutado confirme que el agente arranca.

**Y esto obliga a corregir una afirmación mía:** cerré el pendiente viejo de «parametrización de los imports de gh-aw» diciendo que **ya estaba resuelto** porque el molde los parametriza. El de compilación sí; **el de ejecución estaba roto**. Lo cerré sin probarlo, que es exactamente lo que la regla prohíbe.

### El revisor: nunca funcionó, y eran DOS fallos encadenados

Se puso el token y se relanzó por primera vez. Falló. Leído del registro:

```
Could not fetch an OIDC token. Did you remember to add `id-token: write`
to your workflow permissions?
```

**1. Faltaba `id-token: write`.** `claude-code-action` se autentica por **OIDC**, y ni `revisar.yml` ni `reproducir.yml` lo concedían. Peor: las **llamadas finas del molde no declaraban bloque `permissions` en absoluto** — y un workflow reutilizable **no puede elevar** los permisos de quien lo llama; sin bloque, recibe los **por defecto del repo**. Es el **mismo tipo de fallo** que el del `osv-scanner` (un workflow que usa una action sin darle lo que necesita), y el error no dice en cuál falta.

**2. El disparo.** El revisor se lanza por `workflow_run` y, en ese evento, `PR_NUMBER` sale **vacío** — igual que le pasaba al vigilante, que ya se corrigió. Está anotado: **es el tercer sitio donde aparece el mismo patrón de `workflow_run`**, y hay que comprobarlo en cada workflow que lo use.

**Lo que esto dice del método:** el revisor llevaba **desde el 2026-09-26 sin funcionar**, y el checklist lo daba por «probado». No lo cazó nadie leyendo: salió al **poner el token y ejecutarlo**. Es el cuarto caso hoy del mismo patrón — *lo que no se ejecuta, no está probado*.

### El ejecutor: verde, y sin hacer nada

Con el import arreglado, el agente **arrancó y terminó en `success`** — la primera vez que el flujo corre entero desde un proyecto generado. **Y no abrió ninguna PR.** El propio registro lo decía:

```
Firewall blocked 3 domains
  - api.anthropic.com
  - files.pythonhosted.org
  - pypi.org
```

**La palabra clave del ecosistema no basta.** `network.allowed` llevaba `python`, y eso **no cubre de dónde salen los paquetes**: el firewall bloqueaba PyPI. Sin poder instalar dependencias el agente no puede ejecutar los tests, así que no puede hacer el trabajo…

**…y el workflow termina en VERDE igual.** Ese es el peor modo de fallo de todos: **parece que funciona**. Un job en `success` sin PR es más peligroso que un job en rojo, porque nadie va a mirarlo.

Corregido en las dos plantillas, añadiendo los registros del ecosistema a `network.allowed`. **Pendiente de probar en vivo.**

**Cuarto fallo del ejecutor hoy**, y todos del mismo tipo: **la pieza existía, se daba por buena, y no funcionaba**. Los cuatro salieron al ejecutarla, ninguno leyéndola.

### Y un fallo mío, que reintrodujo el primero

Al arreglar lo de la red creé la rama **desde `main`** y regeneré los ficheros del proyecto desde esa plantilla. Pero **`main` todavía no tiene el arreglo del import** (está en su rama, sin fusionar), así que **volví a meter el macro roto** y el ejecutor murió otra vez con el mismo error de la activación.

**La lección, y es de método:** cuando varios arreglos viven en **ramas sin fusionar**, regenerar desde `main` **deshace los que no estén dentro**. Hay que **juntarlos antes** de probar, o probar siempre desde el mismo sitio.

**Corregido:** una rama `fix/agente-todo` con **los dos arreglos juntos** (import fuera + red abierta), verificada en el `.lock.yml` compilado (`pypi.org` presente, macro roto con cero apariciones).

### El revisor YA funciona — y lo que le queda

Con el token y los dos arreglos, **el revisor corre por primera vez**: Opus, **10 turnos, 21 segundos, 0,15 $ estimados** (`is_error: false`). Frente a los 0,5 s y coste 0 de sus intentos anteriores, que ni arrancaban.

Está **fusionado y probado** lo que hacía falta para llegar hasta ahí: `id-token: write` (autenticación por OIDC) y el `ref` del checkout (leía `main` en vez de la PR).

**Lo que le falta:** **publicar la revisión.** Bajo `workflow_run` no tiene PR sobre la que comentar — el mismo patrón del `workflow_run` que apareció en el vigilante y aquí otra vez. Las salidas son cambiar el disparo a `pull_request` o derivar el número de PR del evento. **Pendiente de decidir.**

**Y una duda abierta, sin explicación:** el consumo de Opus del revisor **no aparece** en el medidor de la suscripción (una cuenta, 20 minutos, 0,15 $ estimados, 0 %). Es contabilidad de Anthropic, no del sistema — no se investiga más por ahora.

### Las ramas de hoy, y por qué es el hueco más grande

Los arreglos de hoy viven en **diez ramas sin fusionar**. Solo entraron en `main` el CI y la documentación. **Mientras no se fusionen, nada de esto llega a ningún proyecto**, y varias se pisan entre sí (dos tocan `revisar.yml`).

Lo correcto es **una sola rama con todo y un PR**, no diez, y **publicar la versión** después.

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
