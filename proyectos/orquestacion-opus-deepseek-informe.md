---
title: Orquestación Opus/DeepSeek — informe de investigación
created: 2026-09-24
updated: 2026-09-24
tags: [agentes, orquestacion, opus, deepseek, investigacion, devsecops]
zona: tecnico
---

Cómo hacer que Opus (suscripción Pro) dirija y DeepSeek (API) programe: qué hay, qué funciona según la comunidad, qué se descarta y qué arquitecturas pasan a pruebas.

## 1. Introducción

Pregunta: cuál es la mejor forma, contrastada por la comunidad y no por el hype, de que Claude Opus planifique y revise mientras un modelo DeepSeek escribe el código y los tests, dentro de [[sistema-desarrollo-con-agentes]].

Restricciones de partida: Opus va por **suscripción Pro**, no por API; DeepSeek por su API oficial; no hay código sin tests; la VM tiene 4 núcleos, 7 GB de RAM, ni `/dev/kvm` ni motor de contenedores instalado (comprobado con `ls /dev/kvm` y `which podman docker bwrap`).

Método: cuatro frentes en paralelo, cada uno con su informe detallado, y verificación propia en fuente primaria de los datos que condicionan la arquitectura (sección 8).

- [[orquestacion-herramientas-y-patrones]] — patrones, herramientas y git
- [[orquestacion-modelos-y-costes]] — modelos DeepSeek, precios, benchmarks, límites de Pro
- [[orquestacion-experiencia-comunidad]] — Hacker News, Reddit, issues: qué funciona y qué falla
- [[orquestacion-seguridad-ejecutor]] — aislamiento, riesgos y controles

## 2. El hallazgo que condiciona todo

**Con Opus por suscripción, DeepSeek solo puede trabajar en un proceso aparte, lanzado por la sesión de Opus. No puede ir dentro de ella.**

- Los subagentes de Claude Code solo aceptan modelos de Anthropic ([docs sub-agents](https://code.claude.com/docs/en/sub-agents)). La petición de alias personalizados se cerró como `NOT_PLANNED` ([#34821](https://github.com/anthropics/claude-code/issues/34821)). La de enrutar por agente a otro proveedor sigue abierta ([#38698](https://github.com/anthropics/claude-code/issues/38698)).
- `ANTHROPIC_BASE_URL` se aplica a toda la sesión, no a un agente concreto. Los gateways que enrutan por nombre de modelo solo funcionan con API key, no con el login de suscripción ([#38698](https://github.com/anthropics/claude-code/issues/38698)).
- En cambio, un proceso aparte con sus propias variables de entorno sí puede hablar con DeepSeek. DeepSeek publica un endpoint oficial compatible con Anthropic, `https://api.deepseek.com/anthropic` ([docs](https://api-docs.deepseek.com/guides/anthropic_api)), y [DeepClaude](https://github.com/aattaran/deepclaude) (678 puntos en [HN](https://news.ycombinator.com/item?id=48002136)) no es más que eso: un script que lanza Claude Code con esas variables apuntando a DeepSeek.

De ahí sale el esqueleto: **Opus orquesta en su sesión, escribe la especificación, lanza el ejecutor en un worktree aislado, pasa gates deterministas y revisa.**

## 3. Considerado y descartado

| Opción | Motivo | Fuente |
|---|---|---|
| Subagentes de Claude Code con DeepSeek | No lo soporta la plataforma | [docs](https://code.claude.com/docs/en/sub-agents), [#34821](https://github.com/anthropics/claude-code/issues/34821) |
| Proxy dentro de la sesión de Opus (claude-code-router, LiteLLM suplantando los alias) | Exige API key, no suscripción. Además, claude-code-router tiene un error 400 sin solución con DeepSeek en modo thinking más tool calls, y LiteLLM tiene fallos de `reasoning_content` | [#38698](https://github.com/anthropics/claude-code/issues/38698), [ccr#1378](https://github.com/musistudio/claude-code-router/issues/1378), [litellm#42310](https://github.com/BerriAI/litellm/issues/42310) |
| Modo architect de Aider con Opus como arquitecto | Aider no puede usar la suscripción; exigiría Opus por API | [aider architect](https://aider.chat/2024/09/26/architect.html) |
| Roo Code | Repo archivado el 2026-05-15 (verificado con `gh api`) | [Roo-Code](https://github.com/RooCodeInc/Roo-Code) |
| Cline y Kilo Code como ejecutor principal | Están centrados en el IDE y tienen fallos abiertos con DeepSeek: se salta el modo Plan, tool calls mal interpretadas y el orquestador genera tareas demasiado grandes. Quedan como alternativa, no como finalistas | [cline#13729](https://github.com/cline/cline/issues/13729), [cline#13819](https://github.com/cline/cline/issues/13819), [kilo#2580](https://github.com/Kilo-Org/kilocode/discussions/2580) |
| Servidores MCP de delegación (de un solo autor) | Prometedores pero sin contrastar: no hay adopción ni terceros que los usen | [ejemplo](https://github.com/fegone/claude-code-delegate-local) |
| OpenHands como orquestador | Sustituye a Opus en lugar de complementarlo. Sirve como referencia para el flujo issue → PR | [OpenHands](https://github.com/OpenHands/OpenHands) |
| DeepSeek como planificador u orquestador | Evidencia de que planifica mal: se salta la fase de arquitectura y genera subtareas sobredimensionadas | [kilo#2580](https://github.com/Kilo-Org/kilocode/discussions/2580) |
| Kata Containers y Firecracker/E2B como sandbox | Necesitan KVM y esta VM no tiene `/dev/kvm`. E2B autoalojado exige Terraform, Nomad y Consul | [kata#9591](https://github.com/kata-containers/kata-containers/issues/9591), [e2b infra](https://github.com/e2b-dev/infra) |
| Sandbox nativo de Claude Code como única barrera | Ha tenido dos bypasses documentados, uno de ellos abierto 5,5 meses | [oddguan](https://oddguan.com/blog/second-time-same-sandbox-anthropic-claude-code-network-allowlist-bypass-data-exfiltration/), [claudecodecamp](https://www.claudecodecamp.com/p/claude-code-sandboxing-how-sandbox-works-and-what-it-doesn-t-protect) |

## 4. Análisis

### 4.1 Patrón con consenso

Un modelo caro planifica y revisa, otro barato implementa. Es un patrón repetido de forma independiente en configuraciones reales ([HN stacks, 171 puntos](https://news.ycombinator.com/item?id=48413629), [HN DeepClaude](https://news.ycombinator.com/item?id=48002136)). Hay voces en contra: «you need the best model» ([HN](https://news.ycombinator.com/item?id=48002136)). Hay también una medición con el orden inverso: en el leaderboard de Aider, el mejor par fue R1 planificando y Sonnet editando ([leaderboard](https://aider.chat/docs/leaderboards/)). Es controversia real, así que la resolvemos midiendo (sección 5).

Lo que la comunidad repite como condición para que funcione:

| Práctica | Por qué | Fuente |
|---|---|---|
| Especificación tipada (OpenAPI/TS) con criterios de aceptación como contrato | En modelos que no son de frontera, el formato de la especificación mueve la calidad; las especificaciones tipadas triplican la cobertura en el modelo débil | [arXiv 2608.21747](https://arxiv.org/pdf/2608.21747) (reciente, sin contrastar como producto), [Addy Osmani](https://addyosmani.com/blog/good-spec/) |
| Gates deterministas; el LLM propone y el gate valida | «Quality gates need to be deterministic, not LLM-based» | [HN, 6 meses de gobernanza](https://news.ycombinator.com/item?id=47139978) |
| Un test no cuenta hasta que se ha visto fallar en rojo | Un test e2e llevaba semanas en verde sin probar nada | [HN](https://news.ycombinator.com/item?id=49227024) |
| El ejecutor no toca tests sin revisión explícita | En SWE-bench-Live, 3 de 4 ejecuciones «aprobadas» habían editado sus propios tests | [substack](https://agnitripathi.substack.com/p/your-coding-agent-can-cheat-on-its), [dev.to](https://dev.to/brasthapp/your-agent-went-green-by-weakening-the-tests-1iha) |
| Mutation testing sobre los ficheros que toca cada PR | Sin él, la cobertura se puede falsear con tests vacíos | [awesome-testing](https://www.awesome-testing.com/2026/08/mutation-testing-for-agent-written-code) |
| Worktree por tarea y merge queue | Los worktrees son la base habitual; los conflictos semánticos sobreviven a ellos; la merge queue es el nuevo cuello de botella | [MindStudio](https://www.mindstudio.ai/blog/git-worktrees-parallel-ai-coding-agents), [Mergify](https://mergify.com/blog/merge-queues-and-ai-coding-agents) |
| Agentes independientes y trazables en lugar de subagentes opacos | «Sub-agents are a black box. I never use them» (una sola fuente, con experiencia larga) | [HN](https://news.ycombinator.com/item?id=47139978) |

### 4.2 Candidatos a ejecutor (el proceso que habla con DeepSeek)

| Candidato | Modo desatendido | Madurez (`gh`, 2026-09-24) | A favor | En contra |
|---|---|---|---|---|
| **Claude Code + endpoint Anthropic de DeepSeek** (patrón DeepClaude) | `claude -p` | [anthropics/claude-code](https://github.com/anthropics/claude-code) 148k★ | Mismo arnés, skills, hooks y `CLAUDE.md` que el orquestador; integración documentada por DeepSeek ([docs](https://api-docs.deepseek.com/guides/anthropic_api)) | Arnés pesado: 23.132 tokens de sistema frente a 1.340 de Pi, y varias veces más tiempo, con la misma calidad ([benchmark](https://nqawhc.github.io/articles/harness-efficiency-not-quality/)). Un nombre de modelo no reconocido cae sin avisar a `deepseek-flash` ([docs](https://api-docs.deepseek.com/guides/anthropic_api)) |
| **OpenCode** | `opencode run "…"` ([docs CLI](https://opencode.ai/docs/cli/)) | [anomalyco/opencode](https://github.com/anomalyco/opencode) 210k★, push hoy | Peso medio (7.197 tokens de sistema), muy activo, integración oficial en la documentación de DeepSeek | Un intermediario retiró DeepSeek del tier gratuito ([HN](https://news.ycombinator.com/item?id=49388835)). No aplica si usamos la API directa |
| **Pi** | Modos print, JSON y RPC, más SDK de TypeScript ([README](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/README.md)) | [earendil-works/pi](https://github.com/earendil-works/pi) 109k★, push hoy | El más ligero (4 tools) y el más eficiente con la misma calidad ([benchmark](https://nqawhc.github.io/articles/harness-efficiency-not-quality/)); usado a diario con deepseek-v4-pro (~10M tokens/día, [HN](https://news.ycombinator.com/item?id=48413629)) | Arnés mínimo: lo que no trae hay que añadirlo con extensiones |
| Aider (en reserva) | `--message` | [Aider-AI/aider](https://github.com/Aider-AI/aider) 49k★, **sin release desde 2025-08** | Patrón arquitecto/editor nativo, años de uso | Ritmo de desarrollo bajo; tardó en reconocer los modelos V4 ([#5255](https://github.com/Aider-AI/aider/issues/5255)) |

Matiz sobre el benchmark de arneses: usa DeepSeek V4 Flash local, con 24 ejecuciones por arnés. Lo que concluye es que **no se detecta** diferencia de calidad, no que sea idéntica. Lo que sí es claro es la diferencia de coste y tiempo ([artículo](https://nqawhc.github.io/articles/harness-efficiency-not-quality/)).

### 4.3 Modelos y coste

| Modelo | Entrada (miss) valle / punta | Entrada (caché) | Salida valle / punta | Fuente |
|---|---|---|---|---|
| `deepseek-flash` (V4.1-Flash) | $0,15 / $0,30 | $0,003 / $0,006 | $0,60 / $1,20 | [pricing](https://api-docs.deepseek.com/quick_start/pricing) |
| `deepseek-v4-pro` | $0,66 / $1,32 | $0,022 / $0,044 | $1,98 / $3,96 | [pricing](https://api-docs.deepseek.com/quick_start/pricing) |

- El horario punta son las 01–04 y 06–10 UTC, de lunes a viernes. **En horario de España casi todo cae en valle**, a mitad de precio ([pricing](https://api-docs.deepseek.com/quick_start/pricing)).
- La caché de prefijos es automática y en disco ([Context Caching](https://api-docs.deepseek.com/guides/kv_cache)). El arnés pesado cuesta poco en entrada; su coste real está en la salida y en el tiempo.
- Los benchmarks públicos se contradicen: 95,2 % frente a 80,6 % en SWE-bench Verified para el mismo modelo ([Fireworks](https://fireworks.ai/blog/DeepSeekV4Pro-Fable5), un proveedor con interés; frente a agregadores). Tomo como referencia la única prueba independiente sobre una especificación real: Opus 4.7 **91**, V4 Pro **77**, V4 Flash **60** sobre 100 ([Kilo](https://blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash)).
- **Punto ciego económico:** Anthropic no publica los límites numéricos de Pro ([soporte](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-claude-max-or-claude-pro-plan)), y los estimadores locales fallan un 20–40 % ([Reddit](https://www.reddit.com/r/ClaudeCode/comments/1sb8fb3/)). Varios usuarios avisan de que la suscripción ya está subvencionada y mezclarla con pago por token puede ahorrar menos de lo que parece ([HN](https://news.ycombinator.com/item?id=48237663)). **El coste de DeepSeek se puede medir con exactitud; el consumo de cuota de Opus solo aproximadamente, con `/usage`.**

### 4.4 Lo que falla (a vigilar en las pruebas)

- Tool calling de DeepSeek: el endpoint Anthropic responde 400 cuando un proxy añade `type: "custom"` a las tools ([9router#3905](https://github.com/decolua/9router/issues/3905)); JSON intermitente ([agent-kit#329](https://github.com/inngest/agent-kit/issues/329)). Los fallos graves reportados con V4 son en autoalojamiento cuantizado ([Reddit](https://www.reddit.com/r/LocalLLaMA/comments/1vtu779/)), que no es nuestro caso.
- Los modelos baratos paran antes de terminar o entran en bucles ([HN](https://news.ycombinator.com/item?id=48688700)).
- «Co-alucinación»: una PR con buena prosa sesga al revisor. Opus debe ejecutar y verificar, no solo leer el diff ([HN](https://news.ycombinator.com/item?id=46292682)).

### 4.5 Seguridad del ejecutor

Detalle en [[orquestacion-seguridad-ejecutor]]. Resumen de lo que aplica a esta VM:

- Aislamiento: **Podman rootless con gVisor**, y un proxy de salida con lista blanca como único camino de red ([gVisor rootless](https://gvisor.dev/docs/user_guide/rootless/), [Podman](https://github.com/containers/podman)). Instalar Podman requiere sudo; tu usuario está en el grupo `sudo`. Queda por comprobar en el laboratorio el soporte rootless de gVisor, porque algunos flags son experimentales ([gvisor#12575](https://github.com/google/gvisor/issues/12575)).
- Credenciales: la API key de DeepSeek no va en el entorno del ejecutor; se inyecta en el proxy de salida. Para GitHub, GitHub App o PAT fine-grained de corta vida ([github.blog](https://github.blog/security/application-security/introducing-fine-grained-personal-access-tokens-for-github/)).
- Riesgos documentados: slopsquatting, con un 19,7 % de paquetes inventados en el estudio de USENIX 2025 ([Socket](https://socket.dev/blog/slopsquatting-how-ai-hallucinations-are-fueling-a-new-class-of-supply-chain-attacks)); MCP tool poisoning ([OWASP](https://owasp.org/www-community/attacks/MCP_Tool_Poisoning)); OWASP LLM01/LLM06 ([LLM Top 10](https://genai.owasp.org/llm-top-10/)).

## 5. Recomendaciones

1. **Esqueleto de arquitectura** (a validar juntos): Opus orquesta y revisa en su sesión de suscripción → especificación tipada con criterios de aceptación → ejecutor DeepSeek en un proceso aparte, dentro de worktree y sandbox → gates deterministas (tests vistos en rojo, cobertura, mutation, SAST/SCA/secretos) → revisión de Opus ejecutando, no solo leyendo → PR → CI → merge queue.
2. **Tres finalistas de ejecutor para pruebas:** Claude Code apuntado a DeepSeek, OpenCode y Pi. Aider en reserva.
3. **Dos modelos:** `deepseek-v4-pro` y `deepseek-flash`.
4. **Pruebas empíricas** con el mismo conjunto de tareas y tests ocultos, midiendo:
   - tasa de éxito
   - dólares de DeepSeek por tarea
   - `/usage` de Opus antes y después
   - tiempo
   - intervenciones de Opus
   - si el ejecutor ha tocado tests
   
   Con dos líneas base obligatorias: Opus solo y DeepSeek solo.
5. El sandbox (Podman con gVisor) se monta para las pruebas desde el principio, no después.

## 6. Dónde se ha buscado

- Documentación oficial: [Claude Code sub-agents](https://code.claude.com/docs/en/sub-agents), [sandboxing](https://code.claude.com/docs/en/sandboxing), [DeepSeek API](https://api-docs.deepseek.com/) (pricing, anthropic_api, kv_cache, rate_limit), [soporte de Claude](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-claude-max-or-claude-pro-plan), [OpenCode CLI](https://opencode.ai/docs/cli/), [OWASP GenAI](https://genai.owasp.org/llm-top-10/).
- GitHub con `gh` autenticado: métricas e issues de todos los repos citados.
- Hacker News (API de Algolia): decenas de consultas; los hilos leídos se listan en [[orquestacion-experiencia-comunidad]].
- Reddit por Chrome real (CDP): r/ClaudeCode, r/ClaudeAI, r/LocalLLaMA.
- Sin aporte: el leaderboard de Aider (desactualizado para modelos vigentes, [aider](https://aider.chat/docs/leaderboards/)), [swebench.com](https://www.swebench.com/) y [tbench.ai](https://www.tbench.ai/leaderboard) (tablas generadas por JavaScript, no extraídas), y un blog de Medium que devolvió 403. La lista completa de fuentes vacías está en cada informe.

## 7. Otros

- **Correcciones a los informes de los subagentes:**
  1. El endpoint Anthropic de DeepSeek **no** rompe las tools MCP locales, porque viajan como tools normales. Lo que ignora es el conector `mcp_servers` del lado del servidor.
  2. **No** pierde la caché, porque DeepSeek la aplica automáticamente ([Context Caching](https://api-docs.deepseek.com/guides/kv_cache)).
  
  Ambas correcciones están verificadas en la documentación oficial y aplicadas en los informes.
- Soberanía del dato con DeepSeek: el usuario la aceptó ([[decisiones]]); no condiciona la arquitectura.
- Pago de la API de DeepSeek: un comentario dice que solo admite Alipay/WeChat ([HN](https://news.ycombinator.com/item?id=48237663)); **sin verificar**. Se comprobará al crear la cuenta.
- Queda por verificar si `claude-code-action` admite el token de suscripción para que Opus revise en CI. Si no, la revisión de Opus se hace en local.
- Contexto humano: supervisar varios agentes en paralelo agota («air traffic control», [HN](https://news.ycombinator.com/item?id=47715217)). Hay que dimensionar el paralelismo con eso en mente.

## 8. Verificado en fuente primaria por el orquestador

`gh api` sobre Roo-Code (archivado), opencode, pi, aider, cline, kilocode y claude-code-router; estado de [#34821](https://github.com/anthropics/claude-code/issues/34821) y [#38698](https://github.com/anthropics/claude-code/issues/38698); documentación de DeepSeek (pricing, anthropic_api, kv_cache); README de DeepClaude; el artículo del benchmark de arneses; `opencode run` en la documentación de la CLI; los modos de Pi en su README; y las capacidades de la VM.

## Enlaces

- [[sistema-desarrollo-con-agentes]] — proyecto
- [[orquestacion-herramientas-y-patrones]] · [[orquestacion-modelos-y-costes]] · [[orquestacion-experiencia-comunidad]] · [[orquestacion-seguridad-ejecutor]]
- [[_index]]
