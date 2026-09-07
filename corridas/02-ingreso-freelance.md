# Corrida 2: Ingreso ambiguo, el agente pregunta en vez de asumir

**Fecha de la corrida:** "ayer" respecto del 06/09/2026 en la conversación → 01/09/2026 (fecha real del ingreso)
**Fuente:** [conversación completa](https://claude.ai/share/1389fb52-ab0c-4504-9093-3a88de188f00)

## Entrada 1

> Tuve este ingreso, más tarde te paso mi número de factura
> *(foto adjunta de un comprobante de transferencia)*

## Salida 1

> Nuevo ingreso a cargar en Ingresos:
>
> 01/09/2026 | 1615799,03 | (Fuente sin especificar) | Transferencia de [nombre del remitente omitido] - "Varios" (Op. [numero omitido])
>
> El concepto dice "Varios", así que no puedo asumir la fuente — ¿es Freelance, Otros, o cuál de la lista?

## Entrada 2

> Es freelance, un cliente que paga abonos mensuales por asesoría comercial, el monto es variable

## Salida 2

> Cargo el ingreso:
> *(usa la herramienta de Airtable: crea fila nueva)*
>
> Cargado ✅
>
> Ingresos — 01/09/2026 | $1.615.799,03 | Freelance | Transferencia de [nombre del remitente omitido] - abono mensual asesoría comercial (Op. [numero omitido])
>
> Avisame cuando tengas el número de factura y lo agrego a la descripción.

## Qué muestra esta corrida

Esta es la corrida más simple y más representativa de la Restricción "ante un dato ambiguo, preguntar en vez de asumir" (`system_prompt.md`): el comprobante decía literalmente "Varios" como concepto, y el agente no adivinó entre las seis fuentes de ingreso válidas, esperó la respuesta del usuario. También muestra que el monto grande y con centavos ($1.615.799,03) se extrajo sin redondeo forzado ni error, directamente de la imagen.
