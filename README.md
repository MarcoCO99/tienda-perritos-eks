# Despliegue de Tienda de Perritos en Amazon EKS

Documentación del proceso de despliegue de la aplicación en un clúster de Kubernetes administrado por AWS (EKS), incluyendo la configuración de infraestructura, imágenes de contenedor, manifiestos de Kubernetes y verificación del sistema.

---

## 1. Configuración del Clúster AWS EKS

| Parámetro             | Valor                           |
| --------------------- | ------------------------------- |
| Nombre del clúster    | `tienda-perritos-eks`           |
| Versión de Kubernetes | 1.35                            |
| Proveedor             | Amazon EKS                      |
| Región                | `us-east-1` (Norte de Virginia) |

### Node Group

Se configuró un grupo de nodos con las siguientes características:

- **Tipo de instancia:** `t3.large`
- **Modelo de capacidad:** `SPOT` (reducción de costos en entorno de laboratorio)
- **Escalado dinámico:** mínimo 1 / deseado 1 / máximo 3 nodos

### IAM

Se asoció el rol `LabEKSNodeRole` a los nodos para otorgarles los permisos necesarios de comunicación con la red de AWS y registro en el clúster.

---

## 2. Repositorios ECR e Imágenes de Contenedor

Los repositorios fueron creados de forma privada en Amazon ECR bajo el ID de cuenta `139906254325`. Las imágenes se subieron manualmente con el tag `eks-v1`.

```
139906254325.dkr.ecr.us-east-1.amazonaws.com/tienda-frontend:eks-v1
139906254325.dkr.ecr.us-east-1.amazonaws.com/tienda-backend:eks-v1
139906254325.dkr.ecr.us-east-1.amazonaws.com/tienda-db:eks-v1
```

### Comandos de inicialización de repositorios ECR

```bash
aws ecr create-repository --repository-name tienda-frontend --region us-east-1
aws ecr create-repository --repository-name tienda-backend --region us-east-1
aws ecr create-repository --repository-name tienda-db --region us-east-1
```

### Autenticación y push de imágenes

```bash
aws ecr get-login-password --region us-east-1 \
  | docker login --username AWS --password-stdin \
    139906254325.dkr.ecr.us-east-1.amazonaws.com

docker tag tienda-frontend:latest \
  139906254325.dkr.ecr.us-east-1.amazonaws.com/tienda-frontend:eks-v1
docker push 139906254325.dkr.ecr.us-east-1.amazonaws.com/tienda-frontend:eks-v1

# Repetir para backend y db
```

---

## 3. Arquitectura de Despliegue en Kubernetes

Todos los recursos se desplegaron dentro del namespace `tienda` para mantener aislamiento lógico. Los manifiestos se encuentran en la carpeta `k8s/` y se aplicaron con:

```bash
kubectl apply -f k8s/ -n tienda
```

### Base de datos (MySQL)

Desplegada mediante un `Deployment` de MySQL. Las credenciales se gestionan a través de un `Secret` de Kubernetes para evitar exponer la contraseña `MYSQL_ROOT_PASSWORD` en texto plano dentro de los manifiestos.

Archivo: `k8s/mysql-secret.yaml`, `k8s/mysql-deployment.yaml`

### Backend

Orquestado para conectar la lógica de negocio con la base de datos a través del servicio interno `tienda-backend`.

Archivo: `k8s/backend-deployment.yaml`

### Frontend

Expuesto al exterior mediante un servicio de tipo `LoadBalancer`, el cual provisionó automáticamente un balanceador de carga clásico (ELB) en AWS.

Archivo: `k8s/frontend-deployment.yaml`

---

## 4. Historial de Cambios Relevantes

### `fix: actualizar referencias de imagen en manifiestos k8s`

Se modificaron los archivos `frontend-deployment.yaml`, `backend-deployment.yaml` y `mysql-deployment.yaml` para reemplazar el ID de cuenta genérico de la pauta por el ID real (`139906254325`) y establecer el tag `eks-v1` en todas las referencias a imágenes ECR.

```yaml
# Antes
image: <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/tienda-frontend:latest

# Despues
image: 139906254325.dkr.ecr.us-east-1.amazonaws.com/tienda-frontend:eks-v1
```

### `feat: creacion de repositorios privados en Amazon ECR`

Registro de comandos CLI utilizados para crear los repositorios de imágenes en ECR previo al proceso de build y push.

---

## 5. Verificación del Sistema

### Conexión al clúster

```bash
aws eks update-kubeconfig --name tienda-perritos-eks --region us-east-1
```

### Verificación de nodos y métricas

```bash
kubectl get nodes -n tienda
kubectl top nodes
```

Consumos registrados en estado estable:

- CPU: 2% a 4%
- Memoria: 15% a 25%

### Logs del plano de control

Los flujos de auditoría del plano de control están habilitados en Amazon CloudWatch bajo el grupo:

```
/aws/eks/tienda-perritos-eks/cluster
```

### URL publica de acceso

La aplicación quedo disponible a traves del DNS del balanceador de carga:

```
k8s-tienda-tiendafr-7e107c9aeb-437c78f88703d838.elb.us-east-1.amazonaws.com
```
