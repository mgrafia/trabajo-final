# Corridas reales

Las tres corridas de acá abajo son extractos textuales de una conversación real y completa entre Martín y el agente, del 27/8/2026 al 6/9/2026, en la app de Claude del celular (proyecto "Finanzas Personales"). La conversación completa, con todos los turnos, no solo los tres extraídos acá, está pública en:

**https://claude.ai/share/1389fb52-ab0c-4504-9093-3a88de188f00**

Esa es la fuente primaria para reconstruir cualquier corrida en su contexto completo (incluye, entre otras, más de una docena de cargas reales: tickets de supermercado con y sin desglose, ingresos, deliveries, y el intercambio donde se estableció el contrato original por primera vez).

## Índice

1. [`01-ticket-qr-coto.md`](01-ticket-qr-coto.md): 26-27/08/2026. Foto de ticket con QR → carga en `Gastos` + desglose completo en `Items Supermercado`, con dos correcciones del usuario en el camino.
2. [`02-ingreso-freelance.md`](02-ingreso-freelance.md): 01/09/2026. Ingreso ambiguo ("Varios") → el agente pregunta en vez de asumir la fuente, según Restricciones del contrato.
3. [`03-nueva-categoria-panaderia.md`](03-nueva-categoria-panaderia.md): 06/09/2026. Gasto en un rubro que no existe en la lista fija → primero el agente dice que no puede crear categorías nuevas (como ya había pasado con "Transferencia"), pero termina creando la opción "Panadería" en Airtable vía `typecast` cuando el usuario insiste. **Hallazgo no documentado en el contrato original**: contradice la Restricción de "el rubro siempre debe ser uno de la lista fija" (ver `DECISIONES.md` §6 y `GOBIERNO_Y_RIESGO.md`).
