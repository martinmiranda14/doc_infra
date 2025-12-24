# IAM - Identity and Access Management

## 📖 ¿Qué es IAM?

**AWS IAM** permite gestionar de forma segura el acceso a servicios y recursos de AWS.

```
IAM = Who can do What on Which resources

Who: Users, Groups, Roles
What: Actions (API calls)
Which: Resources (ARNs)
```

**Características:**
- ✅ Gratuito (no cost)
- ✅ Global service (no región específica)
- ✅ Integrado con todos los servicios AWS
- ✅ Granular permissions

---

## 👥 Identidades

### Root User

```yaml
Root User: Primera cuenta creada

Características:
- Email + password usado para crear cuenta
- Acceso completo e irrestricto
- Cannot be restricted
- No se puede eliminar

⚠️ NO USAR para operaciones diarias

Solo usar para:
- Crear primer usuario IAM admin
- Cambiar plan de soporte
- Cerrar cuenta AWS
- Configurar MFA
```

**Asegurar Root:**
```yaml
✅ Habilitar MFA
✅ No crear access keys
✅ Usar contraseña fuerte
✅ No compartir credenciales
✅ Crear usuarios IAM para operaciones
```

### Users

```yaml
IAM User: Identidad permanente

john@company.com:
  Password: Para AWS Console
  Access Keys: Para AWS CLI/SDK
  Permissions: Via policies
  
Límite: 5,000 usuarios por cuenta
```

**Crear usuario:**
```bash
# Via CLI
aws iam create-user --user-name john

# Agregar a grupo
aws iam add-user-to-group \
  --user-name john \
  --group-name developers

# Crear access key
aws iam create-access-key --user-name john

# Output:
{
  "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
  "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}
```

**Credenciales:**
```yaml
Console password:
  - Username + password
  - Puede requerir MFA
  - Password policy aplicable

Access Keys (CLI/SDK):
  - Access Key ID (público)
  - Secret Access Key (secreto)
  - Máximo 2 keys activos por usuario
  - Rotar cada 90 días
```

### Groups

```yaml
IAM Group: Colección de usuarios

Groups:
  - Developers:
      Users: [john, jane, bob]
      Policies: [EC2FullAccess, S3ReadOnly]
  
  - Admins:
      Users: [alice, charlie]
      Policies: [AdministratorAccess]
  
  - DataScientists:
      Users: [emma, david]
      Policies: [SageMakerFullAccess, S3FullAccess]

⚠️ Grupos NO pueden contener otros grupos
⚠️ Un usuario puede estar en múltiples grupos
```

### Roles

```yaml
IAM Role: Identidad temporal asumible

Diferencia con Users:
- No password ni access keys permanentes
- Credenciales temporales (1-12 horas)
- Asumible por users, services, accounts

Use cases:
1. EC2 instance needs S3 access
2. Lambda function needs DynamoDB access
3. Cross-account access
4. Identity Federation (SSO)
```

**Estructura:**
```yaml
Role: EC2-S3-Access

Trust Policy: (Quién puede asumir)
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "ec2.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}

Permissions Policy: (Qué puede hacer)
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": "*"
  }]
}
```

**Asumir role:**
```bash
# Via CLI
aws sts assume-role \
  --role-arn arn:aws:iam::123456789:role/CrossAccountRole \
  --role-session-name session1

# Recibe credenciales temporales
{
  "Credentials": {
    "AccessKeyId": "ASIAIOSFODNN7EXAMPLE",
    "SecretAccessKey": "...",
    "SessionToken": "...",
    "Expiration": "2024-01-15T12:00:00Z"
  }
}

# Usar credenciales temporales
export AWS_ACCESS_KEY_ID="ASIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="..."
export AWS_SESSION_TOKEN="..."
```

---

## 📜 Policies

### Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ListBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-bucket",
      "Condition": {
        "StringLike": {
          "s3:prefix": ["home/${aws:username}/*"]
        }
      }
    },
    {
      "Sid": "AllowS3ObjectOperations",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/home/${aws:username}/*"
    }
  ]
}
```

**Componentes:**
```yaml
Version: "2012-10-17" (required)

Statement: Array de permisos
  - Sid: Statement ID (optional, descriptivo)
  - Effect: Allow o Deny
  - Action: API actions (s3:GetObject, ec2:*)
  - Resource: ARN del recurso
  - Condition: Condiciones opcionales
```

### Tipos de Policies

#### 1. AWS Managed Policies

```yaml
Características:
- Creadas y mantenidas por AWS
- Actualizadas automáticamente
- No se pueden editar
- Compartidas entre cuentas

Ejemplos:
- AdministratorAccess: Full access
- PowerUserAccess: Admin sin IAM
- ReadOnlyAccess: Solo lectura
- AmazonS3FullAccess: S3 completo
- AmazonEC2ReadOnlyAccess: EC2 read-only
```

#### 2. Customer Managed Policies

```yaml
Características:
- Creadas por ti
- Version control (hasta 5 versiones)
- Reutilizables
- Máximo 6,144 caracteres

Use case: Permisos específicos de tu organización
```

```bash
# Crear policy
aws iam create-policy \
  --policy-name MyCustomPolicy \
  --policy-document file://policy.json

# Attach a usuario
aws iam attach-user-policy \
  --user-name john \
  --policy-arn arn:aws:iam::123456789:policy/MyCustomPolicy
```

#### 3. Inline Policies

```yaml
Características:
- Embedido directamente en user/group/role
- Relación 1:1 (no reutilizable)
- Eliminado cuando se elimina identity

Use case: Permisos únicos para una identidad específica
```

### Policy Variables

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": [
      "arn:aws:s3:::company-bucket/home/${aws:username}/*"
    ]
  }]
}
```

**Variables disponibles:**
```yaml
${aws:username}: IAM username
${aws:userid}: Unique ID
${aws:SourceIp}: IP del cliente
${aws:CurrentTime}: Timestamp
${aws:PrincipalArn}: ARN del principal
${aws:PrincipalAccount}: Account ID
```

---

## 🔒 Permission Evaluation

### Evaluation Logic

```
1. By default, deny all
2. Evaluate all applicable policies
3. Explicit DENY always wins
4. If any ALLOW exists (and no DENY) → ALLOW
```

```yaml
Scenario 1: Single Allow
  Policy A: Allow s3:GetObject
  Result: ✅ ALLOW

Scenario 2: Allow + Deny
  Policy A: Allow s3:*
  Policy B: Deny s3:DeleteObject
  Result for GetObject: ✅ ALLOW
  Result for DeleteObject: ❌ DENY

Scenario 3: No explicit Allow
  Policy A: (no relevant statement)
  Result: ❌ DENY (implicit)
```

### Permission Boundaries

```yaml
Permission Boundary: Límite máximo de permisos

User: john
  Policy: AdministratorAccess (allow all)
  Permission Boundary: S3-Only
  
Effective permissions: S3 only

Use case:
- Delegación segura
- Compliance
- Prevenir privilege escalation
```

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:*",
      "cloudwatch:*"
    ],
    "Resource": "*"
  }]
}
```

### Service Control Policies (SCPs)

```yaml
SCPs: Policies a nivel AWS Organization

Aplicable a:
- Entire organization
- Organization Units (OUs)
- Individual accounts

⚠️ NO aplica a root account de management account

Example use:
- Deny access to specific regions
- Deny deletion of CloudTrail logs
- Enforce encryption
```

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyAllOutsideUS",
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "StringNotEquals": {
        "aws:RequestedRegion": [
          "us-east-1",
          "us-west-2"
        ]
      }
    }
  }]
}
```

---

## 🔑 MFA (Multi-Factor Authentication)

### Tipos de MFA

```yaml
1. Virtual MFA (Recomendado):
   - Google Authenticator
   - Microsoft Authenticator
   - Authy
   
2. Hardware MFA:
   - Yubikey
   - Gemalto token
   
3. SMS MFA:
   - ⚠️ Menos seguro
   - No recomendado
```

### Habilitar MFA

```bash
# Via CLI (Virtual MFA)
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name root-mfa \
  --outfile QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA
aws iam enable-mfa-device \
  --user-name root \
  --serial-number arn:aws:iam::123456789:mfa/root-mfa \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### MFA en Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {
        "aws:MultiFactorAuthPresent": "false"
      }
    }
  }]
}
```

---

## 🔐 Best Practices

### Principio de Mínimo Privilegio

```yaml
❌ MAL:
User: developer
Policy: AdministratorAccess

✅ BIEN:
User: developer
Policy: Custom (solo lo que necesita)
  - EC2: Read, Start, Stop
  - S3: Read bucket "dev-*"
  - CloudWatch: Read logs
```

### Rotar Credenciales

```bash
# Listar access keys
aws iam list-access-keys --user-name john

# Crear nueva key
aws iam create-access-key --user-name john

# Actualizar aplicaciones con nueva key
# ...

# Desactivar key antigua
aws iam update-access-key \
  --user-name john \
  --access-key-id AKIAIOSFODNN7OLD \
  --status Inactive

# Eliminar key antigua (después de validar)
aws iam delete-access-key \
  --user-name john \
  --access-key-id AKIAIOSFODNN7OLD
```

### Usar Roles en lugar de Users

```yaml
❌ MAL:
EC2 instance:
  - Hardcode access keys in app
  - Store keys in /home/.aws/credentials

✅ BIEN:
EC2 instance:
  - Attach IAM role
  - App usa AWS SDK (auto-refresh credentials)
```

### Habilitar CloudTrail

```yaml
CloudTrail: Audit log de todas las API calls

Logs:
  - Who: Usuario/role
  - What: Action (CreateUser, DeleteBucket)
  - When: Timestamp
  - Where: IP address, region
  - Result: Success/failure

Retention: S3 bucket (años)
Analysis: Athena, CloudWatch Insights
```

### Password Policy

```yaml
Password Policy:
  Minimum length: 14
  Require uppercase: Yes
  Require lowercase: Yes
  Require numbers: Yes
  Require symbols: Yes
  Expire in days: 90
  Prevent reuse: 24 passwords
  Allow users to change: Yes
```

### Access Analyzer

```yaml
IAM Access Analyzer:
  - Analiza policies
  - Identifica recursos compartidos externamente
  - Detecta acceso no intencional
  - Valida policies contra best practices

Findings:
  - S3 bucket público
  - IAM role asumible por external account
  - KMS key compartida
```

---

## 👁️ Monitoring y Auditing

### CloudTrail Events

```json
{
  "eventTime": "2024-01-15T10:30:00Z",
  "eventName": "CreateUser",
  "userIdentity": {
    "type": "IAMUser",
    "userName": "admin",
    "arn": "arn:aws:iam::123456789:user/admin"
  },
  "requestParameters": {
    "userName": "john"
  },
  "responseElements": {
    "user": {
      "userName": "john",
      "userId": "AIDAI23EXAMPLE",
      "arn": "arn:aws:iam::123456789:user/john"
    }
  }
}
```

### IAM Credential Report

```bash
# Generar reporte
aws iam generate-credential-report

# Descargar reporte
aws iam get-credential-report --output text

# CSV con:
# - user, arn, user_creation_time
# - password_enabled, password_last_used
# - access_key_1_active, access_key_1_last_used
# - mfa_active
```

### Access Advisor

```yaml
Access Advisor: Last accessed information

Para cada servicio:
  - Last accessed time
  - Services not used

Use case:
  - Identify unused permissions
  - Refine policies
  - Implement least privilege
```

---

## 🌐 Cross-Account Access

### Scenario

```
Account A (Production): 111111111111
Account B (Development): 222222222222

Goal: Users in Account B access resources in Account A
```

### Setup

**Account A (Trust):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::222222222222:root"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

**Account B (Permissions):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "sts:AssumeRole",
    "Resource": "arn:aws:iam::111111111111:role/CrossAccountRole"
  }]
}
```

**User in Account B:**
```bash
# Assume role in Account A
aws sts assume-role \
  --role-arn arn:aws:iam::111111111111:role/CrossAccountRole \
  --role-session-name session1

# Use temporary credentials
```

---

## 🔄 Identity Federation

### SAML 2.0 (Enterprise)

```yaml
Flow:
1. User → Corporate Identity Provider (Okta, AD FS)
2. IdP → SAML assertion
3. SAML assertion → AWS STS
4. STS → Temporary credentials
5. User → Access AWS

Use case: Corporate SSO
```

### Web Identity Federation

```yaml
Supported providers:
- Google
- Facebook
- Amazon (Login with Amazon)
- Any OpenID Connect compatible

Use case: Mobile/web apps con social login
```

### AWS SSO (Recommended)

```yaml
AWS IAM Identity Center (ex-SSO):
  - Centralized access management
  - Multiple AWS accounts
  - Business applications (Salesforce, Office 365)
  - SAML-based

Benefits:
  ✅ Single sign-on
  ✅ Multi-account management
  ✅ Permission sets
  ✅ Audit logs
```

---

## 💰 IAM Costs

```yaml
IAM Service: FREE

Free:
- Users, groups, roles
- Policies
- MFA
- Password policies
- Access Analyzer

Costs (related services):
- CloudTrail: Logging API calls (~$2/month)
- IAM Identity Center: Free (SSO)
- AWS Organizations: Free
```

---

## 📋 IAM Checklist

```yaml
✅ Root account:
   - MFA enabled
   - No access keys
   - Strong password
   - Not used daily

✅ Users:
   - Individual users (no sharing)
   - Strong passwords
   - MFA for console access
   - Rotate access keys (90 days)
   - Groups for permissions

✅ Roles:
   - Use for AWS services (EC2, Lambda)
   - Cross-account access
   - Temporary credentials

✅ Policies:
   - Least privilege
   - No wildcards unless necessary
   - Regular review (Access Advisor)

✅ Monitoring:
   - CloudTrail enabled
   - Access Analyzer enabled
   - Regular credential reports
   - Alert on root usage

✅ Compliance:
   - Password policy enforced
   - MFA enforced for privileged
   - Unused credentials removed
   - Documentation updated
```

---

## 🐛 Troubleshooting

### Access Denied

```bash
# Debug steps:
1. Check user/role has required policy
aws iam list-attached-user-policies --user-name john

2. Check policy allows action
aws iam get-policy-version \
  --policy-arn arn:aws:iam::123456789:policy/MyPolicy \
  --version-id v1

3. Check for explicit DENY
# Review all policies (including SCPs, permission boundaries)

4. Check resource-based policy
aws s3api get-bucket-policy --bucket my-bucket

5. Use Policy Simulator
https://policysim.aws.amazon.com/
```

### Credentials Not Working

```bash
# Verify credentials
aws sts get-caller-identity

# Check if key is active
aws iam list-access-keys --user-name john

# Verify region
aws configure get region

# Check if MFA is required but not provided
```

---

## 📚 Próximos Pasos

Con IAM dominado:

1. **AWS Organizations** → Multi-account management
2. **AWS SSO (Identity Center)** → Single sign-on
3. **Secrets Manager** → Credential management
4. **Security Hub** → Comprehensive security

---

[⬅️ Volver: Lambda](./lambda.md) | [🏠 Volver a Cloud Computing](../README.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
