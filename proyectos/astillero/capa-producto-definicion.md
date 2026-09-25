---
title: Astillero — capa de producto, definición de la fase
created: 2026-09-25
updated: 2026-09-25
tags: [astillero, producto, roadmap, backlog, definicion]
zona: tecnico
---

Qué falta por encima de la ingeniería ya construida ([[flujo-agentes-arquitectura]]): cómo el usuario, como Product Owner, dirige el sistema sin fricción, de forma seria y profesional, replicable en cualquier proyecto nuevo bajo Astillero.

**Cerrado el 2026-09-25.** Resultado: [[capa-producto]].

## 1. Pregunta

¿Cómo captura, prioriza y libera trabajo una sola persona haciendo de Product Owner sobre un motor de ingeniería ya automatizado (Opus diseña, DeepSeek implementa, Opus revisa), con disciplina profesional pero sin ceremonia que no aporte? Restricción explícita del usuario: *«un enfoque más corporativo y serio, pero mientras funcione me da igual»* — la seriedad importa más que imitar el organigrama de una empresa grande.

## 2. Lo que ya existe y no se repite

- Diseño → especificación → tareas → implementación → revisión → merge: **[[flujo-agentes-arquitectura]]**, probado en producción real hoy.
- Replicación a nuevos proyectos: **[[astillero-replicacion]]**, verificada con una compilación real contra el import remoto publicado.
- DevSecOps y CI/CD: **[[desarrollo-agentes-f3-git-cicd-infra]]**, **[[desarrollo-agentes-f4-devsecops]]**.

## 3. Etapas a investigar (no presupuestas, se confirman o se descartan con evidencia)

1. **Captura de la idea.** Qué escribe el Product Owner para arrancar algo — ¿un problema, un resultado deseado, una funcionalidad? Formato mínimo que Opus necesita para no alucinar alcance.
2. **Roadmap y priorización.** Cómo se decide qué se construye antes, con una sola persona decidiendo y varios frentes en marcha. Evidencia real de equipos pequeños dirigidos por producto con ejecución muy automatizada — no procesos de Scrum de 8 personas.
3. **De la idea al diseño técnico.** El puente entre lo que pide el PO y la especificación que ya sabe producir Opus ([[flujo-fase-c1-spec-y-estado]]).
4. **Versionado y releases.** Cuándo algo es una versión liberable, changelog, semver o lo que use de verdad la comunidad a esta escala.
5. **Panel del Product Owner.** Cómo ve el usuario el estado de todo sin tener que leer logs de Actions: qué está en curso, qué espera su aprobación, qué se ha liberado.
6. **Feedback y bugs de vuelta al backlog.** Cómo entra un fallo detectado en producción otra vez al sistema, con la misma disciplina (contrato, tests) que una funcionalidad nueva.
7. **Gobernanza de decisiones de producto.** Dónde queda constancia de que se priorizó X sobre Y y por qué — el equivalente de producto a `areas/decisiones.md`, no otro ADR técnico.

## 4. Qué se descarta a priori, y por qué

- **Ceremonias de equipo humano** (daily standup, planning poker, retro de sprint): no tienen sentido con una sola persona y ejecución automatizada. Se investigan solo si alguna práctica concreta demuestra valor sin necesitar un equipo.
- **Departamentos no técnicos** (marketing, ventas, soporte, finanzas): fuera de alcance, confirmado por el usuario.

## 5. Método

Igual que hoy: subagentes en Sonnet, evidencia real y contrastada (no la más popular, la que funciona), verificación propia del orquestador en las fuentes que sostengan cada pieza del diseño final, todo documentado y con enlaces.

## Enlaces

- [[astillero]] — proyecto
- [[flujo-agentes-arquitectura]] — motor de ingeniería que esta capa dirige
- [[capa-producto]] — resultado de esta fase
- [[_index]]
