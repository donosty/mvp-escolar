# AulaNexo — Sistema de Gestión Escolar

AulaNexo es una plataforma web ficticia de gestión escolar para la Escuela Secundaria General Horizonte. El proyecto tiene un propósito funcional y didáctico: mostrar la evolución de una aplicación desde HTML, CSS y JavaScript nativo hasta una solución full stack con API REST, persistencia relacional, seguridad, pruebas, contenedores y CI/CD.

## Documentación de Sprint 0

1. [Requerimientos del producto](docs/requirements/product-requirements.md)
2. [Diseño de base de datos](docs/database/database-design.md)
3. [Arquitectura del sistema](docs/architecture/system-architecture.md)
4. [Contrato de API](docs/api/api-contract.md)

## Enfoque tecnológico

- Dos frontends que consumen la misma API: Vanilla (HTML/CSS/JavaScript) y React + Vite.
- Backend modular con Node.js y Express.
- PostgreSQL como base de datos y Sequelize como ORM.
- Autenticación mediante sesiones de servidor y cookies HTTP.
- Autorización por roles y verificación de acceso al recurso.
- Vitest, Supertest y Playwright para pruebas.
- Docker Compose y Nginx para ejecución reproducible.
- GitHub Actions y GHCR para CI/CD y publicación de imágenes.

## Alcance y datos

AulaNexo es un proyecto demostrativo. Utiliza exclusivamente datos ficticios. Los pagos son simulados; las boletas, constancias y recibos son documentos de demostración y no documentos oficiales. No se integrarán pasarelas bancarias ni servicios gubernamentales en el MVP.

## Estado del proyecto

**Sprint 0 — documentación y definición.** Los documentos establecen la línea base propuesta. Los puntos marcados como decisiones pendientes deberán validarse antes de iniciar la implementación.

## Orden de lectura

Comienza con los requerimientos para conocer los objetivos, perfiles y funcionalidades. Continúa con arquitectura, modelo de datos y contrato de API. Los cuatro documentos deben evolucionar junto con el código y mantenerse sincronizados mediante Pull Requests.
