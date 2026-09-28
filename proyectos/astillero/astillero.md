---
title: Astillero
created: 2026-09-24
updated: 2026-09-28
tags: [astillero, desarrollo, agentes, opus, deepseek, devsecops, ci-cd, git, producto]
zona: tecnico
---

**Astillero** es el sistema completo, genérico y replicable para desarrollar software de principio a fin con agentes de IA: desde que el usuario tiene una idea, como Product Owner, hasta que el código corre en producción y se mantiene controlado ahí — diseño, especificación, implementación, revisión, CI/CD, DevSecOps, despliegue y operación. Opus 5.5 diseña y revisa; un ejecutor más barato (DeepSeek, por coste, intercambiable) implementa. Verificado en producción real donde se pudo probar en vivo; el resto, diseñado con evidencia real y señalado explícitamente como no ejecutado todavía.

## Estado (2026-09-25): ¿cubre todo el ciclo idea→producción? No lo cubría hasta hoy — ahora sí, con huecos señalados

Pregunta que el usuario hizo explícitamente y que se respondió con una auditoría real, no con una reafirmación: la documentación de ayer llegaba hasta el merge en `main` y ahí se paraba — cero notas sobre despliegue, observabilidad en producción o gestión de incidentes. Confirmado leyendo cada nota, no supuesto. Cerrado hoy con investigación real, misma exigencia de evidencia que el resto del proyecto.

Las cinco piezas, construidas y enlazadas entre sí:

1. **Motor de ingeniería** — [[flujo-agentes-arquitectura]] §1-14. Diseño operable (roles, contratos, máquina de estados, `tdd-guard` como control obligatorio) probado en vivo en un repo real desechable (`blogNetting/prueba-flujo-agentes`): reserva sin colisión, ejecución real de DeepSeek con 2 fallos reales corregidos (falta de permiso `issues: read`, cortafuegos de red sin el ecosistema `python`) y una tercera ejecución con PR real (#13) correctamente creada y marcada para revisión humana por tocar un fichero protegido. **Bootstrap completo desde el 2026-09-26**: `implementar`, `rehacer`, `revisar`, `reconciliar`, `ci`, `deploy` (gate) y `reproducir-bug` (triage) se generan de serie con `copier` — antes solo llegaban 3 de esos 7 a un proyecto nuevo, hueco cerrado con evidencia (`gh aw compile` real en python y node, SHA de cada acción de terceros verificados).
2. **Replicación a proyectos nuevos** — [[astillero-replicacion]]. Repo [blogNetting/astillero](https://github.com/blogNetting/astillero) con los workflows reutilizables (`implementar`/`rehacer`/`revisar`/`reconciliar`) publicados y una plantilla `copier`. Probado de extremo a extremo: un proyecto generado desde cero con `copier` compila contra el import remoto real de este repo.
3. **Capa de producto** — [[capa-producto]]. Cómo el usuario dirige el motor como Product Owner de una sola persona: captura de idea, backlog sin scoring formal (sin evidencia real de que nadie lo use así en solitario), panel en GitHub Projects v2, bugs con el mismo contrato de tarea que una funcionalidad, versionado por checkpoint, registro de decisiones de producto.
4. **Despliegue a producción** — [[flujo-agentes-arquitectura]] §15, nuevo hoy. CD automático en cada merge (sin checkpoint manual aparte del versionado), dónde corre la app, rollback como extensión de §10 (Fallos y recuperación), migraciones de schema siempre en dos tareas — con evidencia real de 40+ operadores en solitario, no manual de empresa grande. **Diseñado, no ejecutado en vivo todavía**: no hay ningún proyecto real desplegado bajo este mecanismo.
5. **DevOps mínimo en producción** — [[devops-minimo]], nuevo hoy, el informe que pediste. Monitorización, alertado, gestión de incidentes (no hace falta on-call formal con un solo operador — hallazgo contraintuitivo con fuente), backup/DR, rotación de secretos, parcheo de dependencias, coste — todo anclado en un caso real auditable (Healthchecks.io, SaaS operado en solitario, stack de producción público). **Diseñado con evidencia real, no ejecutado en vivo**: nada de esto corre todavía sobre una app real de Astillero.

**Despiece por etapas y estado de madurez:** [[la-fabrica]] — las doce etapas del sistema, quién actúa en cada una, y la separación entre lo que ya puede operar solo y lo que no. Los dos agujeros identificados aquel día: **la verificación** (faltaba el oráculo que el agente no pueda tocar) y **la medición** (no existía). **Los dos CERRADOS el 2026-09-27** — construidos, probados en vivo y publicados en el tag `v0.4.0`. Ver [[astillero-bitacora]].

**Lo que sigue sin cerrar, dicho explícito y no escondido:** el sistema de cobertura de tests ya estaba resuelto desde ayer ([[desarrollo-agentes-f4-devsecops]] §3.3, corregido hoy: Vitest con proveedor `v8` nativo en vez de `c8`, Codecov en vez de Coveralls por el plan gratis de repos privados, sin umbral global fijo por ser gameable) pero nunca se ha ejecutado en ninguna de las 3 corridas reales de DeepSeek — sigue siendo diseño verificado, no comportamiento probado. Y no hay gate real: no existe fichero de configuración de Codecov en el repo, así que nadie comprueba que lo nuevo esté cubierto.

**Corrección (2026-09-28):** esta nota decía que «el revisor con Opus tampoco se ha ejecutado todavía». **Ya no es así** — corrió de verdad el 2026-09-28, con veredicto real publicado sobre una PR de prueba. Detalle en [[astillero-bitacora]]. La frase de la parametrización de imports de gh-aw ya estaba desactualizada desde el 2026-09-27 (se cerró ese día, ver «Pendiente de decidir» abajo). Y nuevo hoy: un bug del verificador que sacaba el check en rojo con veredicto correcto — arreglado, probado en vivo y **fusionado en `main`** (PR #28). Falta cortar versión para que llegue a un proyecto real.

Investigación del ciclo completo (fase previa, cerrada el 2026-09-25): [[desarrollo-agentes-investigacion]]. El informe [[orquestacion-opus-deepseek-informe]] es la primera versión, parcial, superada por [[flujo-agentes-arquitectura]].

**Estado del trabajo, al día:** [[astillero-bitacora]] — qué está hecho y comprobado (con su commit), qué está en curso y qué falta. Si una sesión se corta, eso es lo que sobrevive.

**Cómo funciona, explicado entero:** [`docs/manual.md`](https://github.com/blogNetting/astillero/blob/main/docs/manual.md) — cada pieza construida por dentro, los contratos entre ellas, quién puede escribir qué y los modos de fallo.

**Plan de trabajo, al día:** [[astillero-plan]] — lo que queda, en orden y con su porqué.

**El protocolo, obligatorio en cada pieza:** **implementar → probar en vivo (por el camino que usa un proyecto de verdad) → redactar la wiki explicando cómo funciona → actualizar la bitácora.** Documentar no es listar cambios: es explicar el funcionamiento para que nadie tenga que reconstruirlo. El detalle está en [[astillero-bitacora]].

## Repo: Astillero

- **Repo:** [blogNetting/astillero](https://github.com/blogNetting/astillero), privado.
- **Ruta local:** `~/dev/astillero`.
- **Estado (2026-09-25):** creado, con acceso de Actions abierto a los repos de `blogNetting` (para los reusable workflows de [[astillero-replicacion]]). Contiene ya los workflows reutilizables (`implementar`/`rehacer`, con `.lock.yml` compilado; `revisar`/`reconciliar`), la plantilla `copier` y un `project-example/` generado como prueba — verificado en vivo contra el repo real el 2026-09-25.

## Hipótesis iniciales del usuario (no son decisiones)

Se ponen en duda como cualquier otra alternativa: la inteligencia del ciclo la pone Anthropic; un modelo más barato (DeepSeek) implementa; el trabajo se entrega en forma de issues; gitflow. Se mantienen o se descartan según la evidencia de la investigación completa.

## Decisiones cerradas

- **Ejecutor abstracto (2026-09-25).** La arquitectura y la infraestructura funcionan igual sea quien sea el que implementa (DeepSeek, Sonnet u otro). Se usará DeepSeek **por coste**, no por calidad. Cómo se invoca a cada ejecutor es un problema aparte. El diseño y el desglose en tareas los hace Opus 5.5; el resultado siempre se revisa.
- **Forja: GitHub (2026-09-25).**
- **Framework spec-driven: se elige en cada proyecto**, según encaje.
- **No hay piloto** hasta que el sistema esté completo. [[patrimonial]] no arranca antes.

- **Genérico y abstracto.** Tiene que servir para cualquier desarrollo. Stack, plataforma git y despliegue se deciden en cada proyecto. Preferencias del usuario: Python con algún framework, o Node.js.
- **Repos propios.** Este sistema y cada app que cuelgue de él tienen su propio repo, fuera de `2cerebro`, que es público y hace push cada hora. Desde el wiki se llega a cada uno con una nota hub (enlace al repo, ruta local y estado).
- **No hay código sin tests**, sean unitarios o de integración. Cobertura y complejidad: **cerrado el 2026-09-25**, ver [[desarrollo-agentes-f4-devsecops]] §3.3 y [[flujo-agentes-arquitectura]] §8 — sin umbral global fijo (gameable), `patch coverage` cerca del 100% en líneas nuevas + mutation testing del diff.
- **Si se usa DeepSeek, su API oficial es aceptable.** Que los datos estén en China no es un problema para el usuario.
- **Claude va por suscripción Pro (20 €/mes), no por API.** Es una restricción de diseño: hay límites de uso, sobre todo con Opus.
- **Control de coste por saldo prepagado.** El usuario va recargando y decide la viabilidad según el consumo. Hay que poder medir el gasto por modelo y por tarea.

## Bloques (histórico, 2026-09-24 — superado por «Estado»)

> El plan original de 8 bloques, abierto el primer día. Todos resueltos hoy y reflejados en «Estado» arriba; se conserva solo como registro de por dónde empezó el proyecto, no como pendiente activo.

1. Proceso → [[flujo-agentes-arquitectura]]. 2. Orquestador/ejecutor Opus+DeepSeek → resuelto, decisión cerrada arriba. 3. Git → trunk-based, rulesets ([[desarrollo-agentes-f3-git-cicd-infra]]). 4. CI/CD y entornos → §8, §15 de [[flujo-agentes-arquitectura]]. 5. DevSecOps → [[desarrollo-agentes-f4-devsecops]] + [[devops-minimo]]. 6. Integración con 2Cerebro → esta misma nota hub. 7. Economía → coste medido en vivo con DeepSeek, ver «Control de coste» abajo. 8. Piloto → [[patrimonial]], sigue sin arrancar por decisión propia, no por pieza faltante.

## Motor de ingeniería (2026-09-25): construido y probado en vivo

[[flujo-agentes-informe]] (evidencia), [[flujo-agentes-arquitectura]] (diseño operable, con `tdd-guard` como control obligatorio y, desde hoy, la extensión del panel de producto en §14), [[flujo-agentes-runbook]] (puesta en marcha), [[flujo-agentes-evidencia-empirica]] (¿va DeepSeek a implementar bien? — sin garantía, evidencia y mitigación), [[flujo-agentes-prueba-descomposicion]] (Opus real descomponiendo una idea) y el diagrama del mecanismo: https://claude.ai/artifact/LmnofQRqGPyXDdZa6aCihL. Definición de la fase: [[circuito-tareas-definicion]].

**Probado en vivo, de verdad, en `blogNetting/prueba-flujo-agentes`:** reserva sin colisión (5→1 disparo real); tres ejecuciones reales de DeepSeek — la 1ª falló por falta del permiso `issues: read` (diagnóstico honesto del propio agente, sin fabricar implementación), la 2ª pasó tests pero sin acceso de red a PyPI (cortafuegos sin el identificador de ecosistema `python`), la 3ª completó con tests y `mypy` en verde y un PR real (#13) creado y correctamente marcado para revisión humana por tocar un fichero protegido. Ambos fallos, corregidos y generalizados en la documentación (no solo para pip, para cualquier ecosistema).

**Pendiente de tu parte, no mía:** aprobar o rechazar el PR #13 — el diseño exige revisión humana ahí, no la puedo saltar yo.

### Preguntas anteriores (anuladas, historial)

> **Anuladas (2026-09-24).** Estas preguntas daban por buena la hipótesis del usuario (Opus + DeepSeek con issues) y salían de un informe parcial. Las sustituirán las preguntas de la investigación completa del ciclo, que está en curso.

Planteadas el 2026-09-24 tras la investigación del bloque 2. Contexto en [[orquestacion-opus-deepseek-informe]].

1. **¿Validas el esqueleto de arquitectura?** Opus orquesta en su sesión de suscripción, escribe una especificación tipada con criterios de aceptación y lanza el ejecutor DeepSeek como proceso aparte, en un worktree y un sandbox. Después se pasan los gates deterministas (tests vistos en rojo, cobertura, mutation testing, SAST, SCA y secretos), Opus revisa ejecutando, y se sigue con PR, CI y merge queue.
2. **¿Pasan a pruebas los tres ejecutores finalistas?** Son Claude Code apuntado a DeepSeek, OpenCode (`opencode run`) y Pi (modos print, JSON y RPC), con Aider en reserva. Se probarían con `deepseek-v4-pro` y `deepseek-flash`, y con dos líneas base: Opus solo y DeepSeek solo.
3. **¿Montamos el sandbox (Podman rootless con gVisor y proxy de salida) desde el principio de las pruebas?** Requiere sudo, así que lo instala el usuario.

Después de esas respuestas:

- **API key de DeepSeek**: en un fichero fuera del repo (p. ej. `~/.config/deepseek/key`, permisos 600), sin pegarla en el chat. Hay que comprobar el medio de pago: un comentario de HN, sin verificar, dice que solo admite Alipay o WeChat.
- **Diseñar el conjunto de tareas de prueba**, con tests ocultos y métricas: tasa de éxito, $ por tarea, `/usage` de Opus, tiempo, intervenciones y si el ejecutor ha tocado tests.

## Pendiente de decidir

> **Ojo con esta lista: mezcla dos épocas.** Los cuatro primeros puntos son de **antes del research** (2026-09-24/25) y **no se han revisado desde entonces** — varios pueden estar ya cerrados o haber cambiado de forma. Los de **2026-09-27** son de hoy. Antes de dar cualquiera por abierto, comprobar. Esta mezcla sin fechar es lo que hizo que se volviera a plantear como pendiente algo ya tratado.

- ~~Plataforma git por proyecto y nivel de autonomía~~ — **cerrado (2026-09-27).** La plataforma es **GitHub**, y no por inercia: el research la mide (81,1 % de desarrolladores profesionales, 67,8 % de cuota — [[desarrollo-agentes-f3-git-cicd-infra]] §68). Y los puntos donde aprueba una persona **están definidos e implementados**: las rutas sensibles por `CODEOWNERS`, y cuando la puerta para una tarea tras dos fallos.
- ~~Modelo de ramas: trunk-based~~ — **cerrado (2026-09-27).** Decidido por el research: *«trunk-based development (rama corta por issue/PR de agente, mergeada a `main` en <1 día, protegida por CI) es el estándar»* ([[desarrollo-agentes-f3-git-cicd-infra]] §46). Y es lo que el repo hace.
- ~~Parametrización de los imports de gh-aw~~ — **cerrado (2026-09-27): ya está resuelto.** El molde **sí** los parametriza: `imports: .../implementar-core.md@{{ astillero_ref }}` y `network.allowed: [github, api.deepseek.com, {{ package_ecosystem }}]` en `template/.github/workflows/implementar.md.jinja`. El pendiente describía un estado anterior.
- **Pruebas propias de Astillero, con cobertura.** Astillero no tiene tests que comprueben lo que él mismo hace. Hay que crearlos, de forma que al llevar Astillero a otro sitio se puedan **lanzar allí todos los tests y verificar que funciona también allí**, y medir cobertura para comprobar que los escenarios están cubiertos. Nace de un fallo real, no de una precaución: el verificador llevaba desde su primer commit sin poder ejecutar la suite (no instalaba las dependencias del proyecto), y **ninguna prueba lo detectó** — apareció el 2026-09-27 al ejecutarlo en vivo sobre un proyecto generado con `copier`. Detalle en [[decisiones]].
- **Exportar Astillero para crear proyectos.** Qué habría que mover, cómo moverlo y con qué estructura de directorio (incluido sacarlo de la cuenta `blogNetting`). Instrucción tuya: se estudia **cuando todo lo demás esté cerrado y atado**, y entonces te lo pregunto yo. Ver [[decisiones]].
- **El PDF de [[la-fabrica]].** Pedido por ti el 2026-09-27 («recuerda el PDF que te he pedido, cuando lo tengas avísame») y **sin anotar hasta ahora** — se perdía en cuanto cerrara la sesión. Quedó pendiente de los dos barridos que faltaban (verificación práctica y medición). Se avisa cuando esté.
- ~~Las 7 revisiones de Gas City~~ — **cerrado.** El usuario confirma (2026-09-27) que **ya se trató**: no es un pendiente abierto. Lo único vivo de ahí es el código `75`, ya aplicado y dentro de `v0.4.0`. **No volver a plantearlo como pregunta.**
- ~~Documentación interna del repo~~ — **cerrado (2026-09-27).** Los cuatro enlaces rotos resueltos, las afirmaciones falsas corregidas y el manual técnico escrito (`docs/manual.md`, PR #17). El checklist se comprobó contra las corridas reales (PR #18, #20): marcaba como probadas tres piezas que no lo están.

- Cuándo arranca el primer piloto ([[patrimonial]]): el sistema ya está completo; falta que tú lo decidas.

## Enlaces

- [[flujo-agentes-arquitectura]] — motor de ingeniería, diseño operable
- [[astillero-replicacion]] — cómo se replica a cada proyecto nuevo
- [[capa-producto]] — cómo se dirige como Product Owner
- [[devops-minimo]] — informe de DevOps mínimo en producción
- [[astillero-mantenimiento]] — cómo se actualizan proyectos ya en marcha y convivencia con 2Cerebro
- [[flujo-agentes-runbook]] — incluye el skill `/astillero-proyecto`, el punto de entrada real
- [[desarrollo-agentes-investigacion]] — investigación del ciclo completo (síntesis y decisiones)
- [[circuito-tareas-definicion]] — definición de la fase de organización del trabajo con agentes: roles, traspaso, coordinación, topología, revisión, trazabilidad; criterios, método por fases, qué cuenta como contrastado
- [[orquestacion-opus-deepseek-informe]] — informe parcial (superado): conexión entre Claude y DeepSeek
- [[patrimonial]] — candidata a primer piloto
- [[decisiones]] — registro de las decisiones de este proyecto
- [[entorno]] — herramientas de investigación disponibles en esta máquina
- [[verificacion-externa-agentes]] — síntesis del principio de verificación externa: los dos agujeros del sistema
- [[contradiccion-agents-md]] — discrepancia sin resolver sobre el efecto medido de `AGENTS.md`/`CLAUDE.md`
- [[_index]]
