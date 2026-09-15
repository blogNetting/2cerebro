---
title: Registro de decisiones
created: 2026-09-08
updated: 2026-09-09
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

## 2026-09-09 — Estructura de proyectos con alcance abierto

- Cuando un proyecto arranca sin saber qué apartados tendrá, crear solo la nota hub del proyecto con una lista de "frentes de trabajo". Cada frente se saca a su propia nota corta cuando se aborda, no antes.
- Aplicado a [[apartamentos-calle-uruguay]].

## 2026-09-14 — Renombrado del proyecto del dúplex de Carballo

- El proyecto "Alquiler dúplex dividido en dos apartamentos" pasa a llamarse **Apartamentos Calle Uruguay**, nombre más simple ligado a la dirección (R/ Uruguay 2-4, Carballo). Fichero renombrado a `apartamentos-calle-uruguay.md`, actualizados los enlaces entrantes.
- Nuevo frente abierto en el hub: portero automático que no debe molestar a los dos inquilinos a la vez. Sin nota propia todavía — se crea cuando llegue el análisis, siguiendo la regla del apartado anterior.
