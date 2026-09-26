---
title: Astillero — mantenimiento y propagación a proyectos existentes
created: 2026-09-25
updated: 2026-09-26
tags: [astillero, mantenimiento, copier, gh-aw, propagacion, versionado]
zona: tecnico
---

Cómo una actualización de `blogNetting/astillero` llega a un proyecto que ya está en marcha (no solo a uno nuevo), y cómo conviven el código de Astillero con el wiki de 2Cerebro. Extiende [[astillero-replicacion]], no la repite.

## 1. Por qué esto no pasa solo (y por qué ahora es así a propósito)

**Corrección cerrada el 2026-09-26: nada se actualiza solo, ni siquiera los reusable workflows.** Hasta hoy, `revisar.yml`/`reconciliar.yml`/`reproducir-bug.yml` referenciaban `@main` (`uses: blogNetting/astillero/...@main`), y por cómo resuelve GitHub los reusable workflows (`workflow_call`), eso SÍ se actualizaba solo, en tiempo real, en cada ejecución — sin que nadie lo decidiera para ese proyecto en concreto. Decisión del usuario: quiere aprobar cada actualización, no recibirla sola. Se cerró el hueco: los 5 ficheros de la plantilla (`implementar`, `rehacer`, `revisar`, `reconciliar`, `reproducir-bug`) referencian ahora una versión fija — `@{{ '{{' }} astillero_ref {{ '}}' }}` (pregunta nueva de `copier`, tag real) — no `@main`. **Versión actual: `v0.2.1`** — `v0.1.0` se cortó antes de terminar `SECURITY.md`/la plantilla de bug y quedó incompleta; no usar esa, usar siempre la última etiquetada (`gh api repos/blogNetting/astillero/tags` para comprobar cuál es antes de fijar un proyecto nuevo).

**Los imports de gh-aw y las plantillas de `copier` ya eran así, por otro motivo — no por elección, por mecánica.** Confirmado en fuente oficial: los imports remotos de gh-aw (*"Paths matching owner/repo/path@ref are fetched from GitHub at compile time"* ✔︎) se resuelven al ejecutar `gh aw compile`, no al ejecutar el workflow — el `.lock.yml` ya compilado queda congelado con lo que había en el `ref` el día que se compiló. Igual con `copier`: el proyecto tiene su propia copia de los ficheros estáticos, generada una vez.

**Consecuencia, ahora uniforme en las dos piezas:** hace falta un mecanismo activo de sincronización por proyecto, y ese mecanismo propone una versión nueva concreta, nunca "lo último de `main`" sin más.

## 2. Mecanismo: `astillero-update.yml`, un workflow por proyecto

Cron semanal + `workflow_dispatch` manual. Tres pasos:

1. Comprueba si hay un tag más nuevo que el `astillero_ref` actual del proyecto (`gh api repos/blogNetting/astillero/tags`, o la última release de `release-please`). Si no hay nada nuevo, termina sin hacer nada.
2. `copier update --data astillero_ref=<tag-nuevo> --defaults` — actualiza CODEOWNERS/plantillas/referencias de versión con merge a 3 bandas. Corre desatendido de verdad: reutiliza las demás respuestas previas sin preguntar (`--defaults`, confirmado en [copier.readthedocs.io/en/stable/updating](https://copier.readthedocs.io/en/stable/updating/) ✔︎), pero **no resuelve conflictos solo** — si los hay, deja marcadores tipo git inline (o ficheros `.rej`) en el propio diff, citado literal: *«If the update results in conflicts, you should review those manually before committing»* ✔︎.
3. `gh extension install githubnext/gh-aw` (no viene preinstalado en runners hosted) + `gh aw compile` — recompila con la nueva versión ya fijada en `imports:`.

Si genera diff, el workflow abre un PR con el cambio completo — incluido, siempre visible, el cambio de versión (`@v0.1.0` → `@v0.2.1`, por ejemplo). Sin pieza nueva de revisión: cae bajo el `CODEOWNERS` que ya protege `.github/**` ([[flujo-agentes-arquitectura]] §8) — revisión humana obligatoria antes de mergear, igual que cualquier otro cambio a esa ruta. Si `copier` dejó marcadores de conflicto, quedan visibles en ese mismo PR, no se ocultan.

**Cómo se cortan las versiones de Astillero mismo:** `release-please` (`googleapis/release-please-action`, ya instalado en `blogNetting/astillero`) — cada push a `main` con commits en formato conventional-commits abre o actualiza una PR con el `CHANGELOG.md` y el bump de versión; se aprueba esa PR, no se corre nada a mano. Mismo mecanismo ya elegido para versionar cada proyecto ([[capa-producto]] §5), aplicado aquí a Astillero mismo — que sí es una pieza con "consumidores externos" en el sentido de SemVer (cada proyecto que lo importa), a diferencia de un producto sin API pública.

**No encontré ninguna herramienta ya hecha y mantenida que combine estos dos pasos** (gh-aw compile + copier update) en un solo bot — es diseño propio sobre piezas verificadas por separado, no un patrón que alguien más ya publicó. Si aparece uno mejor, se sustituye (regla de skills de `AGENTS.md`).

## 3. Hallazgo que cambia el runbook: `GITHUB_TOKEN` no sirve para tocar `.github/workflows/`

**El `GITHUB_TOKEN` por defecto de Actions no puede modificar ficheros bajo `.github/workflows/`** — este mismo mecanismo lo va a tocar en el paso 1 (recompilar `.lock.yml`). Evidencia real, no de manual: un PR público que resuelve exactamente este mismo problema lo dice explícito en su descripción — *«it needs a GHTOKEN PAT specifically, since GITHUB_TOKEN cannot push changes under .github/workflows/»* ([riscv/docs-spec-template#156](https://github.com/riscv/docs-spec-template/pull/156)) ✔︎ — y coincide con lo que esta misma sesión ya vivió en directo: hizo falta `gh auth refresh -s workflow` para poder hacer push de un cambio a un workflow ([[flujo-agentes-runbook]]).

**Consecuencia de diseño:** `astillero-update.yml` necesita un PAT con scope `workflow` guardado como secreto del proyecto (no el `GITHUB_TOKEN` por defecto). Añadir a la checklist de [[flujo-agentes-runbook]]: crear y guardar ese PAT en cada proyecto que adopte este workflow de sincronización, con el mismo cuidado que `DEEPSEEK_API_KEY`.

## 4. Convivencia Astillero (código) ↔ 2Cerebro (wiki)

Protocolo corto, seguido de facto toda la sesión, nunca escrito hasta hoy:

1. **Decisión o corrección de arquitectura → primero en `areas/decisiones.md` o en la nota del proyecto**, con fecha y motivo.
2. **Implementación → siempre en el repo real** (`blogNetting/astillero` o el del proyecto que corresponda), nunca en el wiki — regla ya cerrada en `AGENTS.md`, sección «Repositorio y artefactos».
3. **La nota del wiki enlaza siempre a la URL del repo real** y a la ruta local si aplica — patrón ya usado en `astillero.md` («Repo: Astillero»).
4. **Una pieza queda marcada «diseñada» hasta que se ejecuta de verdad**; solo entonces pasa a «probada en vivo», con la evidencia concreta (run id, PR, log) en la misma nota. Nunca se asciende de estado sin esa evidencia — es la disciplina que ya se aplicó hoy en `astillero.md` §Estado.
5. **Cuando el repo real cambia de forma que contradice lo documentado, se edita la nota en la misma operación** — el wiki no es append-only.

## Enlaces

- [[astillero-replicacion]] — mecanismo de replicación base, lo que ya funciona solo
- [[flujo-agentes-arquitectura]] — §12 (replicación) y §8 (CODEOWNERS que protege esta ruta)
- [[flujo-agentes-runbook]] — checklist de puesta en marcha, pendiente de añadir el PAT de §3
- [[astillero]] — proyecto
- [[_index]]
