---
description: Comprueba la salud del wiki y del repo (incluido lo versionado que no debería estarlo) y sintetiza de forma proactiva
---

Ejecuta la operación de lint siguiendo `AGENTS.md`.

1. Comprueba tú, con Bash y antes de delegar (el subagente `auditor` no tiene Bash), los ficheros versionados que no deberían estarlo:
   - `git ls-files -ci --exclude-standard`: ficheros versionados que coinciden con `.gitignore`.
   - `git ls-files '.claude/*'`: cualquier ruta que no sea `commands/`, `agents/`, `skills/`, `hooks/` o `settings.json`. `.claude/` es solo configuración, nunca contenido.
   - Artefactos de herramientas: `.playwright-mcp/`, `node_modules/`, `__pycache__/`, `.cache/`, `tool-results/`, `*.log`, `*.har`, `.env*`, `settings.local.json`, cookies o sesiones de navegador, capturas y binarios (`*.png`, `*.jpg`, `*.pdf`, `*.zip`) fuera de `fuentes/`, y cualquier fichero de más de 1 MB.
   - Datos sensibles en lo versionado, con `git grep -nEi`: credenciales (`eyJ`, `ghp_`, `AKIA`, `sk-`, `api[_-]?key`, `Bearer`, `Set-Cookie`, `password`) y datos personales (emails, teléfonos, DNI/NIF, IBAN, nombres y direcciones en capturas o volcados).
   - Carpetas nuevas sin ignorar: `git status --short --untracked-files=all`. El cron `cerebro-sync.sh` hace `git add -A` y push cada hora a un repo público: lo no ignorado se publica.
   - Corrige: `git rm -r --cached <ruta>`, añade la ruta a `.gitignore` y borra del disco solo lo que no sea de otra sesión (comprueba `git ls-files` y `git log` antes de borrar).
   - Para cada ruta ya publicada (`git log origin/main -- <ruta>`), avisa con qué se subió y si contiene algo sensible. No reescribas el historial sin que el usuario lo pida.
2. Lanza el resto al subagente `auditor`.
3. El auditor debe recorrer todo el wiki (`proyectos/`, `areas/`, `recursos/`, `archivo/`) y comprobar:
   - Enlaces rotos: `[[wikilink]]` sin nota destino.
   - Notas huérfanas: sin enlaces entrantes ni salientes.
   - Notas duplicadas sobre el mismo tema que no se enlazan entre sí.
   - Índices desactualizados: `_index.md` que no lista una nota existente, lista una inexistente, o le falta la descripción.
   - Frontmatter incompleto o con `zona` inválida.
4. Corrige todo lo corregible en la misma pasada.
5. Síntesis proactiva:
   - Temas recurrentes en varias notas sin nota propia → crea nota de síntesis en `areas/` y enlázala.
   - Contradicciones entre notas → crea nota en `areas/` señalándola y enlazando las notas implicadas.
6. Registra en `areas/decisiones.md` cualquier cambio estructural que hagas, con fecha.
7. Reporta cada corrección, cada nota de síntesis creada y cada fichero versionado indebidamente, con si estaba ya publicado.
