# Diseño de base de datos — AulaNexo

| Campo | Valor |
|---|---|
| Documento | Database Design |
| Versión | 0.1 — propuesta inicial |
| Motor | PostgreSQL |
| Capa de acceso | Sequelize ORM |
| Estado | Modelo lógico preliminar; sujeto a validación de Sprint 0 |

## 1. Objetivo

Definir el modelo relacional que soportará la información académica, administrativa y financiera del MVP de AulaNexo. El diseño separa identidad de acceso y perfiles de negocio, representa las relaciones entre alumnos y tutores, y utiliza claves foráneas, restricciones únicas e índices para proteger la integridad.

El modelo es una línea base conceptual. Los tipos SQL, longitudes, precisión decimal, nombres definitivos y reglas de borrado deberán confirmarse durante el diseño físico y las migraciones.

## 2. Convenciones

- Nombres de tablas y columnas en snake_case.
- Claves primarias UUID o BIGINT; la elección final deberá ser consistente en toda la aplicación. Se recomienda UUID para identificadores expuestos en APIs y BIGINT también es válido para un MVP local.
- Timestamps almacenados de forma consistente; se recomienda TIMESTAMPTZ en PostgreSQL.
- Montos monetarios almacenados como DECIMAL/NUMERIC, nunca FLOAT.
- Calificaciones almacenadas como DECIMAL/NUMERIC con precisión definida.
- Claves foráneas explícitas.
- Restricciones UNIQUE para evitar duplicados de negocio.
- Los estados usarán valores controlados mediante ENUM o VARCHAR con CHECK; se seleccionará una estrategia única por dominio.
- Campos created_at y updated_at donde corresponda.
- Las bajas lógicas utilizarán is_active o deleted_at solo cuando el historial o las relaciones requieran conservar el registro.

## 3. Dominios y entidades

### 3.1 Identidad y autenticación

#### users

Representa las cuentas que pueden autenticarse.

| Campo | Descripción |
|---|---|
| id | PK |
| email | Correo de acceso; UNIQUE |
| password_hash | Hash de contraseña |
| role | ADMIN, DOCENTE, TUTOR o ALUMNO |
| is_active | Cuenta habilitada |
| last_login_at | Último inicio de sesión |
| created_at / updated_at | Auditoría técnica |

Reglas:
- Nunca guardar contraseñas en texto plano.
- El correo debe ser único normalizado.
- La cuenta inactiva no podrá autenticarse.
- El rol es la clasificación principal de autorización; las relaciones de negocio determinan el alcance sobre recursos.

#### sessions

Almacena sesiones persistentes del servidor para express-session con connect-pg-simple.

| Campo | Descripción |
|---|---|
| sid | Identificador de sesión; PK |
| sess | Datos serializados de sesión |
| expire | Fecha de expiración |

La tabla puede ser creada y gestionada por el adaptador de sesiones. Se excluirá de los modelos de negocio ordinarios y no se expondrá por la API.

### 3.2 Personas y perfiles

#### students

| Campo | Descripción |
|---|---|
| id | PK |
| user_id | FK users.id, opcional según decisión de acceso del alumno |
| student_number | Matrícula ficticia, UNIQUE |
| first_name | Nombre |
| last_name | Apellidos |
| birth_date | Fecha de nacimiento ficticia |
| is_active | Estado del registro |
| created_at / updated_at | Timestamps |

#### guardians

| Campo | Descripción |
|---|---|
| id | PK |
| user_id | FK users.id |
| first_name | Nombre |
| last_name | Apellidos |
| phone | Teléfono ficticio |
| email_contact | Correo de contacto, si difiere del acceso |
| is_active | Estado del perfil |

#### teachers

| Campo | Descripción |
|---|---|
| id | PK |
| user_id | FK users.id |
| employee_number | Clave de empleado ficticia, UNIQUE |
| first_name | Nombre |
| last_name | Apellidos |
| is_active | Estado del perfil |

La separación permite que users contenga credenciales y rol, mientras que cada tabla de perfil contiene los atributos del dominio. Se deberá decidir si toda persona de cada rol tendrá obligatoriamente user_id; para el MVP se recomienda que los perfiles de acceso privados tengan cuenta asociada.

#### student_guardians

Tabla pivote para la relación N:M entre estudiantes y tutores.

| Campo | Descripción |
|---|---|
| student_id | FK students.id |
| guardian_id | FK guardians.id |
| relationship | Parentesco |
| is_primary | Indica tutor principal |
| can_view_academic | Permiso de consulta académica |
| can_view_financial | Permiso de consulta financiera |
| is_active | Relación vigente |
| created_at | Fecha de asociación |

Clave primaria compuesta sugerida: (student_id, guardian_id). Esta tabla es una fuente esencial para la autorización de tutores.

### 3.3 Estructura académica

#### school_years

| Campo | Descripción |
|---|---|
| id | PK |
| name | Ej. 2026–2027 |
| start_date | Inicio |
| end_date | Fin |
| is_active | Ciclo activo |

#### grades_levels

Catálogo de grados: primero, segundo y tercero.

| Campo | Descripción |
|---|---|
| id | PK |
| name | Nombre del grado |
| sort_order | Orden |

#### groups

Grupo concreto dentro de un ciclo y grado.

| Campo | Descripción |
|---|---|
| id | PK |
| school_year_id | FK school_years.id |
| grade_level_id | FK grades_levels.id |
| section | Letra, ej. A o B |
| display_name | Etiqueta, ej. 2° B |
| capacity | Capacidad de demostración |
| is_active | Estado |

Restricción sugerida: UNIQUE(school_year_id, grade_level_id, section).

#### subjects

| Campo | Descripción |
|---|---|
| id | PK |
| code | Clave de materia, UNIQUE |
| name | Nombre |
| description | Descripción opcional |
| is_active | Estado |

#### teacher_assignments

Representa la asignación de un docente a una materia y grupo dentro de un ciclo.

| Campo | Descripción |
|---|---|
| id | PK |
| teacher_id | FK teachers.id |
| subject_id | FK subjects.id |
| group_id | FK groups.id |
| school_year_id | FK school_years.id |
| is_active | Estado |

Restricción sugerida: UNIQUE(teacher_id, subject_id, group_id, school_year_id), ajustable si la escuela requiere co-docencia o más de una asignación equivalente.

#### enrollments

| Campo | Descripción |
|---|---|
| id | PK |
| student_id | FK students.id |
| group_id | FK groups.id |
| school_year_id | FK school_years.id |
| enrollment_date | Fecha |
| status | PENDIENTE, ACTIVA, BAJA o FINALIZADA |
| created_by | FK users.id |

Debe existir una regla para impedir dos inscripciones activas incompatibles del mismo alumno en el mismo ciclo. Puede requerir índice único parcial en PostgreSQL o validación transaccional, según el diseño final.

#### grading_periods

| Campo | Descripción |
|---|---|
| id | PK |
| school_year_id | FK school_years.id |
| name | TRIMESTRE 1, 2 o 3 |
| sequence | 1, 2 o 3 |
| start_date | Inicio |
| end_date | Fin |

Restricción sugerida: UNIQUE(school_year_id, sequence).

#### grades

| Campo | Descripción |
|---|---|
| id | PK |
| student_id | FK students.id |
| teacher_assignment_id | FK teacher_assignments.id |
| grading_period_id | FK grading_periods.id |
| score | Calificación DECIMAL |
| comments | Observación opcional |
| created_by | FK users.id |
| updated_by | FK users.id |
| created_at / updated_at | Timestamps |

Restricción sugerida: UNIQUE(student_id, teacher_assignment_id, grading_period_id). CHECK de rango de 5.0 a 10.0 en base de datos, además de validación de aplicación.

### 3.4 Asistencia y conducta

#### attendance_sessions

Representa la lista que un docente pasa para una asignación y fecha.

| Campo | Descripción |
|---|---|
| id | PK |
| teacher_assignment_id | FK teacher_assignments.id |
| attendance_date | Fecha |
| opened_by | FK users.id |
| status | ABIERTA o CERRADA |
| created_at / updated_at | Timestamps |

Debe definirse si el mismo grupo tendrá una sesión por día, por bloque o por materia. El MVP puede iniciar con una sesión por asignación y fecha.

#### attendance_records

| Campo | Descripción |
|---|---|
| id | PK |
| attendance_session_id | FK attendance_sessions.id |
| student_id | FK students.id |
| status | PRESENTE, RETARDO, FALTA o JUSTIFICADA |
| notes | Observación opcional |
| marked_by | FK users.id |
| created_at / updated_at | Timestamps |

Restricción sugerida: UNIQUE(attendance_session_id, student_id). Para captura masiva se usará bulk insert/upsert con validaciones de autorización y pertenencia al grupo.

#### incidents

| Campo | Descripción |
|---|---|
| id | PK |
| student_id | FK students.id |
| reported_by | FK users.id |
| teacher_assignment_id | FK teacher_assignments.id, nullable según el tipo |
| category | LEVE, GRAVE, RETARDO u otra clasificación cerrada |
| description | Descripción |
| occurred_at | Fecha y hora del evento |
| status | REGISTRADA, EN_SEGUIMIENTO o CERRADA |
| created_at / updated_at | Timestamps |

#### summons

| Campo | Descripción |
|---|---|
| id | PK |
| student_id | FK students.id |
| incident_id | FK incidents.id, nullable si el citatorio agrupa varias incidencias |
| guardian_id | FK guardians.id |
| issued_at | Fecha de emisión |
| scheduled_at | Fecha de cita, opcional |
| status | PENDIENTE, ATENDIDO o CANCELADO |
| notes | Observación |
| created_at / updated_at | Timestamps |

La regla de tres incidencias leves se implementará como regla de negocio configurable del MVP. Se debe decidir si el conteo considera incidencias abiertas, de un periodo determinado o todas las incidencias del ciclo.

### 3.5 Finanzas simuladas

#### tuition_charges

| Campo | Descripción |
|---|---|
| id | PK |
| student_id | FK students.id |
| concept | Concepto del cargo |
| amount | NUMERIC, monto original |
| due_date | Vencimiento |
| issued_at | Emisión |
| status | PENDIENTE, PAGADO o VENCIDO |
| created_by | FK users.id |
| created_at / updated_at | Timestamps |

#### payments

| Campo | Descripción |
|---|---|
| id | PK |
| tuition_charge_id | FK tuition_charges.id |
| guardian_id | FK guardians.id |
| amount | NUMERIC |
| reference | Folio único |
| status | SIMULADO/PENDIENTE/CONFIRMADO/RECHAZADO según flujo final |
| paid_at | Fecha de simulación/confirmación |
| created_at | Timestamp |

El flujo de pago debe distinguir una intención de pago de un pago confirmado. Para el MVP se recomienda una operación explícita de simulación que crea el pago confirmado y actualiza el cargo dentro de una transacción, sin conectar servicios bancarios.

No se almacenarán datos de tarjetas bancarias ni credenciales de pago.

### 3.6 Servicios y contenido

#### service_requests

| Campo | Descripción |
|---|---|
| id | PK |
| student_id | FK students.id |
| guardian_id | FK guardians.id |
| request_type | CONSTANCIA u otro tipo de servicio |
| description | Motivo o detalle |
| status | RECIBIDA, EN_REVISION, APROBADA, RECHAZADA o COMPLETADA |
| requested_at | Fecha |
| resolved_by | FK users.id, nullable |
| resolved_at | Fecha, nullable |

#### materials

| Campo | Descripción |
|---|---|
| id | PK |
| title | Título |
| description | Descripción |
| resource_url | Enlace o ubicación controlada |
| teacher_assignment_id | FK teacher_assignments.id, nullable |
| group_id | FK groups.id, nullable |
| visibility | PUBLICO, INSTITUCIONAL o GRUPO |
| created_by | FK users.id |
| is_active | Estado |
| created_at / updated_at | Timestamps |

El alcance inicial puede utilizar enlaces y archivos de demostración controlados. No se permitirá que el usuario envíe rutas arbitrarias del servidor.

#### calendar_events

| Campo | Descripción |
|---|---|
| id | PK |
| title | Título |
| description | Detalle |
| starts_at | Inicio con zona horaria |
| ends_at | Fin con zona horaria |
| scope | GENERAL, GRADE o GROUP |
| grade_level_id | FK opcional |
| group_id | FK opcional |
| created_by | FK users.id |
| is_published | Publicación |

Se debe utilizar una política consistente de zona horaria: almacenar instantes en TIMESTAMPTZ y presentar en la zona local configurada para la escuela.

### 3.7 Auditoría

#### audit_logs

| Campo | Descripción |
|---|---|
| id | PK |
| user_id | FK users.id |
| role | Rol al momento de la acción |
| action | CREATE, UPDATE, DELETE, PAYMENT u otra acción |
| entity | Nombre del dominio/entidad |
| entity_id | Identificador del registro afectado |
| old_values | JSONB, nullable |
| new_values | JSONB, nullable |
| ip_address | IP si es pertinente y se decide retener |
| user_agent | User agent si es pertinente |
| created_at | Timestamp |

La bitácora no guardará contraseñas, hashes, identificadores de sesión, tokens, datos de tarjeta ni información secreta. Se recomienda una lista permitida de campos auditables para evitar registrar datos personales innecesarios.

## 4. Relaciones principales

| Origen | Relación | Destino | Propósito |
|---|---|---|---|
| users | 1:0..1 | students | Cuenta de alumno |
| users | 1:0..1 | guardians | Cuenta de tutor |
| users | 1:0..1 | teachers | Cuenta de docente |
| students | N:M | guardians | Familia / tutoría mediante student_guardians |
| school_years | 1:N | groups | Grupos de un ciclo |
| grades_levels | 1:N | groups | Grado de cada grupo |
| groups | 1:N | enrollments | Inscripciones |
| students | 1:N | enrollments | Historial de inscripción |
| teachers | 1:N | teacher_assignments | Asignaciones docentes |
| subjects | 1:N | teacher_assignments | Materias impartidas |
| groups | 1:N | teacher_assignments | Grupo atendido |
| teacher_assignments | 1:N | grades | Calificaciones |
| grading_periods | 1:N | grades | Periodo evaluativo |
| teacher_assignments | 1:N | attendance_sessions | Listas de asistencia |
| attendance_sessions | 1:N | attendance_records | Registros de lista |
| students | 1:N | attendance_records | Historial de asistencia |
| students | 1:N | incidents | Incidencias |
| incidents | 0..1:N | summons | Citatorios relacionados |
| students | 1:N | tuition_charges | Cargos escolares |
| tuition_charges | 1:N | payments | Historial de pagos (si se permite parcialidad) |
| students | 1:N | service_requests | Trámites |
| guardians | 1:N | service_requests | Solicitante |
| users | 1:N | audit_logs | Acciones registradas |

## 5. Índices y restricciones relevantes

- UNIQUE(users.email).
- UNIQUE(students.student_number).
- UNIQUE(teachers.employee_number).
- UNIQUE(subjects.code).
- UNIQUE(student_guardians.student_id, student_guardians.guardian_id).
- UNIQUE(groups.school_year_id, groups.grade_level_id, groups.section).
- UNIQUE(grades.student_id, grades.teacher_assignment_id, grades.grading_period_id).
- UNIQUE(attendance_records.attendance_session_id, attendance_records.student_id).
- UNIQUE(payments.reference).
- CHECK para montos positivos.
- CHECK para calificaciones en rango.
- Índices por student_id, guardian_id, group_id, school_year_id, fechas y estados de consulta frecuente.

La elección de índices se validará con consultas reales del MVP; no se deben indexar indiscriminadamente todas las columnas.

## 6. Consultas representativas que debe soportar el modelo

1. Listar los alumnos vinculados a un tutor.
2. Obtener los grupos y materias asignados a un docente.
3. Cargar alumnos inscritos en un grupo y ciclo.
4. Obtener las calificaciones de un alumno por trimestre.
5. Calcular el promedio por periodo y materia.
6. Cargar la lista de asistencia de una asignación y fecha.
7. Consultar cargos y pagos de un alumno a través de un tutor autorizado.
8. Obtener incidencias y citatorios asociados a un alumno.
9. Listar materiales visibles para un grupo.
10. Consultar eventos generales y específicos por grado/grupo.
11. Consultar auditoría por entidad, actor y rango de fechas.

## 7. Transacciones

### 7.1 Confirmación de pago simulado

Dentro de una única transacción:

1. Bloquear o validar el cargo para evitar doble confirmación.
2. Verificar que el tutor puede pagar en nombre del alumno asociado.
3. Crear el registro de pago y folio único.
4. Actualizar el estado/saldo del cargo según la política de parcialidades.
5. Crear la entrada de auditoría.
6. Confirmar la transacción.

Si cualquier paso falla, hacer rollback.

### 7.2 Captura masiva de asistencia

1. Verificar sesión y asignación docente.
2. Verificar que los estudiantes pertenecen al grupo de la asignación.
3. Validar estados y duplicados.
4. Insertar/actualizar registros en bloque.
5. Registrar auditoría resumida de la operación.
6. Confirmar la transacción.

### 7.3 Actualización de calificación

1. Verificar que el docente tenga la asignación.
2. Validar periodo y alumno inscrito.
3. Validar rango y unicidad.
4. Guardar cambio.
5. Guardar auditoría con valores anteriores/nuevos permitidos.
6. Confirmar transacción.

## 8. Migraciones y seeders

- Toda modificación del esquema se realizará mediante migración versionada.
- No se utilizará sincronización destructiva automática del ORM en producción.
- Los seeders crearán roles/perfiles de demostración, ciclos, grados, grupos, materias, asignaciones y datos ficticios.
- Las credenciales de demostración se documentarán únicamente en entornos locales y nunca serán válidas en producción.
- Los seeders deberán poder ejecutarse de forma repetible o indicar claramente si son de carga inicial.

## 9. Decisiones pendientes

- UUID o BIGINT para PK.
- Política exacta de relación user_id con cada perfil.
- Política de inscripciones y cambios de grupo.
- Modelo de asistencia: una sesión diaria, por materia o por bloque.
- Regla de cálculo y redondeo de promedios.
- Permitir o no pagos parciales.
- Estados definitivos del flujo de pagos.
- Cómputo de incidencias leves que genera citatorio.
- Estrategia de archivos: enlaces, volumen local o almacenamiento externo.
- Retención de bitácora y campos personales permitidos.
- Uso de índices parciales y estrategia de borrado lógico.

## 10. Criterios de aceptación del diseño

- Todas las entidades de negocio tienen una finalidad y dueño de datos definidos.
- Las relaciones N:M tienen tabla pivote explícita.
- Las restricciones principales están identificadas.
- Las relaciones de autorización pueden resolverse desde el modelo.
- El esquema soporta las consultas críticas.
- Los flujos financieros y de captura masiva tienen estrategia transaccional.
- Migraciones y seeders están contemplados desde el principio.
- El diseño se valida antes de empezar a crear modelos Sequelize.
