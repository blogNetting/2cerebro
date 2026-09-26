---
title: Astillero — capa de producto
created: 2026-09-25
updated: 2026-09-25
tags: [astillero, producto, roadmap, backlog, dashboard, releases]
zona: tecnico
---

Cómo el usuario dirige Astillero como Product Owner de una sola persona: captura la idea, decide qué va antes, ve el estado sin leer logs, y los bugs entran con la misma disciplina que las funcionalidades. Se apoya en [[flujo-agentes-arquitectura]], no lo sustituye. Definición y evidencia completa en [[capa-producto-definicion]].

**Leyenda:** ✔︎ = verificado por el orquestador en fuente primaria. **Propuesta mía** = inferencia, no hallazgo.

## 1. El hallazgo que ordena toda esta capa

**Buscando activamente lo contrario, no encontré ni un solo operador solo usando scoring formal de priorización (RICE/ICE/WSJF).** Un hilo real de 130 puntos con ~30 operadores solos ([HN](https://news.ycombinator.com/item?id=40078720) ✔︎) no menciona ninguno de esos términos ni una vez; deciden por juicio directo: *«I cope best when I manage to work on the things that feel most important»*.

**Tampoco se sostiene la mesa de apuestas de Shape Up a esta escala.** Verificado en la fuente: el propio libro exige *«one designer and one programmer»* como mínimo ✔︎ — presupone pluralidad de personas compitiendo por presupuesto. Con un solo decisor no hay mesa que adaptar: solo hay decidir.

**Lo que sí sobrevive de Shape Up no es el proceso, son dos campos del documento:** *Appetite* (cuánto esfuerzo vale la pena) y *No-gos* (qué queda fuera explícitamente), porque acotan el problema antes de que Opus tenga que adivinar el alcance.

**Conclusión:** nada de roadmap con ceremonia. Backlog simple en GitHub Issues, ya elegido en [[flujo-agentes-arquitectura]], reordenado por tu criterio.

## 2. Captura de la idea

Dos fuentes independientes (Anthropic oficial, y un practicante) convergen en la misma forma: **tú escribes 3-4 frases, el agente entrevista, y el resultado es el `spec.md`** que ya sabe producir el framework spec-driven elegido ([[flujo-fase-c1-spec-y-estado]]).

**Lo que escribes tú:**
- **Problema:** qué motiva esto.
- **Appetite:** cuánto esfuerzo vale la pena — no una estimación de horas, un límite.
- **No-gos**, si los tienes claros de entrada.

Eso alimenta la entrevista ya documentada oficialmente por Anthropic: *«For larger features, have Claude interview you first... write a complete spec to SPEC.md»* ✔︎ ([[desarrollo-agentes-f1-especificacion]] §3.4). Encaja directo en el campo «Alcance / Fuera de alcance» que el contrato de tarea **ya tiene** ([[flujo-agentes-arquitectura]] §6) — no es una pieza nueva, es la misma disciplina un escalón antes.

## 3. Panel: qué ves sin leer un log

**Hallazgo que decide esto:** el propio `github/gh-aw` — la fuente del patrón que ya usamos (K3, [[flujo-agentes-arquitectura]]) — documenta GitHub Projects v2 como *«the dashboard»*, pero **no lo usa para sí mismo**. Verificado por el orquestador, dos veces: `projectsV2` de la organización sin resultados para `gh-aw`, y la propia página de proyectos del repo en `0 abiertos, 0 cerrados` ✔︎. Usan en su lugar una tabla con badges de Actions — una vista de ingeniero (¿está verde la última ejecución?), no de producto (¿qué espera mi aprobación?).

**Por eso, para ti, la pieza correcta es la que GitHub prescribe y no usa para sí mismo, porque tu pregunta es la de producto, no la de ingeniería:**
- El reconciliador ya existente ([[flujo-agentes-arquitectura]] §7.4), un script con `gh` sin modelo de por medio, llama directamente a `gh project item-edit`: en cada pasada refleja la etiqueta `estado:*` de cada issue como campo `Status` de un tablero de una sola vista, agrupada por estado. **No es `safe-outputs`** — corrección del 2026-09-25: ese mecanismo es exclusivo de workflows gh-aw compilados (`.md`→`.lock.yml`), y el reconciliador es un `.yml` normal de Actions, sin juicio de IA que validar.
- La misma pasada, semanalmente, llama a `gh api graphql` para un semáforo (`ON_TRACK`/`AT_RISK`/`OFF_TRACK`/`COMPLETE`) en la pestaña Updates del tablero — la única lectura que necesitas sin abrir una sola issue.
- Ambas llamadas usan un PAT propio con scope `project` (secreto `GH_AW_WRITE_PROJECT_TOKEN`) ✔︎ — mismo patrón de secreto que ya usas para DeepSeek y Opus, porque el `GITHUB_TOKEN` por defecto no llega a Projects v2.

## 4. Bugs: la misma disciplina, con un paso delante

**El caso real más sólido de toda esta capa:** Metabase construyó Repro-Bot, un agente que **solo reproduce el bug, nunca lo arregla** — verificado en la cita del propio autor: *«We intentionally did not ask Repro-Bot to fix the issue»* ✔︎. Cifra real dada por el autor: **~10 % de falsos positivos**, con validación humana siempre encima, nada automatizado a partir del bot ✔︎ ([HN](https://news.ycombinator.com/item?id=47767829), comentario verificado literal).

**No encontré ni un caso donde un bug reportado en crudo entre directo a un ejecutor barato sin ese paso intermedio.** Coherente con lo ya visto en `daily-spdd-spec-planner` de GitHub: issues poco especificadas fallan en silencio incluso en su propio dogfood ([[flujo-agentes-arquitectura]] §K3).

**Diseño (propuesta mía, apoyada en el caso Metabase):**
1. Plantilla de issue de bug nativa de GitHub, etiqueta automática `tipo:bug`.
2. **Reproducción**, de solo lectura: un workflow disparado por esa etiqueta confirma el bug y localiza el código — como Repro-Bot, no arregla nada.
3. **Re-especificación por Opus**: convierte el hallazgo en el mismo contrato de tarea de [[flujo-agentes-arquitectura]] §6 — Alcance, interfaces, criterios EARS con test que falla sin el fix.
4. A partir de aquí, **cero diferencia con una funcionalidad**: misma máquina de estados, mismo ejecutor, mismo revisor, mismas puertas de CI. Ningún atajo de «hotfix» que se salte tests o revisión.

## 5. Versionado: un checkpoint, no automatismo

`release-please` (ya evaluado en [[desarrollo-agentes-f3-git-cicd-infra]]) encaja mejor que `semantic-release` para un PO solo: abre una PR de release acumulativa que tú apruebas, un punto de checkpoint deliberado, en vez de publicar sola cada vez que se mergea algo — más propio de una librería con consumidores esperando versión que de una app tuya con despliegue continuo. Si ni eso hace falta, las **notas de release nativas de GitHub** ✔︎ bastan: eliges el tag anterior, un botón.

**SemVer no aplica al conjunto de un producto sin API pública** ✔︎ (el propio spec lo circunscribe a software con consumidores externos) — solo tiene sentido en piezas concretas que sí los tengan (un SDK, un CLI instalable).

**Aclaración (2026-09-25):** este checkpoint es de *versión*, no de *despliegue* — son ejes distintos. El despliegue a producción es continuo, en cada merge a `main` ([[flujo-agentes-arquitectura]] §15, con evidencia real de 40+ operadores solos); el PR de `release-please` sigue corriendo aparte, solo para el changelog/tag legible, sin bloquear el CD.

## 6. Registro de decisiones de producto

**Aquí no hay precedente real que citar** — es la sección más débil de esta capa, y lo digo así. Ni siquiera Shape Up, la fuente más cercana, documenta qué pasa con lo que se descarta: los pitches rechazados «vuelven si son importantes», sin registro ni motivo ✔︎ — es la ausencia de la práctica que se buscaba, no un ejemplo de ella.

**Propuesta mía, sin evidencia de adopción externa:** `proyectos/astillero/decisiones-producto.md`, separado de `areas/decisiones.md` (que es técnico). Una entrada por decisión: qué se prioriza, qué se pospone o se descarta **explícitamente**, y por qué — justo lo que Shape Up demuestra que falta incluso en su propia práctica de referencia.

## 7. Lo que se descarta, y por qué

| Descartado | Motivo | Evidencia |
|---|---|---|
| RICE / ICE / WSJF | Cero practicantes solos encontrados usándolo, tras búsqueda activa dirigida a encontrarlo | [[capa-producto-definicion]], frente 1 |
| Mesa de apuestas de Shape Up | Exige pluralidad de personas por diseño del propio libro | `basecamp.com/shapeup/2.2-chapter-08` ✔︎ |
| Now/Next/Later como tablero | Es una herramienta de comunicación hacia otros; con un solo lector, su función central desaparece | [ProdPad](https://www.prodpad.com/blog/now-next-later-roadmap/) |
| SemVer sobre todo el producto | El propio spec lo limita a software con API pública | [semver.org](https://semver.org/) ✔︎ |
| Bug directo a DeepSeek sin triage | Ningún caso real encontrado que lo haga así; Repro-Bot y el propio gh-aw meten un paso intermedio | §4 |
| Tabla de badges como panel de producto | Es lo que usa `github/gh-aw` para sí mismo, pero responde la pregunta de ingeniero, no la de PO | §3 |

## 8. Dónde se ha buscado

`code.claude.com`, `basecamp.com/shapeup` (varios capítulos, fuente primaria), `prodpad.com`, `elezea.com`, `semver.org`, `martinfowler.com`, `docs.github.com`, `github.github.com/gh-aw` (patterns/ProjectOps, IssueOps, LabelOps, MonitorOps, safe-outputs, auth-projects, agent-factory-status), `metabase.com/blog`, Hacker News vía la API de Algolia (más de 40 consultas entre los dos frentes, con las que no dieron nada documentadas explícitamente en [[capa-producto-definicion]]), verificación en vivo del orquestador con `curl` y GraphQL contra `github/gh-aw`.

## Enlaces

- [[astillero]] — proyecto
- [[capa-producto-definicion]] — pregunta, alcance y evidencia completa de esta fase
- [[flujo-agentes-arquitectura]] — motor de ingeniería que esta capa dirige
- [[_index]]
