# Migración a la Nube

## 📖 Introducción

La migración a la nube es el proceso de mover aplicaciones, datos y otros elementos empresariales de infraestructura on-premise o legacy a un entorno cloud.

**No es simplemente "lift and shift"** - requiere planificación estratégica, evaluación y ejecución cuidadosa.

---

## 🎯 Las 6 R's de Migración

Framework de AWS para estrategias de migración cloud.

### 1. Rehost (Lift and Shift)

**Definición:** Mover aplicaciones sin cambios a la nube.

```
On-Premise Server → EC2 Instance
(mismo OS, mismas apps, misma config)
```

**Cuándo usar:**
- Migración rápida necesaria
- Aplicaciones legacy que funcionan bien
- Proof of concept
- Reducción de datacenter urgente

**Ventajas:**
✅ Migración rápida (días/semanas)
✅ Bajo riesgo
✅ No requiere cambios en código
✅ Skills existentes aplican

**Desventajas:**
❌ No optimizado para cloud
❌ No aprovecha servicios managed
❌ Costos potencialmente mayores
❌ Deuda técnica persiste

**Herramientas:**
- AWS Application Migration Service (MGN)
- Azure Migrate
- Google Cloud Migrate for Compute Engine

**Ejemplo:**
```yaml
# On-premise
- Physical server
- VMware VM
- 8 vCPU, 32GB RAM
- Windows Server 2019
- SQL Server Standard

# AWS (Rehost)
- EC2 m5.2xlarge
- 8 vCPU, 32GB RAM  
- Windows Server 2019 AMI
- SQL Server Standard
```

### 2. Replatform (Lift, Tinker and Shift)

**Definición:** Optimizaciones menores sin cambiar arquitectura core.

```
On-Premise DB → RDS (managed database)
Physical server → EC2 con mejor sizing
```

**Cuándo usar:**
- Quieres algunos beneficios cloud
- Tiempo moderado disponible
- Reducir gestión operativa
- Mejorar performance

**Ventajas:**
✅ Mejor que rehost
✅ Aprovecha servicios managed
✅ Menos gestión operativa
✅ Mejoras de costo

**Desventajas:**
❌ Requiere algo de testing
❌ Cambios de configuración
❌ Posible downtime
❌ Vendor lock-in moderado

**Ejemplos comunes:**
```
Before → After
--------------------
Self-managed DB → RDS/Aurora
Self-managed cache → ElastiCache
File server → EFS/S3
Load balancer → ALB/NLB
```

**Caso de estudio:**
```yaml
# Antes
- VM con MySQL
- Backups manuales
- No HA
- Patching manual

# Después (Replatform)
- RDS MySQL
- Backups automáticos
- Multi-AZ HA
- Patching automático
- Ahorro: 30% de gestión operativa
```

### 3. Repurchase (Drop and Shop)

**Definición:** Cambiar a un producto diferente, típicamente SaaS.

```
On-premise Exchange → Microsoft 365
Self-hosted CRM → Salesforce
Jenkins on VM → GitHub Actions
```

**Cuándo usar:**
- Software legacy costoso de mantener
- Disponible SaaS equivalente
- Quieres outsource completamente
- Modelo de licenciamiento mejor en cloud

**Ventajas:**
✅ Cero gestión de infraestructura
✅ Actualizaciones automáticas
✅ Features modernos
✅ Potencial reducción de costos

**Desventajas:**
❌ Re-training de usuarios
❌ Posible pérdida de features
❌ Migración de datos compleja
❌ Vendor lock-in total

**Ejemplos populares:**
```
Category | On-Premise | SaaS
---------|------------|------
Email | Exchange | Microsoft 365, Gmail
CRM | Custom | Salesforce, HubSpot
HR | PeopleSoft | Workday
File Share | Windows Server | Dropbox, Box
Monitoring | Nagios | Datadog, New Relic
```

### 4. Refactor (Re-architect)

**Definición:** Re-imaginar cómo se construye y ejecuta la app usando arquitecturas cloud-native.

```
Monolith → Microservices
VM-based → Serverless
Traditional → Container-based
```

**Cuándo usar:**
- Necesitas escalabilidad cloud
- Modernización es objetivo
- Problemas con arquitectura actual
- Tiempo y budget disponibles

**Ventajas:**
✅ Optimizado para cloud
✅ Escalabilidad máxima
✅ Costos optimizados
✅ Resiliencia mejorada

**Desventajas:**
❌ Tiempo significativo (meses)
❌ Alto costo inicial
❌ Requiere skills cloud-native
❌ Riesgo de proyecto alto

**Patrones de refactoring:**

#### Monolith → Microservices
```
Before:
┌─────────────────────────┐
│    Monolithic App       │
│  ┌──────────────────┐   │
│  │ User Management  │   │
│  │ Products         │   │
│  │ Orders           │   │
│  │ Payments         │   │
│  └──────────────────┘   │
└─────────────────────────┘

After:
┌──────────┐  ┌──────────┐
│ User Svc │  │ Prod Svc │
└──────────┘  └──────────┘
┌──────────┐  ┌──────────┐
│ Order Svc│  │ Pay Svc  │
└──────────┘  └──────────┘
```

#### VM → Serverless
```
Before:
EC2 (24/7) → API → Database
Cost: $100/month even at 1% usage

After:
API Gateway → Lambda → DynamoDB
Cost: $5/month at 1% usage
```

**Ejemplo real:**
```yaml
# Antes: Monolito en VMs
- 3 VMs (web, app, db)
- Manual scaling
- 30 min deployment
- $500/month

# Después: Cloud-native
- S3 + CloudFront (frontend)
- API Gateway + Lambda (backend)
- DynamoDB (database)
- Auto-scaling
- 5 min deployment (CI/CD)
- $150/month
- 10x más escalable
```

### 5. Retire

**Definición:** Apagar aplicaciones que ya no se necesitan.

```
Discovery:
- 100 aplicaciones identificadas
- 20 no usadas en 6+ meses
- 15 duplicadas
→ No migrar, ELIMINAR
```

**Cuándo usar:**
- App no usada
- Funcionalidad duplicada
- Reemplazada por otra solución
- Regulaciones permiten eliminación

**Beneficios:**
✅ Reduce scope de migración
✅ Ahorro inmediato
✅ Menos complejidad
✅ Enfoque en apps críticas

**Proceso:**
```
1. Identificar candidatos
   ↓
2. Validar con stakeholders
   ↓
3. Backup de datos (compliance)
   ↓
4. Apagar gradualmente
   ↓
5. Monitorear por 30-90 días
   ↓
6. Eliminar permanentemente
```

### 6. Retain (Revisit)

**Definición:** Mantener en on-premise, al menos por ahora.

```
Razones para retener:
- Compliance/regulatorio
- Latencia crítica
- Inversión reciente en hardware
- No listo para migrar aún
```

**Cuándo usar:**
- Aplicaciones críticas que requieren más análisis
- Hardware recién comprado
- Regulaciones impiden cloud
- Dependencies complejas

**Estrategia:**
```
1. Documentar razón para retener
2. Establecer fecha de revisión
3. Evaluar híbrido cloud
4. Planificar migración futura
```

---

## 📋 Fases de Migración

### Fase 1: Evaluación (Assessment)

**Objetivo:** Entender qué tienes y qué migrar.

```
Actividades:
1. Inventario de aplicaciones
2. Mapeo de dependencias
3. Assessment de complejidad
4. Análisis de costos
5. Identificación de riesgos
```

**Herramientas:**
- AWS Migration Hub
- Azure Migrate
- Cloudamize
- TSO Logic

**Deliverables:**
- Portfolio de aplicaciones
- Matriz de complejidad
- Business case
- Roadmap de migración

**Ejemplo de assessment:**
```
Application Portfolio:

┌──────────────────────────────────────────────────┐
│ App Name | Users | Dependencies | Strategy      │
├──────────────────────────────────────────────────┤
│ WebApp1  | 1000  | DB, Cache    | Refactor      │
│ LegacyDB | 500   | 10 apps      | Replatform    │
│ FileServ | 100   | None         | Repurchase    │
│ OldCRM   | 5     | None         | Retire        │
│ Mainfrm  | 2000  | Complex      | Retain        │
└──────────────────────────────────────────────────┘
```

### Fase 2: Planificación (Planning)

**Objetivo:** Crear plan detallado de migración.

```
Componentes del plan:
1. Secuencia de migración
2. Arquitectura target
3. Estrategia por app (6 R's)
4. Timeline y recursos
5. Plan de rollback
```

**Consideraciones:**

**Orden de migración:**
```
Ola 1: Apps no críticas (POC)
  ↓
Ola 2: Apps con pocas dependencies
  ↓
Ola 3: Apps críticas
  ↓
Ola 4: Legacy complejos
```

**Arquitectura target:**
```yaml
# Definir por app:
- Región AWS/Azure/GCP
- Networking (VPC, subnets)
- Compute (EC2, containers, lambda)
- Storage (EBS, S3, EFS)
- Database (RDS, DynamoDB)
- Security (IAM, security groups)
- Monitoring (CloudWatch)
- Backup/DR strategy
```

### Fase 3: Migración (Migration)

**Objetivo:** Ejecutar migración con mínimo impacto.

#### Estrategias de cutover:

**Big Bang:**
```
Friday night:
- Shutdown on-premise
- Migrate data
- Switch DNS
- Go live on cloud
Monday morning: 100% cloud
```
✅ Rápido
❌ Alto riesgo

**Phased/Iterative:**
```
Week 1: Migrate 10% users
Week 2: Migrate 25% users
Week 3: Migrate 50% users
Week 4: Migrate 100% users
```
✅ Bajo riesgo
❌ Complejidad operativa

**Parallel Run:**
```
Run on-premise AND cloud simultaneously
Validate results
Gradual cutover
```
✅ Muy seguro
❌ Doble costo temporal

#### Pasos típicos:

**Para VM migration (Rehost):**
```
1. Setup target environment
   - VPC, subnets, security groups
   
2. Install replication agent
   - AWS MGN agent on source VM
   
3. Replicate data
   - Continuous replication to AWS
   
4. Test migration
   - Launch test instance
   - Validate functionality
   
5. Cutover
   - Final sync
   - Switch production
   
6. Decommission source
   - After validation period
```

**Para database migration:**
```
1. Setup target DB (RDS)

2. Schema migration
   - AWS SCT (Schema Conversion Tool)
   
3. Data replication
   - AWS DMS (Database Migration Service)
   - Continuous replication
   
4. Validation
   - Data integrity checks
   
5. Cutover
   - Stop writes to source
   - Final sync
   - Switch app to target
```

### Fase 4: Optimización (Optimization)

**Objetivo:** Aprovechar beneficios cloud completos.

```
Post-migration optimization:
1. Right-sizing
2. Reserved Instances
3. Auto-scaling setup
4. Monitoring enhancement
5. Cost optimization
6. Security hardening
```

**Áreas de optimización:**

**Compute:**
```
- Rightsizing instances
- Spot instances para batch jobs
- Lambda para event-driven
- Containers para microservices
```

**Storage:**
```
- S3 Intelligent-Tiering
- EBS gp3 vs gp2
- Lifecycle policies
- Compression
```

**Database:**
```
- Read replicas para lectura
- Caching (ElastiCache)
- Connection pooling
- Query optimization
```

**Networking:**
```
- CloudFront CDN
- VPC endpoints (evitar NAT gateway)
- Direct Connect para on-premise
```

---

## 💰 Análisis de Costos

### TCO (Total Cost of Ownership)

**On-Premise:**
```
Hardware: $100,000
+ Datacenter: $50,000/year
+ Power/cooling: $20,000/year
+ Staff: $200,000/year
+ Maintenance: $30,000/year
+ Software licenses: $50,000/year
──────────────────────────
= $350,000/year

Over 3 years: $1,150,000
```

**Cloud:**
```
Compute: $100,000/year
+ Storage: $20,000/year
+ Networking: $10,000/year
+ Managed services: $30,000/year
+ Support: $20,000/year
──────────────────────────
= $180,000/year

Over 3 years: $540,000
Savings: $610,000 (53%)
```

### Factores ocultos:

**On-premise hidden costs:**
- Hardware refresh (3-5 años)
- Datacenter upgrades
- Disaster recovery site
- Overprovisioning (30-50%)
- Training staff

**Cloud hidden costs:**
- Data transfer (egress)
- Over-provisioned resources
- Untagged/forgotten resources
- Premium support
- Training/certification

---

## 🚧 Desafíos Comunes

### 1. Legacy Applications

**Problema:**
```
- Código antiguo (COBOL, etc.)
- Sin documentación
- Dependencias desconocidas
- Arquitectura monolítica
```

**Soluciones:**
- Containerización (Docker)
- Strangler pattern (gradual replacement)
- AWS Mainframe Modernization
- Mantener híbrido inicialmente

### 2. Downtime

**Problema:**
```
- 24/7 business operations
- Cero downtime tolerance
- Large databases (TB+)
```

**Soluciones:**
- Database replication (DMS)
- Blue-green deployment
- DNS weighted routing
- Parallel run strategy

### 3. Data Transfer

**Problema:**
```
- 10TB database
- Internet: 100 Mbps
- Transfer time: ~200 hours

Plus: bandwidth costs $$$
```

**Soluciones:**
- AWS Snowball (physical device)
- AWS DataSync
- Direct Connect setup
- Incremental migration

### 4. Compliance

**Problema:**
```
- HIPAA, PCI-DSS, GDPR
- Data sovereignty
- Audit requirements
```

**Soluciones:**
- Use compliant cloud services
- Encryption at rest/transit
- Compliance frameworks (AWS Config)
- Data residency controls

### 5. Skills Gap

**Problema:**
```
- Team conoce on-premise
- No experiencia cloud
- Resistencia al cambio
```

**Soluciones:**
- Training y certificaciones
- Contratar cloud expertise
- Managed services inicialmente
- Cloud Center of Excellence

---

## 📊 Métricas de Éxito

### KPIs de Migración

```yaml
Technical:
- Migration velocity: apps/month
- Downtime per migration: hours
- Success rate: %
- Rollback rate: %

Business:
- Cost savings: %
- Time-to-market: reduction %
- Availability: SLA improvement
- Performance: latency improvement

Operational:
- Deployment frequency: increase %
- Mean time to recovery: reduction
- Change failure rate: %
- Team satisfaction: score
```

---

## 🎓 Best Practices

### 1. Start Small

```
POC → Pilot → Scale
- 1 non-critical app first
- Learn and iterate
- Build confidence
```

### 2. Automate

```
- Infrastructure as Code (Terraform, CloudFormation)
- CI/CD pipelines
- Automated testing
- Configuration management
```

### 3. Security First

```
- Identity and Access Management
- Encryption everywhere
- Network segmentation
- Continuous compliance
```

### 4. Monitor Everything

```
- Application performance
- Infrastructure health
- Costs
- Security
```

### 5. Cloud Center of Excellence

```
Team dedicado para:
- Establecer standards
- Best practices
- Training
- Governance
```

---

## 🛠️ Herramientas

### Discovery

- **AWS Application Discovery Service**
- **Azure Migrate**
- **CloudPhysics**
- **Device42**

### Migration

- **AWS Migration Hub**
- **AWS Application Migration Service (MGN)**
- **Azure Site Recovery**
- **Google Cloud Migrate**

### Database

- **AWS DMS (Database Migration Service)**
- **AWS SCT (Schema Conversion Tool)**
- **Azure Database Migration Service**

### Cost Analysis

- **AWS Migration Evaluator** (ex-TSO Logic)
- **Azure TCO Calculator**
- **Cloudability**

---

## 📚 Próximos Pasos

Con estrategias de migración claras:

1. [**AWS Introducción**](./aws/introduccion.md) → Comenzar con AWS
2. [**Azure Introducción**](./azure/introduccion.md) → Comenzar con Azure
3. [**GCP Introducción**](./gcp/introduccion.md) → Comenzar con GCP

---

## 🔗 Recursos Adicionales

- [AWS Migration Hub](https://aws.amazon.com/migration-hub/)
- [Azure Migration Center](https://azure.microsoft.com/migration/)
- [GCP Migration Center](https://cloud.google.com/migration-center)
- [AWS Migration Whitepaper](https://d1.awsstatic.com/whitepapers/Migration/aws-migration-whitepaper.pdf)
- [Gartner Magic Quadrant: Cloud Migration](https://www.gartner.com/en/documents)

---

[⬅️ Volver: Introducción](./introduccion.md) | [➡️ Siguiente: AWS](./aws/introduccion.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
