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

### El ejecutor: hace el trabajo, y la PR no se crea

Con los dos arreglos juntos, el agente **trabaja de verdad** — es lo mejor que ha salido hoy:

- Lee el contrato, el código y los tests.
- **Se apaña con el firewall**: monta un venv, instala, y **ejecuta pytest y mypy**.
- Escribe el código y los tests, y **crea la rama `feat/resta-calculadora`**.
- **Pide la PR**: `safe-output-items manifest: 1 item(s) logged (types: create_pull_request)`.

Y la PR **no se crea**:

```
App token minting failed (safe_outputs/conclusion/activation): false/false/false
```

**Por qué.** `gh-aw` separa «el agente pide» de «un job con permisos lo ejecuta», y para eso **mintea un token de app**. Según su documentación, eso se configura con un bloque **`github-app:`** (una GitHub App con sus credenciales). **No hay ninguna configurada**, así que intenta el respaldo —y en el workflow compilado **`id-token` aparece 0 veces**, o sea que el respaldo por OIDC tampoco está disponible.

**Tercera vez hoy del mismo tipo de fallo:** una pieza que necesita una credencial o un permiso que no se le ha dado, con un error que no dice cuál falta.

**Lo que hace falta decidir:** configurar una **GitHub App** para los proyectos (o el respaldo por OIDC). Es configuración, no código — y afecta a todos los proyectos, así que va en la misma lista que el token.

**Y el resumen honesto del día con el ejecutor:** de «no arranca» a «hace el trabajo y no puede entregarlo». Cuatro fallos por el camino (import, red, permisos, y este), los cuatro invisibles leyendo.

### El revisor, disparado por etiqueta (decisión tomada y construida)

**El problema, con la causa exacta:** la action de Claude **deduce la PR del evento** — no acepta que se la digan (sus entradas son `trigger_phrase`, `assignee_trigger`, `label_trigger`). Con `workflow_run` no hay PR que deducir, así que corre y no publica.

**Lo que pide el research:** K7 dice *«CI → revisor, con la CI en verde»*, y K8 *«veredicto contra el contrato: review o comentario; si hay cambios, etiqueta `agente:rehacer`»*. Las dos a la vez.

**La solución, en dos pasos:**
1. La CI termina en verde → un job **marca la PR** con la etiqueta `revisar`.
2. Esa marca dispara al revisor con `pull_request: types: [labeled]` — **con la PR en el contexto**, y **después de la CI**.

Se añade `label_trigger: revisar` en el reutilizable (su valor por defecto es `claude`), que es lo que hace que la action active con el evento de etiqueta.

**Por qué no la opción simple** (`pull_request` a secas): publicaría, pero revisaría **antes** de que la CI acabe — trabajo que puede no compilar. Rompe K7.

**Pendiente de ver en vivo.**

### Y un fallo mío que rompió el fichero del revisor

Al **resolver el conflicto** entre las dos ramas del revisor me dejé una línea `permissions:` huérfana antes del bloque real. Con el bloque vacío, **GitHub no carga el workflow**: lo lista **por su ruta** en vez de por su nombre, y cualquier disparo muere al arrancar.

**Se detectó porque la prueba en vivo lo dijo sin rodeos** — mi espera buscaba «Revisar con Opus» y respondía *«could not find any workflows named Revisar con Opus»*. Leyendo el fichero no se veía: el `name:` estaba bien escrito; lo que sobraba era una línea vacía veinte líneas más abajo.

**Corregido**, y anotado como lo que es: **resolver un conflicto a mano también es escribir código, y también hay que validarlo.** `actionlint` lo cazaba —no se pasó antes por él— y ahora pasa.

### Y el ancla de verdad: lo que puedo y no puedo hacer

Hoy se cerró con una conclusión incómoda pero útil, que no es sobre Astillero sino sobre cómo se trabaja aquí. El usuario lo dijo sin rodeos: *«estoy hasta los cojones de que falles y hagas lo que te salga»*.

**Lo que no funcionaba:** una regla escrita (no se cumple sola), y un hook que comprueba un indicador (**se puede satisfacer en falso** — pasó hoy dos veces: tocar la bitácora mientras el plan se quedaba viejo, y tocar un `.md` cualquiera).

**Lo que sí:** reglas de **prohibición** en la configuración. Comprobado en vivo que **se respetan incluso en `bypassPermissions`** — el harness deniega el comando y no llega a ejecutarse. Es lo mismo que el research dice para el agente de Astillero: *«el límite se pone con permisos, no con instrucciones»*.

**Prohibido desde hoy:** fusionar PRs · escribir o borrar secretos · crear, borrar o editar releases · escrituras por la API de GitHub. `git push` queda **a propósito** fuera: es reversible y es como se comparte el trabajo — se añade en una línea si el usuario quiere control total.

Detalle y prueba: [[entorno]].

### Cerrar de verdad: lo que se puede quitar, se quita

El usuario lo dijo sin rodeos: *«que se cumplan mis órdenes, haz lo que sea para que se cumpla y no pase más veces»*. Y tenía razón en el diagnóstico: **una orden que depende de que el modelo se acuerde no es una orden.**

**Lo que se hizo, y por qué este orden:**

1. **Se probó qué sostiene de verdad.** Se comprobó en vivo que una regla de **prohibición** (`deny`) **se respeta incluso en `bypassPermissions`** — el harness deniega y el comando no llega a ejecutarse. Eso no depende de la memoria.
2. **Se prohibió todo lo que sale hacia fuera:** fusionar PRs · escribir o borrar secretos · crear, borrar o editar releases · escrituras por la API · **y `git push`**. Nada sale de manos del modelo; el push lo hace el usuario.
3. **El hook de documentación, estrechado dos veces:** primero exigió un documento **de seguimiento** (no cualquier `.md`), y después que ese documento vaya **después** del código — documentar antes y cambiar después deja el documento describiendo algo que ya no es cierto.
4. **Los cuatro casos, probados uno a uno** antes de darlos por buenos: código sin documentar → bloquea · código con un `.md` cualquiera → bloquea · bitácora antes del código → bloquea · código y después la bitácora → pasa.

**La escala, que era lo que faltaba entender:** una regla escrita **no se cumple sola** · un hook que mira un indicador **se puede satisfacer en falso** · **una prohibición no se puede saltar**. Cuando importe de verdad, va al tercer escalón.

**Lo que sigue sin poder garantizarse, y se dice:** los hooks comprueban **que se tocó el documento correcto y en el orden correcto** — no que lo escrito sea **verdad**. Eso solo lo ve el usuario.

### El hook también se equivocaba, y se arregló

Al estrecharlo para que la documentación fuera **después** del código, empezó a bloquear turnos que sí estaban bien. La causa, encontrada depurándolo contra el transcript real:

**El comando con el que se escribía la bitácora contaba como «código»** — porque **el texto que se redactaba dentro mencionaba `git push` y `git commit`**. El filtro buscaba esas palabras en todo el comando, incluido el contenido que se está escribiendo. Es decir: **escribir sobre git contaba como hacer git.**

Corregido: solo cuenta si el comando **ejecuta de verdad** esas órdenes — al principio o tras `&&`, `;` o `|`. Un `git push` **citado dentro de un texto** ya no engaña al hook.

**Y un hueco que queda, dicho claro:** el hook solo ve el código escrito con las herramientas `Edit`/`Write`. **Si el código se escribe desde `bash`** —como hice yo casi todo el día, con bloques de `python` que escriben ficheros— **no lo ve.** Dos salidas: estrechar más el filtro (con riesgo de falsos positivos al confundir texto con órdenes) o **dejar de escribir código desde `bash`** y usar la herramienta de edición, que además deja el cambio a la vista.

### El agujero de verdad del hook: no veía el trabajo real

Y era el peor de todos. El hook **excluía del recuento todo lo que estuviera en `/tmp`** —se puso así para que un fichero de pruebas no contara como código— y resulta que **los clones de Astillero viven en `/tmp`**. Consecuencia: **el hook nunca ha visto mi trabajo de Astillero.** Podía tocar lo que quisiera y él no se enteraba.

Corregido: se excluyen **los ficheros sueltos de `/tmp`** (`/tmp/algo.txt`), no los árboles de trabajo (`/tmp/astillero-docs/...`).

**Y un segundo fallo que salió al buscar el primero:** el botón decía «documentado» cuando solo se había tocado la bitácora. El manual —que es **la explicación**— no lo exigía nadie. **Justo al revés de la prioridad del usuario: «primordial redactar documentación de cómo funciona Astillero».**

Ahora el hook exige, cuando se toca código **de Astillero**, las **dos** cosas: **el manual** (cómo funciona) y **uno de seguimiento** (el plan o la bitácora), y **el manual después del código**.

**La batería, ocho casos, todos comprobados uno a uno:** código solo → bloquea · con un `.md` cualquiera → bloquea · con bitácora y sin manual → **bloquea** · con manual y bitácora → pasa · código de la máquina → pasa (no es Astillero) · bitácora antes del código → bloquea · y después → pasa.

### La documentación, revisada contra el código: cuatro contradicciones

Se repasó **el manual entero, afirmación por afirmación, contra los ficheros reales**. No que las secciones existieran — que lo que dicen sea verdad. Cuatro cosas no cuadraban, y las cuatro son **errores de Astillero o del manual**, no del research:

| # | Contradicción | Quién manda |
|---|---|---|
| 1 | El manual decía que **el recibo lleva el motivo**. **No lo lleva**: guarda código de salida y recuentos, no la salida | **El research**: *«status codes are lies, outputs are evidence»* → el recibo se queda corto |
| 2 | **`estado:en-curso` no lo pone nadie** | El research lo dibuja (arquitectura §5) → **hueco del código** |
| 3 | **`estado:rehacer` y `estado:humano` no existen**: el tope de rondas no está implementado | El research lo pide → **hueco del código** |
| 4 | **El bucle de rehacer está desconectado**: `rehacer.md` se dispara con `agente:rehacer` y **nadie pone esa etiqueta** — el revisor debería, pero no tiene `issues: write` | El research (K9) → **hueco del código** |

Y una **imprecisión del manual**, corregida: decía «el agente no puede etiquetar». No puede por sí mismo, pero **sí puede pedir** etiquetas por las salidas seguras de `gh-aw`, que aplica un job aparte — solo del patrón `estado:*`.

**Y un fallo mío, del mismo día y del mismo tipo:** dos filas del checklist llevaban sin actualizar desde la mañana porque **mis ediciones anteriores no se aplicaron y no lo comprobé**. El Ejecutor seguía en «probado» cuando lo real es que hace el trabajo y no lo entrega. Es exactamente el fallo que el usuario señala: dar por hecho sin abrir el resultado.

### El cotejo research ↔ montado: los doce contratos, uno a uno

El research define **doce contratos** entre piezas (`flujo-agentes-arquitectura` §4). Comprobados contra el código el 2026-09-28, uno por uno:

| Contrato | Qué es | ¿Construido? |
|---|---|---|
| **K1** · Diseñador → repo | La especificación (`specs/<feature>/spec.md`, `plan.md`, `tasks.md`) como PR | ❌ **No** |
| **K2** · Persona → repo | Aprobación del diseño por revisión de esa PR | ❌ **No** |
| **K3** · Desglosador → tracker | Una épica por *feature* y una sub-issue por tarea, con su contrato y dependencias | ❌ **No** |
| **K4** · Tracker → ejecutor | Etiqueta `agente:implementar` de un solo uso | ✅ El `label_command` de gh-aw, que la retira solo |
| **K5** · Ejecutor → repo | PR con prefijo `[agente] `, `Closes #N` y etiqueta `estado:en-revision` | ✅ Las tres, en `shared/implementar-core.md`. Y la idempotencia: *«si la tarea ya tiene PR abierta, termina sin hacer nada»* |
| **K6** · PR → CI | Checks obligatorios por ruleset | ❌ **No** — necesita GitHub Pro en repo privado, confirmado por el research |
| **K7** · CI → revisor | El disparo del revisor | ✅ Arreglado el 2026-09-27: **por etiqueta tras la CI** |
| **K8** · Revisor → PR | Veredicto, y `agente:rehacer` si pide cambios | ⚠️ **A medias**: comenta, pero **no puede etiquetar** (sin `issues: write`) |
| **K9** · PR → ejecutor (rehacer) | Etiqueta `agente:rehacer` + empujar a la rama de la PR | ⚠️ **Configurado y desconectado**: `push-to-pull-request-branch` está en `shared/rehacer-core.md`, pero **nadie pone la etiqueta** |
| **K10** · PR aprobada → integración | Merge queue | ❌ **No** — mismo motivo que K6 |
| **K11** · Merge → tracker | `Closes #N` cierra la issue | ✅ Nativo de GitHub |
| **K12** · Cierre → reconciliador | Promover lo desbloqueado con `estado:listo` + `agente:implementar` | ✅ `reconciliar.yml`, con `issues: closed` y `blockedBy` |

**Lo que dicen las doce, en una frase:** **cinco no existen** (K1, K2, K3, K6, K10), **dos están a medias y por el mismo motivo** (K8 y K9: el revisor no puede etiquetar, así que el bucle de rehacer no arranca nunca), y **cinco funcionan**.

**Y el hueco de los tres primeros no es un descuido suelto: K1, K2 y K3 son la entrada del sistema** — convertir una idea en especificación, que alguien la apruebe, y partirla en tareas. Es lo que en el plan son la **descomposición (etapa 2)** y la **capa de producto**. **Hoy el sistema sabe ejecutar y verificar, y no sabe empezar.**

### Más errores: el contrato ✓, la cobertura ✗

**El contrato de tarea, comprobado y COINCIDE.** El research pide ocho secciones (`arquitectura` §6) y la plantilla del molde **las trae las ocho**: Objetivo · Contexto · Alcance · Interfaces · Criterios de aceptación (EARS) · Riesgos de intención fuera del EARS · Tests obligatorios · Hecho cuando. Sin huecos.

**Y la cobertura, no coincide — falta el control que el research llama «el correcto».**

El research lo pone como **obligatorio** en dos sitios:

> *«coverage.py/Vitest v8 + **Codecov patch gate obligatorio**, sin umbral global»* (`desarrollo-agentes-f4-devsecops` §159)
> *«Patch/diff coverage — Gate específico para "¿las líneas que tocó este PR están testeadas?" — **el control correcto**»* (`f4-devsecops` §138)

**Lo montado:** el job instala coverage, lo ejecuta, genera `coverage.xml` y lo sube a Codecov con `fail_ci_if_error: false`. Y **no hay ningún fichero de configuración de Codecov en el repo** — comprobado buscando en el árbol de `main`. Es decir: **se sube un número y nadie comprueba nada**. No hay gate por cambio, y tampoco el umbral global que el research descarta — simplemente **no hay gate**.

**Por qué importa, con las palabras del propio research:** la cobertura de línea «es necesaria pero no suficiente y por sí sola es fácil de engañar por un agente que genera tests triviales para cumplir el número». El patch gate es justo lo que **impide subir el número sin cubrir lo nuevo**.

### El hallazgo que más pesa: **el fail-closed no está en vigor**

El manual afirma, y el diseño entero se apoya en ello:

> **«Fail-closed: sin veredicto no se fusiona; el check queda rojo a propósito.»**

**Y eso hoy no es verdad.** Comprobado el 2026-09-28 contra la API real:

```
rulesets de main           → 403 «Upgrade to GitHub Pro…»   → NO EXISTEN
branch protection de main  → 403 «Upgrade to GitHub Pro…»   → NO EXISTE
```

**Un check rojo no impide fusionar si nadie lo declara obligatorio.** Sin ruleset, **ninguno de los trabajos de la CI bloquea nada** — ni el verificador, ni el tipo estricto, ni los tests. El «check rojo a propósito» del verificador es hoy **información en una pantalla, no una puerta**. Cualquiera con permiso de escritura puede fusionar una PR con todo en rojo.

**Por qué no es un descuido menor:** medio sistema está diseñado alrededor de esa puerta. El verificador, la puerta con contador y el contrato de los tres códigos **asumen que su veredicto impide la fusión**. Si no la impide, lo que hay es un semáforo que nadie mira.

**Y la causa está documentada desde antes:** el research ya lo dijo — *«requiere GitHub Pro en repo privado, confirmado con la API real el 2026-09-25»* (K6). **Lo que faltaba era decirlo en el manual**, donde hoy se afirma lo contrario.

### Y tres huecos más del molde, del mismo cotejo

| Qué pide el research (`arquitectura` §8) | Lo que hay |
|---|---|
| **Cobertura del diff** — «Codecov `patch` status, sin umbral global… el control correcto» | ❌ **No existe**: no hay ni fichero de configuración de Codecov. Se sube el número y **nadie comprueba que lo nuevo esté cubierto** |
| **Mutación del diff** — mutmut **sobre los ficheros cambiados**, filtrando antes los mutantes triviales o «generan ruido de CI sin señal» | ⚠️ **Corre pero no es puerta**: `mutmut run` + `mutmut results`, y **nunca falla**. Si sobreviven todos los mutantes, la CI pasa igual. Y **no filtra** los triviales, que es justo el ruido que el research avisa |
| **Dependencias** — «OSV-Scanner **+ Dependabot**» | ⚠️ **OSV sí, Dependabot no**: el árbol de `main` no tiene `.github/dependabot.yml` |

**Y ya está corregido en la documentación** (rama `docs/manual-completo`, PR #27): el manual y `decisiones.md` dicen ahora que **la puerta no está en vigor** y que **la puerta real, mientras tanto, es una persona**.

**El resumen del cotejo, hoy:** de los doce contratos, **cinco no existen** (los tres de la entrada del sistema, y los dos que piden GitHub Pro); **dos están rotos por el mismo motivo** (el revisor no puede etiquetar → el bucle de rehacer no arranca); y **ninguno de los controles de CI bloquea nada**, porque **sin ruleset no hay checks obligatorios**.

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
