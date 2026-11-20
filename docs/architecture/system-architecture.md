# Arquitectura del sistema — AulaNexo

| Campo | Valor |
|---|---|
| Documento | System Architecture |
| Versión | 0.1 — propuesta de Sprint 0 |
| Estilo | Monolito modular con API REST |
| Clientes | Vanilla JS y React |
| Backend | Node.js + Express |
| Persistencia | PostgreSQL + Sequelize |
| Estado | Propuesta para validación |

## 1. Objetivo y principios

AulaNexo se implementará como un monolito modular desplegable como un conjunto pequeño de contenedores. La aplicación expondrá una API REST que será consumida por dos clientes web: una implementación con JavaScript nativo y otra con React.

Principios:

- Separación de responsabilidades.
- Una sola fuente de verdad para reglas de negocio.
- Frontends intercambiables, backend compartido.
- Seguridad aplicada en servidor.
- Persistencia relacional e integridad transaccional.
- Contratos de API explícitos y versionados.
- Pruebas desde los primeros módulos.
- Infraestructura reproducible.
- Observabilidad básica y documentación mantenible.

No se proponen microservicios: su complejidad de despliegue y comunicación no aporta suficiente valor al objetivo didáctico del MVP.

## 2. Vista de contexto

Los actores interactúan con el portal público o con el portal privado. Los clientes privados envían solicitudes HTTP a la API; la API autentica, autoriza, valida y ejecuta casos de uso. La información persistente se almacena en PostgreSQL.

### Componentes principales

| Componente | Responsabilidad |
|---|---|
| Portal público | Información institucional y acceso |
| Frontend Vanilla | Primera implementación pedagógica |
| Frontend React | Segunda implementación por componentes |
| Nginx | Servir build web y enrutar /api |
| API Express | Interfaz HTTP y composición de módulos |
| Servicios de dominio | Reglas de negocio y casos de uso |
| Sequelize | Mapeo objeto-relacional y consultas |
| PostgreSQL | Persistencia de negocio y sesiones |
| GitHub Actions | Integración, validación y entrega |
| GHCR | Registro de imágenes de contenedor |

## 3. Vista lógica

Flujo principal:

Usuario → Frontend → HTTPS/HTTP + JSON + cookie de sesión → Nginx (en despliegue) → Express API → Middlewares → Routes → Controllers → Services → Models/Sequelize → PostgreSQL.

### Frontends

**Vanilla**
- HTML semántico.
- CSS responsivo.
- JavaScript modular.
- Fetch API.
- Manejo de formularios y estados de interfaz.
- La API es consumida mediante un módulo de servicios para evitar fetch dispersos.

**React**
- React + Vite.
- Componentes y composición.
- Props y estado local.
- Hooks.
- React Router.
- Servicios de API reutilizables.
- Formularios y estados de carga/error.
- Envío de cookies con credenciales según la configuración del origen.

Los dos frontends consumirán la misma API y no duplicarán reglas de negocio.

### Backend modular

El backend se organiza por dominios funcionales, por ejemplo:

- auth
- users
- students
- guardians
- teachers
- academics
- attendance
- grades
- tuition
- payments
- incidents
- services
- materials
- calendar
- audit

Cada módulo podrá contener rutas, controladores, validadores y servicios. Los modelos y asociaciones podrán estar centralizados o agrupados por dominio, manteniendo una única inicialización de Sequelize.

### Capas de responsabilidad

| Capa | Responsabilidad | No debe hacer |
|---|---|---|
| Routes | Método, path y middleware | Implementar reglas de negocio |
| Middleware | Autenticación, rol, CSRF y validación | Responder por cada caso de uso complejo |
| Controller | Traducir HTTP a entrada de servicio y mapear respuesta | Contener consultas/negocio extensos |
| Service | Reglas de negocio y transacciones | Depender del objeto Request/Response |
| Model/Repository | Persistencia y asociaciones | Decidir permisos HTTP |
| Config | Conexión y configuración | Guardar secretos en código |
| Utils | Funciones puras compartidas | Convertirse en depósito de lógica de dominio |

## 4. Stack tecnológico

| Área | Tecnología propuesta | Motivo |
|---|---|---|
| Estructura web | HTML5 | Semántica y fundamentos |
| Estilos | CSS3 | Diseño y responsividad |
| Frontend inicial | JavaScript nativo | Fundamentos de programación web |
| Frontend moderno | React + Vite | Componentes y estado |
| Backend | Node.js + Express | API REST y lógica de servidor |
| Base de datos | PostgreSQL | Persistencia relacional |
| ORM | Sequelize | Modelos, asociaciones, migraciones y transacciones |
| Validación | Zod (propuesta) | Esquemas reutilizables de entrada |
| Sesiones | express-session + connect-pg-simple | Sesiones persistentes en PostgreSQL |
| Contraseñas | bcrypt | Hash de contraseñas |
| Cabeceras | Helmet | Cabeceras de seguridad HTTP |
| Rate limiting | express-rate-limit o alternativa revisada | Limitar abuso de endpoints |
| Pruebas unitarias | Vitest | Reglas y unidades |
| Pruebas API | Supertest | Integración HTTP |
| Pruebas navegador | Playwright | Flujos end-to-end |
| Web server | Nginx | Servir frontend y proxy inverso |
| Contenedores | Docker + Docker Compose | Entorno reproducible |
| CI/CD | GitHub Actions | Automatización |
| Registro | GitHub Container Registry | Distribución de imágenes |
| Contrato API | OpenAPI | Documentación verificable |

Las versiones concretas de paquetes se elegirán al iniciar el repositorio de código y se fijarán mediante lockfile. Se deberán revisar compatibilidades y avisos de seguridad antes de instalar.

## 5. Autenticación basada en sesiones

Se utilizará autenticación mediante sesión de servidor y cookie, no JWT como mecanismo principal del navegador.

### Flujo de login

1. El usuario envía correo y contraseña.
2. Express normaliza/busca la cuenta.
3. El servidor verifica que la cuenta esté activa.
4. bcrypt compara la contraseña con password_hash.
5. Express regenera el identificador de sesión después de la autenticación.
6. Se guarda en sesión el user_id y los datos mínimos necesarios para identificar al usuario.
7. La cookie devuelve únicamente el identificador de sesión.
8. Las solicitudes posteriores envían la cookie.
9. El middleware recupera la sesión y establece el contexto de usuario.
10. El cierre de sesión destruye la sesión de servidor y elimina la cookie.

### Configuración de cookie

| Atributo | Política propuesta |
|---|---|
| HttpOnly | true |
| Secure | true en producción HTTPS |
| SameSite | Lax como línea base; validar el flujo CSRF y despliegue |
| Path | / |
| Max-Age / Expires | Vida útil definida y coherente con la sesión |
| Domain | Omitir salvo que exista necesidad explícita |
| Nombre | Nombre propio, no el predeterminado de Express |

No se utilizará MemoryStore en despliegue. Se usará un store persistente como connect-pg-simple respaldado por PostgreSQL.

### CSRF

Como el navegador enviará cookies automáticamente, las operaciones mutables deben protegerse contra CSRF. Se utilizará un token de sincronización asociado a la sesión: el frontend obtiene el token desde un endpoint de bootstrap y lo envía en un encabezado, por ejemplo X-CSRF-Token, en POST, PUT, PATCH y DELETE. La implementación exacta (biblioteca o middleware propio revisado) se definirá antes de implementar auth.

### CORS y origen

En producción se recomienda servir frontend y API bajo el mismo origen mediante Nginx: la ruta /api se redirige al servicio Express. Esto simplifica cookies. En desarrollo, si Vite y Express usan puertos diferentes, el proxy de Vite es preferible a habilitar CORS amplio. Si se configura CORS con credenciales, se debe permitir un origen explícito y nunca combinar credenciales con wildcard.

## 6. Autorización RBAC y acceso a recursos

La API aplicará autorización en tres pasos:

1. **Autenticación:** ¿existe una sesión válida?
2. **Rol:** ¿el rol puede realizar esta operación?
3. **Alcance del recurso:** ¿el usuario está relacionado con el alumno, grupo o asignación?

Ejemplos:

- Un TUTOR solo puede ver un student_id vinculado en student_guardians y con la capacidad requerida.
- Un DOCENTE solo puede leer o escribir sobre sus teacher_assignments activas.
- Un ALUMNO solo puede consultar su propio expediente permitido.
- ADMIN puede operar módulos administrativos, sujeto a reglas de negocio y auditoría.

Ocultar enlaces o botones en React/Vanilla mejora la experiencia, pero no constituye un control de seguridad. El backend debe rechazar toda operación no autorizada.

## 7. Validación, errores y persistencia

- Validar entradas de forma declarativa en el servidor.
- Repetir validaciones esenciales en base de datos mediante CHECK, UNIQUE y FK.
- Normalizar y limitar tamaños de campos.
- Usar consultas parametrizadas a través del ORM o SQL seguro.
- Centralizar el manejo de errores.
- No filtrar stack traces, secretos ni detalles internos al cliente.
- Usar transacciones para operaciones compuestas.
- No utilizar sync({ force: true }) en despliegue; el esquema se controla mediante migraciones.

### Transacciones relevantes

- Pago simulado: pago + actualización de cargo + auditoría.
- Calificación: modificación + auditoría.
- Asistencia masiva: validación del conjunto + persistencia masiva + auditoría.
- Emisión de citatorio: creación/actualización de estado + auditoría.

## 8. Documentación de API

- Prefijo: /api/v1.
- Formato de intercambio: JSON salvo descargas de documentos.
- Recursos y métodos REST consistentes.
- Contrato OpenAPI.
- Códigos HTTP y esquema de errores uniformes.
- Paginación y filtros para listados.
- Identificadores no sustituyen la comprobación de autorización.
- La documentación de endpoint deberá actualizarse al cambiar el comportamiento.

## 9. Testing y calidad

### Pirámide de pruebas propuesta

| Capa | Herramienta | Objetivo |
|---|---|---|
| Unitarias | Vitest | Cálculos, reglas y validaciones |
| Integración | Supertest + PostgreSQL de prueba | Rutas, sesión, permisos y persistencia |
| End-to-end | Playwright | Flujos reales en navegador |

Casos de seguridad prioritarios:
- Login incorrecto.
- Usuario inactivo.
- Acceso anónimo a ruta privada.
- Tutor que prueba un ID de alumno ajeno.
- Docente que intenta operar sobre grupo no asignado.
- Alumno que intenta acceder a recursos financieros.
- CSRF token ausente o inválido.
- Reutilización de una sesión después de logout.

## 10. Contenedores y topología de despliegue

### Desarrollo local

Docker Compose definirá los servicios de infraestructura necesarios. Puede ejecutarse el frontend con Vite fuera de Docker en desarrollo o contenerizarse cuando se quiera reproducir el entorno completo.

### Despliegue demostrativo

Topología propuesta:

- Servicio web Nginx: entrega el build de Vanilla o React y redirige /api.
- Servicio API: proceso Node.js/Express.
- Servicio PostgreSQL: persistencia de negocio y tabla de sesiones.
- Volumen: persistencia de datos PostgreSQL.
- Red interna: comunicación entre Nginx, API y DB.
- Variables y secretos: configuración por entorno; nunca incluidos en la imagen.

La base de datos no expondrá su puerto públicamente en el despliegue. Las imágenes tendrán tags versionados y se evitará depender exclusivamente de latest.

### Salud y operación

- Endpoint /health para health checks.
- Logs estructurados con nivel y contexto.
- No registrar contraseñas, cookies, tokens ni datos personales innecesarios.
- Migraciones ejecutadas como paso controlado de despliegue, no en paralelo desde cada réplica de API.
- Volúmenes y política de respaldo documentados; el MVP demostrará persistencia local, no alta disponibilidad.

## 11. CI/CD

### Continuous Integration

En cada Pull Request y push a ramas de trabajo:

1. Instalar dependencias desde lockfile.
2. Ejecutar lint/format check.
3. Ejecutar pruebas unitarias.
4. Levantar PostgreSQL de prueba y ejecutar migraciones.
5. Ejecutar pruebas de integración.
6. Construir frontend(s) y backend.
7. Construir imágenes de contenedor cuando corresponda.

Los controles obligatorios deberán pasar antes de merge.

### Continuous Delivery

Al integrar cambios a main o crear una versión:

1. Construir imágenes Docker.
2. Etiquetar por SHA y versión/release.
3. Publicar en GHCR con permisos mínimos.
4. Desplegar a un entorno de staging/demostración.
5. Ejecutar health check y smoke tests.
6. Promover a producción/demostración estable mediante aprobación o workflow protegido.

El entorno real de despliegue y sus secretos son decisiones pendientes. No se debe publicar una URL ni afirmar un despliegue hasta que exista infraestructura configurada.

## 12. Flujo Git recomendado

- main: rama estable.
- feat/<modulo>: funcionalidades.
- docs/<tema>: documentación.
- fix/<problema>: correcciones.
- Pull Requests con descripción, evidencia y checklist.
- Commits pequeños con mensajes descriptivos, por ejemplo feat, fix, docs, test, refactor y chore.
- CI requerida antes de merge.
- Releases etiquetados.

## 13. Estructura propuesta del monorepositorio

    mvp-escolar/
    ├── apps/
    │   ├── vanilla/
    │   └── react/
    ├── backend/
    │   └── src/
    │       ├── config/
    │       ├── middlewares/
    │       ├── modules/
    │       ├── models/
    │       ├── services/
    │       ├── routes/
    │       └── utils/
    ├── database/
    │   ├── migrations/
    │   └── seeders/
    ├── infrastructure/
    │   ├── docker/
    │   └── nginx/
    ├── tests/
    │   ├── unit/
    │   ├── integration/
    │   └── e2e/
    ├── docs/
    │   ├── requirements/
    │   ├── architecture/
    │   ├── database/
    │   └── api/
    ├── manual/
    ├── presentation/
    ├── videos/
    ├── .github/workflows/
    ├── docker-compose.yml
    ├── .env.example
    └── README.md

La estructura es una propuesta, no una exigencia para el primer commit de código. Las carpetas se crearán conforme se implementen los módulos.

## 14. Observabilidad y mantenimiento

Para el MVP:
- Logs estructurados de entrada/salida sin información sensible.
- Correlation/request ID para rastrear solicitudes.
- Manejo uniforme de errores.
- Endpoint de salud.
- Registro de acciones sensibles en audit_logs.
- Revisión de dependencias y actualización controlada.
- Documentación de variables de entorno.

Métricas, tracing distribuido, alertas y monitoreo avanzado quedan fuera del alcance inicial.

## 15. Riesgos arquitectónicos

| Riesgo | Mitigación |
|---|---|
| Acoplamiento entre frontend y reglas | Mantener reglas en services del backend |
| Permisos por rol insuficientes | Autorizar también por recurso |
| Sesión insegura | Store persistente, cookie segura y CSRF |
| Diferencias entre Vanilla y React | Contrato API único y pruebas de contrato/flujos |
| Cambios de esquema improvisados | Migraciones revisadas |
| Operaciones parcialmente confirmadas | Transacciones |
| Pipeline que despliega código no probado | Bloqueo de despliegue ante fallos CI |
| Secretos dentro de imágenes/repositorio | Secret manager/Actions secrets y .env local ignorado |

## 16. Decisiones pendientes

- Organización final del monorepo y gestor de paquetes.
- Versiones soportadas de Node.js, React y PostgreSQL.
- Estructura exacta de módulos y modelos Sequelize.
- Biblioteca y estrategia concreta de CSRF.
- Configuración definitiva de sesión y expiración.
- Gestión de archivos.
- Hosting de demostración.
- Política de promoción y rollback del despliegue.
- Retención y formato de logs.
