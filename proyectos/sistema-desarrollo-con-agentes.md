---
title: Sistema de desarrollo de aplicaciones con agentes
created: 2026-09-24
updated: 2026-09-24
tags: [desarrollo, agentes, opus, deepseek, devsecops, ci-cd, git]
zona: tecnico
---

Sistema genérico para desarrollar cualquier aplicación de principio a fin: Opus dirige y DeepSeek escribe el código, sobre git, CI/CD y DevSecOps.

## Estado

Abierto el 2026-09-24. Fase actual: investigar cómo implementar el reparto Opus/DeepSeek (bloque 2). Nada de arquitectura decidido todavía.

## Decisiones cerradas

- **Genérico y abstracto.** Tiene que servir para cualquier desarrollo. Stack, plataforma git y despliegue se deciden en cada proyecto. Preferencias del usuario: Python con algún framework, o Node.js.
- **Repos propios.** Este sistema y cada app que cuelgue de él tienen su propio repo, fuera de `2cerebro`, que es público y hace push cada hora. Desde el wiki se llega a cada uno con una nota hub (enlace al repo, ruta local y estado).
- **No hay código sin tests**, sean unitarios o de integración. La exigencia de cobertura y complejidad se fijará más adelante.
- **DeepSeek por su API oficial es aceptable.** Que los datos estén en China no es un problema para el usuario.
- **Opus va por suscripción Claude Pro (20 €/mes), no por API.** Es una restricción de diseño: el orquestador tiene límites de uso, así que la arquitectura debe gastar poco Opus por tarea.
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

## Pendiente de decidir

- Plataforma git por proyecto y nivel de autonomía, es decir, en qué puntos aprueba una persona. Aplazado a propósito.
- Modelo de ramas: está propuesto trunk-based, falta confirmarlo.

## Enlaces

- [[app-seguimiento-patrimonio]] — candidata a primer piloto
- [[decisiones]] — registro de las decisiones de este proyecto
- [[entorno]] — herramientas de investigación disponibles en esta máquina
- [[_index]]
