#  Microservicio: Springboot-API-REST (Ventas)

##  Descripción
Este microservicio gestiona el flujo de transacciones comerciales de **Innovatech Chile**. Permite el registro de ventas, validación de datos y el control de los estados de despacho asociados a cada compra.

## Arquitectura de Contenedorización (IE1 & IE6)

Se implementó una arquitectura de contenedores basada en el estándar **Docker**, aplicando los siguientes principios DevOps:
* [cite_start]**Multi-Stage Build:** Separa el entorno de construcción (Maven) del de ejecución (JRE), reduciendo el peso de la imagen y eliminando herramientas de compilación innecesarias en producción.
* [cite_start]**Seguridad:** El servicio se ejecuta bajo un usuario de sistema limitado (`innovatech`), mitigando riesgos de escalada de privilegios en el host de AWS[cite: 88].
* **Optimización de Capas:** Se utiliza la distribución **Alpine Linux** por su huella mínima de almacenamiento y mayor seguridad.

##  Gestión de Datos y Persistencia (IE2)

El microservicio utiliza un enfoque de **Persistencia Híbrida**:
1.  **Producción:** Conectividad con **AWS RDS (MySQL)** gestionada mediante variables de entorno para máxima seguridad.
2.  **Testing:** Implementación de **H2 Database (In-memory)** para la ejecución de pruebas unitarias durante el pipeline de CI/CD, asegurando que los tests no afecten los datos productivos.
3.  **Volúmenes:** Se utilizan volúmenes para la persistencia de logs de auditoría de ventas, garantizando la trazabilidad ante reinicios del contenedor.

##  Pipeline de Despliegue (CI/CD) (IE3 & IE7)

El proceso de automatización reside en **GitHub Actions** y se dispara mediante la rama `deploy`:
* **Build & Test:** Validación del código y ejecución de pruebas con perfil `test`.
* **Push:** Publicación de la imagen en el registro de contenedores.
* **Deploy:** Actualización en caliente de la instancia **EC2** vía SSH, garantizando una entrega continua con mínimo tiempo de inactividad.

| Secret Key | Función |
| :--- | :--- |
| `DB_ENDPOINT` | URL del clúster RDS en AWS |
| `DB_USERNAME` | Credenciales de acceso a la base de datos |
| `DB_PASSWORD` | Token de seguridad para la conexión |

##  Endpoints de la API
* **Base URL:** `/api/v1/ventas`
* **Documentación Interactiva:** Acceso vía Swagger UI en `/swagger-ui.html`.
* [cite_start]**Seguridad:** Implementación de **CORS** para permitir la comunicación segura con el Frontend empaquetado vía NPM[cite: 82, 88].

###  Estrategia de Testing y Calidad (IE8)
Este microservicio incluye una configuración de **Doble Persistencia**:
* **H2 (In-Memory):** Se utiliza exclusivamente para el entorno de testing (`application-test.properties`). Esto permite validar la lógica de ventas en el Pipeline de CI/CD sin necesidad de una base de datos real.
* **MySQL (RDS):** Se utiliza para el entorno productivo.
* **Beneficio:** Garantiza la **Trazabilidad** y calidad del código antes de cada despliegue, asegurando que solo el código que pasa los tests llegue a la instancia EC2.