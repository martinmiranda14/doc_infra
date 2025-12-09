# Cloud Computing

Esta sección cubre computación en la nube, desde conceptos fundamentales hasta implementaciones específicas en los principales proveedores cloud: AWS, Azure y GCP.

## 📋 Contenido

### ☁️ Conceptos Generales

#### [Introducción a Cloud Computing](./introduccion.md)
Fundamentos de la nube:
- ¿Qué es Cloud Computing?
- Modelos de servicio (IaaS, PaaS, SaaS)
- Modelos de despliegue (Público, Privado, Híbrido)
- Beneficios y consideraciones
- Comparación de proveedores

**Nivel:** 🟢 Básico

#### [Migración a la Nube](./migracion-cloud.md)
Estrategias de migración:
- Las 6 R's de migración (Rehost, Replatform, Refactor, etc.)
- Evaluación de cargas de trabajo
- Planificación de migración
- Casos de uso y patrones
- Costos y TCO

**Nivel:** 🟡 Intermedio

---

### 🟠 Amazon Web Services (AWS)

Líder del mercado cloud con el ecosistema más amplio de servicios.

#### [Introducción a AWS](./aws/introduccion.md)
Fundamentos de AWS:
- Estructura global (Regiones, AZs, Edge Locations)
- Servicios principales
- Consola y CLI
- IAM básico
- Pricing y Free Tier

**Nivel:** 🟢 Básico

#### [EC2 - Elastic Compute Cloud](./aws/ec2.md)
Computación virtual:
- Tipos de instancias
- AMIs (Amazon Machine Images)
- Storage (EBS, Instance Store)
- Networking (ENI, Security Groups)
- Auto Scaling
- Load Balancers

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [VPC - Virtual Private Cloud](./aws/vpc.md)
Redes en AWS:
- Subnets públicas y privadas
- Internet Gateway y NAT Gateway
- Route Tables
- Security Groups vs NACLs
- VPC Peering y Transit Gateway
- VPN y Direct Connect

**Nivel:** 🟡 Intermedio

#### [S3 - Simple Storage Service](./aws/s3.md)
Almacenamiento de objetos:
- Buckets y objetos
- Storage Classes
- Versionado y lifecycle policies
- Seguridad y encriptación
- Static website hosting
- CloudFront integration

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [RDS - Relational Database Service](./aws/rds.md)
Bases de datos gestionadas:
- Engines soportados (MySQL, PostgreSQL, etc.)
- Multi-AZ y Read Replicas
- Backups y snapshots
- Parameter Groups
- Performance Insights
- Aurora

**Nivel:** 🟡 Intermedio

#### [Lambda](./aws/lambda.md)
Computación serverless:
- Funciones Lambda
- Triggers y eventos
- Layers y runtime
- VPC y networking
- Monitoring y logs
- Casos de uso

**Nivel:** 🟡 Intermedio - 🔴 Avanzado

#### [IAM - Identity and Access Management](./aws/iam.md)
Gestión de accesos:
- Usuarios, grupos, roles
- Políticas (Policies)
- MFA
- Identity Federation
- Best practices de seguridad

**Nivel:** 🟡 Intermedio - 🔴 Avanzado

#### [CloudFormation](./aws/cloudformation.md)
Infraestructura como código:
- Templates (JSON/YAML)
- Stacks y StackSets
- Parameters y outputs
- Funciones intrínsecas
- Change Sets
- Nested stacks

**Nivel:** 🔴 Avanzado

---

### 🔵 Microsoft Azure

Plataforma cloud integrada con ecosistema Microsoft.

#### [Introducción a Azure](./azure/introduccion.md)
Fundamentos de Azure:
- Jerarquía (Subscriptions, Resource Groups)
- Servicios principales
- Portal, CLI y PowerShell
- Azure Active Directory
- Pricing

**Nivel:** 🟢 Básico

#### [Virtual Machines](./azure/virtual-machines.md)
Computación en Azure:
- VM sizes y series
- Availability Sets y Zones
- Managed Disks
- Scale Sets
- Azure Bastion

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [Networking](./azure/networking.md)
Redes en Azure:
- Virtual Networks (VNets)
- Subnets y NSGs
- Application Gateway
- Load Balancer
- VPN Gateway
- ExpressRoute

**Nivel:** 🟡 Intermedio

#### [Storage](./azure/storage.md)
Almacenamiento en Azure:
- Blob Storage
- File Storage
- Queue Storage
- Table Storage
- Redundancia y replicación

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [Azure AD](./azure/azure-ad.md)
Identidad y acceso:
- Usuarios y grupos
- RBAC
- Managed Identities
- Conditional Access
- Integration con on-premise

**Nivel:** 🟡 Intermedio

#### [ARM Templates](./azure/arm-templates.md)
Infraestructura como código:
- Estructura de templates
- Parameters y variables
- Deployment modes
- Linked templates
- Bicep

**Nivel:** 🔴 Avanzado

---

### 🟢 Google Cloud Platform (GCP)

Plataforma cloud de Google con fuerte énfasis en datos y ML.

#### [Introducción a GCP](./gcp/introduccion.md)
Fundamentos de GCP:
- Jerarquía (Organizations, Projects)
- Servicios principales
- Console y gcloud CLI
- IAM en GCP
- Pricing

**Nivel:** 🟢 Básico

#### [Compute Engine](./gcp/compute-engine.md)
Computación virtual:
- Machine types
- Instance templates
- Managed Instance Groups
- Persistent Disks
- Snapshots

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [Networking](./gcp/networking.md)
Redes en GCP:
- VPC y subnets
- Firewall rules
- Cloud Load Balancing
- Cloud VPN
- Cloud Interconnect

**Nivel:** 🟡 Intermedio

#### [Cloud Storage](./gcp/cloud-storage.md)
Almacenamiento de objetos:
- Buckets y objetos
- Storage classes
- Access control
- Lifecycle management
- Signed URLs

**Nivel:** 🟢 Básico - 🟡 Intermedio

#### [GKE - Google Kubernetes Engine](./gcp/gke.md)
Kubernetes gestionado:
- Cluster creation
- Node pools
- Autopilot vs Standard
- Workload Identity
- Integration con GCP services

**Nivel:** 🟡 Intermedio - 🔴 Avanzado

---

## 🎯 Objetivos de Aprendizaje

Al completar esta sección, deberías ser capaz de:

### Conceptos Generales
1. ✅ Entender modelos de servicio cloud (IaaS, PaaS, SaaS)
2. ✅ Conocer estrategias de migración a la nube
3. ✅ Comparar proveedores cloud principales
4. ✅ Evaluar costos y TCO

### AWS
1. ✅ Desplegar y gestionar instancias EC2
2. ✅ Configurar redes con VPC
3. ✅ Usar S3 para almacenamiento de objetos
4. ✅ Gestionar bases de datos con RDS
5. ✅ Crear funciones Lambda serverless
6. ✅ Implementar seguridad con IAM
7. ✅ Automatizar con CloudFormation

### Azure
1. ✅ Trabajar con Virtual Machines
2. ✅ Configurar redes virtuales
3. ✅ Usar Azure Storage
4. ✅ Gestionar identidades con Azure AD
5. ✅ Desplegar con ARM Templates

### GCP
1. ✅ Usar Compute Engine
2. ✅ Configurar VPC networking
3. ✅ Trabajar con Cloud Storage
4. ✅ Desplegar clusters GKE

---

## 🚀 Ruta de Aprendizaje Sugerida

### Para Principiantes

```
1. Introducción a Cloud Computing
   ↓
2. Elegir un proveedor (recomendado: AWS para comenzar)
   ↓
3. AWS Introducción
   ↓
4. EC2 básico
   ↓
5. S3 básico
   ↓
6. IAM básico
```

### Para Intermedios

```
1. Migración a Cloud (conceptos)
   ↓
2. VPC y Networking avanzado
   ↓
3. RDS y bases de datos
   ↓
4. Lambda y serverless
   ↓
5. Explorar segundo proveedor (Azure o GCP)
```

### Para Avanzados

```
1. Multi-cloud strategies
   ↓
2. Infrastructure as Code (CloudFormation, ARM, Terraform)
   ↓
3. Arquitecturas avanzadas
   ↓
4. Cost optimization
   ↓
5. Disaster recovery cross-cloud
```

---

## 💡 Prerequisitos

Antes de comenzar esta sección, se recomienda:

- ✅ Conocimientos de [Fundamentos de Infraestructura](../01-fundamentos/README.md)
- ✅ Familiaridad con redes (IP, subnetting, routing)
- ✅ (Opcional) Experiencia con [Contenedores](../02-contenedores/README.md)
- ✅ Entender conceptos de virtualización

---

## 🔧 Laboratorio Recomendado

Para practicar, necesitarás:

### AWS
- Cuenta AWS (Free Tier disponible)
- AWS CLI instalado
- Tarjeta de crédito (no se cobra en Free Tier)

### Azure
- Cuenta Azure (créditos gratuitos disponibles)
- Azure CLI o PowerShell
- Suscripción activa

### GCP
- Cuenta GCP (créditos iniciales disponibles)
- gcloud CLI instalado
- Billing account

**⚠️ Importante:** Siempre monitorear costos y usar alertas de billing.

---

## 📊 Comparación de Proveedores

| Feature | AWS | Azure | GCP |
|---------|-----|-------|-----|
| **Cuota de mercado** | ~32% | ~23% | ~10% |
| **Servicios** | 200+ | 200+ | 100+ |
| **Regiones** | 30+ | 60+ | 35+ |
| **Fortaleza** | Ecosistema más amplio | Integración Microsoft | ML/Data analytics |
| **Free Tier** | 12 meses + perpetuo | 12 meses + créditos | 90 días + perpetuo |
| **Certificaciones** | 12 | 9 | 5 |

---

## 📚 Recursos Adicionales

### Documentación Oficial
- [AWS Documentation](https://docs.aws.amazon.com/)
- [Azure Documentation](https://docs.microsoft.com/azure/)
- [GCP Documentation](https://cloud.google.com/docs)

### Certificaciones
#### AWS
- AWS Certified Cloud Practitioner (Foundational)
- AWS Certified Solutions Architect - Associate
- AWS Certified Developer - Associate
- AWS Certified SysOps Administrator - Associate

#### Azure
- Azure Fundamentals (AZ-900)
- Azure Administrator (AZ-104)
- Azure Solutions Architect (AZ-305)

#### GCP
- Cloud Digital Leader
- Associate Cloud Engineer
- Professional Cloud Architect

### Training Gratuito
- [AWS Skill Builder](https://skillbuilder.aws/)
- [Microsoft Learn](https://docs.microsoft.com/learn/)
- [Google Cloud Skills Boost](https://www.cloudskillsboost.google/)

---

## 🗺️ Navegación

[⬅️ Volver al índice principal](../README.md) | [➡️ Comenzar con Introducción](./introduccion.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
