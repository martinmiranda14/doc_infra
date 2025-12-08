# Contenedores y Orquestación

Esta sección cubre tecnologías de contenedores, desde conceptos básicos hasta orquestación avanzada. Aprenderás Docker para empaquetar aplicaciones y Kubernetes para gestionarlas a escala.

## 📋 Contenido

### 🐳 Docker

Plataforma para desarrollar, enviar y ejecutar aplicaciones en contenedores.

#### [Introducción a Docker](./docker/introduccion.md)
Conceptos fundamentales de contenedores y Docker:
- ¿Qué son los contenedores?
- Docker vs Máquinas Virtuales
- Arquitectura de Docker
- Casos de uso
- Instalación y primeros pasos

**Nivel:** 🟢 Básico

#### [Dockerfile](./docker/dockerfile.md)
Creación de imágenes personalizadas:
- Sintaxis y estructura
- Instrucciones principales (FROM, RUN, COPY, CMD, ENTRYPOINT)
- Multi-stage builds
- Optimización de imágenes
- Mejores prácticas

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [Docker Compose](./docker/docker-compose.md)
Gestión de aplicaciones multi-contenedor:
- Definición de servicios
- Redes y comunicación
- Volúmenes y persistencia
- Variables de entorno
- Comandos principales

**Nivel:** 🟡 Intermedio

#### [Networking en Docker](./docker/networking.md)
Comunicación entre contenedores:
- Tipos de redes (bridge, host, overlay)
- DNS y service discovery
- Exposición de puertos
- Redes personalizadas
- Troubleshooting

**Nivel:** 🟡 Intermedio

#### [Volúmenes en Docker](./docker/volumes.md)
Persistencia de datos:
- Tipos de almacenamiento (volumes, bind mounts, tmpfs)
- Gestión de volúmenes
- Backup y restore
- Casos de uso

**Nivel:** 🟡 Intermedio

#### [Mejores Prácticas Docker](./docker/best-practices.md)
Optimización y seguridad:
- Seguridad de imágenes
- Optimización de tamaño
- Caching de capas
- Troubleshooting común
- CI/CD con Docker

**Nivel:** 🟡 Intermedio - 🔴 Avanzado

---

### ☸️ Kubernetes

Sistema de orquestación de contenedores para automatizar despliegue, escalado y gestión.

#### [Introducción a Kubernetes](./kubernetes/introduccion.md)
Fundamentos de Kubernetes:
- ¿Qué es Kubernetes?
- Conceptos principales
- Cuándo usar Kubernetes
- Instalación (minikube, kind, k3s)

**Nivel:** 🟢 Básico

#### [Arquitectura de Kubernetes](./kubernetes/arquitectura.md)
Componentes del sistema:
- Control Plane (API Server, etcd, Scheduler, Controller Manager)
- Worker Nodes (kubelet, kube-proxy)
- Addons
- Flujo de comunicación

**Nivel:** 🟡 Intermedio

#### [Pods](./kubernetes/pods.md)
Unidad básica de despliegue:
- Lifecycle de pods
- Patrones multi-contenedor
- Init containers
- Health checks (liveness, readiness)
- Troubleshooting

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [Deployments y ReplicaSets](./kubernetes/deployments.md)
Gestión de aplicaciones:
- Deployments vs ReplicaSets
- Rolling updates
- Rollbacks
- Estrategias de despliegue
- Scaling

**Nivel:** 🟡 Intermedio

#### [Services e Ingress](./kubernetes/services.md)
Exposición de aplicaciones:
- Tipos de Services (ClusterIP, NodePort, LoadBalancer)
- Service Discovery
- Ingress Controllers
- Configuración de rutas
- TLS/SSL

**Nivel:** 🟡 Intermedio

#### [Networking en Kubernetes](./kubernetes/networking.md)
Comunicación en clúster:
- Modelo de red de Kubernetes
- CNI (Container Network Interface)
- Network Policies
- Service Mesh (Istio, Linkerd)

**Nivel:** 🔴 Avanzado

#### [Almacenamiento](./kubernetes/storage.md)
Persistencia en Kubernetes:
- PersistentVolumes (PV)
- PersistentVolumeClaims (PVC)
- StorageClasses
- StatefulSets
- Casos de uso

**Nivel:** 🟡 Intermedio - 🔴 Avanzado

#### [ConfigMaps y Secrets](./kubernetes/configmaps-secrets.md)
Gestión de configuración:
- ConfigMaps para configuración
- Secrets para datos sensibles
- Montaje en pods
- Mejores prácticas de seguridad

**Nivel:** 🟡 Intermedio

#### [Helm](./kubernetes/helm.md)
Gestor de paquetes para Kubernetes:
- Conceptos de Helm (Charts, Releases, Repositories)
- Instalación de aplicaciones
- Creación de Charts
- Templating
- Helm Hooks

**Nivel:** 🟡 Intermedio - 🔴 Avanzado

#### [Mejores Prácticas Kubernetes](./kubernetes/best-practices.md)
Optimización y seguridad:
- RBAC y seguridad
- Resource limits y requests
- Monitoreo y logging
- Disaster recovery
- Multi-tenancy

**Nivel:** 🔴 Avanzado

---

## 🎯 Objetivos de Aprendizaje

Al completar esta sección, deberías ser capaz de:

### Docker
1. ✅ Comprender qué son los contenedores y sus beneficios
2. ✅ Crear y gestionar imágenes con Dockerfile
3. ✅ Ejecutar y administrar contenedores
4. ✅ Usar Docker Compose para aplicaciones multi-contenedor
5. ✅ Configurar redes y persistencia de datos
6. ✅ Aplicar mejores prácticas de seguridad

### Kubernetes
1. ✅ Entender la arquitectura de Kubernetes
2. ✅ Desplegar aplicaciones con Deployments
3. ✅ Exponer servicios con Services e Ingress
4. ✅ Gestionar configuración con ConfigMaps y Secrets
5. ✅ Implementar persistencia con Volumes
6. ✅ Usar Helm para gestionar aplicaciones
7. ✅ Aplicar mejores prácticas de producción

---

## 🚀 Ruta de Aprendizaje Sugerida

### Para Principiantes

```
1. Introducción a Docker
   ↓
2. Dockerfile básico
   ↓
3. Docker Compose
   ↓
4. Introducción a Kubernetes
   ↓
5. Pods y Deployments
   ↓
6. Services básicos
```

### Para Intermedios

```
1. Networking Docker (revisión)
   ↓
2. Arquitectura Kubernetes
   ↓
3. Services e Ingress avanzado
   ↓
4. ConfigMaps y Secrets
   ↓
5. Storage en Kubernetes
   ↓
6. Helm
```

### Para Avanzados

```
1. Networking Kubernetes avanzado
   ↓
2. Service Mesh
   ↓
3. Mejores prácticas de seguridad
   ↓
4. Optimización de recursos
   ↓
5. Multi-cluster y HA
```

---

## 💡 Prerequisitos

Antes de comenzar esta sección, se recomienda:

- ✅ Conocimientos básicos de Linux/comandos de terminal
- ✅ Entender conceptos de redes (IP, puertos, DNS)
- ✅ Familiaridad con conceptos de [Fundamentos](../01-fundamentos/README.md)
- ✅ (Opcional) Conocimientos de desarrollo de aplicaciones

---

## 🔧 Laboratorio Recomendado

Para practicar, necesitarás:

### Docker
- Docker Desktop (Windows/Mac) o Docker Engine (Linux)
- 4GB RAM mínimo, 8GB recomendado
- Cuenta en Docker Hub (opcional, para push de imágenes)

### Kubernetes
- **Local:**
  - minikube (más común para aprendizaje)
  - kind (Kubernetes in Docker)
  - k3s (ligero)
- **Cloud:**
  - GKE (Google Kubernetes Engine)
  - EKS (Amazon Elastic Kubernetes Service)
  - AKS (Azure Kubernetes Service)

---

## 📚 Recursos Adicionales

### Documentación Oficial
- [Docker Docs](https://docs.docker.com/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Helm Docs](https://helm.sh/docs/)

### Certificaciones
- Docker Certified Associate (DCA)
- Certified Kubernetes Administrator (CKA)
- Certified Kubernetes Application Developer (CKAD)
- Certified Kubernetes Security Specialist (CKS)

---

## 🗺️ Navegación

[⬅️ Volver al índice principal](../README.md) | [➡️ Comenzar con Docker](./docker/introduccion.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
