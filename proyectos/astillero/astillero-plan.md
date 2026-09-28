---
title: Plan de trabajo de Astillero
created: 2026-09-27
updated: 2026-09-28
tags: [astillero, plan, trabajo]
zona: tecnico
---

**Lista numerada y completa** de todo lo que queda, con quién depende de quién. Lo ya hecho está en [[astillero-bitacora]]. Se actualiza al cerrar cada tarea.

## Dos reglas que se aplican a TODO lo de aquí

1. **Nada se modifica en Astillero sin venir contrastado con el research.** Si el cambio no está respaldado por una nota del research, **no se hace**: o se pide, o se hace un research nuevo. Y queda dicho de qué nota sale.
2. **Cada tarea cierra con las cuatro cosas:** modificar → **validar siempre** → verificar que funciona por el camino real → **documentar cómo funciona** (en el manual, no en el checklist). Si no está documentado, la tarea no está terminada.

## En curso, PARADA

**1 · Estado verificado (etapa 3).** Empezada y parada a petición. Guardada en la rama `feat/estado-verificado` (**sin PR**, marcada incompleta): solo tiene el esqueleto `marcar_estado`.
- **Falta:** llamarlo en las tres ramas (veredicto → `verificado`; fallo → `sin-verificar`; lectura ciega `75` → `no-verificable`), probar en vivo sobre un proyecto generado con `copier`, redactar y actualizar bitácora.

## Fase 1 — lo siguiente (no depende de nadie)

**2 · Tests propios de Astillero, con cobertura.**
Qué: que el YAML valide, que las plantillas rendericen, que las llamadas finas declaren lo que el workflow llamado acepta, y que la lógica del veredicto acierte en cada caso (0 / 75 / fallo / test borrado / no arrancó).
Listo cuando: corra en CI y **falle** si se reintroduce a propósito cualquiera de los tres fallos de hoy.

**3 · El manual técnico de Astillero — ✅ PRIMERA VERSIÓN HECHA** (PR #17).
`docs/manual.md`, en `main`: cómo funciona el sistema entero, por piezas y en el orden en que ocurren las cosas. Se escribió fundiendo `como-funciona.md` en él.
**Lo que falta:** separar lo que tengo claro de lo que no. Lo de hoy está redactado contra el código y las ejecuciones; lo anterior a esta sesión hay que **releerlo, probarlo y validarlo** antes de darlo por escrito. Y sigue creciendo con cada cambio, en el mismo paso.

## Fase 2 — las etapas que hoy solo son diseño

Una a una: implementar, probar en vivo, redactar, actualizar bitácora. **No se apila.**

**4 · Estado verificado (etapa 3)** — es la tarea 1 cuando se retome. Va primera porque **completa lo que ya está a medias**: el verificador ya emite un veredicto y nada lo escribe en el estado de la tarea.

**5 · Descomposición (etapa 2)** — partir el contrato en tareas **por dependencia de estado**, no por tamaño. Va después porque **toca el despacho, que hoy funciona**.

**6 · Operación (etapa 11)** — monitorización, alertas, copias de seguridad, rotación de secretos.

**7 · Capa de producto** — captura de idea, panel, bugs, versionado: quién dirige el motor.

## Fase 3 — huecos de lo que ya está construido (no depende de nadie)

**8 · Medir la intención.** El hueco que le queda al verificador: hoy comprueba «pasa», no «es lo que se pidió». Un cambio que pasa los tests pero no resuelve lo pedido **pasa**. Es la parte de la etapa 7 que nunca se cerró.

**9 · Probar el raíl de idea sobre un proyecto real — DEJADO POR TI.** Se propuso hacerlo sobre [[patrimonial]], y dijiste que **ahora no quieres meterte en eso**. Se retoma cuando tú digas.

**10 · El flujo de agente completo — el agente YA funciona; falta la identidad.**
Probado en vivo sobre un proyecto generado, y es lo mejor que salió hoy: el agente **lee el contrato, se apaña con el firewall montando un venv, escribe el código y los tests, y crea la rama**. **Pide la PR** (`create_pull_request` en las salidas seguras) y **no se crea**: `App token minting failed`.

**Lo que falta, y es tuyo:** una **GitHub App** que le dé identidad de bot. El paso a paso completo está en `docs/crear-un-proyecto.md` del repo, y el bloque `safe-outputs: github-app:` ya está en las plantillas. **Sin ella el agente hace el trabajo entero y no lo puede entregar.**

**Por qué una App y no el respaldo por OIDC:** el research lo pide — *«identidad de bot vía GitHub App, el autor del PR no es una cuenta humana»* y *«identidad diferenciada por agente, no un token compartido»*. Con OIDC el trabajo saldría como `github-actions`, sin atribución.

**11 · La medición automática — ✅ CERRADO, era un falso pendiente.** Comprobado el 2026-09-28 contra las ejecuciones reales del proyecto de prueba: `Medir` **sí dispara por `schedule`** (corridas reales a las 03:14 y 10:02 de hoy, ambas `success`). Lo que decía este punto ya no era cierto cuando se leyó; se corrige aquí en vez de dejarlo como pendiente abierto.

**12 · La rama de datos de la medición — ARREGLADA, SIN PROBAR.** Arrastraba una copia entera del código del proyecto; ahora se crea **huérfana** y solo lleva `metricas/`. Publicado en la `v0.5.0`. **Falta probarla**, y para eso hay que partir de un proyecto sin esa rama.

**22 · El revisor: que publique su veredicto — ⚠️ PROBADO CON EL PROMPT VIEJO, NO CON EL ACTUAL.**
Es el agente que lee una PR y **la compara contra el contrato de la tarea**. Lo de aquí arriba (disparo por etiqueta tras la CI, contrato K7) **resultó imposible de construir**: GitHub no crea ejecuciones a partir de eventos que dispara el propio `GITHUB_TOKEN`, y la etiqueta nunca disparaba nada. Se cambió a `pull_request` directo, sin esperar a la CI — **desviación explícita de K7**, no un descuido: el revisor ya puede correr con la CI en rojo.
**Probado en vivo de verdad, pero con el prompt de ANTES de K9:** sobre la PR #13 de `proyecto-vigilante` (00:53 del 2026-09-28), con veredicto real publicado y una segunda pasada reconociendo qué cambios ya se habían corregido. Ese prompt no tenía la línea `VEREDICTO-MAQUINA` que se le añadió después para K9.
**Corrección del 2026-09-28 noche, encontrada al preguntarme el usuario directamente si el revisor comenta bien:** **desde que se añadió esa línea, ninguna ejecución ha comentado nada.** Comprobado en las 15 corridas de hoy: las que tocaban `revisar.yml` en la misma PR se saltaron por la protección de `claude-code-action` contra workflows modificados (`Workflow validation failed`) — **y el job salía en `success` igual, escondiéndolo.** No hay ninguna prueba de que el prompt actual siga comentando bien. Detalle completo en [[astillero-bitacora]].
**Cómo se cierra de verdad:** una PR que no toque `revisar.yml`, contra `main` (ya en `v0.7.0`). Pendiente, no hecho.
**Y sigue faltando, tarea 24:** etiquetar `agente:rehacer` cuando pide cambios (K9) — sin eso el bucle de rehacer sigue sin arrancar nunca, y ahora tampoco está confirmado que llegue a pedir cambios correctamente.

**13 · Que la CI del molde no nazca en rojo — ✅ HECHO.** Cinco fallos encontrados, arreglados, documentados y verificados: la CI del proyecto generado pasó a **verde entero**. Comprobado el 2026-09-27 sobre un proyecto generado: **6 fallos de 6**, por **cuatro causas distintas** y solo una es el token que ya sabíamos:
1. **`tipos`** — instala `mypy` pero **no las dependencias del proyecto**, así que mypy no encuentra `pytest`. Fallo del molde.
2. **`dependencias`** — el SHA fijado de `google/osv-scanner-action` **apunta a algo que no es una action** (`Top level 'runs:' section is required`). Fallo del molde.
3. **`tests-cobertura`** — Codecov con `fail_ci_if_error: true` y sin `CODECOV_TOKEN`. Fallo del molde.
4. **`sast`** — eran **dos** cosas, las dos del molde: (a) semgrep marcaba **`github.base_ref` interpolado dentro de un `run:`** → **inyección de comandos**, el **único fallo de seguridad** de los cinco; (b) **`bandit -r . -x tests/` no excluía nada** —bandit compara por prefijo y con `-r .` las rutas salen como `./tests/...`—, así que escaneaba los tests y marcaba sus `assert` (B101), correctos en un test.

**Y un quinto, que apareció al probar el arreglo del 2:** al convertir `dependencias` en llamada a un workflow, GitHub **no cargaba el fichero** (`startup_failure`) porque el workflow llamado exige `actions: read`, `contents: read` y `security-events: write`. **Ese se habría fusionado roto**: `actionlint` lo daba por limpio y el error de GitHub no dice cuál es el problema. Es el mejor argumento para la regla de probar siempre.

Un agente que empiece en un proyecto nuevo se pelea con la CI en vez de con su tarea.

**Arreglados los tres del molde** (rama `fix/ci-del-molde`, pendiente de probar en vivo antes de fusionar): `tipos` instala las dependencias antes de mypy; `dependencias` pasa a llamar al **workflow reutilizable** (no a una action) con `upload-sarif: false`, porque en repo privado el SARIF exige GitHub Advanced Security; y Codecov deja de tumbar la CI sin token, con un aviso explícito para que la ausencia se vea.

**Y en la prueba salió algo sin querer, bueno:** `copier update` **se ha usado por primera vez de verdad** y funciona — aplicó el cambio de `ci.yml` y dejó los conflictos sin resolver en las llamadas finas, exactamente como dice `docs/actualizar-un-proyecto.md` del repo.

**El research que respalda las herramientas es [[desarrollo-agentes-f4-devsecops]]** — los arreglos son corregir la configuración, no cambiar de herramienta.

**14 · Los cuatro pendientes viejos del hub — ✅ HECHO.** Comprobados uno a uno contra el research y el código: **los tres primeros están resueltos** (plataforma = GitHub, medida en el research; trunk-based, decidido y en uso; los imports de gh-aw, ya parametrizados en el molde). **El cuarto** —cuándo arranca el piloto— **sigue abierto y es decisión tuya** (tarea 19).

**15 · Subdividir `proyectos/astillero/`.** Pasó de 30 notas (37 + bitácora + plan). El reparto está preparado; **no se ejecuta sin tu OK**.

**21 · Comprobar que el token de Opus del revisor hace algo de verdad.**
**Pedido por el usuario, y con motivo: «no hay consumo y no me fío de lo que dices».**

Lo que hay **comprobado**, y es solo del registro del runner:
- `ANTHROPIC_BASE_URL` **vacío** → no apuntaba a DeepSeek.
- El CLI reportó `"model": "claude-opus-5-5"`, `num_turns: 10`, `duration_ms: 21197`, `is_error: false`, `total_cost_usd: 0,15`.

Lo que **no cuadra y no tiene explicación**:
- El medidor de uso de su suscripción sigue en **0 %** — una sola cuenta, y pasados 10-20 minutos, así que no es retraso.

**Por qué no vale lo que dije:** todo lo anterior es **lo que el propio CLI declara de sí mismo**. Que un programa diga que llamó a Opus no prueba que la llamada llegara a Opus. Es exactamente el error que este proyecto persigue: *«el verde del agente no es evidencia de nada»*.

**Lo que sí lo cerraría**, y hay que hacerlo desde fuera del runner:
- Comparar el consumo de la cuenta **antes y después** de una revisión, con la misma ventana de tiempo.
- O mirar del lado de la cuenta si aparece la llamada (logs de la organización / facturación).
- Si no aparece por ningún lado: **el revisor no está gastando tu cuota, y eso cambia lo que se puede esperar de él**.

**23 · El verificador salía en rojo con veredicto limpio — ✅ ARREGLADO, PROBADO EN VIVO Y FUSIONADO.**
Encontrado el 2026-09-28 auditando el repo contra GitHub: la última orden del paso `clasificar` de `verificar.yml` era `[ -n "$motivo" ] && printf ...`; con veredicto limpio el motivo queda vacío, esa comprobación da falso (código 1), y al ser la última orden del paso `bash -e` la tomaba como el resultado del paso entero. **Cada PR que verificaba bien salía con el check en rojo igual.**
**Probado dos veces:** reproducido fuera del runner con `bash -e` (exit 1 antes, exit 0 después, con y sin motivo) y en vivo apuntando temporalmente `proyecto-vigilante` a la rama del arreglo — PR de prueba #15, run `36422499482`, `success` sin el exit 1 espurio.
**Estado: ✅ CERRADA DEL TODO.** PR #28 fusionada (`141fed2`), versión **v0.6.0 y luego v0.7.0 cortadas**, y `proyecto-vigilante` **actualizado y confirmado en vivo con el tag real**: `Veredicto: verificado (código 0)`, `success` limpio, sin el `exit code 1` espurio — run del 2026-09-28 con `Uses: .../verificar.yml@refs/tags/v0.7.0`.

**24 · Que el revisor pueda etiquetar `agente:rehacer` (cierra K9) — CÓDIGO FUSIONADO, SIN PROBAR DE EXTREMO A EXTREMO.**
**No bastaba con `issues: write` + pedírselo en el prompt** — habría sido el mismo agujero que K7: el job recibe `GITHUB_TOKEN` por defecto, y GitHub no crea ejecuciones nuevas (el `label_command` de `rehacer.md` sobre `labeled`) a partir de eventos que dispara ese token — confirmado contra la documentación oficial de GitHub (docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow).
**Arreglo:** un paso aparte (nunca el agente) mina un token de la GitHub App ya usada por el ejecutor (`actions/create-github-app-token@v3.2.0`) y lo usa solo para etiquetar. El agente decide leyendo su propio último comentario (línea fija `VEREDICTO-MAQUINA: CAMBIOS_PEDIDOS`/`APROBADO`), sin que se le dé el token — mantiene la separación de permisos del resto del sistema. Sin App configurada, se salta con aviso.
**Probado:** `actionlint` limpio; la lógica de lectura del veredicto, en local con 6 casos — encontró y corrigió un fallo real (detectaba la frase en cualquier parte del comentario, no solo como veredicto).
**NO probado de verdad:** que la etiqueta haga arrancar `rehacer.md`. Un intento de prueba en vivo chocó con que `claude-code-action` se niega a correr si el workflow que lo llama difiere del de `main` (protección oficial contra que una PR robe el token de revisión) — y la forma correcta de probarlo, apuntar `main` del proyecto de prueba directamente, está bloqueada (`git push` a `main` denegado por el clasificador, «Security Weaken»). Y aunque se pudiera, **el proyecto de prueba tampoco tiene la GitHub App configurada** — mismo hueco que la tarea 10.
**Estado: fusionada.** PR #29 fusionada el 2026-09-28 a las 13:51, confirmado leyendo `main`. Ya está en `v0.7.0`, y `proyecto-vigilante` ya lo fija (PR #17, fusionada). Lo que sigue faltando es **solo** la GitHub App — cuando exista, la prueba de extremo a extremo es lo único que queda de esta tarea.

**25 · Fusionar lo que quedaba esperando — ✅ TODO FUSIONADO (2026-09-28).**
`#26` (release 0.6.0), `#27` (manual), `#28` (verificador, tarea 23) y `#29` (K9, tarea 24) — las cuatro fusionadas por el usuario. `release-please` propuso después la `v0.7.0` (PR #30, también fusionada), que ya lleva K9 dentro. Y el pin de `proyecto-vigilante` a `v0.7.0` (PR #17), fusionada y confirmada en vivo.
Confirmado con tres intentos reales denegados durante el trabajo (no solo supuesto): yo no puedo fusionar PRs, ni empujar directo a `main`, ni borrar ramas — las tres acciones las hizo el usuario.

**26 · Limpiar las ramas ya fusionadas.**
De 30 ramas del repo, **26** estaban fusionadas del todo en `main` a fecha del último recuento (2026-09-28, antes de la ronda de fusiones de hoy — el número real hoy es mayor, porque `feat/revisor-etiqueta-rehacer` y `chore/actualizar-v0.7.0` (en `proyecto-vigilante`) se sumaron a la lista de fusionadas y no se ha vuelto a contar). Se intentó borrarlas por la API y **el propio harness lo bloqueó** por ser una acción destructiva hacia fuera. Sigue pendiente de que el usuario las borre o conceda el permiso — **no se ha vuelto a intentar** tras el bloqueo inicial.

## Parqueados y decisiones tuyas — **no se hacen hoy**

**16 · Despliegue (etapa 10)** — aparcado a propósito hasta decidir **dónde** se despliega. *(Esto faltaba en la lista anterior.)*

**17 · Exportar Astillero** — qué mover, cómo y con qué estructura. Aparcado por ti.

**18 · El PDF de [[la-fabrica]]** — aparcado por ti.

**19 · Cuándo arranca el primer piloto ([[patrimonial]])** — decisión tuya. *(Viene de la lista vieja; se confirma o se borra en la tarea 14.)*

**20 · El PAT de `release-please`** — solo el día que se active el ruleset con checks obligatorios (necesita GitHub Pro). **Hoy no hace falta nada.**

---

## Lo que faltaba en la lista anterior

Se me habían quedado fuera **seis**: la **1** (lo empezado y parado), la **8** (medir la intención), la **9**, la **10** y la **11** (tres cosas probadas a medias), la **13** (la CI que nace en rojo) y la **16** (el despliegue, que estaba en los documentos pero no en el plan).

**Actualización 2026-09-28, tras auditar el repo contra GitHub, no contra este documento:** la 11 estaba mal — la medición ya dispara sola, se corrige arriba, en su sitio. La 22 se cerró con una desviación anotada del contrato K7. Nuevas: **23** (bug del verificador, arreglado y sin fusionar), **24** (etiquetar `agente:rehacer`, cierra K9), **25** (tres PRs probadas esperando que las fusiones) y **26** (24 ramas ya fusionadas, pendientes de borrar). Detalle completo de la auditoría en [[astillero-bitacora]].

## Enlaces

- [[astillero-bitacora]] — lo hecho hoy, con su commit y su evidencia
- [[astillero]] — hub del proyecto
- [[la-fabrica]] — el despiece en doce etapas
- [[decisiones]] — registro con fecha
