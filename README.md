# AulaNexo — Sistema de Gestión Escolar

AulaNexo es una plataforma web ficticia de gestión escolar para la Escuela Secundaria General Horizonte. El proyecto tiene un propósito funcional y didáctico: mostrar la evolución de una aplicación desde HTML, CSS y JavaScript nativo hasta una solución full stack con API REST, persistencia relacional, seguridad, pruebas, contenedores y CI/CD.

## Documentación de Sprint 0

- [Requerimientos del producto](docs/requirements/product-requirements.md)
- [Diseño de base de datos](docs/database/database-design.md)
- [Arquitectura del sistema](docs/architecture/system-architecture.md)
- [Contrato de API](docs/api/api-contract.md)

## Enfoque

- Dos frontends que consumen la misma API: Vanilla (HTML/CSS/JavaScript) y React.
- Backend modular con Node.js y Express.
- PostgreSQL como base de datos y Sequelize como ORM.
- Autenticación basada en sesiones de servidor y cookies HTTP.
- Autorización por roles y comprobación de acceso al recurso.
- Pruebas unitarias, de integración y end-to-end.
- Docker Compose para ejecución reproducible.
- GitHub Actions para integración y entrega continuas.

## Alcance y datos

AulaNexo es un proyecto demostrativo. Utiliza exclusivamente datos ficticios. Los pagos son simulados; las boletas, constancias y recibos son documentos de demostración y no documentos oficiales. No se integrarán pasarelas bancarias ni servicios gubernamentales en el MVP.

## Estado

**Sprint 0 — documentación y definición:** en preparación.  
Los documentos describen la línea base propuesta; los detalles marcados como decisiones por cerrar deberán validarse antes de iniciar la implementación.

## Inicio

Consulta primero el documento de requerimientos para conocer los objetivos, perfiles, módulos y criterios de aceptación. Después revisa arquitectura, modelo de datos y contrato de API.
