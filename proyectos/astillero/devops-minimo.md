---
title: Astillero — DevOps mínimo en producción
created: 2026-09-25
updated: 2026-09-25
tags: [astillero, devops, observabilidad, incidentes, backup, secretos]
zona: tecnico
---

Lo mínimo para tener controlada una app ya en producción, dirigida por un operador solo. No repite CI/CD ni DevSecOps de pipeline (ya cerrados en [[desarrollo-agentes-f3-git-cicd-infra]] y [[desarrollo-agentes-f4-devsecops]]), ni el mecanismo de despliegue ([[flujo-agentes-arquitectura]] §15) — cubre lo que pasa **después** de que el código ya corre.

**Eje del hallazgo:** todo lo de abajo está anclado en un caso real, auditable, no en opinión — **Healthchecks.io**, SaaS operado en solitario por Pēteris Caune (HN `cuu508`), repo público activo hoy (`healthchecks/healthchecks`, 10.366★), que publicó su stack de producción completo: [blog.healthchecks.io/2022/02/healthchecks-io-hosting-setup-2022-edition](https://blog.healthchecks.io/2022/02/healthchecks-io-hosting-setup-2022-edition/) ✔︎. Corroborado con un hilo real de 149 puntos de founders solos discutiendo justo este problema: [HN 26203074](https://news.ycombinator.com/item?id=26203074) ✔︎.

## 1. Monitorización y observabilidad

| Pieza | Elección | Por qué | Coste |
|---|---|---|---|
| Métricas de infraestructura | Netdata (agente + Netdata Cloud) | Usado en producción real por Healthchecks.io: *«Netdata agent for monitoring the machines and the services running on them»* ✔︎ | Community gratis hasta 5 nodos — suficiente a esta escala. [netdata.cloud/pricing](https://www.netdata.cloud/pricing/) |
| Errores de aplicación | Sentry | Adoptado por operadores solos reales del hilo HN (`deforciant`) | Plan Developer gratis: 1 usuario, 5.000 errores/mes, 30 días de retención. [sentry.io/pricing](https://sentry.io/pricing/) |
| Uptime externo | HetrixTools (Healthchecks.io) o UptimeRobot (mención real en HN) | Comprobación desde fuera de tu propia infra — si tu servidor cae, tu propio Netdata no te avisa | Ambos tienen tier gratis |
| Página de estado (opcional) | Uptime-Kuma (autoalojado, 91.824★ — con diferencia la herramienta con más adopción real de todo lo investigado) u OpenStatus (SaaS, gratis 1 monitor) | Solo si hay usuarios externos a los que comunicar incidentes | — |

**Descartado:** Grafana/Prometheus self-hosted — activos y con estrellas altas, pero ningún operador solo documentado los citó como su elección real; solo aparecieron en contexto de equipo. No se recomienda montar esa pila para un solo nodo pequeño.

## 2. Alertado

- **Pushover**, citado literalmente por el propio operador de Healthchecks.io: *«For critical notifications I use Pushover with an emergency setting – a repeating full volume alert on phone, regardless of volume settings or Do Not Disturb mode»* ✔︎. Precio exacto no verificado en esta sesión.
- **Twilio** para SMS/llamada, parte del mismo stack real.
- **Better Stack**, alternativa SaaS: plan gratis con 10 monitores y alertas ilimitadas por teléfono/SMS/push; $25/mes por 50 monitores más. [betterstack.com/pricing](https://betterstack.com/pricing) ✔︎.

**Regla:** alerta solo lo que de verdad requiere acción tuya ahora — Sentry y Netdata generan ruido si se conectan todos sus umbrales por defecto a un canal de emergencia.

## 3. Gestión de incidentes: no hace falta on-call formal

**Hallazgo contraintuitivo, verificado en la fuente que abre el debate:** el hilo *"Ask HN: How do solo SaaS founders handle monitoring/PagerDuty?"* (149 puntos) concluye, con consenso mayoritario (`wongarsu`, `ozim`): *«you design a system that doesn't go down while you are on holiday, but with monitoring for the unlikely case that it does»* — [HN 26203074](https://news.ycombinator.com/item?id=26203074) ✔︎. Nadie de ese hilo defiende rotación de guardias tipo PagerDuty con un solo operador.

- **Diseña para resiliencia, no para vigilancia constante**: auto-restart de servicios (`systemd`/`pm2`/equivalente), health checks HTTP con reinicio automático.
- **Caso extremo real, mismo hilo**: un operador (`leesalminen`) compró un teléfono satélite para recibir alertas de Pingdom estando de vacaciones — el coste real de no automatizar bien el failover, no una recomendación a imitar literalmente, sino la prueba de que la alerta que te despierta importa más que la vigilancia activa.
- **Postmortem ligero, sin herramienta dedicada**: *«Write good issue templates for features, bugs, and incidents. Do after incident reports... document the root cause and the recovery»* (`Jugurtha`, mismo hilo) ✔︎ — encaja directo en el mismo mecanismo de issues que ya usa Astillero para todo lo demás, sin pieza nueva.

## 4. Backup y recuperación ante desastres

Todo verificado en la fuente primaria de Healthchecks.io:

- *«Once a day, make a full database backup, encrypt it with gpg, and upload it to AWS S3»* ✔︎.
- *«PostgreSQL 13, streaming replication from primary to standby»* ✔︎.
- **Failover deliberadamente manual**: *«No automatic failover: I can trigger failover with a single command»* ✔︎ — no es negligencia, el propio operador lo confirma en el hilo HN como decisión consciente: el failover automático de bases de datos *«is just a too hard problem»* incluso para quien automatiza el resto ✔︎.
- Hallazgo adicional del fork de despliegue, mismo hilo: `brokegrammer` usa SQLite en modo WAL + **Litestream** para backup continuo — alternativa real si el proyecto usa SQLite en vez de Postgres.

## 5. Rotación de secretos en producción

- Healthchecks.io usa **sops** para guardar secretos de producción — mismo patrón que ya usa Astillero para `DEEPSEEK_API_KEY` en CI.
- Umbral real citado con argumento, no cifra vacía: *«If you have a handful (say less than 10) repos, you can and should just check-in encrypted secrets with git-crypt or SOPS. Whenever master enc/dec keys are rotated, you can afford to make a dozen PRs. If you have hundreds of repos, you'll need [Vault]»* (`rdsubhas`, [HN 23048715](https://news.ycombinator.com/item?id=23048715)) ✔︎ — el umbral es **número de repos, no tamaño de equipo**, y Astillero está muy por debajo de 10.
- **No hay cifra real de cadencia de rotación** (cada cuánto) a esta escala — se omite, no se inventa.
- **Descartado: HashiCorp Vault** — la propia fuente que recomienda `sops` lo descarta explícitamente por debajo de ~10 repos.

## 6. Parcheo de dependencias

**Hallazgo que contradice la asunción cómoda: Healthchecks.io no usa Dependabot ni Renovate.** Verificado directamente contra su repo (`gh api repos/healthchecks/healthchecks/contents/.github` → sin `dependabot.yml`) y su historial real de `requirements.txt` (bumps manuales y periódicos, ej. *"Bump Django and psycopg versions"*) — decisión deliberada, no descuido: la red de seguridad real es *«GitHub Actions for running tests on every commit»* ✔︎.

**Para Astillero, con CI real corriendo en cada tarea del ejecutor (§8 de [[flujo-agentes-arquitectura]]), el bump manual periódico es defendible por el mismo motivo.** Si se prefiere automatizarlo de todos modos, **Renovate** y **Dependabot** siguen activamente mantenidos, sin evidencia real de que un operador solo prefiera uno sobre otro — indiferente a esta escala.

## 7. Coste de referencia

**Única cifra real, en fuente primaria, del coste total de infraestructura de un SaaS operado en solitario:** *«The monthly Hetzner bill is €484»* (Healthchecks.io, incluye servidores + todo el stack de arriba, sin desglose por herramienta) ✔︎.

Costes SaaS puntuales de referencia si no se autoaloja: Sentry Developer 0 € / Team 26 $/mes; Better Stack 0 € / 25 $ por 50 monitores extra; OpenStatus 0 € / 30 $/mes.

## 8. Checklist mínimo operable

Lo justo para decir "está controlado", sin ceremonia añadida:

- [ ] Backup diario automático, cifrado, fuera del servidor (S3 u equivalente) — §4.
- [ ] Replicación de base de datos, failover manual con un solo comando — §4.
- [ ] Un canal de alerta que te despierte de verdad para lo crítico (Pushover/Twilio o Better Stack) — §2.
- [ ] Uptime check externo, no solo métricas desde dentro del propio servidor — §1.
- [ ] Auto-restart de servicios caídos — §3.
- [ ] Secretos de producción en `sops` (o equivalente), no en texto plano ni fuera de git cifrado — §5.
- [ ] CI real en cada commit como red de seguridad de dependencias, con o sin bot de parcheo automático — §6.
- [ ] Gate de cobertura del diff + mutation testing ya cerrado en CI, no en producción — [[flujo-agentes-arquitectura]] §8.
- [ ] Migraciones de schema siempre en dos tareas (expandir / contraer) — [[flujo-agentes-arquitectura]] §15.

## Lo descartado y por qué

| Descartado | Motivo |
|---|---|
| PagerDuty / rotación de guardias formal | Sin ningún caso real de operador solo adoptándolo; el propio hilo de referencia lo rechaza en su respuesta mayoritaria |
| HashiCorp Vault | Descartado explícitamente por la fuente que recomienda `sops`, para menos de ~10 repos |
| Failover automático de base de datos | El propio operador de referencia lo descarta como "problema demasiado difícil"; failover manual con un comando es la práctica real |
| Grafana/Prometheus self-hosted | Sin evidencia de adopción real por operadores solos en lo investigado; aparece solo en contexto de equipo |
| Blue-green / feature flags para despliegue | Sin evidencia de adopción real a esta escala — ver [[flujo-agentes-arquitectura]] §15 |

## Dónde se ha buscado

`blog.healthchecks.io`, repo real `healthchecks/healthchecks` (vía `gh api`/`gh search`, incluido su historial de commits y ausencia de `dependabot.yml`), HN Algolia API (hilos [26203074](https://news.ycombinator.com/item?id=26203074), [23048715](https://news.ycombinator.com/item?id=23048715) y búsquedas adicionales), `sentry.io/pricing`, `betterstack.com/pricing`, `openstatus.dev/pricing`, `netdata.cloud/pricing`, `pushover.net` (precio no verificado, se omite). `WebSearch` de la sesión se agotó antes de empezar este frente (200/200) — resuelto con la API de HN y `gh api`/`gh search` sobre repos reales en vez de descubrimiento por buscador; no impidió ninguna de las conclusiones anteriores.

## Enlaces

- [[flujo-agentes-arquitectura]] — motor de ingeniería y §15, despliegue a producción
- [[desarrollo-agentes-f4-devsecops]] — controles del pipeline, distintos de esto
- [[capa-producto]] — cómo el usuario dirige el conjunto
- [[astillero]] — proyecto
- [[_index]]
