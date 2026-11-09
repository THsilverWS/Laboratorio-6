3 Casos de uso y excepciones

CU03 Reservar ejemplar

Precondiciones. Sesión de estudiante válida; ejemplar existente. La disponibilidad vista en pantalla es informativa y se vuelve a comprobar al confirmar. Si solicitudId ya fue confirmado, se devuelve la misma reserva antes de evaluar el cupo actual.

Flujo principal. 1) El estudiante elige un ejemplar. 2) La pantalla envía ejemplarId y solicitudId. 3) El servicio valida estado del usuario y cupo. 4) Dentro de una transacción bloquea primero el usuario y luego el ejemplar, y revalida cupo, disponibilidad e idempotencia. 5) Crea la reserva ACTIVA con venceEn y cambia el ejemplar a RESERVADO. 6) Confirma la transacción y muestra código y vencimiento.

A1 Usuario bloqueado o cupo agotado. Rechazar sin crear reserva. A2 Competencia por ejemplar o cupo. Revalidar bajo bloqueo; solo la operación que conserve las reglas confirma. A3 Fallo de persistencia. Revertir todos los cambios. A4 Respuesta perdida tras confirmar. Reintentar con el mismo solicitudId y devolver el código existente. Si otra solicitud con ese identificador se confirma mientras se esperaba el bloqueo, se recupera ese resultado.

Postcondición de éxito. Existe una reserva ACTIVA vinculada al usuario y al ejemplar; el ejemplar está RESERVADO. Postcondición de fallo. No se crean cambios parciales ni una segunda reserva para el mismo intento.

Contratos de los demás casos de uso

CU01 y CU02. La autenticación produce una sesión con rol; el catálogo consulta el estado confirmado de los ejemplares. Una búsqueda sin coincidencias devuelve una lista vacía.

CU04. El bibliotecario verifica que la reserva siga ACTIVA y no haya vencido. Crea el préstamo a 7 días y actualiza los tres estados de forma atómica. Si ya venció o fue cancelada, rechaza la entrega y actualiza la vista.

CU05. Se localiza el préstamo ACTIVO y se confirma la recepción. La devolución repetida devuelve el resultado previo y no crea otra devolución. Si no existe un préstamo asociado, se muestra un error de validación.

CU06. Cancelar exige titularidad y vigencia; el temporizador vence desde venceEn. Ambas operaciones compiten con CU04 usando cambio condicional desde ACTIVA. Si otra transición ganó, se devuelve el estado actual y no se libera un ejemplar ya prestado.

4 Roadmap de iteraciones por fase RUP

Horizonte propuesto de seis semanas, con una iteración de una semana. Los roles se asignan a integrantes al iniciar el proyecto: AN analista, AR arquitecto, DE desarrollador, QA responsable de pruebas y LP líder del proyecto. Una persona puede cubrir varios roles; la revisión por pares debe realizarla otra persona.

Hito de inicio LCO. Se alcanza al acordar objetivos, alcance y viabilidad. La línea base documental de I1 permite revisar ese acuerdo; no equivale por sí sola a la aceptación del responsable de biblioteca. LCA, IOC y PR son puertas de decisión de las fases siguientes.

Acciones de mitigación y seguimiento

5 Matriz fase y artefactos

RUP organiza el ciclo en cuatro fases [2]. Las disciplinas se repiten con distinta intensidad: el análisis, diseño, pruebas y gestión de cambios continúan en cada iteración. Los modelos de esta entrega son una anticipación de diseño para revisar riesgos; se consolidan en elaboración.

Coherencia de vocabulario

Libro es el título bibliográfico; Ejemplar es la unidad física que se asigna. Reserva representa el derecho temporal de retiro; Prestamo representa la salida física. La disponibilidad pertenece a Ejemplar y se deriva de sus asignaciones vigentes. Esta distinción evita reservar un título sin identificar qué copia se retira.

Notación. Se emplea UML para actividad, secuencia, clases y estados. El catálogo de CU de la sección 3 define los actores y objetivos. La guía admite UML Activity para el proceso crítico; por ello no se necesita duplicarlo en BPMN. La referencia normativa de UML se incluye en [3].

6 Diagrama de actividad del módulo crítico

ACT01 · CU03 · RF03 · RN01 a RN03. El camino exitoso y los rechazos por reglas, competencia y fallo técnico comparten una única política: no dejar cambios parciales.

Figura 1  Actividad de reservar un ejemplar con validación y rollback

La comprobación inicial mejora la respuesta al usuario. La comprobación bajo bloqueo decide la asignación efectiva. Los reintentos ya confirmados se resuelven antes de entrar a este flujo de creación, como se detalla en SEQ01 y RN05.

7 Diagrama de secuencia de reserva

SEQ01 · CU03 · RF03 · RNF03. Pantalla recibe la solicitud; ServicioReserva aplica las reglas y coordina la transacción; BD conserva los estados confirmados. Los fragmentos alt representan resultados excluyentes.

Figura 2  Interacciones y alternativas de CU03 Reservar ejemplar

El orden de bloqueo es usuario y luego ejemplar. Esto serializa solicitudes del mismo usuario para impedir superar el cupo, aunque seleccionen copias distintas. La transacción también verifica la unicidad de usuarioId y solicitudId antes de crear la reserva.

8 Diagrama de clases del dominio

CLA01 · RF02 a RF06. Las multiplicidades 0..* incluyen el historial: varias reservas pueden referirse al mismo ejemplar a lo largo del tiempo. La restricción de actividad limita la asignación vigente.

Figura 3  Entidades y asociaciones del sistema de biblioteca

Restricciones del modelo

Un Libro agrupa cero o más Ejemplar; cada Ejemplar pertenece a un Libro. Cada Reserva y Prestamo pertenece a un Usuario y un Ejemplar. Una Reserva puede originar como máximo un Prestamo y cada Prestamo procede de una Reserva en el flujo de esta versión.

La asociación Reserva–Prestamo se muestra opcional durante el ciclo de vida. Restricción adicional: un Prestamo existente debe tener una Reserva ATENDIDA; una Reserva ATENDIDA tiene exactamente un Prestamo. El límite 0..1 evita duplicar entregas de la misma reserva.

Invariantes. Por ejemplar existe como máximo una Reserva ACTIVA o un Prestamo ACTIVO, nunca ambos. RESERVADO implica una Reserva ACTIVA; PRESTADO implica un Prestamo ACTIVO; DISPONIBLE implica ausencia de ambos. La pareja usuarioId y solicitudId es única en Reserva.

Los atributos de auditoría se implementarán en un registro separado que identifique entidad, actor, acción y momento; se omiten de este modelo de dominio para mantener legible la estructura principal.

9 Diagrama de estados de Reserva

EST01 · CU04 y CU06 · RF04 y RF06. ACTIVA es el único estado desde el cual se entrega, cancela o vence una reserva. Los estados de cierre son terminales para esa reserva y conservan la información histórica.

Figura 4  Ciclo de vida de Reserva con guardas y efectos

Excepciones y compensaciones

En el instante exacto de vencimiento solo corresponde vencer: la guarda ahora ≥ venceEn excluye entregar y cancelar. Si el temporizador aún no ejecutó la tarea, CU04 igualmente rechaza una reserva fuera de plazo; no depende del refresco de la pantalla.

Entrega, cancelación y vencimiento verifican que el estado previo siga siendo ACTIVA. La operación que pierde una carrera no altera el ejemplar. Así, una cancelación tardía no puede liberar una copia que otra operación acaba de prestar.

Si falla la transacción de entrega, se revierten el préstamo, la reserva y el ejemplar juntos. Si el fallo ocurre antes de confirmar una reserva nueva, se revierte la creación. Si el cliente pierde la respuesta después de confirmar, se recupera el resultado con el mismo solicitudId.

Relación con las devoluciones

La devolución pertenece al ciclo de vida de Prestamo, no reactiva la Reserva ATENDIDA. CU05 cambia Prestamo de ACTIVO a DEVUELTO y Ejemplar de PRESTADO a DISPONIBLE. El nuevo préstamo requerirá una nueva reserva.

10 Matriz de trazabilidad

Cada fila enlaza el objetivo de la Visión con el requisito, su caso de uso, los modelos pertinentes y la prueba de aceptación. El signo de diseño cubierto significa que existe especificación; no acredita una ejecución de software.

Cobertura documental

Los 6 de 6 requisitos funcionales tienen caso de uso y prueba prevista: cobertura de trazabilidad funcional del 100 %. Los 4 de 4 requisitos no funcionales tienen criterio verificable y escenario asociado. Esto mide enlaces documentales; la cobertura de implementación y de ejecución se evaluará en I3 a I5.

Dependencias de consistencia

RF03 y RF06 comparten Ejemplar y Reserva. RF04 añade Prestamo y cierra la reserva sin liberar el ejemplar. RF05 libera la copia al devolverla. Cualquier cambio de cupo, plazo o transición debe actualizar SRS, CU, modelos y pruebas en el mismo cambio de versión.

Ejemplo de propagación. Si se amplía el plazo de reserva a 48 horas, se actualizan RN02, venceEn en CU03, las guardas de EST01, los escenarios CP06, CP08 y CP10 y el manual del piloto. No basta cambiar el texto de la pantalla.

11 Escenarios de aceptación

Los casos siguientes son diseños de prueba para las iteraciones de construcción. Sus resultados esperados se derivan de la SRS; no se presentan como mediciones ni pruebas ejecutadas sobre una aplicación implementada.

Evidencia requerida al ejecutar. Registrar versión, datos de entrada, hora del servidor, resultado observado, estado de BD y veredicto. Para carga, adjuntar percentiles y tasa de errores; para concurrencia, contar asignaciones activas después de cada corrida.

12 Revisión de coherencia y hallazgos

Se realizó una revisión interna de los contratos y diagramas de esta línea base. La revisión independiente por un compañero debe usar el mismo checklist, registrar su identidad y aportar conformidad o nuevos hallazgos antes de la entrega académica.

Registro de hallazgos y correcciones

Cerrado en diseño indica que el contrato fue corregido y contrastado con los modelos. La eficacia de la corrección requiere las pruebas ejecutables de I2 a I5. Revisor par: ____________________  Fecha: __________  Firma o conformidad: ____________________.