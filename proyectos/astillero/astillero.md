---
title: Astillero
created: 2026-09-24
updated: 2026-09-25
tags: [astillero, desarrollo, agentes, opus, deepseek, devsecops, ci-cd, git, producto]
zona: tecnico
---

**Astillero** es el sistema completo, genérico y replicable para desarrollar software de principio a fin con agentes de IA: desde que el usuario tiene una idea, como Product Owner, hasta que el código corre en producción y se mantiene controlado ahí — diseño, especificación, implementación, revisión, CI/CD, DevSecOps, despliegue y operación. Opus 5.5 diseña y revisa; un ejecutor más barato (DeepSeek, por coste, intercambiable) implementa. Verificado en producción real donde se pudo probar en vivo; el resto, diseñado con evidencia real y señalado explícitamente como no ejecutado todavía.

## Estado (2026-09-25): ¿cubre todo el ciclo idea→producción? No lo cubría hasta hoy — ahora sí, con huecos señalados

Pregunta que el usuario hizo explícitamente y que se respondió con una auditoría real, no con una reafirmación: la documentación de ayer llegaba hasta el merge en `main` y ahí se paraba — cero notas sobre despliegue, observabilidad en producción o gestión de incidentes. Confirmado leyendo cada nota, no supuesto. Cerrado hoy con investigación real, misma exigencia de evidencia que el resto del proyecto.

Las cinco piezas, construidas y enlazadas entre sí:

1. **Motor de ingeniería** — [[flujo-agentes-arquitectura]] §1-14. Diseño operable (roles, contratos, máquina de estados, `tdd-guard` como control obligatorio) probado en vivo en un repo real desechable (`blogNetting/prueba-flujo-agentes`): reserva sin colisión, ejecución real de DeepSeek con 2 fallos reales corregidos (falta de permiso `issues: read`, cortafuegos de red sin el ecosistema `python`) y una tercera ejecución con PR real (#13) correctamente creada y marcada para revisión humana por tocar un fichero protegido.
2. **Replicación a proyectos nuevos** — [[astillero-replicacion]]. Repo [blogNetting/astillero](https://github.com/blogNetting/astillero) con los workflows reutilizables (`implementar`/`rehacer`/`revisar`/`reconciliar`) publicados y una plantilla `copier`. Probado de extremo a extremo: un proyecto generado desde cero con `copier` compila contra el import remoto real de este repo.
3. **Capa de producto** — [[capa-producto]]. Cómo el usuario dirige el motor como Product Owner de una sola persona: captura de idea, backlog sin scoring formal (sin evidencia real de que nadie lo use así en solitario), panel en GitHub Projects v2, bugs con el mismo contrato de tarea que una funcionalidad, versionado por checkpoint, registro de decisiones de producto.
4. **Despliegue a producción** — [[flujo-agentes-arquitectura]] §15, nuevo hoy. CD automático en cada merge (sin checkpoint manual aparte del versionado), dónde corre la app, rollback como extensión de §10 (Fallos y recuperación), migraciones de schema siempre en dos tareas — con evidencia real de 40+ operadores en solitario, no manual de empresa grande. **Diseñado, no ejecutado en vivo todavía**: no hay ningún proyecto real desplegado bajo este mecanismo.
5. **DevOps mínimo en producción** — [[devops-minimo]], nuevo hoy, el informe que pediste. Monitorización, alertado, gestión de incidentes (no hace falta on-call formal con un solo operador — hallazgo contraintuitivo con fuente), backup/DR, rotación de secretos, parcheo de dependencias, coste — todo anclado en un caso real auditable (Healthchecks.io, SaaS operado en solitario, stack de producción público). **Diseñado con evidencia real, no ejecutado en vivo**: nada de esto corre todavía sobre una app real de Astillero.

**Lo que sigue sin cerrar, dicho explícito y no escondido:** el sistema de cobertura de tests ya estaba resuelto desde ayer ([[desarrollo-agentes-f4-devsecops]] §3.3, corregido hoy: Vitest con proveedor `v8` nativo en vez de `c8`, Codecov en vez de Coveralls por el plan gratis de repos privados, sin umbral global fijo por ser gameable) pero nunca se ha ejecutado en ninguna de las 3 corridas reales de DeepSeek — sigue siendo diseño verificado, no comportamiento probado. El revisor con Opus tampoco se ha ejecutado todavía (falta tu token). La parametrización real de imports de gh-aw sigue sin resolver (ver «Pendiente de decidir»).

Investigación del ciclo completo (fase previa, cerrada el 2026-09-25): [[desarrollo-agentes-investigacion]]. El informe [[orquestacion-opus-deepseek-informe]] es la primera versión, parcial, superada por [[flujo-agentes-arquitectura]].

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

- Plataforma git por proyecto y nivel de autonomía, es decir, en qué puntos aprueba una persona. Aplazado a propósito.
- Modelo de ramas: está propuesto trunk-based, falta confirmarlo.
- Parametrización real de los imports de gh-aw (`uses:`/`with:`/`import-schema`): intentada y abandonada tras varios errores de compilación reales; hoy el ecosistema de red y demás variables por proyecto se fijan a mano en el wrapper fino, no se pasan como parámetro. Gap conocido, no bloqueante.
- Cuándo arranca el primer piloto ([[patrimonial]]): el sistema ya está completo; falta que tú lo decidas.

## Enlaces

- [[flujo-agentes-arquitectura]] — motor de ingeniería, diseño operable
- [[astillero-replicacion]] — cómo se replica a cada proyecto nuevo
- [[capa-producto]] — cómo se dirige como Product Owner
- [[devops-minimo]] — informe de DevOps mínimo en producción
- [[astillero-mantenimiento]] — cómo se actualizan proyectos ya en marcha y convivencia con 2Cerebro
- [[desarrollo-agentes-investigacion]] — investigación del ciclo completo (síntesis y decisiones)
- [[circuito-tareas-definicion]] — definición de la fase de organización del trabajo con agentes: roles, traspaso, coordinación, topología, revisión, trazabilidad; criterios, método por fases, qué cuenta como contrastado
- [[orquestacion-opus-deepseek-informe]] — informe parcial (superado): conexión entre Claude y DeepSeek
- [[patrimonial]] — candidata a primer piloto
- [[decisiones]] — registro de las decisiones de este proyecto
- [[entorno]] — herramientas de investigación disponibles en esta máquina
- [[_index]]
