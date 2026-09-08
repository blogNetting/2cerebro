---
description: Vacía inbox.md clasificando cada entrada en el wiki
---

Procesa `inbox.md` siguiendo `AGENTS.md`.

1. Lee `inbox.md`. Cada bloque separado por línea en blanco es una entrada.
2. Lánzalo al subagente `archivista` con el contenido de las entradas y la fecha de hoy.
3. Para cada entrada, el archivista debe: clasificarla según el criterio de `AGENTS.md`; leer el `_index.md` de la carpeta destino y hacer `rg` de los términos clave; crear o actualizar la nota correspondiente; enlazar en ambas direcciones; actualizar los `_index.md` afectados; no dejar ninguna nota huérfana.
4. Cuando todas las entradas estén procesadas, deja `inbox.md` con sólo su cabecera:

```
# Inbox

Captura sin clasificar. Una entrada por bloque. `/inbox` la vacía.
```

5. Reporta: entradas procesadas, notas creadas, notas modificadas, enlaces añadidos, índices actualizados.
