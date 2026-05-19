# Tienda Perritos - Sistema de Gestión (Frontend + Backend + DB)

Este repositorio contiene el código fuente y la configuración de infraestructura para el sistema "Tienda Perritos". La aplicación está diseñada con una arquitectura de microservicios, contenedorizada mediante Docker y configurada para despliegue automatizado en AWS a través de GitHub Actions.

## Arquitectura del Sistema

El proyecto se divide en tres componentes principales, cada uno ejecutándose en su propio contenedor:

1. **Frontend (Nginx):** Aplicación SPA servida a través de Nginx. Nginx actúa como proxy inverso, interceptando las peticiones a `/api/` y redirigiéndolas al backend a través de la red privada. Construido usando Dockerfile multi-stage.
2. **Backend (Node.js/Express):** API RESTful que gestiona la lógica de negocio y la conexión a la base de datos. Construido usando Dockerfile multi-stage con usuario sin privilegios (`node`) para mayor seguridad.
3. **Base de Datos (MySQL 8):** Almacenamiento relacional. Utiliza volúmenes nombrados (`named volumes`) de Docker para garantizar la persistencia de los datos ante reinicios o actualizaciones del contenedor.

### Infraestructura en AWS

- **VPC y Subredes:** Despliegue en una VPC con subred pública.
- **EC2:** Tres instancias (t2.micro) con Amazon Linux 2023, gestionadas mediante el rol `LabInstanceProfile`.
- **Seguridad:** Grupos de Seguridad (Security Groups) configurados para permitir tráfico HTTP/SSH al Frontend desde internet, y tráfico interno restringido entre Frontend → Backend (puerto 3001) y Backend → BD (puerto 3306).

## Ejecución Local (Entorno de Desarrollo)

Para levantar el proyecto completo en un entorno local, se proporciona un archivo `docker-compose.yml` que orquesta los tres servicios y crea una red interna (`tienda-network`).

**Requisitos previos:**
- Docker y Docker Compose instalados.

**Pasos:**

1. Clona el repositorio.
2. En la raíz del proyecto, ejecuta:
   ```bash
   docker-compose up -d --build
   ```
3. Accede a la aplicación en tu navegador: `http://localhost`
4. Para detener y limpiar los contenedores:
   ```bash
   docker-compose down
   ```

> **Nota:** La persistencia de datos local se maneja automáticamente mediante el volumen `dbdata` definido en el compose.

## Pipeline de CI/CD (GitHub Actions)

El despliegue está automatizado mediante tres workflows en GitHub Actions.

**Flujo de ejecución:**

1. **Trigger:** El pipeline se activa automáticamente al realizar un `push` en la rama `main` o `deploy` que afecte los directorios `frontend/`, `backend/` o `db/`.
2. **Build:** Se construye la imagen Docker correspondiente basada en el Dockerfile optimizado.
3. **Push:** La imagen se etiqueta y se sube al registro privado de Amazon ECR.
4. **Deploy:** Se utiliza AWS Systems Manager (SSM) para enviar comandos remotamente a la instancia EC2 objetivo. El agente SSM detiene el contenedor antiguo, descarga la nueva imagen desde ECR y levanta el nuevo contenedor inyectando las variables de entorno necesarias.

**Secretos requeridos en GitHub:** Configurar en `Settings > Secrets and variables > Actions`:

| Secreto | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Credenciales AWS |
| `AWS_SECRET_ACCESS_KEY` | Credenciales AWS |
| `AWS_SESSION_TOKEN` | Credenciales AWS |
| `AWS_REGION` | Configuración regional |
| `ECR_REGISTRY` | URI del registro ECR |
| `ECR_REPO_URL_FRONTEND` | URI del repo ECR del frontend |
| `ECR_REPO_URL_BACKEND` | URI del repo ECR del backend |
| `ECR_REPO_URL_DB` | URI del repo ECR de la BD |
| `EC2_FRONTEND_INSTANCE_ID` | ID de instancia EC2 del frontend |
| `EC2_BACKEND_INSTANCE_ID` | ID de instancia EC2 del backend |
| `EC2_DB_INSTANCE_ID` | ID de instancia EC2 de la BD |
| `DB_HOST` | IP privada de la base de datos |

## Convenciones de Commits

| Prefijo | Descripción |
|---|---|
| `feat:` | Nuevas características o funcionalidades |
| `fix:` | Corrección de errores |
| `update:` | Actualizaciones de dependencias o ajustes menores |
| `docs:` | Modificaciones en la documentación |
| `infra:` | Cambios en Dockerfiles, compose o workflows de CI/CD |