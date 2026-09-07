# Gestor de Finanzas Personales

Asistente personal de finanzas basado en Claude, que lleva el registro de gastos e ingresos diarios (a mano, por foto de ticket, o por link de factura virtual) en una base de Airtable, y arma un dashboard de proyección mensual (ingresos vs. gastos fijos vs. variables) tipo contador de familia. El objetivo final es que se convierta en la aplicación de cabecera para las finanzas personales, incluyendo a futuro presupuesto proyectado y cartera de inversiones.

## Rol y funcionamiento

Claude actúa como asistente personal de finanzas: recibe gastos e ingresos por texto, foto de ticket/factura, o link de factura virtual (QR), extrae los datos, clasifica cada gasto en un rubro fijo y lo carga como fila nueva en la base de Airtable **"Finanzas Familia"**. Antes de cargar, siempre confirma los datos en el chat en una línea de texto simple, para poder corregir algo si hace falta.

## Dónde vive esto

- **Datos**: base de Airtable **"Finanzas Familia"**, conectada como conector de la cuenta de Claude (no depende de ninguna conversación puntual ni de que una computadora esté prendida).
- **Uso diario** (carga de gastos/ingresos a mano o con foto): conversación normal en la **app de Claude del celular**, proyecto "Finanzas Personales".
- **Links de factura virtual con QR** (ej. Coto): se procesan en una sesión de **Claude Code** con acceso a terminal, porque el fetch de esos links necesita `curl` (ver más abajo por qué). Esta misma sesión es también donde se arma y mantiene la infraestructura del proyecto (estructura de la base, dashboard, este repo).
- **Documentación**: este repo. No guarda datos personales, solo la definición del sistema (estructura, reglas, roadmap).

Cualquier conversación nueva que se abra necesita recibir el contexto de este README (rol, estructura, reglas) para poder operar — no se transmite solo automáticamente entre conversaciones distintas.

## Estructura de datos (Airtable)

### Tabla `Gastos`
| Fecha | Monto | Tipo (Fijo/Variable) | Rubro | Descripción | Medio de pago | Tarjeta/Banco |

### Tabla `Ingresos`
| Fecha | Monto | Fuente | Descripción |

### Tabla `Presupuesto`
| Rubro | Grupo (Ingreso/Fijo/Variable) | Monto esperado mensual | Mes de referencia |

Refleja el monto objetivo *actual* por rubro (no histórico mes a mes, para no consumir el límite de registros del plan gratuito de Airtable). Se edita a mano para planificar meses futuros — es la tabla que se usa para armar presupuesto.

### Tabla `Items Supermercado`
| Producto | Fecha | Cantidad | Precio unitario | Categoría | Gasto relacionado (link a Gastos) |

Detalle ítem por ítem de las compras de supermercado (cuando hay ticket/factura con detalle disponible, sea de un ticket completo o de un producto suelto), para detectar consumo recurrente vs. puntual y proyectar/optimizar compras futuras. Cualquier gasto de Rubro = Supermercado debería tener también su detalle acá, no solo el total en `Gastos`.

## Rubros

**Gastos Fijos:** Alquiler, Expensas, Luz, Gas, Agua, Internet, Psicóloga, Limpieza, Universidad, Obra social, Suscripciones, Fijo Extra (Disponible) 1 a 10.

**Gastos Variables:** Supermercado, Gimnasio, Salidas Pareja, Salidas Solo, Ocio/Cine, Delivery, Transporte/Nafta, Farmacia, Regalos, Ropa, Mantenimiento Hogar. Si un gasto variable no encaja en ninguno, se usa el rubro **Revisar**.

**Fuentes de Ingreso:** Sueldo, Aguinaldo, Bonus, Rentas/Dividendos, Freelance, Otros.

No se agregan rubros "Extra (Disponible)" para variables (a diferencia de fijos) — quedaron descartados a propósito.

## Reglas

- Montos siempre en pesos argentinos (ARS), redondeados al peso.
- El rubro siempre debe ser uno de la lista fija; nunca se inventa una categoría nueva. Un "Fijo Extra (Disponible) N" solo se usa si el usuario indica explícitamente a qué gasto fijo nuevo corresponde.
- Si falta un dato (monto, fecha, comercio, medio de pago, tarjeta) por foto borrosa o dato no provisto, se deja vacío o "revisar" — nunca se inventa.
- Si no se especifica la fecha, se usa la fecha del día de carga.
- Ante un monto ambiguo, se pregunta en vez de asumir.
- El medio de pago solo tiene tres opciones: Efectivo, Débito, Crédito. **Pendiente de definir**: cómo catalogar pagos por transferencia (ej. Mercado Pago) — hoy no encajan bien en ninguna de las tres.
- No se modifican ni borran filas existentes, solo se agregan filas nuevas (salvo corrección de duplicados por error técnico, ver "Incidentes conocidos").
- El resumen mensual se recalcula con cada carga nueva, no hace falta pedirlo.

## Facturas con código QR

Muchos tickets (ej. Coto) imprimen un QR que enlaza a una factura virtual con el detalle completo de la compra (producto, cantidad, precio, descuentos). A diferencia del QR fiscal de AFIP (que solo trae CUIT/monto/tipo de comprobante), este sí sirve para desglosar la compra en `Items Supermercado`.

Ese link a veces solo responde por `http://` plano (sin TLS) — es el caso de Coto. Las herramientas de web-fetch estándar fuerzan `https://` y fallan (`connect ECONNREFUSED`). La solución es pedir el contenido con `curl` directo por `http://`, algo que solo está disponible en una sesión con terminal (Claude Code), no en la app de Claude del celular. Por eso esos links se mandan en esa sesión y no en el chat de uso diario.

## Dashboard

Interface de Airtable **"Dashboard Finanzas"** → página **"Resumen Mensual"**, con:
- Total de gastos e ingresos del mes corriente (filtrado por mes calendario).
- Gasto por rubro y por tipo (Fijo/Variable).
- Ingresos por fuente.
- Detalle de gastos del mes, agrupado por rubro.
- Tabla de Presupuesto editable, para planificar montos de meses futuros.

Se ve tanto desde la web de Airtable como desde su app mobile.

## Artefactos

**Libro de Rubros** (`artifacts/libro-de-rubros.html`) — panel con dos pestañas, conectado en vivo a Airtable desde el navegador vía la capability `mcp` de Claude (usa el conector "Airtable" del usuario, sin exponer ningún token):

- **Gastos del mes**: navegación mes a mes (no todo mezclado), con ingresos/gastos/saldo del mes, gasto real por rubro comparado contra lo presupuestado (Fijos y Variables, con barra de avance y aviso si se pasó), y el listado de movimientos reales de `Gastos` de ese mes.
- **Presupuesto**: gestión de la tabla `Presupuesto` — ver los rubros de Ingresos/Fijos/Variables agrupados con su monto esperado, editarlo inline, borrar un rubro, o agregar uno nuevo (ej. "Monotributo" como gasto fijo).

Un rubro agregado en la pestaña Presupuesto queda como línea de presupuesto; para que también aparezca como opción en el campo Rubro de `Gastos` (es un singleSelect de opciones fijas) hay que decírselo a Claude la primera vez que se cargue un gasto real de esa categoría.

## Incidentes conocidos

- **Duplicación de registros**: si en el chat del celular la respuesta de Claude se queda "iterando" mucho tiempo después de confirmar una carga, puede ser que la carga ya se haya hecho y el reintento (reenviar "Sí" o un mensaje de más) genere un registro duplicado. Ya pasó una vez con una compra de supermercado (gasto + 9 items duplicados) y se corrigió a mano. Recomendación: esperar y revisar Airtable antes de reenviar la confirmación.

## Roadmap

- [x] Registro de gastos e ingresos (texto, foto de ticket, y link de factura QR).
- [x] Base de Airtable con Gastos, Ingresos, Presupuesto e Items Supermercado.
- [x] Dashboard mensual (gastos del mes, ingresos del mes, presupuesto editable).
- [x] Artefacto "Libro de Rubros" para gestionar categorías y presupuesto en vivo.
- [ ] Definir cómo catalogar pagos por transferencia en Medio de pago.
- [ ] Análisis de consumo recurrente vs. puntual (Items Supermercado) para optimizar compras.
- [ ] Cartera de inversiones (acciones, ONs, bonos) cargada manualmente, con cotización actualizada al consultar.
- [ ] Proyección de ingresos futuros combinando sueldo + cartera.
