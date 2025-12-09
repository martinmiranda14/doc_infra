# EC2 - Elastic Compute Cloud

## 📖 ¿Qué es EC2?

**Amazon EC2** proporciona capacidad de cómputo escalable en la nube. Servidores virtuales bajo demanda.

```
EC2 = Virtual Machines en AWS
```

**Casos de uso:**
- Hosting de aplicaciones web
- Servidores de aplicaciones
- Servidores de bases de datos
- Ambientes de desarrollo/test
- Procesamiento batch
- Gaming servers

---

## 🖥️ Tipos de Instancias

### Nomenclatura

```
t3.medium
│ │   │
│ │   └─ Tamaño (nano, micro, small, medium, large, xlarge, 2xlarge...)
│ └───── Generación (3 = tercera generación)
└─────── Familia (t = burstable)
```

### Familias de Instancias

#### T - Burstable (General Purpose)

```yaml
t3.micro, t3.small, t3.medium, etc.

Características:
- Performance base + burst capability
- Acumula CPU credits cuando idle
- Usa credits cuando necesita CPU
- Ideal para: Web servers, dev environments

Precio: $0.0104/hora (t3.micro)
```

**Ejemplo de uso:**
```
Aplicación web con tráfico variable:
- 10% CPU uso normal → acumula credits
- Spike de tráfico → usa credits, 100% CPU
- Tráfico baja → vuelve a acumular
```

#### M - General Purpose

```yaml
m5.large, m5.xlarge, m5.2xlarge, etc.

Características:
- Balance de compute, memory, networking
- Performance consistente
- Ideal para: Application servers, mid-size databases

Precio: $0.096/hora (m5.large)
```

#### C - Compute Optimized

```yaml
c5.large, c5.xlarge, c6i.2xlarge, etc.

Características:
- Alto ratio CPU to memory
- Ideal para: Batch processing, HPC, gaming servers

Precio: $0.085/hora (c5.large)
```

#### R - Memory Optimized

```yaml
r5.large, r5.xlarge, r6i.2xlarge, etc.

Características:
- Alto ratio memory to CPU
- Ideal para: Databases, caching, big data analytics

Precio: $0.126/hora (r5.large)
```

#### I - Storage Optimized

```yaml
i3.large, i3.xlarge, i3en.2xlarge, etc.

Características:
- NVMe SSD storage
- Alto IOPS
- Ideal para: NoSQL databases, data warehouses

Precio: $0.156/hora (i3.large)
```

#### G/P - GPU Instances

```yaml
g4dn.xlarge, p3.2xlarge, p4d.24xlarge

Características:
- GPU para ML/AI, rendering
- Ideal para: Deep learning, HPC, graphics

Precio: $0.526/hora (g4dn.xlarge)
```

---

## 💾 Storage Options

### EBS (Elastic Block Store)

Discos persistentes para EC2.

#### Tipos de EBS:

**gp3 (General Purpose SSD) - Recomendado**
```yaml
Características:
- 3,000 IOPS baseline
- 125 MB/s throughput
- Escalable hasta 16,000 IOPS
- Mejor precio/performance

Uso: Boot volumes, apps generales
Precio: $0.08/GB/mes
```

**gp2 (General Purpose SSD) - Legacy**
```yaml
Características:
- 3 IOPS por GB (min 100, max 16,000)
- Throughput depende de tamaño
- Burst capability

Uso: Legacy apps
Precio: $0.10/GB/mes
```

**io2 (Provisioned IOPS SSD)**
```yaml
Características:
- Hasta 64,000 IOPS
- Sub-millisecond latency
- 99.999% durabilidad

Uso: Databases críticos
Precio: $0.125/GB/mes + $0.065/IOPS
```

**st1 (Throughput Optimized HDD)**
```yaml
Características:
- 500 MB/s max throughput
- No puede ser boot volume
- Más barato

Uso: Big data, data warehouses
Precio: $0.045/GB/mes
```

**sc1 (Cold HDD)**
```yaml
Características:
- 250 MB/s max throughput
- Lowest cost
- Acceso infrequ

ente

Uso: Archivos, backups
Precio: $0.015/GB/mes
```

### Instance Store

Storage temporal físicamente attached a servidor físico.

```yaml
Características:
- Incluido en precio de instancia
- SUPER rápido (NVMe)
- ⚠️ EFÍMERO - se pierde al stop/terminate
- No todos los tipos tienen

Uso: Cache, buffers, temp data
```

**⚠️ Importante:**
```
STOP EC2 → Instance Store se PIERDE
Reboot → Instance Store se MANTIENE
```

---

## 🌐 Networking

### Security Groups

Firewall virtual para instancias.

```yaml
Security Group: web-sg

Inbound Rules:
  - Type: HTTP, Port: 80, Source: 0.0.0.0/0
  - Type: HTTPS, Port: 443, Source: 0.0.0.0/0
  - Type: SSH, Port: 22, Source: My IP

Outbound Rules:
  - Type: All, Destination: 0.0.0.0/0
```

**Características:**
- **Stateful:** Return traffic automático
- **Default:** Deny all inbound, allow all outbound
- **Multiple SGs:** Pueden ser attached a una instancia

### Elastic IP

IP pública estática.

```
Default: IP pública cambia al stop/start
Elastic IP: IP permanente asignada a tu cuenta

⚠️ Costo: GRATIS si está attached a running instance
          $0.005/hora si NO está en uso
```

### ENI (Elastic Network Interface)

Network interface virtual.

```yaml
ENI eth0:
  - Private IP: 10.0.1.50
  - Public IP: 54.123.45.67
  - Security Groups: [web-sg]
  - MAC address

Uso:
- Múltiples ENIs por instancia
- Failover: Mover ENI entre instancias
- Dual-homed instances
```

---

## 🚀 Lanzar Instancia EC2

### Via Console

```
1. EC2 Dashboard → Launch Instance

2. Name and tags
   Name: my-web-server

3. Application and OS Image (AMI)
   - Amazon Linux 2023 (Free tier eligible)
   - Or: Ubuntu, Windows, etc.

4. Instance type
   - t3.micro (Free tier)

5. Key pair (login)
   - Create new key pair
   - Name: my-key
   - Type: RSA, .pem
   - Download key (¡ÚNICA VEZ!)

6. Network settings
   - VPC: Default
   - Subnet: Default
   - Auto-assign public IP: Enable
   - Security group:
     ☑ SSH (22) from My IP
     ☑ HTTP (80) from Internet

7. Configure storage
   - 8 GB gp3 (Free tier)

8. Advanced details (optional)
   - User data (bootstrap script)

9. Launch instance
```

### Via CLI

```bash
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0bb1c79de3EXAMPLE \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=my-web-server}]' \
  --user-data file://bootstrap.sh
```

### User Data (Bootstrap)

Script que se ejecuta al lanzar instancia.

```bash
#!/bin/bash
# Instalar Apache web server
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

# Crear página de prueba
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```

---

## 📊 Monitoring

### CloudWatch Metrics

**Métricas básicas (5 min interval, gratis):**
```
- CPUUtilization
- NetworkIn/Out
- DiskReadOps/WriteOps
- StatusCheckFailed
```

**Detailed monitoring (1 min interval, $$$):**
```
- Habilitar en launch o después
- $2.10 per instance/mes adicional
```

**⚠️ No incluido:**
- Memory usage
- Disk usage
- Process monitoring

**Solución:** CloudWatch Agent

```bash
# Instalar CloudWatch Agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm

# Configurar (memory, disk, etc.)
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

---

## 🔄 Auto Scaling

### Launch Template

Define configuración para nuevas instancias.

```yaml
Launch Template: web-server-template

AMI: ami-0c55b159cbfafe1f0
Instance type: t3.micro
Key pair: my-key
Security groups: web-sg
User data: |
  #!/bin/bash
  yum install -y httpd
  systemctl start httpd

IAM instance profile: ec2-web-role
```

### Auto Scaling Group

```yaml
Auto Scaling Group: web-asg

Launch template: web-server-template
VPC: Default
Subnets: us-east-1a, us-east-1b, us-east-1c

Desired capacity: 3
Minimum: 2
Maximum: 10

Health check type: ELB
Health check grace period: 300 seconds

Target tracking policy:
  Metric: Average CPU Utilization
  Target: 70%
  
  # Si CPU > 70%: Scale OUT (agregar instancias)
  # Si CPU < 70%: Scale IN (remover instancias)
```

### Scaling Policies

**Target Tracking:**
```yaml
# Mantener métrica en valor target
Target: CPU 70%
AWS auto-adjust capacity
```

**Step Scaling:**
```yaml
# Escalar basado en thresholds
Alarm: CPU > 80% → +2 instancias
Alarm: CPU > 90% → +4 instancias
```

**Scheduled Scaling:**
```yaml
# Escalar en horarios específicos
Mon-Fri 8am: Min=10, Max=50, Desired=10
Mon-Fri 6pm: Min=2, Max=10, Desired=2
```

---

## ⚖️ Load Balancing

### Application Load Balancer (ALB)

Layer 7 (HTTP/HTTPS).

```yaml
ALB: my-web-alb

Listeners:
  - HTTP:80 → Forward to target-group-web
  - HTTPS:443 → Forward to target-group-web

Target Group: target-group-web
  Protocol: HTTP
  Port: 80
  Health check:
    Path: /health
    Interval: 30s
    Timeout: 5s
    Healthy threshold: 2
    Unhealthy threshold: 5

Registered targets:
  - i-0123456789abcdef0 (healthy)
  - i-0123456789abcdef1 (healthy)
  - i-0123456789abcdef2 (healthy)
```

**Path-based routing:**
```yaml
Listener rules:
  - /api/* → target-group-api
  - /images/* → target-group-images
  - /* → target-group-web
```

### Network Load Balancer (NLB)

Layer 4 (TCP/UDP), ultra-high performance.

```yaml
NLB: my-tcp-nlb

Listeners:
  - TCP:3306 → Forward to target-group-database

Characteristics:
- Millions of requests/second
- Static IP per AZ
- Preserve source IP
- Extreme low latency
```

---

## 💰 Pricing

### On-Demand

```
t3.micro: $0.0104/hora = $7.49/mes
t3.medium: $0.0416/hora = $29.97/mes
m5.large: $0.096/hora = $69.12/mes

Ventaja: Flexibilidad
Uso: Dev/test, apps impredecibles
```

### Reserved Instances (RI)

```
Comprometer 1-3 años:
- 1 año: 30-40% descuento
- 3 años: 50-60% descuento

t3.micro RI (1 año):
$0.0062/hora = $4.47/mes (ahorro 40%)

Tipos:
- Standard: No cambios
- Convertible: Cambiar familia/size
```

### Savings Plans

```
Commit $X/hora por 1-3 años
Flexible:
- Cambiar instance type
- Cambiar region
- Aplica a Lambda, Fargate también
```

### Spot Instances

```
Capacidad no usada:
- Hasta 90% descuento
- Puede ser terminado con 2 min warning
- Ideal para: Batch, big data, stateless

t3.micro Spot: $0.0031/hora (70% descuento)

Spot Fleet: Mix de spot + on-demand
```

---

## 📋 Best Practices

### Seguridad

```
✅ Security groups: Least privilege
✅ Usar IAM roles, NO access keys
✅ SSH key pairs seguros
✅ Patch management (Systems Manager)
✅ Encryption at rest (EBS encrypted)
✅ VPC con subnets privadas
```

### Alta Disponibilidad

```
✅ Multi-AZ deployment
✅ Auto Scaling Groups
✅ Load Balancer health checks
✅ EBS snapshots automated
✅ AMI backups
```

### Cost Optimization

```
✅ Right-sizing (CloudWatch metrics)
✅ Stop instances no usadas
✅ Reserved Instances para predictable workloads
✅ Spot instances para fault-tolerant
✅ Auto Scaling para match demand
```

---

## 🐛 Troubleshooting

### Cannot Connect via SSH

```bash
# 1. Check security group
# Debe permitir SSH (22) from your IP

# 2. Check public IP exists
# Instance must have public IP

# 3. Check key permissions
chmod 400 my-key.pem

# 4. Correct username
ssh -i my-key.pem ec2-user@54.123.45.67
# Amazon Linux: ec2-user
# Ubuntu: ubuntu
# RHEL: ec2-user or root

# 5. Check Network ACLs (si existe VPC custom)
```

### Instance Status Check Failed

```
Status check 1/2 failed: Instance status check
→ Problema con instancia (OS, kernel, etc.)
→ Solución: Stop y start (migra a nuevo hardware)

Status check 2/2 failed: System status check
→ Problema con hardware subyacente
→ Solución: Stop y start (migra a nuevo hardware)
```

### High CPU Usage

```bash
# 1. Identificar proceso
top
# or
htop

# 2. CloudWatch alarm
# CPU > 80% → Notification

# 3. Scale out (más instancias)
# or
# Scale up (instance type más grande)
```

---

## 📚 Próximos Pasos

Con EC2 dominado:

1. [**VPC**](./vpc.md) → Networking en profundidad
2. [**S3**](./s3.md) → Object storage
3. [**RDS**](./rds.md) → Managed databases
4. [**Lambda**](./lambda.md) → Serverless compute

---

[⬅️ Volver: AWS Intro](./introduccion.md) | [➡️ Siguiente: VPC](./vpc.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
