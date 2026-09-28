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

**22 · El revisor: que publique su veredicto — ✅ HECHO Y PROBADO EN VIVO (2026-09-28), con una desviación anotada.**
Es el agente que lee una PR y **la compara contra el contrato de la tarea**. Lo de aquí arriba (disparo por etiqueta tras la CI, contrato K7) **resultó imposible de construir**: GitHub no crea ejecuciones a partir de eventos que dispara el propio `GITHUB_TOKEN`, y la etiqueta nunca disparaba nada. Se cambió a `pull_request` directo, sin esperar a la CI — **desviación explícita de K7**, no un descuido: el revisor ya puede correr con la CI en rojo.
**Probado en vivo de verdad:** sobre la PR #13 de `proyecto-vigilante`, con veredicto real publicado y una segunda pasada reconociendo qué cambios ya se habían corregido. Arreglado y fusionado directamente en `main` de Astillero (commits `65a8a35` y `282f97c`, 2026-09-28). Detalle completo en [[astillero-bitacora]] y [[decisiones]].
**Lo que le sigue faltando, y es la tarea 9 nueva de abajo:** etiquetar `agente:rehacer` cuando pide cambios — sin eso el bucle de rehacer (K9) sigue sin arrancar nunca.
**Y un pendiente de fontanería:** el arreglo está en `main`, no en un tag. `patrimonial` sigue fijado a `@v0.2.4` (anterior al arreglo) y `proyecto-vigilante` a `@main` (transitorio, no es cómo se fija un proyecto real). Hay que cortar versión y repasar los pines — mismo motivo que la tarea 23 de abajo.

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
**Estado:** PR #28 **fusionada en `main`** el 2026-09-28 (commit `141fed2`), confirmado leyendo `verificar.yml` de `main` en vivo. **Lo que queda:** el arreglo está en `main`, no en un tag — `proyecto-vigilante` sigue fijado a `@v0.4.0`, así que hasta que se corte versión (la PR #26 ya la trae) y se actualice el pin, un proyecto real no lo recibe.

**24 · Que el revisor pueda etiquetar `agente:rehacer` (cierra K9).**
Con el revisor ya publicando veredicto (tarea 22), lo único que falta para que el bucle de rehacer arranque es que, cuando pida cambios, ponga la etiqueta `agente:rehacer` — hace falta darle permiso `issues: write` y añadirlo al prompt/`--allowedTools`. Sin esto, `rehacer.md` sigue en cero corridas pase lo que pase con el resto.

**25 · Fusionar lo que queda esperando.**
**#28 ya fusionada** (2026-09-28, tarea 23). Quedan dos PRs abiertas en `blogNetting/astillero`:
- **#26** — `chore(main): release 0.6.0` (release-please, automática; ya recoge también el arreglo de la 23 y del revisor — cortarla es lo que hace falta para que los proyectos reales reciban ambos).
- **#27** — `docs/manual-completo`, completa el manual y anota las contradicciones con el research.
Fusionarlas es tuyo — tengo prohibido fusionar PRs.

**26 · Limpiar las ramas ya fusionadas.**
De 30 ramas del repo, **26** están fusionadas del todo en `main` y no aportan nada (recomprobado el 2026-09-28 tras fusionar la #28, `ahead_by: 0` contra `main` cada una — subió de 24 a 26 porque `fix/veredicto-exit-code` se sumó a la lista). Se intentó borrarlas por la API y **el propio harness lo bloqueó** por ser una acción destructiva hacia fuera. Pendiente de que tú las borres o concedas el permiso.

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
