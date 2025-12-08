# Dockerfile: Creación de Imágenes Personalizadas

## 📖 ¿Qué es un Dockerfile?

Un Dockerfile es un archivo de texto que contiene todas las instrucciones necesarias para construir una imagen Docker. Es como una receta que Docker sigue paso a paso para crear tu imagen personalizada.

```
Dockerfile → docker build → Docker Image → docker run → Container
```

---

## 📝 Estructura Básica

### Ejemplo Mínimo

```dockerfile
# Imagen base
FROM ubuntu:20.04

# Ejecutar comandos
RUN apt-get update && apt-get install -y nginx

# Comando por defecto
CMD ["nginx", "-g", "daemon off;"]
```

### Construcción

```bash
# Sintaxis básica
docker build -t nombre-imagen:tag .

# Ejemplo
docker build -t mi-nginx:v1.0 .
```

---

## 🔨 Instrucciones Principales

### 1. FROM - Imagen Base

Define la imagen base desde la cual construir.

```dockerfile
# Imagen oficial de un lenguaje/runtime
FROM node:16

# Imagen base ligera
FROM alpine:3.16

# Imagen específica con tag
FROM python:3.10-slim

# Multi-stage (lo veremos más adelante)
FROM node:16 AS builder
```

**Mejores prácticas:**
- ✅ Usar imágenes oficiales cuando sea posible
- ✅ Especificar tags exactos (no usar `latest`)
- ✅ Preferir imágenes `slim` o `alpine` para menor tamaño

### 2. WORKDIR - Directorio de Trabajo

Establece el directorio de trabajo dentro del contenedor.

```dockerfile
# Crear y establecer directorio de trabajo
WORKDIR /app

# Todos los comandos siguientes se ejecutarán en /app
COPY package.json ./
RUN npm install
```

**Sin WORKDIR (❌ malo):**
```dockerfile
RUN cd /app
COPY package.json /app/
RUN cd /app && npm install
```

**Con WORKDIR (✅ bueno):**
```dockerfile
WORKDIR /app
COPY package.json ./
RUN npm install
```

### 3. COPY vs ADD - Copiar Archivos

Copian archivos desde el host al contenedor.

#### COPY (recomendado)

```dockerfile
# Copiar un archivo
COPY index.html /usr/share/nginx/html/

# Copiar múltiples archivos
COPY package.json package-lock.json ./

# Copiar todo el directorio
COPY ./src /app/src

# Copiar con permisos específicos
COPY --chown=user:group file.txt /app/
```

#### ADD (menos común)

ADD tiene funcionalidades extra:
- Descomprime archivos tar automáticamente
- Puede descargar archivos desde URLs

```dockerfile
# ADD descomprime automáticamente
ADD archive.tar.gz /app/

# Descargar desde URL (no recomendado, usar RUN curl)
ADD https://example.com/file.txt /app/
```

**⚠️ Preferir COPY sobre ADD** salvo que necesites las funcionalidades extras.

### 4. RUN - Ejecutar Comandos

Ejecuta comandos durante la construcción de la imagen.

```dockerfile
# Shell form (ejecuta en /bin/sh -c)
RUN apt-get update && apt-get install -y nginx

# Exec form (recomendado para comandos complejos)
RUN ["apt-get", "update"]

# Múltiples comandos (concatenar con &&)
RUN apt-get update && \
    apt-get install -y \
        python3 \
        python3-pip \
        git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Mejores prácticas:**
- ✅ Combinar comandos relacionados en un solo RUN
- ✅ Limpiar caché de paquetes en el mismo RUN
- ✅ Usar `\` para dividir comandos largos

### 5. ENV - Variables de Entorno

Define variables de entorno.

```dockerfile
# Sintaxis básica
ENV NODE_ENV=production

# Múltiples variables
ENV PORT=3000 \
    HOST=0.0.0.0 \
    DEBUG=false

# Usar variables
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

**Uso en runtime:**
```bash
# Sobrescribir en runtime
docker run -e NODE_ENV=development mi-app
```

### 6. ARG - Argumentos de Build

Variables solo disponibles durante el build.

```dockerfile
# Definir ARG
ARG NODE_VERSION=16
FROM node:${NODE_VERSION}

ARG APP_PORT=3000
ENV PORT=${APP_PORT}

# Con valor por defecto
ARG BUILD_DATE=unknown
```

**Uso:**
```bash
# Pasar argumentos en build time
docker build --build-arg NODE_VERSION=18 -t mi-app .
docker build --build-arg BUILD_DATE=$(date) -t mi-app .
```

**Diferencia ENV vs ARG:**
- `ARG`: Solo durante build
- `ENV`: Durante build Y runtime

### 7. EXPOSE - Documentar Puertos

Documenta qué puertos escucha el contenedor.

```dockerfile
# Documentar puerto
EXPOSE 80

# Múltiples puertos
EXPOSE 80 443

# Puerto con protocolo
EXPOSE 8080/tcp
EXPOSE 8080/udp
```

**⚠️ Nota:** EXPOSE es solo documentación, no publica el puerto. Debes usar `-p` al ejecutar:

```bash
docker run -p 8080:80 mi-app
```

### 8. CMD vs ENTRYPOINT - Comando de Ejecución

#### CMD - Comando Por Defecto

```dockerfile
# Exec form (recomendado)
CMD ["nginx", "-g", "daemon off;"]

# Shell form
CMD nginx -g "daemon off;"

# Parámetros para ENTRYPOINT
CMD ["--help"]
```

#### ENTRYPOINT - Punto de Entrada

```dockerfile
# Comando que siempre se ejecuta
ENTRYPOINT ["python", "app.py"]

# Combinado con CMD para parámetros por defecto
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
```

#### Diferencia CMD vs ENTRYPOINT

**Solo CMD:**
```dockerfile
FROM ubuntu
CMD ["echo", "Hello"]
```
```bash
docker run mi-imagen                # Output: Hello
docker run mi-imagen echo "Bye"     # Output: Bye (sobrescribe CMD)
```

**ENTRYPOINT + CMD:**
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello"]
```
```bash
docker run mi-imagen                # Output: Hello
docker run mi-imagen "Bye"          # Output: Bye (sobrescribe solo CMD)
```

### 9. VOLUME - Volúmenes

Declara puntos de montaje para datos persistentes.

```dockerfile
# Declarar volumen
VOLUME /var/lib/mysql

# Múltiples volúmenes
VOLUME ["/data", "/logs"]
```

### 10. USER - Usuario de Ejecución

Cambia el usuario que ejecuta los comandos.

```dockerfile
# Crear usuario
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Cambiar a usuario no-root
USER appuser

# Todos los comandos siguientes se ejecutan como appuser
WORKDIR /app
CMD ["./app"]
```

**⚠️ Seguridad:** Nunca ejecutes aplicaciones como root en producción.

---

## 🔧 Ejemplos Prácticos

### Aplicación Node.js

```dockerfile
# Usar imagen base oficial
FROM node:16-alpine

# Crear directorio de aplicación
WORKDIR /app

# Copiar archivos de dependencias
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production

# Copiar código fuente
COPY . .

# Exponer puerto
EXPOSE 3000

# Crear usuario no-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Comando de inicio
CMD ["node", "server.js"]
```

**Construcción y ejecución:**
```bash
docker build -t node-app:1.0 .
docker run -d -p 3000:3000 --name my-app node-app:1.0
```

### Aplicación Python

```dockerfile
FROM python:3.10-slim

# Variables de entorno para Python
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# Copiar e instalar dependencias de Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar aplicación
COPY . .

# Usuario no-root
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# Usar gunicorn en producción
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

### Aplicación Go

```dockerfile
FROM golang:1.19-alpine AS builder

WORKDIR /app

# Copiar módulos
COPY go.mod go.sum ./
RUN go mod download

# Copiar código y compilar
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Imagen final mínima
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copiar binario desde builder
COPY --from=builder /app/main .

EXPOSE 8080

CMD ["./main"]
```

---

## 🚀 Multi-Stage Builds

Reduce drásticamente el tamaño de la imagen final.

### Problema Sin Multi-Stage

```dockerfile
FROM node:16

WORKDIR /app
COPY package*.json ./
RUN npm install        # Instala dev + prod dependencies
COPY . .
RUN npm run build      # Build de producción

CMD ["npm", "start"]

# Imagen final: ~1GB (incluye dev dependencies, node_modules completo)
```

### Solución Con Multi-Stage

```dockerfile
# STAGE 1: Build
FROM node:16 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# STAGE 2: Production
FROM node:16-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist

USER node
CMD ["node", "dist/server.js"]

# Imagen final: ~150MB (solo archivos necesarios)
```

### Ejemplo Completo: Aplicación React

```dockerfile
# Build stage
FROM node:16-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copiar configuración personalizada de nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copiar build desde stage anterior
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf:**
```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

### Beneficios Multi-Stage

✅ **Tamaño reducido**: Solo binarios/archivos necesarios en imagen final
✅ **Seguridad**: No incluye herramientas de build
✅ **Simplicidad**: Todo en un Dockerfile
✅ **Cache eficiente**: Cada stage se cachea independientemente

---

## ⚡ Optimización de Imágenes

### 1. Orden de Instrucciones (Cache)

**❌ Malo (invalida cache frecuentemente):**
```dockerfile
FROM node:16
WORKDIR /app
COPY . .                    # Copia todo
RUN npm install             # Se ejecuta siempre que cambie cualquier archivo
CMD ["node", "server.js"]
```

**✅ Bueno (aprovecha cache):**
```dockerfile
FROM node:16
WORKDIR /app
COPY package*.json ./       # Solo copia dependencias
RUN npm install             # Se cachea si package.json no cambia
COPY . .                    # Copia código después
CMD ["node", "server.js"]
```

### 2. Minimizar Capas

**❌ Malo (muchas capas):**
```dockerfile
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y pip
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*
```

**✅ Bueno (una capa):**
```dockerfile
RUN apt-get update && \
    apt-get install -y python3 pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Usar .dockerignore

Equivalente a `.gitignore` para Docker.

**.dockerignore:**
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
*.log
dist
coverage
.vscode
```

**Beneficios:**
- ✅ Builds más rápidos
- ✅ Imágenes más pequeñas
- ✅ No copiar archivos sensibles

### 4. Imágenes Base Ligeras

```dockerfile
# ❌ Grande: ~900MB
FROM ubuntu:20.04

# ✅ Mediano: ~200MB
FROM node:16-slim

# ✅ Pequeño: ~150MB
FROM node:16-alpine

# ✅ Minimal: ~5MB (solo para binarios estáticos)
FROM scratch
COPY myapp /
CMD ["/myapp"]
```

### 5. Limpiar en el Mismo RUN

```dockerfile
# ❌ Malo (archivos quedan en capa anterior)
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get clean  # No reduce tamaño de capas anteriores

# ✅ Bueno (limpia en misma capa)
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

---

## 🛡️ Seguridad

### 1. No Usar Root

```dockerfile
# Crear usuario
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

# Cambiar ownership
RUN chown -R appuser:appgroup /app

# Cambiar a usuario
USER appuser
```

### 2. Escanear Vulnerabilidades

```bash
# Docker Scout (built-in)
docker scout cve mi-imagen

# Trivy
trivy image mi-imagen

# Snyk
snyk container test mi-imagen
```

### 3. No Incluir Secretos

**❌ Nunca hacer esto:**
```dockerfile
ENV DATABASE_PASSWORD=super_secret_123
COPY .env /app/.env
```

**✅ Pasar secretos en runtime:**
```bash
docker run -e DATABASE_PASSWORD=$DB_PASS mi-app
# O usar Docker secrets en Swarm/Kubernetes
```

### 4. Usar Imágenes Oficiales Verificadas

```dockerfile
# ✅ Imagen oficial verificada
FROM node:16-alpine

# ❌ Imagen de usuario desconocido
FROM random-user/node:latest
```

---

## 📊 Ejemplo Completo Optimizado

```dockerfile
# Multi-stage build para Node.js con TypeScript

# Build stage
FROM node:16-alpine AS builder

WORKDIR /app

# Copiar solo archivos de dependencias primero (cache)
COPY package*.json ./
COPY tsconfig.json ./

# Instalar todas las dependencias (incluyendo dev)
RUN npm ci

# Copiar código fuente
COPY ./src ./src

# Compilar TypeScript
RUN npm run build

# Production stage
FROM node:16-alpine

# Metadata
LABEL maintainer="tu-email@example.com"
LABEL version="1.0.0"

# Instalar dumb-init para mejor manejo de señales
RUN apk add --no-cache dumb-init

# Variables de entorno
ENV NODE_ENV=production \
    PORT=3000

WORKDIR /app

# Copiar solo package.json para instalar dependencias de producción
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# Copiar build desde stage anterior
COPY --from=builder /app/dist ./dist

# Crear usuario no-root
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup && \
    chown -R appuser:appgroup /app

# Cambiar a usuario no-root
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js || exit 1

# Exponer puerto
EXPOSE 3000

# Usar dumb-init como PID 1
ENTRYPOINT ["dumb-init", "--"]

# Comando de inicio
CMD ["node", "dist/server.js"]
```

**.dockerignore:**
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.env.local
.DS_Store
*.log
dist
coverage
.vscode
.idea
*.md
Dockerfile
docker-compose.yml
```

**Construcción:**
```bash
docker build -t myapp:1.0.0 .
```

---

## 🔍 Debugging y Troubleshooting

### Ver Historia de Imagen

```bash
# Ver capas de una imagen
docker history mi-imagen

# Con tamaños detallados
docker history --no-trunc mi-imagen
```

### Build con Output Detallado

```bash
# Ver todo el output
docker build --progress=plain -t mi-imagen .

# Sin usar cache
docker build --no-cache -t mi-imagen .

# Build hasta cierto stage
docker build --target builder -t mi-imagen .
```

### Inspeccionar Imagen

```bash
# Ver metadata completa
docker inspect mi-imagen

# Ver solo configuración
docker inspect --format='{{.Config}}' mi-imagen
```

---

## 📚 Próximos Pasos

Ahora que dominas Dockerfile:

1. [**Docker Compose**](./docker-compose.md) → Gestiona aplicaciones multi-contenedor
2. [**Networking**](./networking.md) → Comunicación entre contenedores
3. [**Volumes**](./volumes.md) → Persistencia de datos
4. [**Best Practices**](./best-practices.md) → Profundiza en optimización y seguridad

---

## 🔗 Recursos Adicionales

- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Best Practices Guide](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Multi-stage Builds](https://docs.docker.com/build/building/multi-stage/)

---

[⬅️ Volver: Introducción](./introduccion.md) | [➡️ Siguiente: Docker Compose](./docker-compose.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
