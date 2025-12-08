# Helm - Package Manager para Kubernetes

## 📖 ¿Qué es Helm?

**Helm** es el gestor de paquetes para Kubernetes, similar a apt/yum en Linux o npm en Node.js.

**Problema sin Helm:**
```yaml
# Desplegar WordPress requiere:
- deployment.yaml (50 líneas)
- service.yaml (20 líneas)
- pvc.yaml (15 líneas)
- configmap.yaml (30 líneas)
- secret.yaml (10 líneas)
- ingress.yaml (25 líneas)

Total: 6 archivos, 150+ líneas, difícil de mantener
```

**Solución con Helm:**
```bash
helm install my-wordpress bitnami/wordpress
# Una línea, todo configurado
```

---

## 🎯 Conceptos Principales

### Chart

Paquete de archivos que describe recursos de Kubernetes.

```
wordpress/
├── Chart.yaml          # Metadatos del chart
├── values.yaml         # Configuración por defecto
├── charts/             # Charts dependientes
├── templates/          # Templates de K8s
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl   # Funciones helper
└── README.md
```

### Release

Instancia de un Chart desplegada en el cluster.

```bash
# Un chart puede tener múltiples releases
helm install wordpress-dev bitnami/wordpress
helm install wordpress-prod bitnami/wordpress
# Mismo chart, dos releases diferentes
```

### Repository

Lugar donde se almacenan y comparten Charts.

```bash
# Repositorios populares
https://charts.bitnami.com/bitnami
https://charts.helm.sh/stable
https://prometheus-community.github.io/helm-charts
```

---

## 🚀 Instalación de Helm

### Instalar Helm CLI

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows (Chocolatey)
choco install kubernetes-helm

# Verificar
helm version
# version.BuildInfo{Version:"v3.13.0", ...}
```

### Configurar Repositorios

```bash
# Agregar repo oficial de Bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami

# Agregar otros repos populares
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus https://prometheus-community.github.io/helm-charts
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Actualizar índice de repos
helm repo update

# Listar repos
helm repo list
```

---

## 📦 Usar Charts Existentes

### Buscar Charts

```bash
# Buscar en repos agregados
helm search repo wordpress

# Buscar en Artifact Hub (todos los repos públicos)
helm search hub wordpress

# Ver info de un chart
helm show chart bitnami/wordpress
helm show values bitnami/wordpress  # Ver values.yaml
helm show readme bitnami/wordpress  # Ver README
```

### Instalar un Chart

```bash
# Instalación básica
helm install my-release bitnami/wordpress

# Con custom values
helm install my-wordpress bitnami/wordpress \
  --set wordpressUsername=admin \
  --set wordpressPassword=password123 \
  --set service.type=LoadBalancer

# Con archivo de values
helm install my-wordpress bitnami/wordpress -f custom-values.yaml

# En namespace específico
helm install my-wordpress bitnami/wordpress \
  --namespace production \
  --create-namespace

# Dry-run (ver qué se va a crear)
helm install my-wordpress bitnami/wordpress --dry-run --debug
```

### Listar Releases

```bash
# Ver releases en namespace actual
helm list

# Ver en todos los namespaces
helm list --all-namespaces

# Ver releases no exitosos
helm list --failed
helm list --pending
```

### Actualizar un Release

```bash
# Update con nuevos valores
helm upgrade my-wordpress bitnami/wordpress \
  --set wordpressPassword=newpassword

# Upgrade con archivo
helm upgrade my-wordpress bitnami/wordpress -f values.yaml

# Install o upgrade (si no existe, lo crea)
helm upgrade --install my-wordpress bitnami/wordpress

# Forzar recreación de recursos
helm upgrade my-wordpress bitnami/wordpress --force

# Esperar a que esté ready
helm upgrade my-wordpress bitnami/wordpress --wait --timeout 10m
```

### Rollback

```bash
# Ver historial
helm history my-wordpress

# Rollback a versión anterior
helm rollback my-wordpress

# Rollback a versión específica
helm rollback my-wordpress 3

# Ver diferencia entre revisiones
helm diff revision my-wordpress 1 2  # Requiere plugin helm-diff
```

### Desinstalar

```bash
# Desinstalar release
helm uninstall my-wordpress

# Mantener historial
helm uninstall my-wordpress --keep-history

# Ver releases eliminados (si se usó --keep-history)
helm list --uninstalled
```

---

## 🛠️ Crear tu Propio Chart

### Crear Chart Básico

```bash
# Generar estructura
helm create myapp

cd myapp
tree
# myapp/
# ├── Chart.yaml
# ├── charts/
# ├── templates/
# │   ├── NOTES.txt
# │   ├── _helpers.tpl
# │   ├── deployment.yaml
# │   ├── hpa.yaml
# │   ├── ingress.yaml
# │   ├── service.yaml
# │   ├── serviceaccount.yaml
# │   └── tests/
# └── values.yaml
```

### Chart.yaml

```yaml
apiVersion: v2
name: myapp
description: Mi aplicación Node.js
type: application
version: 0.1.0        # Versión del chart
appVersion: "1.0.0"   # Versión de la app

keywords:
- nodejs
- backend
- api

maintainers:
- name: Tu Nombre
  email: tu@email.com

dependencies:
- name: postgresql
  version: "12.x.x"
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
```

### values.yaml

```yaml
# Configuración por defecto
replicaCount: 3

image:
  repository: myregistry/myapp
  pullPolicy: IfNotPresent
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
  - host: myapp.example.com
    paths:
    - path: /
      pathType: Prefix
  tls:
  - secretName: myapp-tls
    hosts:
    - myapp.example.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

env:
  DATABASE_HOST: postgres
  DATABASE_PORT: "5432"
  LOG_LEVEL: info

secrets:
  DATABASE_PASSWORD: ""  # Override en producción

postgresql:
  enabled: true
  auth:
    username: myapp
    password: changeme
    database: myapp_db
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        
        env:
        {{- range $key, $value := .Values.env }}
        - name: {{ $key }}
          value: {{ $value | quote }}
        {{- end }}
        
        # Secrets
        {{- if .Values.secrets.DATABASE_PASSWORD }}
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ include "myapp.fullname" . }}
              key: database-password
        {{- end }}
        
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
        
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 10
          periodSeconds: 5
        
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

### templates/_helpers.tpl

Funciones reutilizables:

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "myapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "myapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "myapp.labels" -}}
helm.sh/chart: {{ include "myapp.chart" . }}
{{ include "myapp.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

---

## 🎨 Templating Avanzado

### Valores Condicionales

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  # ...
{{- end }}
```

### Loops

```yaml
# Iterar sobre lista
{{- range .Values.extraEnvVars }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}

# Iterar sobre map
{{- range $key, $value := .Values.configMap }}
{{ $key }}: {{ $value | quote }}
{{- end }}
```

### Funciones Útiles

```yaml
# Default value
{{ .Values.replicaCount | default 1 }}

# Convertir a YAML con indentación
{{- toYaml .Values.resources | nindent 10 }}

# Quote string
value: {{ .Values.password | quote }}

# Truncar y limpiar
{{ .Release.Name | trunc 63 | trimSuffix "-" }}

# Uppercase/lowercase
{{ .Values.env | upper }}
{{ .Values.env | lower }}

# Contains
{{- if contains "prod" .Release.Name }}
# ...
{{- end }}
```

### Named Templates

```yaml
# Definir template
{{- define "myapp.database.url" -}}
postgres://{{ .Values.db.user }}:{{ .Values.db.password }}@{{ .Values.db.host }}/{{ .Values.db.name }}
{{- end }}

# Usar template
value: {{ include "myapp.database.url" . }}
```

---

## 📝 Archivo de Values por Ambiente

### values-dev.yaml

```yaml
replicaCount: 1

image:
  tag: "latest"

ingress:
  hosts:
  - host: myapp-dev.local

resources:
  limits:
    cpu: 200m
    memory: 256Mi

postgresql:
  enabled: true
  auth:
    password: dev-password
```

### values-prod.yaml

```yaml
replicaCount: 5

image:
  tag: "1.2.3"

ingress:
  hosts:
  - host: myapp.example.com
  tls:
  - secretName: myapp-prod-tls
    hosts:
    - myapp.example.com

resources:
  limits:
    cpu: 1000m
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20

postgresql:
  enabled: false  # Usar RDS externo

externalDatabase:
  host: prod-db.xyz.rds.amazonaws.com
  port: 5432
```

### Uso

```bash
# Desarrollo
helm install myapp-dev ./myapp -f myapp/values-dev.yaml

# Producción
helm install myapp-prod ./myapp -f myapp/values-prod.yaml
```

---

## 🔄 Ejemplo Completo: Chart de Aplicación

```bash
# Estructura final
myapp/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-prod.yaml
├── charts/
├── templates/
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── secret.yaml
│   ├── service.yaml
│   └── serviceaccount.yaml
└── README.md
```

### templates/configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
data:
  {{- range $key, $value := .Values.env }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
  
  {{- if .Values.postgresql.enabled }}
  DATABASE_HOST: {{ include "myapp.fullname" . }}-postgresql
  {{- else }}
  DATABASE_HOST: {{ .Values.externalDatabase.host }}
  {{- end }}
```

### templates/secret.yaml

```yaml
{{- if .Values.secrets.DATABASE_PASSWORD }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
type: Opaque
data:
  database-password: {{ .Values.secrets.DATABASE_PASSWORD | b64enc | quote }}
{{- end }}
```

### templates/NOTES.txt

```
Thank you for installing {{ .Chart.Name }}!

Your release is named {{ .Release.Name }}.

To access your application:

{{- if .Values.ingress.enabled }}
  http{{ if .Values.ingress.tls }}s{{ end }}://{{ (index .Values.ingress.hosts 0).host }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "myapp.fullname" . }})
  export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "LoadBalancer" .Values.service.type }}
  export SERVICE_IP=$(kubectl get svc --namespace {{ .Release.Namespace }} {{ include "myapp.fullname" . }} --template "{{"{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"}}")
  echo http://$SERVICE_IP:{{ .Values.service.port }}
{{- else }}
  kubectl port-forward --namespace {{ .Release.Namespace }} svc/{{ include "myapp.fullname" . }} 8080:{{ .Values.service.port }}
  echo http://127.0.0.1:8080
{{- end }}
```

---

## 🧪 Testing y Validación

### Linting

```bash
# Verificar sintaxis del chart
helm lint myapp/

# Output:
# ==> Linting myapp/
# [INFO] Chart.yaml: icon is recommended
# 1 chart(s) linted, 0 chart(s) failed
```

### Template Rendering

```bash
# Ver templates renderizados
helm template myapp ./myapp

# Con valores específicos
helm template myapp ./myapp -f values-prod.yaml

# Debug con valores
helm template myapp ./myapp --debug --set replicaCount=5
```

### Dry Run

```bash
# Simular instalación
helm install myapp ./myapp --dry-run --debug

# Ver qué recursos se crearían
helm upgrade myapp ./myapp --dry-run --debug
```

### Chart Testing

```bash
# Instalar plugin de testing
helm plugin install https://github.com/helm-unittest/helm-unittest

# Crear tests
mkdir myapp/tests
cat > myapp/tests/deployment_test.yaml << 'EOF'
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should create deployment with correct replicas
    set:
      replicaCount: 3
    asserts:
    - isKind:
        of: Deployment
    - equal:
        path: spec.replicas
        value: 3
EOF

# Ejecutar tests
helm unittest myapp/
```

---

## 📦 Empaquetar y Distribuir

### Empaquetar Chart

```bash
# Crear .tgz
helm package myapp/
# Output: myapp-0.1.0.tgz

# Con versión específica
helm package myapp/ --version 1.2.3
```

### Crear Repositorio Local

```bash
# Crear directorio
mkdir my-helm-repo

# Copiar charts
cp myapp-0.1.0.tgz my-helm-repo/

# Generar índice
helm repo index my-helm-repo/ --url https://my-domain.com/charts

# Publicar (subir a web server o GitHub Pages)
```

### Usar Repositorio Privado

```bash
# Agregar repo con autenticación
helm repo add private-repo https://charts.example.com \
  --username user \
  --password pass

# O con token
helm repo add private-repo https://charts.example.com \
  --password $GITHUB_TOKEN
```

### ChartMuseum (Repositorio Open Source)

```bash
# Instalar ChartMuseum
helm install chartmuseum stable/chartmuseum

# Subir chart
curl --data-binary "@myapp-0.1.0.tgz" http://chartmuseum:8080/api/charts

# Agregar repo
helm repo add my-charts http://chartmuseum:8080
```

---

## 🔧 Helm Plugins Útiles

```bash
# Diff - Ver cambios antes de upgrade
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade myapp ./myapp -f values-prod.yaml

# Secrets - Encriptar values
helm plugin install https://github.com/jkroepke/helm-secrets
helm secrets install myapp ./myapp -f secrets.yaml

# Push - Subir charts a repos
helm plugin install https://github.com/chartmuseum/helm-push
helm push myapp/ my-repo

# Unittest - Testing de charts
helm plugin install https://github.com/helm-unittest/helm-unittest
helm unittest myapp/
```

---

## 🐛 Troubleshooting

### Release Stuck

```bash
# Ver estado
helm list -a

# Ver recursos creados
kubectl get all -l app.kubernetes.io/instance=myapp

# Forzar eliminación
helm uninstall myapp --no-hooks

# Si aún no se elimina, borrar secret de Helm
kubectl delete secret -l owner=helm,name=myapp
```

### Template Errors

```bash
# Debug con verbose
helm install myapp ./myapp --debug --dry-run

# Ver línea exacta del error
helm template myapp ./myapp --debug 2>&1 | grep -A 5 "Error"
```

### Values No Aplican

```bash
# Verificar qué valores se están usando
helm get values myapp

# Ver values completos (incluyendo defaults)
helm get values myapp --all

# Ver manifest instalado
helm get manifest myapp
```

---

## 📊 Mejores Prácticas

### 1. Versionado Semántico

```yaml
# Chart.yaml
version: 1.2.3  # MAJOR.MINOR.PATCH
# MAJOR: cambios incompatibles
# MINOR: funcionalidad nueva compatible
# PATCH: bug fixes
```

### 2. Documentación

```yaml
# Comentar values.yaml extensivamente
# Incluir README.md con ejemplos
# NOTES.txt con instrucciones post-instalación
```

### 3. Defaults Sensatos

```yaml
# values.yaml debe funcionar out-of-the-box
replicaCount: 2  # No 1 (sin HA)
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m   # Siempre definir requests
    memory: 256Mi
```

### 4. Seguridad

```yaml
# No commitear secrets en values.yaml
# Usar helm-secrets o external-secrets
# SecurityContext por defecto:
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  readOnlyRootFilesystem: true
```

### 5. Dependencies

```yaml
# Chart.yaml
dependencies:
- name: postgresql
  version: "~12.1.0"  # ~ permite patch updates
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled  # Opcional

# Actualizar dependencies
helm dependency update
```

---

## 📚 Próximos Pasos

Con Helm dominado:

1. [**Kubernetes Best Practices**](./best-practices.md) → Producción-ready
2. **CI/CD con Helm** → ArgoCD, Flux
3. **Helmfile** → Gestión de múltiples releases

---

## 🔗 Recursos Adicionales

- [Helm Documentation](https://helm.sh/docs/)
- [Artifact Hub](https://artifacthub.io/) - Buscar charts públicos
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Chart Template Guide](https://helm.sh/docs/chart_template_guide/)

---

[⬅️ Volver: Networking](./networking.md) | [➡️ Siguiente: Best Practices](./best-practices.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
