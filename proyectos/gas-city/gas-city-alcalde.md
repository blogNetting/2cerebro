---
title: Gas City — trabajar con el alcalde: qué hace, quién te pregunta y qué te llega
created: 2026-09-29
updated: 2026-09-29
tags: [gas-city, alcalde, mayor, flujo, aprobacion]
zona: tecnico
---

Cómo es usar Gas City hablando solo con el alcalde: hay dos alcaldes distintos, solo uno planifica contigo, el único que te pregunta es él, y por defecto no se sube nada sin tu permiso. Sacado de la documentación oficial y de los ficheros de los packs, leídos el 2026-09-29.

## 1. Los dos alcaldes

| | Alcalde del pack `gastown` | Alcalde del pack `gascity` (skill `gc.mayor`) |
|---|---|---|
| **Qué hace** | Reparte el trabajo en cuanto lo recibe: *«file -> assign -> grind»* ([prompt](https://github.com/gastownhall/gascity-packs/blob/main/gastown/agents/mayor/prompt.template.md)) | *«inspect, interview, write planning artifacts, create work when approved, and launch formulas»* ([skill](https://github.com/gastownhall/gascity-packs/blob/main/gascity/skills/mayor/SKILL.md)) |
| **Cómo se le habla** | `gc session attach mayor` ([docs](https://docs.gascity.com/getting-started/coming-from-gastown)) | Mencionándolo en tu sesión («Mayor, …») o con «Use skill gc.mayor» ([README del pack](https://github.com/gastownhall/gascity-packs/tree/main/gascity)) |
| **Qué pasa con lo terminado** | El refinery fusiona a `main` por defecto: `merge_strategy` *«defaults to `direct`»*, y `direct` es *«merge to target and push normally»* ([prompt del refinery](https://github.com/gastownhall/gascity-packs/blob/main/gastown/agents/refinery/prompt.template.md)). Para recibir PR hay que marcar cada tarea con `merge_strategy=pr` | No sube nada por defecto: `push` / `open_pr` valen `false` (*«Allow the publish stage to push and open a PR after all checks pass»*) |

**Para lo que quiere el usuario —planificar juntos y desentenderse de lo de dentro— el que encaja es `gc.mayor`.**

## 2. El flujo, paso a paso

Ejemplo ancla: «endpoint que devuelve lo que valen mis monedas en euros» en Patrimonial.

1. **Montar la ciudad una vez**: `gc init`, `gc start`, `gc rig add <ruta>`, `gc import add --name gc https://github.com/gastownhall/gascity-packs.git//gascity` y los roles del pack en `city.toml` (`[rigs.imports.gc]`), después `gc import install`.
2. **Hablar con el alcalde**: «Mayor, quiero un endpoint que devuelva lo que valen mis monedas en euros».
3. **Mira el repo antes de preguntar**, y luego pregunta: *«Interview one material question at a time and include your recommended answer with each question»*.
4. **Requisitos**: escribe `plans/<slug>/requirements.md` (problema, solución, historias de usuario con 2-5 criterios de aceptación, fuera de alcance). No los aprueba solo: *«Do not mark requirements, implementation plans, or task plans approved without explicit user approval»*.
5. **Plan de implementación**, basado en el código real. Tú lo apruebas.
6. **Crea las tareas y lanza el flujo.** Con requisitos y plan aprobados, el punto de entrada es `build-from-decompose`.
7. **Dentro, sin ti**: tareas en paralelo, cada una en su worktree; revisión en tres carriles (aceptación, pruebas de tests, sencillez); arreglos hasta `max_iterations` (10 por defecto).
8. **Lo que te llega**: en `plans/<slug>/`, los artefactos y `factory-run.md`, con *«what ran, what was proven, and the suggested next action»*. Nada subido a GitHub salvo que actives `push` u `open_pr`.

## 3. Quién te pregunta

**Solo el alcalde, en tu sesión.** Los agentes de dentro comparten una plantilla que se lo prohíbe: *«Never ask a human whether to proceed after a successful claim»* ([plantilla](https://github.com/gastownhall/gascity-packs/blob/main/gascity/template-fragments/gc-role-worker.template.md)).

La única excepción documentada es la etapa de requisitos cuando se lanza *dentro* del flujo en modo `interactive`: *«ask only the minimum question needed to unblock the artifact»*. En el camino del §2 no ocurre, porque los requisitos ya los ha hecho el alcalde contigo. **Dónde aparecería esa pregunta —en la sesión tmux de ese agente, o avisándote— no está documentado: sin verificar.**

**Corrección:** una versión anterior de esta explicación decía que «el proceso de dentro puede pararse a preguntarte». Es falso en el camino normal.

## 4. Tus mandos

- `interaction_mode`: `interactive` (por defecto) conserva preguntas y menús de aprobación; `autonomous` decide y deja registro; `headless` nunca pregunta.
- `open_pr=true`: el resultado te llega como PR.
- `max_iterations`: tope de intentos de arreglo.
- `drain_policy`: `separate` (en paralelo) o `same-session` (en serie).

## 5. Aviso para quien use el pack `gastown`

El flujo de [[gas-city-con-2cerebro]] §4 lanza trabajo con `mol-polecat-work`, la fórmula del pack `gastown`, cuyo final es el refinery. Con la `merge_strategy` por defecto (`direct`), **el trabajo se fusiona solo en `main`**, lo que choca con su propio paso 7 («no fundir nada tú hasta tener base de confianza»). Si se usa ese camino, marcar las tareas con `merge_strategy=pr`.

## 6. Usar issues de GitHub, si hace falta

No es el camino por defecto (§2), pero el pack `gascity` trae tres flujos que parten de un issue o una PR en vez de una conversación con el alcalde ([README del pack](https://github.com/gastownhall/gascity-packs/tree/main/gascity)):

| Flujo | Qué hace | Qué te deja |
|---|---|---|
| `github-issue-triage` | Analiza el issue, no toca código | *«Triage report and a sticky issue comment, created or updated; no implementation»* |
| `github-issue-fix` | Implementa y revisa el arreglo | *«Implemented and reviewed issue fix; sticky issue-fix status comment created or updated; optional draft or ready PR»* |
| `github-pr-review` | Revisa una PR ya abierta | *«Review report and a sticky PR comment for the current head, created or updated; no code changes, formal GitHub review, or merge»* |

«Sticky comment» es un único comentario que el flujo va actualizando en el propio issue o PR, en vez de añadir uno nuevo cada vez — así el seguimiento queda ahí, no disperso.

Con esto, el camino GitHub-primero sería: abres el issue tú (o lo hace el alcalde tras planificar), lanzas `github-issue-fix` sobre él, y el resultado — el estado y, si quieres, la PR — vive en GitHub en vez de solo en `plans/`. Sigue valiendo lo del §1: si quieres PR en vez de fusión directa, tienes que pedirlo (aquí ya es el propio flujo el que abre PR, «optional draft or ready PR», así que no hace falta tocar `merge_strategy`).

**No verificado:** si estos tres flujos disparan solos al abrirse un issue o una PR (evento) o si hay que lanzarlos a mano cada vez. La página de [Orders](/tutorials/07-orders) —donde vive el disparo por evento— no se ha leído todavía.

## 7. Operar con Claude y DeepSeek a la vez

El detalle completo, con el porqué de cada regla, está en [[gas-city-instalacion-y-modelos]] §4. Resumen para operar:

- **Declara `upstream` en cada agente, siempre.** Sin él, el agente hereda lo que tenga el entorno del servidor tmux en ese momento — en esta máquina, DeepSeek — y no lo que pone `city.toml`.
- **Fija `model` en cada agente, siempre.** El harness `claude` no trae modelo por defecto; sin fijarlo, puede acabar pidiéndole a Anthropic un nombre de modelo de DeepSeek, o viceversa.
- **Un `upstream` por proveedor**, con `base_url` explícito en los dos — incluido el de Anthropic, aunque en una máquina normal no haría falta ponerlo.
- Con eso, un agente `mayor` puede ir en Opus/Sonnet reales y los `polecat`/`obrero` que implementan en DeepSeek, o cualquier combinación, mezclando ambos en la misma ciudad sin que se crucen.

## 8. Verificación

Citas comprobadas de forma mecánica contra los ficheros descargados del repo `gastownhall/gascity-packs` (skill `gc.mayor`, README del pack `gascity`, plantilla `gc-role-worker`, `build-basic/requirements.md`, prompts del alcalde y del refinery de `gastown`) y contra `docs.gascity.com/getting-started/coming-from-gastown`. **No probado en una ciudad real.**

## Enlaces

- [[_index]] — índice de esta carpeta
- [[gas-city-traje-a-medida]] — por qué Gas City
- [[gas-city-con-2cerebro]] — el camino con el pack `gastown` y cómo convive con el wiki
- [[gas-city-instalacion-y-modelos]] — instalación y modelos
- [[gas-city-operacion-real]] — el reparto Opus/DeepSeek aplicado de verdad a `mayor` y `obrero-seek`, y el fallo real que impedía que el alcalde arrancara
- [[gas-city-tmux-scroll]] — por qué el scroll con la rueda del ratón no funcionaba en esta misma sesión de tmux y cómo se arregló de raíz
