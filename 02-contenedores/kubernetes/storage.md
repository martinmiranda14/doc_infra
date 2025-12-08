# Storage en Kubernetes

## 📖 Persistencia de Datos

Los contenedores son efímeros, pero los datos deben persistir. Kubernetes proporciona abstracciones para gestionar almacenamiento persistente.

## 🔄 Tipos de Almacenamiento

### emptyDir
Temporal, se borra cuando el pod muere.

```yaml
volumes:
- name: cache
  emptyDir: {}
```

### hostPath
Monta directorio del nodo (no recomendado en producción).

```yaml
volumes:
- name: data
  hostPath:
    path: /data
    type: Directory
```

### PersistentVolume (PV) y PersistentVolumeClaim (PVC)

**Arquitectura:**
```
Administrador → crea → PersistentVolume (infraestructura)
Developer → solicita → PersistentVolumeClaim
Kubernetes → vincula → PVC con PV
Pod → usa → PVC
```

## 💾 PersistentVolume (PV)

Recurso de almacenamiento en el cluster provisionado por administrador.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

**Access Modes:**
- `ReadWriteOnce` (RWO): Un nodo, lectura-escritura
- `ReadOnlyMany` (ROX): Múltiples nodos, solo lectura
- `ReadWriteMany` (RWX): Múltiples nodos, lectura-escritura

**Reclaim Policies:**
- `Retain`: Mantener datos después de eliminar PVC
- `Delete`: Eliminar volumen automáticamente
- `Recycle`: Limpiar y reutilizar (deprecated)

## 📝 PersistentVolumeClaim (PVC)

Solicitud de almacenamiento por usuario.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

## 🔌 Usar PVC en Pod

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
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: data-claim
```

## ⚙️ StorageClass

Provisiona PVs dinámicamente.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

**Proveedores:**
- AWS: `kubernetes.io/aws-ebs`
- GCP: `kubernetes.io/gce-pd`
- Azure: `kubernetes.io/azure-disk`
- Local: `kubernetes.io/no-provisioner`

## 🗄️ StatefulSets

Para aplicaciones stateful que necesitan:
- Identidad de red estable
- Almacenamiento persistente por pod
- Orden en despliegue/escalado

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 20Gi
```

**Características:**
- Pods con nombres predecibles: `postgres-0`, `postgres-1`, `postgres-2`
- PVC por pod: `data-postgres-0`, `data-postgres-1`, `data-postgres-2`
- Orden garantizado

## 📊 Ejemplo Completo: Base de Datos

```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: db-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
allowVolumeExpansion: true
---
# StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: db-storage
      resources:
        requests:
          storage: 50Gi
---
# Headless Service
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

## 🔍 Comandos Útiles

```bash
# Ver PVs
kubectl get pv

# Ver PVCs
kubectl get pvc

# Ver StorageClasses
kubectl get storageclass
kubectl get sc

# Detalles
kubectl describe pv pv-data
kubectl describe pvc data-claim

# Ver qué pod usa un PVC
kubectl get pods -o json | jq '.items[] | select(.spec.volumes[]?.persistentVolumeClaim.claimName=="data-claim") | .metadata.name'
```

## 🐛 Troubleshooting

### PVC Stuck en Pending

```bash
kubectl describe pvc data-claim

# Causas comunes:
# - No hay PV disponible que coincida
# - StorageClass no existe
# - Access mode no coincide
# - Tamaño insuficiente
```

### PV No Se Vincula

```bash
# Verificar compatibilidad
kubectl get pv
kubectl get pvc

# Debe coincidir:
# - storageClassName
# - accessModes
# - capacity (PV >= PVC)
```

---

[⬅️ Volver: ConfigMaps/Secrets](./configmaps-secrets.md) | [➡️ Siguiente: Networking](./networking.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
