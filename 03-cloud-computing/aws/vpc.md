# VPC - Virtual Private Cloud

## 📖 ¿Qué es VPC?

**Amazon VPC** permite lanzar recursos de AWS en una red virtual aislada lógicamente que tú defines.

```
VPC = Tu propia red privada en AWS
- Control completo de networking
- Aislamiento de recursos
- Conectividad a internet controlada
- Conexión segura a on-premise
```

**Analogía:**
```
VPC es como tu propia oficina en un edificio (AWS Region)
- Defines los pisos y salas (subnets)
- Controlas quién entra/sale (security groups, NACLs)
- Decides si conectar con otras oficinas (peering)
```

---

## 🏗️ Componentes de VPC

### VPC

Rango de direcciones IP privadas para tus recursos.

```yaml
VPC: my-app-vpc
CIDR Block: 10.0.0.0/16
  → Rango: 10.0.0.0 - 10.0.255.255
  → IPs disponibles: 65,536

Region: us-east-1
Tenancy: default (shared hardware)
```

**CIDR Blocks permitidos:**
```
/16 (más grande):  65,536 IPs
/17:               32,768 IPs
/18:               16,384 IPs
...
/28 (más pequeño): 16 IPs
```

### Subnets

Subdivisiones del VPC en Availability Zones.

```yaml
VPC: 10.0.0.0/16

Subnets:
  # Public subnets (tiene ruta a Internet Gateway)
  - Name: public-subnet-1a
    CIDR: 10.0.1.0/24
    AZ: us-east-1a
    Available IPs: 251 (256 - 5 reserved)
  
  - Name: public-subnet-1b
    CIDR: 10.0.2.0/24
    AZ: us-east-1b
    Available IPs: 251
  
  # Private subnets (sin ruta directa a internet)
  - Name: private-subnet-1a
    CIDR: 10.0.11.0/24
    AZ: us-east-1a
  
  - Name: private-subnet-1b
    CIDR: 10.0.12.0/24
    AZ: us-east-1b
```

**AWS reserva 5 IPs por subnet:**
```
Subnet: 10.0.1.0/24

Reserved:
10.0.1.0   → Network address
10.0.1.1   → VPC router
10.0.1.2   → DNS server
10.0.1.3   → Reserved (future use)
10.0.1.255 → Broadcast (no soportado pero reservado)

Usables: 10.0.1.4 - 10.0.1.254 (251 IPs)
```

---

## 🌐 Conectividad a Internet

### Internet Gateway (IGW)

Permite acceso a internet desde VPC.

```yaml
Internet Gateway: igw-abc123
Attached to: my-app-vpc

Función:
- NAT para instancias con IP pública
- 1:1 mapping entre private IP y public IP
- Highly available (AWS managed)
```

**Arquitectura:**
```
Internet
   ↕
Internet Gateway (IGW)
   ↕
VPC Route Table
   ↕
Public Subnet → EC2 instances (con public IP)
```

**Route table para subnet pública:**
```yaml
Route Table: public-rt

Routes:
  - Destination: 10.0.0.0/16
    Target: local  # Tráfico interno VPC
  
  - Destination: 0.0.0.0/0
    Target: igw-abc123  # Todo lo demás → Internet
```

### NAT Gateway

Permite que instancias en subnets privadas accedan a internet (outbound only).

```yaml
NAT Gateway: nat-xyz789
Subnet: public-subnet-1a (debe estar en subnet pública)
Elastic IP: 54.123.45.67

Características:
- Managed by AWS
- Highly available en AZ
- Scales automáticamente hasta 45 Gbps
- $0.045/hora + $0.045/GB procesado
```

**Arquitectura:**
```
Internet
   ↕
Internet Gateway
   ↕
Public Subnet
   ↕
NAT Gateway (con Elastic IP)
   ↕
Private Subnet → EC2 instances (sin public IP)
```

**Route table para subnet privada:**
```yaml
Route Table: private-rt

Routes:
  - Destination: 10.0.0.0/16
    Target: local
  
  - Destination: 0.0.0.0/0
    Target: nat-xyz789  # Internet via NAT Gateway
```

**Alta Disponibilidad:**
```yaml
# NAT Gateway por AZ para HA
NAT-1a in public-subnet-1a → Para private-subnet-1a
NAT-1b in public-subnet-1b → Para private-subnet-1b
NAT-1c in public-subnet-1c → Para private-subnet-1c

Costo: 3 × $0.045/hora = $97.20/mes
```

### NAT Instance (alternativa legacy)

```yaml
NAT Instance: EC2 instance configurado para NAT

Ventajas:
- Más barato (solo costo EC2)
- Más control

Desventajas:
- Debes gestionar (patching, scaling)
- Single point of failure
- Performance limitado por instance type

⚠️ AWS recomienda NAT Gateway
```

---

## 🔒 Seguridad

### Security Groups

Firewall stateful a nivel de instancia (ENI).

```yaml
Security Group: web-sg
VPC: my-app-vpc

Inbound Rules:
  - Type: HTTP
    Protocol: TCP
    Port: 80
    Source: 0.0.0.0/0
    Description: Allow HTTP from internet
  
  - Type: HTTPS
    Protocol: TCP
    Port: 443
    Source: 0.0.0.0/0
    Description: Allow HTTPS from internet
  
  - Type: SSH
    Protocol: TCP
    Port: 22
    Source: 10.0.0.0/16
    Description: SSH from within VPC only

Outbound Rules:
  - Type: All traffic
    Protocol: All
    Port: All
    Destination: 0.0.0.0/0
    Description: Allow all outbound
```

**Características:**
- **Stateful:** Return traffic automático
- **Default:** Deny all inbound, allow all outbound
- **Attachment:** Asociado a ENI
- **Multiple:** Múltiples SGs por instancia (rules OR'd)

**Referenciar otros Security Groups:**
```yaml
Security Group: app-sg

Inbound Rules:
  - Type: Custom TCP
    Port: 8080
    Source: sg-web123  # Solo desde web-sg
    
Security Group: db-sg

Inbound Rules:
  - Type: MySQL/Aurora
    Port: 3306
    Source: sg-app456  # Solo desde app-sg
```

### Network ACLs (NACLs)

Firewall stateless a nivel de subnet.

```yaml
Network ACL: public-nacl
Subnets: public-subnet-1a, public-subnet-1b

Inbound Rules:
  Rule # | Type  | Protocol | Port | Source      | Allow/Deny
  100    | HTTP  | TCP      | 80   | 0.0.0.0/0   | ALLOW
  110    | HTTPS | TCP      | 443  | 0.0.0.0/0   | ALLOW
  120    | SSH   | TCP      | 22   | 1.2.3.4/32  | ALLOW
  *      | All   | All      | All  | 0.0.0.0/0   | DENY

Outbound Rules:
  Rule # | Type  | Protocol | Port     | Dest        | Allow/Deny
  100    | HTTP  | TCP      | 80       | 0.0.0.0/0   | ALLOW
  110    | HTTPS | TCP      | 443      | 0.0.0.0/0   | ALLOW
  120    | Custom| TCP      | 1024-65535| 0.0.0.0/0  | ALLOW (ephemeral)
  *      | All   | All      | All      | 0.0.0.0/0   | DENY
```

**Características:**
- **Stateless:** Debes permitir return traffic explícitamente
- **Default:** Default NACL permite todo
- **Custom:** Custom NACL niega todo por default
- **Rules:** Evaluadas en orden numérico
- **Ephemeral ports:** 1024-65535 para return traffic

**Security Groups vs NACLs:**
```
┌─────────────────┬──────────────────┬──────────────────┐
│                 │ Security Groups  │ Network ACLs     │
├─────────────────┼──────────────────┼──────────────────┤
│ Level           │ Instance (ENI)   │ Subnet           │
│ State           │ Stateful         │ Stateless        │
│ Rules           │ Allow only       │ Allow & Deny     │
│ Evaluation      │ All rules        │ Ordered (#)      │
│ Default         │ Deny all inbound │ Allow all        │
│ Use case        │ Instance level   │ Subnet level     │
└─────────────────┴──────────────────┴──────────────────┘
```

---

## 🔗 VPC Connectivity

### VPC Peering

Conexión privada entre dos VPCs.

```yaml
VPC Peering: pcx-abc123

Requester VPC: vpc-111 (10.0.0.0/16)
Accepter VPC: vpc-222 (172.16.0.0/16)

Status: Active

Características:
- No overlapping CIDR blocks
- Not transitive (A↔B, B↔C ≠ A↔C)
- Cross-region supported
- Cross-account supported
- No bandwidth bottleneck
- Usa AWS backbone network
```

**Route tables con peering:**
```yaml
# VPC-A Route Table
Routes:
  - 10.0.0.0/16 → local
  - 172.16.0.0/16 → pcx-abc123

# VPC-B Route Table
Routes:
  - 172.16.0.0/16 → local
  - 10.0.0.0/16 → pcx-abc123
```

### Transit Gateway

Hub central para conectar múltiples VPCs y on-premise.

```yaml
Transit Gateway: tgw-xyz789

Attachments:
  - VPC-A (10.0.0.0/16)
  - VPC-B (172.16.0.0/16)
  - VPC-C (192.168.0.0/16)
  - VPN-Connection
  - Direct-Connect

Routing: Simple hub-and-spoke
Max throughput: 50 Gbps per VPC attachment
```

**Ventajas sobre VPC Peering:**
```
VPC Peering:
  3 VPCs → 3 connections (full mesh)
  10 VPCs → 45 connections
  
Transit Gateway:
  N VPCs → N connections (hub-and-spoke)
  10 VPCs → 10 connections
  
✅ Escalabilidad
✅ Gestión simplificada
✅ Transitive routing
```

---

## 🏢 Hybrid Connectivity

### VPN (Virtual Private Network)

Conexión encriptada sobre internet.

```yaml
Site-to-Site VPN

AWS Side:
  - Virtual Private Gateway (VGW)
    Attached to: VPC
  
On-Premise Side:
  - Customer Gateway (CGW)
    Public IP: 203.0.113.5
    Device: Cisco ASA / Fortinet / etc.

VPN Connection:
  - 2 tunnels (redundancy)
  - IPsec encryption
  - BGP or static routing

Throughput: Up to 1.25 Gbps per tunnel
Latency: Variable (internet-based)
Cost: $0.05/hora + data transfer
```

**Arquitectura:**
```
On-Premise Datacenter
   ↕ (IPsec Tunnel 1)
Customer Gateway
   ↕ (Internet)
Virtual Private Gateway
   ↕
VPC
```

### Direct Connect

Conexión dedicada física.

```yaml
AWS Direct Connect

Connection:
  - Dedicated line (no internet)
  - 1 Gbps or 10 Gbps
  - Private connection

Location:
  - AWS Direct Connect location
  - Colocation facility

Virtual Interfaces (VIF):
  - Private VIF → VPC
  - Public VIF → AWS public services (S3, DynamoDB)
  - Transit VIF → Transit Gateway

Benefits:
✅ Consistent network performance
✅ Lower bandwidth costs
✅ Private connection (no internet)
✅ Supports 802.1q VLANs

Cost: Port-hour fee + data transfer out
Setup time: 1-2 months
```

**Direct Connect vs VPN:**
```
┌──────────────┬──────────────┬──────────────┐
│              │ VPN          │ Direct Connect│
├──────────────┼──────────────┼──────────────┤
│ Setup        │ Hours        │ Weeks/months │
│ Speed        │ <1.25 Gbps   │ 1-10 Gbps    │
│ Cost         │ $            │ $$$          │
│ Latency      │ Variable     │ Consistent   │
│ Transport    │ Internet     │ Private      │
│ Encryption   │ IPsec (built)│ MACsec (opt) │
└──────────────┴──────────────┴──────────────┘
```

---

## 📊 VPC Flow Logs

Captura información sobre tráfico IP.

```yaml
Flow Log: vpc-flowlogs

Target:
  - VPC: my-app-vpc (todos los ENIs)
  - Or Subnet: specific subnet
  - Or ENI: specific instance

Destination:
  - CloudWatch Logs
  - S3 Bucket

Traffic: All | Accepted | Rejected

Format:
<version> <account-id> <interface-id> <srcaddr> <dstaddr> 
<srcport> <dstport> <protocol> <packets> <bytes> 
<start> <end> <action> <log-status>
```

**Ejemplo de log:**
```
2 123456789 eni-abc123 10.0.1.5 172.217.167.46 
49152 443 6 20 4000 
1620000000 1620000060 ACCEPT OK

Interpretación:
- Source: 10.0.1.5:49152
- Destination: 172.217.167.46:443 (HTTPS)
- Protocol: 6 (TCP)
- Action: ACCEPT
- 20 packets, 4000 bytes transferred
```

**Casos de uso:**
```
✅ Troubleshooting connectivity
✅ Security analysis
✅ Compliance auditing
✅ Cost optimization (identify high traffic)
```

---

## 🎯 Arquitectura Típica: 3-Tier

```yaml
VPC: production-vpc (10.0.0.0/16)

# Public Tier - Load Balancers
Public Subnets:
  - 10.0.1.0/24 (us-east-1a)
  - 10.0.2.0/24 (us-east-1b)
  
  Resources:
    - ALB
    - NAT Gateways
    - Bastion hosts

# Application Tier
Private Subnets (App):
  - 10.0.11.0/24 (us-east-1a)
  - 10.0.12.0/24 (us-east-1b)
  
  Resources:
    - EC2 app servers
    - Auto Scaling Group

# Database Tier
Private Subnets (DB):
  - 10.0.21.0/24 (us-east-1a)
  - 10.0.22.0/24 (us-east-1b)
  
  Resources:
    - RDS Multi-AZ
    - ElastiCache
```

**Security Groups:**
```yaml
# ALB Security Group
alb-sg:
  Inbound: 80, 443 from 0.0.0.0/0

# App Security Group
app-sg:
  Inbound: 8080 from alb-sg
  Outbound: 3306 to db-sg, 443 to 0.0.0.0/0

# DB Security Group
db-sg:
  Inbound: 3306 from app-sg only
  Outbound: None
```

---

## 💰 Pricing

```yaml
VPC: Free
Subnets: Free
Route Tables: Free
Internet Gateway: Free
Security Groups: Free
NACLs: Free

NAT Gateway:
  - $0.045/hora = $32.40/mes
  - $0.045/GB procesado

VPN Connection:
  - $0.05/hora = $36/mes
  - Data transfer out: estándar

VPC Peering:
  - No hourly charge
  - Data transfer: $0.01-0.02/GB

Transit Gateway:
  - $0.05/hora per attachment
  - $0.02/GB procesado

Direct Connect:
  - Port fee: $0.30/hora (1 Gbps)
  - Data transfer out: reducido

VPC Endpoints (Interface):
  - $0.01/hora per AZ
  - $0.01/GB procesado
```

---

## 📋 Best Practices

### Planning

```yaml
✅ Plan CIDR blocks cuidadosamente (no se pueden cambiar)
✅ Usar RFC 1918 address space:
   - 10.0.0.0/8
   - 172.16.0.0/12
   - 192.168.0.0/16
✅ Dejar espacio para crecimiento
✅ Avoid overlap con on-premise networks
✅ Multi-AZ deployment
```

### Security

```yaml
✅ Principle of least privilege (Security Groups)
✅ Private subnets para app/db tiers
✅ NAT Gateway en public subnet
✅ Bastion host en public subnet (con SG restrictivo)
✅ VPC Flow Logs enabled
✅ NACLs como segunda capa de defensa
✅ No default VPC en producción
```

### High Availability

```yaml
✅ Multi-AZ (mínimo 2 AZs)
✅ NAT Gateway por AZ
✅ Distribute resources across AZs
✅ Use ELB health checks
```

### Cost Optimization

```yaml
✅ VPC Endpoints para S3/DynamoDB (evitar NAT Gateway)
✅ Single NAT Gateway para dev/test
✅ Right-size Direct Connect
✅ Monitor data transfer (CloudWatch)
```

---

## 🐛 Troubleshooting

### Cannot Connect to Instance in Public Subnet

```bash
Checklist:
☑ Instance has public IP?
☑ Security Group allows port (22/3389)?
☑ Network ACL allows port?
☑ Route table has route to IGW (0.0.0.0/0 → igw-xxx)?
☑ IGW attached to VPC?
☑ Subnet auto-assign public IP enabled?
```

### Instance in Private Subnet Can't Access Internet

```bash
Checklist:
☑ NAT Gateway exists in public subnet?
☑ NAT Gateway has Elastic IP?
☑ Private subnet route table has route (0.0.0.0/0 → nat-xxx)?
☑ Security Group allows outbound?
☑ NACL allows outbound + return traffic?
```

### VPC Peering Not Working

```bash
Checklist:
☑ Peering connection status = Active?
☑ Route tables updated in BOTH VPCs?
☑ No overlapping CIDR blocks?
☑ Security Groups allow traffic from peer VPC?
☑ NACLs allow traffic?
```

---

## 🔧 Comandos CLI Útiles

```bash
# Crear VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Crear subnet
aws ec2 create-subnet \
  --vpc-id vpc-abc123 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Crear Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway \
  --vpc-id vpc-abc123 \
  --internet-gateway-id igw-xyz

# Crear NAT Gateway
aws ec2 create-nat-gateway \
  --subnet-id subnet-public \
  --allocation-id eipalloc-xyz

# Route table
aws ec2 create-route-table --vpc-id vpc-abc123
aws ec2 create-route \
  --route-table-id rtb-abc \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xyz

# Security Group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web servers" \
  --vpc-id vpc-abc123

aws ec2 authorize-security-group-ingress \
  --group-id sg-abc123 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-abc123 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs
```

---

## 📚 Próximos Pasos

Con VPC dominado:

1. [**RDS**](./rds.md) → Bases de datos en VPC
2. [**Lambda**](./lambda.md) → Serverless en VPC
3. [**IAM**](./iam.md) → Security profundo
4. **AWS Transit Gateway** → Advanced networking

---

[⬅️ Volver: EC2](./ec2.md) | [➡️ Siguiente: RDS](./rds.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
