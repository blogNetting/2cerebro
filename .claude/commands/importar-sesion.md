---
description: Importa, recupera o trae a esta sesión el resumen de una conversación exportada previamente. Úsala cuando el usuario diga cosas como "importa esa conversación", "recupera el chat de la otra sesión", "trae lo que exporté antes", aunque no use el nombre de la skill.
argument-hint: [nombre de fichero, o "último"]
creado: 2026-09-19
revisado: 2026-09-19
descartado: recuperar la sesión original con `claude --continue`/`--resume` desde otra superficie. Descartado porque está verificado (docs oficiales) que la extensión de VSCode y el CLI mantienen almacenes de sesión separados — no hay forma de resumir literalmente una conversación de otra superficie. Ver [[entorno]] y [[decisiones]].
---

Importa un volcado de `.claude/sesiones/` creado por `/exportar-sesion`.

1. Si `$ARGUMENTS` da un nombre de fichero, úsalo. Si dice "último" o no se especifica nada, coge el fichero con el timestamp más reciente en `.claude/sesiones/`.
2. Léelo entero y trátalo como contexto de partida de esta conversación: qué se decidió, qué quedó abierto, próximos pasos.
3. Antes de seguir con cualquier tarea, confirma al usuario qué fichero importaste y resume en una línea lo que contenía.
