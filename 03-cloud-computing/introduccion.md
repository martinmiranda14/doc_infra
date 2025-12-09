# Introducción a Cloud Computing

## 📖 ¿Qué es Cloud Computing?

**Cloud Computing** (Computación en la Nube) es la entrega de recursos de TI bajo demanda a través de Internet con precios de pago por uso.

### Definición

```
Cloud Computing = 
  Recursos de TI +
  Bajo demanda +
  Por Internet +
  Pago por uso
```

En lugar de comprar, poseer y mantener servidores físicos y datacenters, puedes acceder a servicios tecnológicos como poder de cómputo, almacenamiento y bases de datos según sea necesario.

---

## 🎯 Características Esenciales

### 1. Autoservicio Bajo Demanda

Puedes aprovisionar recursos automáticamente sin interacción humana con el proveedor.

```
Antes (On-Premise):
1. Solicitar servidor → 2-4 semanas
2. Aprobar presupuesto
3. Comprar hardware
4. Instalar y configurar
5. Usar

Ahora (Cloud):
1. Click en consola → 2 minutos
2. Usar
```

### 2. Acceso Amplio a la Red

Servicios disponibles en la red, accesibles desde cualquier dispositivo.

### 3. Pool de Recursos

Recursos compartidos entre múltiples clientes (multi-tenancy).

### 4. Elasticidad Rápida

Escalar recursos hacia arriba o abajo automáticamente según demanda.

### 5. Servicio Medido

Uso monitoreado, controlado y reportado. Pagas solo lo que usas.

---

## 🏗️ Modelos de Servicio

### IaaS - Infrastructure as a Service

**Qué obtienes:** Infraestructura de TI virtualizada.

```
Tú gestionas:
- Aplicaciones
- Datos
- Runtime
- Middleware
- Sistema Operativo

Proveedor gestiona:
- Virtualización
- Servidores
- Storage
- Networking
```

**Ejemplos:**
- AWS EC2
- Azure Virtual Machines
- Google Compute Engine

**Casos de uso:**
- Hosting de aplicaciones
- Desarrollo y testing
- Disaster recovery
- Web hosting

**Ventajas:**
✅ Control total del sistema operativo
✅ Flexibilidad máxima
✅ No necesitas hardware físico

**Desventajas:**
❌ Debes gestionar OS, patching, seguridad
❌ Mayor responsabilidad operativa

### PaaS - Platform as a Service

**Qué obtienes:** Plataforma completa para desarrollo y despliegue.

```
Tú gestionas:
- Aplicaciones
- Datos

Proveedor gestiona:
- Runtime
- Middleware
- Sistema Operativo
- Virtualización
- Servidores
- Storage
- Networking
```

**Ejemplos:**
- AWS Elastic Beanstalk
- Azure App Service
- Google App Engine
- Heroku

**Casos de uso:**
- Desarrollo de aplicaciones web
- API backends
- Microservicios

**Ventajas:**
✅ Enfoque en código, no en infraestructura
✅ Despliegue rápido
✅ Scaling automático

**Desventajas:**
❌ Menos control del entorno
❌ Posible vendor lock-in
❌ Limitaciones de runtime/lenguajes

### SaaS - Software as a Service

**Qué obtienes:** Aplicación completa lista para usar.

```
Tú gestionas:
- Datos (tu contenido)

Proveedor gestiona:
- TODO lo demás (aplicación, plataforma, infraestructura)
```

**Ejemplos:**
- Gmail, Outlook
- Salesforce
- Dropbox, Google Drive
- Slack, Microsoft Teams
- GitHub

**Casos de uso:**
- Email empresarial
- CRM
- Colaboración
- Productividad

**Ventajas:**
✅ Sin gestión de infraestructura
✅ Acceso desde cualquier lugar
✅ Actualizaciones automáticas

**Desventajas:**
❌ Cero control de infraestructura
❌ Dependencia del proveedor
❌ Preocupaciones de privacidad de datos

### Comparación Visual

```
         On-Premise    IaaS         PaaS         SaaS
        ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
You     │ Apps     │ │ Apps     │ │ Apps     │ │          │
Manage  │ Data     │ │ Data     │ │ Data     │ │          │
        │ Runtime  │ │ Runtime  │ │          │ │          │
        │ Middle   │ │ Middle   │ │          │ │          │
        │ OS       │ │ OS       │ │          │ │          │
        ├──────────┤ ├──────────┤ ├──────────┤ │          │
Provider│          │ │ Virtual  │ │ Runtime  │ │ Apps     │
Manages │          │ │ Servers  │ │ Middle   │ │ Data     │
        │          │ │ Storage  │ │ OS       │ │ Runtime  │
        │          │ │ Network  │ │ Virtual  │ │ Middle   │
        │          │ │          │ │ Servers  │ │ OS       │
        │          │ │          │ │ Storage  │ │ Virtual  │
        │          │ │          │ │ Network  │ │ Servers  │
        │          │ │          │ │          │ │ Storage  │
        │          │ │          │ │          │ │ Network  │
        └──────────┘ └──────────┘ └──────────┘ └──────────┘
```

---

## 🌐 Modelos de Despliegue

### 1. Nube Pública

Infraestructura compartida, disponible para público general.

**Características:**
- Multi-tenancy
- Pago por uso
- Sin CAPEX
- Escalabilidad masiva

**Proveedores:**
- AWS, Azure, GCP
- DigitalOcean, Linode
- Oracle Cloud

**Ventajas:**
✅ Sin inversión inicial
✅ Escalabilidad infinita
✅ Mantenimiento del proveedor
✅ Alta disponibilidad

**Desventajas:**
❌ Menos control
❌ Preocupaciones de seguridad/compliance
❌ Latencia para algunos casos

**Cuándo usar:**
- Startups
- Aplicaciones web públicas
- Desarrollo y testing
- Workloads con demanda variable

### 2. Nube Privada

Infraestructura dedicada para una sola organización.

**Tipos:**
- **On-premise:** En tu datacenter (OpenStack, VMware)
- **Hosted:** Gestionada por tercero pero dedicada

**Ventajas:**
✅ Control total
✅ Seguridad y compliance
✅ Personalización completa
✅ Rendimiento predecible

**Desventajas:**
❌ Alto CAPEX
❌ Gestión propia
❌ Escalabilidad limitada
❌ Requiere equipo especializado

**Cuándo usar:**
- Sector financiero/salud (compliance)
- Datos altamente sensibles
- Legacy applications
- Requisitos de latencia muy bajos

### 3. Nube Híbrida

Combinación de nube pública y privada.

```
┌─────────────────┐         ┌─────────────────┐
│  Private Cloud  │◄────────►│  Public Cloud   │
│                 │         │                 │
│ • Legacy apps   │         │ • Web apps      │
│ • Sensitive data│         │ • Dev/test      │
│ • Compliance    │         │ • Bursting      │
└─────────────────┘         └─────────────────┘
```

**Ventajas:**
✅ Mejor de ambos mundos
✅ Flexibilidad máxima
✅ Cloud bursting
✅ Gradual migration

**Desventajas:**
❌ Complejidad de gestión
❌ Integración desafiante
❌ Requiere expertise en ambos
❌ Costos de conectividad

**Cuándo usar:**
- Migración gradual a cloud
- Datos sensibles + workloads variables
- Disaster recovery
- Cumplir múltiples requisitos

### 4. Multi-Cloud

Uso de múltiples proveedores cloud.

```
        ┌─────────────┐
        │  Your Apps  │
        └──────┬──────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌───▼───┐  ┌───▼───┐
│  AWS  │  │ Azure │  │  GCP  │
└───────┘  └───────┘  └───────┘
```

**Ventajas:**
✅ Evitar vendor lock-in
✅ Best-of-breed services
✅ Resiliencia mejorada
✅ Negociación de precios

**Desventajas:**
❌ Complejidad máxima
❌ Múltiples equipos/skills
❌ Gestión de identidad compleja
❌ Costos de egreso de datos

**Cuándo usar:**
- Grandes empresas
- Evitar dependencia de un proveedor
- Aprovechar servicios específicos
- Requisitos regulatorios geográficos

---

## 💰 Modelo de Costos

### CAPEX vs OPEX

**CAPEX (Capital Expenditure):**
```
Inversión inicial grande → Activo que se deprecia

On-Premise:
- Compra de servidores: $100,000
- Datacenter setup: $500,000
- Amortización: 3-5 años
```

**OPEX (Operational Expenditure):**
```
Gasto operativo mensual → Gasto deducible

Cloud:
- Pago mensual según uso
- Sin inversión inicial
- Gasto variable
```

### Pricing Models

#### 1. Pay-as-you-go

Pagas por segundo/hora/GB usado.

```
EC2 instance: $0.10/hora
Usas 100 horas = $10
```

#### 2. Reserved Instances

Compromiso de 1-3 años = descuento 30-75%.

```
EC2 Reserved (1 año): $0.065/hora
Ahorro: 35%
```

#### 3. Spot Instances

Capacidad no usada a precio reducido (hasta 90% descuento).

```
EC2 Spot: $0.03/hora
Puede ser interrumpido con 2 min notice
```

#### 4. Savings Plans

Compromiso de gasto por hora = descuento.

```
Commit $10/hora por 1 año = 20-40% descuento
Flexible entre servicios
```

### Factores de Costo

1. **Compute:** Tipo y tamaño de instancias
2. **Storage:** Cantidad, tipo, durabilidad
3. **Networking:** Transferencia de datos, especialmente egress
4. **Additional services:** Balanceadores, IPs, etc.

---

## 🏆 Beneficios de Cloud

### 1. Agilidad

```
Provisión de recursos: minutos vs semanas
Experimentación: bajo costo de failure
Time-to-market: más rápido
```

### 2. Elasticidad

```
Escalado automático según demanda:
  ┌─────────────────────────────┐
  │     Traffic Pattern         │
  │         ▄▄▄▄                │
  │        ▐████▌               │
  │    ▄▄▄▄▐████▌▄▄▄            │
  │▄▄▄▄▐████████████▌▄▄▄        │
  └─────────────────────────────┘
   Resources scale automatically
```

### 3. Ahorro de Costos

- Sin CAPEX inicial
- Pago por uso real
- Economías de escala del proveedor
- Reducción de personal de datacenter

### 4. Alcance Global

```
Despliegue global en minutos:
US-East → Europe → Asia → Australia
  5 min     5 min    5 min    5 min
```

### 5. Seguridad

- Proveedores invierten millones en seguridad
- Compliance certificado (SOC, ISO, PCI-DSS)
- DDoS protection
- Actualizaciones automáticas

### 6. Innovación

Acceso a tecnologías avanzadas:
- Machine Learning / AI
- IoT platforms
- Big Data analytics
- Quantum computing

---

## ⚠️ Consideraciones y Desafíos

### 1. Seguridad y Compliance

```
❌ Errores comunes:
- Buckets S3 públicos
- Credenciales en código
- Security groups permisivos

✅ Mejores prácticas:
- Least privilege
- Encriptación
- MFA
- Auditoría
```

### 2. Costos Inesperados

```
Causas:
- Recursos olvidados
- Data transfer charges
- Over-provisioning
- Falta de monitoreo

Solución:
- Alerts de billing
- Tagging
- Auto-shutdown
- Reserved capacity
```

### 3. Vendor Lock-in

```
Riesgo:
- Servicios propietarios (DynamoDB, CosmosDB)
- Difícil migración

Mitigación:
- Usar estándares (Kubernetes, Postgres)
- Abstraction layers
- Multi-cloud strategy
```

### 4. Complejidad

```
Desafíos:
- 200+ servicios en AWS
- Curva de aprendizaje
- Cambios constantes

Solución:
- Certificaciones
- Documentación
- Start simple
```

### 5. Latencia

```
Problema:
- Datos en cloud, usuarios on-premise
- Cross-region traffic

Solución:
- CDN
- Edge locations
- Hybrid architecture
```

---

## 📊 Comparación de Proveedores

| Feature | AWS | Azure | GCP | Otros |
|---------|-----|-------|-----|-------|
| **Fundación** | 2006 | 2010 | 2008 | - |
| **Cuota mercado** | ~32% | ~23% | ~10% | ~35% |
| **Regiones** | 30+ | 60+ | 35+ | - |
| **Servicios** | 200+ | 200+ | 100+ | - |
| **Fortaleza** | Más amplio | Microsoft stack | ML/Analytics | - |
| **Certificaciones** | 12 | 9 | 5 | - |
| **Free Tier** | 12m + perpetuo | 12m + créditos | 90d + perpetuo | Varía |

### AWS (Amazon Web Services)

**Ventajas:**
- Ecosistema más maduro y amplio
- Mayor comunidad
- Más regiones geográficas
- Innovación constante

**Ideal para:**
- Startups
- Microservicios
- Contenedores/Kubernetes
- Serverless

### Azure (Microsoft Azure)

**Ventajas:**
- Integración con Microsoft (AD, Office 365)
- Hybrid cloud robusto
- Enterprise agreements
- Windows workloads

**Ideal para:**
- Empresas con stack Microsoft
- .NET applications
- Hybrid scenarios
- Enterprise customers

### GCP (Google Cloud Platform)

**Ventajas:**
- Machine Learning (TensorFlow)
- Big Data (BigQuery)
- Kubernetes (GKE)
- Pricing competitivo

**Ideal para:**
- Data analytics
- ML/AI workloads
- Kubernetes-first
- Startups tech-savvy

### Otros Proveedores

- **DigitalOcean:** Simple, developers, startups
- **Linode:** VPS simple y económico
- **Oracle Cloud:** Databases, enterprise
- **IBM Cloud:** Enterprise, Watson AI
- **Alibaba Cloud:** Asia-Pacific

---

## 🎓 Casos de Uso Comunes

### 1. Hosting de Aplicaciones Web

```
Arquitectura típica:
Internet → Load Balancer → Web Servers → Database
                ↓
          Object Storage (S3)
```

### 2. Backup y Disaster Recovery

```
On-Premise → Backup to Cloud Storage
             ↓
         Recovery en caso de desastre
```

### 3. Desarrollo y Testing

```
Dev → Test → Staging → Production
(escalar/eliminar ambientes fácilmente)
```

### 4. Big Data y Analytics

```
Ingest → Storage → Processing → Analytics → Visualization
(IoT)   (S3)      (Spark)      (Redshift)  (BI tools)
```

### 5. Machine Learning

```
Data → Training → Model → Inference
      (GPUs)     (Store)  (Serverless)
```

---

## 🚀 Primeros Pasos

### Elegir un Proveedor

**Para comenzar, recomendamos AWS porque:**
1. Mayor cuota de mercado
2. Más demanda laboral
3. Mejor documentación y comunidad
4. Free Tier generoso

### Roadmap de Aprendizaje

```
1. Crear cuenta (Free Tier)
   ↓
2. Completar tutorial básico
   ↓
3. Lanzar primera instancia (EC2)
   ↓
4. Configurar almacenamiento (S3)
   ↓
5. Entender IAM y seguridad
   ↓
6. Explorar servicios managed (RDS, Lambda)
   ↓
7. Certificación (Cloud Practitioner)
```

### Recursos de Aprendizaje

**Gratuitos:**
- AWS Skill Builder
- Microsoft Learn
- Google Cloud Skills Boost
- YouTube channels

**Pagos:**
- A Cloud Guru / Pluralsight
- Udemy courses
- Cloud Academy
- Linux Academy

---

## 📋 Checklist de Inicio

- [ ] Crear cuenta en proveedor cloud
- [ ] Configurar MFA en cuenta root
- [ ] Entender Free Tier limits
- [ ] Configurar billing alerts
- [ ] Explorar consola web
- [ ] Instalar CLI
- [ ] Lanzar primera instancia
- [ ] Configurar tags para organización
- [ ] Revisar mejores prácticas de seguridad
- [ ] Unirse a comunidades (Reddit, foros)

---

## 📚 Próximos Pasos

Con los conceptos de cloud dominados:

1. [**Migración a Cloud**](./migracion-cloud.md) → Estrategias y planificación
2. [**AWS Introducción**](./aws/introduccion.md) → Comenzar con AWS
3. [**Azure Introducción**](./azure/introduccion.md) → Comenzar con Azure
4. [**GCP Introducción**](./gcp/introduccion.md) → Comenzar con GCP

---

## 🔗 Recursos Adicionales

- [NIST Definition of Cloud Computing](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-145.pdf)
- [Cloud Computing Patterns](https://www.cloudcomputingpatterns.org/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)
- [GCP Architecture Framework](https://cloud.google.com/architecture/framework)

---

[⬅️ Volver: Cloud Computing](./README.md) | [➡️ Siguiente: Migración a Cloud](./migracion-cloud.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
