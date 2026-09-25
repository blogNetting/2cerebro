# Índice: proyectos/sistema-desarrollo-con-agentes

Sistema genérico para desarrollar software con agentes de IA: hub del proyecto, investigaciones y decisiones.

<!-- una línea por nota: [[nombre-de-nota]] — descripción -->

- [[sistema-desarrollo-con-agentes]] — hub del proyecto: decisiones cerradas, hipótesis, bloques y preguntas abiertas
- [[desarrollo-agentes-investigacion]] — síntesis de la investigación del ciclo completo con agentes: estándar de la comunidad fase a fase, modelos de Anthropic por fase, trazabilidad, hipótesis del usuario contrastadas y arquitectura propuesta
- [[desarrollo-agentes-f1-especificacion]] — de la idea a las tareas: frameworks spec-driven (Spec Kit, OpenSpec, BMAD, Kiro), formatos de requisitos y modelo por fase
- [[desarrollo-agentes-f2-ejecucion-y-trazabilidad]] — de la tarea a la PR: agentes issue→PR, orquestación, datos de revisión y merge, quién hizo qué
- [[desarrollo-agentes-f3-git-cicd-infra]] — trunk-based, rulesets, merge queue, forja, CI/CD y despliegue a pequeña escala, DORA con IA
- [[desarrollo-agentes-f4-devsecops]] — ASVS, SLSA, Scorecard, SSDF-AI, toolchain por etapa, mutation testing y no repudio
- [[orquestacion-opus-deepseek-informe]] — síntesis de la investigación de cómo Opus (suscripción) dirige y DeepSeek programa: hallazgo clave, descartes, 3 ejecutores finalistas y plan de pruebas
- [[orquestacion-herramientas-y-patrones]] — patrones y herramientas de orquestador/ejecutor (Aider, Cline, Kilo, routers, DeepClaude…) con métricas de repos e integración con git
- [[orquestacion-modelos-y-costes]] — modelos DeepSeek vigentes, precios, benchmarks frente a Claude, límites de Pro y boceto de coste por tarea
- [[orquestacion-experiencia-comunidad]] — qué reportan los usuarios reales: patrones que funcionan, fallos, costes y disciplina de tests
- [[orquestacion-seguridad-ejecutor]] — aislamiento (Podman + gVisor), catálogo de riesgos y controles por capa para el agente ejecutor
- [[circuito-tareas-definicion]] — definición de la fase actual: cómo se organiza realmente el trabajo con agentes (roles, traspaso, coordinación, topología, revisión, trazabilidad); criterios, método por fases, qué cuenta como contrastado y cuándo termina
- [[flujo-agentes-informe]] — informe de la investigación sobre cómo se organiza de verdad el trabajo con agentes: correcciones, descartes (Beads incluido), las seis preguntas con evidencia, recomendaciones y comparación con la idea de partida
- [[flujo-agentes-arquitectura]] — diseño operable: roles y modelos, contratos entre piezas, máquina de estados, reserva y caducidad con primitivas de GitHub Actions, configuración redactada, fallos y cambio de modelo
- [[flujo-agentes-runbook]] — puesta en marcha paso a paso, recorrido de una tarea con cada conexión verificada y prueba de humo
- [[flujo-fase-a-practicas-reales]] — fase A: 15 patrones de organización observados en la práctica
- [[flujo-fase-a2-practica-a-escala]] — fase A2: OpenAI, Stripe, Ramp, Spotify, Anthropic y la especificación de Symphony
- [[flujo-fase-b-implementaciones]] — fase B: inventario de herramientas y funciones nativas
- [[flujo-fase-b2-inventario-corregido]] — fase B2: corrección del inventario y de citas
- [[flujo-fase-c1-spec-y-estado]] — fase C1: frameworks de especificación y sistemas de estado contra R1–R13
- [[flujo-fase-c2-orquestacion-ejecucion-revision]] — fase C2: orquestadores, ejecutores, revisión y trazabilidad con interfaces exactas
