Tienda de Perritos - Orquestación Proactiva con Amazon EKS
Este proyecto representa la transición de la empresa Innovatech Chile hacia una arquitectura de microservicios escalable, resiliente y automatizada en la nube
. Se utiliza Amazon EKS para la orquestación de contenedores y GitHub Actions para el despliegue continuo (CI/CD)
.
🏗️ Arquitectura del Sistema
La solución implementa una aplicación multicapa compuesta por:
Frontend: Interfaz de usuario que se comunica con el backend.
Backend: API de lógica de negocio.
Base de Datos: Instancia de MySQL para persistencia de datos
.
Infraestructura en AWS:
Amazon EKS: Clúster administrado con nodos worker tipo T3.LARGE en modalidad SPOT
.
Amazon ECR: Repositorio privado para el almacenamiento de imágenes versionadas (eks-v1)
.
VPC & Networking: Subredes públicas y privadas con balanceo de carga mediante Application Load Balancer (ALB)
.
Amazon CloudWatch: Monitoreo detallado de logs del plano de control y métricas de rendimiento
.
🚀 Tecnologías Utilizadas
Orquestador: Kubernetes (v1.34) vía Amazon EKS
.
Contenedores: Docker
.
CI/CD: GitHub Actions
.
Infraestructura como Código: Manifiestos YAML (Deployments, Services, HPA, Secrets)
.
🛠️ Configuración y Despliegue
Requisitos Previos
AWS CLI configurado con credenciales de AWS Academy
.
kubectl instalado y vinculado al clúster
.
Docker Desktop operativo
.
Variables de Entorno (Secrets de GitHub)
Para que el pipeline funcione, deben configurarse los siguientes Actions Secrets en el repositorio
:
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN (Obligatorio para AWS Academy)
AWS_REGION (us-east-1)
EKS_CLUSTER_NAME
EKS_NAMESPACE (tienda)
Pasos de Instalación Manual
Vincular clúster:
Desplegar recursos:
📈 Resiliencia y Escalabilidad
El clúster ha sido validado bajo las siguientes pruebas de estrés y fallos
:
Auto-healing: Kubernetes detecta fallos en los Pods y los recrea automáticamente para mantener el estado deseado
.
HPA (Horizontal Pod Autoscaler): El sistema escala automáticamente el número de réplicas de los Pods cuando el uso de CPU supera el umbral definido (ej. 70%)
.
📊 Monitoreo y Observabilidad
La salud del sistema se supervisa mediante:
Métricas en tiempo real: Uso de kubectl top pods y kubectl top nodes
.
Logs del Plano de Control: Auditoría de api, scheduler y controllerManager en CloudWatch Logs
.
