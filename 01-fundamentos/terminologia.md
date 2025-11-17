# Terminología de Infraestructura IT

Glosario completo de términos esenciales en infraestructura IT, organizado por categorías para facilitar la consulta.

## 📑 Índice de Categorías

- [Términos Generales](#-términos-generales)
- [Computación](#-computación)
- [Almacenamiento](#-almacenamiento)
- [Redes](#-redes)
- [Cloud Computing](#-cloud-computing)
- [Contenedores y Orquestación](#-contenedores-y-orquestación)
- [Seguridad](#-seguridad)
- [Monitoreo y Observabilidad](#-monitoreo-y-observabilidad)
- [DevOps y Automatización](#-devops-y-automatización)
- [Bases de Datos](#-bases-de-datos)

---

## 🌐 Términos Generales

### API (Application Programming Interface)
Conjunto de definiciones y protocolos que permiten la comunicación entre diferentes aplicaciones de software.

### Availability (Disponibilidad)
Porcentaje de tiempo que un sistema está operativo y accesible. Se mide comúnmente en "nueves" (99.9%, 99.99%, etc.).

### CAPEX (Capital Expenditure)
Gasto de capital en infraestructura física (servidores, equipos, instalaciones) que se amortiza a lo largo del tiempo.

### Cluster
Grupo de servidores o nodos que trabajan juntos como un sistema único para proporcionar alta disponibilidad, escalabilidad y rendimiento.

### Datacenter
Instalación física que alberga sistemas informáticos y componentes asociados, como almacenamiento y redes.

### Downtime
Periodo durante el cual un sistema no está disponible o no funciona correctamente.

### Endpoint
Punto de conexión en una red donde un dispositivo o servicio puede ser accedido.

### Failover
Proceso automático de cambiar a un sistema de respaldo redundante cuando el sistema principal falla.

### Latency (Latencia)
Tiempo de retardo en la transmisión de datos entre dos puntos. Generalmente medido en milisegundos (ms).

### OPEX (Operating Expenditure)
Gastos operativos continuos (servicios cloud, licencias, mantenimiento) que se pagan de forma recurrente.

### Redundancy (Redundancia)
Duplicación de componentes críticos del sistema para aumentar la confiabilidad y disponibilidad.

### SLA (Service Level Agreement)
Acuerdo que define el nivel de servicio esperado entre un proveedor y un cliente, incluyendo métricas como uptime.

### SPOF (Single Point of Failure)
Componente único cuyo fallo causaría la caída de todo el sistema.

### Throughput (Rendimiento)
Cantidad de datos procesados o transmitidos en un período de tiempo determinado.

### Uptime
Tiempo durante el cual un sistema está operativo y funcionando correctamente.

---

## 💻 Computación

### Bare Metal
Servidor físico sin capa de virtualización, donde el sistema operativo se ejecuta directamente sobre el hardware.

### CPU (Central Processing Unit)
Unidad central de procesamiento que ejecuta instrucciones de programas.

### Core
Unidad de procesamiento individual dentro de un CPU. Los CPUs modernos son multi-core.

### GPU (Graphics Processing Unit)
Procesador especializado originalmente para gráficos, ahora usado también para computación paralela (ML, mining, etc.).

### Hypervisor
Software que crea y gestiona máquinas virtuales. Ejemplos: VMware ESXi, KVM, Hyper-V.

### Instance
Servidor virtual en la nube. También llamado VM o máquina virtual.

### RAM (Random Access Memory)
Memoria volátil de acceso rápido usada para datos y programas en ejecución.

### Server
Computadora o programa que proporciona servicios a otros programas o dispositivos (clientes).

### Virtualization (Virtualización)
Tecnología que permite ejecutar múltiples sistemas operativos en un solo servidor físico.

### VM (Virtual Machine)
Sistema operativo completo ejecutándose de forma aislada sobre hardware virtualizado.

### vCPU (Virtual CPU)
CPU virtual asignada a una máquina virtual.

---

## 💾 Almacenamiento

### Block Storage
Almacenamiento de datos en bloques de tamaño fijo, típicamente usado para discos de VMs. Ejemplos: AWS EBS, Azure Disk Storage.

### Backup
Copia de seguridad de datos que puede ser restaurada en caso de pérdida o corrupción.

### Cold Storage
Almacenamiento de bajo costo para datos raramente accedidos. Alta latencia de recuperación.

### DAS (Direct-Attached Storage)
Almacenamiento conectado directamente a un servidor sin red intermedia.

### Hot Storage
Almacenamiento de alto rendimiento para datos frecuentemente accedidos.

### IOPS (Input/Output Operations Per Second)
Métrica que mide el rendimiento de dispositivos de almacenamiento.

### NAS (Network-Attached Storage)
Dispositivo de almacenamiento conectado a la red que proporciona almacenamiento de archivos compartido.

### Object Storage
Almacenamiento de datos como objetos con metadata. Ideal para datos no estructurados. Ejemplos: AWS S3, Azure Blob Storage.

### RAID (Redundant Array of Independent Disks)
Tecnología que combina múltiples discos para redundancia y/o rendimiento.
- **RAID 0**: Striping (rendimiento, sin redundancia)
- **RAID 1**: Mirroring (redundancia)
- **RAID 5**: Striping con paridad (balance rendimiento/redundancia)
- **RAID 10**: Combinación de RAID 1 y 0

### SAN (Storage Area Network)
Red dedicada de alta velocidad que conecta servidores con dispositivos de almacenamiento.

### Snapshot
Copia instantánea del estado de un sistema de almacenamiento en un momento específico.

### SSD (Solid-State Drive)
Dispositivo de almacenamiento sin partes móviles, más rápido que discos duros tradicionales.

---

## 🌐 Redes

### Bandwidth (Ancho de banda)
Capacidad máxima de transmisión de datos de una conexión de red, medida en bits por segundo (bps).

### CDN (Content Delivery Network)
Red distribuida de servidores que entrega contenido a usuarios basándose en su ubicación geográfica.

### CIDR (Classless Inter-Domain Routing)
Notación para especificar rangos de direcciones IP. Ejemplo: 192.168.1.0/24

### DHCP (Dynamic Host Configuration Protocol)
Protocolo que asigna automáticamente direcciones IP a dispositivos en una red.

### DNS (Domain Name System)
Sistema que traduce nombres de dominio (www.ejemplo.com) a direcciones IP.

### Firewall
Sistema de seguridad que monitorea y controla el tráfico de red entrante y saliente basado en reglas.

### Gateway
Dispositivo que actúa como punto de acceso entre diferentes redes.

### IP Address (Dirección IP)
Identificador numérico único para un dispositivo en una red.
- **IPv4**: 192.168.1.1 (32 bits)
- **IPv6**: 2001:0db8:85a3:0000:0000:8a2e:0370:7334 (128 bits)

### LAN (Local Area Network)
Red que conecta dispositivos en un área geográfica limitada (oficina, edificio).

### Load Balancer
Dispositivo que distribuye tráfico de red entre múltiples servidores para optimizar recursos y disponibilidad.

### NAT (Network Address Translation)
Técnica que permite que múltiples dispositivos compartan una única dirección IP pública.

### Port
Punto de conexión lógico en un dispositivo de red. Ejemplos: 80 (HTTP), 443 (HTTPS), 22 (SSH).

### Proxy
Servidor intermediario que actúa como gateway entre clientes y otros servidores.

### Router
Dispositivo que reenvía paquetes de datos entre redes.

### SSL/TLS (Secure Sockets Layer / Transport Layer Security)
Protocolos criptográficos para comunicaciones seguras por internet.

### Subnet (Subred)
División lógica de una red IP más grande.

### Switch
Dispositivo que conecta múltiples dispositivos en una LAN y reenvía datos al destino correcto.

### TCP/IP (Transmission Control Protocol / Internet Protocol)
Conjunto de protocolos de comunicación fundamentales de internet.

### VPN (Virtual Private Network)
Red privada que se extiende a través de una red pública, permitiendo comunicación segura.

### WAN (Wide Area Network)
Red que abarca un área geográfica amplia, conectando múltiples LANs.

### VLAN (Virtual LAN)
Segmentación lógica de una LAN física en múltiples redes virtuales aisladas.

---

## ☁️ Cloud Computing

### Auto-scaling
Capacidad de ajustar automáticamente recursos computacionales basado en demanda.

### Elastic Compute
Capacidad de escalar recursos de cómputo hacia arriba o abajo según necesidad.

### IaaS (Infrastructure as a Service)
Modelo cloud que provee infraestructura virtualizada (servidores, storage, redes).

### Multi-cloud
Uso de servicios de múltiples proveedores cloud simultáneamente.

### PaaS (Platform as a Service)
Modelo cloud que provee plataforma para desarrollo y despliegue sin gestionar infraestructura.

### Region
Ubicación geográfica donde un proveedor cloud tiene datacenters.

### SaaS (Software as a Service)
Modelo cloud donde aplicaciones se entregan completamente como servicio.

### Serverless
Modelo de ejecución donde el proveedor gestiona la infraestructura automáticamente.

### Availability Zone
Datacenter aislado dentro de una región cloud para alta disponibilidad.

### Edge Computing
Procesamiento de datos cerca de donde se generan en lugar de en datacenters centralizados.

### Hybrid Cloud
Combinación de infraestructura on-premise con servicios cloud.

### Public Cloud
Servicios cloud disponibles al público general a través de internet.

### Private Cloud
Infraestructura cloud dedicada exclusivamente a una organización.

### Vendor Lock-in
Dependencia de un proveedor específico que dificulta cambiar a alternativas.

---

## 🐳 Contenedores y Orquestación

### Container (Contenedor)
Unidad estándar de software que empaqueta código y dependencias para ejecutarse de forma consistente.

### Container Image
Plantilla inmutable que contiene todo lo necesario para ejecutar un contenedor.

### Container Registry
Repositorio para almacenar y distribuir imágenes de contenedores. Ejemplos: Docker Hub, ECR, GCR.

### Docker
Plataforma para desarrollar, enviar y ejecutar aplicaciones en contenedores.

### Dockerfile
Archivo de texto con instrucciones para construir una imagen Docker.

### Docker Compose
Herramienta para definir y ejecutar aplicaciones multi-contenedor.

### Kubernetes (K8s)
Sistema de orquestación de contenedores para automatizar despliegue, escalado y gestión.

### Pod
Unidad mínima de despliegue en Kubernetes, puede contener uno o más contenedores.

### Node
Máquina worker en un cluster de Kubernetes donde se ejecutan pods.

### Deployment
Objeto de Kubernetes que gestiona réplicas de pods y actualizaciones.

### Service
Abstracción en Kubernetes que define acceso lógico a un conjunto de pods.

### Namespace
Agrupación lógica de recursos en Kubernetes para organización y aislamiento.

### Helm
Gestor de paquetes para Kubernetes que facilita instalación de aplicaciones.

### Ingress
Objeto de Kubernetes que gestiona acceso HTTP/HTTPS externo a servicios.

### ConfigMap
Objeto de Kubernetes para almacenar configuración en pares clave-valor.

### Secret
Objeto de Kubernetes para almacenar información sensible (contraseñas, tokens).

---

## 🔒 Seguridad

### Authentication (Autenticación)
Proceso de verificar la identidad de un usuario o sistema.

### Authorization (Autorización)
Proceso de determinar qué acciones puede realizar un usuario autenticado.

### Certificate (Certificado)
Documento digital que verifica la identidad de una entidad y contiene su clave pública.

### Encryption (Cifrado)
Proceso de codificar información para que solo personas autorizadas puedan acceder.

### IAM (Identity and Access Management)
Sistema para gestionar identidades digitales y controlar acceso a recursos.

### IDS (Intrusion Detection System)
Sistema que monitorea tráfico de red en busca de actividad sospechosa.

### IPS (Intrusion Prevention System)
Sistema que detecta Y bloquea actividad maliciosa en tiempo real.

### MFA (Multi-Factor Authentication)
Método de autenticación que requiere múltiples formas de verificación.

### OAuth
Protocolo de autorización que permite acceso delegado seguro.

### Penetration Testing (Pentesting)
Prueba autorizada de seguridad que simula ataques para encontrar vulnerabilidades.

### PKI (Public Key Infrastructure)
Framework para crear, gestionar y revocar certificados digitales.

### RBAC (Role-Based Access Control)
Control de acceso basado en roles asignados a usuarios.

### SSH (Secure Shell)
Protocolo para acceso remoto seguro a sistemas.

### SSL Certificate
Certificado digital que autentica identidad y habilita conexiones cifradas.

### Vulnerability (Vulnerabilidad)
Debilidad en un sistema que puede ser explotada para comprometer seguridad.

### WAF (Web Application Firewall)
Firewall especializado en proteger aplicaciones web de ataques.

### Zero Trust
Modelo de seguridad que no confía en nada automáticamente y verifica todo.

---

## 📊 Monitoreo y Observabilidad

### Alert (Alerta)
Notificación automática cuando una métrica excede un umbral definido.

### Dashboard
Interfaz visual que muestra métricas y estado de sistemas en tiempo real.

### Log
Registro cronológico de eventos en un sistema.

### Metrics (Métricas)
Mediciones numéricas del comportamiento del sistema (CPU, memoria, latencia, etc.).

### Monitoring (Monitoreo)
Proceso continuo de recolectar y analizar datos sobre el estado del sistema.

### Observability (Observabilidad)
Capacidad de entender el estado interno de un sistema basándose en sus salidas.

### Trace (Traza)
Seguimiento de una petición a través de múltiples servicios en un sistema distribuido.

### APM (Application Performance Monitoring)
Monitoreo del rendimiento y disponibilidad de aplicaciones.

### SLI (Service Level Indicator)
Métrica cuantificable del nivel de servicio.

### SLO (Service Level Objective)
Objetivo específico para un SLI (ej: latencia < 100ms en 99% de peticiones).

---

## 🔧 DevOps y Automatización

### Ansible
Herramienta de automatización para gestión de configuración y despliegue.

### CI/CD (Continuous Integration / Continuous Deployment)
Prácticas de integrar código frecuentemente y desplegar automáticamente.

### Git
Sistema de control de versiones distribuido para código fuente.

### GitOps
Práctica de usar Git como fuente única de verdad para infraestructura y aplicaciones.

### IaC (Infrastructure as Code)
Gestión de infraestructura mediante código y archivos de configuración.

### Jenkins
Servidor de automatización open-source para CI/CD.

### Pipeline
Serie automatizada de pasos para construir, probar y desplegar software.

### Terraform
Herramienta IaC para provisionar infraestructura en múltiples proveedores.

### Blue-Green Deployment
Estrategia de despliegue con dos entornos idénticos para minimizar downtime.

### Canary Deployment
Despliegue gradual a un subconjunto de usuarios antes del rollout completo.

### Rolling Update
Actualización gradual de instancias sin downtime total.

---

## 🗄️ Bases de Datos

### ACID
Propiedades de transacciones en bases de datos: Atomicity, Consistency, Isolation, Durability.

### Database Replication
Copia de datos de una base de datos a otra para redundancia y rendimiento.

### NoSQL
Bases de datos no relacionales. Tipos: documento (MongoDB), clave-valor (Redis), columnar (Cassandra), grafo (Neo4j).

### RDBMS (Relational Database Management System)
Sistema de gestión de bases de datos relacionales. Ejemplos: MySQL, PostgreSQL, Oracle.

### Read Replica
Copia de solo lectura de una base de datos para distribuir carga de lectura.

### Sharding
Particionamiento horizontal de datos entre múltiples bases de datos.

### SQL (Structured Query Language)
Lenguaje estándar para gestionar bases de datos relacionales.

### Transaction
Secuencia de operaciones de base de datos que se ejecutan como una unidad atómica.

---

## 📖 Guía de Uso

### Cómo usar este glosario:

1. **Búsqueda rápida**: Usa Ctrl+F / Cmd+F para buscar términos específicos
2. **Aprendizaje progresivo**: Lee categorías completas para entender conceptos relacionados
3. **Referencia**: Marca esta página para consultar términos desconocidos
4. **Contexto**: Los términos están ordenados alfabéticamente dentro de cada categoría

### Niveles de conocimiento sugeridos:

- **🟢 Básico**: Términos fundamentales que todo profesional IT debe conocer
- **🟡 Intermedio**: Términos para roles especializados en infraestructura
- **🔴 Avanzado**: Términos para arquitectos y especialistas

## 📚 Recursos Adicionales

Para profundizar en estos conceptos:

- [Conceptos Básicos](./conceptos-basicos.md) - Explicaciones detalladas de fundamentos
- Documentación oficial de tecnologías específicas
- Certificaciones: AWS, Azure, GCP, CKAD, CKA

---

[⬅️ Volver a Fundamentos](./README.md) | [⬅️ Ir a Conceptos Básicos](./conceptos-basicos.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
