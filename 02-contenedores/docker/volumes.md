# Volúmenes en Docker: Persistencia de Datos

## 📖 El Problema de la Persistencia

Los contenedores son **efímeros** por naturaleza. Cuando eliminas un contenedor, todos los datos dentro de él se pierden.

```bash
# Crear contenedor con base de datos
docker run --name db postgres:14

# Insertar datos...

# Eliminar contenedor
docker rm db

# ❌ Todos los datos se perdieron!
```

**Solución:** Usar volúmenes para persistir datos fuera del ciclo de vida del contenedor.

---

## 🗂️ Tipos de Almacenamiento

Docker proporciona tres formas de persistir datos:

```
┌─────────────────────────────────────────────┐
│          Host Machine                       │
├─────────────────────────────────────────────┤
│                                             │
│  ┌────────────────┐    ┌──────────────┐    │
│  │    Volumes     │    │ Bind Mounts  │    │
│  │ /var/lib/docker│    │ /host/path   │    │
│  │   /volumes/    │    │              │    │
│  └───────┬────────┘    └──────┬───────┘    │
│          │                     │            │
│     ┌────▼─────────────────────▼────┐       │
│     │       Container                │      │
│     │   /data     /app/config        │      │
│     └────────────────────────────────┘      │
│                                             │
│  ┌────────────────┐                         │
│  │    tmpfs       │ (RAM)                   │
│  └────────────────┘                         │
│                                             │
└─────────────────────────────────────────────┘
```

### 1. Volumes (Recomendado ✅)

Gestionados completamente por Docker. Almacenados en `/var/lib/docker/volumes/`.

**Ventajas:**
- ✅ Gestión automática por Docker
- ✅ Pueden ser compartidos entre contenedores
- ✅ Funcionan en cualquier OS
- ✅ Fácil backup/restore
- ✅ Pueden usarse con volume drivers (cloud storage, etc.)

### 2. Bind Mounts

Montan un directorio específico del host en el contenedor.

**Ventajas:**
- ✅ Control total sobre la ubicación
- ✅ Útil para desarrollo (hot reload)

**Desventajas:**
- ❌ Dependientes del path del host
- ❌ Pueden crear problemas de permisos
- ❌ Menos portables

### 3. tmpfs Mounts

Almacenamiento en memoria RAM (volátil).

**Uso:**
- ✅ Datos temporales sensibles
- ✅ Alto rendimiento
- ❌ Se pierde al detener contenedor

---

## 📦 Trabajar con Volumes

### Crear y Gestionar Volumes

```bash
# Crear volume
docker volume create mi-volumen

# Listar volumes
docker volume ls

# Inspeccionar volume
docker volume inspect mi-volumen

# Salida:
[
    {
        "CreatedAt": "2023-10-10T10:00:00Z",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/mi-volumen/_data",
        "Name": "mi-volumen",
        "Scope": "local"
    }
]

# Eliminar volume
docker volume rm mi-volumen

# Limpiar volumes no usados
docker volume prune
```

### Usar Volumes en Contenedores

```bash
# Montar volume nombrado
docker run -d \
  --name db \
  -v mi-volumen:/var/lib/postgresql/data \
  postgres:14

# Docker crea el volume automáticamente si no existe
docker run -d -v datos:/app/data mi-app

# Volume anónimo (Docker genera nombre aleatorio)
docker run -d -v /app/data mi-app
```

### Montar en Modo Lectura

```bash
# Read-only mount
docker run -d -v mi-volumen:/data:ro nginx
```

---

## 🔗 Bind Mounts

### Sintaxis

```bash
# Sintaxis básica: -v HOST_PATH:CONTAINER_PATH
docker run -d \
  -v /path/on/host:/path/in/container \
  mi-imagen

# Ejemplo real: desarrollo web
docker run -d \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx

# Ahora puedes editar archivos en ./html y verlos en el navegador inmediatamente
```

### Formato Largo (Mount)

```bash
# Formato más explícito y recomendado
docker run -d \
  --mount type=bind,source=/host/path,target=/container/path,readonly \
  mi-imagen
```

### Ejemplo: Desarrollo Node.js con Hot Reload

```bash
# Montar código fuente para desarrollo
docker run -d \
  -p 3000:3000 \
  -v $(pwd):/app \
  -v /app/node_modules \
  -w /app \
  node:16 \
  npm run dev

# Explicación:
# -v $(pwd):/app           → Monta código fuente
# -v /app/node_modules     → Previene sobrescribir node_modules
# -w /app                  → Establece working directory
```

**docker-compose.yml equivalente:**
```yaml
version: '3.8'

services:
  app:
    image: node:16
    working_dir: /app
    volumes:
      - ./:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    command: npm run dev
```

---

## 💾 tmpfs Mounts

### Uso

```bash
# Montar en RAM
docker run -d \
  --tmpfs /app/cache \
  mi-app

# Con tamaño limitado
docker run -d \
  --tmpfs /app/cache:size=100m \
  mi-app

# Formato largo
docker run -d \
  --mount type=tmpfs,destination=/app/cache,tmpfs-size=100m \
  mi-app
```

**Casos de uso:**
- Caché temporal
- Datos sensibles que no deben tocarel disco
- Archivos temporales de procesamiento

---

## 🐳 Volumes en Docker Compose

### Ejemplo Completo

```yaml
version: '3.8'

services:
  # Base de datos con volume nombrado
  db:
    image: postgres:14
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    environment:
      POSTGRES_PASSWORD: example

  # Aplicación con bind mounts para desarrollo
  app:
    build: .
    volumes:
      # Código fuente (bind mount)
      - ./src:/app/src

      # Configuración read-only
      - ./config:/app/config:ro

      # Logs persistentes (volume nombrado)
      - app-logs:/app/logs

      # Cache en memoria
      - type: tmpfs
        target: /app/cache
        tmpfs:
          size: 100000000  # 100MB

  # Nginx con configuración
  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./static:/usr/share/nginx/html:ro

# Declarar volumes nombrados
volumes:
  postgres-data:
    driver: local
  app-logs:
    driver: local
```

### Compartir Volumes Entre Servicios

```yaml
version: '3.8'

services:
  writer:
    image: alpine
    volumes:
      - shared-data:/data
    command: sh -c "echo 'Hello' > /data/message.txt && sleep infinity"

  reader:
    image: alpine
    volumes:
      - shared-data:/data:ro  # Read-only
    command: sh -c "cat /data/message.txt && sleep infinity"

volumes:
  shared-data:
```

---

## 💼 Casos de Uso Prácticos

### 1. Base de Datos PostgreSQL

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      # Datos persistentes
      - postgres-data:/var/lib/postgresql/data

      # Scripts de inicialización
      - ./init-scripts:/docker-entrypoint-initdb.d:ro

      # Configuración personalizada
      - ./postgresql.conf:/etc/postgresql/postgresql.conf:ro

    # Usar configuración personalizada
    command: postgres -c config_file=/etc/postgresql/postgresql.conf

volumes:
  postgres-data:
```

### 2. WordPress + MySQL

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress

  wordpress:
    image: wordpress:latest
    depends_on:
      - db
    ports:
      - "8000:80"
    volumes:
      # Contenido de WordPress
      - wp-content:/var/www/html/wp-content

      # Uploads de usuarios
      - wp-uploads:/var/www/html/wp-content/uploads

      # Tema personalizado (desarrollo)
      - ./my-theme:/var/www/html/wp-content/themes/my-theme

    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_PASSWORD: rootpass

volumes:
  db-data:
  wp-content:
  wp-uploads:
```

### 3. Logs Centralizados

```yaml
version: '3.8'

services:
  app1:
    image: myapp:latest
    volumes:
      - logs:/var/log/app

  app2:
    image: myapp:latest
    volumes:
      - logs:/var/log/app

  log-collector:
    image: fluentd:latest
    volumes:
      - logs:/var/log/apps:ro
      - ./fluentd.conf:/fluentd/etc/fluent.conf:ro

volumes:
  logs:
```

### 4. Desarrollo con Hot Reload

```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    volumes:
      # Código fuente
      - ./frontend/src:/app/src

      # NO sobrescribir node_modules
      - /app/node_modules

      # Variables de entorno local
      - ./frontend/.env.local:/app/.env.local

    environment:
      - CHOKIDAR_USEPOLLING=true  # Para que funcione hot reload en Docker

    ports:
      - "3000:3000"

    command: npm run dev

  backend:
    build: ./backend
    volumes:
      - ./backend:/app
      - /app/node_modules

    ports:
      - "8000:8000"

    command: npm run dev
```

---

## 💾 Backup y Restore

### Backup de Volume

```bash
# Método 1: Usar contenedor temporal
docker run --rm \
  -v mi-volumen:/data \
  -v $(pwd):/backup \
  alpine \
  tar czf /backup/backup-$(date +%Y%m%d).tar.gz -C /data .

# Método 2: Docker cp (si contenedor está corriendo)
docker cp mi-contenedor:/var/lib/postgresql/data ./backup/
```

### Restore de Volume

```bash
# Crear volume si no existe
docker volume create mi-volumen

# Restaurar desde backup
docker run --rm \
  -v mi-volumen:/data \
  -v $(pwd):/backup \
  alpine \
  sh -c "cd /data && tar xzf /backup/backup-20231010.tar.gz"
```

### Script de Backup Automático

```bash
#!/bin/bash
# backup-volumes.sh

VOLUME_NAME=$1
BACKUP_DIR="./backups"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

docker run --rm \
  -v ${VOLUME_NAME}:/data:ro \
  -v $(pwd)/${BACKUP_DIR}:/backup \
  alpine \
  tar czf /backup/${VOLUME_NAME}_${DATE}.tar.gz -C /data .

echo "Backup creado: ${BACKUP_DIR}/${VOLUME_NAME}_${DATE}.tar.gz"

# Mantener solo últimos 7 backups
ls -t ${BACKUP_DIR}/${VOLUME_NAME}_*.tar.gz | tail -n +8 | xargs -r rm

# Uso:
# ./backup-volumes.sh postgres-data
```

---

## 🔍 Inspección y Debugging

### Ver Contenido de Volume

```bash
# Método 1: Contenedor temporal
docker run --rm -it \
  -v mi-volumen:/data \
  alpine \
  ls -la /data

# Método 2: Acceso directo (Linux/Mac)
sudo ls -la /var/lib/docker/volumes/mi-volumen/_data

# Método 3: Docker Desktop (todos los sistemas)
docker run --rm -it \
  -v mi-volumen:/inspect \
  alpine sh
```

### Ver Qué Contenedores Usan un Volume

```bash
# Inspeccionar volume
docker volume inspect mi-volumen

# Buscar en contenedores
docker ps -a --filter volume=mi-volumen
```

### Ver Espacio Usado

```bash
# Tamaño de todos los volumes
docker system df -v

# Tamaño de volume específico
docker run --rm -v mi-volumen:/data alpine du -sh /data
```

---

## ⚠️ Problemas Comunes

### 1. Permisos en Bind Mounts

**Problema:**
```bash
# Error: Permission denied al escribir archivos
```

**Solución:**
```bash
# Opción 1: Cambiar ownership en host
sudo chown -R 1000:1000 ./data

# Opción 2: Especificar user en container
docker run -u 1000:1000 -v $(pwd)/data:/data mi-imagen

# Opción 3: En docker-compose
services:
  app:
    user: "1000:1000"
    volumes:
      - ./data:/data
```

### 2. Node_modules Sobrescritos

**Problema:**
```yaml
# node_modules del host sobrescriben los del contenedor
volumes:
  - ./:/app
```

**Solución:**
```yaml
volumes:
  - ./:/app
  - /app/node_modules  # Volume anónimo previene sobrescritura
```

### 3. Cambios no se Reflejan

**Problema:** Editas archivos pero no ves cambios en contenedor.

**Soluciones:**
```bash
# Verificar mount
docker inspect mi-contenedor | grep -A 20 Mounts

# En Windows/Mac, habilitar file sharing
# Docker Desktop → Settings → Resources → File Sharing

# Para hot reload en algunos frameworks
environment:
  - CHOKIDAR_USEPOLLING=true
```

---

## 🛡️ Seguridad

### Mejores Prácticas

```yaml
services:
  app:
    volumes:
      # ✅ Configuración read-only
      - ./config:/app/config:ro

      # ✅ Secretos con permisos limitados
      - ./secrets:/run/secrets:ro,mode=0400

      # ❌ Nunca montar socket de Docker sin razón
      # - /var/run/docker.sock:/var/run/docker.sock
```

### Volumes Encriptados

```bash
# Usar driver de terceros para encriptación
docker volume create \
  --driver vieux/sshfs \
  --opt sshcmd=user@remote:/data \
  --opt password=$(cat password.txt) \
  encrypted-volume
```

---

## 📚 Próximos Pasos

Con volúmenes dominados:

1. [**Best Practices**](./best-practices.md) → Optimización y seguridad completa de Docker

---

## 🔗 Recursos Adicionales

- [Docker Volumes Documentation](https://docs.docker.com/storage/volumes/)
- [Storage Drivers](https://docs.docker.com/storage/storagedriver/)

---

[⬅️ Volver: Networking](./networking.md) | [➡️ Siguiente: Best Practices](./best-practices.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
