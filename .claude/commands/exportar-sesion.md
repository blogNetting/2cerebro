---
description: Exporta, vuelca o comparte esta conversación/chat/sesión a un fichero para importarlo en otra sesión de claude (VSCode, CLI, u otra). Úsala cuando el usuario diga cosas como "exporta esta conversación", "vuelca el chat", "comparte esto con la otra sesión", aunque no use el nombre de la skill.
argument-hint: [nombre corto del tema; opcional "completo" para transcripción literal]
creado: 2026-09-19
revisado: 2026-09-19
descartado: volcado literal de la transcripción por defecto. Se descarta como comportamiento por defecto porque le cuesta a la sesión receptora casi el mismo contexto que haber estado en la conversación; queda disponible solo si se pide explícitamente en `$ARGUMENTS`. Ver [[entorno]] y [[decisiones]].
---

Vuelca el estado de esta conversación a `.claude/sesiones/` para que otra sesión de `claude` (VSCode, CLI, u otra) lo pueda importar. Ese directorio no es wiki: no lo clasifiques ni lo enlaces con las reglas de `AGENTS.md`, no entra en el `/lint`.

1. Redacta un resumen de estado, no la transcripción literal, salvo que `$ARGUMENTS` pida explícitamente "completo" o algo equivalente. El resumen debe cubrir: qué se decidió, qué quedó abierto o pendiente de confirmar, y los próximos pasos concretos.
2. Crea `.claude/sesiones/` si no existe.
3. Nombre de fichero: slug corto (de `$ARGUMENTS` o del tema de la conversación) + timestamp, formato `<slug>-<YYYYMMDD-HHmm>.md`.
4. Frontmatter simple: `titulo`, `timestamp`, `origen` (VSCode/CLI/u otra si se sabe).
5. Dile al usuario el nombre exacto del fichero y cómo importarlo en la otra sesión: `/importar-sesion <fichero>`.
