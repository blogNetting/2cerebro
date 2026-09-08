---
description: Comprueba la salud del wiki y sintetiza de forma proactiva
---

Ejecuta la operación de lint siguiendo `AGENTS.md`.

1. Lánzalo al subagente `auditor`.
2. El auditor debe recorrer todo el wiki (`proyectos/`, `areas/`, `recursos/`, `archivo/`) y comprobar:
   - Enlaces rotos: `[[wikilink]]` sin nota destino.
   - Notas huérfanas: sin enlaces entrantes ni salientes.
   - Notas duplicadas sobre el mismo tema que no se enlazan entre sí.
   - Índices desactualizados: `_index.md` que no lista una nota existente, lista una inexistente, o le falta la descripción.
   - Frontmatter incompleto o con `zona` inválida.
3. Corrige todo lo corregible en la misma pasada.
4. Síntesis proactiva:
   - Temas recurrentes en varias notas sin nota propia → crea nota de síntesis en `areas/` y enlázala.
   - Contradicciones entre notas → crea nota en `areas/` señalándola y enlazando las notas implicadas.
5. Registra en `areas/decisiones.md` cualquier cambio estructural que hagas, con fecha.
6. Reporta cada corrección y cada nota de síntesis creada.
