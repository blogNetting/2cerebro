---
title: Organización del trabajo con agentes — definición de la fase de investigación
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, roles, orquestacion, revision, investigacion, definicion]
zona: tecnico
---

Definición de la fase actual: cómo se organiza realmente el trabajo de desarrollo con agentes de IA, desde el diseño hasta el código revisado. Qué se pregunta, qué está decidido, qué no se presupone, cómo se investiga y cuándo se da por terminada.

> **Reescrita el 2026-09-25.** La versión anterior presuponía «tareas independientes» y un circuito fijo de cuatro pasos, y empezaba por las herramientas. El usuario lo corrigió: eso nunca lo pidió, y primero hay que averiguar cómo se hace de verdad. El descubrimiento que se lanzó con esa versión (F1) queda anulado como resultado; sus datos brutos solo sirven de materia prima para la fase B.

## 1. Pregunta

**¿Cómo se organiza realmente, en la práctica contrastada, el trabajo de desarrollo de software cuando intervienen agentes de IA desde el diseño hasta el código revisado, y cuál es la mejor forma de hacerlo?**

Se desglosa en seis preguntas. Ninguna tiene respuesta presupuesta:

1. **Roles.** Qué papeles existen (diseñar, dividir el trabajo, implementar, probar, revisar, integrar, u otros) y cómo se reparten entre modelos, agentes y personas.
2. **Traspaso.** En qué unidad pasa el trabajo de un rol a otro y qué contiene (el *handoff*, la entrega de trabajo entre roles): una especificación, una lista de tareas, una issue, un plan, una conversación compartida… o lo que resulte.
3. **Coordinación.** Dónde vive el estado del trabajo, cómo se asigna, cómo se evita que dos agentes se pisen y qué pasa cuando algo falla.
4. **Topología.** Cuántos agentes trabajan a la vez y cómo se organizan: uno solo, varios en paralelo, una jerarquía, u otra forma.
5. **Revisión.** Quién revisa, con qué controles y cómo vuelve el trabajo si no pasa.
6. **Trazabilidad.** Cómo se sabe después qué rol, agente o modelo hizo cada cosa.

## 2. Qué está decidido

Son restricciones, no hipótesis.

- **Quien ocupa cada rol es intercambiable.** La arquitectura no depende del modelo que haga cada papel. Para implementar se usará DeepSeek por coste.
- La forja es **GitHub**.
- Claude va por **suscripción Pro**, con límites de uso.
- **No hay código sin tests.**
- Todo debe funcionar en la VM actual: 4 núcleos, 7 GB de RAM, sin KVM y sin motor de contenedores instalado (se puede instalar).
- Rige `AGENTS.md`, en particular la sección «Lo que menciona el usuario no condiciona nada».

## 3. Idea de partida del usuario

Es una hipótesis. **Se compara al final y no guía la búsqueda.**

Opus 5.5 hace el diseño y genera el trabajo; uno o varios ejecutores lo implementan; el resultado se revisa; entre medias hay algún sistema que guarda el trabajo pendiente.

El informe final lleva una sección obligatoria: **«Qué se hace así en la práctica, qué no, y qué encaja con lo que habíamos hablado»**.

## 4. Alcance

**Dentro:** las seis preguntas de la sección 1, para desarrollo de software de principio a fin (diseño → código revisado e integrado).

**Fuera:**
- El conector técnico con cada modelo; ver [[orquestacion-opus-deepseek-informe]].
- CI/CD y DevSecOps generales; ver [[desarrollo-agentes-investigacion]].
- Cualquier piloto o implementación.

## 5. Criterios de evaluación

Salen del problema y se aplican por igual a prácticas, patrones y herramientas. No hay pesos fijos: C13 (madurez y evidencia) decide si algo puede recomendarse.

| # | Criterio | Qué se mira |
|---|---|---|
| C1 | División y especificación del trabajo | Granularidad, qué contiene el traspaso, criterios de aceptación |
| C2 | Roles | Qué roles hay, cómo se reparten entre modelos y personas, y si permite cambiar quién ocupa cada rol |
| C3 | Estado del trabajo | Dónde vive, si persiste y se versiona, dependencias entre piezas |
| C4 | Asignación y concurrencia | Cómo se reparte el trabajo y cómo se evitan choques. *Claim* es la reserva de una pieza de trabajo por un agente; *lease* es una reserva con caducidad que la libera si el agente muere |
| C5 | Topología | Uno, varios o jerarquía; aislamiento entre agentes, por ejemplo con *worktree* (una copia de trabajo git independiente) o contenedor |
| C6 | Revisión y rehacer | Quién revisa, con qué controles automáticos y cómo vuelve el trabajo |
| C7 | Trazabilidad | Qué rol, agente o modelo hizo cada cambio y quién lo aprobó |
| C8 | Fallos y recuperación | Bucles, errores, abandono, reintentos, escalado a una persona |
| C9 | Coste y observabilidad | Tokens, dinero, tiempo y consumo de la cuota Pro por unidad de trabajo |
| C10 | Coste operativo | Dependencias, procesos en segundo plano, complejidad; si cabe en la VM |
| C11 | Encaje con GitHub | PRs, checks, rulesets, `CODEOWNERS` (el fichero que asigna revisores obligatorios por ruta) |
| C12 | Seguridad | Inyección de instrucciones a través del trabajo que se traspasa; permisos de cada rol |
| C13 | Madurez y evidencia | Uso real documentado, mantenimiento y **evidencia en contra** |

## 5 bis. Requisitos del estado del trabajo

Qué tiene que resolver la pieza donde vive el trabajo pendiente cuando un agente lo coge. Se extraen de los problemas que atacan las herramientas de este tipo (por ejemplo, el README de [Beads](https://github.com/gastownhall/beads)) y de la práctica de las fases A y A2. Son requisitos, no una herramienta. Beads y cualquier alternativa se miden contra ellos por igual.

| # | Requisito | Problema que evita |
|---|---|---|
| R1 | Estado persistente y estructurado que sobrevive a sesiones y reinicios de contexto | El agente pierde el hilo al empezar una sesión nueva |
| R2 | Dependencias entre piezas y detección automática de lo que está **listo** (sin bloqueos) | Coger algo cuyo prerrequisito no está hecho |
| R3 | **Reserva atómica** (*claim*): asignar y marcar «en curso» en una sola operación | Dos agentes con la misma pieza |
| R4 | Recuperación de reservas abandonadas: caducidad o *lease* | Una pieza queda bloqueada para siempre si el agente muere (fallo documentado en Symphony, fase A2) |
| R5 | Identificadores sin colisión entre ramas y agentes; fusión sin conflictos | Choques al trabajar en paralelo en varias ramas |
| R6 | Sincronización entre máquinas y agentes | El estado difiere según dónde se mire |
| R7 | Interfaz para agentes independiente del modelo: CLI con salida JSON, API o MCP (*Model Context Protocol*, el protocolo estándar para dar herramientas a un agente) | Atarse a un arnés o a un vendor |
| R8 | Jerarquía: épica → tarea → subtarea | No poder reflejar el desglose del diseño |
| R9 | Historial de auditoría por pieza: quién, cuándo, qué cambió | No saber quién hizo qué |
| R10 | Economía de contexto: resumen de lo cerrado e inyección del contexto justo al empezar | Llenar la ventana de contexto con historia irrelevante |
| R11 | Memoria del proyecto persistente para los agentes | Repetir errores ya resueltos |
| R12 | Vínculo con GitHub (issues, PRs, commits) y visibilidad para personas | Que el trabajo viva en un sitio que nadie ve |
| R13 | Coste operativo asumible en la VM (procesos en segundo plano, base de datos) | Complejidad que no compensa (crítica práctica a Beads, [HN](https://news.ycombinator.com/item?id=46487580)) |

## 6. Método

Principio: **primero cómo se hace realmente; después con qué herramientas; al final, la comparación con la idea de partida.**

| Fase | Qué | Cómo | Salida |
|---|---|---|---|
| A. Prácticas reales | Cómo organizan este trabajo los equipos que lo hacen en producción | Informes de equipos con datos, flujos que documentan los propios vendors sobre su trabajo interno, estudios académicos, hilos técnicos con experiencia de primera mano. **Sin partir de herramientas** | Catálogo de patrones de organización, con la evidencia de cada uno y de qué preguntas de la sección 1 responde |
| B. Implementaciones | Qué herramientas, productos y funciones nativas materializan esos patrones | Barrido sistemático:<br>- **Todo proyecto de ≥500★ encontrado aparece clasificado** como relevante o no, con motivo<br>- Documentación completa de las funciones nativas de los arneses de agentes (Claude Code, Codex, Gemini CLI, OpenCode, Cursor, Copilot, Kiro, Amp, y los que aparezcan)<br>- Productos de los grandes vendors<br>- Papers verificados en su página<br>- Control de estrellas infladas: muestreo de fechas y cobertura independiente | Lista completa y clasificada, más una preselección con motivos y exclusiones |
| C. Evaluación profunda | Cada patrón y herramienta preseleccionados contra C1–C13 | Documentación, código, issues y experiencias; **evidencia en contra obligatoria** | Tabla de evaluación |
| D. Verificación y síntesis | Comprobación en fuente primaria de cada dato que sostiene una conclusión | La hace el orquestador | Informe final con el formato de `AGENTS.md`, más la sección de comparación con la idea de partida |

**Revisión entre fases.** Ninguna fase pasa a la siguiente sin que el orquestador la revise:
- que estén cubiertos todos los enfoques;
- que los datos clave estén verificados en fuente;
- que no se cuele un encuadre heredado de lo que ha dicho el usuario;
- que no haya huecos.

Si falla algo, la fase se repite o se completa. Así se detectaron los fallos del primer descubrimiento: no vio los *agent teams* de Claude Code y le faltaban candidatos con mucha adopción.

**Puntos de control con el usuario:**
- al terminar A+B, con la preselección;
- al final, con el informe.

**Reparto de trabajo en la investigación:**
- La búsqueda y el análisis los hacen subagentes en **Sonnet 5**.
- La revisión, la verificación y la síntesis las hace el orquestador (Opus).
- Máximo dos subagentes a la vez, sin subagentes anidados, para no agotar la cuota.

## 7. Qué cuenta como «contrastado»

- Uso real documentado por **al menos dos fuentes independientes**, o por una fuente primaria con datos cuantitativos (por ejemplo, un informe de un equipo con cifras propias).
- Una afirmación de un vendor sobre su propio producto no basta por sí sola.
- Las estrellas miden popularidad, no uso.
- Se desconfía de las fuentes con fechas imposibles o con interés comercial no declarado (caso Uvik, [[desarrollo-agentes-investigacion]]).
- Si solo hay evidencia débil, se dice. «Prometedor sin contrastar» no se recomienda como opción principal.

## 8. Reglas de evidencia

- Cada afirmación con fuente lleva su URL en la misma línea.
- Por cada conclusión: fuente primaria, número de fuentes independientes, tipo de evidencia (documentación oficial, afirmación de un vendor, práctica reportada, estudio, benchmark) y confianza, con el motivo.
- Lo no verificado se marca «sin verificar» o se omite.
- Las inferencias propias se etiquetan como tales.
- Cada sigla o término especializado se explica la primera vez que aparece.

## 9. Cuándo termina la fase

- Cada una de las seis preguntas de la sección 1 tiene respuesta con evidencia, o está marcada explícitamente como «sin evidencia suficiente».
- Cada familia de enfoques tiene al menos un representante evaluado o descartado con motivo.
- La comparación con la idea de partida está hecha.

## 10. Entregables

- **Flujo completo y operable, sin instalar nada todavía:**
  1. Arquitectura: componentes y qué rol cubre cada uno.
  2. **Contratos entre piezas:** qué entrega cada paso al siguiente, en qué formato y por qué interfaz (CLI, MCP, API, git).
  3. Configuración redactada: ficheros y parámetros listos para aplicar.
  4. Runbook paso a paso desde un proyecto vacío.
  5. **Comprobación de coherencia:** recorrido completo de una unidad de trabajo sobre el papel, verificando en la documentación de cada pieza que cada conexión existe de verdad.
  6. Escenarios de fallo y su recuperación.
  7. Cómo se cambia el modelo de cada rol.
  8. Coste y trazabilidad.

- **Procedimiento replicable** para cualquier proyecto: cómo se hace cada paso, desde lo que define el modelo que diseña hasta el trabajo implementado y revisado por otro modelo. Qué herramientas hacen falta y cómo se configuran, sin depender de qué modelo ocupe cada rol.

- Informes de fase en esta carpeta.
- Síntesis final con este orden:
  1. introducción
  2. descartado y por qué
  3. análisis
  4. recomendaciones
  5. dónde se ha buscado
  6. otros
  7. comparación con la idea de partida
- Decisiones propuestas, no preguntas genéricas.

## 11. Bloqueos y cómo puede ayudar el usuario

| Bloqueo | Efecto | Ayuda posible |
|---|---|---|
| Cupo de búsqueda web agotado (200 por sesión) | Solo lectura directa de páginas, `gh` y APIs | Abrir una sesión nueva de Claude Code para las fases A y B; es posible que el cupo sea por sesión (**sin verificar**) |
| API de arXiv (repositorio de papers) responde 406 desde esta máquina | Se usa la búsqueda web de arxiv.org, que sí funciona | — |
| API de Semantic Scholar (buscador académico) responde 429 sin clave | Búsqueda académica limitada | Pedir una clave gratuita en semanticscholar.org/product/api |
| Reddit y X solo se leen con el Chrome real | Parte de la fase A depende de ello | Chrome encendido en `192.168.1.5:9222` |
| Límite de uso del plan Pro | El trabajo se para hasta el reinicio | — |

## Enlaces

- [[sistema-desarrollo-con-agentes]] — proyecto
- [[desarrollo-agentes-investigacion]] — investigación previa del ciclo completo
- [[orquestacion-opus-deepseek-informe]] — conexión con cada modelo (fuera de alcance aquí)
- [[_index]]
