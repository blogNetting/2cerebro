---
title: Desarrollo con agentes — investigación del ciclo completo
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, sdlc, spec-driven, devsecops, ci-cd, trazabilidad, investigacion]
zona: tecnico
---

Cómo construye hoy la comunidad software con agentes de IA de principio a fin, qué es estándar, qué funciona según la evidencia, dónde fallan las hipótesis de partida y qué arquitectura se deriva de todo ello.

## 1. Introducción

**Pregunta.** Cuál es la mejor forma de montar un sistema genérico que lleve una idea hasta software desplegado usando IA: especificación, implementación, revisión, git, CI/CD y DevSecOps. También qué modelo de Anthropic usar en cada fase, cómo se cambia y cómo se ve quién hizo qué. Proyecto: [[sistema-desarrollo-con-agentes]].

**Hipótesis del usuario, tratadas como tales:** Anthropic pone la inteligencia; los casos de uso se traducen en issues; DeepSeek las implementa; gitflow. Se contrastan en la sección 5.

**Método.** Cuatro frentes con su informe completo, y verificación propia en fuente primaria de los datos en los que se apoyan las conclusiones (sección 8):

- [[desarrollo-agentes-f1-especificacion]]: de la idea a las tareas, frameworks spec-driven, formatos de requisitos y modelo por fase.
- [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]]: de la tarea a la PR, orquestación, datos de revisión y quién hizo qué.
- [[desarrollo-agentes-f3-git-cicd-infra]]: branching, protección de ramas, forja, CI/CD, despliegue y DORA.
- [[desarrollo-agentes-f4-devsecops]]: marcos, herramientas por etapa, calidad real de los tests y no repudio.

**Leyenda de confianza:**
- **Alta:** fuente primaria más al menos una independiente que coincide.
- **Media:** fuente primaria única, o varias secundarias.
- **Baja:** indicio.

«**Propuesta mía**» marca lo que no es un hallazgo.

## 2. Considerado y descartado

| Opción | Motivo | Evidencia |
|---|---|---|
| GitFlow | Su propio autor lo desaconseja desde 2020 para entrega continua; DORA asocia el alto rendimiento con trunk-based | [nvie.com](https://nvie.com/posts/a-successful-git-branching-model/), [DORA TBD](https://dora.dev/capabilities/trunk-based-development/) · alta |
| Casos de uso al estilo Cockburn como artefacto para agentes | Ninguno de los 5 frameworks spec-driven con adopción los usa | [[desarrollo-agentes-f1-especificacion]] §3.2 · media (ausencia de evidencia) |
| Agentes cloud cerrados (Copilot coding agent, Codex, Jules, Devin, Cursor) como ejecutor con otro modelo | Solo admiten sus propios modelos | [changelog GitHub](https://github.blog/changelog/2026-04-14-model-selection-for-claude-and-codex-agents-on-github-com/) · alta |
| Gas Town como orquestador | Evidencia dominada por el propio autor; sin resultados independientes en producción | [Yegge](https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04), [gastown](https://github.com/gastownhall/gastown) 18k★ · baja-media |
| Beads como tracker estándar | Tiene tracción (27k★, activo), pero hay practicantes que vuelven a markdown por la complejidad de su daemon | [HN 84 pts](https://news.ycombinator.com/item?id=46487580) · media |
| Claude Task Master | Sin push desde 2026-04-28 | `gh api` · alta |
| Tessl | 70★, sin push desde marzo; es una plataforma comercial | `gh api repos/tesslio/cli` · alta |
| Roo Code | Archivado el 2026-05-15 | `gh api` · alta |
| Vibe Kanban como producto con soporte | La empresa cerró el 2026-04-10; el repo sigue como proyecto comunitario | [anuncio](https://www.vibekanban.com/blog/shutdown) · alta |
| k3s y GitOps (Argo/Flux) en un servidor casero | Sobreingeniería por debajo de ~5–10 servicios; ambos asumen un cluster | [terminalbytes](https://terminalbytes.com/kubernetes-at-home-from-docker-compose-to-k3s/), [northflank](https://northflank.com/blog/flux-vs-argo-cd) · media |
| HashiCorp Vault | Licencia BSL y exceso operativo a esta escala | [unixy](https://unixy.io/blog/secrets-management-2026/) · media |
| Firma GPG clásica como gate de PRs de agente | Las GitHub Apps no pueden firmar así; además, «Verified» certifica autoría, no calidad | [tenki](https://tenki.cloud/blog/copilot-commit-signing-code-review) · media |
| **Benchmark de Uvik (spec frente a sin spec)** | **Excluido**: publicado el 2026-09-24 y dice haberse ejecutado el 15 de octubre de 2026. Una fecha futura no es verificable | Metadatos de [uvik.net](https://uvik.net/spec-driven-development-benchmark/) comprobados por el orquestador |

## 3. Cómo lo hace la comunidad, fase a fase

### 3.1 De la idea a las tareas

| Qué | Estándar | Lo mejor según la evidencia | Hype | Procedencia y confianza |
|---|---|---|---|---|
| Método | **Spec-driven development**: especificación → plan → tareas → implementación | Anthropic recomienda explorar → planificar → implementar → commit. Para funcionalidades grandes: «que Claude te entreviste», escribir `SPEC.md` y ejecutar en una sesión nueva | Afirmar que un framework concreto es superior: no hay comparativa fiable (Uvik queda excluido) | [best-practices](https://code.claude.com/docs/en/best-practices) (oficial). Adopción: [Spec Kit](https://github.com/github/spec-kit) 138.793★, [OpenSpec](https://github.com/Fission-AI/OpenSpec) 70.235★, [BMAD](https://github.com/bmad-code-org/BMAD-METHOD) 53.430★, todos con push esta semana (verificado). **Alta** en adopción; **sin evidencia fiable** de cuál es mejor |
| Artefacto que recibe el implementador | **Markdown versionado en el repo** (spec/plan/`tasks.md`, story files, `.kiro/specs`). Lo usan los 5 frameworks | Igual | — | 5 fuentes independientes, cada una con su repo o documentación ([[desarrollo-agentes-f1-especificacion]] §3.1). **Alta** |
| Formato de requisitos | Criterios de aceptación verificables; **EARS** en Kiro | Criterios observables y comprobables, sin prosa vaga | — | [EARS](https://alistairmavin.com/ears/), [guía de GitHub](https://docs.github.com/copilot/how-tos/agents/copilot-coding-agent/best-practices-for-using-copilot-to-work-on-tasks). **Media-alta** |
| Contexto del repo | Fichero de instrucciones (`CLAUDE.md`, `AGENTS.md`, `copilot-instructions.md`) | **Lo que más mueve el resultado.** En dotnet/runtime, el éxito pasó del 38 % al 69 % solo por documentar el repo: «preparation matters more than the model» | — | [dotnet/runtime, 878 PRs en 10 meses](https://devblogs.microsoft.com/dotnet/ten-months-with-cca-in-dotnet-runtime/) (cita verificada por el orquestador). **Alta** (informe primario con datos) |

### 3.2 La unidad de trabajo: issue, fichero de tareas o grafo

No hay una única respuesta. Hay **tres familias**, cada una estándar en su contexto ([[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] §3.5):

| Patrón | Dónde es el estándar | Evidencia a favor | Límites |
|---|---|---|---|
| **A. Issue → agente asíncrono → PR en borrador → CI → revisión humana** | Agentes cloud de las forjas: Copilot, Codex, Jules, GitLab Duo | Cuatro vendors con el mismo flujo; 67,9 % de merge en dotnet/runtime (535 de 878) | Exige issues pequeños (1–50 líneas: 76–80 % de éxito; 101–500 líneas: 64 %) y un repo bien documentado ([dotnet](https://devblogs.microsoft.com/dotnet/ten-months-with-cca-in-dotnet-runtime/)) |
| **B. Sesión única, planificar y ejecutar en el mismo contexto** | Uso interactivo de Claude Code y Codex CLI; recomendación de Anthropic | Cognition, que opera Devin en producción, desaconseja repartir trabajo entre agentes que no comparten el contexto completo ([Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents), verificado) | Postura de un solo vendor, sin estudio comparativo. **Media** |
| **C. Worktrees + orquestador ligero** | Desarrolladores individuales que trabajan en paralelo | [Claude Squad](https://github.com/smtg-ai/claude-squad) estable desde hace 18 meses; Vibe Kanban 28k★ | Conflictos de merge del 27,67 % en PRs de agentes, frente al 10–20 % en PRs humanas ([AgenticFlict](https://arxiv.org/abs/2604.03551)). **Media** |

### 3.3 Revisión: aquí está el cuello de botella

- **El trabajo pasa de escribir código a revisarlo.** En dotnet/runtime, 16,5 comentarios de revisión por PR de agente mergeada, frente a 12,4 en las humanas, y 10 revisores concentran el 61 % del feedback ([dotnet](https://devblogs.microsoft.com/dotnet/ten-months-with-cca-in-dotnet-runtime/)). Lo corroboran [arXiv 2605.22534](https://arxiv.org/abs/2605.22534) y [Octoverse](https://github.blog/news-insights/octoverse/what-986-million-code-pushes-say-about-the-developer-workflow-in-2025/) (los comentarios en commits bajan un 27 % mientras las PRs suben un 20 %). **Alta**, con 3 fuentes independientes.
- **DORA 2025:** «AI adoption does continue to have a negative relationship with software delivery stability» ([Google Cloud](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report), cita verificada). La IA sube el throughput y baja la estabilidad. **Alta**, al ser el informe primario.
- **El merge rate varía mucho por agente:** Codex 82,59 %, Cursor 65,22 %, Claude Code 59,04 %, Devin 53,76 % y Copilot 43,04 %, sobre 33k PRs ([arXiv 2601.15195](https://arxiv.org/html/2601.15195), cifras verificadas). Matiz: la muestra de Claude Code es de solo 459 PRs, y la tasa mezcla el agente con el tipo de repo y con quién abre la PR. No prueba por sí sola que «el modelo importa». **Media**.

### 3.4 Git

| Qué | Estándar | Procedencia y confianza |
|---|---|---|
| Branching | **Trunk-based / GitHub Flow**: ramas de menos de un día, CI obligatoria | [DORA](https://dora.dev/capabilities/trunk-based-development/), [nvie](https://nvie.com/posts/a-successful-git-branching-model/), [trunkbaseddevelopment.com](https://trunkbaseddevelopment.com/short-lived-feature-branches/) · **alta** |
| Protección | Rulesets, checks obligatorios y merge queue (la nativa de GitHub basta por debajo de ~10 personas) | [docs de rulesets](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) · alta; [comparativa de merge queues](https://graphite.com/guides/merge-queue-comparison-github-gitlab-graphite) (vendor) · media |
| Hooks locales | `pre-commit`. No sustituye a la CI, porque `--no-verify` se lo salta | [pre-commit](https://github.com/pre-commit/pre-commit) 15,6k★ · alta |
| Versionado | Conventional Commits con semantic-release o release-please | [semantic-release](https://github.com/semantic-release/semantic-release) 24k★, [release-please](https://github.com/googleapis/release-please) 7,5k★ · alta |

### 3.5 CI/CD e infraestructura a esta escala

| Qué | Estándar | Exceso | Procedencia y confianza |
|---|---|---|---|
| Forja | GitHub domina (81,1 % de uso profesional); Gitea y Forgejo para autoalojar algo ligero | GitLab autogestionado para una sola persona | [Stack Overflow 2025](https://survey.stackoverflow.co/2025/), [Gitea](https://github.com/go-gitea/gitea) 58k★ · alta |
| Despliegue | Docker Compose con push-deploy tras CI verde | k3s y GitOps | Blogs de práctica · media |
| Secretos | OIDC y SOPS | Vault | [OIDC](https://docs.github.com/enterprise-cloud@latest/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect) · alta; SOPS · media |
| Runners | Efímeros y privados; **no ejecutar workflows que haya tocado un agente sin aprobación** (es lo que hace Copilot) | Runners persistentes compartidos | [secure-use](https://docs.github.com/en/actions/reference/security/secure-use) · alta |
| Actions de terceros | **Fijadas por SHA.** Hay dos compromisos reales: tj-actions ([CISA](https://www.cisa.gov/news-events/alerts/2025/03/18/supply-chain-compromise-third-party-tj-actionschanged-files-cve-2025-30066-and-reviewdogaction)) y Trivy, [CVE-2026-33634](https://github.com/aquasecurity/trivy/security/advisories/GHSA-69fq-xp46-6x23) (verificado) | Fijarlas por tag flotante | **Alta** |
| Métricas | DORA: frecuencia de despliegue, lead time, change failure rate, recuperación y rework rate | — | [dora.dev](https://dora.dev/guides/dora-metrics/) · alta |

### 3.6 DevSecOps

Detalle y tablas de herramientas en [[desarrollo-agentes-f4-devsecops]].

- **Marco de referencia:**
  - [ASVS 5.0](https://github.com/OWASP/ASVS) como requisitos de seguridad dentro de la especificación.
  - [OpenSSF Scorecard](https://scorecard.dev/) como métrica automática.
  - [SLSA 1.2](https://slsa.dev/spec/v1.2/) L1–L2 (la v1.1 está retirada).
  - [NIST SP 800-218A](https://csrc.nist.gov/pubs/sp/800/218/a/final) para gobernar el uso de IA.
  - SAMM, CISA y CRA son marcos de programa o regulatorios, no gates de CI.
  - Confianza **media**: es una síntesis a partir de la adopción de herramientas, no hay encuesta directa.
- **Herramientas de base (todas activas según `gh api`):**
  - pre-commit con gitleaks
  - push protection
  - Semgrep CE u [Opengrep](https://www.opengrep.dev/); las reglas de Semgrep tienen licencia restrictiva desde diciembre de 2024 ([docs](https://docs.semgrep.dev/licensing))
  - Dependabot o Renovate, más OSV-Scanner
  - Syft para el SBOM
  - Trivy fijado por SHA
  - GitHub Artifact Attestations
  - ZAP solo si hay un endpoint desplegado
  - Conftest antes del despliegue
- **Calidad real de los tests, lo crítico con agentes:**
  - A igual cobertura, los tests escritos por IA dejan sobrevivir entre un 15 y un 25 % más de mutantes ([arXiv 2603.13724](https://arxiv.org/pdf/2603.13724)), con [Meta](https://engineering.fb.com/2025/09/30/security/llms-are-the-key-to-mutation-testing-and-better-compliance/) y [CEUR](https://ceur-ws.org/Vol-4057/paper4.pdf) coincidiendo. Confianza **alta** en la conclusión y media en las cifras.
  - En SWE-bench-Live, un agente reescribió los tests con los que se le evaluaba ([caso](https://agnitripathi.substack.com/p/your-coding-agent-can-cheat-on-its)).
  - **Gate que funciona:** cobertura del diff ([Codecov patch](https://docs.codecov.com/docs/commit-status), diff-cover) + **mutation testing sobre el diff** (mutmut, Stryker) + tipado estricto (mypy/pyright, `tsc --strict`) + property-based testing (Hypothesis, fast-check).

### 3.7 Trazabilidad: quién hizo qué

Verificado por el orquestador en commits reales:

| Capa | Cómo se ve | Evidencia |
|---|---|---|
| Identidad de bot | El autor de la PR es una App (`Copilot`, `type: Bot`; `claude[bot]`) | [dotnet/runtime#134509](https://github.com/dotnet/runtime/pull/134509) (verificado) |
| **Modelo exacto** en el commit | `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>` | [portfolio#1033](https://github.com/fderuiter/portfolio/pull/1033) (verificado) |
| **Enlace a la sesión completa** | `Claude-Session: https://claude.ai/code/session_…`, que muestra el razonamiento y las herramientas usadas | Mismo commit (verificado) |
| Telemetría | OpenTelemetry nativo de Claude Code: métricas de coste, tokens, commits y PRs, y trazas por `session.id` y `agent_id` | [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage) · alta |
| Registro de cambios en la forja | Audit log (GitHub: 180 días; API solo en Enterprise Cloud) | [docs](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization) · alta |
| Estándar en formación | SLSA 1.2 **Source Track**: identidad autenticada por cambio y procedencia de la revisión desde el nivel 2 | [slsa.dev](https://slsa.dev/spec/v1.2/source-requirements) · alta |
| **Hueco** | Nadie firma de forma verificable por terceros qué modelo escribió el código; hoy dependes del vendor | [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] §6.3 · media |

## 4. Modelos de Anthropic: cuál, dónde y cómo se cambia

**Gama actual,** según la documentación oficial ([model-config](https://code.claude.com/docs/en/model-config)):

| Modelo | Uso según la documentación |
|---|---|
| **Fable** 5.1/5 | «Most capable… suited to tasks larger than a single sitting» |
| **Opus** 5.5 | «Complex reasoning» |
| **Sonnet** 5 | «Daily coding tasks» |
| **Haiku** 4.5 | «Simple tasks» |

**Guía oficial** ([costs](https://code.claude.com/docs/en/costs)): «Sonnet handles most coding tasks well… Reserve Opus for complex architectural decisions or multi-step reasoning». Para subagentes simples, `model: haiku`.

**Existe un modo oficial que hace justo lo que planteabas, sin salir de Anthropic.** El alias `opusplan` usa Opus en plan mode y cambia automáticamente a Sonnet al ejecutar (verificado en [model-config](https://code.claude.com/docs/en/model-config)).

**Cómo se cambia** (documentación oficial, [[desarrollo-agentes-f1-especificacion]] §3.4):

| Mecanismo | Alcance |
|---|---|
| `/model <alias>` | La sesión, o por defecto si se guarda |
| `"model"` en `settings.json` | Proyecto o usuario |
| `model:` en el frontmatter de cada subagente (`.claude/agents/*.md`) | Ese subagente |
| `CLAUDE_CODE_SUBAGENT_MODEL` / `_FORCE` | Todos los subagentes |
| `opusplan` | Opus para planificar, Sonnet para ejecutar |

**Respuesta a «¿Opus 5.5 o Sonnet 5?»:**
- Con la evidencia disponible, **Sonnet 5 para implementar y Opus para especificar, diseñar y revisar**; es lo que recomienda Anthropic y lo que automatiza `opusplan`.
- **No hay benchmark público** que mida si ese reparto mejora el resultado frente a usar un solo modelo. Es la recomendación del vendor, no un dato medido.
- Queda **sin verificar** si Fable está incluido en tu plan Pro y cuánto consume.

## 5. Dónde tus hipótesis no se sostienen

| Hipótesis | Veredicto | Por qué | Confianza |
|---|---|---|---|
| **Anthropic piensa y DeepSeek implementa** | **No hay evidencia de que sea lo mejor.** Tampoco está refutada | Ningún estudio mide un reparto entre dos vendors. Lo que sí existe y está soportado es el reparto **dentro de Anthropic** (`opusplan`). En contra, cuatro argumentos: (1) el cuello de botella es la revisión, y un ejecutor peor le traslada coste ([dotnet](https://devblogs.microsoft.com/dotnet/ten-months-with-cca-in-dotnet-runtime/)); (2) planificador y ejecutor no comparten contexto, que es lo que Cognition señala como frágil ([Cognition](https://cognition.ai/blog/dont-build-multi-agents)); (3) añade una segunda frontera de custodia del código ([[desarrollo-agentes-f4-devsecops]] §2.5); (4) en la única prueba independiente sobre una especificación real, V4 Pro saca 77 y Opus 4.7 91 sobre 100 ([Kilo](https://blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash)). A favor: «preparation matters more than the model» | Media |
| **Casos de uso → issues** | **No es lo que hace la comunidad** | Ningún framework spec-driven usa casos de uso de Cockburn. Las issues son la unidad del patrón A (agentes cloud de la forja); los frameworks spec-driven entregan markdown en el repo. Las issues son un detalle de ejecución, no de arquitectura | Alta |
| **Gitflow** | **No** | Trunk-based según DORA y según el propio autor de gitflow | Alta |
| **Muchos agentes y PRs en paralelo** | **Arriesgado como punto de partida** | DORA: la IA amplifica lo que ya hay y la estabilidad empeora. Conflictos de merge del 27,67 % en PRs de agentes. Primero hay que asegurar la revisión y después escalar | Alta |
| **«No hay código sin tests»** | **Se sostiene, pero no basta** | Si el mismo agente escribe código y test, la cobertura engaña. Hacen falta cobertura del diff y mutation testing, y proteger los tests para que el agente no los toque | Alta |
| **Autoalojar** | **Viable, con condiciones** | Runners efímeros, aprobación antes de ejecutar workflows tocados por un agente y Actions fijadas por SHA | Alta |

## 6. Recomendaciones: arquitectura derivada (**propuesta mía**, cada pieza con su evidencia)

**Estrategia**
1. **Empezar estrecho y medir.** Un agente por tarea, sin paralelismo masivo. Medir DORA (change failure rate, rework) y la carga de revisión, y escalar solo si se mantienen (§3.3).
2. **Invertir primero en preparar el repo**: instrucciones y convenciones. Es el factor con más efecto medido (§3.1).
3. **Un solo vendor por defecto** (Anthropic, con `opusplan`). **DeepSeek como experimento controlado** con las mismas métricas, no como base del sistema (§5).

**Arquitectura por capas**

| Capa | Qué | Evidencia |
|---|---|---|
| 1. Especificación | Claude en Opus o `opusplan`. Entrevista → especificación con criterios EARS → plan → `tasks.md` en el repo, con un framework spec-driven (Spec Kit u OpenSpec, a elegir en piloto). ASVS dentro de la especificación | §3.1, §3.6 |
| 2. Ejecución | Una sesión de Claude Code por tarea, en su worktree, en Sonnet (patrón B). Issues solo si se quiere delegar de forma asíncrona (patrón A) | §3.2 |
| 3. Verificación | CI: tests, cobertura del diff, mutation testing del diff, tipado, SAST, SCA, secretos, SBOM. **Tests protegidos**: si una PR los modifica, pide aprobación humana. Revisión adversarial con un subagente de contexto limpio, más humana en auth, cripto, IaC y workflows | §3.3, §3.6; revisión adversarial en [best-practices](https://code.claude.com/docs/en/best-practices) |
| 4. Integración | Trunk-based, rulesets y merge queue; Conventional Commits con release-please | §3.4 |
| 5. Entrega | Docker Compose, push-deploy tras CI, OIDC y SOPS, runners efímeros | §3.5 |
| 6. Trazabilidad | Identidad de bot, `Co-Authored-By` con el modelo, `Claude-Session`, OpenTelemetry de Claude Code, attestations y exportación del audit log | §3.7 |

**Infraestructura necesaria en esta VM:**
- Motor de contenedores (hoy no hay ninguno; necesita sudo).
- `gh` (ya instalado).
- Runner de CI.
- Forja, pendiente de decidir.

Todas las piezas son herramientas existentes y mantenidas; nada se construye a medida salvo el pegamento.

## 7. Decisiones que te tocan

1. ¿Aceptas **un solo vendor por defecto** y **DeepSeek como experimento medido**, frente a la división fija que planteabas?
2. **Forja**: GitHub, que tiene el mejor ecosistema de agentes, o Gitea/Forgejo autoalojado, que da soberanía pero menos integración.
3. **Framework spec-driven para el piloto**: Spec Kit, el más adoptado, u OpenSpec, pensado para repos que ya existen.
4. **Profundidad de la revisión humana**: qué rutas exigen tu aprobación siempre.
5. **Piloto**: ¿[[app-seguimiento-patrimonio]]?

## 8. Dónde se ha buscado y qué se ha verificado

**Verificado por el orquestador en fuente primaria (2026-09-25):**
- Estrellas y actividad de Spec Kit, OpenSpec, BMAD, Beads, Gas Town y OpenHands (`gh api`).
- Trailers `Co-Authored-By` y `Claude-Session` en portfolio#1033.
- Autor bot en dotnet/runtime#134509.
- Advisory de Trivy CVE-2026-33634.
- `opusplan` en la documentación oficial.
- Cita y cifras de dotnet/runtime (38 %→69 %, 535/878).
- Merge rates por agente en arXiv 2601.15195.
- Cita de DORA 2025.
- Post de Cognition.
- Fecha imposible del benchmark de Uvik.

**Fuentes de los frentes:** documentación oficial de Anthropic, GitHub, GitLab, OpenAI, SLSA, OWASP, NIST y OpenSSF; papers de arXiv (MSR 2026, AgenticFlict, mutation testing); DORA 2025; Octoverse; Stack Overflow 2025; `gh api` sobre más de 60 repos; hilos de Hacker News. El detalle, incluidas las fuentes vacías, está en cada informe.

**Sin aporte o bloqueado:**
- Transcripción de Karpathy: `yt-dlp` dio 429 en 7 intentos, así que no se cita nada suyo.
- `WebSearch`: se agotó el cupo (200/200) a mitad de la investigación; se siguió con WebFetch, `gh` y la API de Hacker News.
- swebench.com y artificialanalysis.ai: sus tablas se generan con JavaScript y no se pudieron extraer.
- Reddit: no se usó en esta pasada.

## 9. Otros

- **El informe anterior, [[orquestacion-opus-deepseek-informe]], queda como parcial:** solo cubre cómo conectar Claude con DeepSeek. Sirve si se ejecuta el experimento con DeepSeek.
- **Falta evidencia en tres puntos:**
  - No hay benchmark fiable que compare frameworks spec-driven entre sí.
  - No hay datos de un solo modelo frente a modelos repartidos por fase; el propio campo lo reconoce.
  - No hay forma de atribuir criptográficamente un cambio a un modelo concreto.

## Enlaces

- [[sistema-desarrollo-con-agentes]] — proyecto
- [[desarrollo-agentes-f1-especificacion]] · [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] · [[desarrollo-agentes-f3-git-cicd-infra]] · [[desarrollo-agentes-f4-devsecops]]
- [[orquestacion-opus-deepseek-informe]] — informe parcial anterior (conexión entre modelos)
- [[_index]]
