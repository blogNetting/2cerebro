---
description: Responde una pregunta consultando el wiki
argument-hint: [pregunta]
---

Responde a `$ARGUMENTS` consultando el wiki, siguiendo `AGENTS.md`.

1. Lee los `_index.md` relevantes para localizar notas candidatas por concepto.
2. Sigue los `[[wikilinks]]` de esas notas para ampliar el contexto.
3. Sólo si falta información, haz `rg` de los términos clave en todo el wiki.
4. No consultes `archivo/` salvo que la pregunta lo pida explícitamente.
5. Si durante la consulta una nota de `recursos/` resulta relevante para una pregunta técnica, aplica la regla de promoción de `AGENTS.md`: muévela a la zona técnica, cambia `zona`, y actualiza enlaces e índices.
6. Responde de forma directa, al nivel de un par técnico. Cita al final las notas de las que sale la respuesta como `[[nombre-de-nota]]`.
7. Si el wiki no cubre la pregunta, dilo y sugiere ingerir una fuente.
