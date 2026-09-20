---
title: Buscar vuelos en Vueling por URL directa
created: 2026-09-19
updated: 2026-09-19
tags: [vueling, vuelos, playwright, cdp, entorno]
zona: tecnico
---

Método que funcionó para barrer un mes de vuelos Vueling sin usar el autocomplete de su web: URLs directas del calendario y del buscador, y extracción por JS.

## Método

1. `WebSearch` y `WebFetch` no bastan: dan precios sueltos y desactualizados, sin horarios reales. Hay que escalar a CDP (ver [[entorno]]).
2. Calendario de precios de Vueling por URL directa, que barre un mes completo por ruta:
   `https://www.vueling.com/es/calendario-de-precios?originCalendar=ORIGEN&destinationCalendar=DESTINO&dateComplete=DD/MM/YYYY`
   Da el precio mínimo de cada día del mes. Ojo: ese mínimo puede corresponder a un vuelo que no cumple la franja horaria pedida; hay que verificar vuelo a vuelo.
3. Resultados reales, vuelo a vuelo (horarios y precio), one-way, por fecha exacta:
   `https://tickets.vueling.com/booking/flightSearch?o=ORIGEN&d=DESTINO&dd=YYYY-MM-DD&adt=1&chd=0&inf=0&c=es-ES&cur=EUR&tt=OW`
   Extraer por JS (`browser_evaluate`) sobre `.vy-flight-selector` y `.vy-flight-journey_hour`: cuesta mucho menos contexto que un snapshot completo.
4. Para confirmar que ninguna otra aerolínea cubre la ruta: Google Flights (`google.com/travel/flights?q=...`).
5. No usar el input de origen/destino de la home (`#originInput` y similares): es un autocomplete Angular poco cooperativo con automatización, y los eventos sintéticos no abren el desplegable de forma fiable. Construir las URLs directas de arriba.

## Cuidado con las restricciones de llegada

«Salida a partir de las X» no implica sin límite superior: hay que comprobar también la hora de llegada frente al último tren o bus. Ese error invalidó dos recomendaciones en la búsqueda real, ver [[vuelos-sevilla-octubre-2026]].

## Enlaces

- Búsqueda concreta donde se aplicó: [[vuelos-sevilla-octubre-2026]].
- Herramientas y escalado web: [[entorno]].
