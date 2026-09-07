# Gobierno y riesgo

> "La pregunta profesional no es «¿puede hacerlo solo?» sino «¿qué pasa si lo hace mal, y quién firma?» La responsabilidad no se delega." (material de Clase 1)

## Qué sistemas toca el agente, y con qué permisos

| Sistema | Acceso | Alcance real vs. alcance contractual |
|---|---|---|
| Airtable (base "Finanzas Familia") | Conector de la cuenta: leer y crear/editar filas | El conector se autoriza a nivel de **cuenta de Airtable**, no de base individual (técnicamente el agente podría ver o tocar otras bases de la misma cuenta si existieran). El límite a "solo esta base" es una restricción del contrato (`system_prompt.md`, Restricciones), no un permiso de OAuth acotado. **Es una brecha de gobierno real, no resuelta**: hoy se sostiene por disciplina de prompt, no por control de acceso técnico. |
| Terminal (`curl`), solo en sesión de Claude Code | Ejecución de comandos de shell sin sandboxing adicional | Se usa acotado a un `curl http://` puntual para leer una factura QR, pero el permiso real de esa sesión es de terminal completa (podría leer o modificar cualquier archivo del filesystem, no solo hacer el fetch). Mismo patrón que el punto anterior: el uso está acotado por instrucción, no por permiso. |
| Dashboard / artefacto HTML (`Libro de Rubros`) | Lectura en vivo de Airtable desde el navegador, vía conector del usuario | Sin credenciales expuestas en el artefacto (usa el conector, no un token embebido). Este es el punto donde el diseño sí acota el acceso técnicamente, no solo por instrucción. |
| Rutina automática de recordatorio (pausada) | Lectura de `Presupuesto`/`Tarjetas` + envío de push | Pausada precisamente porque el canal de aviso (push) no se pudo confirmar como entregado (ver `DECISIONES.md` §4). Mientras esté pausada, el riesgo de "aviso silencioso perdido" es cero porque no corre. |

**Hallazgo confirmado con evidencia real** (corrida completa en [`corridas/03-nueva-categoria-panaderia.md`](corridas/03-nueva-categoria-panaderia.md), narrado en `DECISIONES.md` §6): la Restricción "el rubro siempre debe ser uno de la lista fija, nunca se inventa una categoría nueva" **no es un límite técnico del conector, es una convención de prompt**. El agente demostró en producción que puede modificar la estructura del campo `Rubro` (agregar una opción nueva) usando el parámetro `typecast` de la API de Airtable, cuando el usuario se lo pidió con la instrucción correcta. Esto es más permiso del que el contrato original asumía que existía: el "no puedo" que el agente respondió dos veces antes no era cierto en sentido estricto, solo reflejaba que no había probado esa vía.

## Qué puede salir mal, y qué pasa cuando sale mal

| Falla | Cómo se detecta | Qué pasa |
|---|---|---|
| El agente clasifica mal un rubro (ej. Delivery cargado como Salidas) | Revisión visual del usuario en la confirmación de una línea, antes de cargar | Si se detecta antes de confirmar, se corrige en el chat sin tocar Airtable. Si se confirma mal y se detecta después, hay que editarlo a mano en Airtable (el agente no borra ni corrige filas existentes por regla, salvo duplicado técnico confirmado). |
| Dato faltante por foto borrosa o mensaje ambiguo | El contrato prohíbe inventar: el campo queda vacío o "Revisar" | El usuario lo completa después, a mano o en una carga de seguimiento. Nunca hay un dato inventado silencioso: es el peor tipo de error en un sistema financiero, porque no se nota hasta que el total no cierra. |
| Confirmación reenviada por latencia del chat → carga duplicada | Ya pasó una vez (ver `DECISIONES.md` §5): se nota porque el total del mes no cierra o aparece una fila repetida | Corrección manual en Airtable. No hay salvaguarda automática todavía: es el riesgo operativo más concreto y sin resolver del sistema. |
| El conector de Airtable o de terminal se usa fuera del alcance que declara el contrato (ver tabla de arriba) | No hay detección automática: depende de que el agente respete la instrucción | Ningún control técnico lo impediría hoy. Mitigación actual: el alcance de la tarea es angosto (cargar filas, no administrar la cuenta), y el usuario es el único que interactúa con el agente. |
| Push de la rutina automática no llega (ya ocurrió) | El usuario nota que no recibió el aviso de un vencimiento | La rutina quedó pausada hasta resolver el canal de entrega, no se dejó corriendo "total, algo hace" (ver `DECISIONES.md` §4). |
| El agente modifica la estructura de la base (crear una opción de rubro) en vez de solo agregar filas, **ya ocurrió** | Se nota en el mensaje del agente ("con typecast pude crear la nueva opción...") y en que el campo `Rubro` tiene una opción nueva y permanente en Airtable | No se deshizo: la opción "Panadería" quedó creada. No hay control técnico que lo hubiera evitado, la única salvaguarda es que el agente lo transparentó en el chat en vez de hacerlo en silencio. Ver hallazgo arriba y `DECISIONES.md` §6. |

## Qué se revisa antes de confiar en una salida

- **Cada carga puntual**: la línea de confirmación en el chat, antes de que el agente escriba en Airtable. Este es el único punto de control humano obligatorio del sistema, y el contrato lo fuerza explícitamente (`system_prompt.md`, Restricciones: "Nunca cargar sin confirmación explícita").
- **El agregado mensual**: el dashboard de Airtable (no el chat) es donde el usuario audita el total del mes contra lo esperado, en una revisión periódica, no por transacción.
- **El código del artefacto y la estructura de la base**: cambios de infraestructura (agregar una pestaña, cambiar un gráfico, tocar el schema de una tabla) se revisan mirando el resultado renderizado antes de darlos por buenos, no hay tests automáticos, la verificación es visual y manual.

## Quién firma

El usuario firma cada carga real con su confirmación explícita en el chat: es una firma por transacción, no por lote ni por mes. El agente nunca tiene la última palabra sobre si un dato entra a la base: puede errar en la clasificación o en la extracción, pero no puede comprometer el registro sin ese "sí" puntual. La responsabilidad sobre las decisiones financieras que se toman mirando el dashboard (cuánto gastar, qué presupuesto fijar) es enteramente del usuario, el agente no opina ni recomienda, solo registra y agrega.

## Nivel de delegación (L0–L4, vocabulario del curso)

| Función | Nivel | Por qué |
|---|---|---|
| Registrar un gasto/ingreso puntual | **L1 · Proponer** | El agente extrae y redacta la confirmación, el humano aprueba cada carga, una por una, antes de que se ejecute. |
| Mantenimiento del dashboard/artefacto (agregar pestañas, gráficos, ajustar el schema) | **L2 · Ejecutar con revisión** | El agente trabaja solo construyendo o modificando el artefacto, el humano revisa el resultado final renderizado, no cada paso intermedio. Es el nivel por defecto de esta materia. |
| Recordatorio automático de vencimientos (diseñado, hoy pausado) | **L3 · Ejecutar y avisar** (nunca alcanzado en la práctica) | Estaba pensado para correr sola todos los días y avisar por push, el humano auditaría por muestreo. Se pausó porque el aviso (la mitad que hace de L3 algo seguro) nunca llegó confirmado al destinatario. |
| Cualquier decisión de gasto, ahorro o inversión | **No delegado (L0 como techo)** | El agente puede como mucho responder preguntas sobre los datos ya cargados, nunca decide ni recomienda qué hacer con la plata. |

No se usa **L4 (Autónomo)** en ningún punto: los datos son financieros reales, y un error no es "barato y reversible" (corregirlo significa editar a mano una fila en la base de la familia).
