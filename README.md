# Gestor de Finanzas Personales

## Qué construí

Un agente que lleva el registro de gastos e ingresos diarios de una familia (a mano, por foto de ticket, o por link de factura virtual con QR) clasificándolos en una base de Airtable real ("Finanzas Familia") y armando un dashboard de proyección mensual (ingresos vs. gastos fijos vs. variables). Es para uso propio, todos los días, desde la app de Claude del celular. El detalle completo del contrato (schema de datos, rubros, reglas) vive en [`prompts/system_prompt.md`](prompts/system_prompt.md), este README es el resumen.

## Cómo se lo pedí

El contrato completo, textual, está en [`prompts/system_prompt.md`](prompts/system_prompt.md) y [`prompts/user_prompt.md`](prompts/user_prompt.md) (framework de seis piezas del curso: Rol, Contexto, Tarea, Restricciones, Formato, Ejemplos). Las líneas centrales de cada pieza, citadas tal cual:

1. **Rol**: "Sos el asistente personal de finanzas de la familia. Tu trabajo es llevar el registro de gastos e ingresos diarios en una base de Airtable (...) No sos un asesor financiero: no das recomendaciones de inversión ni juicios sobre el presupuesto, solo registrás y organizás datos con precisión."
2. **Contexto**: "Base de Airtable 'Finanzas Familia' (...) Tablas: `Gastos`, `Ingresos`, `Presupuesto`, `Items Supermercado`, `Tarjetas` (...) Moneda: pesos argentinos (ARS), redondeados al peso."
3. **Tarea**: "1. Extraer fecha, monto, rubro o fuente, descripción, medio de pago y tarjeta/banco (...) 4. Confirmar los datos extraídos en el chat, en una línea de texto simple. 5. Recién después de que el usuario confirme, cargar la fila nueva."
4. **Restricciones**: "Nunca inventar un dato faltante (...) El rubro siempre tiene que ser uno de la lista fija (...) Nunca cargar una fila en Airtable sin la confirmación explícita del usuario en el chat."
5. **Formato**: "Confirmación en el chat (antes de cargar): una línea de texto simple. `Gasto: \$14.500 · Supermercado (Variable) · Coto · Débito · 05/09/2026`"
6. **Ejemplos**: cuatro casos completos en `prompts/system_prompt.md` (gasto simple, ticket con QR, ingreso, dato incompleto marcado "Revisar").

El mensaje real que arrancó el contrato en producción (27/08/2026, primera línea de la [conversación real](https://claude.ai/share/1389fb52-ab0c-4504-9093-3a88de188f00) que documentan las `corridas/`) es la versión anterior, más corta, de este mismo contrato. `system_prompt.md` es la formalización posterior de esas mismas seis piezas, no una reescritura de cero (ver `DECISIONES.md` §1 y §7).

El mensaje de arranque de conversación (`prompts/user_prompt.md`) resuelve un problema real: el contrato no viaja solo entre superficies (app del celular vs. Claude Code), así que hay que repetirlo o dejarlo como instrucción persistente del proyecto.

## Qué funciona

- Carga de gastos e ingresos por texto, foto de ticket, e ingresos por monto declarado, probado con corridas reales (ver [`corridas/`](corridas/)).
- Desglose de tickets de supermercado con QR de factura virtual (ej. Coto) vía `curl` en una sesión de Claude Code, cuando el link solo responde por `http://`.
- Dashboard en vivo (**Libro de Rubros**, [`artifacts/libro-de-rubros.html`](artifacts/libro-de-rubros.html)): gasto del mes por rubro contra presupuesto, gestión de presupuesto inline, análisis de consumo recurrente vs. puntual, vencimientos de gastos fijos y tarjetas, y tres gráficos (evolución mensual, distribución por rubro, cumplimiento de presupuesto).
- Reglas de no-invención de datos, probadas en la práctica: un dato ambiguo queda marcado "Revisar" en vez de completarse solo.

## Qué falta o qué falló

- **El push de la rutina automática de recordatorio de vencimientos nunca llegó al celular**, a pesar de que la lógica se probó con datos reales y la API confirmó el envío. Quedó pausada (ver [`DECISIONES.md`](DECISIONES.md) §4).
- **Se agregó y se revirtió una pestaña de Cartera de inversiones** el mismo día: se construyó antes de tener resuelto cómo cargar y mantener ese dato (ver `DECISIONES.md` §3).
- **Incidente de duplicación**: reenviar una confirmación mientras el chat está "iterando" generó una carga duplicada una vez (gasto + 9 ítems). Corregido a mano, sin salvaguarda automática todavía.
- **Medio de pago por transferencia** (ej. Mercado Pago) no encaja en las tres opciones actuales (Efectivo/Débito/Crédito), sin resolver a propósito.
- **Brecha de permisos no resuelta**: el conector de Airtable se autoriza a nivel de cuenta completa, no de la base "Finanzas Familia" puntual. El límite hoy es una regla del contrato, no un permiso técnico acotado. Ver [`GOBIERNO_Y_RIESGO.md`](GOBIERNO_Y_RIESGO.md).
- **Cartera de inversiones y proyección de ingresos futuros** (sueldo + cartera): en el roadmap, no construido.

## Qué aprendí

Un contrato bueno no se escribe de una: el de este sistema se formó por uso real durante semanas (a mano, en el README) antes de existir como `system_prompt.md` explícito para esta entrega. Formalizarlo después fue mucho más fácil que si lo hubiera intentado escribir del todo antes de usar el sistema ni una vez. También aprendí que la parte más frágil de un agente con herramientas reales no es el modelo de lenguaje, sino la integración de plataforma (el push que nunca llegó no fue una falla de razonamiento, fue un canal de entrega roto), y que documentar esa falla en vez de esconderla o seguir "como si funcionara" es lo que hace confiable al resto del sistema. Por último, la disciplina de "nunca inventar, dejar Revisar" resultó más valiosa que cualquier regla de clasificación fina: en finanzas, un dato vacío se nota y se corrige, un dato inventado se filtra al total y no se nota hasta que ya es tarde.

## Estructura del repositorio

- [`prompts/`](prompts/) (contrato del agente: system prompt + user prompt)
- [`corridas/`](corridas/) (evidencia de corridas reales: entrada, salida, fecha)
- [`DECISIONES.md`](DECISIONES.md) (historia del proceso: iteraciones, fallas, cambios de alcance)
- [`ANALISIS_ECONOMICO.md`](ANALISIS_ECONOMICO.md) (costo por corrida, proyección anual, elección de modelo)
- [`GOBIERNO_Y_RIESGO.md`](GOBIERNO_Y_RIESGO.md) (permisos, riesgos, supervisión, niveles de delegación L0–L4)
- [`artifacts/`](artifacts/) (el dashboard vivo: Libro de Rubros)
