# Introducción a AWS (Amazon Web Services)

## 📖 ¿Qué es AWS?

**Amazon Web Services (AWS)** es la plataforma cloud más completa y ampliamente adoptada, ofreciendo más de 200 servicios desde datacenters globalmente.

### Historia

```
2006: Launch (S3, EC2, SQS)
2010: Amazon.com migra completamente
2015: $10B revenue run rate
2020: $45B revenue
2024: Líder del mercado (32% cuota)
```

---

## 🌍 Infraestructura Global

### Regiones (Regions)

Ubicaciones geográficas donde AWS tiene datacenters.

```
Regiones principales:
- us-east-1 (Virginia) - Más antigua y económica
- us-west-2 (Oregon)
- eu-west-1 (Irlanda)
- ap-southeast-1 (Singapur)
- sa-east-1 (São Paulo)

Total: 30+ regiones globalmente
```

**Criterios para elegir región:**
1. **Compliance:** Regulaciones de datos (GDPR, etc.)
2. **Latencia:** Cercanía a usuarios
3. **Servicios:** No todos los servicios en todas las regiones
4. **Costo:** Precios varían por región

### Availability Zones (AZs)

Datacenters aislados dentro de una región.

```
┌─────────────────────────────────────┐
│          Region (us-east-1)         │
│                                     │
│  ┌──────────┐  ┌──────────┐       │
│  │ AZ-1a    │  │ AZ-1b    │       │
│  │          │  │          │       │
│  └──────────┘  └──────────┘       │
│                                     │
│  ┌──────────┐  ┌──────────┐       │
│  │ AZ-1c    │  │ AZ-1d    │       │
│  │          │  │          │       │
│  └──────────┘  └──────────┘       │
└─────────────────────────────────────┘

Cada AZ:
- 1+ datacenter
- Alimentación independiente
- Networking redundante
- Conectados con baja latencia
```

**Alta disponibilidad:**
```yaml
# Deploy en múltiples AZs
Web Tier:
  - us-east-1a: 2 instancias
  - us-east-1b: 2 instancias
  - us-east-1c: 2 instancias

Database:
  - Primary: us-east-1a
  - Standby: us-east-1b
```

### Edge Locations

Puntos de presencia para CloudFront CDN y Route 53.

```
400+ Edge Locations globalmente
- Más distribuidas que regiones
- Caching de contenido
- DNS resolution
- DDoS protection
```

---

## 🎯 Servicios Principales

### Categorías de Servicios

#### 1. Compute

**EC2 (Elastic Compute Cloud)**
- Servidores virtuales
- Más usado y fundamental

**Lambda**
- Serverless compute
- Pay per invocation

**ECS/EKS**
- Container orchestration
- Docker/Kubernetes

#### 2. Storage

**S3 (Simple Storage Service)**
- Object storage
- Ilimitado, altamente disponible

**EBS (Elastic Block Store)**
- Block storage para EC2
- Como discos duros virtuales

**EFS (Elastic File System)**
- Network file system
- Compartido entre instancias

#### 3. Database

**RDS (Relational Database Service)**
- MySQL, PostgreSQL, SQL Server, etc.
- Managed database

**DynamoDB**
- NoSQL database
- Serverless, alta performance

**Aurora**
- MySQL/PostgreSQL compatible
- 5x performance de MySQL

#### 4. Networking

**VPC (Virtual Private Cloud)**
- Red virtual privada
- Control completo de networking

**Route 53**
- DNS service
- Alta disponibilidad

**CloudFront**
- CDN (Content Delivery Network)
- Global distribution

#### 5. Security & Identity

**IAM (Identity and Access Management)**
- Control de accesos
- Usuarios, grupos, roles

**KMS (Key Management Service)**
- Encryption keys
- Managed encryption

#### 6. Management

**CloudWatch**
- Monitoring y logs
- Metrics y alarmas

**CloudFormation**
- Infrastructure as Code
- Templates JSON/YAML

**Systems Manager**
- Gestión de instancias
- Patching, automation

---

## 💰 Pricing y Free Tier

### Modelos de Pricing

#### 1. Pay-as-you-go

```
EC2: $0.10/hora
S3: $0.023/GB/mes
Data transfer: $0.09/GB

Ventaja: Flexibilidad
```

#### 2. Save when you commit

**Reserved Instances (1-3 años)**
```
Standard RI: 40-60% descuento
Convertible RI: 30-54% descuento
```

**Savings Plans**
```
Commit $X/hora por 1-3 años
Flexible entre servicios
```

#### 3. Pay less by using more

```
S3 Tiered pricing:
First 50 TB: $0.023/GB
Next 450 TB: $0.022/GB
Over 500 TB: $0.021/GB
```

### AWS Free Tier

#### Always Free

```
Lambda: 1M requests/mes
DynamoDB: 25GB storage
CloudFront: 1TB transfer/mes
CloudWatch: 10 custom metrics
```

#### 12 Months Free

```
EC2: 750 horas/mes t2.micro/t3.micro
S3: 5GB standard storage
RDS: 750 horas/mes db.t2.micro
CloudFront: 50GB transfer
```

#### Trials

```
SageMaker: 2 meses
Lightsail: 1 mes
Inspector: 90 días
```

**⚠️ Importante:**
- Configurar billing alerts
- No todos los servicios tienen Free Tier
- Excesos se cobran automáticamente

---

## 🔐 IAM Básico

### Conceptos Fundamentales

#### Root Account

```
❌ NO USAR para operaciones diarias
✅ Solo para:
  - Crear primer usuario admin
  - Configurar billing
  - Cerrar cuenta
  
SIEMPRE:
- Habilitar MFA
- No crear access keys
```

#### Users

```yaml
# Usuario individual
User: john@company.com
Permissions:
  - Attach policies
  - Add to groups
```

#### Groups

```yaml
# Grupo de usuarios con permisos comunes
Group: Developers
Users:
  - john
  - jane
  - bob
Policies:
  - EC2FullAccess
  - S3ReadOnly
```

#### Roles

```yaml
# Para services, no usuarios
Role: EC2-S3-Access
Trusted entity: EC2
Permissions:
  - S3ReadWriteAccess

# EC2 instance asume el role
# No necesita credentials hardcoded
```

#### Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### Best Practices IAM

```
✅ Habilitar MFA para root
✅ Crear usuarios IAM individuales
✅ Usar groups para permisos
✅ Least privilege principle
✅ Rotar credentials regularmente
✅ Usar roles para applications
✅ Monitorear actividad (CloudTrail)

❌ Usar root account
❌ Compartir credentials
❌ Hardcode access keys
❌ Dar permisos "*" sin necesidad
```

---

## 🖥️ Consola AWS

### Acceso a AWS

#### 1. AWS Management Console

```
https://console.aws.amazon.com/

- Interface web
- Punto-y-click
- Ideal para comenzar
- Algunos servicios solo en consola
```

#### 2. AWS CLI

```bash
# Instalación
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install

# Configuración
$ aws configure
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name: us-east-1
Default output format: json

# Ejemplo de uso
$ aws ec2 describe-instances
$ aws s3 ls
$ aws rds describe-db-instances
```

#### 3. AWS SDKs

```python
# Python (boto3)
import boto3

s3 = boto3.client('s3')
response = s3.list_buckets()

for bucket in response['Buckets']:
    print(bucket['Name'])
```

```javascript
// Node.js
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

s3.listBuckets((err, data) => {
  console.log(data.Buckets);
});
```

#### 4. CloudShell

```
Browser-based shell en AWS Console
- Pre-configurado con CLI
- 1GB persistent storage
- No necesita instalar nada
```

---

## 🚀 Primeros Pasos

### 1. Crear Cuenta AWS

```
1. https://aws.amazon.com/
2. "Create an AWS Account"
3. Email y password
4. Información de billing (tarjeta requerida)
5. Verificación de identidad
6. Elegir support plan (Basic = Free)
```

### 2. Asegurar Root Account

```bash
1. AWS Console → IAM
2. "Activate MFA on root account"
3. Elegir MFA device (Google Authenticator, etc.)
4. Escanear QR code
5. Ingresar 2 códigos consecutivos
```

### 3. Crear Usuario Admin

```yaml
1. IAM → Users → Add User
Name: admin-user
Access type: ☑ Password ☑ Access keys

2. Permissions
☑ Add user to group
   Create group: Admins
   Policy: AdministratorAccess

3. Review y Create

4. ⚠️ GUARDAR Access Keys (última oportunidad)
```

### 4. Configurar Billing Alerts

```
1. AWS Console → Billing Dashboard
2. "Billing preferences"
3. ☑ Receive Billing Alerts
4. Save preferences

5. CloudWatch → Alarms → Create Alarm
6. Select metric → Billing → Total Estimated Charge
7. Threshold: $10 (o tu límite)
8. Notification: Create SNS topic (tu email)
```

### 5. Primer Servicio: S3 Bucket

```bash
# Via Console
1. S3 → Create bucket
2. Name: my-first-bucket-[unique-id]
3. Region: us-east-1
4. Block all public access: ☑
5. Create bucket

# Via CLI
$ aws s3 mb s3://my-first-bucket-12345
$ echo "Hello AWS" > hello.txt
$ aws s3 cp hello.txt s3://my-first-bucket-12345/
$ aws s3 ls s3://my-first-bucket-12345/
```

---

## 📋 Navegación en Console

### Dashboard Principal

```
┌─────────────────────────────────────────┐
│ AWS Management Console                  │
├─────────────────────────────────────────┤
│ Services (buscar servicios)             │
│                                         │
│ Recently visited:                       │
│   - EC2                                 │
│   - S3                                  │
│   - RDS                                 │
│                                         │
│ Favorites: (pin servicios frecuentes)   │
│                                         │
│ Cost & Usage (billing actual)           │
│ Health Dashboard (issues de AWS)        │
└─────────────────────────────────────────┘
```

### Tips de Navegación

```
1. Buscar servicios: Alt+S
2. Favoritos: Pin común-used services
3. Region selector: Top-right corner
4. Resource Groups: Organizar recursos
5. AWS Console Mobile App: iOS/Android
```

---

## 💡 Mejores Prácticas Iniciales

### Seguridad

```
✅ Habilitar MFA en root
✅ No usar root para ops diarias
✅ Usar IAM roles en EC2
✅ Enable CloudTrail (audit log)
✅ Rotate access keys cada 90 días
```

### Costos

```
✅ Billing alerts configurados
✅ Usar tags para tracking
✅ Free tier awareness
✅ Stop resources not in use
✅ Review Cost Explorer mensualmente
```

### Arquitectura

```
✅ Multi-AZ para producción
✅ Backups automatizados
✅ Usar managed services cuando posible
✅ Infrastructure as Code (CloudFormation)
```

---

## 🎓 Aprendizaje y Certificaciones

### Certificaciones AWS

#### Foundational
```
AWS Certified Cloud Practitioner
- Entry level
- No experiencia técnica requerida
- Conceptos generales cloud
```

#### Associate
```
Solutions Architect - Associate
- Diseño de sistemas en AWS
- Más demandada

Developer - Associate
- Desarrollo en AWS
- CI/CD, Lambda

SysOps Administrator - Associate
- Operaciones en AWS
- Monitoring, security
```

#### Professional
```
Solutions Architect - Professional
- Arquitecturas complejas
- Multi-region, hybrid

DevOps Engineer - Professional
- Automation, CI/CD avanzado
```

#### Specialty
```
- Advanced Networking
- Security
- Machine Learning
- Database
- Data Analytics
- SAP on AWS
```

### Recursos de Aprendizaje

**Gratuitos:**
```
- AWS Skill Builder (oficial)
- AWS Workshops (hands-on)
- AWS Well-Architected Labs
- AWS YouTube channel
- Free tier hands-on
```

**Pagos:**
```
- A Cloud Guru
- Udemy (Stephane Maarek)
- Linux Academy
- Pluralsight
- WhizLabs (practice exams)
```

---

## 🔗 Documentación y Recursos

### Enlaces Importantes

```
AWS Documentation:
https://docs.aws.amazon.com/

AWS Service Health:
https://health.aws.amazon.com/

AWS Pricing Calculator:
https://calculator.aws/

AWS Architecture Center:
https://aws.amazon.com/architecture/

AWS Whitepapers:
https://aws.amazon.com/whitepapers/
```

### Comunidad

```
AWS re:Post (Q&A):
https://repost.aws/

AWS User Groups:
Local meetups worldwide

AWS re:Invent:
Annual conference (Las Vegas)

AWS Summits:
Regional conferences
```

---

## 📚 Próximos Pasos

Con AWS básico cubierto, profundiza en servicios core:

1. [**EC2**](./ec2.md) → Compute virtual
2. [**VPC**](./vpc.md) → Networking
3. [**S3**](./s3.md) → Object storage
4. [**RDS**](./rds.md) → Databases
5. [**Lambda**](./lambda.md) → Serverless
6. [**IAM**](./iam.md) → Security profundo

---

[⬅️ Volver: Cloud Computing](../README.md) | [➡️ Siguiente: EC2](./ec2.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
