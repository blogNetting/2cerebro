---
title: Rentabilidad inmobiliaria con datos
created: 2026-09-29
updated: 2026-09-29
tags: [inmobiliario, inversion, datos, rentabilidad]
zona: tecnico
---

Hub del proyecto: buscar buenas rentabilidades en inversión inmobiliaria apoyándose en datos, usando el modelo de negocio de Prophero como una hipótesis a contrastar, no como premisa.

## Objetivo

Montar un método propio, basado en datos, para identificar inmuebles con buena rentabilidad de inversión. Sin alcance de investigación cerrado todavía — pendiente de que el usuario lo confirme antes de lanzar más búsqueda (ver Estado).

## Qué es Prophero (verificado, 2026-09-29)

Proptech de inversión inmobiliaria en España, activa desde 2021. Modelo "llave en mano": el cliente compra un inmueble físico a su nombre (no participaciones) y Prophero se encarga de búsqueda, negociación, reforma, amueblado y gestión del alquiler. Dice usar análisis de datos ("+250 variables", "más de 80 millones de datos") para elegir zonas — [www.prophero.com/es/resultados](https://www.prophero.com/es/resultados/), fuente vendor.

- **Comisiones**: 7.500 € de servicio (1.500 € inicial + 3.000 € al seleccionar inmueble + 3.000 € tras compra/reforma/alquiler) + 5-7% anual sobre la renta de gestión — [help.prophero.com/modelo-de-negocio](https://help.prophero.com/es/doc-center/modelo-de-negocio), fuente vendor; comisiones confirmadas de forma independiente en [vivirdeinmuebles.com/prophero-opiniones](https://vivirdeinmuebles.com/prophero-opiniones/).
- **Entrada mínima**: 100.000 € (o 25.000 € con su sistema de "tickets").
- **Rentabilidad que anuncian**: 6-7% neto anual en alquiler residencial — mismo rango que confirma el análisis independiente de vivirdeinmuebles.com con un caso real ("Rentabilidad neta = 6.000€ / 100.000€ = 6% neto").
- **Opiniones de clientes, mixtas**: Trustpilot 3,5-4,0/5 sobre 200-255 reseñas ([trustpilot.com/review/prophero.es](https://www.trustpilot.com/review/prophero.es)). Recurrente en positivas: gestión llave en mano cómoda. Recurrente en negativas y en foros (Rankia, foro Balio, Burbuja.info): retrasos largos en reforma y puesta en alquiler (casos de 6-8 meses sin ingresos), cambios constantes de gestor, costes totales por encima de lo esperado.
- **Referencia oficial de mercado**: el Banco de España sitúa la rentabilidad bruta media del alquiler en España en el 3,0% a finales de 2025 (Informe de Estabilidad Financiera, otoño 2025) — por debajo de lo que anuncia Prophero, lo que sugiere que su cifra se basa en una selección de inmuebles concreta, no en el mercado medio.

Sin verificar todavía: si el "análisis de datos" es una ventaja real y diferencial frente a competidores, o principalmente argumento comercial; qué parte de su resultado depende de negociación/escala (no replicable por un particular) frente a qué parte es puramente selección por datos (sí replicable).

## Fuentes disponibles

- `fuentes/prophero-transcripciones-2026-09-29/` — 8 transcripciones de vídeos de YouTube sobre Prophero e inversión inmobiliaria, aportadas por el usuario. **Sin ingerir todavía**: pendiente de extraer citas textuales y contrastarlas con lo de arriba.

## Estado

2026-09-29: proyecto recreado tras un primer intento cerrado sin confirmar alcance (ver `areas/decisiones.md`, entrada del mismo día). Lo de arriba sobre Prophero ya está verificado con fuente y cita. **No se lanza más investigación (transcripciones, competidores, criterio de rentabilidad propio) hasta que el usuario confirme el alcance.**

## Próximos pasos

- Ingerir las 8 transcripciones de `fuentes/prophero-transcripciones-2026-09-29/`: extraer afirmaciones con cita textual, contrastarlas con lo ya verificado arriba.
- Confirmar con el usuario el alcance antes de seguir: ¿solo entender Prophero, o construir un método propio de scoring con datos públicos?

## Relacionado en el wiki

- [[apartamentos-calle-uruguay]] — inversión inmobiliaria ya en marcha del usuario; referencia de contexto, no parte de este método
- [[fiscalidad-alquiler-por-habitaciones]] — fiscalidad del alquiler que cualquier cálculo de rentabilidad neta tiene que incorporar
- [[patrimonial]] — dashboard de patrimonio donde acabaría reflejándose cualquier inmueble que resulte de este proyecto
