# Kubernetes Best Practices

## 📖 Introducción

Esta guía cubre las mejores prácticas para ejecutar Kubernetes en producción, desde seguridad hasta optimización de recursos.

---

## 🔒 Seguridad

### 1. RBAC (Role-Based Access Control)

**Principio:** Mínimo privilegio - dar solo los permisos necesarios.

#### Roles vs ClusterRoles

```yaml
# Role: Permisos en un namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding: Asignar Role a usuario/grupo
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```yaml
# ClusterRole: Permisos a nivel cluster
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-global
subjects:
- kind: Group
  name: sre-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

#### ServiceAccounts

```yaml
# ServiceAccount para pods
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: production
---
# Deployment usando ServiceAccount
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      serviceAccountName: app-sa  # No usar default
      containers:
      - name: app
        image: myapp:1.0
```

**Mejores prácticas:**
- ❌ No usar `default` ServiceAccount
- ❌ No dar permisos `*` (wildcard)
- ✅ Crear ServiceAccount específico por aplicación
- ✅ Usar namespaces para separar ambientes

### 2. Pod Security

#### Security Context

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    # A nivel de pod
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  
  containers:
  - name: app
    image: myapp:1.0
    
    securityContext:
      # A nivel de contenedor
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      runAsUser: 1000
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE  # Solo si necesita puertos < 1024
    
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /app/cache
  
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

#### Pod Security Standards (PSS)

```yaml
# A nivel de namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**Niveles:**
- **Privileged**: Sin restricciones (no recomendado en prod)
- **Baseline**: Mínimas restricciones
- **Restricted**: Muy restrictivo (recomendado)

### 3. Network Policies

```yaml
# Default Deny All
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
---
# Allow solo tráfico necesario
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### 4. Secrets Management

```yaml
# ❌ MAL: Secret en Git
apiVersion: v1
kind: Secret
data:
  password: cGFzc3dvcmQxMjM=  # Esto está en Git!

# ✅ BIEN: Usar herramientas externas
```

**Herramientas recomendadas:**
- **Sealed Secrets**: Encriptación para Git
- **External Secrets Operator**: Integración con vaults externos
- **Vault**: HashiCorp Vault
- **Cloud Native**: AWS Secrets Manager, GCP Secret Manager, Azure Key Vault

#### External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: db-credentials
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: prod/database/password
```

### 5. Image Security

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        # ✅ Usar tags específicos, NO :latest
        image: myregistry.com/myapp:v1.2.3
        
        # ✅ Verificar imagen con digest
        # image: myregistry.com/myapp@sha256:abc123...
        
        imagePullPolicy: Always  # O IfNotPresent con tags específicos
      
      # ✅ Usar imagePullSecrets para registries privados
      imagePullSecrets:
      - name: regcred
```

**Scanning de imágenes:**
```bash
# Trivy
trivy image myapp:1.0

# Grype
grype myapp:1.0

# Anchore
anchore-cli image add myapp:1.0
anchore-cli image vuln myapp:1.0 all
```

---

## 🎯 Gestión de Recursos

### 1. Requests y Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      # SIEMPRE definir requests
      requests:
        memory: "256Mi"
        cpu: "250m"      # 0.25 CPU cores
      
      # Limits opcionales pero recomendados
      limits:
        memory: "512Mi"
        cpu: "500m"
```

**Impacto de requests y limits:**

| Configuración | QoS Class | Eviction Priority | Uso |
|---------------|-----------|-------------------|-----|
| Sin requests/limits | BestEffort | Primero en ser eliminado | ❌ No usar |
| Solo requests | Burstable | Medio | ⚠️ Desarrollo |
| Requests = Limits | Guaranteed | Último en ser eliminado | ✅ Producción |

### 2. LimitRanges

```yaml
# Valores por defecto y límites en namespace
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: production
spec:
  limits:
  # Por contenedor
  - type: Container
    default:
      memory: 512Mi
      cpu: 500m
    defaultRequest:
      memory: 256Mi
      cpu: 250m
    max:
      memory: 2Gi
      cpu: 2000m
    min:
      memory: 64Mi
      cpu: 100m
  
  # Por Pod
  - type: Pod
    max:
      memory: 4Gi
      cpu: 4000m
  
  # Por PVC
  - type: PersistentVolumeClaim
    max:
      storage: 100Gi
    min:
      storage: 1Gi
```

### 3. ResourceQuotas

```yaml
# Límites totales en namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
  namespace: production
spec:
  hard:
    # Límites de compute
    requests.cpu: "100"
    requests.memory: 100Gi
    limits.cpu: "200"
    limits.memory: 200Gi
    
    # Límites de objetos
    pods: "100"
    services: "50"
    persistentvolumeclaims: "20"
    secrets: "100"
    configmaps: "100"
    
    # Storage
    requests.storage: 500Gi
```

### 4. HorizontalPodAutoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  
  minReplicas: 3      # Mínimo para HA
  maxReplicas: 20
  
  metrics:
  # CPU
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  
  # Memory
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  
  # Custom metrics (con Prometheus Adapter)
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Esperar 5min antes de bajar
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60  # Máximo 50% de pods por minuto
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30  # Duplicar pods cada 30s si es necesario
```

### 5. VerticalPodAutoscaler (VPA)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  
  updatePolicy:
    updateMode: "Auto"  # Auto, Initial, Off
  
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2000m
        memory: 2Gi
      controlledResources:
      - cpu
      - memory
```

---

## 🔄 Alta Disponibilidad

### 1. Pod Disruption Budgets (PDB)

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
  namespace: production
spec:
  minAvailable: 2  # Mínimo 2 pods siempre disponibles
  # O usar maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

### 2. Anti-Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      # Distribuir pods en diferentes nodos
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: myapp
            topologyKey: kubernetes.io/hostname
      
      # O en diferentes zonas
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: myapp
              topologyKey: topology.kubernetes.io/zone
```

### 3. TopologySpreadConstraints

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 6
  template:
    spec:
      topologySpreadConstraints:
      # Distribuir equitativamente en zonas
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: myapp
      
      # Distribuir equitativamente en nodos
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: myapp
```

### 4. Health Checks Robustos

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    
    # Startup probe: Para apps con inicio lento
    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30  # 30 intentos
      periodSeconds: 10     # Cada 10s = 5 minutos máximo
    
    # Liveness probe: Detectar deadlocks
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3   # Reiniciar después de 3 fallos
      successThreshold: 1
    
    # Readiness probe: Dejar de enviar tráfico
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 2
      successThreshold: 1
```

---

## 📊 Observabilidad

### 1. Logging

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    
    # Logs a stdout/stderr (no a archivos)
    env:
    - name: LOG_OUTPUT
      value: "stdout"
    - name: LOG_FORMAT
      value: "json"  # Structured logging
```

**Stack recomendado:**
- **EFK**: Elasticsearch + Fluentd + Kibana
- **PLG**: Promtail + Loki + Grafana (más ligero)
- **Cloud native**: CloudWatch, Stackdriver, Azure Monitor

### 2. Metrics

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  labels:
    app: myapp
  annotations:
    # Prometheus scraping
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
spec:
  ports:
  - name: metrics
    port: 8080
  selector:
    app: myapp
```

**Métricas esenciales:**
- RED: Rate, Errors, Duration
- USE: Utilization, Saturation, Errors
- Golden Signals: Latency, Traffic, Errors, Saturation

### 3. Tracing

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        env:
        # OpenTelemetry
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://otel-collector:4317"
        - name: OTEL_SERVICE_NAME
          value: "myapp"
```

**Herramientas:**
- Jaeger
- Zipkin
- OpenTelemetry
- Cloud: X-Ray, Cloud Trace, Application Insights

### 4. Labels y Annotations

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  
  # Labels: Para selección y organización
  labels:
    app: myapp
    tier: backend
    version: v1.2.3
    environment: production
    team: backend-team
  
  # Annotations: Metadata adicional
  annotations:
    description: "Main API backend"
    owner: "backend-team@company.com"
    runbook: "https://wiki.company.com/myapp-runbook"
    version: "1.2.3"
    build-date: "2024-01-15"
    git-commit: "abc123"
spec:
  template:
    metadata:
      labels:
        app: myapp
        tier: backend
        version: v1.2.3
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
```

**Labels recomendados:**
```yaml
labels:
  app.kubernetes.io/name: myapp
  app.kubernetes.io/instance: myapp-prod
  app.kubernetes.io/version: "1.2.3"
  app.kubernetes.io/component: backend
  app.kubernetes.io/part-of: ecommerce-platform
  app.kubernetes.io/managed-by: helm
```

---

## 🚀 Despliegues

### 1. Rolling Update Configurado

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 10
  
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2         # Máximo 2 pods extra durante update
      maxUnavailable: 0   # Nunca bajar de 10 pods
  
  minReadySeconds: 10     # Esperar 10s antes de considerar ready
  
  progressDeadlineSeconds: 600  # Fallar si no completa en 10min
  
  revisionHistoryLimit: 5       # Mantener 5 revisiones para rollback
```

### 2. Pre-stop Hook

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    
    lifecycle:
      # Antes de terminar pod
      preStop:
        exec:
          command:
          - /bin/sh
          - -c
          - |
            # Esperar que termine el tráfico en curso
            sleep 15
            # Graceful shutdown
            kill -SIGTERM 1
  
  terminationGracePeriodSeconds: 60  # Tiempo máximo para terminar
```

### 3. Probes durante Rollout

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  # IMPORTANTE: readinessProbe corto para rollouts rápidos
  periodSeconds: 2
  failureThreshold: 3
```

---

## 💾 Persistencia

### 1. StorageClass Optimizada

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"  # Siempre encriptar
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer  # Esperar a que pod se programe
reclaimPolicy: Retain  # No eliminar datos automáticamente
```

### 2. Backups Automáticos

```yaml
# Velero para backups
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: "0 2 * * *"  # 2 AM diario
  template:
    includedNamespaces:
    - production
    - staging
    storageLocation: aws-s3
    ttl: 720h  # 30 días
```

---

## 🌐 Networking

### 1. Service con Session Affinity

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  sessionAffinity: ClientIP  # Sticky sessions
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600  # 1 hora
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: myapp
```

### 2. Ingress Optimizado

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
  annotations:
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "100"
    
    # Connection limits
    nginx.ingress.kubernetes.io/limit-connections: "10"
    
    # Timeouts
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    
    # Body size
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    
    # SSL
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              number: 80
```

---

## 🏁 Checklist de Producción

### Seguridad

- [ ] RBAC configurado (no usar default ServiceAccount)
- [ ] Pod Security Standards aplicados
- [ ] Network Policies (default deny)
- [ ] Secrets gestionados externamente (Vault, External Secrets)
- [ ] Imágenes escaneadas (Trivy, Grype)
- [ ] Security context en todos los pods
- [ ] TLS en Ingress
- [ ] Encriptación at-rest habilitada

### Recursos

- [ ] Requests definidos en todos los contenedores
- [ ] Limits definidos
- [ ] LimitRanges configurados
- [ ] ResourceQuotas en namespaces
- [ ] HPA configurado
- [ ] PodDisruptionBudgets para apps críticas

### Alta Disponibilidad

- [ ] Mínimo 3 réplicas por aplicación
- [ ] Anti-affinity configurado
- [ ] TopologySpreadConstraints
- [ ] Multiple availability zones
- [ ] Liveness y readiness probes
- [ ] PodDisruptionBudget configurado

### Observabilidad

- [ ] Logs en formato estructurado (JSON)
- [ ] Métricas expuestas (/metrics)
- [ ] Prometheus scraping habilitado
- [ ] Distributed tracing
- [ ] Dashboards en Grafana
- [ ] Alertas configuradas
- [ ] Labels estandarizados

### Despliegues

- [ ] Rolling updates configurado
- [ ] PreStop hooks
- [ ] Graceful shutdown
- [ ] Rollback plan
- [ ] Blue-green o canary para apps críticas
- [ ] CI/CD pipeline
- [ ] Automated testing

### Networking

- [ ] Ingress con TLS
- [ ] Rate limiting
- [ ] Network Policies
- [ ] Service Mesh (para microservicios complejos)
- [ ] DNS configurado correctamente

### Persistencia

- [ ] StorageClass apropiado
- [ ] Backups automáticos (Velero)
- [ ] Disaster recovery plan
- [ ] PV con encryption
- [ ] Retention policies

### Documentación

- [ ] Runbooks actualizados
- [ ] Architecture diagrams
- [ ] Disaster recovery procedures
- [ ] Oncall procedures
- [ ] Dependencies documentadas

---

## 🚫 Anti-patterns a Evitar

### 1. Usar :latest

```yaml
# ❌ MAL
image: nginx:latest

# ✅ BIEN
image: nginx:1.21.6
# O mejor aún:
image: nginx:1.21.6@sha256:abc123...
```

### 2. Sin Health Checks

```yaml
# ❌ MAL: Pod sin probes

# ✅ BIEN: Con liveness y readiness
livenessProbe:
  httpGet:
    path: /health
    port: 8080
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

### 3. Secrets en Git

```yaml
# ❌ MAL: Secret en Git
apiVersion: v1
kind: Secret
data:
  password: cGFzc3dvcmQ=

# ✅ BIEN: External Secrets
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
```

### 4. Sin Resource Limits

```yaml
# ❌ MAL: Sin requests/limits

# ✅ BIEN
resources:
  requests:
    memory: 256Mi
    cpu: 250m
  limits:
    memory: 512Mi
    cpu: 500m
```

### 5. Una Sola Réplica

```yaml
# ❌ MAL: Sin HA
replicas: 1

# ✅ BIEN: Alta disponibilidad
replicas: 3
```

### 6. Privileged Containers

```yaml
# ❌ MAL
securityContext:
  privileged: true

# ✅ BIEN
securityContext:
  runAsNonRoot: true
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
```

---

## 📚 Recursos Adicionales

- [Kubernetes Production Best Practices](https://learnk8s.io/production-best-practices)
- [12 Factor App](https://12factor.net/)
- [CNCF Cloud Native Trail Map](https://github.com/cncf/trailmap)
- [Kubernetes Failure Stories](https://k8s.af/)

---

[⬅️ Volver: Helm](./helm.md) | [🏠 Volver a Contenedores](../README.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
