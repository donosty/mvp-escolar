# Contrato de API — AulaNexo

| Campo | Valor |
|---|---|
| Documento | API Contract |
| Versión | 0.1 — propuesta inicial |
| Base path | /api/v1 |
| Transporte | HTTP en desarrollo; HTTPS en despliegue |
| Formato principal | JSON |
| Autenticación | Sesión de servidor mediante cookie |
| Estado | Contrato preliminar para validación |

## 1. Propósito

Definir las convenciones de comunicación entre los clientes web (Vanilla y React) y el backend Express. La API será la única interfaz de acceso a las reglas de negocio y a los datos persistentes.

La documentación OpenAPI será la fuente formal del contrato cuando el backend comience a implementarse. Este documento establece una primera organización de recursos, permisos y ejemplos.

## 2. Convenciones generales

- Base path: /api/v1.
- Recursos en plural y nombres consistentes.
- JSON para solicitudes/respuestas, excepto descargas de PDF.
- Fechas/instantes en ISO 8601.
- Montos en formato decimal representado como string en JSON para evitar pérdida de precisión del cliente.
- Los identificadores serán opacos; conocer un ID nunca implica permiso.
- Listados tendrán paginación y filtros cuando el conjunto pueda crecer.
- Los endpoints mutables deberán validar CSRF cuando se utilicen cookies de sesión.
- Los códigos HTTP describirán el resultado de la operación.
- El servidor será la fuente de verdad para rol, propiedad, estados y cálculos.

## 3. Sesión y autenticación

### Endpoints

| Método | Ruta | Acceso | Descripción |
|---|---|---|---|
| GET | /auth/csrf | Público / bootstrap | Obtiene token CSRF asociado a la sesión |
| POST | /auth/login | Público | Inicia sesión |
| POST | /auth/logout | Autenticado | Destruye sesión |
| GET | /auth/me | Autenticado | Devuelve usuario y permisos/contexto mínimo |

#### POST /auth/login

Request:

    {
      "email": "admin@aulanexo.test",
      "password": "demo-password"
    }

Respuesta 200:

    {
      "data": {
        "user": {
          "id": "user-id",
          "email": "admin@aulanexo.test",
          "role": "ADMIN",
          "displayName": "Dirección"
        }
      }
    }

La respuesta establece la cookie de sesión HttpOnly. La contraseña nunca se devuelve ni se registra.

Errores posibles: 400 validación, 401 credenciales no válidas, 403 cuenta desactivada, 429 límite de intentos.

#### GET /auth/me

Respuesta 200:

    {
      "data": {
        "user": {
          "id": "user-id",
          "role": "TUTOR",
          "displayName": "Tutor de demostración"
        }
      }
    }

Respuesta 401 si no hay una sesión autenticada.

#### POST /auth/logout

Respuesta 204 cuando la sesión se destruye y la cookie se invalida.

## 4. Convenciones de solicitud y respuesta

### 4.1 Respuesta exitosa con recurso

    {
      "data": {
        "id": "resource-id",
        "name": "Ejemplo"
      }
    }

### 4.2 Respuesta de colección

    {
      "data": [],
      "meta": {
        "page": 1,
        "pageSize": 20,
        "totalItems": 0,
        "totalPages": 0
      }
    }

### 4.3 Error uniforme

    {
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "La solicitud contiene datos inválidos.",
        "details": [
          {
            "field": "email",
            "message": "El correo no tiene un formato válido."
          }
        ],
        "requestId": "request-id"
      }
    }

No se enviarán stack traces, SQL, contraseñas, secretos ni detalles de infraestructura.

### 4.4 Códigos HTTP

| Código | Uso |
|---|---|
| 200 | Consulta o actualización exitosa |
| 201 | Recurso creado |
| 202 | Solicitud aceptada para procesamiento posterior, si aplica |
| 204 | Operación exitosa sin cuerpo |
| 400 | Datos mal formados o validación general |
| 401 | No autenticado |
| 403 | Autenticado, pero sin permiso |
| 404 | Recurso no encontrado o no visible según política |
| 409 | Conflicto de estado o duplicado |
| 422 | Regla de negocio no satisfecha, si se adopta esta convención |
| 429 | Límite de solicitudes excedido |
| 500 | Error interno inesperado |

Se deberá elegir entre 400 y 422 para validaciones semánticas y usarlo de forma consistente.

## 5. Paginación, búsqueda y filtros

Convención recomendada:
- page: página base 1.
- pageSize: tamaño máximo controlado por servidor.
- q: búsqueda de texto.
- sort: campo permitido.
- order: asc o desc.
- Filtros adicionales por estado, grupo, fecha o ciclo.

Ejemplo de consulta: GET /students?page=1&pageSize=20&q=ana&groupId=group-id&status=ACTIVE.

El servidor deberá usar una lista permitida de campos ordenables y evitar concatenar parámetros del usuario directamente en SQL.

## 6. Recursos y endpoints propuestos

Los endpoints siguientes expresan la intención funcional. Los nombres y esquemas definitivos se reflejarán en OpenAPI antes de considerarse estables.

### 6.1 Alumnos

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /students | ADMIN | Listar alumnos con búsqueda y filtros |
| POST | /students | ADMIN | Registrar alumno |
| GET | /students/:studentId | ADMIN, TUTOR autorizado, ALUMNO propio, DOCENTE según contexto | Obtener expediente visible |
| PATCH | /students/:studentId | ADMIN | Actualizar datos permitidos |
| DELETE | /students/:studentId | ADMIN | Desactivar registro según política |
| GET | /students/:studentId/guardians | ADMIN, TUTOR autorizado | Consultar tutores vinculados |
| GET | /students/:studentId/grades | ADMIN, TUTOR autorizado, ALUMNO propio, DOCENTE asignado | Consultar calificaciones |
| GET | /students/:studentId/attendance | ADMIN, TUTOR autorizado, ALUMNO propio, DOCENTE asignado | Consultar asistencia |
| GET | /students/:studentId/incidents | ADMIN, TUTOR autorizado, ALUMNO propio, DOCENTE autorizado | Consultar incidencias |

### 6.2 Tutores y asociaciones

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /guardians | ADMIN | Listar tutores |
| POST | /guardians | ADMIN | Crear tutor |
| GET | /guardians/:guardianId | ADMIN, TUTOR propio | Consultar perfil |
| PATCH | /guardians/:guardianId | ADMIN, TUTOR propio en campos permitidos | Actualizar perfil |
| GET | /guardians/:guardianId/students | ADMIN, TUTOR propio | Listar alumnos asociados |
| POST | /students/:studentId/guardians | ADMIN | Vincular tutor y alumno |
| PATCH | /students/:studentId/guardians/:guardianId | ADMIN | Modificar parentesco/permisos/estado |
| DELETE | /students/:studentId/guardians/:guardianId | ADMIN | Desactivar vínculo |

La autorización de tutoría se realizará contra la relación persistida, no contra un studentId enviado por el cliente sin comprobar.

### 6.3 Usuarios

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /users | ADMIN | Listar cuentas |
| POST | /users | ADMIN | Crear cuenta |
| GET | /users/:userId | ADMIN, usuario propio en campos permitidos | Consultar cuenta |
| PATCH | /users/:userId | ADMIN, usuario propio en campos permitidos | Actualizar |
| PATCH | /users/:userId/status | ADMIN | Activar/desactivar cuenta |

No se expondrá password_hash en ninguna respuesta.

### 6.4 Estructura académica

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET/POST | /school-years | ADMIN (GET público limitado si aplica) | Consultar/crear ciclos |
| PATCH | /school-years/:id | ADMIN | Actualizar ciclo |
| GET/POST | /grade-levels | ADMIN | Catálogo de grados |
| GET/POST | /groups | ADMIN | Consultar/crear grupos |
| PATCH | /groups/:id | ADMIN | Actualizar grupo |
| GET/POST | /subjects | ADMIN | Consultar/crear materias |
| PATCH | /subjects/:id | ADMIN | Actualizar materia |
| GET/POST | /teacher-assignments | ADMIN | Consultar/crear asignaciones |
| PATCH | /teacher-assignments/:id | ADMIN | Actualizar asignación |
| GET | /teachers/me/assignments | DOCENTE | Consultar asignaciones del docente autenticado |
| GET/POST | /enrollments | ADMIN | Consultar/crear inscripciones |
| PATCH | /enrollments/:id | ADMIN | Cambiar estado o grupo conforme a reglas |

### 6.5 Asistencia

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /attendance/sessions | ADMIN, DOCENTE asignado | Listar sesiones |
| POST | /attendance/sessions | DOCENTE asignado, ADMIN | Crear sesión |
| GET | /attendance/sessions/:sessionId | ADMIN, DOCENTE asignado | Consultar lista |
| POST | /attendance/sessions/:sessionId/records/bulk | DOCENTE asignado | Crear/actualizar registros masivos |
| PATCH | /attendance/records/:recordId | DOCENTE autorizado, ADMIN bajo auditoría | Ajustar registro |
| POST | /attendance/sessions/:sessionId/close | DOCENTE asignado, ADMIN | Cerrar sesión |

Ejemplo de captura masiva:

    {
      "records": [
        { "studentId": "student-1", "status": "PRESENTE" },
        { "studentId": "student-2", "status": "RETARDO" },
        { "studentId": "student-3", "status": "FALTA" }
      ]
    }

El servidor comprobará que todos los alumnos pertenecen al grupo de la asignación y que el docente tiene permiso. Una solicitud parcialmente inválida deberá seguir una política definida: se recomienda rechazar todo el lote y no guardar registros parciales.

### 6.6 Calificaciones y periodos

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET/POST | /grading-periods | ADMIN | Consultar/crear trimestres |
| PATCH | /grading-periods/:id | ADMIN | Actualizar periodo |
| GET | /grades | ADMIN, DOCENTE asignado, TUTOR autorizado, ALUMNO propio | Listar con filtros |
| POST | /grades | DOCENTE asignado | Capturar calificación |
| GET | /grades/:gradeId | ADMIN, DOCENTE autorizado, TUTOR/ALUMNO autorizados | Consultar |
| PATCH | /grades/:gradeId | DOCENTE asignado, ADMIN con auditoría | Actualizar |
| GET | /students/:studentId/report-card | ADMIN, TUTOR autorizado, ALUMNO propio | Consultar resumen de boleta |
| GET | /students/:studentId/report-card.pdf | ADMIN, TUTOR autorizado | Descargar PDF demostrativo |

El servidor validará periodo, asignación, pertenencia del alumno al grupo y rango 5.0–10.0. El cálculo del promedio será del servidor y no aceptará un promedio enviado por el cliente como fuente de verdad.

### 6.7 Colegiaturas y pagos simulados

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /tuition/charges | ADMIN, TUTOR autorizado | Consultar cargos |
| POST | /tuition/charges | ADMIN | Crear cargo |
| GET | /tuition/charges/:chargeId | ADMIN, TUTOR autorizado | Consultar cargo |
| PATCH | /tuition/charges/:chargeId | ADMIN | Modificar cargo conforme a política |
| GET | /payments | ADMIN, TUTOR autorizado | Consultar pagos |
| POST | /payments/simulate | TUTOR autorizado | Ejecutar pago ficticio |
| GET | /payments/:paymentId | ADMIN, TUTOR propietario | Consultar pago |
| GET | /payments/:paymentId/receipt.pdf | ADMIN, TUTOR propietario | Descargar recibo demostrativo |

Ejemplo:

    {
      "chargeId": "charge-id",
      "amount": "1200.00",
      "paymentMethod": "TRANSFERENCIA_SIMULADA"
    }

El backend calculará o verificará el saldo y evitará que el cliente pague más de lo permitido. La operación generará un folio no repetido y se ejecutará dentro de una transacción. Este endpoint no contactará un banco ni almacenará información de tarjetas.

### 6.8 Incidencias y citatorios

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /incidents | ADMIN, DOCENTE autorizado, TUTOR/ALUMNO según alcance | Listar |
| POST | /incidents | DOCENTE autorizado, ADMIN | Registrar incidencia |
| GET | /incidents/:incidentId | Roles autorizados | Consultar |
| PATCH | /incidents/:incidentId | DOCENTE autorizado, ADMIN | Actualizar estado/campos permitidos |
| GET | /summons | ADMIN, DOCENTE autorizado, TUTOR autorizado | Listar citatorios |
| POST | /summons | ADMIN, DOCENTE autorizado según política | Emitir citatorio |
| PATCH | /summons/:summonsId | ADMIN, DOCENTE autorizado | Actualizar seguimiento |

El disparador automático por tres incidencias leves se ejecutará en el servicio de dominio y deberá ser idempotente para no crear múltiples citatorios por el mismo conjunto de incidencias.

### 6.9 Servicios escolares

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /service-requests | ADMIN, TUTOR propietario | Listar solicitudes |
| POST | /service-requests | TUTOR autorizado | Crear solicitud |
| GET | /service-requests/:id | ADMIN, TUTOR propietario | Consultar |
| PATCH | /service-requests/:id/status | ADMIN | Actualizar estado |
| GET | /service-requests/:id/document.pdf | ADMIN, TUTOR propietario | Descargar documento demostrativo si está disponible |

### 6.10 Materiales y calendario

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /materials | Según visibilidad y grupo | Consultar materiales visibles |
| POST | /materials | DOCENTE asignado, ADMIN | Publicar material |
| PATCH | /materials/:id | Autor autorizado, ADMIN | Actualizar |
| DELETE | /materials/:id | Autor autorizado, ADMIN | Desactivar |
| GET | /calendar-events | Público para eventos publicados / privado según alcance | Consultar eventos |
| POST | /calendar-events | ADMIN | Crear evento |
| PATCH | /calendar-events/:id | ADMIN | Actualizar |
| DELETE | /calendar-events/:id | ADMIN | Desactivar |

Los recursos privados deberán comprobar visibilidad por grupo o asignación. Para el MVP se recomienda comenzar con enlaces a materiales y no aceptar rutas arbitrarias del servidor.

### 6.11 Auditoría y salud

| Método | Ruta | Roles | Descripción |
|---|---|---|---|
| GET | /audit-logs | ADMIN | Consultar auditoría con filtros |
| GET | /health | Público o red interna | Estado básico de API y dependencias según política |

## 7. Reglas de autorización resumidas

| Recurso | ADMIN | DOCENTE | TUTOR | ALUMNO |
|---|---|---|---|---|
| Usuarios | CRUD | — | Propio limitado | Propio limitado |
| Alumnos | CRUD | Asignados/lectura | Vinculados/lectura | Propio/lectura |
| Tutores | CRUD | — | Propio/lectura | — |
| Grupos y materias | CRUD | Asignados/lectura | — | Propio/lectura |
| Asistencia | Supervisión | Escribir en asignación propia | Hijos/lectura | Propia/lectura |
| Calificaciones | Supervisión | Escribir en asignación propia | Hijos/lectura | Propias/lectura |
| Cargos/pagos | CRUD/consulta | — | Hijos/consulta y pago simulado | — |
| Incidencias | Supervisión | Registrar/gestionar autorizadas | Hijos/lectura | Propias/lectura |
| Materiales | CRUD | Publicar para asignación | Visibles | Visibles para grupo |
| Solicitudes | Todas | — | Crear/consultar propias | — |
| Auditoría | Consulta | — | — | — |

Esta tabla es orientativa; cada endpoint requerirá una política concreta de rol, propiedad, relación y alcance.

## 8. Seguridad del contrato

- Los endpoints mutables usan verificación CSRF.
- Las cookies de sesión no se exponen a JavaScript (HttpOnly).
- El frontend usa fetch con credenciales según el origen configurado.
- El servidor regenera y destruye sesiones en login/logout.
- Los errores no revelan si existe una cuenta cuando ello pueda facilitar enumeración.
- Los listados respetan filtros de autorización antes de paginar.
- Los endpoints de descarga validan acceso al documento antes de enviarlo.
- La limitación de intentos se aplica a login y operaciones sensibles.
- Se auditan cambios críticos sin almacenar secretos ni información personal innecesaria.

## 9. Convención de idempotencia y concurrencia

- POST de pagos debe evitar doble procesamiento; utilizar un identificador de idempotencia o una clave de operación única.
- Captura masiva de asistencia deberá tener una política de actualización/reintento explícita.
- Actualizaciones concurrentes de calificación deben evitar sobrescritura silenciosa; puede usarse updated_at o control de versión.
- Cambios de estado financieros deben validar el estado previo permitido.
- La transacción y las restricciones de la base de datos son la garantía final frente a carreras.

## 10. Documentación OpenAPI

Antes de considerar estable la API, crear una especificación OpenAPI con:
- Schemas de request y response.
- Esquemas de error.
- Parámetros de paginación y filtros.
- Cookie/session security scheme documentado.
- Operaciones y roles requeridos.
- Ejemplos para flujos de negocio principales.
- Respuestas de error y códigos HTTP.
- Descargas PDF documentadas como application/pdf.

## 11. Criterios de aceptación del contrato

- Los dos frontends pueden implementar sus casos de uso sin acceso directo a la base de datos.
- Cada operación tiene método, ruta, entrada, salida y permiso definidos.
- Los identificadores se validan contra la identidad autenticada.
- Los códigos HTTP y errores son consistentes.
- Paginación y filtros están documentados.
- Pagos y capturas masivas tienen reglas de idempotencia/transacción.
- CSRF está documentado para operaciones mutables con cookies.
- La especificación OpenAPI coincide con los endpoints implementados.
