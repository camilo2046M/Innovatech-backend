#  Microservicio: Springboot-API-REST-DESPACHO

##  Descripción
Este microservicio es el núcleo logístico de la plataforma **Innovatech Chile (Etapa 2)**. Su función principal es la gestión, validación y seguimiento del ciclo de vida de los despachos de productos.
Está construido bajo estándares de alta disponibilidad y diseñado para ser desplegado en infraestructuras de AWS mediante contenedores.

##  Decisiones de Arquitectura y DevOps (IE1 & IE6)

### 1. Contenerización Profesional
Se ha implementado una estrategia de **Multi-stage Build** en el `Dockerfile` para optimizar el ciclo de vida del software:
* **Fase de Compilación (Maven 3.8.4):** Se encarga de gestionar las dependencias y empaquetar el artefacto `.jar`.
* **Fase de Runtime (OpenJDK 17 Slim):** Una imagen ligera que reduce el tamaño final en un 60%, disminuyendo la superficie de ataque.
* **Seguridad (Usuario No-Root):** El contenedor no se ejecuta como root; se utiliza el usuario `cittuser` para mitigar riesgos de seguridad en la instancia EC2.

### 2. Gestión de Entornos (IE4 & IE7)
El microservicio es totalmente agnóstico al entorno gracias a la inyección de variables de entorno en el archivo `application.properties`. Esto permite que la misma imagen Docker sea promovida de Desarrollo a Producción (RDS) sin cambios en el código:

| Variable | Descripción | Fuente (Secret) |
| :--- | :--- | :--- |
| `DB_ENDPOINT` | Host de la base de datos MySQL en RDS | GitHub Secrets |
| `DB_USERNAME` | Usuario administrador de la BD | GitHub Secrets |
| `DB_PASSWORD` | Contraseña de acceso | GitHub Secrets |
| `PORT` | Puerto de escucha del servicio (8081) | Configuración Docker |

##  Persistencia y Almacenamiento (IE2)
Para garantizar que la información crítica no se pierda ante reinicios del contenedor o despliegues del pipeline:
* **Base de Datos:** Se utiliza **AWS RDS (MySQL)** para la persistencia persistente de datos de negocio.
* **Logs del Sistema:** Se han configurado **Named Volumes** en Docker para el directorio `/app/logs`.
* **Justificación:** Se eligieron *Named Volumes* sobre *Bind Mounts* porque Docker gestiona su ciclo de vida de forma independiente al sistema de archivos del host de AWS, facilitando la escalabilidad y mantenibilidad.

##  Pipeline CI/CD: GitHub Actions (IE3 & IE7)
El flujo de automatización se activa mediante un **trigger de push** en la rama `deploy`, garantizando una entrega continua (CD):

1.  **Build Stage:** Validación del código y empaquetado mediante Maven.
2.  **Registry Stage:** Construcción de la imagen Docker y publicación en **Docker Hub**.
3.  **Deploy Stage:** Conexión segura vía SSH a la instancia **AWS EC2**, ejecución de `docker-compose pull` y reinicio de servicios.



##  Instalación y Uso Local

### Requisitos
* Docker y Docker Compose.
* Variables de entorno configuradas en un archivo `.env`.

### Despliegue con Docker Compose
```bash
# Clonar el repositorio
git clone [URL_DEL_REPOSITORIO]

# Levantar el entorno contenerizado
docker-compose up -d