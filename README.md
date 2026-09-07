# Gestor de Finanzas Personales

Asistente personal de finanzas basado en Claude, que lleva el registro de gastos e ingresos diarios (a mano o por foto de ticket) en una base de Airtable, y arma un dashboard de proyección mensual (ingresos vs. gastos fijos vs. variables) tipo contador de familia.

## Rol y funcionamiento

Claude actúa como asistente personal de finanzas: recibe gastos e ingresos por texto o foto de ticket/factura (incluyendo links de factura virtual vía QR), extrae los datos, clasifica cada gasto en un rubro fijo y lo carga como fila nueva en la base de Airtable **"Finanzas Familia"**. Antes de cargar, siempre confirma los datos en el chat.

## Estructura de datos (Airtable)

### Tabla `Gastos`
| Fecha | Monto | Tipo (Fijo/Variable) | Rubro | Descripción | Medio de pago | Tarjeta/Banco |

### Tabla `Ingresos`
| Fecha | Monto | Fuente | Descripción |

### Tabla `Presupuesto`
| Rubro | Grupo (Ingreso/Fijo/Variable) | Monto esperado mensual | Mes de referencia |

Refleja el monto objetivo *actual* por rubro (no histórico mes a mes, para no consumir el límite de registros del plan gratuito de Airtable). Se edita a mano para planificar meses futuros.

### Tabla `Items Supermercado`
| Producto | Fecha | Cantidad | Precio unitario | Categoría | Gasto relacionado (link a Gastos) |

Detalle ítem por ítem de las compras de supermercado (cuando hay ticket/factura con detalle disponible), para detectar consumo recurrente vs. puntual y proyectar/optimizar compras futuras.

## Rubros

**Gastos Fijos:** Alquiler, Expensas, Luz, Gas, Agua, Internet, Psicóloga, Limpieza, Universidad, Obra social, Suscripciones, Fijo Extra (Disponible) 1 a 10.

**Gastos Variables:** Supermercado, Gimnasio, Salidas Pareja, Salidas Solo, Ocio/Cine, Delivery, Transporte/Nafta, Farmacia, Regalos, Ropa, Mantenimiento Hogar.

**Fuentes de Ingreso:** Sueldo, Aguinaldo, Bonus, Rentas/Dividendos, Freelance, Otros.

## Reglas

- Montos siempre en pesos argentinos (ARS).
- El rubro siempre debe ser uno de la lista fija; si un gasto variable no encaja, se marca "Revisar". Un "Fijo Extra (Disponible) N" solo se usa si el usuario indica explícitamente a qué gasto fijo nuevo corresponde.
- Si falta un dato (monto, fecha, comercio, medio de pago, tarjeta) por foto borrosa o dato no provisto, se deja vacío o "revisar" — nunca se inventa.
- Si no se especifica la fecha, se usa la fecha del día de carga.
- Ante un monto ambiguo, se pregunta en vez de asumir.
- No se modifican ni borran filas existentes, solo se agregan filas nuevas (salvo corrección de duplicados por error técnico).
- El resumen mensual se recalcula con cada carga nueva.

## Facturas con código QR

Muchos tickets (ej. Coto) imprimen un QR que enlaza a una factura virtual con el detalle completo de la compra. Ese link a veces solo responde por `http://` plano (sin TLS), lo cual falla con herramientas de web-fetch que fuerzan `https://`. Cuando eso pasa, se resuelve pidiendo el link directamente en una sesión con acceso a terminal (`curl`).

## Dashboard

Interface de Airtable **"Dashboard Finanzas"** → página **"Resumen Mensual"**, con:
- Total de gastos e ingresos del mes corriente (filtrado por mes calendario).
- Gasto por rubro y por tipo (Fijo/Variable).
- Ingresos por fuente.
- Detalle de gastos del mes, agrupado por rubro.
- Tabla de Presupuesto editable, para planificar montos de meses futuros.

## Roadmap

- [x] Registro de gastos e ingresos (texto y foto de ticket).
- [x] Base de Airtable con Gastos, Ingresos, Presupuesto e Items Supermercado.
- [x] Dashboard mensual (gastos del mes, ingresos del mes, presupuesto editable).
- [ ] Análisis de consumo recurrente vs. puntual (Items Supermercado) para optimizar compras.
- [ ] Cartera de inversiones (acciones, ONs, bonos) cargada manualmente, con cotización actualizada al consultar.
- [ ] Proyección de ingresos futuros combinando sueldo + cartera.
