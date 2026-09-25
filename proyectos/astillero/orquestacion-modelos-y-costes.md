---
title: Orquestación Opus/DeepSeek: modelos y costes
created: 2026-09-24
updated: 2026-09-24
tags: [agentes, deepseek, costes, benchmarks]
zona: tecnico
---

Modelos DeepSeek vigentes, precios, límites, benchmarks frente a Claude, límites del plan Pro y boceto de coste por tarea.

Informe de un frente de la investigación. Síntesis y decisiones en [[orquestacion-opus-deepseek-informe]].

Fecha de la investigación: 2026-09-24. Todo lo que sigue lleva su fuente en la misma línea; lo que no tiene enlace va marcado «sin verificar».

---

## 1. DeepSeek — API oficial hoy

### 1.1 Modelos vigentes (api-docs.deepseek.com)

DeepSeek retiró `deepseek-chat` y `deepseek-reasoner` el 2026-07-24 (apuntaban temporalmente a V4-Flash) y consolidó la gama en dos modelos únicos, cada uno con modo razonamiento activable: [Change Log](https://api-docs.deepseek.com/updates/)

| Modelo | ID API | Contexto | Output máx | Razonamiento | Notas |
|---|---|---|---|---|---|
| DeepSeek-V4.1-Flash | `deepseek-flash` | 1M tokens | 384K tokens | Soporta modo no-pensante y pensante (thinking, por defecto) | Release 2026-09-10, sustituye a V4-Flash/V4-Flash-Vision-Exp (nombres antiguos siguen aceptados, facturados como Flash). Multimodal nativo. [api-docs.deepseek.com/quick_start/pricing](https://api-docs.deepseek.com/quick_start/pricing) |
| DeepSeek-V4-Pro-0813 | `deepseek-v4-pro` | 1M tokens | 384K tokens | Soporta no-pensante y pensante; 3 niveles de esfuerzo: low/high/max | GA 2026-08-13. Formato Responses API (adaptado a Codex). [api-docs.deepseek.com/quick_start/pricing](https://api-docs.deepseek.com/quick_start/pricing) |

DeepSeek-V3.2 / V3.2-Speciale (lanzados 2025-12-01, 671B params / 37B activos, primer modelo con "thinking" integrado en tool-use) quedaron superados por la generación V4, lanzada 2026-04-24. [DeepSeek-V3.2 Release](https://api-docs.deepseek.com/news/news251201/) — [Change Log](https://api-docs.deepseek.com/updates/)

### 1.2 Precios por 1M tokens (off-peak / peak)

Peak: 01:00–04:00 y 06:00–10:00 UTC, lunes a viernes, excluyendo festivos chinos. Off-peak = mitad del precio peak. [api-docs.deepseek.com/quick_start/pricing](https://api-docs.deepseek.com/quick_start/pricing)

| Modelo | Input cache hit | Input cache miss | Output |
|---|---|---|---|
| `deepseek-flash` | $0.003 / $0.006 | $0.15 / $0.30 | $0.60 / $1.20 |
| `deepseek-v4-pro` | $0.022 / $0.044 | $0.66 / $1.32 | $1.98 / $3.96 |

Nota: cifras de terceros agregadores (pricepertoken.com, otros) muestran precios distintos para "deepseek-v3.2" ($0.214–$3.00/1M según proveedor de reventa) — corresponden a proveedores de inferencia de terceros, no a la API oficial de DeepSeek, y a un modelo ya retirado. Usar solo la tabla de api-docs.deepseek.com como referencia de coste real. [pricepertoken.com](https://pricepertoken.com/pricing-page/model/deepseek-deepseek-v3.2) (fuente de contraste, no autoritativa)

### 1.3 Compatibilidad de API

- **Formato OpenAI-compatible**: base URL `https://api.deepseek.com`. [api-docs.deepseek.com](https://api-docs.deepseek.com/)
- **Endpoint Anthropic-compatible oficial**: `https://api.deepseek.com/anthropic`. Se usa instalando el SDK/CLI de Anthropic y apuntando `ANTHROPIC_BASE_URL` a ese endpoint con `ANTHROPIC_API_KEY` = clave de DeepSeek. [api-docs.deepseek.com/guides/anthropic_api](https://api-docs.deepseek.com/guides/anthropic_api)
  - **Mapeo automático de modelos** (esto es clave para el diseño planteado): pedir `claude-opus` → se factura y ejecuta como `deepseek-v4-pro`; pedir `claude-haiku`/`claude-sonnet` → `deepseek-flash`. Cualquier nombre no reconocido cae a `deepseek-flash`. Es decir: **se puede apuntar Claude Code directamente contra DeepSeek** cambiando solo `ANTHROPIC_BASE_URL`/`ANTHROPIC_API_KEY`, sin tocar el harness. [api-docs.deepseek.com/guides/anthropic_api](https://api-docs.deepseek.com/guides/anthropic_api)
  - Soportado en ese endpoint: visión (base64/URL/ficheros), tool calls, modo thinking, salida JSON, streaming, multi-turno, Files API (con header beta `files-api-2025-04-14`).
  - No soportado: content types de "document" y "search result", resultados de code execution, el conector `mcp_servers` y `container` del lado del servidor (las tools MCP locales de Claude Code viajan como tools normales), `top_k`/`top_p` (salvo en thinking mode, mínimo 0.95). `budget_tokens` en thinking se ignora; casi todos los metadata excepto `user_id` se ignoran; `disable_parallel_tool_use` no se respeta.
  - DeepSeek documenta también integración "sin código" con Claude Code, GitHub Copilot y OpenCode como backend, remitiendo a una Agent Integrations Guide (`/quick_start/agent_integrations/claude_code`) cuyo contenido detallado no pude extraer del fetch (la página solo repite el base URL y los dos IDs de modelo). [api-docs.deepseek.com/guides/claude_code](https://api-docs.deepseek.com/guides/claude_code)

### 1.4 Function/tool calling

- Soportado en ambos modelos vía el formato compatible OpenAI/Anthropic. No encontré página oficial dedicada con cifras de fiabilidad (la guía `function_calling` de los docs no detalla límites ni tasa de error). [api-docs.deepseek.com/guides/function_calling](https://api-docs.deepseek.com/guides/function_calling) (página consultada, sin datos cuantitativos)
- Terceros: OpenRouter reporta que V4.1 Flash acepta `tools`/`tool_choice` en 24 de 25 proveedores que lo sirven, y soporta structured outputs vía `response_format` con JSON schema. [openrouter.ai/deepseek/deepseek-v4.1-flash](https://openrouter.ai/deepseek/deepseek-v4.1-flash)
- Test práctico (blog Kilo, ver §4): tool calling de V4 Flash "se sostuvo mejor de lo esperado", sin argumentos malformados ni bucles de reintento típicos de modelos baratos; V4 Pro tiene ventaja estadísticamente significativa en cumplimiento de instrucciones (step gating, pausas requeridas, formato, límites de fichero) frente a Flash. [blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash](https://blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash)

### 1.5 Rate limits

No son "requests por minuto" sino **límites de concurrencia** a nivel de cuenta: [api-docs.deepseek.com/quick_start/rate_limit](https://api-docs.deepseek.com/quick_start/rate_limit)

| Modelo | Conexiones concurrentes |
|---|---|
| `deepseek-flash` | 2,500 |
| `deepseek-v4-pro` | 500 |

Al superar el límite: HTTP 429. Ampliable gratis con "capacity expansion request" según necesidad de negocio. Con cuota ampliada, también se aplica el mismo límite por `user_id` individual. No documentan límite de tokens/minuto explícito en esta página.

### 1.6 Retención de datos / términos relevantes para código

- Política de privacidad: procesamiento y almacenamiento por defecto en China; se usa para entrenar/mejorar modelos por defecto, con opt-out. **Desde la actualización de marzo 2026, las cuentas de API de pago no se usan para entrenamiento por defecto** (opt-out disponible igualmente). [DeepSeek Privacy Policy](https://cdn.deepseek.com/policies/en-US/deepseek-privacy-policy.html) vía [meetily.ai resumen](https://meetily.ai/llm-privacy/deepseek) — el resumen de terceros no es la fuente primaria, pero la propia política confirma el procesamiento en RPC.
- No hay ventana de retención fija publicada: retienen "durante el tiempo necesario para prestar el servicio", lenguaje abierto que permite retención indefinida.
- Términos de la Open Platform (vigentes 2026-04-29) no dan excepción separada a los inputs de desarrollador: remiten a la política de privacidad general. [DeepSeek Open Platform Terms of Service](https://cdn.deepseek.com/policies/en-US/deepseek-open-platform-terms-of-service.html)
- **Implicación práctica para código propietario/cliente**: procesamiento en China, sin garantía de no-retención, sin ZDR (zero data retention) documentado a diferencia de Anthropic. Para un ingeniero DevSecOps esto es el punto duro de la decisión, no el coste.

---

## 2. Benchmarks de código: DeepSeek vs Claude

### 2.1 Aider Polyglot leaderboard

El fetch a `aider.chat/docs/leaderboards/` devolvió datos que parecen desactualizados (entradas de generación GPT-5/o3-pro/Claude-Opus-4-20250514, sin `deepseek-v4-pro` ni `deepseek-flash` ni Claude Opus 5/Sonnet 5). Lo dejo tal cual para no inventar cifras actuales:

| Modelo (entrada más reciente vista) | Pass rate | Coste del run |
|---|---|---|
| GPT-5 (high) | 88.0% | $29.08 |
| GPT-5 (medium) | 86.7% | $17.69 |
| O3-Pro (high) | 84.9% | $146.32 |
| DeepSeek-V3.2-Exp (Reasoner) | 74.2% | $1.30 |
| DeepSeek R1 (0528) | 71.4% | $4.80 |
| DeepSeek-V3.2-Exp (Chat) | 70.2% | $0.88 |
| Claude-Opus-4-20250514 (32k thinking) | 72.0% | $65.75 |
| Claude-Opus-4-20250514 (no think) | 70.7% | $68.63 |
| Claude-Sonnet-4-20250514 (32k thinking) | 61.3% | $26.58 |

Combo architect/editor documentado: **O3 (high) + GPT-4.1** = 78.2%, $17.55. **DeepSeek R1 + Claude-3-5-Sonnet** = 64.0%, $13.29 — este último es la única pareja "modelo caro planifica / modelo barato edita" que aparece en la tabla, y va en la dirección contraria a la que plantea el diseño (aquí el barato planifica y el caro edita). [aider.chat/docs/leaderboards/](https://aider.chat/docs/leaderboards/)

**No hay entradas Aider Polyglot para DeepSeek V4/V4.1-Flash ni Claude Opus 5/Sonnet 5/Fable 5 a fecha de esta consulta** — o el leaderboard no se ha actualizado con los modelos vigentes, o mi fetch cayó en una versión cacheada. Pendiente de verificación directa (abrir la página en navegador, no permitido en esta tarea).

### 2.2 SWE-bench Verified

- Fuente vendor (Fireworks.ai, que aloja inferencia de DeepSeek — tiene interés comercial en el resultado): **DeepSeek V4 Pro 95.2%** en SWE-bench Verified (500 tareas) vs **Claude Fable 5 85.4%**. Coste por tarea resuelta: DeepSeek $0.309 vs Fable 5 $0.808 (~3x más barato). En LiveCodeBench la brecha se amplía: $0.040 vs $0.225. [fireworks.ai/blog/DeepSeekV4Pro-Fable5](https://fireworks.ai/blog/DeepSeekV4Pro-Fable5)
  - **Importante**: la comparación es contra **Claude Fable 5** (modelo insignia de Anthropic, $10/$50 por MTok — tier por encima de Opus), no contra Claude Opus 5 ($5/$25) ni Sonnet 5 ($2/$10). No encontré cifra SWE-bench Verified oficial/neutral de Opus 5 o Sonnet 5 para comparar en igualdad de condiciones — esta comparación concreta favorece a DeepSeek en la métrica de coste por partida doble (modelo más barato vs el modelo más caro del rival).
  - El mismo post reconoce debilidad de V4 Pro en Java: 48.9% vs 74.5% de Fable 5 en la porción Java de Aider Polyglot — relevante si el trabajo real tiene stack Java/enterprise.
  - Con "oracle routing" (enrutado post-hoc, límite superior no desplegable) combinando ambos modelos: 92.4% a $0.279/tarea vs $4.510 de Fable-solo — el propio artículo advierte que un router real "captura solo una fracción" de esa cifra ideal.
- **Corrección 2026-09-25, resuelto por el orquestador**: no era una contradicción entre fuentes sobre el mismo modelo. El orquestador descargó `swebench.com` directamente (no está solo renderizado por JS: el HTML trae los datos embebidos) — el leaderboard oficial **no tiene ni una entrada de V4, V4 Pro, Opus 5.5, Sonnet 5 ni Fable**, y no se actualiza desde el 1 de septiembre de 2026. Rastreado el origen de cada cifra: 80,6% es **DeepSeek V4 base** (24-abr-2026, citado en HN sobre el anuncio oficial de DeepSeek); 95,2% es **DeepSeek V4 Pro 0813** (GA 13-ago-2026, cuatro meses después), medido por Fireworks con harness propio, sin envío al leaderboard oficial. Son dos modelos distintos, ninguno verificable en la fuente que se supone que mide esto. Detalle en [[flujo-agentes-evidencia-empirica]] §2.

### 2.3 Terminal-Bench

- Terminal-Bench 2.1: **DeepSeek-V4.1-Flash lidera con 0.906**, entre los más baratos con score dentro del 10% del líder ($0.06/1M input, DeepSeek-V4-Flash-0731, 0.827). Claude Code (agente, no se especifica qué modelo exacto en la cita) 0.880. [codingfleet.com/blog/terminal-bench-leaderboard-2026](https://codingfleet.com/blog/terminal-bench-leaderboard-2026/) — agregador de terceros, sin acceso directo confirmado a tbench.ai (JS-rendered, fetch no extrajo tabla: [tbench.ai/leaderboard](https://www.tbench.ai/leaderboard)).
- Terminal-Bench 4.0 (más difícil, lanzado 1–2 sept 2026): Codex + GPT-6 Astra lidera con 58.2%; **Claude Code + Fable 5.1: 57.9% al doble de coste por run**; Claude Fable 5.1 solo: 57.9% ± 3.8; **Opus 5: 51.8% ± 3.4**. No aparece DeepSeek V4/V4.1-Flash en el conjunto TB4.0 citado — sugiere que en la variante más dura (agentic, terminal real) DeepSeek aún no tiene entrada pública, o cae fuera del top citado por el agregador. [codingfleet.com/blog/terminal-bench-leaderboard-2026](https://codingfleet.com/blog/terminal-bench-leaderboard-2026/) — sin verificar contra la fuente primaria.

### 2.4 LiveCodeBench

- Cifras contradictorias entre agregadores: uno sitúa **DeepSeek-V4-Pro-Max en 0.935 (líder)**; otro (fecha 25-ago-2026) sitúa el líder en Gemini 3 Pro Preview 91.7%, con **DeepSeek V3.2 Speciale en 89.6%** (tercero) y sin mencionar V4-Pro-Max en el top. No pude reconciliar ambas cifras con una fuente primaria única de LiveCodeBench. Claude Opus 5 no aparece con cifra LiveCodeBench específica en las búsquedas — solo una cifra "coding-arena" (26.7) de significado distinto no comparable. **Marcar todo este apartado como sin verificar de forma fiable**, solo agregadores de terceros. [llm-stats.com/benchmarks/livecodebench](https://llm-stats.com/benchmarks/livecodebench)

### 2.5 Test cualitativo independiente (Kilo.ai)

Metodología: mismo spec (backend de orquestación de workflows, 20 endpoints, estado persistente, leases, reintentos, streaming de eventos) dado a 4 modelos, puntuado 0-100 por un evaluador. [blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash](https://blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash)

| Modelo | Score | Coste del run completo |
|---|---|---|
| Claude Opus 4.7 | 91/100 | — (referencia: output ~89x el de V4 Flash) |
| DeepSeek V4 Pro | 77/100 | — |
| Kimi K2.6 | 68/100 | — |
| DeepSeek V4 Flash | 60/100 | $0.02 |

- Coste por punto: V4 Flash ~30x más barato que Kimi K2.6, ~100x más barato que Opus 4.7.
- Fallos de V4 Pro: leases expirados no bloquean workers que igual completan el paso; el cap de paralelismo bloquea trabajo no relacionado; el build de TypeScript falla pese a que los tests pasan.
- Fallos de V4 Flash: endpoint crítico montado en ruta incorrecta (404); workflows fallidos siguen repartiendo trabajo a los pasos restantes; mismo bug de lease expirado que Pro; validación rechaza arrays JSON válidos.
- Conclusión del propio test: Opus 4.7 mantiene ventaja clara en lógica de timing/recuperación compleja; V4 Pro supera a Kimi K2.6 a igual o menor coste; V4 Flash es "viable para primeros intentos con revisión humana", no para producción sin repaso.

---

## 3. Claude Pro — límites de Claude Code y precios API

### 3.1 Documentación oficial (support.claude.com / claude.com)

- Pro y Claude Code **comparten el mismo pool de uso** ("shares the same usage limits as the rest of your plan — your work in the terminal and your chats draw from one pool"). [claude.com/pricing](https://claude.com/pricing)
- Estructura de límites: ventana de sesión de **5 horas** (rolling) + **límite semanal**, visibles en Settings > Usage. **Los documentos oficiales que pude leer NO publican cifras numéricas exactas** (nº de prompts, nº de horas, tokens) — ni la página de "usage limit best practices" ni la de "using Claude Code with your Pro/Max plan" las dan. [support.claude.com — usage limit best practices](https://support.claude.com/en/articles/9797557-usage-limit-best-practices), [support.claude.com — Claude Code con Pro/Max](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-claude-max-or-claude-pro-plan)
- Precio Pro: **$17/mes facturación anual, $20/mes mensual**. [claude.com/pricing](https://claude.com/pricing)
- **Opus SÍ está incluido en Pro** a fecha de esta consulta (Opus 5.5 disponible para Pro/Max/Team/Enterprise, incluido fast mode en Claude Code) — hubo confusión documental anterior en 2026 que Anthropic corrigió; no hay plan que excluya Opus del todo, aunque hubo un hilo de GitHub Issues quejándose de acceso inconsistente. [búsqueda con fuentes oficiales citadas: anthropic.com/claude/opus, support.anthropic.com] — [GitHub issue #58059](https://github.com/anthropics/claude-code/issues/58059) (queja de usuario, no fuente autoritativa, pero confirma que hubo fricción real)
- Si se agota el límite del plan, Anthropic ofrece créditos API a "standard API rates" opcionales, o recomienda subir a Max 5x. [support.claude.com/en/articles/11145838](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-claude-max-or-claude-pro-plan)

### 3.2 Estimaciones de terceros (sin confirmación oficial numérica — usar con cautela)

- Opus consume el límite semanal ~5x más rápido que Sonnet por ser más intensivo computacionalmente (cifra de blog de terceros, no de Anthropic). [búsqueda agregada, sin URL única fiable — sin verificar]
- Pro: aproximadamente 45 prompts por ventana de 5 horas y "40-80 horas Sonnet/semana" citado por blogs de terceros — **cifras no confirmadas en fuente oficial**, tratarlas como orden de magnitud, no como dato duro.
- El 13/14-sept-2026 terminó una promoción de +50% de límite semanal; desde entonces el límite estándar quedó fijado permanentemente un 25% por encima del nivel pre-promoción. El 22-sept-2026 Anthropic subió también los límites de 5 horas en Pro/Max/Team. [búsqueda agregada citando cambios recientes, sin URL de anuncio oficial de Anthropic localizada en esta sesión — sin verificar la fuente primaria del anuncio]

### 3.3 Precios API actuales (referencia de coste, Claude API — vía skill `claude-api` de este entorno y claude.com/pricing)

| Modelo | Input $/1M | Output $/1M | Cache read $/1M | Contexto |
|---|---|---|---|---|
| Claude Opus 5 (`claude-opus-5`) | $5.00 | $25.00 | — | 1M |
| Claude Opus 5.5 (`claude-opus-5-5`, lanzamiento — solo si se nombra explícitamente) | $4.00 | $20.00 | $0.20 | 1M |
| Claude Sonnet 5 (`claude-sonnet-5`) | $2.00 | $10.00 | $0.20 | 1M |
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | 200K |
| Claude Fable 5 / 5.1 (tier superior a Opus) | $10.00 | $50.00 | — | 1M |

Nota: estos precios son de la API de pago (no del plan Pro/Max, que es de cuota fija mensual). Comparar DeepSeek (API de pago, $0.15–$3.96/1M según modelo/dirección/peak-offpeak) contra Claude vía **plan Pro** (cuota fija, sin coste marginal por token dentro del límite) — son dos modelos de precio distintos y no directamente comparables en $/token; la comparación real debe hacerse en $/tarea completada dentro de los límites del plan Pro.

---

## 4. Análisis y experiencias reales con setups híbridos (barato codea / caro revisa)

### 4.1 Cuándo sí compensó

- **Verificación loop 4x DeepSeek** (Ironbee, blog + hilo HN de 288 comentarios, 729 puntos): un loop de verificación (skills de browser-testing, debug, visual-testing en bucle) llevó a DeepSeek de ~20 a ~80 en "Web-Bench", igualando a Opus **a ~1/7 del coste**. Comentario de HN sin respuesta: cuánto más tarda el loop 4x de DeepSeek frente a Opus directo — **el ahorro en $ no incluye el coste en tiempo/latencia**, dato ausente. [ironbee.medium.com](https://ironbee.medium.com/what-a-verification-loop-adds-to-a-coding-agent-a-first-look-5049017e636e) — [hilo HN](https://news.ycombinator.com/item?id=48817519)
- **DeepSeek Reasonix** (HN, 729 puntos, 288 comentarios): agente de código nativo de DeepSeek con caching agresivo y coste bajo — indica que hay tracción real de la comunidad en usar DeepSeek como motor de codificación de agentes, no solo chat. [esengine.github.io/DeepSeek-Reasonix](https://esengine.github.io/DeepSeek-Reasonix/) — [hilo HN](https://news.ycombinator.com/item?id=48256953)
- Comentario de usuario real en el hilo del anuncio de precio permanente de V4 Pro: 65M tokens procesados por **$1.5 total**, considerado "mejor que GLM 5.1 para tareas de código complejas" tras 3 semanas de uso. [hilo HN — comentario de Sphax](https://news.ycombinator.com/item?id=48237663)
- DeepSeek usado específicamente **como "segunda opinión"/revisor secundario de código** ya generado por otro modelo — "a veces detecta cosas que otros modelos no ven" (comentario HN, uso exactamente al revés del planteado por el usuario: barato revisa, no solo codea). [hilo HN — comentario de Havoc](https://news.ycombinator.com/item?id=48237663)

### 4.2 Cuándo NO compensó / escepticismo medido

- Hilo "Smart model routing directly in Claude, Codex and Cursor" (216 puntos, 113 comentarios) — el router es la instancia más cercana al diseño planteado (orquestador que decide qué modelo ejecuta) y la crítica de la comunidad es directa:
  - **Pérdida de cache**: cambiar de modelo entre turnos invalida el prompt cache (TTL 5 min); varios comentaristas dudan de que el ahorro por modelo barato compense el re-procesamiento sin cache. [comentario stpedgwdgfhgdd / ai_slop_hater](https://news.ycombinator.com/item?id=48688700)
  - **Modelos pequeños paran antes de completar, producen errores y bucles** — riesgo no cuantificado en el diseño del router, según _pdp_. [hilo HN](https://news.ycombinator.com/item?id=48688700)
  - Un comentario nota que routing por API pricing puede salir **más caro** que usar directamente la suscripción de Claude Code o Codex CLI, si el patrón de uso ya cabía dentro de los límites de plan fijo. [comentario emilio_srg2](https://news.ycombinator.com/item?id=48688700)
- En el hilo de precios de DeepSeek, comentario relevante sobre el techo de la comparación: **"incluso a estos precios, las suscripciones de Claude y Codex me salen más baratas que el pricing por token cuando mi uso ronda los límites de sesión — supongo que las suscripciones están fuertemente subvencionadas"** (guelo). Esto es exactamente el caso de uso del usuario (Pro de 20€, orquestación con Opus dentro de cuota) — sugiere que **mezclar Opus-por-plan-fijo con DeepSeek-por-token puede no ahorrar tanto como parece**, porque el coste marginal de Opus dentro del plan Pro ya es ~0 hasta el límite. [hilo HN](https://news.ycombinator.com/item?id=48237663)
- Un comentario (zftnb666) menciona que el pago de la API de DeepSeek solo admite Alipay/WeChat y sugiere un intermediario de terceros ("api-hub.cc") para pagar con tarjeta — **esto es una afirmación de un comentario de HN sin verificación oficial, y el uso de un proxy de terceros no oficial introduce un riesgo de seguridad/datos adicional que contradice el perfil DevSecOps del usuario**; no lo tomes como recomendación, solo como dato de que el pago internacional de DeepSeek puede tener fricción. [hilo HN](https://news.ycombinator.com/item?id=48237663) — sin verificar si DeepSeek admite tarjetas internacionales directamente en api-docs.deepseek.com (no comprobado en esta sesión).

---

## 5. Modelo de coste aproximado (boceto, con supuestos explícitos)

**No hay datos públicos de "coste por tarea típica" con esta combinación exacta (DeepSeek codificando + Opus revisando)** — construyo una estimación a partir de piezas verificadas, marcando cada supuesto.

Supuestos:
- Tarea típica de codificación agéntica: ~40K tokens de contexto leído por turno de código (repo + instrucciones), ~4K tokens de output de código por turno, ~6 turnos de iteración con DeepSeek antes de pasar a revisión — **supuesto, no medido**, basado en orden de magnitud del test Kilo.ai (§2.5) donde una spec completa de 20 endpoints costó $0.02 con V4 Flash.
- Revisión de Opus: 1 pasada de revisión de ~40K tokens de contexto (diff + repo) + 2K tokens de output de comentarios — dentro de la cuota fija del plan Pro (20€/mes), coste marginal ≈ $0 mientras no se supere el límite semanal (cifra exacta de ese límite no publicada oficialmente, ver §3.2).

| Componente | Cálculo | Coste estimado |
|---|---|---|
| Codificación con `deepseek-v4-pro` (thinking, peak) | 6 turnos × (40K input miss $0.66/1M + 4K output $3.96/1M) | 6 × ($0.0264 + $0.0158) ≈ **$0.25/tarea** |
| Codificación con `deepseek-flash` (thinking, peak) — alternativa más barata | 6 turnos × (40K input $0.30/1M + 4K output $1.20/1M) | 6 × ($0.012 + $0.0048) ≈ **$0.10/tarea** |
| Revisión con Opus vía plan Pro (20€/mes) | Dentro de cuota, sin coste marginal hasta el límite semanal (no publicado) | **≈$0 marginal, pero limitado por el pool compartido Pro** |
| Revisión equivalente vía API Opus 5 de pago (referencia, fuera de plan) | 40K input × $5/1M + 2K output × $25/1M | $0.20 + $0.05 = **$0.25/tarea** |

**Coste total estimado por tarea**: ~$0.10–$0.25 en DeepSeek + ~$0 marginal en Opus-vía-Pro (mientras no se agote la cuota) → del orden de **decenas de céntimos por tarea completa**, muy por debajo de usar Opus de punta a punta vía API ($0.25 solo de "escribir" + $0.25 de revisar ≈ $0.50, y eso sin contar que escribir código real suele tomar más turnos que revisar).

**Lo que este boceto NO captura, y que la evidencia de §4 sí señala como riesgo real**:
1. El límite semanal compartido de Pro (Opus consume ~5x más rápido que Sonnet según terceros, §3.2) puede agotarse antes de lo esperado si Opus revisa con frecuencia — no hay cifra oficial para calcular el punto de quiebre.
2. Pérdida de cache al alternar entre DeepSeek y Claude en el mismo flujo (si se usa el mismo agente/sesión) — no aplica si son dos herramientas separadas (Claude Code para orquestar/revisar, llamada aparte a DeepSeek para codificar), que es como está planteado el diseño, así que este riesgo es menor aquí que en el caso "router automático" de §4.2.
3. Turnos extra por errores/bucles de DeepSeek en tareas complejas (Java, lógica de timing/recuperación, ver §2.5) pueden multiplicar el número de turnos de codificación varias veces, borrando la ventaja de precio unitario.
4. Retención de datos en China sin ZDR — coste no monetario pero relevante para el criterio DevSecOps del usuario (ver §1.6).

---

## 6. Fuentes consultadas

Oficiales:
- https://api-docs.deepseek.com/quick_start/pricing
- https://api-docs.deepseek.com/ (overview)
- https://api-docs.deepseek.com/updates/ (changelog)
- https://api-docs.deepseek.com/guides/anthropic_api
- https://api-docs.deepseek.com/quick_start/rate_limit
- https://api-docs.deepseek.com/guides/function_calling
- https://api-docs.deepseek.com/guides/claude_code
- https://api-docs.deepseek.com/news/news251201/
- https://cdn.deepseek.com/policies/en-US/deepseek-privacy-policy.html
- https://cdn.deepseek.com/policies/en-US/deepseek-open-platform-terms-of-service.html
- https://support.claude.com/en/articles/11145838-using-claude-code-with-your-claude-max-or-claude-pro-plan
- https://support.claude.com/en/articles/9797557-usage-limit-best-practices
- https://claude.com/pricing (redirigido desde anthropic.com/pricing)
- Skill interna `claude-api` de este entorno (tabla de precios API cacheada 2026-06-24, contrastada con claude.com/pricing en vivo)

Benchmarks / terceros:
- https://blog.kilo.ai/p/we-tested-deepseek-v4-pro-and-flash
- https://aider.chat/docs/leaderboards/ (posiblemente desactualizado, ver §2.1)
- https://fireworks.ai/blog/DeepSeekV4Pro-Fable5 (fuente con interés comercial — aloja DeepSeek)
- https://www.swebench.com/ (leaderboard oficial, tabla no extraíble por render JS)
- https://www.tbench.ai/leaderboard (leaderboard oficial, tabla no extraíble por render JS)
- https://codingfleet.com/blog/terminal-bench-leaderboard-2026/
- https://codingfleet.com/blog/swe-bench-pro-leaderboard-2026/
- https://llm-stats.com/benchmarks/livecodebench
- https://openrouter.ai/deepseek/deepseek-v4.1-flash
- https://pricepertoken.com/pricing-page/model/deepseek-deepseek-v3.2 (contraste, no autoritativa)
- https://github.com/anthropics/claude-code/issues/58059

Hacker News (vía hn.algolia.com API):
- https://news.ycombinator.com/item?id=48256953 (DeepSeek Reasonix, 729 pts)
- https://news.ycombinator.com/item?id=48237663 (DeepSeek V4 Pro precio permanente, 621 pts — hilo con anécdotas de coste real)
- https://news.ycombinator.com/item?id=48817519 (verification loop 4x DeepSeek, 39 pts)
- https://news.ycombinator.com/item?id=48688700 (smart model routing, 216 pts — hilo con escepticismo sobre ahorro real)
- Búsquedas sin resultado útil: "DeepSeek Claude Code hybrid" (tags=story) — solo devolvió un resultado irrelevante.

---

## 7. Preguntas abiertas / pendientes de verificar

1. **Cifras exactas de límites Pro (5h / semanal) en horas o mensajes** — Anthropic no las publica en soporte oficial. Solo hay estimaciones de terceros no confirmadas (§3.2). Recomendación: medir empíricamente con `/usage` en la propia cuenta antes de dimensionar el diseño.
2. **Aider Polyglot con modelos vigentes** (DeepSeek V4/V4.1-Flash, Claude Opus 5/Sonnet 5/Fable 5) — no localizado; requiere reconsulta directa de la página (posible caché) o del CSV/JSON del repo `aider-AI/aider` en GitHub (no consultado, `gh` no se usó para este dato).
3. **Resuelto 2026-09-25** (ver [[flujo-agentes-evidencia-empirica]] §2): no era la misma variante de modelo. 80,6% = V4 base; 95,2% = V4 Pro 0813, cuatro meses después, medido solo por un vendor con interés comercial. Ninguna cifra está en el leaderboard oficial, que lleva 7 meses sin actualizarse.
4. **LiveCodeBench**: cifras de terceros contradictorias y sin cifra de Claude Opus 5 directamente comparable.
5. **Terminal-Bench**: tabla oficial no extraída (JS), solo vía agregador de terceros — pendiente de reconsulta con herramienta capaz de renderizar JS (fuera del alcance permitido en esta tarea, que excluye navegador/Playwright).
6. **Pago internacional de la API DeepSeek con tarjeta** (fuera de Alipay/WeChat) — mencionado en un comentario de HN sin verificar contra `api-docs.deepseek.com`.

## Enlaces

- [[orquestacion-opus-deepseek-informe]] — síntesis de la investigación y arquitecturas candidatas
- [[astillero]] — proyecto al que pertenece
- [[_index]]
