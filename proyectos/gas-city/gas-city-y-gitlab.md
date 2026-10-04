---
title: Gas City y el forge — qué depende de GitHub y cuánto cuesta GitLab
created: 2026-10-04
updated: 2026-10-04
tags: [gas-city, forge, github, gitlab, beads, rigs, ciudad]
zona: tecnico
---

Qué piezas de Gas City están atadas a GitHub, cuáles no, y qué cuesta trabajar contra GitLab: la capa de tareas ya lo habla de serie; lo que es GitHub-only es la automatización de PRs.

## 1. Las dependencias de GitHub, una a una

Comprobadas en la instalación real de esta máquina (`/home/netting/gas-city`, `~/.gc/`, `/home/netting/dev/patrimonial`), no de memoria.

| Dependencia | Dónde vive | ¿Ata los repos del usuario? |
|---|---|---|
| Los packs se descargan de `github.com/gastownhall/...` | `city.toml`, `pack.toml`, `packs.lock` (con el SHA fijado) | **No** — es GitHub como proveedor de la herramienta. Está cacheado en `~/.gc/cache` |
| `merge_strategy=pr` / `mr` usa `gh` | prompt del refinery del pack (el mismo SHA fijado en `packs.lock`) | **Sí, solo si se activa.** El defecto es `direct` |
| Flujos `github-issue-*` y `github-pr-review` | `.gc/scripts/github_api.py` (lleva `github.com` literal: *"GitHub URL must start with https://github.com/"*) y `github_reports.py`; las fórmulas son ficheros sueltos del pack (`github-issue-fix.formula.toml`, `github-issue-triage.formula.toml`, `github-pr-review.formula.toml`) | Sí, pero son opt-in: no forman parte de la cadena por defecto (`build-from-plan`, `do-work`, `fix-convoy`…) |
| `gc github pr` (monitor de PRs) | el propio binario `gc`: `gc github pr backfill — Query configured GitHub PR readiness monitors` | Sí, pero opt-in |
| Beads sync | `.beads/config.yaml` de **cada repo**: `sync.remote: "git+https://github.com/blogNetting/patrimonial.git"` | **No es API de GitHub: es un remoto git** (de ahí la rama `__dolt_remote_info__` en `origin`) |

Textual del refinery: *"`direct` — merge to target and push normally"* · *"`mr` / `pr` — push the rebased source branch and create or update a GitHub PR"* · *"Do not call `gh pr create` for the work bead"* · *"verify `gh pr view` reports an open same-repository PR"* ([prompt del refinery](https://raw.githubusercontent.com/gastownhall/gascity-packs/3b3b89f2011e06d84459aa7bea1552382f13930a/gastown/agents/refinery/prompt.template.md)).

## 2. Lo que NO depende de GitHub (verificado)

- **Cero menciones** de `github` ni de `gh` en `agents/`, `template-fragments/`, `formulas/` y `commands/` de la ciudad.
- El flujo por defecto es local y con **git puro**: `git fetch --prune origin` → `git rebase origin/$TARGET` → `git merge --ff-only temp` → `git push origin $TARGET`.
- Los artefactos caen en `plans/<slug>/` y los beads viven en un Dolt **local** (`dolt_mode: server` en `.beads/metadata.json`).
- **Beads es local-first**: la base de tareas vive en el Dolt de la ciudad; el forge es un espejo y el sitio donde poner la puerta.

## 3. La pieza que ya habla GitLab: `bd`

`bd` (beads) trae **GitLab como ciudadano de primera, con la misma paridad que GitHub**:

```
bd gitlab  →  projects · pull · push · status · sync
bd github  →  repos    · pull · push · status · sync
```

Configuración del lado GitLab: `gitlab.url`, `gitlab.token`, `gitlab.project_id`, `gitlab.group_id`, `gitlab.default_project_id` (`bd gitlab --help`). El lado GitHub: `github.token`, `github.owner`, `github.repo`, `github.repository`, `github.url`.

Es decir, **la mitad de issues/tablero ya está construida para GitLab**. Lo que no existe para GitLab es la mitad de «comentar y revisar en el MR».

## 4. El coste de GitLab, por capa

| Capa | ¿GitLab? | Coste |
|---|---|---|
| Repo como remoto (push, ramas, worktrees) | ✅ indiferente | Cero, es git |
| Bucle alcalde → obreros → `plans/` | ✅ | Cero |
| Fusión `merge_strategy=direct` | ✅ git puro | Cero |
| Beads sync | ✅ cambiar la URL | Una línea en `.beads/config.yaml` |
| Issues/tablero (`bd gitlab`) | ✅ nativo | 5 variables. Cero código |
| `merge_strategy=pr` (el refinery abre el MR) | ❌ usa `gh` | Escribir el equivalente (`glab`/API) |
| Flujos `github-issue-*` y `github-pr-review` | ❌ `github.com` a fuego | Reescribir los scripts que comentan en el issue/MR |
| `gc github pr` (monitor) | ❌ solo GitHub | No hay equivalente |

**Conclusión:** con `merge_strategy=direct` y `bd gitlab` para el tablero, el coste es **configuración, no desarrollo**. El desarrollo aparece solo si se quiere que el agente abra y comente MRs por su cuenta. En el pack, la palabra «gitlab» no aparece ni una vez: ahí está el trabajo pendiente.

## 5. Rigs, forja y ciudad

**El forge no vive en la ciudad: vive en cada rig**, dentro de su propio repo.

Config resuelta de la ciudad `NeTT-City` (2 rigs), sin ningún campo de host ni remoto:

```
[[rigs]]
name = "patrimonial"   path = "/home/netting/dev/patrimonial"   default_branch = "main"
[[rigs]]
name = "radar"         path = "/home/netting/dev/radar"         default_branch = "main"
```

Y cada repo lleva el suyo:

```
# /home/netting/dev/radar/.beads/config.yaml
sync:
    remote: "git+https://github.com/blogNetting/radar.git"
```

Consecuencias:

- **Varios rigs en repos distintos es lo normal** — ya ocurre hoy. Y **mezclar forjas entre rigs no da problema estructural**: no hay ninguna capa por encima que asuma un forge.
- **Meter un rig GitLab en esta misma ciudad** cuesta, todo por-rig: `gc rig add`, el `sync.remote` + claves `gitlab.*` en ese repo, y un patch `[[patches.agent]] dir = "<rig>"` si se quieren apagar/reescribir los agentes con marca GitHub.
- **Ojo con ese último punto:** el pack instala sus roles GitHub **por rig**, en todos. En el config resuelto aparece un `issue-triager` por rig, descrito como *"Issue triager for GitHub issue evidence, repro analysis, and triage reports"*. Un rig nuevo nacería con uno dentro; no se dispara solo (corre únicamente al lanzar los flujos `github-*`), pero está ahí.
- **¿Ciudad nueva?** No hace falta por el forge. Se justifica solo para cambiar la configuración de la ciudad entera (proveedores/upstreams, reparto de agentes) o para aislar la base de tareas. Dos ciudades son dos servidores de Dolt y dos sitios donde mirar.

## 6. Sin verificar

- No se ha ejecutado `bd gitlab sync` contra un GitLab real (aquí no hay cuenta ni token): la paridad de comandos está leída del propio binario, no probada en marcha.
- No se ha probado beads sync contra un remoto git que no sea GitHub (es git estándar, debería dar igual).
- Dónde se registran los monitores de `gc github pr backfill` («configured monitors»): no aparecen en el config resuelto.
- En el binario `gc` hay al menos una mención a `GITLAB_CI`; función sin comprobar.

## Enlaces

- [[gas-city-alcalde]] — el flujo del alcalde y los tres flujos que parten de GitHub
- [[gas-city-operacion-real]] — la ciudad `NeTT-City` en marcha
- [[gas-city-con-2cerebro]] — qué es un rig y por qué 2cerebro no lo es
- [[forges-gratis-con-puerta-en-main]] — qué forge da puerta gratis en privado
- [[control-de-versiones-y-ci]] — el caso concreto de Patrimonial
- [[_index]]
