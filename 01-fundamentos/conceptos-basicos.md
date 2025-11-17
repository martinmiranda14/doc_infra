# Conceptos Básicos de Infraestructura IT

## 📖 Introducción

La infraestructura IT es el conjunto de componentes físicos y virtuales que soportan las operaciones, servicios y aplicaciones de tecnología de una organización. Es la base sobre la cual se construyen y operan todos los sistemas informáticos.

## 🏗️ ¿Qué es la Infraestructura IT?

La infraestructura IT comprende todos los componentes necesarios para operar y gestionar entornos de tecnología empresariales. Incluye tanto elementos de hardware, software, redes, como instalaciones físicas.

### Componentes Principales

```
┌─────────────────────────────────────────────────┐
│         INFRAESTRUCTURA IT COMPLETA            │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐  ┌──────────────┐            │
│  │   Hardware   │  │   Software   │            │
│  │  Servidores  │  │  Sistemas    │            │
│  │  Storage     │  │  Apps        │            │
│  │  Networking  │  │  Databases   │            │
│  └──────────────┘  └──────────────┘            │
│                                                 │
│  ┌──────────────┐  ┌──────────────┐            │
│  │    Redes     │  │  Facilities  │            │
│  │  WAN/LAN     │  │  Datacenters │            │
│  │  Internet    │  │  Cooling     │            │
│  │  Seguridad   │  │  Power       │            │
│  └──────────────┘  └──────────────┘            │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 1. Hardware

**Servidores**
- Máquinas físicas o virtuales que ejecutan aplicaciones y servicios
- Pueden ser de propósito general o especializados
- Ejemplos: servidores web, de bases de datos, de aplicaciones

**Almacenamiento (Storage)**
- Sistemas para guardar datos de forma persistente
- Puede ser local (DAS), en red (NAS), o de bloques (SAN)
- Incluye discos duros, SSDs, arrays de almacenamiento

**Equipos de Red**
- Routers: enrutan tráfico entre diferentes redes
- Switches: conectan dispositivos dentro de una red local
- Firewalls: protegen la red de accesos no autorizados
- Load Balancers: distribuyen carga entre múltiples servidores

### 2. Software

**Sistemas Operativos**
- Linux (Ubuntu, CentOS, Red Hat)
- Windows Server
- Unix variants (FreeBSD, Solaris)

**Software de Gestión**
- Herramientas de monitoreo (Prometheus, Nagios, Zabbix)
- Gestión de configuración (Ansible, Puppet, Chef)
- Orquestación (Kubernetes, Docker Swarm)

**Aplicaciones y Servicios**
- Bases de datos (MySQL, PostgreSQL, MongoDB)
- Servidores web (Apache, Nginx)
- Middleware y servicios empresariales

### 3. Redes

**Componentes de Red**
- Cableado físico (fibra óptica, cobre)
- Redes inalámbricas (Wi-Fi, 5G)
- Protocolos de comunicación (TCP/IP, HTTP, DNS)

**Seguridad de Red**
- Firewalls perimetrales
- IDS/IPS (Detección y Prevención de Intrusiones)
- VPNs para acceso remoto seguro
- Segmentación de redes (VLANs)

### 4. Facilities (Instalaciones)

**Datacenters**
- Espacios físicos diseñados para alojar equipos IT
- Control ambiental (temperatura, humedad)
- Seguridad física (acceso controlado, vigilancia)

**Sistemas de Soporte**
- Alimentación eléctrica (UPS, generadores)
- Sistemas de enfriamiento (HVAC)
- Supresión de incendios

## 🏢 Modelos de Infraestructura

### 1. On-Premise (Local)

Infraestructura ubicada y operada en las instalaciones de la organización.

**Ventajas:**
- ✅ Control total sobre hardware y datos
- ✅ Personalización completa
- ✅ Cumplimiento normativo más fácil en algunos casos
- ✅ Sin dependencia de internet para operaciones locales

**Desventajas:**
- ❌ Alto costo inicial (CAPEX)
- ❌ Requiere espacio físico y mantenimiento
- ❌ Escalabilidad limitada
- ❌ Responsabilidad total de actualizaciones y seguridad

**Casos de uso:**
- Organizaciones con requisitos estrictos de seguridad
- Aplicaciones con baja latencia crítica
- Industrias reguladas (banca, salud, gobierno)

### 2. Cloud (Nube)

Recursos de TI proporcionados como servicio a través de internet.

**Tipos de Cloud:**

**IaaS (Infrastructure as a Service)**
- Provisión de recursos de computación virtualizados
- Ejemplos: AWS EC2, Azure VMs, Google Compute Engine
- Control sobre OS y aplicaciones

**PaaS (Platform as a Service)**
- Plataforma para desarrollo y despliegue de aplicaciones
- Ejemplos: Heroku, Google App Engine, Azure App Service
- Sin gestión de infraestructura subyacente

**SaaS (Software as a Service)**
- Aplicaciones completas entregadas por internet
- Ejemplos: Gmail, Salesforce, Microsoft 365
- Solo configuración, sin gestión técnica

**Ventajas:**
- ✅ Escalabilidad elástica
- ✅ Pago por uso (OPEX vs CAPEX)
- ✅ Despliegue rápido
- ✅ Acceso global
- ✅ Actualizaciones automáticas

**Desventajas:**
- ❌ Dependencia del proveedor (vendor lock-in)
- ❌ Costos variables y potencialmente altos a largo plazo
- ❌ Menor control sobre infraestructura
- ❌ Preocupaciones de seguridad y privacidad

**Casos de uso:**
- Startups y empresas en crecimiento
- Cargas de trabajo variables
- Aplicaciones web y móviles
- Desarrollo y testing

### 3. Híbrida

Combinación de infraestructura on-premise y cloud.

**Ventajas:**
- ✅ Flexibilidad para colocar cargas donde más convenga
- ✅ Optimización de costos
- ✅ Cumplimiento normativo manteniendo datos sensibles on-premise
- ✅ Capacidad de burst a cloud para picos de demanda

**Desventajas:**
- ❌ Complejidad de gestión
- ❌ Requiere expertise en múltiples plataformas
- ❌ Costos de integración
- ❌ Potenciales problemas de latencia entre entornos

**Casos de uso:**
- Migración gradual a cloud
- Datos sensibles on-premise, aplicaciones en cloud
- Disaster recovery
- Optimización de costos

### 4. Multi-Cloud

Uso de servicios de múltiples proveedores cloud simultáneamente.

**Ventajas:**
- ✅ Evita vendor lock-in
- ✅ Selección del mejor servicio de cada proveedor
- ✅ Redundancia y alta disponibilidad
- ✅ Optimización de costos

**Desventajas:**
- ❌ Alta complejidad operativa
- ❌ Requiere expertise en múltiples plataformas
- ❌ Costos de integración y gestión
- ❌ Dificultad en gobernanza unificada

## 🏛️ Arquitecturas Comunes

### Arquitectura de Tres Capas (3-Tier)

```
┌─────────────────────────────────┐
│   Capa de Presentación (Web)   │
│   - Servidores Web              │
│   - Interfaces de Usuario       │
└─────────────┬───────────────────┘
              │
┌─────────────▼───────────────────┐
│   Capa de Aplicación (App)     │
│   - Lógica de Negocio          │
│   - APIs y Servicios           │
└─────────────┬───────────────────┘
              │
┌─────────────▼───────────────────┐
│   Capa de Datos (Data)         │
│   - Bases de Datos             │
│   - Almacenamiento             │
└─────────────────────────────────┘
```

**Características:**
- Separación clara de responsabilidades
- Escalabilidad independiente por capa
- Facilita mantenimiento y actualizaciones
- Arquitectura probada y bien entendida

### Arquitectura de Microservicios

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Service  │  │ Service  │  │ Service  │
│    A     │  │    B     │  │    C     │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     └─────────────┼─────────────┘
                   │
            ┌──────▼──────┐
            │  API Gateway │
            └─────────────┘
```

**Características:**
- Servicios pequeños e independientes
- Despliegue independiente
- Tecnologías heterogéneas
- Escalabilidad granular
- Complejidad en comunicación y orquestación

### Arquitectura Serverless

**Características:**
- Sin gestión de servidores
- Ejecución basada en eventos
- Escalado automático
- Pago por ejecución
- Ideal para cargas irregulares

## 📐 Principios de Diseño

### 1. Alta Disponibilidad (High Availability)

Diseñar sistemas que minimicen el tiempo de inactividad.

**Estrategias:**
- Redundancia en todos los niveles
- Eliminación de puntos únicos de falla (SPOF)
- Balanceo de carga
- Failover automático

**Métricas:**
- SLA (Service Level Agreement): 99.9%, 99.99%, 99.999%
- Downtime permitido:
  - 99.9% = 8.76 horas/año
  - 99.99% = 52.56 minutos/año
  - 99.999% = 5.26 minutos/año

### 2. Escalabilidad

Capacidad de crecer para manejar mayor carga.

**Escalabilidad Vertical (Scale Up)**
- Agregar más recursos a un servidor existente
- Más CPU, RAM, almacenamiento
- Límite físico máximo
- Generalmente requiere downtime

**Escalabilidad Horizontal (Scale Out)**
- Agregar más servidores
- Distribución de carga
- Escalabilidad casi ilimitada
- Requiere arquitectura apropiada

### 3. Resiliencia

Capacidad de recuperarse de fallos.

**Prácticas:**
- Diseño para el fallo
- Degradación elegante
- Circuit breakers
- Retry con backoff exponencial
- Timeouts apropiados

### 4. Seguridad

Protección de datos y sistemas.

**Principios:**
- Defensa en profundidad (múltiples capas)
- Principio de mínimo privilegio
- Segmentación de red
- Cifrado en tránsito y en reposo
- Autenticación y autorización robustas

### 5. Observabilidad

Capacidad de entender el estado del sistema.

**Pilares:**
- **Logs**: Registros de eventos
- **Métricas**: Mediciones numéricas (CPU, memoria, latencia)
- **Traces**: Seguimiento de peticiones distribuidas

**Herramientas:**
- Prometheus + Grafana
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Jaeger, Zipkin (tracing)

### 6. Automatización

Reducir intervención manual.

**Áreas:**
- Infrastructure as Code (IaC)
- CI/CD pipelines
- Auto-scaling
- Backups automáticos
- Remediation automática

## 💡 Mejores Prácticas

### Documentación

- ✅ Mantén documentación actualizada
- ✅ Diagramas de arquitectura claros
- ✅ Runbooks para procedimientos comunes
- ✅ Documentación de configuraciones

### Planificación de Capacidad

- ✅ Monitoreo continuo de uso de recursos
- ✅ Proyecciones de crecimiento
- ✅ Pruebas de carga regulares
- ✅ Presupuesto para escalabilidad

### Disaster Recovery

- ✅ Backups regulares y testados
- ✅ RTO (Recovery Time Objective) definido
- ✅ RPO (Recovery Point Objective) definido
- ✅ Plan de DR documentado y practicado

### Seguridad

- ✅ Actualizaciones y parches regulares
- ✅ Auditorías de seguridad periódicas
- ✅ Gestión de accesos e identidades
- ✅ Monitoreo de seguridad 24/7

## 🎯 Checklist de Infraestructura Básica

Para establecer una infraestructura sólida:

- [ ] Definir requisitos de disponibilidad (SLA)
- [ ] Establecer estrategia de backup y recovery
- [ ] Implementar monitoreo y alertas
- [ ] Configurar seguridad de red (firewalls, segmentación)
- [ ] Documentar arquitectura y procedimientos
- [ ] Planificar escalabilidad
- [ ] Establecer procesos de cambio
- [ ] Implementar logging centralizado
- [ ] Definir políticas de acceso
- [ ] Planificar disaster recovery

## 📚 Próximos Pasos

Ahora que entiendes los conceptos básicos:

1. Revisa la [Terminología](./terminologia.md) para familiarizarte con términos técnicos
2. Explora secciones específicas según tus necesidades:
   - Contenedores (Docker, Kubernetes)
   - Cloud (AWS, Azure, GCP)
   - Redes y Seguridad
   - Servidores On-Premise

---

[⬅️ Volver a Fundamentos](./README.md) | [➡️ Ir a Terminología](./terminologia.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
