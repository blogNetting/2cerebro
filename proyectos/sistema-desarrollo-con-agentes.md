---
title: Sistema de desarrollo de aplicaciones con agentes
created: 2026-09-24
updated: 2026-09-25
tags: [desarrollo, agentes, opus, deepseek, devsecops, ci-cd, git]
zona: tecnico
---

Sistema genérico para desarrollar cualquier aplicación de principio a fin, con IA, git, CI/CD y DevSecOps. Cómo se reparte el trabajo entre modelos se decide con evidencia.

## Estado

Abierto el 2026-09-24. Investigación del ciclo completo cerrada el 2026-09-25: [[desarrollo-agentes-investigacion]]. Faltan las decisiones del usuario (ver abajo). El informe [[orquestacion-opus-deepseek-informe]] es parcial y solo cubre la conexión entre modelos.

## Hipótesis iniciales del usuario (no son decisiones)

Se ponen en duda como cualquier otra alternativa: la inteligencia del ciclo la pone Anthropic; un modelo más barato (DeepSeek) implementa; el trabajo se entrega en forma de issues; gitflow. Se mantienen o se descartan según la evidencia de la investigación completa.

## Decisiones cerradas

- **Genérico y abstracto.** Tiene que servir para cualquier desarrollo. Stack, plataforma git y despliegue se deciden en cada proyecto. Preferencias del usuario: Python con algún framework, o Node.js.
- **Repos propios.** Este sistema y cada app que cuelgue de él tienen su propio repo, fuera de `2cerebro`, que es público y hace push cada hora. Desde el wiki se llega a cada uno con una nota hub (enlace al repo, ruta local y estado).
- **No hay código sin tests**, sean unitarios o de integración. La exigencia de cobertura y complejidad se fijará más adelante.
- **Si se usa DeepSeek, su API oficial es aceptable.** Que los datos estén en China no es un problema para el usuario.
- **Claude va por suscripción Pro (20 €/mes), no por API.** Es una restricción de diseño: hay límites de uso, sobre todo con Opus.
- **Control de coste por saldo prepagado.** El usuario va recargando y decide la viabilidad según el consumo. Hay que poder medir el gasto por modelo y por tarea.

## Bloques

1. Proceso: fases, qué documento sale de cada una y dónde aprueba una persona. Los criterios de aceptación son el contrato que recibe el ejecutor.
2. Orquestador y ejecutor, Opus y DeepSeek: se decide por investigación y pruebas, no por opinión. Pesa lo que funciona, lo que la comunidad ha probado y está contrastado, por encima de la novedad o el hype. Steve Yegge y Andrej Karpathy son pistas a las que recurrir si falta por dónde buscar, no frentes obligatorios.
3. Git: ramas protegidas y un procedimiento para que no se pise el código. Puede que la herramienta que se elija ya resuelva esto.
4. CI/CD y entornos.
5. DevSecOps: controles del pipeline y seguridad del propio sistema de agentes.
6. Integración con 2Cerebro: notas hub y skills.
7. Economía: coste real, incluido el retrabajo y las revisiones de Opus.
8. Piloto: seguramente [[app-seguimiento-patrimonio]], sin confirmar. Solo empieza cuando este sistema esté listo.

## Preguntas abiertas para el usuario (siguiente sesión)

Vigentes desde el 2026-09-25. Salen de [[desarrollo-agentes-investigacion]], sección 7.

1. ¿Un solo vendor por defecto (Anthropic con `opusplan`) y DeepSeek como experimento medido, en lugar de la división fija Anthropic piensa / DeepSeek implementa?
2. Forja: GitHub (mejor ecosistema de agentes) o Gitea/Forgejo autoalojado (soberanía).
3. Framework spec-driven para el piloto: Spec Kit u OpenSpec.
4. Qué rutas exigen siempre aprobación humana (auth, cripto, IaC, workflows, tests).
5. Piloto: ¿[[app-seguimiento-patrimonio]]?

### Preguntas anteriores (anuladas)

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

## Enlaces

- [[desarrollo-agentes-investigacion]] — investigación del ciclo completo (síntesis y decisiones)
- [[orquestacion-opus-deepseek-informe]] — informe parcial: conexión entre Claude y DeepSeek
- [[app-seguimiento-patrimonio]] — candidata a primer piloto
- [[decisiones]] — registro de las decisiones de este proyecto
- [[entorno]] — herramientas de investigación disponibles en esta máquina
- [[_index]]
