# Networking en Kubernetes

## 📖 Modelo de Red de Kubernetes

Kubernetes implementa un modelo de red plano donde:
- **Todos los Pods pueden comunicarse entre sí** sin NAT
- **Todos los Nodos pueden comunicarse con todos los Pods** sin NAT
- **La IP que un Pod ve de sí mismo** es la misma que ven otros

```
Requisitos fundamentales:
1. Pod-to-Pod communication (mismo nodo o diferentes)
2. Pod-to-Service communication
3. External-to-Service communication
```

---

## 🔌 CNI (Container Network Interface)

### ¿Qué es CNI?

Especificación y plugins para configurar redes de contenedores. Kubernetes delega la red a plugins CNI.

### Plugins CNI Populares

#### Calico

**Características:**
- Routing BGP (Layer 3)
- Network Policies avanzadas
- Alto rendimiento
- Soporte para eBPF

```bash
# Instalar Calico
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Verificar
kubectl get pods -n kube-system -l k8s-app=calico-node
```

#### Flannel

**Características:**
- Simple y ligero
- VXLAN overlay network
- Ideal para comenzar

```bash
# Instalar Flannel
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

#### Cilium

**Características:**
- Basado en eBPF
- Observabilidad avanzada
- Security features
- Service Mesh integrado

```bash
# Instalar con Helm
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --namespace kube-system
```

#### Weave Net

**Características:**
- Encryption automático
- Multicast support
- Simple setup

### Comparación de CNI Plugins

| Plugin | Tipo | Performance | Network Policy | Complejidad |
|--------|------|-------------|----------------|-------------|
| **Calico** | Layer 3 (BGP) | ⭐⭐⭐⭐⭐ | ✅ Avanzadas | Media |
| **Flannel** | Overlay (VXLAN) | ⭐⭐⭐ | ❌ No | Baja |
| **Cilium** | eBPF | ⭐⭐⭐⭐⭐ | ✅ Avanzadas | Alta |
| **Weave** | Overlay | ⭐⭐⭐ | ✅ Básicas | Baja |

---

## 🛡️ Network Policies

### ¿Qué son?

Especificaciones de cómo grupos de Pods pueden comunicarse entre sí y con otros endpoints.

**Por defecto:** Todos los Pods pueden comunicarse libremente (no hay políticas).

### Componentes de una Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-policy
  namespace: default
spec:
  podSelector:        # A qué pods aplica
    matchLabels:
      app: myapp
  
  policyTypes:        # Tipos de políticas
  - Ingress           # Tráfico entrante
  - Egress            # Tráfico saliente
  
  ingress:            # Reglas de entrada
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 8080
  
  egress:             # Reglas de salida
  - to:
    - podSelector:
        matchLabels:
          role: database
    ports:
    - protocol: TCP
      port: 5432
```

### Ejemplo 1: Deny All (Default Deny)

```yaml
# Bloquear TODO el tráfico a pods con app=backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  # Sin reglas ingress/egress = denegar todo
```

### Ejemplo 2: Allow Specific Ingress

```yaml
# Solo permitir tráfico desde frontend a backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Ejemplo 3: Allow from Namespace

```yaml
# Permitir tráfico desde namespace "monitoring"
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring
  namespace: production
spec:
  podSelector: {}  # Todos los pods en namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 9090  # Prometheus
```

### Ejemplo 4: Allow External Traffic

```yaml
# Permitir tráfico desde IPs específicas
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 203.0.113.0/24  # Rango de IPs permitido
        except:
        - 203.0.113.50/32     # Excepto esta IP
    ports:
    - protocol: TCP
      port: 80
```

### Ejemplo 5: Database Policy (Completa)

```yaml
# Política completa para base de datos
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: postgres-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  - Egress
  
  # Ingress: Solo desde backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 5432
  
  # Egress: Solo DNS y dentro del cluster
  egress:
  # DNS queries
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
  
  # Comunicación entre réplicas de postgres
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
```

---

## 🌐 DNS en Kubernetes

### CoreDNS

Kubernetes usa CoreDNS para descubrimiento de servicios.

```bash
# Ver pods de CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Ver ConfigMap
kubectl get configmap coredns -n kube-system -o yaml
```

### Resolución DNS

**Formato:**
```
<service-name>.<namespace>.svc.cluster.local
```

**Ejemplos:**
```bash
# Desde pod en namespace "default"
curl http://backend                           # Mismo namespace
curl http://backend.default                   # Especificar namespace
curl http://backend.default.svc.cluster.local # FQDN completo

# Desde pod en namespace "production"
curl http://api.production.svc.cluster.local
```

### DNS para Pods

```
<pod-ip-with-dashes>.<namespace>.pod.cluster.local

Ejemplo:
10.244.1.5 → 10-244-1-5.default.pod.cluster.local
```

### Custom DNS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-dns
spec:
  containers:
  - name: app
    image: nginx
  
  # DNS personalizado
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
    - 8.8.8.8
    - 1.1.1.1
    searches:
    - custom.local
    options:
    - name: ndots
      value: "2"
```

**DNS Policies:**
- `Default`: Hereda del nodo
- `ClusterFirst`: CoreDNS primero (default)
- `ClusterFirstWithHostNet`: Para pods con hostNetwork
- `None`: Configuración manual

---

## 🕸️ Service Mesh

### ¿Qué es Service Mesh?

Capa de infraestructura para gestionar comunicación service-to-service con:
- Observabilidad
- Seguridad (mTLS)
- Traffic management
- Resilience (retries, circuit breakers)

```
Sin Service Mesh:
Service A → Service B

Con Service Mesh:
Service A → Sidecar Proxy → Sidecar Proxy → Service B
            (Envoy)                  (Envoy)
```

### Istio

#### Arquitectura

```
Control Plane (istiod):
- Pilot: Traffic management
- Citadel: Certificate management
- Galley: Configuration

Data Plane:
- Envoy sidecars en cada pod
```

#### Instalación

```bash
# Descargar Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.0
export PATH=$PWD/bin:$PATH

# Instalar
istioctl install --set profile=demo -y

# Habilitar sidecar injection automático
kubectl label namespace default istio-injection=enabled

# Verificar
kubectl get pods -n istio-system
```

#### Virtual Service (Traffic Routing)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  # 90% tráfico a v1
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  # 10% tráfico a v2 (canary)
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
```

#### Destination Rule

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100
```

#### mTLS (Mutual TLS)

```yaml
# Habilitar mTLS estricto
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # STRICT, PERMISSIVE, DISABLE
```

### Linkerd

**Ventajas:**
- Más ligero que Istio
- Más simple de configurar
- Menos overhead

```bash
# Instalar CLI
curl -sL https://run.linkerd.io/install | sh

# Instalar en cluster
linkerd install | kubectl apply -f -

# Verificar
linkerd check

# Inject sidecar en deployment
kubectl get deploy webapp -o yaml | linkerd inject - | kubectl apply -f -
```

### Comparación Service Mesh

| Feature | Istio | Linkerd | Consul Connect |
|---------|-------|---------|----------------|
| **Complejidad** | Alta | Baja | Media |
| **Performance** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Observabilidad** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **mTLS** | ✅ | ✅ | ✅ |
| **Traffic Split** | ✅ | ✅ | ✅ |
| **Multi-cluster** | ✅ | ✅ | ✅ |

---

## 🔄 Ejemplo Completo: Microservicios con Network Policies

```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce
---
# Frontend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: ecommerce
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - name: frontend
        image: frontend:1.0
        ports:
        - containerPort: 80
---
# Backend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: ecommerce
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      tier: api
  template:
    metadata:
      labels:
        app: backend
        tier: api
    spec:
      containers:
      - name: backend
        image: backend:1.0
        ports:
        - containerPort: 8080
---
# Database StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: ecommerce
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
      tier: database
  template:
    metadata:
      labels:
        app: postgres
        tier: database
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
---
# Network Policy: Frontend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: ecommerce
spec:
  podSelector:
    matchLabels:
      tier: web
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Permitir desde Internet (LoadBalancer/Ingress)
  - from: []
    ports:
    - protocol: TCP
      port: 80
  egress:
  # Solo puede llamar a backend
  - to:
    - podSelector:
        matchLabels:
          tier: api
    ports:
    - protocol: TCP
      port: 8080
  # Permitir DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
---
# Network Policy: Backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: ecommerce
spec:
  podSelector:
    matchLabels:
      tier: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Solo desde frontend
  - from:
    - podSelector:
        matchLabels:
          tier: web
    ports:
    - protocol: TCP
      port: 8080
  egress:
  # Solo a database
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
  # DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
---
# Network Policy: Database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: ecommerce
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Solo desde backend
  - from:
    - podSelector:
        matchLabels:
          tier: api
    ports:
    - protocol: TCP
      port: 5432
  egress:
  # DNS solamente
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

---

## 🐛 Troubleshooting de Red

### Pod No Puede Alcanzar Otro Pod

```bash
# 1. Verificar IPs de pods
kubectl get pods -o wide

# 2. Test de conectividad básica
kubectl exec -it pod-a -- ping <IP-pod-b>

# 3. Verificar Network Policies
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>

# 4. Ver qué políticas afectan un pod
kubectl get networkpolicies --all-namespaces -o wide

# 5. Test desde pod temporal
kubectl run test --image=busybox --rm -it --restart=Never -- sh
wget -O- http://service-name:port

# 6. Verificar DNS
kubectl exec -it pod-a -- nslookup service-name
kubectl exec -it pod-a -- cat /etc/resolv.conf
```

### Service No Resuelve

```bash
# 1. Verificar Service y Endpoints
kubectl get service my-service
kubectl get endpoints my-service

# 2. Test DNS
kubectl run dnstest --image=busybox:1.28 --rm -it --restart=Never -- nslookup my-service

# 3. Ver logs de CoreDNS
kubectl logs -n kube-system -l k8s-app=kube-dns

# 4. Verificar CoreDNS está corriendo
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

### Network Policy No Funciona

```bash
# 1. Verificar CNI plugin soporta Network Policies
# Flannel NO soporta Network Policies
# Calico, Cilium, Weave SÍ soportan

# 2. Ver si hay políticas conflictivas
kubectl get networkpolicies -A

# 3. Describir política
kubectl describe networkpolicy my-policy

# 4. Ver logs del CNI plugin
# Para Calico:
kubectl logs -n kube-system -l k8s-app=calico-node
```

### Debugging Avanzado

```bash
# tcpdump en pod
kubectl run tcpdump --image=nicolaka/netshoot --rm -it --restart=Never -- bash
tcpdump -i eth0 -n

# Verificar iptables (en nodo)
sudo iptables -L -n -v -t nat | grep <service-name>

# Ver reglas de kube-proxy
kubectl logs -n kube-system -l k8s-app=kube-proxy

# Netshoot (debugging container)
kubectl run netshoot --image=nicolaka/netshoot --rm -it --restart=Never -- bash
# Herramientas disponibles: tcpdump, curl, dig, traceroute, etc.
```

---

## 📊 Mejores Prácticas

### 1. Default Deny

```yaml
# Crear política de denegar todo por defecto
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

Luego agregar políticas específicas para permitir tráfico necesario.

### 2. Segmentación por Namespace

```yaml
# Bloquear tráfico entre namespaces por defecto
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-namespace
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}  # Solo desde mismo namespace
```

### 3. Permitir DNS Siempre

```yaml
egress:
- to:
  - namespaceSelector:
      matchLabels:
        name: kube-system
    podSelector:
      matchLabels:
        k8s-app: kube-dns
  ports:
  - protocol: UDP
    port: 53
```

### 4. Separar Políticas por Tier

```yaml
# Frontend, Backend, Database en diferentes políticas
# Usar labels: tier=frontend, tier=backend, tier=database
```

### 5. Monitoreo de Red

```bash
# Con Istio/Linkerd: dashboards automáticos
istioctl dashboard kiali

# Metrics de red
kubectl top pods
kubectl top nodes

# Prometheus queries para latencia, throughput
```

---

## 🔗 Recursos Adicionales

- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [CNI Specification](https://github.com/containernetworking/cni)
- [Calico Documentation](https://docs.projectcalico.org/)
- [Istio Documentation](https://istio.io/latest/docs/)
- [Linkerd Documentation](https://linkerd.io/2/overview/)

---

[⬅️ Volver: Storage](./storage.md) | [➡️ Siguiente: Helm](./helm.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
