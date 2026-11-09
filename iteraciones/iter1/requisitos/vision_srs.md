Planificación iterativa con RUP y modelos UML

Guía de laboratorio del tema 6

Curso  Modelado y Desarrollo de Aplicaciones  CIIN1322PProyecto  Sistema de reservas y préstamos de biblioteca

Objetivo

Planificar las fases de inicio, elaboración, construcción y transición de un sistema de biblioteca, asignando entregables por iteración y verificando la coherencia entre la Visión, la SRS breve, los casos de uso y los modelos UML.

I Datos y desarrollo del procedimiento

La línea base iter1 define el problema, seis requisitos funcionales, cuatro requisitos no funcionales y el diseño preliminar del módulo de reservas. La decisión principal es tratar la disponibilidad como un cambio transaccional, porque dos solicitudes pueden competir por el mismo ejemplar. La implementación y las pruebas ejecutables se distribuyen en las iteraciones posteriores.

1 Documento de Visión versión 1

Problema. El registro manual de reservas y préstamos puede producir duplicidad de asignaciones, consultas desactualizadas y poca visibilidad de las reservas vencidas. El sistema propuesto centraliza la disponibilidad por ejemplar y conserva el historial de cada operación.

Usuarios interesados. El estudiante consulta y reserva; el bibliotecario entrega y recibe ejemplares; el responsable de biblioteca valida las reglas y acepta las entregas. Un temporizador del sistema procesa vencimientos.

Alcance. Autenticación por rol, consulta del catálogo, reserva de ejemplares, entrega contra reserva, devolución, cancelación y vencimiento. No se incluyen pagos, multas, compras, préstamo entre sedes ni integración con sistemas universitarios.

Objetivos de negocio. OBJ01: impedir asignaciones activas incompatibles de un ejemplar. OBJ02: permitir consultar disponibilidad y reservar sin intervención del bibliotecario. OBJ03: registrar quién realizó cada cambio y cuándo.

Criterio de éxito. La aceptación exige cubrir los seis requisitos funcionales, verificar exclusión mutua e idempotencia y demostrar que el catálogo refleja el estado confirmado. Los umbrales de calidad de la SRS se validarán durante construcción y transición.

2 Especificación breve de requisitos

SRS versión 1. El producto es una aplicación web con una base de datos transaccional. El catálogo y las cuentas se cargan previamente para el piloto. Los valores de cupo y plazos siguientes son reglas propuestas para este proyecto académico y requieren validación del responsable de biblioteca.

| ID | Requisito y criterio de aceptación |
| --- | --- |
| RF01 | Autenticar a estudiantes y bibliotecarios. Una sesión inválida no puede modificar reservas, préstamos ni devoluciones; cada rol solo accede a las operaciones autorizadas. |
| RF02 | Consultar libros por título o autor y mostrar ejemplares disponibles. Un ejemplar RESERVADO o PRESTADO no se ofrece como disponible. |
| RF03 | Reservar un ejemplar disponible para un estudiante habilitado. Al confirmar, generar código y venceEn; crear Reserva ACTIVA y Ejemplar RESERVADO en una sola transacción. |
| RF04 | Registrar la entrega de una reserva ACTIVA y vigente, solo por bibliotecario. Crear Préstamo ACTIVO, cambiar Reserva a ATENDIDA y Ejemplar a PRESTADO en la misma transacción. |
| RF05 | Registrar la devolución, solo por bibliotecario. Cambiar Préstamo a DEVUELTO, registrar fechaRetorno y liberar el ejemplar sin duplicar efectos si se repite la operación. |
| RF06 | Cancelar una reserva propia y vigente o vencerla automáticamente. Pasar ACTIVA a CANCELADA o VENCIDA y liberar el ejemplar de forma atómica. Conservar el historial. |

Reglas de negocio

RN01. Solo un usuario ACTIVO reserva; la suma de reservas ACTIVA y préstamos ACTIVO de ese usuario no supera 3. La entrega sustituye una reserva por un préstamo y no incrementa esa suma.RN02. Una reserva vence a las 24 horas exactas de su creación. Entregar o cancelar requiere ahora < venceEn; vencer requiere ahora ≥ venceEn. Se usa el reloj del servidor.RN03. Cada ejemplar admite como máximo una asignación activa: una reserva o un préstamo, nunca ambas. Las solicitudes concurrentes se serializan y revalidan dentro de la transacción.RN04. El préstamo dura 7 días calendario. La devolución no cobra multas.RN05. La pareja usuarioId y solicitudId identifica un intento de reserva; un reintento confirmado devuelve el mismo código, incluso si se perdió la respuesta original.

| ID | Requisito no funcional y verificación prevista |
| --- | --- |
| RNF01 | Seguridad: control por rol, transporte HTTPS y contraseñas con hash adaptativo. Verificar operaciones prohibidas por rol y ausencia de contraseñas legibles. |
| RNF02 | Rendimiento: percentil 95 de consulta y reserva ≤ 2 s con 50 sesiones concurrentes y 10 000 ejemplares, medido durante 5 min sin fallos inducidos. |
| RNF03 | Integridad: cero asignaciones incompatibles y cero reservas duplicadas al competir 20 solicitudes por un ejemplar y al repetir solicitudId. |
| RNF04 | Auditoría: cada cambio confirmado registra actor, entidad, acción y fecha del servidor. Comparar registros de auditoría con reservas, préstamos y devoluciones. |