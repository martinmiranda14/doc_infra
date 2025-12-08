# Networking en Docker

## 📖 ¿Cómo Funciona la Red en Docker?

Docker crea redes virtuales que permiten a los contenedores comunicarse entre sí y con el mundo exterior. Cada contenedor puede estar conectado a una o más redes.

```
┌─────────────────────────────────────────────────┐
│              Host Machine                       │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │  Bridge Network (default)                │  │
│  │                                          │  │
│  │   ┌──────────┐    ┌──────────┐          │  │
│  │   │Container │───>│Container │          │  │
│  │   │   172.17 │    │   172.17 │          │  │
│  │   └──────────┘    └──────────┘          │  │
│  └─────────┬────────────────────────────────┘  │
│            │                                    │
│     ┌──────▼──────┐                             │
│     │  docker0    │ (Virtual bridge)            │
│     └──────┬──────┘                             │
│            │                                    │
│     ┌──────▼──────┐                             │
│     │   eth0      │ (Physical interface)        │
│     └──────┬──────┘                             │
└────────────┼────────────────────────────────────┘
             │
        Internet/LAN
```

---

## 🌐 Tipos de Redes

### 1. Bridge (por defecto)

Red privada interna creada en el host. Los contenedores pueden comunicarse entre sí y acceder a internet vía NAT.

```bash
# Listar redes
docker network ls

# Salida típica:
NETWORK ID     NAME      DRIVER    SCOPE
abc123...      bridge    bridge    local
def456...      host      host      local
ghi789...      none      null      local
```

**Características:**
- ✅ Red por defecto si no especificas otra
- ✅ Contenedores en misma red pueden comunicarse
- ✅ Aislamiento entre diferentes redes bridge
- ✅ Port mapping para exponer al host

**Ejemplo:**
```bash
# Crear contenedores en red bridge por defecto
docker run -d --name web1 nginx
docker run -d --name web2 nginx

# Se pueden comunicar por nombre de contenedor
docker exec web1 ping web2  # Funciona
```

### 2. Host

El contenedor comparte la misma red que el host. No hay aislamiento de red.

```bash
# Usar red host
docker run -d --network host nginx

# Nginx estará disponible directamente en puerto 80 del host
# No necesitas -p 80:80
```

**Características:**
- ✅ Mejor performance (sin overhead de NAT)
- ✅ Acceso directo a todos los puertos del host
- ❌ Sin aislamiento de red
- ❌ Solo funciona en Linux
- ❌ Conflictos de puerto si múltiples contenedores usan mismo puerto

**Uso típico:**
- Herramientas de monitoreo que necesitan ver toda la red del host
- Performance crítica

### 3. None

Sin red. Aislamiento completo de red.

```bash
# Contenedor sin red
docker run -d --network none alpine sleep 3600
```

**Uso típico:**
- Procesamiento de datos que no requiere red
- Máxima seguridad/aislamiento

### 4. Overlay

Red multi-host para Docker Swarm. Permite comunicación entre contenedores en diferentes hosts.

```bash
# Crear red overlay (requiere Swarm mode)
docker network create -d overlay my-overlay-network
```

**Características:**
- ✅ Comunicación entre contenedores en múltiples hosts
- ✅ Encriptación opcional
- ❌ Requiere Docker Swarm o Kubernetes

### 5. Macvlan

Asigna una dirección MAC a cada contenedor, haciéndolos aparecer como dispositivos físicos en la red.

```bash
# Crear red macvlan
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan-net
```

**Uso típico:**
- Aplicaciones legacy que esperan estar directamente en la red física
- Necesidad de direcciones IP específicas

---

## 🔧 Gestión de Redes

### Crear Redes Personalizadas

```bash
# Crear red bridge personalizada
docker network create mi-red

# Con subnet específica
docker network create --subnet=172.20.0.0/16 mi-red

# Con gateway personalizado
docker network create \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  mi-red

# Con opciones avanzadas
docker network create \
  --driver=bridge \
  --subnet=172.20.0.0/16 \
  --ip-range=172.20.240.0/20 \
  --gateway=172.20.0.1 \
  --opt "com.docker.network.bridge.name"="my-bridge" \
  mi-red
```

### Conectar Contenedores a Redes

```bash
# Al crear el contenedor
docker run -d --name app --network mi-red nginx

# Conectar contenedor existente
docker network connect mi-red app

# Con IP específica
docker network connect --ip 172.20.0.10 mi-red app

# Desconectar de red
docker network disconnect mi-red app
```

### Inspeccionar Redes

```bash
# Ver detalles de red
docker network inspect mi-red

# Ver qué contenedores están en la red
docker network inspect mi-red | grep -A 10 "Containers"

# Ver todas las redes de un contenedor
docker inspect app | grep -A 20 "Networks"
```

### Eliminar Redes

```bash
# Eliminar red específica
docker network rm mi-red

# Limpiar redes no usadas
docker network prune
```

---

## 🔌 Comunicación Entre Contenedores

### DNS Automático

Docker proporciona DNS automático en redes bridge personalizadas.

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  backend:
    image: node:16
    networks:
      - app-network

  db:
    image: postgres:14
    networks:
      - app-network

networks:
  app-network:
```

```bash
# Desde backend, puedes acceder a:
curl http://db:5432
# Docker resuelve "db" a la IP del contenedor
```

### Por Nombre vs Por IP

**✅ Usar nombres (recomendado):**
```bash
DATABASE_URL=postgres://db:5432/mydb
```

**❌ Usar IPs (no recomendado):**
```bash
DATABASE_URL=postgres://172.20.0.5:5432/mydb
# IPs pueden cambiar cuando reinicies contenedores
```

### Aliases de Red

```bash
# Dar múltiples nombres a un contenedor
docker network connect --alias database --alias db mi-red postgres-container

# Ahora accesible por:
# - postgres-container
# - database
# - db
```

En docker-compose:
```yaml
services:
  db:
    image: postgres:14
    networks:
      app-network:
        aliases:
          - database
          - postgres
```

---

## 🚪 Exposición de Puertos

### Port Publishing

```bash
# Formato: -p HOST:CONTAINER
docker run -d -p 8080:80 nginx

# Puerto aleatorio en host
docker run -d -p 80 nginx  # Host asigna puerto disponible

# Solo localhost
docker run -d -p 127.0.0.1:8080:80 nginx

# Múltiples puertos
docker run -d -p 80:80 -p 443:443 nginx

# UDP
docker run -d -p 53:53/udp dns-server
```

### EXPOSE en Dockerfile

```dockerfile
# EXPOSE es solo documentación
EXPOSE 80 443

# No publica puertos, solo documenta
# Debes usar -p al ejecutar
```

### Ver Puertos Publicados

```bash
# Listar puertos de un contenedor
docker port mi-contenedor

# Salida:
80/tcp -> 0.0.0.0:8080
443/tcp -> 0.0.0.0:8443
```

---

## 🔐 Aislamiento de Redes

### Redes Separadas por Ambiente

```yaml
version: '3.8'

services:
  # Frontend (pública)
  web:
    image: nginx
    networks:
      - frontend

  # Backend (privada)
  api:
    image: node:16
    networks:
      - frontend  # Accesible por web
      - backend   # Puede hablar con DB

  # Database (solo backend)
  db:
    image: postgres:14
    networks:
      - backend   # Solo accesible por API

networks:
  frontend:
  backend:
    internal: true  # Sin acceso a internet
```

### Red Interna (sin Internet)

```bash
# Crear red sin acceso externo
docker network create --internal mi-red-interna

# Contenedores en esta red NO pueden acceder a internet
docker run --network mi-red-interna alpine ping google.com
# Fallará
```

---

## 📊 Ejemplo Práctico Completo

### Aplicación con Múltiples Redes

**Arquitectura:**
```
┌──────────────┐
│   Internet   │
└──────┬───────┘
       │
┌──────▼───────┐
│    Nginx     │ (frontend network)
└──────┬───────┘
       │
┌──────▼───────┐
│  API Server  │ (frontend + backend networks)
└──────┬───────┘
       │
    ┌──┴──┐
┌───▼───┐ │
│  DB   │ │ (backend network - internal)
└───────┘ │
   ┌──────▼───┐
   │  Redis   │
   └──────────┘
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - frontend
    depends_on:
      - api

  # API Server
  api:
    build: ./api
    environment:
      DB_HOST: postgres
      REDIS_HOST: redis
    networks:
      - frontend
      - backend
    depends_on:
      - postgres
      - redis

  # PostgreSQL Database
  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

  # Redis Cache
  redis:
    image: redis:7-alpine
    networks:
      - backend

  # Adminer (DB GUI) - Solo en desarrollo
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    networks:
      - backend
    profiles:
      - dev

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # Sin internet

volumes:
  db-data:
```

**nginx.conf:**
```nginx
events {
    worker_connections 1024;
}

http {
    upstream api_backend {
        server api:3000;
    }

    server {
        listen 80;

        location /api/ {
            proxy_pass http://api_backend/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location / {
            root /usr/share/nginx/html;
            try_files $uri $uri/ /index.html;
        }
    }
}
```

**Ejecutar:**
```bash
# Producción (sin adminer)
docker-compose up -d

# Desarrollo (con adminer)
docker-compose --profile dev up -d

# Verificar redes
docker network ls | grep myapp
```

---

## 🔍 Debugging de Redes

### Verificar Conectividad

```bash
# Ping entre contenedores
docker exec container1 ping container2

# Verificar DNS
docker exec container1 nslookup container2

# Instalar herramientas de red temporalmente
docker exec -it container1 sh
apk add --no-cache curl
curl http://container2:3000

# Usar contenedor temporal para testing
docker run --rm --network mi-red nicolaka/netshoot
# Contenedor con todas las herramientas de red
```

### Inspeccionar Tráfico

```bash
# Ver interfaces de red en contenedor
docker exec container1 ip addr

# Ver rutas
docker exec container1 ip route

# Ver puertos en escucha
docker exec container1 netstat -tlnp
```

### Problemas Comunes

#### 1. Contenedores no se ven entre sí

```bash
# Verificar que están en la misma red
docker network inspect mi-red

# Verificar DNS
docker exec container1 nslookup container2

# Solución: Asegura que están en red personalizada (no default bridge)
```

#### 2. No hay acceso a internet desde contenedor

```bash
# Verificar DNS
docker exec container1 ping 8.8.8.8  # Ping a IP funciona
docker exec container1 ping google.com  # Ping a dominio falla

# Problema: DNS no configurado
# Solución: Especificar DNS servers
docker run --dns 8.8.8.8 --dns 8.8.4.4 mi-imagen
```

#### 3. Puerto no accesible desde host

```bash
# Verificar que puerto está publicado
docker port mi-contenedor

# Verificar que aplicación escucha en 0.0.0.0, no localhost
# Mal:  app.listen(3000, 'localhost')
# Bien: app.listen(3000, '0.0.0.0')
```

---

## ⚡ Performance y Optimización

### Bridge vs Host Performance

| Operación | Bridge | Host | Diferencia |
|-----------|--------|------|------------|
| Latencia | ~50µs | ~10µs | Host 5x más rápido |
| Throughput | 9-10 Gbps | 10 Gbps | Casi igual |

**Recomendación:** Usa bridge para mayor seguridad, host solo si performance es crítica.

### DNS Caching

Docker no cachea DNS por defecto. Para mejorar:

```yaml
services:
  app:
    image: myapp
    dns:
      - 8.8.8.8
      - 1.1.1.1
    dns_opt:
      - "timeout:1"
      - "attempts:2"
```

---

## 🛡️ Seguridad

### Mejores Prácticas

```yaml
# ✅ Bueno: Redes separadas
networks:
  frontend:
    # Internet-facing
  backend:
    internal: true  # Sin acceso a internet

# ✅ Bueno: Exponer solo puertos necesarios
services:
  api:
    # NO exponer puerto de DB
    # DB solo accesible internamente

# ✅ Bueno: Usar secrets para datos sensibles
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Encriptación

```bash
# Encriptar tráfico en overlay networks
docker network create \
  --driver overlay \
  --opt encrypted \
  secure-network
```

---

## 📚 Próximos Pasos

Con networking dominado:

1. [**Volumes**](./volumes.md) → Persistencia de datos
2. [**Best Practices**](./best-practices.md) → Optimización y seguridad completa

---

## 🔗 Recursos Adicionales

- [Docker Networking Overview](https://docs.docker.com/network/)
- [Network Driver Comparison](https://docs.docker.com/network/#network-drivers)

---

[⬅️ Volver: Docker Compose](./docker-compose.md) | [➡️ Siguiente: Volumes](./volumes.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
