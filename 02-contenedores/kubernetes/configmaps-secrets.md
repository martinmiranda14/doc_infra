# ConfigMaps y Secrets en Kubernetes

## 📖 Gestión de Configuración

Kubernetes separa la configuración del código mediante **ConfigMaps** (configuración general) y **Secrets** (datos sensibles).

**Principio:** Aplicaciones deben ser configurables sin rebuilds.

```
Imagen del contenedor (inmutable)
    +
ConfigMaps / Secrets (configuración)
    =
Aplicación en ejecución
```

---

## 🗂️ ConfigMaps

### ¿Qué es un ConfigMap?

Almacena datos de configuración en pares clave-valor. **No encriptado**, para datos no sensibles.

### Crear ConfigMaps

#### Método 1: Desde Literales

```bash
kubectl create configmap app-config \
  --from-literal=database.host=postgres.default.svc.cluster.local \
  --from-literal=database.port=5432 \
  --from-literal=log.level=info
```

#### Método 2: Desde Archivo

**app.properties:**
```properties
database.host=postgres.default.svc.cluster.local
database.port=5432
log.level=info
max.connections=100
```

```bash
kubectl create configmap app-config \
  --from-file=app.properties
```

#### Método 3: Desde Directorio

```bash
# Crea un ConfigMap con todos los archivos del directorio
kubectl create configmap app-config \
  --from-file=./config/
```

#### Método 4: YAML Declarativo

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  # Pares clave-valor simples
  database.host: "postgres.default.svc.cluster.local"
  database.port: "5432"
  log.level: "info"

  # Archivo completo como valor
  app.properties: |
    database.host=postgres.default.svc.cluster.local
    database.port=5432
    log.level=info
    max.connections=100

  # JSON/YAML como valor
  config.json: |
    {
      "database": {
        "host": "postgres",
        "port": 5432
      },
      "features": {
        "caching": true,
        "analytics": false
      }
    }
```

```bash
kubectl apply -f configmap.yaml
```

### Ver ConfigMaps

```bash
# Listar
kubectl get configmaps
kubectl get cm  # Short name

# Ver contenido
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

---

## 🔒 Secrets

### ¿Qué es un Secret?

Almacena datos sensibles (contraseñas, tokens, keys). **Base64 encoded**, más seguro que ConfigMaps.

⚠️ **Nota:** Base64 NO es encriptación. En clusters de producción, usar encriptación at-rest.

### Tipos de Secrets

**1. Opaque (Genérico)**
```bash
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=supersecret123
```

**2. docker-registry (Pull de imágenes privadas)**
```bash
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com
```

**3. tls (Certificados TLS)**
```bash
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

### Crear Secrets

#### Desde Literales

```bash
kubectl create secret generic api-keys \
  --from-literal=api-key=abc123xyz \
  --from-literal=api-secret=secret456
```

#### Desde Archivos

```bash
# Archivo con contraseña
echo -n 'admin' > ./username.txt
echo -n 'supersecret' > ./password.txt

kubectl create secret generic db-auth \
  --from-file=./username.txt \
  --from-file=./password.txt
```

#### YAML Declarativo

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  # Valores en base64
  username: YWRtaW4=        # admin
  password: c3VwZXJzZWNyZXQ= # supersecret
---
# O usando stringData (automáticamente convierte a base64)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials-v2
type: Opaque
stringData:
  username: admin
  password: supersecret
```

```bash
# Encodear en base64
echo -n 'admin' | base64
# YWRtaW4=

# Decodear
echo 'YWRtaW4=' | base64 --decode
# admin
```

---

## 📦 Usar ConfigMaps y Secrets en Pods

### 1. Como Variables de Entorno

#### ConfigMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # Variable individual
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.host

    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log.level

    # Todas las keys del ConfigMap
    envFrom:
    - configMapRef:
        name: app-config
```

#### Secret

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # Variable individual
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username

    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password

    # Todas las keys del Secret
    envFrom:
    - secretRef:
        name: db-credentials
```

### 2. Como Archivos (Volumes)

#### ConfigMap como Volumen

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
      readOnly: true

  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: default.conf
```

**ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80;
        server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
```

#### Secret como Volumen

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true

  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
      defaultMode: 0400  # Permisos read-only para owner
```

Archivos creados:
```
/etc/secrets/username  (contiene: admin)
/etc/secrets/password  (contiene: supersecret)
```

---

## 🔄 Ejemplo Completo: Aplicación con DB

```yaml
# 1. ConfigMap - Configuración general
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  DATABASE_HOST: "postgres-service"
  DATABASE_PORT: "5432"
  DATABASE_NAME: "myapp"
  LOG_LEVEL: "info"
  CACHE_ENABLED: "true"

  app-config.yaml: |
    server:
      port: 8080
      timeout: 30s
    features:
      registration: true
      payments: false
---
# 2. Secret - Credenciales
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secret
type: Opaque
stringData:
  DATABASE_USERNAME: "appuser"
  DATABASE_PASSWORD: "P@ssw0rd123"
  JWT_SECRET: "my-super-secret-jwt-key"
  API_KEY: "abc123xyz456"
---
# 3. Deployment usando ConfigMap y Secret
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
        image: webapp:2.0
        ports:
        - containerPort: 8080

        # Variables de entorno desde ConfigMap
        envFrom:
        - configMapRef:
            name: webapp-config
        - secretRef:
            name: webapp-secret

        # También variables individuales
        env:
        - name: DB_CONNECTION_STRING
          value: "postgres://$(DATABASE_USERNAME):$(DATABASE_PASSWORD)@$(DATABASE_HOST):$(DATABASE_PORT)/$(DATABASE_NAME)"

        # Montar archivo de configuración
        volumeMounts:
        - name: config-file
          mountPath: /app/config
          readOnly: true

        # Recursos
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"

      volumes:
      - name: config-file
        configMap:
          name: webapp-config
          items:
          - key: app-config.yaml
            path: config.yaml
```

---

## 🔐 Mejores Prácticas de Seguridad

### 1. Secrets: No Commitear a Git

```bash
# ❌ MAL: Secret en repositorio
apiVersion: v1
kind: Secret
data:
  password: cGFzc3dvcmQxMjM=  # Esto está en Git!

# ✅ BIEN: Usar herramientas externas
# - Sealed Secrets
# - External Secrets Operator
# - Vault
```

### 2. RBAC para Secrets

```yaml
# Role que solo puede leer ConfigMaps, NO Secrets
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: configmap-reader
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
# No incluye "secrets"
```

### 3. Encryption at Rest

**Habilitar encriptación de Secrets en etcd:**

```yaml
# /etc/kubernetes/enc/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aesabc:
        keys:
        - name: key1
          secret: <base64-encoded-32-byte-key>
    - identity: {}
```

```bash
# Agregar a kube-apiserver
--encryption-provider-config=/etc/kubernetes/enc/encryption-config.yaml
```

### 4. Usar Herramientas Especializadas

#### Sealed Secrets

```bash
# Instalar controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Instalar CLI
brew install kubeseal

# Crear SealedSecret
kubectl create secret generic mysecret --from-literal=password=secret123 --dry-run=client -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml

# Seguro para commitear
git add sealed-secret.yaml
```

#### External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: db-credentials
  data:
  - secretKey: password
    remoteRef:
      key: prod/db/password
```

---

## 🔄 Actualizar ConfigMaps y Secrets

### Actualización

```bash
# Editar ConfigMap
kubectl edit configmap app-config

# O recrear
kubectl delete configmap app-config
kubectl create configmap app-config --from-file=new-config.yaml

# Aplicar cambios
kubectl apply -f configmap.yaml
```

### Hot Reload

**⚠️ Importante:**
- Variables de entorno: **NO** se actualizan automáticamente (requiere restart)
- Volúmenes: **SÍ** se actualizan automáticamente (~60s delay)

```yaml
# Variables de entorno: Requiere restart
env:
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: log.level

# Después de cambiar ConfigMap:
kubectl rollout restart deployment/myapp
```

```yaml
# Volúmenes: Auto-actualiza (app debe reload)
volumeMounts:
- name: config
  mountPath: /etc/config

volumes:
- name: config
  configMap:
    name: app-config

# ConfigMap cambia → archivo actualiza en ~1min
# App debe watch el archivo y reload
```

### Estrategia: Immutable ConfigMaps/Secrets

```yaml
# ConfigMap inmutable (K8s 1.21+)
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2  # Versionar nombre
immutable: true
data:
  key: value

# Al cambiar config:
# 1. Crear nuevo ConfigMap (v3)
# 2. Actualizar Deployment para usar v3
# 3. Rollout deployment
# 4. Eliminar ConfigMap antiguo (v2)
```

---

## 📋 Ejemplo Avanzado: Multi-Ambiente

```yaml
# ConfigMap base
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-base
data:
  APP_NAME: "MyApp"
  LOG_FORMAT: "json"
---
# ConfigMap desarrollo
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-dev
data:
  ENV: "development"
  DATABASE_HOST: "postgres-dev"
  LOG_LEVEL: "debug"
  CACHE_ENABLED: "false"
---
# ConfigMap producción
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-prod
data:
  ENV: "production"
  DATABASE_HOST: "postgres-prod.xyz.rds.amazonaws.com"
  LOG_LEVEL: "warn"
  CACHE_ENABLED: "true"
---
# Deployment desarrollo
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-dev
spec:
  template:
    spec:
      containers:
      - name: app
        envFrom:
        - configMapRef:
            name: app-config-base
        - configMapRef:
            name: app-config-dev  # Sobrescribe valores
        - secretRef:
            name: app-secrets-dev
---
# Deployment producción
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-prod
spec:
  template:
    spec:
      containers:
      - name: app
        envFrom:
        - configMapRef:
            name: app-config-base
        - configMapRef:
            name: app-config-prod  # Sobrescribe valores
        - secretRef:
            name: app-secrets-prod
```

---

## 🐛 Troubleshooting

### ConfigMap/Secret No Existe

```bash
# Error en pod
kubectl describe pod myapp-xyz
# Events: Error: configmap "app-config" not found

# Verificar existe
kubectl get configmap app-config
kubectl get secret db-credentials

# Ver en qué namespace está
kubectl get configmap --all-namespaces | grep app-config
```

### Key No Existe

```bash
# Error: key "database.host" not found in ConfigMap

# Ver keys disponibles
kubectl get configmap app-config -o jsonpath='{.data}'

# Verificar nombre exacto (case-sensitive)
```

### Permisos de Volumen

```bash
# Error: Permission denied al leer /etc/secrets/password

# Verificar permisos del volumen
volumes:
- name: secret-volume
  secret:
    secretName: db-credentials
    defaultMode: 0400  # Solo owner read

# O en el mount
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
  readOnly: true
```

### Secret No Decodifica

```bash
# Ver Secret encoded
kubectl get secret db-credentials -o yaml

# Decodear manualmente
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 --decode
```

---

## 📚 Próximos Pasos

Con ConfigMaps y Secrets dominados:

1. [**Storage**](./storage.md) → Persistencia de datos con PV/PVC
2. [**Networking**](./networking.md) → Políticas y comunicación avanzada
3. [**Helm**](./helm.md) → Gestión de aplicaciones con charts

---

## 🔗 Recursos Adicionales

- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- [External Secrets Operator](https://external-secrets.io/)

---

[⬅️ Volver: Services](./services.md) | [➡️ Siguiente: Storage](./storage.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
