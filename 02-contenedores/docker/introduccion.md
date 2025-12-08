# Introducción a Docker

## 📖 ¿Qué son los Contenedores?

Los contenedores son unidades estándar de software que empaquetan código y todas sus dependencias para que la aplicación se ejecute de manera rápida y confiable de un entorno de computación a otro.

### Analogía del Mundo Real

Piensa en un contenedor como un contenedor de transporte marítimo:
- **Estándar**: Mismo tamaño y forma, compatible con cualquier barco/camión/grúa
- **Aislado**: El contenido está separado de otros contenedores
- **Portátil**: Puede moverse fácilmente entre diferentes medios de transporte
- **Predecible**: Sabes exactamente qué hay dentro

Los contenedores de software funcionan de la misma manera:
```
┌─────────────────────────────────────┐
│   Contenedor de Aplicación         │
├─────────────────────────────────────┤
│  Aplicación + Dependencias          │
│  • Código fuente                    │
│  • Librerías                        │
│  • Configuración                    │
│  • Runtime                          │
└─────────────────────────────────────┘
```

---

## 🐳 ¿Qué es Docker?

Docker es una plataforma de código abierto que permite automatizar el despliegue de aplicaciones dentro de contenedores de software.

### Componentes Principales

```
┌─────────────────────────────────────────────────┐
│             DOCKER PLATFORM                     │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐      ┌──────────────┐        │
│  │ Docker CLI   │─────>│ Docker Engine│        │
│  │ (Cliente)    │      │ (Daemon)     │        │
│  └──────────────┘      └──────┬───────┘        │
│                               │                 │
│                        ┌──────▼───────┐         │
│                        │ Containers   │         │
│                        │ Images       │         │
│                        │ Networks     │         │
│                        │ Volumes      │         │
│                        └──────────────┘         │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Docker Engine**: El motor que ejecuta y gestiona contenedores
**Docker CLI**: Interfaz de línea de comandos para interactuar con Docker
**Docker Hub**: Registro público de imágenes Docker (como GitHub para contenedores)

---

## 🆚 Contenedores vs Máquinas Virtuales

### Máquinas Virtuales (VMs)

```
┌─────────────────────────────────────┐
│           Servidor Físico           │
├─────────────────────────────────────┤
│          Hardware                   │
├─────────────────────────────────────┤
│      Sistema Operativo Host         │
├─────────────────────────────────────┤
│         Hypervisor                  │
├──────────┬──────────┬───────────────┤
│   VM 1   │   VM 2   │    VM 3      │
│ Guest OS │ Guest OS │  Guest OS    │
│   App A  │   App B  │    App C     │
└──────────┴──────────┴──────────────┘
```

### Contenedores

```
┌─────────────────────────────────────┐
│           Servidor Físico           │
├─────────────────────────────────────┤
│          Hardware                   │
├─────────────────────────────────────┤
│      Sistema Operativo Host         │
├─────────────────────────────────────┤
│         Docker Engine               │
├──────────┬──────────┬───────────────┤
│  Cont. 1 │  Cont. 2 │   Cont. 3    │
│   App A  │   App B  │    App C     │
│   + Deps │   + Deps │    + Deps    │
└──────────┴──────────┴──────────────┘
```

### Comparación

| Característica | Contenedores | Máquinas Virtuales |
|----------------|--------------|-------------------|
| **Tamaño** | MB (megabytes) | GB (gigabytes) |
| **Inicio** | Segundos | Minutos |
| **Aislamiento** | Proceso | Completo (OS) |
| **Overhead** | Bajo | Alto |
| **Portabilidad** | Alta | Media |
| **Densidad** | 100s por host | 10s por host |

### Cuándo usar cada uno

**Contenedores:**
- ✅ Microservicios
- ✅ CI/CD pipelines
- ✅ Aplicaciones cloud-native
- ✅ Desarrollo local que replica producción
- ✅ Escalado horizontal rápido

**Máquinas Virtuales:**
- ✅ Ejecutar diferentes sistemas operativos
- ✅ Aplicaciones legacy que requieren OS completo
- ✅ Aislamiento de seguridad estricto
- ✅ Cuando necesitas kernel diferente

---

## 🏗️ Arquitectura de Docker

### Componentes Detallados

```
┌────────────────────────────────────────────────┐
│  Docker Client (docker)                        │
│  • docker run                                  │
│  • docker build                                │
│  • docker pull                                 │
└────────────┬───────────────────────────────────┘
             │ REST API
┌────────────▼───────────────────────────────────┐
│  Docker Daemon (dockerd)                       │
│  ┌──────────────────────────────────────────┐ │
│  │  Container Runtime                       │ │
│  │  • containerd                            │ │
│  │  • runc                                  │ │
│  └──────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────┐ │
│  │  Images                                  │ │
│  │  • Layers (read-only)                    │ │
│  │  • Union file system                     │ │
│  └──────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────┐ │
│  │  Containers                              │ │
│  │  • Running instances                     │ │
│  │  • Writable layer                        │ │
│  └──────────────────────────────────────────┘ │
└────────────┬───────────────────────────────────┘
             │
┌────────────▼───────────────────────────────────┐
│  Docker Registry (Docker Hub, ECR, etc.)       │
│  • Public images                               │
│  • Private images                              │
└────────────────────────────────────────────────┘
```

### Flujo de Trabajo Típico

1. **Escribir** → Crear un Dockerfile
2. **Build** → Construir una imagen
3. **Push** → Subir imagen a registry (opcional)
4. **Pull** → Descargar imagen desde registry
5. **Run** → Ejecutar contenedor desde imagen

---

## 💡 Conceptos Clave

### 1. Imagen (Image)

Una imagen es una plantilla de solo lectura con instrucciones para crear un contenedor.

**Características:**
- Inmutable (no cambia)
- Compuesta por capas (layers)
- Se construye desde un Dockerfile
- Se almacena en registries

**Ejemplo:**
```bash
# Listar imágenes locales
docker images

# Ejemplo de salida:
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    605c77e624dd   2 weeks ago    141MB
postgres      14        a6a41bbfd9de   3 weeks ago    376MB
```

### 2. Contenedor (Container)

Un contenedor es una instancia ejecutable de una imagen.

**Características:**
- Tiene estado (running, stopped, etc.)
- Puede ser modificado
- Aislado de otros contenedores
- Tiene su propia capa de escritura

**Ejemplo:**
```bash
# Listar contenedores en ejecución
docker ps

# Listar todos los contenedores (incluyendo detenidos)
docker ps -a
```

### 3. Capas (Layers)

Las imágenes están compuestas por capas apiladas:

```
┌─────────────────────────┐
│  CMD ["nginx"]          │  ← Capa 4 (Comando)
├─────────────────────────┤
│  COPY index.html        │  ← Capa 3 (Archivos)
├─────────────────────────┤
│  RUN apt-get install    │  ← Capa 2 (Paquetes)
├─────────────────────────┤
│  FROM ubuntu:20.04      │  ← Capa 1 (Base)
└─────────────────────────┘
```

**Ventajas:**
- Reutilización de capas entre imágenes
- Eficiencia en almacenamiento
- Builds más rápidos (cache)

### 4. Registry

Almacenamiento y distribución de imágenes Docker.

**Opciones populares:**
- **Docker Hub**: Registry público oficial
- **Amazon ECR**: Elastic Container Registry (AWS)
- **Google GCR**: Google Container Registry
- **Azure ACR**: Azure Container Registry
- **Harbor**: Open source, auto-hospedado

---

## 🎯 Casos de Uso

### 1. Desarrollo Local

**Problema**: "En mi máquina funciona"

**Solución con Docker**:
```bash
# Todos los desarrolladores usan el mismo entorno
docker run -v $(pwd):/app -p 3000:3000 node:16 npm start
```

### 2. Microservicios

Cada servicio en su propio contenedor:
```
┌──────────┐  ┌──────────┐  ┌──────────┐
│  API     │  │  Auth    │  │ Database │
│Container │─>│Container │  │Container │
└──────────┘  └──────────┘  └──────────┘
```

### 3. CI/CD

Pipeline de integración continua:
```bash
# Build
docker build -t myapp:latest .

# Test
docker run myapp:latest npm test

# Deploy
docker push myapp:latest
```

### 4. Entornos de Testing

Crear y destruir entornos rápidamente:
```bash
# Levantar base de datos de prueba
docker run -d -e POSTGRES_PASSWORD=test postgres:14

# Ejecutar tests
npm test

# Limpiar
docker rm -f $(docker ps -aq)
```

### 5. Aplicaciones Legacy

Empaquetar aplicaciones antiguas con sus dependencias específicas:
```dockerfile
FROM python:2.7  # Python 2.7 ya no está soportado
COPY legacy-app /app
RUN pip install -r requirements.txt
```

---

## 🚀 Instalación

### Linux (Ubuntu/Debian)

```bash
# Actualizar repositorios
sudo apt-get update

# Instalar dependencias
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Agregar clave GPG oficial de Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Agregar repositorio
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Agregar usuario al grupo docker (evita usar sudo)
sudo usermod -aG docker $USER
```

### macOS

```bash
# Opción 1: Docker Desktop (recomendado)
# Descargar desde: https://www.docker.com/products/docker-desktop

# Opción 2: Homebrew
brew install --cask docker
```

### Windows

1. **Windows 10/11 Pro/Enterprise:**
   - Descargar Docker Desktop desde: https://www.docker.com/products/docker-desktop
   - Requiere WSL 2 (Windows Subsystem for Linux)

2. **Windows 10/11 Home:**
   - Actualizar a WSL 2
   - Instalar Docker Desktop con WSL 2 backend

### Verificar Instalación

```bash
# Verificar versión
docker --version
# Output: Docker version 24.0.0, build xyz

# Ejecutar contenedor de prueba
docker run hello-world

# Verificar información del sistema
docker info
```

---

## 🔧 Primeros Pasos

### Comando Básico: docker run

```bash
# Sintaxis básica
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# Ejemplo: Ejecutar nginx
docker run nginx

# Con opciones comunes
docker run -d -p 8080:80 --name mi-nginx nginx
```

**Opciones importantes:**
- `-d`: Modo detached (background)
- `-p HOST:CONTAINER`: Mapeo de puertos
- `--name`: Nombre del contenedor
- `-v`: Montar volúmenes
- `-e`: Variables de entorno
- `--rm`: Eliminar contenedor al parar

### Ejemplo Práctico: Servidor Web

```bash
# 1. Ejecutar nginx en puerto 8080
docker run -d -p 8080:80 --name webserver nginx

# 2. Verificar que está corriendo
docker ps

# 3. Acceder en navegador: http://localhost:8080

# 4. Ver logs
docker logs webserver

# 5. Detener contenedor
docker stop webserver

# 6. Eliminar contenedor
docker rm webserver
```

### Ejemplo Práctico: Base de Datos

```bash
# Ejecutar PostgreSQL
docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:14

# Conectar a la base de datos
docker exec -it postgres-db psql -U postgres myapp
```

### Comandos Esenciales

```bash
# GESTIÓN DE CONTENEDORES
docker ps                    # Listar contenedores en ejecución
docker ps -a                 # Listar todos los contenedores
docker start CONTAINER       # Iniciar contenedor
docker stop CONTAINER        # Detener contenedor
docker restart CONTAINER     # Reiniciar contenedor
docker rm CONTAINER          # Eliminar contenedor
docker logs CONTAINER        # Ver logs
docker exec -it CONTAINER bash  # Acceder al contenedor

# GESTIÓN DE IMÁGENES
docker images                # Listar imágenes
docker pull IMAGE            # Descargar imagen
docker rmi IMAGE             # Eliminar imagen
docker build -t NAME .       # Construir imagen
docker tag IMAGE NEW_NAME    # Etiquetar imagen

# LIMPIEZA
docker system prune          # Limpiar recursos no usados
docker container prune       # Eliminar contenedores detenidos
docker image prune           # Eliminar imágenes sin usar
docker volume prune          # Eliminar volúmenes sin usar
```

---

## 📊 Ejemplo Completo: Aplicación Node.js

### 1. Aplicación Simple

**app.js:**
```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('¡Hola desde Docker!');
});

app.listen(port, '0.0.0.0', () => {
  console.log(`App escuchando en http://localhost:${port}`);
});
```

**package.json:**
```json
{
  "name": "docker-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

### 2. Ejecutar Sin Docker (tradicional)

```bash
npm install
node app.js
```

### 3. Ejecutar Con Docker

```bash
# Usar imagen oficial de Node.js
docker run -d \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  node:16 \
  sh -c "npm install && node app.js"

# Acceder: http://localhost:3000
```

---

## 💡 Ventajas de Docker

### Para Desarrolladores

✅ **Entorno consistente**: Mismo entorno en desarrollo, testing y producción
✅ **Onboarding rápido**: Nuevos desarrolladores productivos en minutos
✅ **Aislamiento**: Diferentes versiones de dependencias sin conflictos
✅ **Reproducibilidad**: "Funciona en mi máquina" → "Funciona en cualquier máquina"

### Para Operaciones

✅ **Despliegue rápido**: Segundos vs minutos/horas
✅ **Escalabilidad**: Fácil escalar horizontalmente
✅ **Eficiencia de recursos**: Más aplicaciones en mismo hardware
✅ **Portabilidad**: Mismo contenedor en cualquier cloud o on-premise

### Para la Organización

✅ **Reducción de costos**: Mejor utilización de recursos
✅ **Faster time to market**: Ciclos de desarrollo más rápidos
✅ **Microservicios**: Facilita arquitecturas modernas
✅ **DevOps**: Integración perfecta con CI/CD

---

## ⚠️ Consideraciones y Limitaciones

### Desventajas

❌ **Curva de aprendizaje**: Requiere entender nuevos conceptos
❌ **Complejidad**: Puede ser excesivo para aplicaciones simples
❌ **Debugging**: Más difícil que en entornos tradicionales
❌ **Persistencia**: Requiere gestión cuidadosa de datos
❌ **Seguridad**: Nuevas vulnerabilidades a considerar

### No es una Bala de Plata

Docker no resuelve:
- Código mal escrito
- Arquitectura deficiente
- Falta de tests
- Problemas de performance inherentes a la aplicación

---

## 📚 Próximos Pasos

Ahora que entiendes los fundamentos de Docker:

1. [**Dockerfile**](./dockerfile.md) → Aprende a crear tus propias imágenes
2. [**Docker Compose**](./docker-compose.md) → Gestiona aplicaciones multi-contenedor
3. [**Networking**](./networking.md) → Comunicación entre contenedores
4. [**Volumes**](./volumes.md) → Persistencia de datos
5. [**Best Practices**](./best-practices.md) → Optimización y seguridad

---

## 🔗 Recursos Adicionales

- [Documentación Oficial de Docker](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/) - Explorar imágenes públicas
- [Play with Docker](https://labs.play-with-docker.com/) - Playground online gratuito
- [Docker Cheat Sheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)

---

[⬅️ Volver a Contenedores](../README.md) | [➡️ Siguiente: Dockerfile](./dockerfile.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
