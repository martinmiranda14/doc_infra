# Services en Kubernetes

## 📖 ¿Qué es un Service?

Un **Service** es una abstracción que define un conjunto lógico de Pods y una política para acceder a ellos. Proporciona una IP estable y DNS name para acceder a pods efímeros.

**Problema sin Services:**
```
Pods tienen IPs dinámicas:
- Pod 1: 10.244.1.5  (muere)
- Pod 2: 10.244.2.8  (muere)
- Pod 3: 10.244.1.12 (nuevo)

¿Cómo saber a qué IP conectarse?
```

**Solución con Service:**
```
Service: my-service.default.svc.cluster.local
IP estable: 10.96.0.10

Service → balancea → Pods (IPs cambiantes)
```

---

## 🌐 Tipos de Services

### 1. ClusterIP (Por Defecto)

Expone el Service en una IP interna del cluster. **Solo accesible desde dentro del cluster.**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP  # Por defecto, puede omitirse
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80        # Puerto del Service
    targetPort: 8080  # Puerto del Pod
```

```bash
# Crear
kubectl apply -f backend-service.yaml

# Ver
kubectl get service backend-service

# Salida:
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
backend-service   ClusterIP   10.96.100.50   <none>        80/TCP    1m

# Acceder desde pod en cluster
curl http://backend-service.default.svc.cluster.local
# o simplemente
curl http://backend-service
```

**Uso típico:**
- Comunicación interna entre microservicios
- Bases de datos
- Servicios que no necesitan acceso externo

### 2. NodePort

Expone el Service en un puerto en **cada nodo del cluster**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80          # Puerto interno del Service
    targetPort: 8080  # Puerto del Pod
    nodePort: 30080   # Puerto en cada nodo (30000-32767)
```

```bash
# Acceder
# <NodeIP>:<NodePort>
curl http://192.168.1.10:30080
curl http://192.168.1.11:30080  # Cualquier nodo funciona

# En minikube
minikube service web-service --url
```

**Flujo:**
```
Internet → Node IP:30080 → Service:80 → Pod:8080
```

**Uso típico:**
- Desarrollo/testing
- Acceso directo sin load balancer
- Clusters on-premise sin LB externo

### 3. LoadBalancer

Provisiona un load balancer externo (en cloud providers).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

```bash
kubectl get service frontend-service

# Salida (en AWS/GCP/Azure):
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
frontend-service   LoadBalancer   10.96.50.100   52.14.23.145     80:31234/TCP   2m

# Acceder
curl http://52.14.23.145
```

**En cloud:**
- AWS: Crea ELB/ALB
- GCP: Crea Google Cloud Load Balancer
- Azure: Crea Azure Load Balancer

**En local (minikube):**
```bash
# minikube tunnel en otra terminal
minikube tunnel

# Ahora el LoadBalancer tendrá EXTERNAL-IP
```

**Uso típico:**
- Aplicaciones web públicas
- APIs públicas
- Producción en cloud

### 4. ExternalName

Mapea un Service a un DNS name externo.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: my-database.external.com
  ports:
  - port: 3306
```

```bash
# Desde pods, acceder como:
mysql -h external-db.default.svc.cluster.local
# Redirige a: my-database.external.com
```

**Uso típico:**
- Bases de datos externas al cluster
- APIs de terceros
- Servicios en migración a K8s

---

## 🎯 Selectors y Endpoints

### Service con Selector

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp  # Selecciona pods con este label
  ports:
  - port: 80
    targetPort: 8080
```

Kubernetes automáticamente crea **Endpoints** con IPs de pods que coinciden:

```bash
kubectl get endpoints my-service

# Salida:
NAME         ENDPOINTS                               AGE
my-service   10.244.1.5:8080,10.244.2.8:8080,...     5m
```

### Service sin Selector (Manual)

Para apuntar a recursos externos o específicos:

```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
---
# Endpoints manual
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service  # Mismo nombre que Service
subsets:
- addresses:
  - ip: 192.168.1.100
  - ip: 192.168.1.101
  ports:
  - port: 80
```

---

## 🔌 Descubrimiento de Servicios

### DNS

CoreDNS automáticamente crea registros DNS para Services:

```
Formato:
<service-name>.<namespace>.svc.cluster.local

Ejemplos:
backend.default.svc.cluster.local
database.prod.svc.cluster.local
```

```bash
# Desde pod, puedes usar:
curl http://backend                                    # Mismo namespace
curl http://backend.default                             # Especificar namespace
curl http://backend.default.svc.cluster.local          # FQDN completo
```

### Variables de Entorno

Kubernetes inyecta variables de entorno en pods:

```bash
# Para Service "backend" en namespace "default":
BACKEND_SERVICE_HOST=10.96.100.50
BACKEND_SERVICE_PORT=80
```

⚠️ **Limitación:** Solo para Services creados ANTES del Pod.

---

## 🚪 Exposición Avanzada con Ingress

### ¿Qué es Ingress?

Gestiona acceso HTTP/HTTPS externo a Services. Un solo punto de entrada para múltiples services.

```
Internet
    ↓
Ingress (+ TLS)
    ↓
┌─────┼─────┐
│     │     │
Service A  Service B
```

### Ingress Controller

Primero necesitas un Ingress Controller (no viene por defecto):

```bash
# NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Verificar
kubectl get pods -n ingress-nginx
```

### Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  # Host-based routing
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80

  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 8080
```

### Path-based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
spec:
  rules:
  - http:
      paths:
      # /web → web-service
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80

      # /api → api-service
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

### TLS/SSL

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
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

---

## 🔄 Ejemplo Completo

```yaml
# 1. Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:1.21
        ports:
        - containerPort: 80
---
# 2. ClusterIP Service (interno)
apiVersion: v1
kind: Service
metadata:
  name: webapp-internal
spec:
  type: ClusterIP
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
---
# 3. LoadBalancer Service (externo)
apiVersion: v1
kind: Service
metadata:
  name: webapp-external
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
---
# 4. Ingress (HTTP routing)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-internal
            port:
              number: 80
```

---

## 🐛 Troubleshooting

### Service No Responde

```bash
# 1. Verificar Service existe
kubectl get service my-service

# 2. Ver endpoints
kubectl get endpoints my-service
# Si vacío: selector no coincide con pods

# 3. Verificar pods
kubectl get pods -l app=myapp
# Deben existir pods con label correcto

# 4. Verificar pods están Ready
kubectl get pods -l app=myapp
# READY debe ser N/N

# 5. Test desde pod temporal
kubectl run test --image=busybox --rm -it --restart=Never -- sh
wget -O- my-service:80

# 6. Ver logs de pods
kubectl logs -l app=myapp
```

### LoadBalancer Stuck en Pending

```bash
kubectl get service my-service

# EXTERNAL-IP = <pending>

# Causas:
# - No estás en cloud provider (AWS/GCP/Azure)
# - Cloud controller no configurado
# - Cuota excedida en cloud

# Solución local:
# Usar NodePort o Ingress
# O minikube tunnel
```

### Ingress No Funciona

```bash
# 1. Verificar Ingress Controller instalado
kubectl get pods -n ingress-nginx

# 2. Ver Ingress
kubectl describe ingress my-ingress

# 3. Ver logs del controller
kubectl logs -n ingress-nginx <ingress-controller-pod>

# 4. Verificar DNS/hosts
# /etc/hosts en desarrollo:
# 127.0.0.1 myapp.local

# 5. Test con curl
curl -H "Host: myapp.example.com" http://<ingress-ip>
```

---

## 📊 Comparación de Tipos

| Tipo | Acceso | IP Externa | Uso | Costo Cloud |
|------|--------|------------|-----|-------------|
| **ClusterIP** | Solo interno | No | Servicios internos | Gratis |
| **NodePort** | Nodos del cluster | No | Dev/test | Gratis |
| **LoadBalancer** | Internet | Sí | Producción | $$ por LB |
| **Ingress** | Internet (HTTP/S) | Sí | Múltiples apps | $ (un LB) |

**Recomendación Producción:**
```
Internet
    ↓
Ingress + LoadBalancer (1 único LB)
    ↓
┌───────┼───────┐
│       │       │
Service A  Service B  (ClusterIP internos)
```

---

## 📚 Próximos Pasos

Con Services dominados:

1. [**ConfigMaps y Secrets**](./configmaps-secrets.md) → Gestión de configuración
2. [**Storage**](./storage.md) → Persistencia de datos
3. [**Networking**](./networking.md) → Políticas y mesh avanzado

---

## 🔗 Recursos Adicionales

- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [DNS for Services](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

---

[⬅️ Volver: Deployments](./deployments.md) | [🏠 Volver a Contenedores](../README.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
