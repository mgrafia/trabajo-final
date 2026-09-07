# Corrida 1: Ticket con QR (Coto), carga + desglose de ítems

**Fecha de la corrida:** 27/08/2026 (gasto real del 26/08/2026)
**Fuente:** [conversación completa](https://claude.ai/share/1389fb52-ab0c-4504-9093-3a88de188f00), turnos del 27 ago

## Entrada 1

> Este Qr es una factura para que subas
> *(foto adjunta, con los dedos tapando parte del ticket)*

## Salida 1

> No pude leer varios datos clave porque el ticket está tapado por los dedos y no se ve el nombre del comercio. Esto es lo que sí puedo leer:
> - Total: $12.361,75 (dos ítems: 6.115,00 + 6.246,75)
> - Aparece "Vto.: 20260905" (podría ser vencimiento, no fecha de compra)
>
> Me falta para poder cargarlo: Comercio/rubro, Fecha de la compra, Medio de pago y tarjeta/banco. ¿Me confirmás estos tres datos, o me mandás la foto completa del ticket?

*(el usuario insiste con "accedé al QR": el agente responde que no puede decodificar un QR desde una foto ni navegar a un link sin que se lo den directamente, le pide la foto completa)*

## Entrada 2

*(foto del ticket completo, sin dedos tapando)*

## Salida 2

> Ahora sí, ticket completo. Esto es lo que leo:
>
> Fila a agregar → Gastos:
> 26/08/2026 | 41513,24 | Variable | Supermercado | Compra Coto (Av. Santa Fe 3760) | Crédito | Visa (xxxx omitido)
>
> ¿Confirmás para cargarlo así?

## Entrada 3

> si

## Salida 3

> Ahora sí, creo la fila en Gastos.
> *(usa la herramienta de Airtable: crea fila nueva)*
>
> Cargado ✅
>
> Gastos — 26/08/2026 | $41.513,24 | Variable | Supermercado | Compra Coto (Av. Santa Fe 3760) | Crédito | Visa (xxxx omitido)
>
> Ojo: el ticket también trae el detalle de los ítems comprados (churrasquito, arroz, jugo, queso, etc.) que podrían ir en la tabla "Items Supermercado" si querés ese nivel de detalle. ¿Lo cargo también, o con la fila de Gastos alcanza?

## Entrada 4

> dale

## Salida 4

> Cargo el detalle de ítems, vinculado al gasto de Coto. Así los voy a clasificar (algunos nombres del ticket son ambiguos, marqué mi mejor interpretación):
>
> | Producto | Cant. | Precio unit. | Categoría |
> |---|---|---|---|
> | Churrasquito de cerdo | 0,875 kg | $12.000 | Carnicería |
> | Arroz largo Finodos Hnos 1kg | 1 | $1.770 | Almacén |
> | Jugo Manzana del Valle 1L | 1 | $2.100 | Bebidas |
> | Dicomere | 1 | $2.700 | Otros (no identifico el producto) |
> | Chorizo cerdo/vacío Costanzo 400g | 1 | $5.800 | Carnicería |
> | Pas. Entrenuts | 1 | $4.399,99 | Otros (no identifico el producto) |
> | Queso blanco light Santa Clara 300g | 1 | $3.125 | Lácteos |
> | Queso rallado Ilolay 120g (2x1) | 2 | $6.115 | Lácteos |
> | Aguila | 1 | $11.250 | Bebidas (asumo cerveza; avisame si no) |
>
> ¿Cargo así o corregís algo antes?

## Entrada 5

> la linea dicomere es harina de avena y Pas Entrenuts es pasta de mani

## Salida 5

> Listo, cargados los 9 ítems en "Items Supermercado" corrigiendo Dicomere → Harina de avena y Pas. Entrenuts → Pasta de maní, ambos en categoría Almacén, y todos vinculados al gasto de Coto del 26/08.

## Verificación posterior (mismo día, vía `curl` al link del QR)

El usuario pasó después el link directo de la factura virtual (`http://apps01.coto.com.ar/TicketMobile/Ticket/...`) "no para que lo subas, sino para corroborar". El agente accedió con la herramienta de fetch y confirmó:

> Sí, accedí. El total coincide perfecto con lo cargado: $41.513,24, 26/08/26, pagado con tarjeta crédito/débito. (...) Aguila = Chocolate amargo 70% cacao Águila 130g (no era bebida, era chocolate) — ¿corrijo la categoría de Bebidas a Almacén?

El usuario respondió que ya lo había corregido él mismo, el dato quedó consistente con la factura oficial sin intervención adicional del agente.

## Qué muestra esta corrida

- El contrato en acción: el agente no inventa el comercio ni la fecha cuando la foto está tapada, pide más información en vez de asumir (Restricciones, `system_prompt.md`).
- El punto de control humano (Restricciones): nada se carga en Airtable sin el "sí" explícito, incluso cuando los datos ya están claros.
- El mecanismo de QR con `curl` funcionando de verdad sobre un link real `http://`, tal como está documentado en `system_prompt.md` (Contexto/Restricciones).
