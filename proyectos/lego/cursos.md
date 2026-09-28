---
title: Lego — cursos y formación que enseñan a montarlo
created: 2026-09-28
updated: 2026-09-28
tags: [lego, cursos, formacion, practico]
zona: tecnico
---

Material didáctico **con enlace directo**, en castellano y en inglés, que enseña a construir esto —no a discutirlo. Verificado: duración, número de lecciones y precio.

## La plataforma: Claude Academy

**`https://academy.claude.com`** — es la plataforma de formación de Anthropic, **gratuita**, con **insignia de finalización** en la mayoría de cursos. **Y tiene versión en castellano: `https://academy.claude.com/es/courses`** — la misma URL con `/es/` delante.

**Cursos del fabricante, en castellano, con enlace directo:**

| Curso | Lecciones | Duración | Enlace |
|---|---|---|---|
| **Claude Code en acción** ⭐ | 9 | 1 h | [academy.claude.com/es/courses/claude-code-in-action](https://academy.claude.com/es/courses/claude-code-in-action) |
| **La guía del SDLC nativo de IA** ⭐ | 14 | 1 h | [academy.claude.com/es/courses/ai-native-sdlc-playbook](https://academy.claude.com/es/courses/ai-native-sdlc-playbook) |
| Claude Code 101 | 12 | 1,5 h | [academy.claude.com/es/courses/claude-code-101](https://academy.claude.com/es/courses/claude-code-101) |
| Introducción a los subagentes | 4 | 45 min | [academy.claude.com/es/courses/introduction-to-subagents](https://academy.claude.com/es/courses/introduction-to-subagents) |
| Introducción a las habilidades de agente | 6 | 1 h | [academy.claude.com/es/courses/introduction-to-agent-skills](https://academy.claude.com/es/courses/introduction-to-agent-skills) |
| Introducción al Model Context Protocol | 10 | 1 h | [academy.claude.com/es/courses/introduction-to-model-context-protocol](https://academy.claude.com/es/courses/introduction-to-model-context-protocol) |
| MCP: Temas avanzados | 11 | 1,5 h | [academy.claude.com/es/courses/model-context-protocol-advanced-topics](https://academy.claude.com/es/courses/model-context-protocol-advanced-topics) |
| Construir equipos humano-agente efectivos (beta) | 5 | 45 min | [academy.claude.com/es/courses/building-effective-human-agent-teams](https://academy.claude.com/es/courses/building-effective-human-agent-teams) |
| Para desarrolladores (AI Fluency) | 9 | — | [academy.claude.com/es/courses/ai-fluency-for-builders](https://academy.claude.com/es/courses/ai-fluency-for-builders) |
| Construyendo con la API de Claude | 67 | 9 h | [academy.claude.com/es/courses/building-with-the-claude-api](https://academy.claude.com/es/courses/building-with-the-claude-api) |

**Los dos que importan para Lego son los dos primeros**, y el primero es **exactamente el tema**:

### «Claude Code en acción», y su programa

Su descripción oficial: *«Ejecuta sesiones largas de Claude Code sin supervisión en las que puedas confiar: dirige, configura, automatiza y verifica»*. Y su objetivo declarado: *«poder apuntar a Claude a horas de trabajo, irte, y comprobar lo que hizo con confianza»*.

**El programa, y fíjate en que es punto por punto el informe:**

| Módulo | Lecciones |
|---|---|
| **Dirigir el trabajo** | Dirigir sesiones largas; modo plan; dirigir la compactación |
| **Configurar** | **Un `CLAUDE.md` que se cumple** · **Skills de verificación** · **Modos de permiso** · **Hooks** |
| **Automatizar** | **Rutinas y modo sin interfaz** · **GitHub Actions y revisión de código** |
| **Verificar y compartir** | **«Confía en ello: verificar ejecuciones no supervisadas»** · Plugins |

Y las frases del temario que **dan la razón a lo que quedó en pie tras la auditoría**:

- *«escribir un `CLAUDE.md` **escueto que Claude siga de verdad**»* → el informe llegó a lo mismo por otra vía: las reglas que no se cumplen no son reglas.
- *«**imponer las reglas no negociables con hooks**»* → **exactamente** la conclusión de que la prosa no es una restricción y lo que obliga es el entorno.
- *«**verificar las ejecuciones no supervisadas en proporción a lo poco que miraste**»* → el principio que faltaba, y es mejor que cualquiera de los míos.
- *«**cerrar los turnos con resultados de tests reales**, con hooks»* → la verificación como puerta.

**Es gratis, son 9 lecciones, dura una hora, y está en castellano.** No he encontrado nada mejor en toda la investigación.

### «La guía del SDLC nativo de IA»

14 lecciones, una hora. **SDLC** son las siglas en inglés del ciclo de vida del desarrollo de software, es decir: **cómo se organiza de principio a fin el trabajo con agentes**. Es la versión oficial del «ciclo de vida» que el material del curso que me diste llamaba desarrollo dirigido por especificación.

## La otra fuente en castellano

**El curso de BIG school** que ya tenía —*Curso de Desarrollo con IA, Programa con agentes*—, en tres días: fundamentos, el arnés (`AGENTS.md`, MCP, skills), y **el tercer día entero dedicado al desarrollo dirigido por especificación**, con su ciclo, sus tres niveles y la estructura de artefactos. Está en [[material-practico]] con el detalle.

## Verificación de los enlaces

- **La plataforma responde y la versión en castellano existe**: `academy.claude.com/es/courses` carga el catálogo completo traducido, con duraciones e insignias.
- **Los cursos son gratis**: la inscripción en la versión inglesa dice literalmente `Register | FREE`, y la web declara «cursos, tutoriales y casos de uso gratuitos». **Requiere cuenta de Skilljar, no de Anthropic.**
- **`anthropic.com/learn` redirige** a `academy.claude.com` (redirección 308): si guardas el enlace antiguo, funciona igual.

## Lo que queda abierto

**No he podido confirmar si los cursos están subtitulados o doblados al castellano o sólo traducidos por escrito.** La página está en castellano; el idioma del audio y de los subtítulos no lo declara. **Sin verificar.**

**Y falta el resto del encargo**, que sigue buscándose en tres frentes: vídeos y canales de YouTube que lo demuestren, implementaciones reales con su repositorio, y las comunidades donde se comparte. Ya ha aparecido un curso en castellano de pago en Udemy, pendiente de verificar.

## Lo mejor que hay para APRENDER A CONSTRUIRLO (no sólo a usarlo)

Esto es lo que faltaba, y es lo más valioso de todo el material:

| Recurso | Qué es | Por qué destaca |
|---|---|---|
| **[shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)** | **17 lecciones**, curso de 0 a 1 de *ingeniería del arnés* en Python | **77.742★ y 12.500 forks.** Cada capítulo añade **un mecanismo**: bucle, despacho de herramientas, permisos, ganchos, planificar-y-ejecutar, subagentes, skills, compactación, memoria, **grafo de tareas en disco**, cron, equipos de agentes, MCP, y **bucle dirigido por objetivo con evaluador independiente**. Es la respuesta a «cómo se monta», lección por lección |
| **[decodingai/building-a-coding-agent-from-scratch](https://github.com/decodingai-magazine/building-a-coding-agent-from-scratch-course)** | Curso gratis (Apache-2.0), 8 artículos y 4 vídeos | **486★.** Construye un agente estilo Claude Code **entero en Python**: bucle → herramientas → permisos → **entorno aislado** → memoria → compactación → subagentes → **evaluaciones**. Termina con dos modos: interfaz interactiva y funciones sin servidor |
| **[ghuntley/how-to-build-a-coding-agent](https://github.com/ghuntley/how-to-build-a-coding-agent)** | Taller escrito: agente **en Go** por incrementos | **5.851★.** Del chat simple a lectura, listado, bash, edición y búsqueda con ripgrep. *«300 líneas de código en un bucle»*. **El autor es el creador del patrón Ralph** |
| **[snarktank/ralph](https://github.com/snarktank/ralph)** | La implementación del bucle que **repite hasta completar todos los ítems de un PRD**, con instancia limpia cada vuelta | **21.872★.** No es un curso: es el artefacto, y se aprende leyéndolo |
| **[Panaversity — Loop Engineering: Crash Course](https://agentfactory.panaversity.org/docs/loop-engineering-crash-course)** | Curso gratis (~2 h) con simulaciones y laboratorios | **Seis piezas, cuatro tipos de latido, y —lo que importa— condiciones de parada y puertas humanas** |
| [Microsoft Learn — Implement SDD using GitHub Spec Kit](https://learn.microsoft.com/es-es/training/modules/spec-driven-development-github-spec-kit-enterprise-developers/) | **13 unidades con ejercicio**, en español, gratis | Constitución, spec, plan, tareas, implementación, escalado y CI/CD. Escenario empresarial sobre código existente |
| [DeepLearning.AI — Spec-Driven Development with Coding Agents](https://www.deeplearning.ai/courses/spec-driven-development-with-coding-agents) | 1 h 16 m, 15 vídeos. Paul Everitt (JetBrains) | Constitución → spec → implementación → validación → **replanificación** → segunda feature. Genera specs desde documentación heredada y **empaqueta el flujo como skill** |

**DATO:** *«todo el material bueno sobre bucles autónomos tiene menos de un año»* y **«lo mejor documentado no son cursos, son repos»** — los artefactos con más señal son implementaciones legibles, no currículos.

## Documentación oficial en castellano (esto no lo tenía)

**DATO — la documentación de Claude Code está traducida al español de verdad**, no con traductor automático: `code.claude.com/docs/es/overview` funciona, y en las 12 páginas probadas la gemela en español devuelve 200.

- [Subagentes](https://code.claude.com/docs/es/sub-agents) · [Ganchos](https://code.claude.com/docs/es/hooks) · [Skills](https://code.claude.com/docs/es/skills) · [Modo sin interfaz](https://code.claude.com/docs/es/headless) · [Agent SDK](https://code.claude.com/docs/es/agent-sdk/overview)

**Y el manual que faltaba para montarlo uno mismo:** [Agent SDK — cómo funciona el bucle](https://code.claude.com/docs/en/agent-sdk/agent-loop), [herramientas propias](https://code.claude.com/docs/en/agent-sdk/custom-tools), sesiones, permisos y coste. **Aquí está el CÓMO de construir un agente propio**, no de usar el de otro.

**DATO:** **Anthropic es el único fabricante con traducción real de la documentación de su agente de código.** No existe documentación oficial de OpenAI Codex en español.

## Cursos en castellano, y los precios reales

**Gratis, verificado:** [Vraven](https://cursos.vraven.app/curso-claude-code.html) (13 módulos, ~5 h, **certificado incluido y gratis**, cubre subagentes, skills, hooks y MCP) · [Zero to Claude Code](https://claude2code.com/es) (150 lecciones, ~15 h, para principiantes absolutos) · [Hugging Face Agents Course, en español](https://huggingface.co/learn/agents-course/es/unit0/introduction) (**certificado también gratis**) · [AI Agents for Beginners de Microsoft, en español](https://microsoft.github.io/ai-agents-for-beginners/translations/es/) (**75.996★**, 18 lecciones) · [SDD con OpenSpec, de Web Reactiva](https://www.webreactiva.com/cursos/sdd-openspec).

**Vídeos en castellano, con visitas reales:** [Benjamín Cordero — CLAUDE CODE 2026: Curso Completo](https://www.youtube.com/watch?v=h49d1-d_fYk) (**6 h 17 m, 851.000 visitas**) · [MoureDev](https://www.youtube.com/watch?v=TCq7eZ9Lhc8) (3 h 08, 194K) · [EDteam — La forma correcta de programar con IA: SDD](https://www.youtube.com/watch?v=p2WA672HrdI) (18 min, 171K) · [Fazt](https://www.youtube.com/watch?v=Bf7hfpItrDk) (1 h 13, 144K) · [HolaMundo](https://www.youtube.com/watch?v=zVK9CAdsc-g) · [BettaTech — adaptando Claude Code para SDD](https://www.youtube.com/watch?v=ElGlTv2A_bM) (con [repositorio](https://github.com/betta-tech/harness-sdd)) · [CodelyTV](https://www.youtube.com/watch?v=-ZvNbTWXfc).

**De pago, si hay que elegir uno solo:** [**EDteam — Spec Driven Development**](https://ed.team/cursos/sdd), **16 USD**, ~1 h 48, y **construye un CLI real de gestión de tareas en TypeScript con SQLite usando Spec Kit y Claude Code**. O el de [Udemy](https://www.udemy.com/course/curso-completo-de-spec-driven-development-sdd-y-agentes-ia/), **11,99 €**, 10 h 20, **2.401 alumnos**, cubre Spec Kit y OpenSpec con Claude Code, Codex, Kiro y Ollama local.

**Avisos de los datos:** el «gratis» de DeepLearning.AI es **«gratis por tiempo limitado durante la beta»**, no por política. Google Skills **no es gratis ilimitado: son 35 créditos al mes**. Y la **certificación de Anthropic es de pago** (Pearson VUE, ~125 USD por intento) — no confundir la insignia gratuita con el certificado.

## Enlaces

- [[material-practico]] — el curso de BIG school y la guía de Claude Code
- [[crear-la-tarea]] — el formato, que el curso del fabricante confirma
- [[montaje-documentado]] — los montajes reales, con sus ficheros
- [[etapas]] — qué está maduro y qué necesita tu mano
