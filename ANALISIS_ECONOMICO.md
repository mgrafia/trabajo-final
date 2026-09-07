# Análisis económico

## Aclaración de partida

Hoy el sistema corre bajo una suscripción de Claude (costo fijo mensual, no por token), no facturado por uso de API. Lo que sigue modela **cuánto costaría si corriera "en serio" facturado por token vía la API de Anthropic** (que es la pregunta que pide la consigna, y la misma lógica que se usó en clase para el ejemplo del informe semanal automatizado, Clase 2: "la cuenta que importa"). Precios vigentes (por millón de tokens, entrada/salida):

| Modelo | Entrada | Salida |
|---|---|---|
| Claude Opus 5 (frontier, referencia) | \$5,00 | \$25,00 |
| Claude Sonnet 5 (el que efectivamente corre hoy en la app) | \$2,00 | \$10,00 |
| Claude Haiku 4.5 (liviano) | \$1,00 | \$5,00 |

## Qué cuesta una corrida

Se definen dos tipos de corrida, según lo que realmente varía en el volumen de tokens:

**1 · Carga simple** (texto libre o foto de ticket sin desglose, la mayoría de las corridas): extracción + clasificación + confirmación + escritura en Airtable.
- Entrada: ~800 tokens (mensaje del usuario + contexto reciente de la conversación + definición de las herramientas de Airtable disponibles).
- Salida: ~200 tokens (línea de confirmación + llamada a la herramienta de Airtable + acuse breve).

**2 · Carga con desglose de factura QR** (ej. ticket de Coto con detalle ítem por ítem, vía `curl`): el contenido de la factura fetcheada entra como contexto adicional.
- Entrada: ~3.000 tokens (HTML/detalle de la factura + instrucciones).
- Salida: ~500 tokens (fila de `Gastos` + N filas de `Items Supermercado` estructuradas).

Costo por corrida:

| Corrida | Opus 5 | Sonnet 5 | Haiku 4.5 |
|---|---|---|---|
| Simple | \$0,009 | \$0,0036 | \$0,0018 |
| Con desglose QR | \$0,0275 | \$0,011 | \$0,0055 |

## Proyección de uso real

Volumen estimado a partir del uso real documentado (gastos e ingresos de una familia, más 1-2 compras de supermercado con ticket detallado por semana):

- ~5 cargas simples por día → 35/semana
- ~1,5 cargas con desglose QR por semana

| | Por semana | Por año (×52) |
|---|---|---|
| Opus 5 | \$0,356 | **\$18,53** |
| Sonnet 5 | \$0,143 | **\$7,41** |
| Haiku 4.5 | \$0,071 | **\$3,70** |

## Elección de modelo, justificada

El criterio del curso es **el modelo más chico que hace bien la tarea**. Acá el costo en dólares no es lo que decide (a este volumen personal, hasta el modelo frontier cuesta menos de USD 20 al año), lo que decide es qué tan exigente es la tarea:

- **Clasificar y extraer datos de un mensaje corto y bien especificado** (monto, rubro de una lista fija, fecha) es exactamente el caso de "tarea repetitiva bien especificada" que la tabla de la Clase 2 asigna a los modelos livianos. **Recomendación: Claude Haiku 4.5** para la carga del día a día (texto libre e ingresos).
- **Interpretar una foto de ticket** (OCR de una imagen real, a veces borrosa, con montos y comercios no siempre legibles) es un caso más ambiguo. Mientras no haya evidencia propia de que Haiku 4.5 lo resuelve con la misma confiabilidad que un modelo más grande en fotos difíciles, conviene mantener **Sonnet 5** para esa ruta específica: es el modelo que efectivamente corre hoy en la app, y no hay incidentes de mala lectura de foto registrados en `DECISIONES.md`, así que bajar de modelo ahí es un cambio a validar, no a asumir.
- **Opus 5** (o más) solo tendría sentido para las funciones todavía no construidas que exigen razonamiento real (proyección de ingresos combinando sueldo + cartera, o cualquier análisis que compare escenarios), nunca para la carga rutinaria.

En números: pasar toda la carga simple de Sonnet 5 a Haiku 4.5 ahorra ~\$3,70 de los ~\$7,41 anuales estimados (la mitad del costo total, sin tocar la ruta de fotos). Es un ahorro real pero pequeño en términos absolutos, el valor del ejercicio es el criterio (no pagar por razonamiento que la tarea no necesita), más que el monto.

## Qué no está incluido en esta cuenta

- El costo de las sesiones de Claude Code usadas para mantener la infraestructura del proyecto (el dashboard, este mismo repositorio): es trabajo de desarrollo, no una "corrida" del agente en producción, y no escala con el uso diario.
- El caching de prompts (contexto estable como el system prompt, cacheado entre turnos de una misma conversación): usarlo bajaría el costo de entrada de cada corrida, pero requiere medirlo con tráfico real en vez de estimarlo, queda como próxima optimización, no como número ya aplicado acá.
