---
title: Astillero — cómo se replica el sistema a cada proyecto
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, github-actions, gh-aw, reusable-workflows, copier, replicacion]
zona: tecnico
---

Cómo se monta un repo "plataforma" (**Astillero**, nombre elegido por el usuario el 2026-09-25) con los workflows y la configuración reutilizables del flujo de [[flujo-agentes-arquitectura]], de forma que (1) un proyecto nuevo (p. ej. [[patrimonial]]) se crea a partir de él ya configurado, y (2) cuando Astillero cambia después, ese cambio llega también a los proyectos que ya existen, no solo a los nuevos.

## 1. Introducción

**Pregunta.** Qué mecanismo usar para que la lógica y la configuración de [[flujo-agentes-arquitectura]] no se copien y pegen a mano en cada repo de proyecto, y para que un cambio posterior en Astillero se propague a los proyectos ya creados. Verificado en fuente primaria, no propuesto a ojo (corrige la propuesta sin verificar de la conversación previa, sobre todo el punto de si gh-aw admite includes remotos).

## 2. Considerado y descartado

| Opción | Motivo del descarte | Evidencia |
|---|---|---|
| Copiar los ficheros a mano en cada repo nuevo, sin mecanismo de propagación | Es justo el problema que plantea esta nota: un cambio en Astillero no llegaría nunca a los proyectos ya creados | — |
| `cruft` para sincronizar plantilla | Sin push desde 2024-12-25 (**~21 meses** parado a fecha de esta nota) | `gh api repos/cruft/cruft` · alta |
| `BetaHuhn/repo-file-sync-action` | Sin push desde 2024-08-05 (**~2 años** parado), 366★, muy por debajo de las alternativas activas | `gh api repos/BetaHuhn/repo-file-sync-action` · alta |
| Rulesets de organización con la regla "required workflows" (fuerza que un repo ejecute cierto workflow) | Es GA, pero exige **GitHub Team de pago a nivel de organización**; `blogNetting` es cuenta de usuario personal, no organización (`gh api users/blogNetting` → `"type": "User"`). No hace falta de todas formas: los reusable workflows y los imports de gh-aw (§3) ya resuelven la propagación sin esto | [GitHub Changelog, jun-2025](https://github.blog/changelog/2025-06-16-organization-rulesets-now-available-for-github-team-plans/) · alta |
| `git subtree` / `git submodule` para incluir los workflows compartidos | No se llegó a evaluar en profundidad: los dos mecanismos nativos de GitHub (§3) cubren la parte que más cambia (la lógica) sin necesitar git de por medio, y para lo estático `copier` resuelve mejor el caso de "cada proyecto ha podido divergir un poco" gracias al merge a 3 bandas | Descartado por cobertura suficiente de las otras opciones, no por evidencia en contra directa |

## 3. Análisis: dos mecanismos según el tipo de pieza

| Tipo de pieza | Mecanismo | Qué hace falta una vez | Qué pasa cuando Astillero cambia |
|---|---|---|---|
| Lógica de ejecución (`revisar.yml`, `reconciliar.yml`) | **Reusable workflows** de GitHub Actions (`workflow_call`) | En Astillero: `Settings → Actions → General → Access → "Accessible from repositories owned by 'blogNetting' user"`. Cada proyecto tiene un fichero de 2-3 líneas: `uses: blogNetting/astillero/.github/workflows/revisar.yml@main` | **Nada.** El siguiente run de cada proyecto ya usa la versión nueva |
| Lógica del ejecutor (`implementar.md`, `rehacer.md`, gh-aw) | **Imports remotos de gh-aw** | Cada proyecto tiene un `.md` fino con `imports: [blogNetting/astillero/shared/implementar-core.md@v1.2.0]` | Se sube una versión (tag) nueva en Astillero; cada proyecto actualiza el `@ref` cuando quiere adoptarla — control explícito, no automático |
| Configuración estática (`CODEOWNERS` base, esqueleto de `AGENTS.md`, labels, plantilla de contrato de tarea) | **Copier** (`copier update`) | Astillero es la plantilla de copier; cada proyecto nace con `copier copy` y guarda `.copier-answers.yml` | Workflow programado en cada proyecto que corre `copier update` y **abre una PR** — no es Astillero empujando, es cada proyecto tirando y una persona aprobando |

### 3.1 Reusable workflows cross-repo, verificado

Confirmado en la documentación oficial que un repo privado puede llamar a un reusable workflow de **otro** repo privado de la **misma cuenta personal** (no hace falta organización): *"To grant access to other private repositories, in the Access section at the bottom of the page, select Accessible from repositories owned by 'USERNAME' user."* — [docs.github.com](https://docs.github.com/en/actions/creating-actions/sharing-actions-and-workflows-from-your-private-repository) · alta (doc oficial, cita literal). `secrets: inherit` funciona entre repos de la misma cuenta bajo esa configuración — [docs.github.com/reusing-workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows) · alta. Límite: máximo 10 niveles de anidamiento de reusable workflows, mismo doc.

### 3.2 gh-aw admite imports remotos — cierra el punto que había quedado sin verificar en [[flujo-agentes-arquitectura]] §7.1

*"Paths matching `owner/repo/path@ref` are fetched from GitHub at compile time."* Sintaxis: `imports: [acme-org/shared-workflows/shared/reporting.md@v2.1.0]`, acepta tag, rama o SHA. Se cachean por SHA de commit en `.github/aw/imports/` para compilación sin conexión; necesita `GITHUB_TOKEN` con acceso al repo remoto; solo un fichero de agente por import. — [github.github.com/gh-aw/reference/imports](https://github.github.com/gh-aw/reference/imports/) · alta (doc oficial de la pieza, cita literal). `gh-aw` en sí: activo de verdad, 5.181★, último push el mismo día de esta nota — `gh api repos/githubnext/gh-aw`.

### 3.3 Copier para lo estático

Comunidad y actividad, verificado con `gh api` a fecha de esta nota:

| Herramienta | ★ | Último push | Veredicto |
|---|---|---|---|
| **copier** | 3.599 | 2026-09-18 (7 días) | Activo. Update con **merge a 3 bandas** vía `.copier-answers.yml`: respeta lo que cada proyecto ya haya tocado, no lo pisa a lo bruto |
| cruft | 1.587 | 2024-12-25 (~21 meses) | Descartado por falta de mantenimiento |
| repo-file-sync-action | 366 | 2024-08-05 (~2 años) | Descartado por falta de mantenimiento y baja adopción |

Validación independiente: *"Copier wins for fleet management due to native `copier update` 3-way merge... the update feature is not a nice extra but becomes core infrastructure"* cuando se gestionan varios repos desde una plantilla común — [copier.readthedocs.io/comparisons](https://copier.readthedocs.io/en/stable/comparisons/) · media-alta (documentación del propio proyecto, pero coincide con la comparativa independiente encontrada en la búsqueda).

**Patrón de automatización real, no solo documentado:** el uso extendido no es que Astillero empuje cambios, sino que **cada proyecto** lleva un workflow programado (cron semanal + `workflow_dispatch`) que corre `copier update` y abre una PR para revisión — ejemplo real en producción: [riscv/docs-spec-template#156](https://github.com/riscv/docs-spec-template/pull/156), acción reutilizable equivalente en [fohte/copier-update-action](https://github.com/fohte/copier-update-action) · media (ejemplos de uso real, no doc oficial, pero coincide el patrón en varias fuentes independientes). Encaja con "quien implementa no aprueba" ya establecido en el diseño: el cambio de plantilla llega como PR, no se aplica solo.

## 4. Recomendaciones

1. **Astillero es un repo real**, no solo el documento del wiki. Contiene: `.github/workflows/revisar.yml`, `reconciliar.yml` y `reproducir.yml` (reusable), `shared/implementar-core.md` y `rehacer-core.md` (importables por gh-aw), y la plantilla de copier (`CODEOWNERS` con usuario real, `AGENTS.md`, `docs/contrato-tarea.md`, `{{_copier_conf.answers_file}}.jinja` — genera `.copier-answers.yml`, sin él no hay `copier update` posible, bug real encontrado y corregido el 2026-09-26 — y en `.github/workflows/`: `implementar`, `rehacer`, `revisar`, `reconciliar`, `ci`, `deploy`, `reproducir-bug`).
2. **Un solo ajuste manual, una vez:** activar "Accessible from repositories owned by 'blogNetting' user" en Astillero.
3. **Cada proyecto nuevo** (p. ej. Patrimonial) se crea con `copier copy gh:blogNetting/astillero .`, con wrappers finos que apuntan a los reusable workflows y a los imports de gh-aw fijados por versión, más un workflow propio de `copier update` para recibir actualizaciones futuras de lo estático.
4. **Versionado de Astillero:** tags semánticos (`v1.2.0`), porque tanto los imports de gh-aw como copier referencian versiones explícitas — un cambio no se propaga solo a lo que depende de una versión fijada; cada proyecto decide cuándo sube el `@ref`. Los reusable workflows si son la excepción: si se referencian por `@main` en vez de por tag, sí se propagan sin que nadie tenga que actuar — a decidir por pieza según se quiera control o automatismo.

## 5. Dónde se ha buscado

- **Resuelto en el paso 1 (WebSearch) o 2 (WebFetch), sin necesitar `navegador-cdp`:** todas las páginas de esta nota son documentación oficial (docs.github.com, github.github.com/gh-aw) o repos de GitHub, accesibles sin bloqueo.
- `gh api` sobre `githubnext/gh-aw`, `copier-org/copier`, `cruft/cruft`, `BetaHuhn/repo-file-sync-action`, `users/blogNetting`.
- WebFetch: `docs.github.com/actions/how-tos/reuse-automations/reuse-workflows`, `docs.github.com/actions/sharing-automations/reusing-workflows`, `docs.github.com/actions/creating-actions/sharing-actions-and-workflows-from-your-private-repository`, `github.github.com/gh-aw/reference/imports`.
- WebSearch: permisos de reusable workflows entre repos privados de cuenta personal; comparativa copier/cruft 2026; automatización real de `copier update` con PR; plan de GitHub necesario para rulesets de organización con "required workflows".
- **Sin aporte:** no se ha buscado en Reddit/HN para esta nota — la documentación oficial bastó para responder las tres preguntas planteadas, sin bloqueos que forzaran escalar.

## 6. Otros

- Esta nota **cierra** el punto 🧪 de [[flujo-agentes-arquitectura]] §7.1 sobre si gh-aw admite includes remotos: sí, y con eso el ejecutor no necesita copiarse a cada repo tampoco, solo los workflows de revisor y reconciliador (que ya se sabía que sí, por ser reusable workflows nativos).
- El repo de Astillero es **privado** (decisión del usuario, 2026-09-25: todo repo de este ecosistema se crea privado, sin excepción), con el ajuste de "Access" de §3.1 activado. GitHub Pro (~4 $/mes de cuenta, no por repo) sigue pendiente de contratar para que Astillero y cada proyecto tengan ruleset y merge queue propios.
- **Cerrado (2026-09-25):** el repo `blogNetting/astillero` existe, con los workflows reutilizables, la plantilla de copier y un `project-example/` de prueba ya publicados — verificado en vivo. Cómo se propagan cambios posteriores a proyectos ya bootstrapped: [[astillero-mantenimiento]].

## Enlaces

- [[flujo-agentes-arquitectura]] — diseño operable que esta nota hace replicable
- [[flujo-agentes-runbook]] — puesta en marcha por proyecto, a actualizar con este mecanismo
- [[astillero-mantenimiento]] — cómo se propagan las actualizaciones a proyectos ya en marcha, no solo a los nuevos
- [[astillero]] — proyecto
- [[patrimonial]] — primer proyecto candidato a usar este mecanismo
- [[_index]]
