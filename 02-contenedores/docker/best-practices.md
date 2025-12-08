# Mejores Prácticas de Docker

Esta guía compila las mejores prácticas para usar Docker en producción, cubriendo seguridad, performance, optimización y troubleshooting.

---

## 🛡️ Seguridad

### 1. Nunca Ejecutar como Root

**❌ Malo:**
```dockerfile
FROM node:16
COPY . /app
CMD ["node", "server.js"]
# Corre como root (UID 0)
```

**✅ Bueno:**
```dockerfile
FROM node:16
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Crear usuario no privilegiado
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 appuser && \
    chown -R appuser:nodejs /app

USER appuser

CMD ["node", "server.js"]
```

### 2. Escanear Vulnerabilidades

```bash
# Docker Scout (built-in en Docker Desktop)
docker scout cve mi-imagen:latest

# Trivy (recomendado)
docker run aquasec/trivy image mi-imagen:latest

# Snyk
snyk container test mi-imagen:latest

# Incluir en CI/CD
# .github/workflows/security.yml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    severity: 'CRITICAL,HIGH'
```

### 3. Usar Imágenes Oficiales y Verificadas

```dockerfile
# ✅ Imagen oficial verificada
FROM node:16-alpine

# ✅ Con digest para inmutabilidad
FROM node:16-alpine@sha256:a6c...

# ❌ Imagen no verificada
FROM randomuser/node:latest
```

### 4. Minimizar Superficie de Ataque

```dockerfile
# Multi-stage build
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Imagen final minimalista
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

# Eliminar utilidades innecesarias
RUN apk del apk-tools && \
    rm -rf /var/cache/apk/*

USER node
CMD ["node", "dist/server.js"]
```

### 5. No Incluir Secretos en Imágenes

**❌ Malo:**
```dockerfile
ENV DATABASE_PASSWORD=supersecret123
COPY .env /app/.env
```

**✅ Bueno:**
```bash
# Pasar en runtime
docker run -e DATABASE_PASSWORD=$DB_PASS mi-app

# Docker secrets (Swarm/Kubernetes)
docker secret create db_password ./password.txt

# Docker Compose con secrets
docker-compose.yml:
secrets:
  db_password:
    file: ./secrets/db_password.txt

services:
  app:
    secrets:
      - db_password
```

### 6. Limitar Recursos

```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

```bash
# Docker run
docker run -d \
  --cpus=".5" \
  --memory="512m" \
  --memory-reservation="256m" \
  mi-app
```

### 7. Read-Only Filesystem

```yaml
services:
  app:
    image: myapp
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

### 8. Drop Capabilities

```yaml
services:
  app:
    image: myapp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE  # Solo si necesita puertos < 1024
```

### 9. Security Scanning en CI/CD

```yaml
# .github/workflows/docker.yml
name: Docker Security

on: [push]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Run Trivy vulnerability scanner
        run: |
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy image --severity HIGH,CRITICAL myapp:${{ github.sha }}

      - name: Run Hadolint
        run: docker run --rm -i hadolint/hadolint < Dockerfile
```

---

## ⚡ Performance y Optimización

### 1. Optimizar Capas de Imagen

**❌ Malo - Muchas capas:**
```dockerfile
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y pip
RUN apt-get install -y git
RUN apt-get clean
```

**✅ Bueno - Pocas capas:**
```dockerfile
RUN apt-get update && \
    apt-get install -y \
        python3 \
        pip \
        git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 2. Aprovechar Build Cache

**❌ Malo - Cache ineficiente:**
```dockerfile
COPY . /app
RUN npm install
```

**✅ Bueno - Cache optimizado:**
```dockerfile
# Copiar solo archivos de dependencias primero
COPY package*.json ./
RUN npm ci

# Copiar código después
COPY . .
```

### 3. Usar .dockerignore

**.dockerignore:**
```
# Archivos de control de versiones
.git
.gitignore
.gitattributes

# Dependencias (se instalan en build)
node_modules
vendor
__pycache__

# Archivos de desarrollo
*.log
*.md
.env*
.vscode
.idea

# Archivos de build
dist
build
target

# Tests
test
tests
*.test.js
*.spec.js
coverage

# CI/CD
.github
.gitlab-ci.yml
Jenkinsfile

# Docker
Dockerfile*
docker-compose*
.dockerignore

# OS
.DS_Store
Thumbs.db
```

### 4. Multi-Stage Builds

```dockerfile
# Stage 1: Builder
FROM golang:1.19 AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Stage 2: Runtime (solo ~5MB)
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]

# Resultado: 1GB builder → 10MB imagen final
```

### 5. Imágenes Base Ligeras

| Imagen Base | Tamaño | Uso |
|-------------|--------|-----|
| `ubuntu:latest` | ~77MB | Desarrollo, debugging |
| `debian:slim` | ~69MB | Balance general |
| `alpine:latest` | ~5MB | Producción optimizada |
| `node:16-alpine` | ~110MB | Node.js ligero |
| `distroless/nodejs` | ~90MB | Solo runtime, sin shell |
| `scratch` | 0MB | Binarios estáticos Go/Rust |

```dockerfile
# Para Go/Rust - Imagen mínima absoluta
FROM scratch
COPY myapp /
CMD ["/myapp"]
```

### 6. BuildKit y Cache Mounts

```dockerfile
# syntax=docker/dockerfile:1

FROM node:16 AS builder

# Cache de npm en host
RUN --mount=type=cache,target=/root/.npm \
    npm install

# Cache de build
RUN --mount=type=cache,target=/app/.next/cache \
    npm run build
```

Habilitar BuildKit:
```bash
export DOCKER_BUILDKIT=1
docker build -t myapp .

# O permanente en /etc/docker/daemon.json
{
  "features": {
    "buildkit": true
  }
}
```

### 7. Compress Layers

```bash
# Usar --squash para combinar capas (experimental)
docker build --squash -t myapp:slim .

# O docker-slim para optimización automática
docker-slim build myapp:latest
```

---

## 📊 Monitoreo y Logging

### 1. Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### 2. Logging Best Practices

```dockerfile
# Logs a STDOUT/STDERR (no a archivos)
CMD ["node", "server.js"]

# NO esto:
CMD ["node", "server.js", ">", "/var/log/app.log"]
```

**Configurar logging driver:**
```yaml
services:
  app:
    image: myapp
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

**Logging centralizado:**
```yaml
services:
  app:
    image: myapp
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: myapp.logs
```

### 3. Métricas y Monitoreo

```yaml
# Prometheus + Grafana stack
version: '3.8'

services:
  app:
    image: myapp
    ports:
      - "3000:3000"
      - "9090:9090"  # Métricas

  prometheus:
    image: prom/prometheus
    ports:
      - "9091:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  prometheus-data:
  grafana-data:
```

---

## 🔧 Desarrollo y CI/CD

### 1. Development vs Production

**docker-compose.yml (base):**
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
```

**docker-compose.override.yml (dev - auto-cargado):**
```yaml
version: '3.8'
services:
  app:
    build:
      target: development
    volumes:
      - ./src:/app/src
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=*
    command: npm run dev
```

**docker-compose.prod.yml:**
```yaml
version: '3.8'
services:
  app:
    image: myapp:${VERSION}
    environment:
      - NODE_ENV=production
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
```

Uso:
```bash
# Desarrollo
docker-compose up

# Producción
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### 2. CI/CD Pipeline

**GitHub Actions:**
```yaml
# .github/workflows/docker.yml
name: Docker Build & Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Run tests
        run: |
          docker run --rm ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main-${{ github.sha }} npm test
```

### 3. Versionado de Imágenes

```bash
# Semántico
docker build -t myapp:1.2.3 .
docker build -t myapp:1.2 .
docker build -t myapp:1 .
docker build -t myapp:latest .

# Por commit
docker build -t myapp:${GIT_SHA} .

# Por branch + commit
docker build -t myapp:main-${GIT_SHA} .

# Con fecha
docker build -t myapp:$(date +%Y%m%d)-${GIT_SHA} .
```

### 4. Tagging Strategy

```bash
# Build con múltiples tags
docker build \
  -t myapp:latest \
  -t myapp:1.2.3 \
  -t myapp:1.2 \
  -t myapp:1 \
  -t ghcr.io/usuario/myapp:1.2.3 \
  .

# Push todos
docker push --all-tags ghcr.io/usuario/myapp
```

---

## 🔍 Troubleshooting

### 1. Debugging de Contenedores

```bash
# Ver logs
docker logs -f --tail=100 container-name

# Entrar a contenedor corriendo
docker exec -it container-name sh

# Ejecutar comando sin modificar contenedor
docker exec container-name ps aux

# Ver procesos
docker top container-name

# Inspeccionar configuración
docker inspect container-name

# Ver estadísticas en vivo
docker stats container-name

# Ver cambios en filesystem
docker diff container-name
```

### 2. Debugging de Imágenes

```bash
# Ver historia de capas
docker history myapp:latest

# Ver tamaño por capa
docker history --no-trunc --format "{{.Size}}\t{{.CreatedBy}}" myapp:latest

# Explorar imagen sin ejecutar
docker run --rm -it --entrypoint sh myapp:latest

# Crear contenedor sin iniciar
docker create --name temp myapp:latest
docker export temp | tar -t | less
docker rm temp
```

### 3. Problemas Comunes

#### Contenedor se Detiene Inmediatamente

```bash
# Ver logs
docker logs container-name

# Causa común: CMD termina inmediatamente
# ❌ Malo
CMD echo "Hello"

# ✅ Bueno
CMD ["node", "server.js"]  # Proceso long-running
```

#### Out of Memory

```bash
# Ver uso de memoria
docker stats

# Aumentar límite
docker run -m 1g myapp

# Ver OOM kills
dmesg | grep oom
```

#### Problemas de Permisos

```bash
# Ver UID/GID en contenedor
docker exec container-name id

# Ajustar permisos
chown -R 1000:1000 ./data

# Ejecutar como usuario específico
docker run --user 1000:1000 myapp
```

#### Build Muy Lento

```bash
# Ver qué se está copiando
docker build --progress=plain -t myapp .

# Verificar .dockerignore
# Asegurar que node_modules, .git, etc. estén excluidos
```

---

## 📋 Checklist de Producción

Antes de desplegar en producción:

### Seguridad
- [ ] No corre como root
- [ ] Imagen escaneada (sin vulnerabilidades críticas)
- [ ] Sin secretos en imagen
- [ ] Imagen oficial/verificada
- [ ] Read-only filesystem donde sea posible
- [ ] Capabilities mínimas
- [ ] Resource limits configurados

### Performance
- [ ] Multi-stage build
- [ ] Imagen optimizada (<500MB idealmente)
- [ ] .dockerignore configurado
- [ ] Capas minimizadas
- [ ] Build cache optimizado

### Operaciones
- [ ] Health checks configurados
- [ ] Logs a STDOUT/STDERR
- [ ] Logging driver configurado
- [ ] Restart policy apropiada
- [ ] Monitoreo configurado (Prometheus, etc.)
- [ ] Backups de volúmenes configurados

### CI/CD
- [ ] Build automático en CI
- [ ] Tests automáticos
- [ ] Security scanning en pipeline
- [ ] Versionado consistente
- [ ] Rollback strategy definida

---

## 🎯 Anti-Patterns a Evitar

### ❌ 1. Usar :latest en Producción

```yaml
# ❌ Malo
services:
  app:
    image: myapp:latest

# ✅ Bueno
services:
  app:
    image: myapp:1.2.3
```

### ❌ 2. Instalar Dependencias en Runtime

```dockerfile
# ❌ Malo
CMD npm install && node server.js

# ✅ Bueno
RUN npm ci --only=production
CMD ["node", "server.js"]
```

### ❌ 3. Actualizar Paquetes en Dockerfile

```dockerfile
# ❌ Malo (no reproducible)
RUN apt-get update && apt-get upgrade -y

# ✅ Bueno (usar imagen base actualizada)
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y specific-package
```

### ❌ 4. Muchos Procesos en Un Contenedor

```dockerfile
# ❌ Malo - Supervisor con multiple apps
CMD supervisord -c /etc/supervisor/conf.d/supervisord.conf

# ✅ Bueno - Un proceso por contenedor, orquestar con Compose
docker-compose.yml:
  services:
    web:
      image: nginx
    app:
      image: myapp
    worker:
      image: myapp
      command: worker
```

### ❌ 5. Hardcodear Configuración

```dockerfile
# ❌ Malo
ENV DATABASE_URL=postgres://prod-db:5432/app

# ✅ Bueno - Pasar en runtime
docker run -e DATABASE_URL=$DB_URL myapp
```

---

## 🔗 Recursos Adicionales

- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Security](https://docs.docker.com/engine/security/)
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

---

[⬅️ Volver: Volumes](./volumes.md) | [🏠 Volver a Contenedores](../README.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
