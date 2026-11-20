# Requerimientos del producto — AulaNexo

| Campo | Valor |
|---|---|
| Producto | AulaNexo — Sistema de Gestión Escolar |
| Institución de referencia | Escuela Secundaria General Horizonte |
| Documento | Product Requirements Document (PRD) |
| Versión | 0.1 — línea base de Sprint 0 |
| Estado | Borrador para validación |
| Audiencia | Equipo de desarrollo, docente responsable y estudiantes que utilizarán el material didáctico |

## 1. Resumen ejecutivo

AulaNexo será una plataforma web de gestión escolar para una secundaria ficticia. Tendrá un portal institucional público y un área privada con funcionalidades diferenciadas para Dirección, Docentes, Tutores y Alumnos.

El proyecto se construirá progresivamente en dos versiones de frontend: primero con HTML, CSS y JavaScript nativo; después con React. Ambas versiones consumirán la misma API REST y utilizarán el mismo backend y base de datos. La solución incorporará persistencia relacional, un ORM, autenticación basada en sesiones, autorización por roles, pruebas automatizadas, Docker y un pipeline CI/CD.

El producto debe ser suficientemente realista para mostrar prácticas de ingeniería de software, pero mantener un alcance acotado y demostrable.

## 2. Contexto y problema

Una institución escolar necesita centralizar información académica, administrativa y de comunicación. En el escenario de AulaNexo, Dirección administra la estructura escolar y los datos institucionales; los docentes gestionan asistencia, calificaciones e incidencias; los tutores dan seguimiento a sus hijos y a los pagos; y los alumnos consultan su propia información académica.

El sistema permitirá mostrar cómo distintos perfiles colaboran sobre información relacionada, manteniendo límites de acceso y trazabilidad.

## 3. Visión y objetivos

### 3.1 Visión

Ofrecer una demostración integral de un sistema escolar web, desde la estructura semántica de una página hasta una aplicación full stack probada, contenerizada y automatizada.

### 3.2 Objetivos de producto

- Publicar información institucional y servicios escolares en un portal público.
- Centralizar operaciones escolares básicas en un portal privado.
- Separar las capacidades de cada usuario mediante roles y autorización en el servidor.
- Mantener datos consistentes mediante un modelo relacional y restricciones de integridad.
- Exponer funcionalidades mediante una API REST versionada.
- Implementar las mismas funcionalidades principales en un frontend Vanilla y en React.
- Mostrar una cadena de desarrollo reproducible, probada y automatizada.
- Producir videos, manual y presentación que expliquen el proceso a personas principiantes.

### 3.3 Objetivos didácticos

Al finalizar el material, el estudiante debería poder explicar:

- La función de HTML, CSS y JavaScript en el navegador.
- La separación entre frontend, backend y base de datos.
- El propósito de HTTP, JSON y una API REST.
- El modelado relacional, claves y asociaciones.
- Qué resuelve un ORM y qué conceptos SQL siguen siendo necesarios.
- La diferencia entre autenticación y autorización.
- Cómo funcionan las cookies de sesión.
- Qué son las pruebas unitarias, de integración y end-to-end.
- Qué aportan Docker, Git, GitHub Actions y CI/CD.

## 4. Alcance del producto

### 4.1 Dentro del MVP

1. Portal institucional público.
2. Inicio y cierre de sesión.
3. Cuatro roles: ADMIN, DOCENTE, TUTOR y ALUMNO.
4. Gestión de usuarios y perfiles.
5. Registro de alumnos y vinculación con tutores.
6. Administración de ciclos escolares, grados, grupos, materias y asignaciones docentes.
7. Registro masivo de asistencia por grupo y fecha.
8. Captura y consulta de calificaciones por trimestre.
9. Cálculo de promedios y generación de boletas PDF demostrativas.
10. Gestión de cargos escolares y pagos simulados.
11. Registro de incidencias y seguimiento de citatorios.
12. Solicitudes de servicios escolares, incluyendo constancias demostrativas.
13. Publicación y consulta de materiales educativos.
14. Calendario de eventos institucionales.
15. Búsqueda, filtros y resultados tabulares.
16. Bitácora de auditoría para operaciones críticas.
17. Frontend Vanilla y frontend React consumiendo la misma API.
18. Pruebas automatizadas, Docker Compose y CI/CD.

### 4.2 Fuera del MVP

- Cobro de dinero real o conexión con pasarelas bancarias.
- Integración con bancos, sistemas gubernamentales o plataformas oficiales de control escolar.
- Emisión de documentos oficiales, fiscales o con validez institucional.
- Envío real de SMS o mensajes de WhatsApp.
- Aplicación móvil nativa.
- Arquitectura de microservicios.
- Operación multiinstitución o multi-tenant.
- Alta disponibilidad, balanceo avanzado o recuperación ante desastres.
- Analítica predictiva o modelos de inteligencia artificial.

### 4.3 Supuestos y restricciones

- La escuela, las personas y los registros serán ficticios.
- Se utilizará un ciclo escolar demostrativo 2026–2027, tres grados y grupos A y B por grado.
- El calendario académico de demostración tendrá tres trimestres.
- Los importes estarán expresados en MXN.
- Los pagos y documentos son simulaciones educativas.
- El backend será la fuente de verdad para permisos, validaciones y reglas de negocio.
- Las dos aplicaciones frontend no tendrán bases de datos ni reglas de negocio independientes.
- El alcance podrá ajustarse durante Sprint 0, pero cualquier cambio deberá documentarse.

## 5. Personas y perfiles

### 5.1 Administrador — Dirección / Control Escolar

**Propósito:** administrar el catálogo y supervisar la operación de la institución.

Capacidades:

- Gestionar usuarios y perfiles de acceso.
- Registrar alumnos, tutores y docentes.
- Vincular uno o varios tutores con cada alumno.
- Gestionar ciclos escolares, grados, grupos y materias.
- Crear asignaciones de docente, materia y grupo.
- Registrar inscripciones.
- Generar cargos de colegiatura y consultar adeudos.
- Registrar o validar pagos simulados.
- Administrar eventos y materiales.
- Consultar indicadores generales.
- Consultar auditoría de operaciones críticas.

### 5.2 Docente — Profesor / Prefectura

**Propósito:** gestionar las actividades académicas y de seguimiento de sus grupos asignados.

Capacidades:

- Consultar grupos y materias que tiene asignados.
- Consultar listas de alumnos.
- Registrar asistencia por grupo en una operación masiva.
- Capturar y actualizar calificaciones trimestrales.
- Consultar promedios de sus grupos.
- Registrar incidencias y retardos.
- Emitir o dar seguimiento a citatorios según los permisos definidos.
- Publicar materiales para sus grupos o materias.

### 5.3 Tutor — Padre, madre o tutor legal

**Propósito:** consultar información de los alumnos que tiene vinculados.

Capacidades:

- Consultar la lista de hijos/alumnos vinculados.
- Cambiar el contexto entre estudiantes asociados.
- Consultar calificaciones y boletas.
- Descargar boletas demostrativas.
- Consultar asistencias, incidencias y citatorios.
- Consultar cargos, pagos e historial financiero.
- Realizar pagos simulados.
- Descargar recibos demostrativos.
- Crear solicitudes de servicios escolares.
- Consultar el calendario y avisos correspondientes.

### 5.4 Alumno — Estudiante

**Propósito:** consultar su propia información académica y recursos.

Capacidades:

- Consultar grupo, materias y asignaturas.
- Consultar calificaciones.
- Consultar asistencias.
- Consultar materiales compartidos con su grupo.
- Consultar avisos e incidencias propias, en modo de solo lectura.

El alumno no tendrá acceso a módulos financieros ni administrativos.

## 6. Portal público

El portal público debe poder recorrerse sin iniciar sesión.

| Sección | Contenido / comportamiento |
|---|---|
| Inicio | Presentación, avisos y accesos principales |
| Institución | Historia, misión, visión e información general ficticia |
| Oferta educativa | Grados, grupos y materias de referencia |
| Admisiones | Requisitos y proceso de inscripción |
| Servicios escolares | Trámites disponibles y orientación |
| Colegiaturas | Información de cuotas y modalidades de demostración |
| Material de consulta | Recursos públicos seleccionados |
| Calendario | Eventos institucionales públicos |
| Contacto | Datos ficticios y formulario demostrativo |
| Acceso | Entrada al portal privado |

## 7. Requerimientos funcionales

Los identificadores se utilizarán para trazabilidad entre requerimientos, pruebas, issues y criterios de aceptación.

### 7.1 Identidad y acceso

- **RF-001 — Inicio de sesión:** el sistema permitirá autenticar a un usuario activo mediante correo y contraseña.
- **RF-002 — Cierre de sesión:** el sistema destruirá la sesión en el servidor y limpiará la cookie.
- **RF-003 — Sesión actual:** el cliente podrá consultar la identidad y rol del usuario autenticado.
- **RF-004 — Protección de rutas:** las operaciones privadas requerirán una sesión válida.
- **RF-005 — Autorización por rol:** el servidor verificará que el rol pueda ejecutar la operación.
- **RF-006 — Autorización por recurso:** el servidor comprobará la relación del usuario con el alumno, grupo o asignación solicitada.
- **RF-007 — Navegación por rol:** el frontend mostrará las opciones correspondientes al perfil, sin sustituir la validación del backend.

### 7.2 Usuarios, alumnos y tutores

- **RF-008:** ADMIN podrá crear, consultar, actualizar y desactivar usuarios.
- **RF-009:** ADMIN podrá registrar alumnos, tutores y docentes.
- **RF-010:** ADMIN podrá asociar varios tutores con un alumno y un tutor con varios alumnos.
- **RF-011:** la asociación alumno-tutor conservará parentesco, indicador de tutor principal y estado de la relación.
- **RF-012:** el TUTOR solo podrá consultar los alumnos vinculados a su perfil.
- **RF-013:** el ALUMNO solo podrá consultar su propio expediente permitido.

### 7.3 Estructura académica e inscripciones

- **RF-014:** ADMIN podrá gestionar ciclos escolares.
- **RF-015:** ADMIN podrá gestionar grados y grupos dentro de un ciclo escolar.
- **RF-016:** ADMIN podrá gestionar materias.
- **RF-017:** ADMIN podrá crear asignaciones de docente, materia y grupo.
- **RF-018:** ADMIN podrá registrar la inscripción de un alumno a un grupo de un ciclo escolar.
- **RF-019:** el sistema evitará inscripciones activas duplicadas para el mismo alumno y ciclo escolar.

### 7.4 Asistencia

- **RF-020:** DOCENTE podrá consultar los grupos que tiene asignados.
- **RF-021:** DOCENTE podrá abrir una sesión de asistencia para una fecha y asignación.
- **RF-022:** DOCENTE podrá marcar cada alumno como PRESENTE, RETARDO, FALTA o JUSTIFICADA.
- **RF-023:** el sistema permitirá guardar los registros de asistencia de un grupo mediante una operación masiva.
- **RF-024:** el servidor verificará que el docente tenga asignada la materia/grupo correspondiente.
- **RF-025:** TUTOR y ALUMNO podrán consultar la asistencia de los estudiantes autorizados.

### 7.5 Calificaciones y boletas

- **RF-026:** DOCENTE podrá consultar alumnos de sus asignaciones.
- **RF-027:** DOCENTE podrá capturar y actualizar calificaciones por alumno, asignación y trimestre.
- **RF-028:** el servidor validará que la calificación se encuentre dentro del rango de demostración de 5.0 a 10.0.
- **RF-029:** el sistema impedirá duplicados para la misma combinación de alumno, asignación y periodo.
- **RF-030:** el sistema calculará promedios mediante una regla documentada.
- **RF-031:** TUTOR y ALUMNO podrán consultar las calificaciones que les correspondan.
- **RF-032:** TUTOR podrá generar y descargar una boleta PDF demostrativa con folio verificable dentro de AulaNexo.

### 7.6 Colegiaturas y pagos simulados

- **RF-033:** ADMIN podrá crear cargos escolares asociados a alumnos.
- **RF-034:** el cargo tendrá monto, fecha de emisión, vencimiento y estado.
- **RF-035:** el estado del cargo podrá ser PENDIENTE, PAGADO o VENCIDO.
- **RF-036:** TUTOR podrá consultar únicamente los cargos de sus alumnos vinculados.
- **RF-037:** TUTOR podrá iniciar un pago simulado.
- **RF-038:** el sistema registrará el pago, actualizará el cargo y generará un folio de recibo.
- **RF-039:** las escrituras relacionadas con la confirmación del pago se realizarán dentro de una transacción.
- **RF-040:** TUTOR podrá consultar el historial y descargar recibos demostrativos.
- **RF-041:** ADMIN podrá consultar cargos y pagos del plantel.

### 7.7 Incidencias y citatorios

- **RF-042:** DOCENTE autorizado podrá registrar incidencias relacionadas con un alumno.
- **RF-043:** cada incidencia almacenará categoría, descripción, fecha, alumno y usuario que la registró.
- **RF-044:** el sistema podrá generar un citatorio al cumplirse la regla demostrativa de acumulación de tres incidencias leves.
- **RF-045:** el citatorio tendrá un estado de seguimiento, por ejemplo PENDIENTE y ATENDIDO.
- **RF-046:** TUTOR podrá consultar incidencias y citatorios de sus alumnos.
- **RF-047:** ADMIN podrá supervisar las incidencias del plantel.

### 7.8 Servicios escolares

- **RF-048:** TUTOR podrá crear una solicitud de servicio escolar para un alumno vinculado.
- **RF-049:** el sistema soportará solicitudes de constancia y otros trámites demostrativos definidos.
- **RF-050:** ADMIN podrá consultar y actualizar el estado de las solicitudes.
- **RF-051:** TUTOR podrá consultar el estado de sus solicitudes.
- **RF-052:** los documentos generados serán demostrativos y mostrarán claramente su carácter no oficial.

### 7.9 Materiales y calendario

- **RF-053:** DOCENTE podrá publicar materiales para las asignaciones/grupos autorizados.
- **RF-054:** ADMIN podrá administrar materiales institucionales.
- **RF-055:** ALUMNO podrá consultar materiales compartidos con su grupo.
- **RF-056:** ADMIN podrá crear y modificar eventos institucionales.
- **RF-057:** el calendario permitirá filtrar eventos por fecha y alcance (general, grado o grupo).

### 7.10 Búsqueda y auditoría

- **RF-058:** los listados administrativos incluirán búsqueda y filtros relevantes.
- **RF-059:** los resultados se presentarán en tablas o listas legibles.
- **RF-060:** el sistema registrará en una bitácora las operaciones sensibles, en particular cambios de calificaciones, pagos e incidencias.
- **RF-061:** la bitácora conservará actor, rol, acción, entidad, identificador y fecha.
- **RF-062:** ADMIN podrá consultar la bitácora; otros roles no tendrán acceso general a ella.

## 8. Reglas de negocio

- **RN-001 — Perfiles:** cada cuenta tendrá un rol primario entre ADMIN, DOCENTE, TUTOR y ALUMNO.
- **RN-002 — Relación familiar:** la relación alumno-tutor será muchos-a-muchos y se representará mediante una tabla intermedia.
- **RN-003 — Propiedad del expediente:** un tutor solo podrá consultar alumnos asociados a su perfil.
- **RN-004 — Asignaciones docentes:** un docente solo podrá operar sobre grupos/materias que tenga asignados y que estén vigentes.
- **RN-005 — Ciclo escolar:** los grupos y las inscripciones pertenecen a un ciclo escolar.
- **RN-006 — Calificaciones:** el rango de demostración será 5.0–10.0; su precisión y regla de redondeo se definirán antes de codificar.
- **RN-007 — Unicidad de calificación:** no habrá dos calificaciones activas para alumno, asignación y periodo.
- **RN-008 — Asistencia:** cada alumno tendrá como máximo un registro por sesión de asistencia.
- **RN-009 — Pago:** un pago confirmado actualizará el cargo relacionado y tendrá un folio único.
- **RN-010 — Integridad financiera:** cargo, pago y registro de auditoría se confirmarán en una transacción.
- **RN-011 — Incidencia:** la regla de tres incidencias leves es una regla ficticia y configurable del MVP.
- **RN-012 — Auditoría:** no se permitirá alterar la bitácora mediante las operaciones normales de la API.
- **RN-013 — Datos de prueba:** los datos iniciales serán ficticios y se cargarán mediante seeders.
- **RN-014 — Documentos:** boletas, recibos y constancias llevarán una leyenda de documento demostrativo/no oficial.

## 9. Requerimientos no funcionales

### Seguridad

- **RNF-001:** todas las operaciones privadas se ejecutarán detrás de autenticación.
- **RNF-002:** la autorización y la propiedad de recursos se validarán del lado servidor.
- **RNF-003:** las contraseñas se almacenarán mediante un algoritmo de hash adaptativo (bcrypt, con parámetros revisados para el entorno).
- **RNF-004:** la cookie de sesión será HttpOnly; en producción usará Secure y una política SameSite apropiada.
- **RNF-005:** se regenerará el identificador de sesión después de autenticar.
- **RNF-006:** se protegerán las operaciones mutables contra CSRF.
- **RNF-007:** habrá limitación de intentos de inicio de sesión.
- **RNF-008:** secretos y credenciales no se versionarán.
- **RNF-009:** los datos de prueba no contendrán información real de menores.

### Calidad y mantenibilidad

- **RNF-010:** el backend separará rutas, controladores, servicios, modelos y middlewares.
- **RNF-011:** la API tendrá un prefijo de versión.
- **RNF-012:** los endpoints y modelos tendrán documentación actualizada.
- **RNF-013:** las funcionalidades críticas contarán con pruebas automatizadas.
- **RNF-014:** el proyecto tendrá instrucciones reproducibles de instalación y ejecución.
- **RNF-015:** los errores se responderán con una estructura consistente sin exponer trazas en producción.

### Usabilidad y accesibilidad

- **RNF-016:** el portal y los paneles serán responsivos.
- **RNF-017:** formularios tendrán etiquetas, mensajes de validación y estados accesibles.
- **RNF-018:** tablas y acciones principales serán comprensibles para usuarios no técnicos.
- **RNF-019:** el manual de usuario utilizará lenguaje no técnico, capturas y señalización visual.

### Operación

- **RNF-020:** la aplicación podrá levantarse localmente mediante Docker Compose.
- **RNF-021:** la API expondrá un endpoint de salud.
- **RNF-022:** CI ejecutará validaciones y pruebas antes de aceptar cambios.
- **RNF-023:** las imágenes de contenedor serán versionadas y publicadas en GHCR cuando el pipeline esté habilitado.

## 10. Matriz resumida de permisos

| Capacidad | ADMIN | DOCENTE | TUTOR | ALUMNO |
|---|---:|---:|---:|---:|
| Administrar usuarios | Sí | No | No | No |
| Gestionar estructura académica | Sí | No | No | No |
| Registrar alumnos/tutores | Sí | No | No | No |
| Consultar grupos asignados | Todos | Propios | No | Propio |
| Pasar asistencia | Supervisión | Propios | No | No |
| Capturar calificaciones | Supervisión/corrección | Propias asignaciones | No | No |
| Consultar calificaciones | Todas | Propias asignaciones | Hijos vinculados | Propias |
| Consultar pagos | Todos | No | Hijos vinculados | No |
| Simular pago | No (validación administrativa) | No | Hijos vinculados | No |
| Registrar incidencias | Supervisión | Según asignación/permisos | No | No |
| Consultar incidencias | Todas | Propias/relacionadas | Hijos vinculados | Propias |
| Publicar materiales | Institucional | Asignaciones propias | No | No |
| Consultar materiales | Todos | Permitidos | Públicos/permitidos | Grupo propio |
| Consultar auditoría | Sí | No | No | No |

La matriz es una línea base. La autorización final deberá expresarse también a nivel de recurso y probarse mediante casos positivos y negativos.

## 11. Criterios de aceptación del MVP

El MVP se considerará funcionalmente aceptado cuando:

1. El portal público sea navegable y responsivo.
2. Los cuatro roles puedan iniciar y cerrar sesión.
3. Las rutas protegidas rechacen usuarios anónimos.
4. Un usuario no pueda acceder a información ajena cambiando identificadores.
5. ADMIN pueda registrar alumnos y tutores y relacionarlos.
6. DOCENTE pueda registrar asistencias en bloque y calificaciones dentro de su asignación.
7. TUTOR pueda consultar la información de sus alumnos vinculados.
8. Los pagos simulados actualicen cargos, creen recibos y generen auditoría de manera consistente.
9. Los módulos de materiales, calendario, incidencias y solicitudes tengan sus flujos principales.
10. La búsqueda presente resultados correctos en tablas o listas.
11. Las versiones Vanilla y React consuman la misma API.
12. Las pruebas críticas pasen.
13. La aplicación pueda ejecutarse con Docker Compose.
14. El pipeline de CI ejecute los controles definidos.
15. Estén listos los videos, manual y presentación requeridos por la actividad.

## 12. Entregables educativos y relación con la actividad

| Requisito de la actividad | Evidencia de AulaNexo |
|---|---|
| Página escolar y video HTML | Portal público y Video 1 |
| CSS y responsividad | Interfaz responsive y Video 2 |
| JavaScript, formularios, DB y buscador | Frontend Vanilla, API, PostgreSQL y Video 3 |
| Versión con framework | Frontend React y Video 4 |
| Demostración funcional, CRUD y checklist | Video 5 y lista de cotejo |
| Manual sin lenguaje técnico | Manual PDF visual por perfil |
| Presentación tecnológica | Presentación de AulaNexo |
| Extensión técnica complementaria | Video 6 de Docker, testing y CI/CD |

## 13. Riesgos y mitigaciones

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Alcance demasiado amplio | Retraso y funcionalidades incompletas | Congelar alcance MVP y priorizar flujos principales |
| Reglas ambiguas de negocio | Reprocesos en backend y DB | Definir reglas y criterios de aceptación en Sprint 0 |
| Permisos incorrectos | Exposición de información | Autorización en servidor y pruebas negativas |
| Cambios al modelo de datos tardíos | Migraciones complejas | Revisar ERD antes de implementar módulos |
| Duplicación entre frontends | Inconsistencia funcional | API y reglas de negocio compartidas |
| Falta de datos de demostración | Videos poco claros | Seeders con escenarios ficticios |
| Configuración insegura | Riesgo de exposición | Variables de entorno, cookies seguras, CSRF y revisión de dependencias |
| CI/CD prematuro | Automatización frágil | Incorporarlo progresivamente después de pruebas y contenedores |
| Documentación desactualizada | Dificultad para mantener y enseñar | Actualizar documentos junto con cambios aprobados |

## 14. Decisiones pendientes de cierre en Sprint 0

- Regla exacta de promedio trimestral y redondeo.
- Si las calificaciones se capturarán por materia y trimestre o habrá componentes evaluativos adicionales.
- Política para edición de asistencias después del cierre de una sesión.
- Flujo exacto de autorización/validación de pagos simulados.
- Estados completos de solicitudes y citatorios.
- Formato y método de verificación de boletas PDF.
- Política de retención y desactivación de usuarios/alumnos.
- Si los perfiles de alumno, tutor y docente serán obligatorios para toda cuenta de esos roles.
- Estrategia final de subida de archivos: enlaces o almacenamiento local controlado.
- Entorno concreto de despliegue y dominio para la demostración.

Estas decisiones se resolverán antes de que los módulos dependientes entren a implementación. No deben inferirse silenciosamente durante la codificación.

## 15. Roadmap de alto nivel

| Sprint | Objetivo | Evidencia |
|---|---|---|
| 0 | Requerimientos, modelo de datos, arquitectura, API y prototipos | Documentación aprobada |
| 1 | Portal y estructura HTML | Video 1 |
| 2 | Diseño visual responsive | Video 2 |
| 3 | JavaScript, API, ORM, DB, sesiones y módulos | Video 3 |
| 4 | Reconstrucción de frontend con React | Video 4 |
| 5 | Pruebas, demostración y documentación de usuario | Video 5, manual y presentación |
| 6 | Docker, CI/CD y despliegue demostrativo | Video 6 |
| 7 | Estabilización y entrega | Release 1.0 |

## 16. Definición de terminado (Definition of Done)

Una historia se considerará terminada cuando:

- Cumpla los criterios de aceptación.
- Valide entradas en cliente y servidor cuando corresponda.
- Verifique autenticación, rol y acceso al recurso.
- Mantenga integridad transaccional cuando aplique.
- Incluya pruebas relevantes.
- No introduzca secretos ni datos reales.
- Actualice documentación técnica/contrato API si hubo cambios.
- Haya sido revisada mediante Pull Request.
- Pase las verificaciones automatizadas de CI.

