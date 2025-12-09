# RDS - Relational Database Service

## 📖 ¿Qué es RDS?

**Amazon RDS** es un servicio de bases de datos relacionales gestionado que facilita configurar, operar y escalar bases de datos en la nube.

```
RDS = Managed Relational Databases
- AWS gestiona: Hardware, OS, patching, backups
- Tú gestionas: Schema, queries, tuning
```

**Ventajas sobre self-managed DB en EC2:**
```
✅ Automated backups
✅ Automated patching
✅ Multi-AZ para HA
✅ Read Replicas para scaling
✅ Monitoring integrado
✅ Simplified management

❌ Menos control (no root access)
❌ No todas las extensiones disponibles
```

---

## 🗄️ Engines Soportados

### MySQL

```yaml
Engine: MySQL
Versions: 5.7, 8.0
Max storage: 64 TB
Max connections: Depends on instance type

Use cases:
- Web applications
- E-commerce
- LAMP stack
```

### PostgreSQL

```yaml
Engine: PostgreSQL
Versions: 11, 12, 13, 14, 15
Max storage: 64 TB
Extensions: PostGIS, pg_stat_statements, etc.

Use cases:
- Complex queries
- Geospatial data
- JSON workloads
```

### MariaDB

```yaml
Engine: MariaDB
Versions: 10.3, 10.4, 10.5, 10.6
Max storage: 64 TB
Compatible: MySQL (drop-in replacement)

Use cases:
- MySQL migration
- Open-source preference
```

### Oracle

```yaml
Engine: Oracle Database
Versions: 12c, 19c, 21c
Editions: SE2, EE, EE for RAC
License: BYOL or included

Max storage: 64 TB

Use cases:
- Enterprise applications
- Oracle ecosystem
- Legacy migrations
```

### SQL Server

```yaml
Engine: Microsoft SQL Server
Versions: 2016, 2017, 2019, 2022
Editions: Express, Web, Standard, Enterprise

Max storage: 16 TB

Use cases:
- Windows environments
- .NET applications
- Microsoft ecosystem
```

### Amazon Aurora

```yaml
Engine: Aurora (MySQL/PostgreSQL compatible)
Compatibility: MySQL 5.7, 8.0 / PostgreSQL 11-15

Performance:
- 5x faster than MySQL
- 3x faster than PostgreSQL

Max storage: 128 TB (auto-grows)

Features:
- Serverless option
- Global Database
- Parallel Query

Use cases:
- High performance
- High availability
- Cloud-native apps
```

---

## 🏗️ Tipos de Instancias

### General Purpose (T, M classes)

```yaml
db.t3.micro: 2 vCPU, 1 GB RAM
  Use: Dev/test, small apps
  Price: ~$15/month

db.t3.medium: 2 vCPU, 4 GB RAM
  Use: Small production
  Price: ~$61/month

db.m5.large: 2 vCPU, 8 GB RAM
  Use: Medium production
  Price: ~$145/month
```

### Memory Optimized (R, X classes)

```yaml
db.r5.large: 2 vCPU, 16 GB RAM
  Use: Memory-intensive
  Price: ~$193/month

db.r5.4xlarge: 16 vCPU, 128 GB RAM
  Use: Large databases
  Price: ~$1,546/month

db.x1e.32xlarge: 128 vCPU, 3,904 GB RAM
  Use: In-memory databases
  Price: ~$26,688/month
```

---

## 💾 Storage

### Storage Types

**General Purpose SSD (gp2/gp3)**
```yaml
Type: gp3 (recommended)
Size: 20 GB - 64 TB
Baseline: 3,000 IOPS, 125 MB/s
Max: 16,000 IOPS, 1,000 MB/s

Use: Most workloads
Price: $0.115/GB/month
```

**Provisioned IOPS SSD (io1/io2)**
```yaml
Type: io2
Size: 100 GB - 64 TB
IOPS: 1,000 - 256,000
Throughput: Up to 4,000 MB/s

Use: I/O intensive, low latency
Price: $0.125/GB/month + $0.10/IOPS
```

**Magnetic (legacy)**
```yaml
Type: standard
Size: 20 GB - 3 TB

⚠️ Deprecated, no usar
```

### Storage Autoscaling

```yaml
Storage Autoscaling: Enabled

Maximum storage threshold: 1000 GB
Current allocated: 100 GB

Triggers autoscaling when:
- Free space < 10%
- Low-storage condition > 5 min
- 6+ hours since last modification

Scales in 10% or 10GB increments (whichever is greater)
```

---

## 🔄 Alta Disponibilidad

### Multi-AZ Deployment

```yaml
Multi-AZ: Enabled

Architecture:
┌─────────────────────────────────────┐
│     RDS Instance (Primary)          │
│     AZ-1a                           │
│     10.0.1.50                       │
└───────────┬─────────────────────────┘
            │ Synchronous Replication
            ↓
┌─────────────────────────────────────┐
│     RDS Instance (Standby)          │
│     AZ-1b                           │
│     10.0.2.50                       │
└─────────────────────────────────────┘

Single DNS endpoint:
mydb.abc123.us-east-1.rds.amazonaws.com
→ Points to active instance

Automatic failover: 1-2 minutes
- Hardware failure
- AZ outage
- Planned maintenance
```

**Características:**
- **Synchronous replication:** Zero data loss
- **Automatic failover:** No manual intervention
- **Same region only:** No cross-region
- **+2x cost:** Pay for standby instance
- **No read traffic:** Standby is NOT a read replica

### Single-AZ vs Multi-AZ

```
┌─────────────────┬──────────┬───────────┐
│                 │ Single-AZ│ Multi-AZ  │
├─────────────────┼──────────┼───────────┤
│ Availability    │ ~99.5%   │ ~99.95%   │
│ Failover        │ Manual   │ Auto      │
│ Downtime        │ Minutes  │ 1-2 min   │
│ Data loss risk  │ Yes      │ No        │
│ Cost            │ $        │ $$        │
│ Use case        │ Dev/test │ Production│
└─────────────────┴──────────┴───────────┘
```

---

## 📖 Read Replicas

```yaml
Read Replicas: Up to 15 per source

Purpose:
- Scale read workloads
- Offload reporting queries
- Near-zero impact on primary

Replication: Asynchronous

Cross-Region: Yes (extra latency)

Promotion: Can promote to standalone DB
```

**Arquitectura:**
```
┌─────────────────────────┐
│   Primary DB Instance   │
│   (us-east-1a)         │
│   Writes + Reads       │
└───────┬─────────────────┘
        │ Async Replication
        ├──────────┬──────────┐
        ↓          ↓          ↓
    ┌───────┐  ┌───────┐  ┌───────┐
    │ Read  │  │ Read  │  │ Read  │
    │Replica│  │Replica│  │Replica│
    │  #1   │  │  #2   │  │  #3   │
    └───────┘  └───────┘  └───────┘
    Reads only  Reads only  Reads only
```

**Use cases:**
```yaml
1. Read-heavy workloads:
   - Analytics queries → Read Replica
   - Transactional writes → Primary
   
2. Disaster Recovery:
   - Cross-region Read Replica
   - Promote if region fails
   
3. Reporting:
   - Daily reports → Read Replica
   - Avoid impacting production
```

**Replication Lag:**
```bash
# Monitor replication lag
aws rds describe-db-instances \
  --db-instance-identifier mydb-replica \
  --query 'DBInstances[0].StatusInfos'

# CloudWatch metric: ReplicaLag (seconds)
# Typical: <1 second
# Alert if: >5 seconds
```

---

## 🔐 Security

### Encryption

**At Rest:**
```yaml
Encryption at rest: Enabled
KMS Key: aws/rds (default) or custom CMK

Encrypts:
- DB instance storage
- Automated backups
- Read replicas
- Snapshots

⚠️ Cannot enable after creation
⚠️ Must create new encrypted instance
```

**In Transit:**
```yaml
SSL/TLS: Supported for all engines

# MySQL SSL connection
mysql -h mydb.xyz.rds.amazonaws.com \
  -u admin -p \
  --ssl-ca=rds-ca-2019-root.pem \
  --ssl-mode=REQUIRED

# PostgreSQL SSL
psql "host=mydb.xyz.rds.amazonaws.com \
      dbname=mydb user=postgres \
      sslmode=require"
```

### Network Security

**VPC Placement:**
```yaml
VPC: production-vpc
Subnet Group: db-subnet-group
  - private-subnet-1a (10.0.21.0/24)
  - private-subnet-1b (10.0.22.0/24)

Public accessibility: No

Security Group: db-sg
  Inbound:
    - MySQL (3306) from app-sg only
  Outbound:
    - None (no outbound needed)
```

### IAM Database Authentication

```yaml
IAM DB Auth: Enabled

Benefits:
- No passwords stored in code
- Tokens expire every 15 minutes
- Centralized access control

# Generate auth token
TOKEN=$(aws rds generate-db-auth-token \
  --hostname mydb.xyz.rds.amazonaws.com \
  --port 3306 \
  --username iamuser)

# Connect with token
mysql -h mydb.xyz.rds.amazonaws.com \
  -u iamuser \
  --password=$TOKEN \
  --ssl-ca=rds-ca-2019-root.pem
```

---

## 💾 Backups

### Automated Backups

```yaml
Backup retention: 7 days (default), up to 35 days
Backup window: 03:00-04:00 UTC (1-hour window)
Latest restorable time: Up to 5 minutes ago

How it works:
1. Daily full snapshot (during backup window)
2. Transaction logs every 5 minutes
3. Stored in S3
4. Point-in-time recovery (PITR)

Cost: Free (within DB storage size)
```

**Point-in-Time Recovery:**
```bash
# Restore to specific time
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier mydb \
  --target-db-instance-identifier mydb-restored \
  --restore-time 2024-01-15T10:30:00Z

# Or restore to latest restorable time
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier mydb \
  --target-db-instance-identifier mydb-restored \
  --use-latest-restorable-time
```

### Manual Snapshots

```yaml
Manual Snapshots: User-initiated

Characteristics:
- Retained until explicitly deleted
- Can share with other AWS accounts
- Can copy cross-region
- Incremental (only changed blocks)

Cost: $0.095/GB/month (S3 standard storage)
```

```bash
# Create manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-2024-01

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snapshot-2024-01

# Copy snapshot to another region
aws rds copy-db-snapshot \
  --source-db-snapshot-identifier arn:aws:rds:us-east-1:123456:snapshot:mydb-snap \
  --target-db-snapshot-identifier mydb-snap-copy \
  --source-region us-east-1 \
  --region us-west-2
```

---

## 📊 Monitoring

### CloudWatch Metrics

**Database Metrics:**
```yaml
CPUUtilization: Percent
DatabaseConnections: Count
FreeableMemory: Bytes
FreeStorageSpace: Bytes
ReadIOPS, WriteIOPS: Count/Second
ReadLatency, WriteLatency: Seconds
ReadThroughput, WriteThroughput: Bytes/Second
```

**Enhanced Monitoring:**
```yaml
Enhanced Monitoring: Enabled
Granularity: 1 second (default: 60 seconds)

Additional metrics:
- OS processes
- RDS processes
- CPU per core
- File system utilization

Cost: ~$1.44/month per instance
```

### Performance Insights

```yaml
Performance Insights: Enabled
Retention: 7 days (free) or 2 years ($$$)

Features:
- Top SQL queries
- Wait events analysis
- Database load visualization
- Historical performance data

Dashboard: Real-time + historical analysis
```

**Example query:**
```sql
-- Top queries by execution time
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

---

## 🔧 Maintenance

### Maintenance Window

```yaml
Maintenance Window: Sun 03:00-04:00 UTC

Actions performed:
- OS patching
- Database engine updates
- Required hardware maintenance

Auto minor version upgrade: Enabled/Disabled

Apply immediately: Yes (for critical patches)
                  No (wait for window)
```

### Parameter Groups

```yaml
Parameter Group: my-mysql-params

Parameters:
  max_connections: 500 (default: depends on instance)
  innodb_buffer_pool_size: 16G
  slow_query_log: 1
  long_query_time: 2
  log_queries_not_using_indexes: 1

Apply: Reboot required (most parameters)
       or Dynamic (some parameters)
```

### Option Groups

```yaml
Option Group: my-oracle-options

Options:
  - OEM (Oracle Enterprise Manager)
  - NATIVE_NETWORK_ENCRYPTION
  - SSL
  
Note: Mostly for Oracle/SQL Server
      MySQL/PostgreSQL use extensions
```

---

## 💰 Pricing

### Instance Pricing

```yaml
On-Demand:
  db.t3.micro: $0.017/hour = $12.24/month
  db.t3.medium: $0.068/hour = $48.96/month
  db.m5.large: $0.192/hour = $138.24/month
  db.r5.large: $0.240/hour = $172.80/month

Reserved Instances (1 year):
  db.m5.large: $0.118/hour = $85/month (40% savings)
  
Reserved Instances (3 years):
  db.m5.large: $0.083/hour = $60/month (58% savings)
```

### Storage Pricing

```yaml
General Purpose (gp3):
  $0.115/GB/month
  
  100 GB = $11.50/month
  500 GB = $57.50/month

Provisioned IOPS (io2):
  Storage: $0.125/GB/month
  IOPS: $0.10/provisioned IOPS
  
  100 GB + 10,000 IOPS = $1,012.50/month
```

### Backup Pricing

```yaml
Automated backups: Free (within DB size)
Manual snapshots: $0.095/GB/month

Example:
  DB size: 100 GB
  Automated: Free
  3 manual snapshots (100 GB each): $28.50/month
```

### Multi-AZ Pricing

```yaml
Multi-AZ: 2x instance price

Example:
  Single-AZ db.m5.large: $138.24/month
  Multi-AZ db.m5.large: $276.48/month
```

---

## 📋 Best Practices

### High Availability

```yaml
✅ Multi-AZ for production
✅ Automated backups enabled (retention ≥ 7 days)
✅ Manual snapshots before major changes
✅ Read Replicas for read scaling
✅ Cross-region Read Replica for DR
```

### Security

```yaml
✅ Deploy in private subnets
✅ Restrictive security groups (app tier only)
✅ Encryption at rest enabled
✅ SSL/TLS in transit
✅ IAM database authentication
✅ Regular password rotation
✅ No public accessibility
✅ Audit logging enabled
```

### Performance

```yaml
✅ Right-size instance type
✅ Monitor Performance Insights
✅ Enable Enhanced Monitoring
✅ Tune parameter groups
✅ Use Read Replicas for read-heavy
✅ Consider Aurora for high performance
```

### Cost Optimization

```yaml
✅ Reserved Instances for steady-state
✅ Stop dev/test instances when not in use (Single-AZ only)
✅ Delete old snapshots
✅ Right-size storage (autoscaling)
✅ Use gp3 instead of gp2
✅ Remove unused Read Replicas
```

---

## 🐛 Troubleshooting

### Connection Issues

```bash
# Check security group
# Must allow port from app security group

# Check DB is available
aws rds describe-db-instances \
  --db-instance-identifier mydb \
  --query 'DBInstances[0].DBInstanceStatus'

# Check endpoint
aws rds describe-db-instances \
  --db-instance-identifier mydb \
  --query 'DBInstances[0].Endpoint'

# Test connection
mysql -h mydb.xyz.rds.amazonaws.com -u admin -p
```

### High CPU

```bash
# Check CloudWatch CPUUtilization

# Check Performance Insights for slow queries

# Solutions:
1. Optimize queries (indexes, query tuning)
2. Scale up (larger instance type)
3. Scale out (Read Replicas)
4. Enable query cache (if supported)
```

### Out of Storage

```bash
# Enable storage autoscaling
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --max-allocated-storage 1000 \
  --apply-immediately

# Or manually increase
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --allocated-storage 200 \
  --apply-immediately
```

### Slow Queries

```sql
-- Enable slow query log (MySQL)
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- View slow queries
SELECT * FROM mysql.slow_log;

-- PostgreSQL
-- Enable in parameter group: log_min_duration_statement = 2000
-- View in CloudWatch Logs
```

---

## 📚 Próximos Pasos

Con RDS dominado:

1. [**Aurora**](./aurora.md) → High-performance cloud-native DB
2. [**Lambda**](./lambda.md) → Serverless + RDS integration
3. [**DMS**](./dms.md) → Database migration service
4. **ElastiCache** → Redis/Memcached caching

---

[⬅️ Volver: VPC](./vpc.md) | [➡️ Siguiente: Lambda](./lambda.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
