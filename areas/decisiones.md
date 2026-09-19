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

## 2026-09-19 — Escalado de búsquedas web y navegador por CDP

- Investigación y decisión: ver [[entorno]] para el detalle de MCPs evaluados.
- Skills nuevas, organizadas por capacidad y no por sitio: `/investigar-web` (capas 1-2, `WebSearch`→`WebFetch`, autónoma) y `/navegador-cdp` (capa 3, Chrome real por CDP). Descartado un tercer skill `precio-producto`: el usuario decidió que la comparación de precio/producto es solo un criterio de búsqueda dentro de `investigar-web`, no una capacidad aparte — dos skills bastan.
- MCP elegido para el paso 3: **Playwright MCP** (Microsoft), conectado a un Chrome ya corriendo vía `--cdp-endpoint`. Descartado `chrome-devtools-mcp` (Google): también conecta por CDP a un navegador existente y cumple el criterio eliminatorio, pero está orientado a depurar (red, performance, consola) y no a conducir el navegador; cada tool de más consume contexto en cada turno sin aportar a este caso de uso. Descartado Puppeteer MCP (deprecado/archivado) y navegadores cloud tipo BrowserBase (lanzan su propio navegador, no el existente).
- Descartada la capa de datos de producto/precio sin scraping (Keepa API, SerpApi): requieren API key de pago y el usuario decidió no incluirla por ahora. Si se revisita, evaluar bajo el mismo criterio de "herramienta existente antes que script improvisado".
- Registrado en `.mcp.json` del proyecto: `playwright-cdp`, con `--cdp-endpoint=${CDP_ENDPOINT:-http://192.168.1.5:9222}`. Limitación asumida: el endpoint se fija al arrancar el proceso MCP; cambiar de máquina dentro de la misma sesión no es posible, hace falta relanzar `claude` con `CDP_ENDPOINT` puesto a la IP de la otra máquina.
- Regla añadida a `AGENTS.md`: reutilización de skills — comprobar si ya existe una antes de resolver algo o de crear una nueva, y registrar en cada skill qué alternativa se descartó y por qué.

## 2026-09-19 — Puente de contexto entre sesiones (VSCode ↔ CLI)

- Verificado en docs oficiales: la extensión de VSCode y el CLI mantienen almacenes de sesión separados; `claude --continue`/`--resume` no puede recuperar una conversación de la otra superficie. Sin solución nativa.
- Solución: directorio `.claude/sesiones/` (gitignored, fuera del PARA, no lo tocan `/lint` ni las reglas de clasificación de `AGENTS.md`). Skills `/exportar-sesion` y `/importar-sesion` — ver [[entorno]].
- Por defecto se vuelca un resumen de estado, no la transcripción literal: cuesta menos contexto a la sesión receptora. La transcripción completa queda disponible solo si se pide explícitamente.
