---
title: Sistemas de research con agentes
created: 2026-09-27
updated: 2026-09-27
tags: [research, herramientas, agentes, meta]
zona: tecnico
---

Qué sistema usar para investigar en profundidad con agentes, evaluado contra la restricción real de esta máquina. Conclusión: no hay producto que lo resuelva y ninguno de los que tienen mejor evidencia corre aquí.

## 1. Qué se pregunta y por qué

Se pedía la mejor herramienta —MCP, skill, framework o método— para coger un problema, segmentarlo y llegar a hallazgos verificados con fuentes, «lo mejor de lo mejor que esté comprobado». La pregunta se reformula antes de mirar candidatos: el criterio de admisión sale de la evidencia sobre dónde fallan, no de una lista de productos. Detalle de la decisión en [[decisiones]]; herramientas ya montadas en [[entorno]].

## 2. El criterio, investigado antes que los candidatos

El fallo dominante de esta categoría **no es encontrar: es que la cita no sostenga la frase**.

- **Dónde se introduce el error** — Hirsch et al., EMNLP 2026, [arXiv:2608.24306](https://arxiv.org/abs/2608.24306): el **84,7 % de los errores del informe final de AI-Q se originan en el orquestador**, no en el retrieval. Y *«casi todos los agentes cometen muchísimos errores, con la excepción de los que resumen un solo documento»*. Consecuencia directa: resumir documento a documento antes de sintetizar.
- **Cuánto falla la cita** — DeepTRACE, ICLR 2026 ([proceedings](https://proceedings.iclr.cc/paper_files/paper/2026/hash/ad08767706825033b99122332293033d-Abstract-Conference.html)): precisión de citas del **40-80 %** según sistema. «Más citas no mejora la fiabilidad».
- **Cuánto se fabrica** — GhostCite, [arXiv:2602.06718](https://arxiv.org/abs/2602.06718): **14,23 %–94,93 %** de fabricación entre 13 modelos; 1,07 % de 2,2 M de citas reales son inválidas, con **+80,9 % en 2025**. En biomedicina, [arXiv:2609.14988](https://arxiv.org/abs/2609.14988): **55,4 % fabricadas**, ningún modelo superó el 54,6 % correcto.
- **El caso real mejor documentado** — KPMG retiró en octubre de 2025 el informe *Total Experience*: GPTZero auditó **45 citas y solo 5 apuntaban a fuentes reales** ([indianexpress](https://indianexpress.com/article/technology/artificial-intelligence/kpmg-retracts-ai-study-hallucinations-fake-citations-10738768/lite/)). El término acuñado: *vibe citing*.
- **El techo medido** — TaxoBench, [arXiv:2601.12369](https://arxiv.org/abs/2601.12369): el mejor agente recupera el **20,92 %** de los papers que citan los expertos.

## 3. Considerado y descartado, con el motivo

**Descartados por hardware** — el filtro que decide. Esta máquina son **4 núcleos, 7 GB de RAM, sin GPU y sin Docker** ([[entorno]]), así que no corren aquí:

- **Tongyi DeepResearch** (Alibaba, 30B MoE) — 19.990★, el patrón de comparación universal, con la comunidad más grande (365 pts y 153 comentarios en [HN](https://news.ycombinator.com/item?id=45789602)). Repo parado desde febrero de 2026.
- **MiroThinker** — mejores números abiertos (BrowseComp 74,0; GAIA-165 82,7). En X, promoción pagada.
- **DR Tulu** (AI2) — el mejor aval académico: [arXiv:2511.19399](https://arxiv.org/abs/2511.19399), ICML 2026, 710★, supera a los abiertos en +15,6 % y alcanza a los propietarios en +0,7 %, 1000× más barato. **No está en el catálogo público de OpenRouter**, y el modelo de 8B necesita GPU.
- **Local Deep Research** — pide Docker y GPU. Funciona y aguanta 6 meses en producción con una RTX 3060, pero no aquí.
- **NVIDIA AI-Q** — mejor sistema abierto de DeepResearch Bench (**55,95**, por encima de OpenAI 46,45, Gemini 49,71 y Claude Research 45,00), pero su documentación dice «requires significant infra setup» y el estudio de EMNLP mide que su orquestador origina el 84,7 % de los errores finales.
- **AstaBench** (AI2) — benchmark, no agente; su README pide 10-30 GB de RAM. El agente de producción de Asta no es abierto.

**Descartados por falta de evidencia, aunque tengan estrellas**:

- **deer-flow** (ByteDance) — 83.005★, el más estrellado de la categoría, **sin un solo benchmark de terceros**. Su versión 2.0 es un «superagente» general; el research quedó en la rama `1.x`.
- **hyperresearch** — dice liderar DeepResearch Bench y **no aparece en el CSV público del leaderboard** ([comprobado](https://huggingface.co/spaces/muset-ai/DeepResearch-Bench-Leaderboard)).
- **STORM** (31.506★) — un año sin commits. **langchain open_deep_research** (12.680★) — archivado. **dzhng**, **nickscamara** — abandonados. **InternLM/MindSearch** — 14 meses parado.
- **OWL** — presumía de «Top 1 en GAIA» y su fundador admitió en HN que no lo habían medido todavía.
- **ROMA** (5.179★, [arXiv:2602.01848](https://arxiv.org/abs/2602.01848)) — el diseño que mejor encaja con la tesis (descomposición recursiva + Verificador), y mide **+31,4 puntos en SEAL-0** (evidencia contradictoria) frente a +11,1 en FRAMES (multi-hop simple). Descartado como herramienta: **sin licencia**, 3 contribuyentes, dormido desde febrero. Se queda como evidencia: dividir rinde casi el triple cuando hay que **reconciliar contradicciones**.

**Descartado como capa de conocimiento (no es búsqueda)**: `claude-obsidian` (15.231★, MIT, plugin de Claude Code) — aporta fuentes inmutables, registro de afirmaciones y lint, pero **la mayor parte ya está en este esquema**. Se toma la idea del registro de citas textuales y se escribe como regla, sin instalar código de terceros en un repo público con push horario. Motivo completo en [[decisiones]].

## 4. Recomendaciones

1. **Buscar**: `/investigar-web` con su escalado obligatorio a `/navegador-cdp` ([[entorno]]), y fuentes directas — API de GitHub, API de Algolia de HN, la fuente primaria. El deep research de otras IAs, solo para descubrir pistas.
2. **Verificar**: el `/deep-research` integrado de Claude Code sirve para su fase adversarial (3 votos, 2 refutaciones para matar). **No sirve para descubrir**: medido en vivo, 110 agentes y 4,2 M de tokens para encontrar tres repos de 102★, 0★ y 1★, perdiéndose los de 15.000 y 83.000.
3. **Guardar**: este wiki, documento a documento, aplicando la sección «Verificación de citas» de `AGENTS.md`.
4. **Literatura científica**: PaperQA2, el mejor avalado ([arXiv:2409.13740](https://arxiv.org/abs/2409.13740), motor del sistema publicado en *Nature*), con la pega de que el corpus de pago con el que logra sus números no es abierto.

## 5. Dónde se ha buscado

- **GitHub** (`gh api`): búsqueda por tema y por organización en ~30 orgs (allenai, InternLM, Tencent, antgroup, microsoft, google-research, SalesforceAIResearch, NVIDIA, inclusionAI, stepfun-ai, MoonshotAI, QwenLM, SakanaAI, ServiceNow, rlresearch…), y verificación individual de ~50 repos.
- **X**, con el navegador real por CDP: 19 búsquedas. **No sirve como fuente de validación en esta categoría**: las genéricas devuelven listículos y cuentas con etiqueta de *partnership* pagado; solo las hechas por nombre de paper o institución dieron respaldo verificable.
- **arXiv** (API y páginas de abstract), **Hacker News** (API de Algolia), **Hugging Face** (modelos, spaces y datasets), **npm y PyPI** (descargas), y el **CSV del leaderboard de DeepResearch Bench**.
- **Comunidad**: hilos de Hacker News (Tongyi 365 pts, Local Deep Research 190, Sakana AI-Scientist 203, smolagents 395) y de r/LocalLLaMA leídos por espejo — con una advertencia que vale para el futuro: **la auditoría de comunidad más citada de la categoría es falsa en un dato contable**. Atribuye 173 issues abiertos a `gpt-researcher` cuando la API de GitHub da **5**, con un 96,7 % de respuesta a los issues de 2026 ([hilo](https://www.reddit.com/r/LocalLLaMA/comments/1t4e83m/)).

## 6. Lo que no encaja arriba

- **Reddit es accesible de forma intermitente** por espejos (Redlib) y archivos históricos (PullPush); el acceso directo está bloqueado por la propia API de Anthropic ([issue #88941](https://github.com/anthropics/claude-code/issues/88941)).
- **Los benchmarks de esta categoría no son criterio de compra**: la mitad los escribe un vendor que sale bien parado (DRACO lo escribe Perplexity, el libro blanco de SciSpace lo escribe SciSpace), y el juez LLM mueve las puntuaciones absolutas 10-25 puntos.
- **Regla operativa que aparece en toda pila local**: las instancias de SearXNG se banean rápido; hay que autoalojarlas o rotar VPNs. Ver [[entorno]].
- **Lo que ninguna herramienta de esta comparativa resuelve**: que la investigación se quede en lo primero que cumple. Eso no es un problema de herramienta sino de método — [[metodo-de-investigacion]].
