# DECISIONES: historia del proceso

Este archivo cuenta cómo llegó el sistema a su forma actual: iteraciones reales del contrato y del producto, qué falló, qué se achicó y por qué. La fuente primaria es el historial de commits del repo [`gestor-de-finanzas-personales`](https://github.com/mgrafia/gestor-de-finanzas-personales) del que este trabajo final se empaquetó (ver "Empaquetado para la entrega" al final).

## 1 · De la idea al sistema base

El punto de partida fue un problema propio: llevar gastos e ingresos familiares sin depender de una planilla que nadie actualiza. La primera decisión de diseño fue **dónde vive el dato**: no en la conversación (se pierde, no es consultable), sino en una base de Airtable real, conectada como conector persistente de la cuenta. El chat es la interfaz de carga, no el almacenamiento.

Segunda decisión: **qué tan estructurado tiene que ser el registro**. Se definieron cinco tablas desde el arranque (`Gastos`, `Ingresos`, `Presupuesto`, `Items Supermercado`, `Tarjetas`) y una lista fija de rubros (la alternativa, rubros libres que el agente define sobre la marcha, se descartó explícitamente porque un rubro "inventado" por conversación revienta cualquier reporte agregado más adelante). El costo de esta rigidez: agregar un rubro nuevo requiere un paso manual (avisarle a Claude la primera vez que se carga un gasto real de esa categoría, para que lo use como opción del campo `Rubro`, que es `singleSelect`).

## 2 · El dashboard: iteración visual real

El primer artefacto (`Libro de Rubros`) se lanzó con dos pestañas (Gastos del mes / Presupuesto) y creció por partes: `Consumo` (para separar gasto recurrente de puntual), `Vencimientos`, `Gráficos`. La parte más iterada fue justamente los gráficos:

- Se armó un gráfico de torta por rubro.
- Se **reemplazó por un gráfico de barras de cumplimiento de presupuesto** (la torta no dejaba ver qué rubro se había pasado del monto esperado, que es la pregunta real que importa mes a mes).
- Después se **trajo de vuelta el donut**, pero como complemento (no reemplazo) de las barras de cumplimiento: el donut responde "¿en qué se fue la plata este mes?" (todos los rubros con gasto, tengan o no presupuesto cargado) y las barras responden "¿me pasé del presupuesto?" (solo rubros con presupuesto). Son dos preguntas distintas, el error inicial fue tratarlas como si una reemplazara a la otra.
- La evolución mensual pasó de línea a barras apiladas, y se agregó Ingresos al lado para poder comparar de un vistazo (ajuste motivado por legibilidad en el celular, no solo estética).

**Falla real, encontrada y corregida:** el layout de dos columnas no colapsaba a una sola en mobile. Se detectó probando en el celular real (no en el navegador de escritorio) y se corrigió aparte.

## 3 · La función que se agregó y se sacó: Cartera

Se agregó una pestaña `Cartera` (posiciones de inversión agrupadas por broker) como paso hacia el objetivo de largo plazo del README ("cartera de inversiones" + "proyección de ingresos futuros"). **Se revirtió el mismo día, a pedido del usuario.** No quedó registrado en el commit el motivo puntual, pero la señal es clara: se construyó antes de que estuviera claro cómo se iba a cargar y mantener ese dato (¿manual? ¿con cotización en vivo?), y una función a medias en el dashboard principal es peor que no tenerla. Queda en el Roadmap como pendiente, no como feature activa.

## 4 · La automatización que no funcionó

Se documentó y probó una rutina automática de recordatorio de vencimientos (Fijos y Tarjetas) corriendo todos los días a las 9:00, con notificación push al celular. **La lógica funcionó**: se probó con datos reales, detectó correctamente los vencimientos, y la API de la rutina confirmó "Mobile push requested". **El push nunca llegó al celular.** Se investigó (permisos de notificación de la app, hipótesis de que las rutinas de `claude.ai/code` no tienen canal de entrega hacia la app de chat estándar) sin resolverlo, y la rutina quedó **pausada** en vez de borrada: la lógica es reusable en cuanto se resuelva el canal de entrega. Es la falla más honesta del proyecto: un L2 completo (agente ejecuta solo, avisa) que se cayó no en la lógica del agente sino en la integración de plataforma.

## 5 · Reglas que quedaron sin resolver a propósito

- **Medio de pago por transferencia** (ej. Mercado Pago): las tres opciones actuales (Efectivo, Débito, Crédito) no cubren este caso. Se dejó sin resolver en vez de forzar una clasificación que no es ni débito ni crédito, un dato mal clasificado es peor que un campo pendiente.
- **Incidente de duplicación**: si el chat del celular queda "iterando" después de una confirmación, reenviar el "Sí" puede generar una carga duplicada (pasó una vez con una compra de supermercado: gasto + 9 ítems duplicados, corregido a mano). No se resolvió con una regla automática, quedó documentado como advertencia de uso, porque la causa es de latencia de la interfaz, no del contrato del agente.

## 6 · El límite que no era técnico: `typecast` y la lista fija de rubros

El contrato asume que "el rubro siempre debe ser uno de la lista fija" es un límite técnico: el agente no tiene forma de crear una opción nueva en el campo de Airtable. Eso se sostuvo dos veces (al pedir "Transferencia" como medio de pago, y al pedir "Panadería" como rubro): el agente respondió que no tenía permiso ni herramienta para modificar la estructura de la base.

**La tercera vez, con una instrucción distinta ("Definí una nueva categoría..."), el agente encontró y usó el parámetro `typecast` de la API de Airtable, que sí permite crear una opción nueva en un campo de selección al escribir un valor que no existe todavía.** Ver la corrida completa en [`corridas/03-nueva-categoria-panaderia.md`](corridas/03-nueva-categoria-panaderia.md).

Esto no fue el agente desobedeciendo el contrato: hizo exactamente lo que se le pidió, con la herramienta que tenía disponible, y avisó con transparencia lo que había hecho. **El error fue de diseño del contrato**: se documentó una restricción de negocio (los rubros son una lista cerrada, a propósito, para no romper los reportes agregados) como si fuera un límite técnico de permisos, y no lo es. La lista fija sigue siendo la regla correcta para el día a día, pero ya no se puede asumir que es infranqueable por diseño, depende de que el modelo no use `typecast`, lo cual es una salvaguarda mucho más débil de lo que el contrato original daba a entender. Queda como pendiente de gobierno real en `GOBIERNO_Y_RIESGO.md`, no como algo resuelto.

## 7 · Empaquetado para la entrega del trabajo final

El sistema se construyó y se usa de verdad en el repositorio [`gestor-de-finanzas-personales`](https://github.com/mgrafia/gestor-de-finanzas-personales), sin la estructura que pide la consigna del trabajo final (README libre en vez de estándar, sin `prompts/`, sin `corridas/`, sin este mismo archivo). Para la entrega se decidió **no reescribir el sistema**, sino empaquetarlo:

1. Se duplicó el repo completo (con historial de commits) a `trabajo-final`.
2. Se repuntó el remoto a un repositorio propio (`github.com/mgrafia/trabajo-final`), separado del proyecto original en uso.
3. Se extrajo el contrato implícito que ya vivía disperso en el README (rol, reglas, schema) a `prompts/system_prompt.md` y `prompts/user_prompt.md`, siguiendo el framework de seis piezas del curso (Rol, Contexto, Tarea, Restricciones, Formato, Ejemplos): no se inventó un contrato nuevo, se formalizó el que ya estaba operando.
4. El README se reescribió al formato estándar de la materia, el detalle técnico que no entra en ese formato (schema completo de Airtable, descripción de cada pestaña del dashboard) se movió a `prompts/system_prompt.md` y queda linkeado desde el README.

Esta misma decisión, documentar el empaquetado en vez de esconderlo, es intencional: el trabajo real no empezó el día de la entrega, y ocultar eso hubiera sido menos honesto que mostrar la costura.
