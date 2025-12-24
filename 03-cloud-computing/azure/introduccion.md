# Introducción a Microsoft Azure

## 📖 ¿Qué es Azure?

**Microsoft Azure** es la plataforma cloud de Microsoft que ofrece más de 200 servicios para crear, desplegar y gestionar aplicaciones.

```
Azure = Microsoft Cloud Platform
- Segunda plataforma más grande (23% cuota mercado)
- Fuerte integración con ecosistema Microsoft
- Hybrid cloud robusto
- Enterprise-focused
```

**Historia:**
```
2010: Launch (como Windows Azure)
2014: Renombrado a Microsoft Azure
2020: $50B revenue
2024: 60+ regiones globally
```

---

## 🌍 Infraestructura Global

### Geografías y Regiones

```yaml
Geografía: Mercado que contiene múltiples regiones
  - Americas
  - Europe
  - Asia Pacific
  - Middle East and Africa

Regiones: 60+ worldwide
  - East US, East US 2
  - West Europe, North Europe
  - Southeast Asia
  - Brazil South
  - UAE North

Regiones especiales:
  - Government (US Gov, China)
  - Sovereign clouds
```

**Elegir región:**
```yaml
Criterios:
1. Proximity to users (latency)
2. Data residency/compliance (GDPR)
3. Service availability
4. Pricing (varies by region)
5. Availability Zones support
```

### Availability Zones

```yaml
Availability Zone (AZ):
  - Physically separate datacenters
  - Within same region
  - Independent power, cooling, networking
  - <2ms latency between AZs

Regions with AZs: 30+
Each region: Minimum 3 AZs

Use case:
  - High availability
  - Disaster recovery
  - Zone-redundant services
```

```
┌─────────────────────────────────────┐
│         Region (East US)            │
│                                     │
│  ┌──────────┐  ┌──────────┐       │
│  │   AZ 1   │  │   AZ 2   │       │
│  │          │  │          │       │
│  └──────────┘  └──────────┘       │
│                                     │
│  ┌──────────┐                      │
│  │   AZ 3   │                      │
│  │          │                      │
│  └──────────┘                      │
└─────────────────────────────────────┘
```

### Paired Regions

```yaml
Region Pairs:
  - East US ↔ West US
  - North Europe ↔ West Europe
  - Southeast Asia ↔ East Asia

Benefits:
  - Disaster recovery
  - Sequential updates (one at a time)
  - Data residency (within same geography)
  - Replication options (GRS)
```

---

## 🏢 Jerarquía de Azure

### Management Structure

```
Azure AD Tenant (Entra ID)
    │
    └─── Management Groups
            │
            └─── Subscriptions
                    │
                    └─── Resource Groups
                            │
                            └─── Resources
```

#### Azure AD Tenant

```yaml
Tenant: Instancia dedicada de Azure Active Directory

Características:
  - Único por organización
  - Global identity service
  - Gestión de usuarios y grupos
  - SSO, MFA, Conditional Access

Example:
  company.onmicrosoft.com
```

#### Management Groups

```yaml
Management Groups: Container para subscriptions

Hierarchy:
  Root Management Group
  ├─ Production MG
  │  ├─ Subscription-Prod-Web
  │  └─ Subscription-Prod-DB
  └─ Development MG
     ├─ Subscription-Dev
     └─ Subscription-Test

Use case:
  - Policies at scale
  - RBAC inheritance
  - Cost management
```

#### Subscriptions

```yaml
Subscription: Billing boundary

Types:
  - Free (12 months + credits)
  - Pay-As-You-Go
  - Enterprise Agreement (EA)
  - CSP (Cloud Solution Provider)

Limits:
  - 10,000 resource groups per subscription
  - Quota limits (VMs, storage accounts, etc.)

Use case:
  - Separate billing
  - Environment separation
  - Department separation
```

#### Resource Groups

```yaml
Resource Group: Logical container for resources

Characteristics:
  - All resources must be in a resource group
  - One resource = one resource group
  - Can move resources between groups
  - Delete group = delete all resources

Best practices:
  - Group by lifecycle (app tier, project)
  - Same region (recommended)
  - RBAC at RG level
```

```yaml
Example:
Resource Group: production-web-rg
  Region: East US
  Resources:
    - VM: web-vm-01
    - Disk: web-vm-01_OsDisk
    - NIC: web-vm-01-nic
    - Public IP: web-vm-01-ip
    - NSG: web-nsg
    - Load Balancer: web-lb
```

---

## 🎯 Servicios Principales

### Compute

**Virtual Machines**
```yaml
- VMs en cloud
- Windows, Linux
- Series: B, D, E, F, etc.
- Similar a EC2
```

**App Service**
```yaml
- PaaS para web apps
- .NET, Java, Node.js, Python
- Auto-scaling
- Similar a Elastic Beanstalk
```

**Azure Kubernetes Service (AKS)**
```yaml
- Managed Kubernetes
- Container orchestration
- Similar a EKS/GKE
```

**Azure Functions**
```yaml
- Serverless compute
- Event-driven
- Similar a Lambda
```

### Storage

**Blob Storage**
```yaml
- Object storage
- Hot, Cool, Archive tiers
- Similar a S3
```

**Disk Storage**
```yaml
- Managed disks for VMs
- SSD, HDD
- Similar a EBS
```

**File Storage**
```yaml
- SMB file shares
- Cloud file server
- Similar a EFS
```

### Networking

**Virtual Network (VNet)**
```yaml
- Private network
- Subnets, NSGs
- Similar a VPC
```

**Load Balancer**
```yaml
- Layer 4 load balancing
- Public, Internal
- Similar a ELB/NLB
```

**Application Gateway**
```yaml
- Layer 7 load balancer
- WAF capable
- Similar a ALB
```

### Databases

**Azure SQL Database**
```yaml
- Managed SQL Server
- Serverless option
- Similar a RDS SQL Server
```

**Cosmos DB**
```yaml
- NoSQL multi-model
- Global distribution
- Similar a DynamoDB
```

**Azure Database for MySQL/PostgreSQL**
```yaml
- Managed MySQL/PostgreSQL
- Flexible Server
- Similar a RDS
```

---

## 🔐 Azure Active Directory (Entra ID)

### Identity Management

```yaml
Azure AD (Entra ID):
  - Cloud-based identity service
  - Authentication y authorization
  - SSO support
  - MFA built-in

Users:
  - Cloud-only identities
  - Synced from on-premise AD
  - External identities (B2B, B2C)

Groups:
  - Security groups
  - Microsoft 365 groups
  - Dynamic membership
```

### RBAC (Role-Based Access Control)

```yaml
RBAC Components:
  - Security Principal: Who (user, group, service principal)
  - Role Definition: What (Owner, Contributor, Reader)
  - Scope: Where (subscription, RG, resource)

Built-in Roles:
  Owner:
    - Full access including RBAC
  
  Contributor:
    - Manage resources
    - Cannot grant access
  
  Reader:
    - View only
  
  Custom Roles:
    - Define your own
```

**Example:**
```yaml
Assignment:
  Principal: john@company.com
  Role: Contributor
  Scope: /subscriptions/abc-123/resourceGroups/production-rg
  
Effect: John can manage resources in production-rg
```

---

## 💰 Pricing

### Pricing Models

**Pay-As-You-Go**
```yaml
- Per second billing (VMs)
- Per GB (Storage)
- Per transaction (Functions)

Advantages: Flexibility
Disadvantages: Higher cost
```

**Reserved Instances**
```yaml
Term: 1 or 3 years
Discount: 30-70%
Flexibility: Instance size flexible

Payment options:
  - Upfront
  - Monthly
  - No upfront (3 year only)
```

**Spot VMs**
```yaml
Discount: Up to 90%
Risk: Can be evicted
Use case: Batch, dev/test
```

**Azure Hybrid Benefit**
```yaml
Bring your own license:
  - Windows Server
  - SQL Server
  
Savings: Up to 85%

Requirements:
  - Active Software Assurance
  - Or subscription licenses
```

### Free Tier

**Always Free:**
```yaml
- App Service: 10 web apps (F1 tier)
- Functions: 1M executions/month
- Cosmos DB: 25 GB + 1000 RU/s
- Blob Storage: 5 GB LRS
- Bandwidth: 15 GB outbound/month
```

**12 Months Free:**
```yaml
- VMs: 750 hours/month B1S
- Managed Disks: 64 GB × 2 (P6)
- File Storage: 5 GB
- SQL Database: 250 GB
- Bandwidth: 15 GB/month
```

---

## 🖥️ Herramientas de Gestión

### Azure Portal

```yaml
URL: https://portal.azure.com

Features:
  - GUI web-based
  - Dashboards personalizables
  - Cloud Shell integrado
  - Mobile app disponible
```

### Azure CLI

```bash
# Instalación
# macOS
brew install azure-cli

# Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Windows
winget install -e --id Microsoft.AzureCLI

# Login
az login

# Listar subscriptions
az account list --output table

# Set default subscription
az account set --subscription "My Subscription"

# Ejemplos
az group list
az vm list
az storage account list
```

### Azure PowerShell

```powershell
# Instalación
Install-Module -Name Az -AllowClobber

# Login
Connect-AzAccount

# Ejemplos
Get-AzResourceGroup
Get-AzVM
New-AzResourceGroup -Name "MyRG" -Location "EastUS"
```

### Azure Cloud Shell

```yaml
Cloud Shell:
  - Browser-based shell
  - Bash o PowerShell
  - Pre-configured tools
  - 5 GB persistent storage

Access:
  - Portal (top bar)
  - shell.azure.com
  - Mobile app
```

### ARM Templates

```yaml
ARM (Azure Resource Manager):
  - Infrastructure as Code
  - JSON format
  - Declarative
  - Idempotent

Similar a: CloudFormation (AWS)
```

**Example:**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "mystorageaccount",
      "location": "eastus",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
    }
  ]
}
```

### Bicep

```yaml
Bicep: DSL for ARM templates
  - More concise than JSON
  - Better dev experience
  - Transpiles to ARM JSON
```

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: 'mystorageaccount'
  location: 'eastus'
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

---

## 🚀 Primeros Pasos

### Crear Cuenta

```
1. https://azure.microsoft.com/free/
2. Registrar con email
3. Verificar identidad (teléfono, tarjeta)
4. $200 credit por 30 días
5. 12 meses de servicios gratuitos
```

### Configurar Entorno

```bash
# 1. Install Azure CLI
brew install azure-cli

# 2. Login
az login

# 3. Ver subscriptions
az account list --output table

# 4. Set default
az account set --subscription "Free Trial"

# 5. Crear primer resource group
az group create \
  --name myfirst-rg \
  --location eastus

# 6. Ver recursos
az resource list --resource-group myfirst-rg
```

### Crear Primera VM

```bash
# Via CLI
az vm create \
  --resource-group myfirst-rg \
  --name myVM \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys

# Output includes Public IP
# SSH: ssh azureuser@<PublicIP>
```

---

## 📊 Monitoring y Governance

### Azure Monitor

```yaml
Azure Monitor:
  - Metrics y logs
  - Dashboards
  - Alerts
  - Application Insights (APM)

Integrated with:
  - All Azure services
  - On-premise (agents)
  - Multi-cloud
```

### Azure Policy

```yaml
Azure Policy:
  - Enforce compliance
  - Standards y regulations
  - Remediation

Examples:
  - Require tags on resources
  - Allowed VM sizes
  - Allowed regions
  - Require encryption
```

### Cost Management

```yaml
Cost Management + Billing:
  - Cost analysis
  - Budgets y alerts
  - Recommendations
  - Export data

Features:
  - Cost by resource group
  - Cost by tag
  - Forecast
  - Advisor recommendations
```

---

## 🏆 Fortalezas de Azure

```yaml
✅ Hybrid Cloud:
   - Azure Arc
   - Azure Stack
   - Seamless on-premise integration

✅ Microsoft Ecosystem:
   - Active Directory integration
   - Office 365
   - Dynamics 365
   - Windows Server/SQL Server

✅ Enterprise:
   - Enterprise agreements
   - Compliance (80+ certifications)
   - Support options

✅ AI/ML:
   - Azure OpenAI Service
   - Machine Learning
   - Cognitive Services

✅ DevOps:
   - Azure DevOps
   - GitHub integration
   - CI/CD built-in
```

---

## 📋 Comparación AWS vs Azure

| Feature | AWS | Azure |
|---------|-----|-------|
| **Compute** | EC2 | Virtual Machines |
| **Object Storage** | S3 | Blob Storage |
| **Block Storage** | EBS | Managed Disks |
| **File Storage** | EFS | Azure Files |
| **Networking** | VPC | Virtual Network |
| **Load Balancer** | ELB/ALB | Load Balancer/App Gateway |
| **SQL Database** | RDS | Azure SQL Database |
| **NoSQL** | DynamoDB | Cosmos DB |
| **Serverless** | Lambda | Azure Functions |
| **Containers** | ECS/EKS | ACI/AKS |
| **IaC** | CloudFormation | ARM Templates/Bicep |
| **IAM** | IAM | Azure AD + RBAC |

---

## 🎓 Certificaciones

### Fundamentals
```
AZ-900: Azure Fundamentals
  - Entry level
  - Cloud concepts
  - Azure services
```

### Associate
```
AZ-104: Azure Administrator
  - Manage subscriptions
  - Implement storage
  - Deploy VMs
  - Configure networking

AZ-204: Azure Developer
  - Develop Azure compute
  - Azure storage
  - Security
  - Monitoring

AZ-305: Azure Solutions Architect Expert (requires AZ-104)
  - Design infrastructure
  - Design security
  - Design data storage
```

### Specialty
```
AZ-500: Security
AZ-700: Networking
AZ-400: DevOps
DP-203: Data Engineering
AI-102: AI Engineer
```

---

## 📚 Próximos Pasos

Con Azure introducción cubierto:

1. [**Virtual Machines**](./virtual-machines.md) → Compute
2. [**Storage**](./storage.md) → Blob, File, Disk
3. [**Networking**](./networking.md) → VNet, NSG, LB
4. [**Azure AD**](./azure-ad.md) → Identity management

---

[⬅️ Volver: Cloud Computing](../README.md) | [➡️ Siguiente: Virtual Machines](./virtual-machines.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
