# Pods en Kubernetes

## 📖 ¿Qué es un Pod?

Un **Pod** es la unidad más pequeña y básica de despliegue en Kubernetes. Representa uno o más contenedores que se ejecutan juntos en el mismo nodo y comparten recursos.

```
┌─────────────────────────────────────┐
│             POD                     │
├─────────────────────────────────────┤
│  Shared Network (IP: 10.244.1.5)   │
│  Shared Storage (Volumes)           │
│                                     │
│  ┌──────────────┐  ┌──────────────┐│
│  │ Container 1  │  │ Container 2  ││
│  │   (nginx)    │  │  (sidecar)   ││
│  └──────────────┘  └──────────────┘│
└─────────────────────────────────────┘
```

**Características clave:**
- Comparten dirección IP
- Comparten namespace de red (localhost)
- Pueden compartir volúmenes
- Co-located y co-scheduled
- Efímeros (reemplazables)

---

## 📝 Crear Pods

### Pod Simple

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

```bash
# Crear pod
kubectl apply -f nginx-pod.yaml

# Ver pods
kubectl get pods

# Ver detalles
kubectl describe pod nginx-pod

# Ver logs
kubectl logs nginx-pod

# Acceder al contenedor
kubectl exec -it nginx-pod -- /bin/bash
```

### Pod Multi-Contenedor

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  # Contenedor principal
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: shared-data
      mountPath: /data

  # Sidecar para logs
  - name: log-shipper
    image: fluentd:latest
    volumeMounts:
    - name: shared-data
      mountPath: /data
      readOnly: true

  volumes:
  - name: shared-data
    emptyDir: {}
```

---

## 🔄 Lifecycle del Pod

### Estados del Pod

```
Pending → Running → Succeeded/Failed
   ↓         ↓
Unknown   CrashLoopBackOff
```

**1. Pending**
- Pod creado pero contenedores no están corriendo
- Esperando scheduling
- Descargando imágenes

**2. Running**
- Pod asignado a nodo
- Al menos un contenedor está corriendo

**3. Succeeded**
- Todos los contenedores terminaron exitosamente
- No serán reiniciados

**4. Failed**
- Todos los contenedores terminaron
- Al menos uno falló

**5. Unknown**
- No se puede obtener estado del pod
- Usualmente problema de comunicación con nodo

### Fase de Creación

```bash
1. Usuario crea Pod
   kubectl apply -f pod.yaml

2. API Server valida y guarda en etcd
   Estado: Pending

3. Scheduler asigna a nodo
   Actualiza pod.spec.nodeName

4. kubelet en nodo detecta asignación
   - Descarga imagen
   - Crea contenedores
   Estado: ContainerCreating

5. Contenedores inician
   Estado: Running

6. Ready checks pasan
   Pod listo para recibir tráfico
```

### Restart Policies

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restart-demo
spec:
  restartPolicy: Always  # Always (default), OnFailure, Never

  containers:
  - name: app
    image: myapp:1.0
```

**Always (por defecto):**
- Reinicia contenedor siempre que termine
- Ideal para servicios long-running

**OnFailure:**
- Solo reinicia si falla (exit code != 0)
- Ideal para jobs/tasks

**Never:**
- No reinicia nunca
- Ideal para one-time tasks

---

## 💪 Patrones Multi-Contenedor

### 1. Sidecar Pattern

Contenedor auxiliar que extiende funcionalidad del principal.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  containers:
  # Aplicación principal
  - name: app
    image: my-web-app:1.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  # Sidecar: envía logs a servicio externo
  - name: log-agent
    image: fluentd:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
      readOnly: true

  volumes:
  - name: logs
    emptyDir: {}
```

**Casos de uso:**
- Log shipping
- Monitoring agents
- Service mesh proxies (Istio, Linkerd)

### 2. Ambassador Pattern

Proxy que representa la aplicación principal hacia el exterior.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-example
spec:
  containers:
  # Aplicación
  - name: app
    image: legacy-app:1.0
    # Conecta a localhost:6379 (cree que es Redis local)

  # Ambassador: proxy a Redis cluster real
  - name: redis-proxy
    image: redis-proxy:1.0
    ports:
    - containerPort: 6379
    env:
    - name: REDIS_CLUSTER
      value: "redis-cluster.default.svc.cluster.local"
```

**Casos de uso:**
- Simplificar conexiones a servicios externos
- Cacheo local
- Rate limiting

### 3. Adapter Pattern

Transforma salida del contenedor principal a formato estándar.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-example
spec:
  containers:
  # Aplicación con logs en formato propietario
  - name: app
    image: legacy-app:1.0
    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  # Adapter: convierte logs a formato JSON
  - name: log-adapter
    image: log-formatter:1.0
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
      readOnly: true
    - name: formatted-logs
      mountPath: /var/log/formatted

  volumes:
  - name: logs
    emptyDir: {}
  - name: formatted-logs
    emptyDir: {}
```

---

## ⚙️ Configuración de Pods

### Resources (Recursos)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:  # Mínimo garantizado
        memory: "64Mi"
        cpu: "250m"  # 0.25 CPU
      limits:  # Máximo permitido
        memory: "128Mi"
        cpu: "500m"  # 0.5 CPU
```

**Unidades:**
- **CPU:** milicores (m) → 1000m = 1 CPU
- **Memory:** Mi (Mebibytes), Gi (Gibibytes)

**QoS Classes:**
```
Guaranteed   → requests = limits
Burstable    → requests < limits
BestEffort   → sin requests ni limits
```

### Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # Variable simple
    - name: LOG_LEVEL
      value: "info"

    # Desde ConfigMap
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.host

    # Desde Secret
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password

    # Field reference
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name

    # Resource reference
    - name: CPU_REQUEST
      valueFrom:
        resourceFieldRef:
          containerName: app
          resource: requests.cpu
```

### Volumes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-demo
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    # emptyDir: temporal, se borra con el pod
    - name: cache
      mountPath: /cache

    # ConfigMap: archivos de configuración
    - name: config
      mountPath: /etc/config

    # Secret: archivos sensibles
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true

    # PersistentVolumeClaim: persistente
    - name: data
      mountPath: /data

  volumes:
  - name: cache
    emptyDir: {}

  - name: config
    configMap:
      name: app-config

  - name: secrets
    secret:
      secretName: app-secrets

  - name: data
    persistentVolumeClaim:
      claimName: app-pvc
```

---

## 🏥 Health Checks (Probes)

### Tipos de Probes

**1. Liveness Probe**
- ¿El contenedor está vivo?
- Si falla → Kubernetes reinicia contenedor

**2. Readiness Probe**
- ¿El contenedor está listo para recibir tráfico?
- Si falla → Se elimina de endpoints de Service

**3. Startup Probe**
- ¿El contenedor ha iniciado?
- Útil para apps con arranque lento
- Mientras falla, se deshabilitan liveness/readiness

### Métodos de Verificación

#### HTTP GET

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-probe
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080

    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 15
      periodSeconds: 10
      timeoutSeconds: 1
      successThreshold: 1
      failureThreshold: 3

    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

#### TCP Socket

```yaml
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```

#### Exec Command

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Configuración de Probes

```yaml
probe:
  initialDelaySeconds: 10  # Espera antes de primera verificación
  periodSeconds: 5         # Frecuencia de verificación
  timeoutSeconds: 1        # Timeout por verificación
  successThreshold: 1      # Éxitos consecutivos para marcar healthy
  failureThreshold: 3      # Fallos consecutivos para marcar unhealthy
```

### Ejemplo Completo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-example
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080

    # Startup: solo al inicio
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
      # Máximo 5 minutos para arrancar (30 * 10s)

    # Liveness: está funcionando?
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 0  # Deshabilitado hasta que startup pase
      periodSeconds: 10
      failureThreshold: 3

    # Readiness: listo para tráfico?
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 5
```

---

## 🔧 Init Containers

Contenedores que se ejecutan **antes** de los contenedores principales.

**Características:**
- Ejecutan secuencialmente
- Deben completarse exitosamente
- Útiles para setup/preparación

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  # Init containers
  initContainers:
  # 1. Esperar a que base de datos esté lista
  - name: wait-for-db
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup db-service; do echo waiting for db; sleep 2; done']

  # 2. Clonar repositorio
  - name: git-clone
    image: alpine/git
    command:
    - git
    - clone
    - https://github.com/user/repo.git
    - /data
    volumeMounts:
    - name: code
      mountPath: /data

  # 3. Configurar permisos
  - name: set-permissions
    image: busybox
    command: ['sh', '-c', 'chmod -R 755 /data']
    volumeMounts:
    - name: code
      mountPath: /data

  # Contenedor principal
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: code
      mountPath: /app/code

  volumes:
  - name: code
    emptyDir: {}
```

**Casos de uso:**
- Esperar servicios dependientes
- Configuración inicial
- Clonar código/configuración
- Migración de bases de datos
- Generar configuración dinámica

---

## 🐛 Troubleshooting de Pods

### Ver Estado

```bash
# Listar pods
kubectl get pods
kubectl get pods -o wide  # Más detalles

# Ver detalles completos
kubectl describe pod <pod-name>

# Ver eventos
kubectl get events --sort-by='.lastTimestamp'

# Ver logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container
kubectl logs <pod-name> --previous  # Logs del contenedor anterior
kubectl logs <pod-name> -f  # Follow (tail -f)
kubectl logs <pod-name> --tail=100  # Últimas 100 líneas
```

### Problemas Comunes

#### ImagePullBackOff

```
Error: ErrImagePull / ImagePullBackOff

Causas:
- Imagen no existe
- Typo en nombre de imagen
- Sin credenciales para registry privado
- Tag no existe

Solución:
kubectl describe pod <name>  # Ver error exacto
kubectl get pod <name> -o yaml | grep image  # Ver imagen configurada
```

#### CrashLoopBackOff

```
Error: CrashLoopBackOff

Causas:
- Aplicación falla al iniciar
- Comando incorrecto
- Falta configuración/secretos
- Health check falla

Solución:
kubectl logs <pod-name>  # Ver por qué falla
kubectl logs <pod-name> --previous  # Ver intento anterior
kubectl describe pod <pod-name>  # Ver eventos
```

#### Pending

```
Estado: Pending (por mucho tiempo)

Causas:
- Recursos insuficientes en cluster
- PersistentVolume no disponible
- Node selectors/affinity no coinciden
- Taints/tolerations

Solución:
kubectl describe pod <name>  # Ver eventos
kubectl get nodes  # Ver capacidad
kubectl top nodes  # Ver uso actual
```

#### Error

```
Estado: Error / Failed

Causas:
- Exit code != 0
- OOMKilled (sin memoria)
- Error en comando

Solución:
kubectl logs <pod-name>
kubectl get pod <name> -o yaml  # Ver terminationReason
```

### Debugging Interactivo

```bash
# Ejecutar shell en pod
kubectl exec -it <pod-name> -- /bin/sh

# Ejecutar comando
kubectl exec <pod-name> -- ls /app

# Port forward para acceder localmente
kubectl port-forward <pod-name> 8080:80
# Acceder en http://localhost:8080

# Copiar archivos
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# Pod temporal para debugging
kubectl run debug --image=busybox -it --rm --restart=Never -- sh
```

---

## 📚 Próximos Pasos

Con Pods dominados:

1. [**Deployments**](./deployments.md) → Gestión declarativa de Pods
2. [**Services**](./services.md) → Exposición y networking
3. [**ConfigMaps y Secrets**](./configmaps-secrets.md) → Gestión de configuración

---

## 🔗 Recursos Adicionales

- [Pod Overview](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

---

[⬅️ Volver: Arquitectura](./arquitectura.md) | [➡️ Siguiente: Deployments](./deployments.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
