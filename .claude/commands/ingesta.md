---
description: Ingiere una fuente y actualiza el wiki
argument-hint: [ruta a fichero de fuentes/ o ruta externa]
---

Ejecuta la operación de ingesta sobre `$ARGUMENTS` siguiendo `AGENTS.md`.

1. Localiza la fuente. Si la ruta es externa al wiki, cópiala tal cual a `fuentes/` con nombre descriptivo en minúsculas con guiones y añade su línea al `fuentes/_index.md` (origen y fecha). Si ya está en `fuentes/`, úsala directamente. La fuente original nunca se modifica ni se borra.
2. Lánzalo al subagente `archivista`. Pásale la ruta de la fuente en `fuentes/` y la fecha de hoy.
3. El archivista debe: leer la fuente entera; extraer el conocimiento en notas cortas y monotemáticas; clasificar cada una según el criterio de `AGENTS.md`; para cada nota, leer el `_index.md` de la carpeta destino y hacer `rg` de los términos clave antes de escribir; crear o actualizar notas; enlazar en ambas direcciones; actualizar todos los `_index.md` afectados; no dejar ninguna nota huérfana.
4. Reporta: fuente guardada, notas creadas, notas modificadas, enlaces añadidos, índices actualizados.
