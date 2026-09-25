---
title: Flujo de desarrollo con agentes — prueba real de descomposición
created: 2026-09-25
updated: 2026-09-25
tags: [agentes, opus, spec-driven, prueba-real]
zona: tecnico
---

¿Sabe Opus convertir una idea en tareas bien especificadas? Prueba real, no simulada: Opus 5.5 (en un subagente aparte, no el orquestador describiéndose a sí mismo) recibió una idea de una frase y produjo una especificación y cuatro tareas, creadas de verdad como issues en el repo de prueba `blogNetting/prueba-flujo-agentes`.

## 1. Resultado

**Sí, y con margen.** La especificación (issues #5–#10) fijó 8 decisiones de diseño explícitas con su alternativa descartada, un contrato completo por tarea (interfaces con firmas exactas, tabla de casos entrada→salida, criterios en formato EARS con su test asociado), y una división en grafo de dependencias (una tarea base, dos en paralelo, una que las compone) razonada explícitamente contra alternativas más finas y más gruesas.

**Verificación, no autoevaluación:** antes de publicar nada, escribió una implementación de referencia que cumple los cuatro contratos. Pasan 54 tests y `mypy --strict`. Esa comprobación **encontró un fallo real en su propio primer borrador** (`statistics.median` desborda con valores grandes) y lo corrigió antes de publicar — el mismo patrón de «verificar antes de aceptar» que se le exige al orquestador en todo este proyecto.

## 2. Lo que la prueba destapó, no lo que se le pidió

1. **El disparo del ejecutor no espera a que le pidas nada.** Al poner `agente:implementar` en la tarea #7, el workflow saltó solo. De 5 eventos generados (por editar varias etiquetas a la vez), **gh-aw se saltó 4 y ejecutó 1**, que falló donde tenía que fallar: sin `DEEPSEEK_API_KEY`. Verificado por el orquestador en los logs reales de GitHub Actions. Es la primera confirmación en producción de que la deduplicación de `label_command` funciona, no solo de que está documentada.
2. **Riesgo real de bloqueo identificado por el propio Opus:** `pyproject.toml` es ruta de `CODEOWNERS` en el diseño. Si la protección de ficheros de `create-pull-request` (gh-aw) lo trata como protegido, la primera tarea no puede cerrarla el agente — necesita una persona. Queda como punto a confirmar en la prueba de humo real ([[flujo-agentes-runbook]]).
3. **El repo de prueba no tenía CI.** Opus redefinió «checks obligatorios» como `pytest` y `mypy` locales, documentándolo explícitamente en cada contrato en vez de fingir que existía una CI que no está.
4. **Se saltó el orden del runbook a propósito**, y lo dijo: puso la etiqueta de ejecución antes de que la PR de especificación estuviera aprobada, porque el encargo se lo pedía. En operación real, ese orden sería un fallo (K2 antes que K4 en [[flujo-agentes-arquitectura]] §4).

## 3. Autocrítica de Opus, sin filtrar

- Dos criterios de aceptación no son tests de pytest, sino comandos que tienen que salir con código 0 (`pytest` y `mypy` en sí mismos) — lo marcó explícitamente en vez de disfrazarlo de test automático.
- Un test depende del tiempo de ejecución de la máquina (con margen de 60×, pero lo señaló como debilidad).
- Un test se apoya en un detalle de implementación (nombres de import exactos) para poder sustituir dependencias con `monkeypatch`.
- Señaló, sin que se lo pidieran, un hueco de seguridad real: sin límite de tamaño de cuerpo en bytes, un POST de cientos de MB se procesa entero antes de validar. Lo dejó fuera de alcance de forma explícita, no lo ocultó.

## 4. Qué significa esto para el diseño

- **La calidad de la descomposición no es el cuello de botella** que yo (el orquestador) había asumido sin comprobar. El cuello de botella real, según [[flujo-agentes-evidencia-empirica]], está más adelante: si el ejecutor sigue el contrato de verdad.
- **La plantilla de contrato de tarea** ([[flujo-agentes-arquitectura]] §6) queda validada como suficiente para que Opus produzca especificaciones ejecutables — con una única espec de prueba, no generalizable sin más repeticiones.
- Esta prueba **no mide si DeepSeek va a implementar bien estos contratos**. Eso solo se sabe con la clave real (pendiente, ver [[flujo-agentes-informe]]).

## 5. Artefactos

- Especificación e implementación de referencia completas: en el scratchpad de la sesión, no en el wiki.
- Issues reales: `blogNetting/prueba-flujo-agentes` #5 (PR de especificación), #6 (épica), #7–#10 (tareas). Repo privado, desechable.

## Enlaces

- [[flujo-agentes-informe]] — síntesis general
- [[flujo-agentes-arquitectura]] — contrato de tarea usado en la prueba
- [[flujo-agentes-evidencia-empirica]] — por qué la descomposición no es lo que falta comprobar
- [[astillero]] — proyecto
- [[_index]]
