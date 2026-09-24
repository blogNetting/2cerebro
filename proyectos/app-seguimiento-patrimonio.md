---
title: Patrimonial
created: 2026-09-24
updated: 2026-09-24
tags: [finanzas, patrimonio, webapp, self-hosted, dashboard]
zona: tecnico
---

Patrimonial: aplicación web responsive, autoalojada en el servidor casero, para ver todo el patrimonio en un dashboard y gestionar el gasto a partir de extractos bancarios.

## Estado

Idea abierta el 2026-09-24. Nombre decidido: **Patrimonial** (2026-09-24) — funciona en castellano como el término de "sociedad patrimonial" (holding personal) y en inglés como adjetivo de *patrimony*, con acento natural en "mo" que da el efecto buscado. Sin decisiones de stack ni de arquitectura todavía.

## Alcance

- Despliegue en el servidor casero. Web responsive (móvil y escritorio).
- Acceso protegido por contraseña.
- Dashboard con todo el patrimonio:
  - Inmuebles (real estate), con valor actual estimado.
  - Cripto, con valor actual.
  - Plan de pensiones.
  - Otras inversiones.
  - Ahorro.
- Gestión del gasto: importar extractos bancarios en Excel/CSV y analizarlos.

## Preguntas abiertas

- Fuentes de valoración de inmuebles y de cotización de cripto.
- Formato de los extractos de cada banco y cómo normalizarlos.
- Stack y forma de despliegue en el servidor casero.

## Enlaces

- [[apartamentos-calle-uruguay]] — inmueble que entraría en el dashboard
- [[fiscalidad-alquiler-por-habitaciones]] — ingresos y gastos del alquiler, candidatos a seguirse en la app
- [[sistema-desarrollo-con-agentes]] — sistema con el que se desarrollará; esta app es la candidata a primer piloto
- [[_index]]
