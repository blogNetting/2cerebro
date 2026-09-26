---
title: Patrimonial
created: 2026-09-24
updated: 2026-09-25
tags: [finanzas, patrimonio, webapp, self-hosted, dashboard]
zona: tecnico
---

Patrimonial: aplicación web responsive, autoalojada en el servidor casero, para ver todo el patrimonio en un dashboard y gestionar el gasto a partir de extractos bancarios.

## Estado

Nombre decidido: **Patrimonial** (2026-09-24) — funciona en castellano como "sociedad patrimonial" (holding personal) y en inglés como adjetivo de *patrimony*, con acento natural en "mo". En fase de toma de requisitos (2026-09-25), modelando por entidades antes de decidir stack. Sin decisiones de stack ni de despliegue todavía.

**Pendiente al arrancar el diseño formal (framework spec-driven elegido, primera sesión de Opus):** el modelo de entidades y las fuentes de datos de esta nota se **trasladan** a `specs/patrimonial/spec.md` en `~/dev/patrimonial` (contrato K1 de [[flujo-agentes-arquitectura]]), no se duplican. Esta nota se resume entonces a hub — qué es, estado, enlace al repo, enlaces de vida no técnica ([[apartamentos-calle-uruguay]], [[fiscalidad-alquiler-por-habitaciones]]) — y deja de llevar el detalle técnico.

## Principio transversal: pasividad

Por defecto, si existe una fuente de solo lectura (API, dirección pública), se usa; la entrada manual es la excepción, no la norma. Se aplica de forma distinta según el activo: total en cripto, imposible gratis en banca (ver Fuentes), inexistente en inmuebles/pensiones.

## Diseño

Modo oscuro como requisito de UI desde el primer boceto. Pendiente: ¿dark-only o dark por defecto con opción a claro?

## Alcance

- Despliegue en el servidor casero. Web responsive (móvil y escritorio).
- Acceso protegido por contraseña. Si se expone fuera de la LAN, hace falta HTTPS real y valorar 2FA (pregunta abierta, ver más abajo).
- Dashboard con todo el patrimonio: inmuebles, cripto, plan de pensiones, otras inversiones, ahorro.
- Gestión del gasto: importar movimientos bancarios y analizarlos por categoría.

## Modelo de entidades

| Entidad | Qué es |
|---|---|
| Fuente | de dónde vienen los datos: API, dirección pública o manual |
| Activo | lo que se posee, con su tipo |
| Posición | cantidad de un activo en una fuente, ahora mismo |
| Transacción | movimiento con fecha: compra, venta, transferencia, ingreso, gasto |
| Categoría | clasificación de transacciones de gasto |
| Pasivo | deuda que resta del patrimonio neto (por decidir si entra) |
| Snapshot histórico | foto del patrimonio neto en una fecha, para evolución |

### Fuente — cerrado

- **Cripto**: lectura pasiva por API/dirección pública, solo lectura. Fuentes actuales (lista abierta, se amplía con el tiempo): wallet nativa XRP (dirección pública contra XRPL, sin key), Bitvavo (API con key de solo lectura), Tangem (wallet nativa multi-activo, direcciones públicas).
  - Credenciales: solo en el servidor (env/secreto), nunca en el repo ni el wiki. Scope de solo lectura al crear la key. IP-whitelist si el exchange lo permite. El navegador nunca ve la key, todo pasa por el backend.
  - Refresco: botón manual + automático mínimo diario (detalle de implementación pendiente).
  - Fallo de fuente: warning visible + último valor conocido con su fecha, nunca error mudo ni dato fresco falso.
- **Banco** (Abanca, CaixaBank): no existe forma gratuita de acceso automático para un particular — los agregadores PSD2 (GoCardless, Tink, Salt Edge, Plaid) son B2B, sales-gated, sin plan self-serve barato, y GoCardless cerró su tier gratuito a nuevos registros. Mecanismo elegido: exportación manual de **Norma 43 (N43)**, el estándar AEB soportado por Abanca y CaixaBank (y en general la banca española grande) — un solo parser para los dos bancos, sin mecanismos distintos por banco. Deduplicación por el **campo de referencia del movimiento** del propio fichero (clave: cuenta + referencia), sin hash ni cálculo añadido: si el banco da un ID, se usa directamente. El paso de exportar sigue siendo manual en cualquier formato (CSV, OFX o N43) — eso no lo resuelve el formato, así que se pueden descargar rangos de fechas solapados sin miedo a duplicar. Pendiente de comprobar contra un fichero N43 real de cada banco cuando se implemente (si la referencia viniera vacía o repetida, hace falta un plan B, no confirmado que vaya a pasar).
- **Inmuebles, plan de pensiones, otras inversiones**: manual puro, sin fuente externa posible.

## Preguntas abiertas

- Nombre y detalle de "otras inversiones": qué entra exactamente.
- Multiusuario: ¿solo tú, o también pareja/familia con su propio patrimonio?
- Acceso desde fuera de la LAN (internet) o solo dentro de casa/VPN — decide si hace falta 2FA.
- Patrimonio neto: ¿se restan pasivos (hipotecas, préstamos) o el dashboard es solo de activos?
- Histórico de evolución del patrimonio neto: ¿desde la v1 o se deja para más adelante?
- Rentabilidad del alquiler (ingresos - gastos) de los inmuebles alquilados: ¿entra en esta app o se queda en [[fiscalidad-alquiler-por-habitaciones]]?
- Categorización del gasto: manual, por reglas, o algo más automático.
- Saldo de cada cuenta: ¿sale de sumar los movimientos importados, o se sigue tecleando aparte?
- Backups de los datos del servidor casero.
- Dark-only o dark + toggle a claro.
- Stack y forma de despliegue.

## Enlaces

- [[apartamentos-calle-uruguay]] — inmueble que entraría en el dashboard
- [[fiscalidad-alquiler-por-habitaciones]] — ingresos y gastos del alquiler, candidatos a seguirse en la app
- [[astillero]] — sistema con el que se desarrollará; esta app es la candidata a primer piloto
- [[_index]]

## Repo

- **Repo:** [blogNetting/patrimonial](https://github.com/blogNetting/patrimonial), privado.
- **Ruta local:** `~/dev/patrimonial`.
- **Estado (2026-09-26):** arrancado con la skill `astillero-proyecto` y completo, fijado a **Astillero v0.2.2** (no `@main`: actualizaciones controladas, no automáticas). Se actualizó dos veces el mismo día porque Astillero se está desarrollando en paralelo, en otra sesión: `v0.1.0` tenía `AGENTS.md` con secciones vacías (corregido en `v0.2.1`); `v0.2.1` migró `AGENTS.md` al formato oficial [agents.md](https://agents.md/) (`v0.2.2`). Bootstrap de copier completo: `AGENTS.md`, `CODEOWNERS` con `@blogNetting`, `docs/contrato-tarea.md`, `docs/SECURITY.md`, plantilla de bug, workflows de ejecutor/rehacer/revisor/reconciliador/CI/despliegue, compilados con `gh aw compile --approve` tras verificar que el único secreto nuevo (`DEEPSEEK_API_KEY`) coincide con el diseño. Ecosistema **Python** (solo puertas de CI y red del ejecutor, no es el framework de la app). Secretos `DEEPSEEK_API_KEY` y `CLAUDE_CODE_OAUTH_TOKEN` configurados por el usuario. Sin código de aplicación todavía. Entrevista de diseño formal (K1) en curso — se resincronizará contra Astillero otra vez antes de cerrar la especificación, no en cada tag nuevo.
