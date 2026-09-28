---
title: Bitácora de Astillero — movida al repo
created: 2026-09-27
updated: 2026-09-28
tags: [astillero, estado, bitacora, trabajo]
zona: tecnico
---

**Movida al repo el 2026-09-28**, a petición del usuario: https://github.com/blogNetting/astillero/blob/main/docs/bitacora.md

## Por qué se movió

Se construyó un mecanismo de CI (`documentar.yml`) que obliga a cualquier modelo o persona que toque código de Astillero a actualizar también su documentación — la misma política que ya se probó como hook de Claude Code, pero aplicada a cualquiera, no solo a quien use ese editor. Un CI del repo de Astillero no puede ver ni exigir nada sobre una nota de este wiki, que vive en un repo aparte (`2cerebro`). Por eso la bitácora tenía que vivir dentro del propio repo para que ese mecanismo la pudiera alcanzar.

No es la migración completa de "exportar Astillero" (que sigue aparcada, sin fecha) — es solo esta pieza, la mínima necesaria para que el gate funcione.

## Dónde está ahora

- **Bitácora:** [`docs/bitacora.md`](https://github.com/blogNetting/astillero/blob/main/docs/bitacora.md)
- **Plan:** `astillero-plan` — también movido, ver su propia nota
- **Cómo funciona el mecanismo que las exige:** `docs/manual.md` §11c, en el mismo repo

## Lo que sigue en el wiki

La investigación, el diseño y las decisiones de arquitectura de Astillero — `astillero`, `la-fabrica`, `flujo-agentes-arquitectura`, `decisiones` y el resto de notas de `proyectos/astillero/` — siguen aquí. Solo se movió el seguimiento del trabajo (qué se hizo, qué falta), porque es lo único que el mecanismo de CI necesitaba exigir.

## Enlaces

- [[astillero]] — hub del proyecto
- [[astillero-plan]] — plan de trabajo, movido el mismo día
- [[decisiones]] — registro de decisiones, incluida la de este movimiento
