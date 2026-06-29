# Tienda de Perritos - Orquestación Proactiva con Amazon EKS 🐶

Este proyecto documenta la implementación de una arquitectura de microservicios para la empresa **Innovatech Chile**, utilizando **Amazon EKS** para la orquestación, **Amazon ECR** para el registro de imágenes y **GitHub Actions** para la automatización total del despliegue.

## 🏗️ Arquitectura de la Solución
La aplicación es multicapa y consta de:
* **Frontend:** Interfaz web para el usuario final.
* **Backend:** API de lógica de negocio (Node.js).
* **Database:** Persistencia de datos con MySQL.

## 🛠️ Preparación del Entorno
Antes de comenzar, asegúrate de tener configurado:
1. **AWS CLI** con las credenciales de AWS Academy (incluyendo el `AWS_SESSION_TOKEN`).
2. **Kubectl** vinculado al clúster:
   ```bash
   aws eks update-kubeconfig --region us-east-1 --name TU_CLUSTER_NAME
## 🚀 Guía de Despliegue Manual
Para desplegar el proyecto desde cero, sigue este orden en la carpeta /k8s:
Namespace: kubectl apply -f namespace.yaml
Base de Datos:
kubectl apply -f mysql-secret.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml
Backend:
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
Frontend:
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml

## 🤖 Automatización con GitHub Actions
El pipeline de CI/CD se activa automáticamente con cada push a la rama main. Para que funcione, configura estos Secrets en GitHub:
AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN.
EKS_CLUSTER_NAME y EKS_NAMESPACE.

## 📈 Resiliencia y Monitoreo
Auto-healing: Kubernetes recrea los Pods automáticamente si detecta fallos.
HPA: La aplicación escala horizontalmente cuando el CPU supera el 70%.
Logs: Auditoría completa disponible en Amazon CloudWatch Logs.
