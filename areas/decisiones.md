---
title: Registro de decisiones
created: 2026-09-08
updated: 2026-09-08
tags: [meta, arquitectura]
zona: tecnico
---

Decisiones de arquitectura del wiki y correcciones del usuario, con fecha. Consultar antes de proponer cambios estructurales.

## 2026-09-08 — Creación del wiki

- Patrón LLM Wiki de Karpathy. Tres capas: fuentes brutas inmutables, wiki mantenido por el LLM, esquema en `AGENTS.md`.
- Metodología PARA: `proyectos/`, `areas/`, `recursos/`, `archivo/`. Más `fuentes/` para material bruto e `inbox.md` para captura.
- `AGENTS.md` como fuente única de verdad; `CLAUDE.md` sólo lo importa. Objetivo: compatibilidad con Codex y Gemini CLI.
- Clasificación automática, sin preguntar al usuario. Enlace bidireccional obligatorio. Índices por carpeta obligatorios desde la primera nota.
- Formato de nota: frontmatter YAML plano (title, created, updated, tags, zona), resumen de una línea, cuerpo. Ficheros cortos y monotemáticos.
- Operaciones: ingesta, consulta (`/buscar`), lint con síntesis proactiva.
- Subagentes: `archivista` (ingesta e inbox), `auditor` (lint y síntesis).
- Umbral de subdivisión de carpeta: 30 notas.
