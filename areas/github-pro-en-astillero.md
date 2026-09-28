---
title: GitHub Pro en Astillero — qué habilita y qué piezas dependen de él
created: 2026-09-28
updated: 2026-09-28
tags: [astillero, github, meta]
zona: tecnico
---

Tema recurrente en las notas de Astillero: qué se pierde en un repo privado sin GitHub Pro y qué contratos del sistema dependen de esa puerta. No es una decisión de compra — nadie lo ha pedido — sino un hallazgo técnico condicional.

## Qué no existe sin Pro (repo privado)

Comprobado el 2026-09-25 contra la API real: en un repo privado del plan Free, tanto `POST /rulesets` como la protección de rama clásica devuelven `403 — "Upgrade to GitHub Pro or make this repository public to enable this feature"`. Sin Pro (o sin hacer el repo público), no hay rulesets, ni protección de rama, ni merge queue. Cita textual en [[flujo-agentes-runbook]] §2.

## Qué piezas de Astillero dependen de esa puerta

- **K6** (PR → CI, checks obligatorios) y **K10** (PR aprobada → integración, merge queue): los dos contratos constan como **no construidos** por este motivo, confirmado con la API real el 2026-09-28. Detalle del cotejo en [[astillero-bitacora]].
- **Etapa 9 de la fábrica** (aprobación y merge): «no bloquea de verdad: sin ruleset … ningún check es obligatorio, así que se puede fusionar en rojo». [[la-fabrica]].
- `release-please` con checks obligatorios: el día que se active el ruleset, la PR de release necesitaría un PAT (`ASTILLERO_RELEASE_TOKEN`) en vez del `GITHUB_TOKEN` por defecto, que no dispara los checks. [[flujo-agentes-runbook]] §2, [[astillero-mantenimiento]].

## Qué NO cambia

El sistema sigue funcionando sin rulesets ni merge queue; lo que falta es esa puerta extra, no una pieza del flujo. Ni se ha contratado ni se ha decidido hacerlo: la corrección del 2026-09-26 quitó del wiki la palabra «pendiente» y cualquier recomendación implícita de pagarlo — es un dato técnico neutro ([[decisiones]], entrada del 2026-09-26). La propagación entre repos tampoco lo necesita: se resuelve con reusable workflows e imports de gh-aw, y la regla "required workflows" de organización se descartó por exigir GitHub Team ([[astillero-replicacion]] §2).

## Precio

La cifra que dan el wiki (~4 $/mes en [[decisiones]]; 4 $/mes en [[flujo-agentes-runbook]] §2) **no está verificada en fuente primaria** — contradicción abierta en [[contradiccion-precio-github-pro]], sin cifra elegida.

## Enlaces

- [[flujo-agentes-runbook]]
- [[astillero-replicacion]]
- [[la-fabrica]]
- [[astillero-bitacora]]
- [[flujo-agentes-arquitectura]]
- [[astillero-mantenimiento]]
- [[decisiones]]
- [[contradiccion-precio-github-pro]]
