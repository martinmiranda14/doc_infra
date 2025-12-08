# Introducción a Kubernetes

## 📖 ¿Qué es Kubernetes?

Kubernetes (también conocido como **K8s**) es un sistema de orquestación de contenedores open-source que automatiza el despliegue, escalado y gestión de aplicaciones en contenedores.

**Origen:** Desarrollado originalmente por Google, basado en más de 15 años de experiencia ejecutando cargas de trabajo en producción.

```
┌─────────────────────────────────────────────┐
│          Kubernetes Cluster                 │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   Pod    │  │   Pod    │  │   Pod    │  │
│  │ App + DB │  │   API    │  │   Web    │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│        ▲              ▲              ▲      │
│        └──────────────┴──────────────┘      │
│                   │                         │
│         Kubernetes Control Plane            │
│  (Scheduler, Controller, API Server)        │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 🎯 ¿Por Qué Kubernetes?

### Problemas que Resuelve

#### Sin Kubernetes (❌)

```bash
# Gestión manual de contenedores en múltiples servidores
ssh server1
docker run -d app:v1

ssh server2
docker run -d app:v1

# ¿Qué pasa si un contenedor falla?
# ¿Cómo balancear carga?
# ¿Cómo actualizar sin downtime?
# ¿Cómo escalar a 100 instancias?
```

#### Con Kubernetes (✅)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3  # 3 instancias automáticas
  template:
    spec:
      containers:
      - name: app
        image: app:v1

# Kubernetes automáticamente:
# ✅ Distribuye pods en nodos disponibles
# ✅ Re-crea pods si fallan
# ✅ Balancea carga entre instancias
# ✅ Actualiza sin downtime
# ✅ Escala horizontal automáticamente
```

---

## 💡 Conceptos Principales

### 1. Cluster

Un cluster de Kubernetes consiste en un conjunto de máquinas (nodos) que ejecutan aplicaciones en contenedores.

```
Cluster = Control Plane + Worker Nodes
```

### 2. Node (Nodo)

Una máquina física o virtual que ejecuta contenedores.

**Tipos:**
- **Control Plane Nodes (Master):** Gestionan el cluster
- **Worker Nodes:** Ejecutan las aplicaciones

### 3. Pod

La **unidad mínima** de despliegue en Kubernetes. Un pod puede contener uno o más contenedores.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

**Características:**
- Comparte red (misma IP)
- Comparte almacenamiento
- Efímero (puede ser recreado en cualquier momento)

### 4. Deployment

Gestiona el estado deseado de los pods (réplicas, actualizaciones, rollbacks).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### 5. Service

Expone una aplicación corriendo en pods como un servicio de red.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

### 6. Namespace

Mecanismo para aislar grupos de recursos dentro de un cluster.

```bash
# Namespaces por defecto
kubectl get namespaces

# Salida típica:
default           # Recursos sin namespace específico
kube-system       # Recursos del sistema K8s
kube-public       # Recursos públicos
kube-node-lease   # Heartbeats de nodos
```

---

## 🏗️ Arquitectura Básica

```
┌────────────────────────────────────────────────┐
│           KUBERNETES CLUSTER                   │
├────────────────────────────────────────────────┤
│                                                │
│  ┌─────────────────────────────────────────┐  │
│  │      CONTROL PLANE (Master)             │  │
│  ├─────────────────────────────────────────┤  │
│  │  • API Server (kube-apiserver)          │  │
│  │  • etcd (almacenamiento)                │  │
│  │  • Scheduler (kube-scheduler)           │  │
│  │  • Controller Manager                    │  │
│  │  • Cloud Controller Manager (opcional)  │  │
│  └─────────────────────────────────────────┘  │
│                      │                         │
│                      │ API                     │
│         ┌────────────┼────────────┐            │
│         │            │            │            │
│  ┌──────▼─────┐ ┌───▼──────┐ ┌──▼────────┐   │
│  │   Node 1   │ │  Node 2  │ │  Node 3   │   │
│  ├────────────┤ ├──────────┤ ├───────────┤   │
│  │ • kubelet  │ │ • kubelet│ │ • kubelet │   │
│  │ • kube-    │ │ • kube-  │ │ • kube-   │   │
│  │   proxy    │ │   proxy  │ │   proxy   │   │
│  │ • Pods     │ │ • Pods   │ │ • Pods    │   │
│  └────────────┘ └──────────┘ └───────────┘   │
│                                                │
└────────────────────────────────────────────────┘
```

### Componentes del Control Plane

**API Server (kube-apiserver)**
- Punto de entrada al cluster
- Expone la API de Kubernetes (REST)
- Todo pasa por aquí

**etcd**
- Base de datos clave-valor
- Almacena todo el estado del cluster
- Debe tener backups!

**Scheduler (kube-scheduler)**
- Decide en qué nodo ejecutar cada pod
- Considera recursos, políticas, afinidad

**Controller Manager**
- Ejecuta controladores (loops de control)
- Ejemplos: Node Controller, Replication Controller

### Componentes de Worker Node

**kubelet**
- Agente que corre en cada nodo
- Asegura que los contenedores estén corriendo
- Se comunica con API Server

**kube-proxy**
- Mantiene reglas de red en los nodos
- Permite comunicación de red a pods

**Container Runtime**
- Software que ejecuta contenedores
- Ejemplos: containerd, CRI-O, Docker (deprecated)

---

## 🚀 ¿Cuándo Usar Kubernetes?

### ✅ Casos de Uso Ideales

**1. Microservicios**
- Múltiples servicios independientes
- Necesidad de escalar servicios individualmente
- Comunicación entre servicios

**2. Aplicaciones Cloud-Native**
- Aplicaciones stateless
- Diseñadas para contenedores
- Requieren alta disponibilidad

**3. CI/CD Avanzado**
- Despliegues frecuentes
- Canary/Blue-Green deployments
- A/B testing

**4. Multi-Cloud/Híbrido**
- Portabilidad entre clouds
- Mismo stack en on-premise y cloud
- Evitar vendor lock-in

**5. Alto Tráfico Variable**
- Auto-scaling basado en demanda
- Black Friday, eventos especiales
- Tráfico impredecible

### ❌ Cuándo NO Usar Kubernetes

**Aplicaciones simples**
- Monolitos pequeños
- Bajo tráfico predecible
- Docker Compose puede ser suficiente

**Equipo pequeño sin expertise**
- Curva de aprendizaje empinada
- Requiere operación y mantenimiento
- Overhead operacional alto

**Recursos limitados**
- Control plane requiere recursos
- Mínimo recomendado: 3 nodos
- Alternativas más ligeras: k3s, k0s

**Aplicaciones legacy sin modificar**
- Mejor usar VMs
- Kubernetes requiere adaptar aplicaciones
- Stateful apps complejas difíciles

---

## 💻 Opciones de Instalación

### 1. Desarrollo Local

#### minikube (Más Popular)

```bash
# Instalación (macOS)
brew install minikube

# Iniciar cluster
minikube start

# Verificar
kubectl get nodes

# Dashboard web
minikube dashboard
```

**Características:**
- ✅ Fácil de usar
- ✅ Soporta múltiples drivers (Docker, VirtualBox, etc.)
- ✅ Add-ons útiles (ingress, metrics-server)
- ❌ Solo un nodo

#### kind (Kubernetes IN Docker)

```bash
# Instalación
brew install kind

# Crear cluster
kind create cluster

# Cluster multi-nodo
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```

**Características:**
- ✅ Muy rápido
- ✅ Múltiples nodos
- ✅ Ideal para CI/CD
- ❌ Requiere Docker

#### k3s (Ligero)

```bash
# Instalación
curl -sfL https://get.k3s.io | sh -

# Verificar
sudo k3s kubectl get nodes
```

**Características:**
- ✅ Muy ligero (~70MB)
- ✅ Ideal para edge/IoT
- ✅ Producción en recursos limitados
- ✅ Incluye Traefik, CoreDNS

### 2. Cloud Managed

#### GKE (Google Kubernetes Engine)

```bash
# Crear cluster
gcloud container clusters create my-cluster \
  --num-nodes=3 \
  --zone=us-central1-a

# Configurar kubectl
gcloud container clusters get-credentials my-cluster
```

**Ventajas:**
- ✅ Control plane gratis
- ✅ Auto-scaling nodos
- ✅ Integración con Google Cloud

#### EKS (Amazon Elastic Kubernetes Service)

```bash
# Crear cluster (usando eksctl)
eksctl create cluster \
  --name my-cluster \
  --region us-west-2 \
  --nodes 3

# Configurar kubectl
aws eks update-kubeconfig --name my-cluster
```

**Ventajas:**
- ✅ Integración con AWS
- ✅ Seguridad IAM nativa
- ✅ Fargate para serverless pods

#### AKS (Azure Kubernetes Service)

```bash
# Crear cluster
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3

# Configurar kubectl
az aks get-credentials \
  --resource-group myResourceGroup \
  --name myAKSCluster
```

**Ventajas:**
- ✅ Integración con Azure
- ✅ Azure AD para autenticación
- ✅ Azure Monitor integrado

### 3. On-Premise

#### kubeadm (Oficial)

```bash
# En todos los nodos
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# En el master
sudo kubeadm init

# En workers
sudo kubeadm join <master-ip>:6443 --token <token>
```

#### Rancher

- UI web para gestionar múltiples clusters
- Instalación simplificada
- Catálogo de aplicaciones

---

## 🛠️ kubectl - La Herramienta CLI

### Instalación

```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verificar
kubectl version --client
```

### Comandos Básicos

```bash
# Ver información del cluster
kubectl cluster-info

# Ver nodos
kubectl get nodes

# Ver pods
kubectl get pods

# Ver todos los recursos
kubectl get all

# Ver en todos los namespaces
kubectl get pods -A

# Ver detalles de un recurso
kubectl describe pod <pod-name>

# Ver logs
kubectl logs <pod-name>

# Ejecutar comando en pod
kubectl exec -it <pod-name> -- sh

# Aplicar configuración
kubectl apply -f deployment.yaml

# Eliminar recurso
kubectl delete pod <pod-name>
```

### Contextos y Namespaces

```bash
# Ver contextos (clusters configurados)
kubectl config get-contexts

# Cambiar contexto
kubectl config use-context <context-name>

# Ver namespace actual
kubectl config view --minify | grep namespace

# Cambiar namespace por defecto
kubectl config set-context --current --namespace=<namespace>

# Trabajar con namespace específico
kubectl get pods -n kube-system
```

---

## 📝 Primer Deployment

### Crear Deployment

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```bash
# Aplicar
kubectl apply -f nginx-deployment.yaml

# Verificar
kubectl get deployments
kubectl get pods
kubectl get rs  # ReplicaSets
```

### Exponer con Service

```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer  # NodePort para local
```

```bash
# Aplicar
kubectl apply -f nginx-service.yaml

# Ver servicio
kubectl get services

# Acceder (minikube)
minikube service nginx-service
```

### Escalar

```bash
# Escalar a 5 réplicas
kubectl scale deployment nginx-deployment --replicas=5

# Verificar
kubectl get pods
```

### Actualizar

```bash
# Actualizar imagen
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Ver rollout
kubectl rollout status deployment/nginx-deployment

# Ver historial
kubectl rollout history deployment/nginx-deployment

# Rollback
kubectl rollout undo deployment/nginx-deployment
```

---

## 🎓 Conceptos Clave para Recordar

| Concepto | Descripción |
|----------|-------------|
| **Pod** | Unidad mínima, uno o más contenedores |
| **Deployment** | Gestiona réplicas y actualizaciones |
| **Service** | Expone pods como servicio de red |
| **Namespace** | Aislamiento lógico de recursos |
| **Label** | Etiqueta clave-valor para organizar |
| **Selector** | Selecciona recursos por labels |

---

## 📚 Próximos Pasos

Ahora que entiendes los fundamentos:

1. [**Arquitectura**](./arquitectura.md) → Componentes detallados del cluster
2. [**Pods**](./pods.md) → Lifecycle, patterns, troubleshooting
3. [**Deployments**](./deployments.md) → Estrategias de despliegue
4. [**Services**](./services.md) → Exposición y networking

---

## 🔗 Recursos Adicionales

- [Documentación Oficial](https://kubernetes.io/docs/)
- [Kubernetes Tutorials](https://kubernetes.io/docs/tutorials/)
- [Katacoda Interactive Learning](https://www.katacoda.com/courses/kubernetes)
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

---

[⬅️ Volver a Contenedores](../README.md) | [➡️ Siguiente: Arquitectura](./arquitectura.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
