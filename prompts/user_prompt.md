# User Prompt: mensaje de arranque

El `system_prompt.md` no viaja solo entre conversaciones distintas (no hay memoria compartida entre la app del celular y una sesión de Claude Code, por ejemplo). Por eso, cualquier conversación nueva donde se vaya a usar el agente arranca pegando este mensaje:

```
Actuá como mi asistente personal de finanzas. Estas son tus instrucciones (Rol, Contexto,
Tarea, Restricciones, Formato y Ejemplos), seguilas al pie de la letra:

[pegar el contenido completo de prompts/system_prompt.md]

A partir de ahora te voy a pasar gastos e ingresos por texto, foto de ticket, o link de
factura con QR. Extraé los datos, clasificalos, confirmame en una línea antes de cargar
nada, y cargalo en Airtable recién después de mi confirmación.
```

En la práctica, esto se resolvió de dos formas distintas (ver `DECISIONES.md`):

- **App de Claude del celular**: el mensaje de arranque vive como instrucciones persistentes del proyecto "Finanzas Personales" (se carga una sola vez al crear el proyecto, no hace falta repetirlo en cada chat nuevo dentro de ese proyecto).
- **Claude Code (terminal)**: se pega el bloque de arriba al principio de la sesión, porque ahí no hay concepto de "proyecto" con instrucciones persistentes. Esta superficie se usa solo para los links QR que necesitan `curl` y para mantener este mismo repositorio.

## Variantes: mensajes típicos del día a día

No son un template fijo (el punto 3 del contrato, Tarea, ya cubre cómo interpretarlos), pero estas son las formas reales en que llega la información:

- Texto libre y corto: `"Pagué $8.200 el gimnasio con débito hoy"`
- Foto de un ticket o factura (a veces con QR de factura virtual detallada, ej. Coto)
- Link directo a una factura virtual con QR
- Ingreso: `"cobré el sueldo, 850000"`
- Dato incompleto a propósito: `"gasté como 5 lucas en algo de la casa, no me acuerdo bien la fecha"`, el agente tiene que dejarlo marcado como "Revisar", no inventarlo (Restricciones, `system_prompt.md`).
