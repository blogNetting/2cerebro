---
title: Londres, patinetes, carné y pago del transporte
created: 2026-10-28
updated: 2026-10-28
tags: [viaje, londres, transporte, patinetes, tarjetas]
zona: general
---

Patinetes de alquiler en Londres (solo Lime y Voi, sin bonos ilimitados), carné que hay que llevar y cómo pagar todo el transporte al mínimo coste.

Parte de [[viaje-londres-diciembre-2026]]. Trayectos de aeropuerto en [[londres-transporte-aeropuertos]]. Informe completo con citas y enlaces: `/mnt/hgfs/2Cerebro/Londres-patinetes-carne-pago.html` (fuera del repo).

## Conclusiones (28-oct-2026)

- **Patinetes:** solo operan Lime y Voi ([TfL](https://tfl.gov.uk/modes/driving/electric-scooter-rental-trial): «The rental e-scooters are currently provided by 2 different operators», es decir, los patinetes de alquiler los ofrecen actualmente 2 empresas). Ningún bono es de días ilimitados. LimePass son minutos que caducan ([ayuda de Lime](https://help.li.me/hc/en-gb/articles/21280766615963-What-is-LimePass): «must be used before they expire», es decir, hay que gastarlos antes de que caduquen). Voi: desbloqueo a £0 y 27 p/min, según los datos oficiales de [voi.com/city/london](https://www.voi.com/city/london). Los precios de Voi+, de los paquetes de Voi y de LimePass no están publicados en ninguna web oficial: solo en la app. LimePrime: precio no publicado en li.me («Check in-app», consulta la app); la prensa da £6,99 al mes y £1,70 por viaje de hasta 20 minutos ([BikeBiz](https://bikebiz.com/lime-launches-limeprime-subscription-across-the-uk/)). Según la [web de LimePrime](https://www.li.me/limeprime), sus ventajas no funcionan en viajes internacionales. El blog de Voi para Reino Unido de 2023 daba un pase de día de hasta 200 minutos por £7 ([voi.com](https://www.voi.com/blog/save-pounds-and-pennies-with-voi-in-the-uk)); no está confirmado que siga vigente.
- **Contratos de Reino Unido** ([Voi](https://eu-central-1-shared-euc1-02.graphassets.com/AMcYXQcgeRNu5q0szWSm1z/cmnohugoj7kmt07vxx8shjmw1), [Lime](https://www.li.me/en-gb/limepass-terms-and-conditions)):
  - Seguro a terceros incluido: «there is no option to use a Vehicle without insurance» (no se puede usar un vehículo sin seguro).
  - Suscripciones sin permanencia y cancelables en cualquier momento. Voi+ y LimePrime no incluyen viajes: son un descuento a £1,70 por viaje.
  - Viaje máximo: 1 hora en Voi y 90 minutos en Lime.
  - Al acabarse el paquete de minutos, el viaje sigue a la tarifa normal.
  - Voi y Lime piden el carné en Londres, antes del primer viaje; desde España solo se crea la cuenta y se añade la tarjeta.
  - En los viajes en grupo de Voi, los invitados solo pueden ir en bici, no en patinete.
- **Carné:** la app miDGT solo vale en España ([DGT](https://sede.dgt.gob.es/export/sites/dgt/es/appmidgt/index.html)). El carné completo de la UE vale en Reino Unido ([gov.uk](https://www.gov.uk/guidance/e-scooter-trials-guidance-for-users)). Hay que llevar el de plástico: el art. 164 de la ley de tráfico británica de 1988 obliga a presentarlo si lo pide la policía.
- **Pago:** sin contacto, una tarjeta por persona y siempre la misma. Topes de 2026: £8,90 al día en zonas 1–2 y bus a £1,85 con tope de £5,55 desde el 1 de noviembre de 2026 ([TfL](https://content.tfl.gov.uk/adult-fares.pdf), [GLA](https://www.london.gov.uk/md3517-july-2026-fare-changes)). N26 cobra 0 % cualquier día. Revolut cobra 1 % en fin de semana ([tarifas de Revolut](https://cdn.revolut.com/terms_and_conditions/pdf/comisiones_personales_plan_est_ndar_66cc991f_1.7.4_1788272225_es.pdf)). Oyster y Visitor Oyster no compensan (£10,50 de alta que no se devuelven).
- **Bonos:** el London Pass no incluye transporte. El único bono de día ilimitado es el de las bicis de TfL (Santander Cycles), a £3,50.

- **Decisión de patinete (28-oct, con precios que pasó el usuario, sin verificar en la web oficial):** Voi gana casi siempre. Con viajes de 16 minutos o más, Voi+ (£2,99 al mes + £1,70 por viaje). Con viajes cortos, el bono de Voi que cubra los minutos: 120 por £15, 200 por £23 o 300 por £32. Flota: máximo 4.715 patinetes entre Lime y Voi, y unas 2.000 plazas de aparcamiento (fase 3 de TfL, septiembre de 2026).

## Cómo funciona la calculadora del HTML

El HTML abre solo con la calculadora. El resto (resumen, patinetes, carné, pagar transporte, ahorro, fuentes) va en pestañas de un menú superior: cambian con el ancla de la URL (`#calc`, `#carne`…), sin recargar.

- **Entradas:** días de uso, minutos al día y viajes al día (de ahí sale la duración media del viaje). Todo es por persona.
- **Coste de cada opción, por empresa:**
  - *Pago por uso:* desbloqueo × viajes + precio por minuto × minutos totales.
  - *Suscripción:* cuota + £/viaje × viajes, más el precio por minuto normal por los minutos que pasen de los incluidos en cada viaje.
  - *Un tipo de bono:* el viaje se parte en tramos de «validez» días (se usa la validez mínima, la conservadora). En cada tramo se elige cuántos bonos comprar; lo que no cubren se paga a la tarifa normal, con un desbloqueo por cada viaje nuevo.
  - *Mezcla de bonos:* solo con bonos cuya validez cubre todo el viaje.
  - Las opciones se ordenan de más barata a más cara y se marca la ganadora con su precio por minuto.
- **Escalera:** para cada empresa recorre de 1 a 180 min/día y agrupa los tramos con la misma opción más barata, sin suscripciones, con los días y la duración media de las barras.
- **Ajustes ocultos (⚙):** desbloqueo, precio por minuto, viaje máximo, suscripciones y bonos editables, con filas que se pueden añadir y borrar. Se guardan en el `localStorage` del navegador (almacenamiento local, por dispositivo) con la clave `londres-patinetes-precios-v1`. «Restaurar precios iniciales» borra esa clave.
- **Validado el 28-oct-2026 en Chrome real:**
  - A 1920 px y a 390 px (tamaño de iPhone): el menú queda en una fila y la página no se sale de la pantalla.
  - Sin errores de JavaScript en la consola.
  - Editar y guardar precios, añadir bonos y restaurar funcionan.
  - Dos resultados comprobados a mano: Voi+ a £30,19 y el pase de 200 de Lime a £48,79, con 4 días, 60 min/día y 4 viajes/día.

## Llegada, vuelta y moverse barato (añadido 28-oct-2026)

- **Heathrow T5 → piso:** metro Piccadilly + Jubilee, £5,90 por persona (tarifa punta por pasar por zona 1). Solo buses: £3,70, pero son 5 buses y 115 min. Ir sin pasar por la zona 1, saliendo en la estación del Overground de West Hampstead, costaría £2,60 (API de TfL), pero el viernes por la noche no hay trenes para hacerlo.
- **Piso → Gatwick:** Thameslink de las 02:20 pagando con la tarjeta en el torno, £10,70 fuera de hora punta, frente a £22,30 con billete. Según FlyerTalk (comunidad, no TfL), Gatwick no entra en el tope diario.
- **Por la ciudad:** solo bus, como mucho £5,55 al día; metro, como mucho £8,90. El patinete no es lo más barato para ir al centro (10 km). Sirve para saltos cortos ya en el centro.
- El HTML se publicó como Artifact privado. La copia que se publica es el mismo HTML sin las etiquetas `<html>`, `<head>` y `<body>` (las que envuelven la página), porque el visor las añade.

## Revisar

- Precios de los paquetes de Lime y Voi en la app, ya en Londres.
