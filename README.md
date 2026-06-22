1. Flujo y arquitectura del sistema
El proyecto se divide en 2 grandes conceptos que funcionan en conjunto con el fin de automatizar todo el deploy.

1.1 Automatización con Github actions(CI/CD)

a. El desarrollador hace un push el cual Github Actions detecta y activa el archivo deploy.yml en la carpeta workflows el cual lee los secrets (credenciales de aws) definidos en el archivo para poder acceder a la cuenta aws

b. Github Actions detecta y compila los Dockerfile de las 3 capas para crear las imagenes de lo contenedores y despues subirla a un repositorio de imagenes ECR privado 

c. Github Actions notifica a ECS sobre la nueva imagen, aws la toma y actualiza el Task Definition para refrescar los contenedores en producción.

1.2 Infraestructura en AWS 
El proyecto es un modelo de 3 capas tradicional el cual separa Frontend, Backend y Base de datos.
Todo el proyecto vive dentro de una VPC (virtual private cloud) segura y aislada en su respectiva región de AWS

- Capa Frontend: El contenedor de esta capa corre de manera automatizada con Fargate y expone el puerto 80. Cualquier usuario puede acceder a la interfaz del front

-Capa Backend: El contenedor de esta capa igualmente corre de forma automatizada con Fargate, pero expone el puerto 3001. Gracias al security group con su regla de entrada HTTP, solo permite peticiones directamente desde el Frontend que expuso ese puerto HTTP

-Capa Base De Datos: El contenedor de esta capa esta gestionado por una EC2 el cual corre en el puerto 3006, cuenta con un security group el cual solo permite peticiones desde la capa backend 

2. Componentes del proyecto:
Infraestructura AWS 
- ECS Cluster Fargate
- Application Load Balancer (ALB)
- RDS MySQL
- ECR (Elastic Container Registry)
- Secrets Manager
- Security Groups
- IAM Roles
- CloudWatch Logs

CI/CD
- GitHub Actions Workflows
- Docker Images
- ECR Push/Pull

Monitoreo
- CloudWatch Logs
- CloudWatch Metrics
- CloudWatch Container Insights

3. Requisitos previos 
Para Desarrollo Local
- Git
- Docker Desktop

Para Despliegue en AWS
- AWS Account (Academy)
- AWS CLI configurada
- Credenciales de AWS (Access Key ID + Secret Access Key)

Para CI/CD
- Repositorio GitHub con acceso para crear Actions
- Secrets configurados en GitHub (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)

4. Variables de entorno
Para compilación local solo se necesita saber la direccion ip pública de la api dentro del frontend

VITE_API_URL=http://localhost:3001

Para producción AWS ECS / Fargate
- VITE_API_URL=http://<IP_PUBLICA_O_DNS_DEL_BACKEND>:3001

Backend:
PORT=3001
DB_HOST=bd          
DB_PORT=3306
DB_USER=root
DB_PASSWORD=tu_clave_local
DB_NAME=innovatech

Para producción AWS ECS / EC2
PORT=3001
DB_HOST=<IP_PRIVADA_DE_LA_INSTANCIA_EC2_BD>
DB_PORT=3306
DB_USER=root        # O el usuario que definieras en AWS
DB_PASSWORD=tu_clave_segura_aws
DB_NAME=innovatech  

5. Configuración de secretos en Github (Github secrets)
Este conjunto de secretos son vitales para la conexión de aws con Github Actions

- AWS_ACCESS_KEY_ID: Clave de acceso de la cuenta AWS
- AWS_SECRET_ACCESS_KEY: Clave secreta de AWS
- AWS_SESSION_TOKEN: Token temporal (Obligatorio en laboratorios)
- AWS_REGION: Región de AWS donde se despliega

6. Pipeline CI/CD (Workflow)
El proyecto utiliza GitHub Actions para automatizar el ciclo de CI/CD. Contamos con tres pipelines que comparten un mismo flujo base, diferenciándose únicamente sus configuraciones específicas.

Flujo común de los 3 pipeline:
- Descargar código: Se inicializa un entorno limpio en un servidor virtual de GitHub (ubuntu-latest) y se clona la versión más reciente del código.

- Autenticación en AWS: Se configuran las credenciales temporales del laboratorio inyectando de forma segura los GitHub Secrets

- Inicio de sesión en ECR:Docker se autentica contra Amazon Elastic Container Registry para obtener los permisos de subida de imágenes

- Construcción y despliegue (Build and push): se ejecuta el Build de la imagen con el archivo Dockerfile y se sube el paquete en el repositorio de AWS ECR con la etiqueta :latest

7. Verificación del despliegue:
simplemente mediante la ip pública del frontend en el navegador deberiamos ser capaces de verificar si es que el deploy fue exitoso.
http://<ip_publica>:3001

Endpoint de prueba: http://<ip_publica>:3001/api/items
