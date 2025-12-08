# Arquitectura de Kubernetes

## 📖 Visión General

Kubernetes sigue una arquitectura **maestro-trabajador** (master-worker) con un plano de control centralizado y múltiples nodos trabajadores.

```
┌─────────────────────────────────────────────────────────┐
│              KUBERNETES CLUSTER                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌────────────── CONTROL PLANE ─────────────────┐      │
│  │                                               │      │
│  │  ┌──────────────┐  ┌──────────────┐          │      │
│  │  │ API Server   │  │    etcd      │          │      │
│  │  │ (frontend)   │←→│ (data store) │          │      │
│  │  └──────┬───────┘  └──────────────┘          │      │
│  │         │                                     │      │
│  │  ┌──────▼───────┐  ┌──────────────┐          │      │
│  │  │  Scheduler   │  │ Controller   │          │      │
│  │  │              │  │  Manager     │          │      │
│  │  └──────────────┘  └──────────────┘          │      │
│  └───────────────────────┬─────────────────────┘      │
│                          │ API                         │
│        ┌─────────────────┼─────────────────┐           │
│        │                 │                 │           │
│  ┌─────▼─────┐    ┌──────▼────┐    ┌──────▼────┐      │
│  │  Node 1   │    │  Node 2   │    │  Node 3   │      │
│  ├───────────┤    ├───────────┤    ├───────────┤      │
│  │ kubelet   │    │ kubelet   │    │ kubelet   │      │
│  │ kube-proxy│    │ kube-proxy│    │ kube-proxy│      │
│  │ runtime   │    │ runtime   │    │ runtime   │      │
│  ├───────────┤    ├───────────┤    ├───────────┤      │
│  │   Pods    │    │   Pods    │    │   Pods    │      │
│  └───────────┘    └───────────┘    └───────────┘      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🎛️ Control Plane (Plano de Control)

El Control Plane gestiona el cluster y toma decisiones globales. Todos los componentes pueden ejecutarse en una o múltiples máquinas (HA).

### 1. API Server (kube-apiserver)

**El corazón de Kubernetes.** Frontend del control plane que expone la API REST.

**Responsabilidades:**
- Punto de entrada para todas las operaciones administrativas
- Valida y procesa requests REST
- Actualiza objetos en etcd
- Único componente que habla con etcd

**Comunicación:**
```
kubectl → API Server → etcd
kubelet → API Server
scheduler → API Server
controller → API Server
```

**Características:**
- Stateless (todo el estado en etcd)
- Horizontalmente escalable
- Autenticación y autorización
- Admission controllers

**Ejemplo de flujo:**
```bash
# 1. Usuario ejecuta
kubectl apply -f deployment.yaml

# 2. kubectl hace POST request a API Server
POST /apis/apps/v1/namespaces/default/deployments

# 3. API Server:
#    - Autentica usuario
#    - Autoriza operación
#    - Ejecuta admission controllers
#    - Valida objeto
#    - Escribe a etcd
#    - Retorna respuesta

# 4. Deployment Controller detecta cambio
#    - Crea ReplicaSet

# 5. ReplicaSet Controller detecta cambio
#    - Crea Pods

# 6. Scheduler detecta pods sin nodo
#    - Asigna nodos

# 7. kubelet detecta pods asignados
#    - Ejecuta contenedores
```

### 2. etcd

**Base de datos distribuida clave-valor** que almacena todo el estado del cluster.

**Características:**
- Consistencia fuerte (Raft consensus)
- Distribuido y replicado
- Watch API para cambios
- Solo accesible por API Server

**Qué almacena:**
```
/registry/
  /pods/default/nginx-pod
  /deployments/default/nginx-deployment
  /services/default/nginx-service
  /nodes/worker-1
  /configmaps/default/app-config
  /secrets/default/db-password
```

**Alta Disponibilidad:**
```
# Cluster etcd recomendado: 3, 5 o 7 nodos
┌─────────┐  ┌─────────┐  ┌─────────┐
│ etcd-1  │←→│ etcd-2  │←→│ etcd-3  │
│ (leader)│  │(follower)│ │(follower)│
└─────────┘  └─────────┘  └─────────┘
```

**⚠️ Crítico:**
- **SIEMPRE hacer backups de etcd**
- Perder etcd = perder todo el cluster

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Restore
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```

### 3. Scheduler (kube-scheduler)

**Asigna pods a nodos.** Observa pods recién creados sin nodo asignado y selecciona el mejor nodo.

**Proceso de Scheduling:**

1. **Filtering (Predicates):**
   - Elimina nodos que no cumplen requisitos
   - Ejemplos:
     - Recursos suficientes (CPU, memoria)
     - Port disponible
     - Taints/Tolerations
     - Node selectors
     - Affinity rules

2. **Scoring (Priorities):**
   - Califica nodos restantes
   - Balanceo de recursos
   - Distribución de pods
   - Afinidad/anti-afinidad

3. **Binding:**
   - Asigna pod al nodo con mayor score
   - Actualiza etcd vía API Server

**Ejemplo:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  nodeSelector:
    disktype: ssd  # Solo nodos con esta label
```

**Factores de scheduling:**
- Resource requests/limits
- Node affinity/anti-affinity
- Pod affinity/anti-affinity
- Taints y tolerations
- Topology spread constraints

### 4. Controller Manager (kube-controller-manager)

**Ejecuta controllers** que regulan el estado del cluster.

**Controllers principales:**

#### Node Controller
```
Responsabilidad: Monitorear nodos
- Detecta nuevos nodos
- Elimina nodos caídos
- Marca nodos como NotReady
```

#### Replication Controller
```
Responsabilidad: Mantener número correcto de pods
- Observa ReplicaSets/Deployments
- Crea/elimina pods según replicas deseadas
```

#### Endpoint Controller
```
Responsabilidad: Poblar objetos Endpoints
- Conecta Services con Pods
- Actualiza endpoints cuando pods cambian
```

#### Service Account Controller
```
Responsabilidad: Crear ServiceAccounts por defecto
- Crea SA para nuevos namespaces
- Gestiona tokens
```

**Control Loop Pattern:**
```go
for {
  desired := getDesiredState()
  current := getCurrentState()

  if current != desired {
    makeChanges()
  }

  wait()
}
```

**Ejemplo - Deployment Controller:**
```
Usuario: kubectl create deployment nginx --replicas=3

1. API Server escribe Deployment a etcd
2. Deployment Controller detecta cambio
3. Controller crea ReplicaSet
4. ReplicaSet Controller detecta cambio
5. Controller crea 3 Pods
6. Scheduler asigna Pods a Nodes
7. kubelet ejecuta contenedores
```

### 5. Cloud Controller Manager (CCM)

**Interactúa con proveedores cloud.** Separa lógica específica de cloud.

**Responsabilidades:**
- **Node Controller:** Verificar si nodos cloud fueron eliminados
- **Route Controller:** Configurar rutas en infraestructura cloud
- **Service Controller:** Crear/actualizar load balancers cloud
- **Volume Controller:** Crear/adjuntar/montar volúmenes cloud

**Ejemplo - LoadBalancer en AWS:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: myapp

# CCM automáticamente:
# 1. Crea ELB en AWS
# 2. Configura health checks
# 3. Asigna DNS público
# 4. Actualiza Service con IP externa
```

---

## 🖥️ Worker Nodes (Nodos Trabajadores)

Ejecutan las aplicaciones en contenedores.

### 1. kubelet

**Agente principal en cada nodo.** Asegura que contenedores estén corriendo.

**Responsabilidades:**
- Registra nodo en API Server
- Observa PodSpecs asignados al nodo
- Ejecuta contenedores vía runtime
- Reporta estado de pods
- Ejecuta probes (liveness, readiness)
- Monta volúmenes

**Proceso:**
```
1. kubelet consulta API Server
   "¿Hay pods asignados a mí?"

2. API Server responde con PodSpecs

3. kubelet llama a Container Runtime
   "Ejecuta estos contenedores"

4. Runtime (containerd) ejecuta contenedores

5. kubelet monitorea contenedores
   - Health checks
   - Resource usage
   - Termination

6. kubelet reporta estado a API Server
```

**Configuración:**
```yaml
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
authorization:
  mode: Webhook
clusterDomain: cluster.local
clusterDNS:
  - 10.96.0.10
```

### 2. kube-proxy

**Mantiene reglas de red** en nodos para permitir comunicación con pods.

**Modos de operación:**

#### iptables (por defecto)
```bash
# kube-proxy crea reglas iptables
iptables -t nat -A KUBE-SERVICES \
  -p tcp --dport 80 \
  -j KUBE-SVC-NGINX

# Balancea entre pods
iptables -t nat -A KUBE-SVC-NGINX \
  -m statistic --mode random --probability 0.33 \
  -j KUBE-SEP-POD1
```

#### IPVS (más performante)
```bash
# Mejor performance para muchos servicios
# Algoritmos: rr, lc, dh, sh, sed, nq
ipvsadm -A -t 10.96.0.1:80 -s rr
ipvsadm -a -t 10.96.0.1:80 -r 10.244.1.5:80
ipvsadm -a -t 10.96.0.1:80 -r 10.244.2.5:80
```

**Flujo de tráfico:**
```
Client → Service IP (10.96.0.1:80)
         ↓ (kube-proxy iptables rule)
       Pod IP (10.244.1.5:80)
```

### 3. Container Runtime

**Software que ejecuta contenedores.** Kubernetes usa CRI (Container Runtime Interface).

**Opciones:**

**containerd (Recomendado)**
```bash
# Configuración
cat > /etc/containerd/config.toml <<EOF
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
EOF
```

**CRI-O**
- Lightweight
- Diseñado específicamente para Kubernetes
- Compatible con OCI

**Docker (Deprecated desde 1.20)**
- Usa dockershim
- Reemplazado por containerd
- Eliminado en v1.24

---

## 🔄 Add-ons del Cluster

### 1. DNS (CoreDNS)

**Servicio DNS para el cluster.** Permite descubrimiento de servicios.

```yaml
# Pods pueden resolver:
curl http://nginx-service.default.svc.cluster.local

# Formato:
# <service-name>.<namespace>.svc.cluster.local
```

**Configuración CoreDNS:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

### 2. Ingress Controller

**Gestiona acceso HTTP/HTTPS externo.** No viene por defecto.

**Opciones populares:**
- NGINX Ingress Controller
- Traefik
- HAProxy
- Istio Gateway

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### 3. Metrics Server

**Recolecta métricas de recursos.** Necesario para HPA y `kubectl top`.

```bash
# Instalar
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Ver métricas
kubectl top nodes
kubectl top pods
```

### 4. Dashboard

**UI web para Kubernetes.**

```bash
# Instalar
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Acceder
kubectl proxy
# http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

---

## 🔐 Comunicación y Seguridad

### Comunicación entre Componentes

```
┌─────────────────────────────────────┐
│     Secure Communication            │
├─────────────────────────────────────┤
│                                     │
│  API Server ←─────────────────┐    │
│      ↕ TLS                     │    │
│   ┌──┴──┬──────┬─────┐         │    │
│   │     │      │     │         │    │
│  etcd  Sched Ctrl  CCM         │    │
│                                │    │
│  ┌─────────────────────────────┤    │
│  │ Worker Node                 │    │
│  │                             │    │
│  │  kubelet ←───────── API     │    │
│  │     ↕ TLS          Server   │    │
│  │  kube-proxy                 │    │
│  └─────────────────────────────┘    │
│                                     │
└─────────────────────────────────────┘
```

**Certificados TLS:**
- CA del cluster
- API Server certificate
- kubelet client certificate
- etcd peer certificates
- Service Account signing key

### Puertos Importantes

**Control Plane:**
```
6443  - API Server (HTTPS)
2379  - etcd client
2380  - etcd peer
10250 - kubelet API
10251 - kube-scheduler
10252 - kube-controller-manager
```

**Worker Nodes:**
```
10250 - kubelet API
10256 - kube-proxy health
30000-32767 - NodePort Services
```

---

## 🏗️ Topologías de Alta Disponibilidad

### 1. Stacked etcd

Control plane y etcd en los mismos nodos.

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Master 1      │  │   Master 2      │  │   Master 3      │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ API Server      │  │ API Server      │  │ API Server      │
│ Scheduler       │  │ Scheduler       │  │ Scheduler       │
│ Controller Mgr  │  │ Controller Mgr  │  │ Controller Mgr  │
│ etcd            │←→│ etcd            │←→│ etcd            │
└─────────────────┘  └─────────────────┘  └─────────────────┘
         ↓                    ↓                    ↓
    ┌────┴────────────────────┴────────────────────┴────┐
    │              Load Balancer (kube-apiserver)       │
    └───────────────────────────────────────────────────┘
```

**Ventajas:**
- ✅ Menos nodos
- ✅ Más simple
- ✅ Menor costo

**Desventajas:**
- ❌ Menos resiliente
- ❌ Fallo de nodo afecta etcd Y control plane

### 2. External etcd

etcd en cluster separado.

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Master 1      │  │   Master 2      │  │   Master 3      │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ API Server      │  │ API Server      │  │ API Server      │
│ Scheduler       │  │ Scheduler       │  │ Scheduler       │
│ Controller Mgr  │  │ Controller Mgr  │  │ Controller Mgr  │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                    │
         └────────────────────┼────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
         ┌────▼────┐     ┌────▼────┐     ┌───▼─────┐
         │ etcd-1  │←───→│ etcd-2  │←───→│ etcd-3  │
         └─────────┘     └─────────┘     └─────────┘
```

**Ventajas:**
- ✅ Más resiliente
- ✅ etcd aislado
- ✅ Mejor para producción crítica

**Desventajas:**
- ❌ Más nodos (mínimo 6)
- ❌ Más complejo
- ❌ Mayor costo

---

## 📊 Flujo Completo de Deployment

```
1. Usuario:
   kubectl apply -f deployment.yaml

2. kubectl:
   → POST /apis/apps/v1/deployments a API Server

3. API Server:
   → Autentica, autoriza, valida
   → Escribe a etcd
   → Notifica a watchers

4. Deployment Controller:
   → Observa nuevo Deployment
   → Crea ReplicaSet

5. API Server:
   → Escribe ReplicaSet a etcd

6. ReplicaSet Controller:
   → Observa nuevo ReplicaSet
   → Crea Pods (según replicas)

7. API Server:
   → Escribe Pods a etcd

8. Scheduler:
   → Observa Pods sin nodo
   → Evalúa nodos disponibles
   → Asigna cada Pod a un nodo
   → Actualiza Pod.spec.nodeName

9. API Server:
   → Actualiza Pods en etcd

10. kubelet (en nodo asignado):
    → Observa Pod asignado a su nodo
    → Descarga imagen
    → Llama a Container Runtime
    → Inicia contenedores

11. Container Runtime:
    → Ejecuta contenedores
    → Reporta estado a kubelet

12. kubelet:
    → Ejecuta probes
    → Reporta estado a API Server

13. API Server:
    → Actualiza status de Pod
    → Pod pasa a Running
```

---

## 📚 Próximos Pasos

Ahora que entiendes la arquitectura:

1. [**Pods**](./pods.md) → Lifecycle, patterns, configuración
2. [**Deployments**](./deployments.md) → Gestión de aplicaciones
3. [**Services**](./services.md) → Networking y exposición

---

## 🔗 Recursos Adicionales

- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Kubernetes Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [etcd Documentation](https://etcd.io/docs/)

---

[⬅️ Volver: Introducción](./introduccion.md) | [➡️ Siguiente: Pods](./pods.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
