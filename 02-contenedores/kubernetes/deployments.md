# Deployments en Kubernetes

## 📖 ¿Qué es un Deployment?

Un **Deployment** proporciona actualizaciones declarativas para Pods y ReplicaSets. Describes el estado deseado y el Deployment Controller cambia el estado actual al estado deseado.

```
Deployment
    ↓ (gestiona)
ReplicaSet
    ↓ (crea)
  Pods
```

**¿Por qué Deployments y no Pods directos?**
- ✅ Actualiz

aciones sin downtime (rolling updates)
- ✅ Rollback automático
- ✅ Scaling declarativo
- ✅ Self-healing (recreación de pods)
- ✅ Historial de versiones

---

## 📝 Crear Deployment

### Deployment Básico

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3  # Número deseado de pods
  selector:
    matchLabels:
      app: nginx  # Debe coincidir con template labels
  template:  # Template para crear pods
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
# Crear
kubectl apply -f nginx-deployment.yaml

# Ver deployments
kubectl get deployments

# Salida:
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           1m

# Ver pods creados
kubectl get pods -l app=nginx

# Ver ReplicaSets
kubectl get rs

# Detalles completos
kubectl describe deployment nginx-deployment
```

### Deployment con Recursos y Probes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080

        # Recursos
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"

        # Environment
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database.host

        # Health checks
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

## 🔄 Estrategias de Actualización

### 1. Rolling Update (Por Defecto)

Actualiza pods gradualmente, uno a la vez.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-deployment
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Máximo de pods extra durante update
      maxUnavailable: 1  # Máximo de pods no disponibles

  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:2.0.0
```

**Proceso:**
```
Estado Inicial: 10 pods v1.0

1. Crea 2 nuevos pods v2.0 (maxSurge=2)
   Pods: 10 v1.0 + 2 v2.0 = 12 total

2. Espera que 2 v2.0 estén Ready

3. Elimina 1 pod v1.0 (maxUnavailable=1)
   Pods: 9 v1.0 + 2 v2.0 = 11 total

4. Repite hasta reemplazar todos

Estado Final: 10 pods v2.0
```

**Actualizar imagen:**
```bash
# Método 1: kubectl set image
kubectl set image deployment/myapp-deployment \
  myapp=myapp:2.0.0

# Método 2: kubectl edit
kubectl edit deployment myapp-deployment

# Método 3: actualizar YAML y aplicar
kubectl apply -f deployment.yaml

# Ver progreso
kubectl rollout status deployment/myapp-deployment

# Pausar rollout
kubectl rollout pause deployment/myapp-deployment

# Reanudar
kubectl rollout resume deployment/myapp-deployment
```

### 2. Recreate

Elimina todos los pods antiguos antes de crear nuevos. **Causa downtime.**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-deployment
spec:
  replicas: 5
  strategy:
    type: Recreate  # Downtime durante update

  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:2.0.0
```

**Proceso:**
```
1. Elimina todos los 5 pods v1.0
2. Espera que terminen
3. Crea 5 nuevos pods v2.0
4. Espera que estén Ready

⚠️ Durante paso 2-3: 0 pods disponibles
```

**Cuándo usar:**
- Incompatibilidad entre versiones
- No se puede correr v1 y v2 simultáneamente
- Recursos limitados

---

## ⏮️ Rollback

### Historial de Rollouts

```bash
# Ver historial
kubectl rollout history deployment/myapp-deployment

# Salida:
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=deployment.yaml
2         kubectl set image deployment/myapp myapp=myapp:2.0
3         kubectl set image deployment/myapp myapp=myapp:3.0

# Ver detalles de revisión específica
kubectl rollout history deployment/myapp-deployment --revision=2
```

### Hacer Rollback

```bash
# Rollback a revisión anterior
kubectl rollout undo deployment/myapp-deployment

# Rollback a revisión específica
kubectl rollout undo deployment/myapp-deployment --to-revision=2

# Ver estado del rollback
kubectl rollout status deployment/myapp-deployment
```

### Anotaciones para Historial

```bash
# Agregar causa de cambio
kubectl apply -f deployment.yaml --record

# O con annotation
kubectl annotate deployment/myapp-deployment \
  kubernetes.io/change-cause="Actualizar a v3.0 con fix de seguridad"
```

---

## 📈 Scaling

### Manual Scaling

```bash
# Escalar a 5 réplicas
kubectl scale deployment myapp-deployment --replicas=5

# Escalar con condición
kubectl scale deployment myapp-deployment --replicas=10 \
  --current-replicas=5

# Ver estado
kubectl get deployment myapp-deployment
```

### Actualizar YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 10  # Cambiar número
  # ... resto de spec
```

```bash
kubectl apply -f deployment.yaml
```

### Auto-Scaling (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # 70% CPU
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # 80% memory
```

```bash
# Crear HPA con kubectl
kubectl autoscale deployment myapp-deployment \
  --cpu-percent=70 \
  --min=2 \
  --max=10

# Ver HPA
kubectl get hpa

# Salida:
NAME        REFERENCE                  TARGETS   MINPODS   MAXPODS   REPLICAS
myapp-hpa   Deployment/myapp-deployment  45%/70%   2         10        3
```

**Requisitos para HPA:**
- Metrics Server instalado
- Resource requests definidos en pods

---

## 🎯 Estrategias Avanzadas de Despliegue

### Blue-Green Deployment

Dos ambientes idénticos: Blue (actual) y Green (nuevo).

```yaml
# Blue deployment (actual)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
---
# Green deployment (nuevo)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:2.0.0
---
# Service apunta a blue
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Cambiar a green para switch
  ports:
  - port: 80
```

**Proceso:**
```
1. Blue está en producción
2. Desplegar Green
3. Probar Green
4. Cambiar Service selector a green
5. Eliminar Blue cuando todo esté OK
```

### Canary Deployment

Liberar gradualmente a porcentaje de usuarios.

```yaml
# Stable deployment (90% tráfico)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp
        track: stable
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
---
# Canary deployment (10% tráfico)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: myapp
        image: myapp:2.0.0
---
# Service balancea entre ambos
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp  # Ambos labels
  ports:
  - port: 80
```

---

## 🔧 Configuración Avanzada

### Revision History Limit

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  revisionHistoryLimit: 5  # Mantener últimas 5 revisiones
  # ...
```

### Progress Deadline

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  progressDeadlineSeconds: 600  # 10 minutos timeout
  # ...
```

### Min Ready Seconds

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  minReadySeconds: 30  # Espera 30s antes de considerar pod Ready
  # ...
```

---

## 🐛 Troubleshooting

### Deployment No Progresa

```bash
# Ver estado
kubectl get deployment myapp-deployment

# Si AVAILABLE < READY
kubectl describe deployment myapp-deployment

# Ver eventos
kubectl get events --sort-by='.lastTimestamp'

# Ver pods
kubectl get pods -l app=myapp

# Causas comunes:
# - Imagen no existe
# - Recursos insuficientes
# - Readiness probe falla
# - ImagePullBackOff
```

### Rollout Stuck

```bash
# Ver status
kubectl rollout status deployment/myapp-deployment

# Causas:
# - progressDeadlineSeconds excedido
# - Pods no pasan readiness probe
# - Recursos insuficientes

# Solución:
kubectl rollout undo deployment/myapp-deployment
```

### Pods Reiniciándose Constantemente

```bash
# Ver logs
kubectl logs deployment/myapp-deployment

# Ver por qué reinician
kubectl describe pod <pod-name>

# Causas:
# - Liveness probe falla
# - OOMKilled (sin memoria)
# - Aplicación crashea
```

---

## 📚 Próximos Pasos

Con Deployments dominados:

1. [**Services**](./services.md) → Exponer aplicaciones
2. [**ConfigMaps y Secrets**](./configmaps-secrets.md) → Gestión de configuración
3. [**Networking**](./networking.md) → Comunicación avanzada

---

## 🔗 Recursos Adicionales

- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Performing a Rolling Update](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)

---

[⬅️ Volver: Pods](./pods.md) | [➡️ Siguiente: Services](./services.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
