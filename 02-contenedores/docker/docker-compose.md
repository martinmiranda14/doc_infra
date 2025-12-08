# Docker Compose: Aplicaciones Multi-Contenedor

## 📖 ¿Qué es Docker Compose?

Docker Compose es una herramienta para definir y ejecutar aplicaciones Docker multi-contenedor. Usa archivos YAML para configurar los servicios de tu aplicación, y con un solo comando, creas e inicias todos los servicios.

### Problema que Resuelve

**Sin Compose (❌):**
```bash
# Crear red
docker network create mi-red

# Ejecutar base de datos
docker run -d --name db --network mi-red \
  -e POSTGRES_PASSWORD=secret \
  -v db-data:/var/lib/postgresql/data \
  postgres:14

# Ejecutar backend
docker run -d --name api --network mi-red \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://db:5432 \
  mi-api:latest

# Ejecutar frontend
docker run -d --name web --network mi-red \
  -p 8080:80 \
  mi-web:latest
```

**Con Compose (✅):**
```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

  api:
    image: mi-api:latest
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://db:5432
    depends_on:
      - db

  web:
    image: mi-web:latest
    ports:
      - "8080:80"
    depends_on:
      - api

volumes:
  db-data:
```

```bash
# Un solo comando
docker-compose up -d
```

---

## 📦 Instalación

### Docker Desktop (Windows/Mac)

Docker Compose viene incluido con Docker Desktop.

```bash
# Verificar instalación
docker-compose --version
# o
docker compose version  # Compose V2 (plugin)
```

### Linux

```bash
# Compose V2 (recomendado)
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Compose V1 (legacy)
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

**Nota:** Compose V2 usa `docker compose` (sin guión), V1 usa `docker-compose` (con guión).

---

## 📝 Estructura del Archivo compose.yml

### Versiones

```yaml
# Version (opcional en Compose V2)
version: '3.8'

# Secciones principales
services:   # Contenedores a ejecutar
networks:   # Redes personalizadas
volumes:    # Volúmenes nombrados
configs:    # Configuraciones (Swarm)
secrets:    # Secretos (Swarm)
```

### Ejemplo Básico

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: unless-stopped

  app:
    build: ./app
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## 🔧 Configuración de Servicios

### Imagen vs Build

#### Usar Imagen Existente

```yaml
services:
  web:
    image: nginx:1.21-alpine
    # Descarga imagen del registry
```

#### Construir desde Dockerfile

```yaml
services:
  app:
    build: .
    # Busca Dockerfile en el directorio actual
```

#### Build Avanzado

```yaml
services:
  app:
    build:
      context: ./app              # Directorio de contexto
      dockerfile: Dockerfile.prod  # Archivo específico
      args:                       # Argumentos de build
        - NODE_VERSION=16
        - BUILD_DATE=${BUILD_DATE}
      target: production          # Stage específico en multi-stage
      cache_from:                 # Cache desde imágenes
        - app:latest
```

### Puertos

```yaml
services:
  web:
    ports:
      # HOST:CONTAINER
      - "8080:80"           # Mapeo simple
      - "443:443"           # Múltiples puertos
      - "127.0.0.1:3000:3000"  # Solo localhost

      # Formato largo
      - target: 80
        published: 8080
        protocol: tcp
        mode: host
```

### Variables de Entorno

```yaml
services:
  app:
    # Forma 1: Map
    environment:
      NODE_ENV: production
      PORT: 3000
      DEBUG: "false"

    # Forma 2: Array
    environment:
      - NODE_ENV=production
      - PORT=3000

    # Forma 3: Archivo .env
    env_file:
      - .env
      - .env.local
```

**.env:**
```env
NODE_ENV=production
DATABASE_URL=postgres://db:5432/mydb
SECRET_KEY=my-secret-key
```

### Volúmenes

```yaml
services:
  app:
    volumes:
      # Volumen nombrado
      - app-data:/app/data

      # Bind mount (directorio host)
      - ./src:/app/src

      # Bind mount read-only
      - ./config:/app/config:ro

      # tmpfs mount (en memoria)
      - type: tmpfs
        target: /app/cache

volumes:
  # Declarar volúmenes nombrados
  app-data:
  db-data:
    driver: local
    driver_opts:
      type: none
      device: /path/on/host
      o: bind
```

### Redes

```yaml
services:
  web:
    networks:
      - frontend
      - backend

  app:
    networks:
      - backend

  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # Sin acceso a internet
```

### Dependencias

```yaml
services:
  web:
    depends_on:
      - app
      # web espera a que app esté creado (no necesariamente listo)

  app:
    depends_on:
      db:
        condition: service_healthy
      # Espera a que db esté "healthy"

  db:
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### Restart Policies

```yaml
services:
  app:
    restart: no              # No reiniciar (default)
    # restart: always        # Siempre reiniciar
    # restart: on-failure    # Solo si falla
    # restart: unless-stopped # Siempre excepto si se detuvo manualmente
```

### Health Checks

```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s      # Cada cuánto verificar
      timeout: 10s       # Timeout de la prueba
      retries: 3         # Intentos antes de marcar unhealthy
      start_period: 40s  # Tiempo de gracia al iniciar
```

---

## 🚀 Comandos Principales

### Levantar Servicios

```bash
# Iniciar todos los servicios
docker-compose up

# En background (detached)
docker-compose up -d

# Reconstruir imágenes antes de iniciar
docker-compose up --build

# Iniciar servicios específicos
docker-compose up web db

# Escalar servicios
docker-compose up --scale app=3
```

### Detener Servicios

```bash
# Detener servicios (mantiene contenedores)
docker-compose stop

# Detener y eliminar contenedores
docker-compose down

# Eliminar contenedores, redes Y volúmenes
docker-compose down -v

# Eliminar contenedores, redes, volúmenes E imágenes
docker-compose down --rmi all -v
```

### Ver Estado

```bash
# Listar servicios en ejecución
docker-compose ps

# Ver logs
docker-compose logs

# Logs de servicio específico
docker-compose logs app

# Seguir logs en tiempo real
docker-compose logs -f

# Últimas 100 líneas
docker-compose logs --tail=100
```

### Gestión de Servicios

```bash
# Reiniciar servicios
docker-compose restart

# Reiniciar servicio específico
docker-compose restart app

# Pausar servicios
docker-compose pause

# Reanudar servicios
docker-compose unpause

# Ejecutar comando en servicio
docker-compose exec app sh

# Ejecutar comando one-off
docker-compose run app npm test
```

### Build y Push

```bash
# Construir o reconstruir imágenes
docker-compose build

# Build sin cache
docker-compose build --no-cache

# Build con argumentos
docker-compose build --build-arg NODE_VERSION=18

# Pull imágenes
docker-compose pull

# Push imágenes
docker-compose push
```

---

## 💼 Ejemplos Prácticos

### Ejemplo 1: Stack MEAN (MongoDB, Express, Angular, Node)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # MongoDB
  mongodb:
    image: mongo:6
    container_name: mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    networks:
      - mean-network

  # Backend Node.js/Express
  backend:
    build: ./backend
    container_name: backend
    ports:
      - "3000:3000"
    environment:
      MONGO_URL: mongodb://admin:password@mongodb:27017
      NODE_ENV: development
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - mongodb
    networks:
      - mean-network
    command: npm run dev

  # Frontend Angular
  frontend:
    build: ./frontend
    container_name: frontend
    ports:
      - "4200:4200"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    networks:
      - mean-network
    command: npm start

volumes:
  mongo-data:

networks:
  mean-network:
    driver: bridge
```

### Ejemplo 2: WordPress + MySQL

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    networks:
      - wp-network

  wordpress:
    image: wordpress:latest
    depends_on:
      - db
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html
    networks:
      - wp-network

  phpmyadmin:
    image: phpmyadmin:latest
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: rootpassword
    networks:
      - wp-network

volumes:
  db_data:
  wordpress_data:

networks:
  wp-network:
```

### Ejemplo 3: Microservicios con Nginx Reverse Proxy

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
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - auth-service
      - user-service
      - product-service
    networks:
      - frontend

  # Auth Service
  auth-service:
    build: ./services/auth
    environment:
      DB_HOST: auth-db
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      - auth-db
    networks:
      - frontend
      - auth-backend

  auth-db:
    image: postgres:14
    environment:
      POSTGRES_DB: auth
      POSTGRES_PASSWORD: ${AUTH_DB_PASSWORD}
    volumes:
      - auth-db-data:/var/lib/postgresql/data
    networks:
      - auth-backend

  # User Service
  user-service:
    build: ./services/users
    environment:
      DB_HOST: user-db
    depends_on:
      - user-db
    networks:
      - frontend
      - user-backend

  user-db:
    image: postgres:14
    environment:
      POSTGRES_DB: users
      POSTGRES_PASSWORD: ${USER_DB_PASSWORD}
    volumes:
      - user-db-data:/var/lib/postgresql/data
    networks:
      - user-backend

  # Product Service
  product-service:
    build: ./services/products
    environment:
      MONGO_URL: mongodb://product-db:27017/products
    depends_on:
      - product-db
    networks:
      - frontend
      - product-backend

  product-db:
    image: mongo:6
    volumes:
      - product-db-data:/data/db
    networks:
      - product-backend

  # Redis Cache (compartido)
  redis:
    image: redis:7-alpine
    networks:
      - frontend

volumes:
  auth-db-data:
  user-db-data:
  product-db-data:

networks:
  frontend:
  auth-backend:
    internal: true
  user-backend:
    internal: true
  product-backend:
    internal: true
```

### Ejemplo 4: Stack de Desarrollo con Hot Reload

```yaml
version: '3.8'

services:
  # Frontend React
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src
      - /app/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true  # Para hot reload en Docker
      - REACT_APP_API_URL=http://localhost:8000
    command: npm start

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/devdb
      REDIS_URL: redis://redis:6379
    depends_on:
      - db
      - redis
    command: npm run dev

  # PostgreSQL
  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: devdb
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # Adminer (DB GUI)
  adminer:
    image: adminer:latest
    ports:
      - "8080:8080"
    depends_on:
      - db

volumes:
  postgres-data:
```

---

## 🌍 Múltiples Ambientes

### Archivos Override

**docker-compose.yml** (base):
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
```

**docker-compose.override.yml** (desarrollo - se aplica automáticamente):
```yaml
version: '3.8'

services:
  app:
    volumes:
      - ./src:/app/src
    environment:
      NODE_ENV: development
    command: npm run dev
```

**docker-compose.prod.yml** (producción):
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest  # Usa imagen pre-built
    environment:
      NODE_ENV: production
    restart: always
    deploy:
      replicas: 3
```

**Uso:**
```bash
# Desarrollo (usa override automáticamente)
docker-compose up

# Producción
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Variables de Entorno por Ambiente

```bash
# .env.development
NODE_ENV=development
DATABASE_URL=postgres://localhost:5432/dev
API_URL=http://localhost:3000

# .env.production
NODE_ENV=production
DATABASE_URL=postgres://prod-db:5432/prod
API_URL=https://api.example.com
```

```bash
# Cargar archivo específico
docker-compose --env-file .env.development up
docker-compose --env-file .env.production up
```

---

## 🔍 Debugging y Troubleshooting

### Ver Configuración Final

```bash
# Ver configuración compilada (con todos los overrides)
docker-compose config

# Validar sintaxis
docker-compose config --quiet
```

### Logs Detallados

```bash
# Logs de todos los servicios
docker-compose logs -f

# Filtrar por servicio y timestamp
docker-compose logs -f --tail=50 --timestamps app

# Logs de múltiples servicios
docker-compose logs -f app db
```

### Inspeccionar Contenedores

```bash
# Ver procesos en servicio
docker-compose top app

# Estadísticas en tiempo real
docker stats $(docker-compose ps -q)

# Acceder a shell de servicio
docker-compose exec app sh
docker-compose exec app bash

# Ejecutar comando sin servicio corriendo
docker-compose run --rm app npm test
```

### Problemas Comunes

#### 1. Puertos en uso

```bash
# Error: port is already allocated
# Solución: Cambiar puerto en docker-compose.yml
ports:
  - "8081:80"  # Cambiar 8080 a 8081
```

#### 2. Volúmenes no actualizan

```bash
# Recrear contenedores y volúmenes
docker-compose down -v
docker-compose up --build
```

#### 3. Contenedor se detiene inmediatamente

```bash
# Ver logs
docker-compose logs app

# Verificar que hay un proceso long-running
# CMD en Dockerfile no debe terminar
```

---

## 📚 Próximos Pasos

Con Docker Compose dominado, continúa con:

1. [**Networking**](./networking.md) → Redes avanzadas entre contenedores
2. [**Volumes**](./volumes.md) → Persistencia y gestión de datos
3. [**Best Practices**](./best-practices.md) → Optimización y producción

---

## 🔗 Recursos Adicionales

- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Compose CLI Reference](https://docs.docker.com/compose/reference/)
- [Awesome Compose Examples](https://github.com/docker/awesome-compose)

---

[⬅️ Volver: Dockerfile](./dockerfile.md) | [➡️ Siguiente: Networking](./networking.md)

> ⚙️ **Nota**: Este documento fue generado con asistencia de IA (Claude Code). Se recomienda revisar y validar el contenido antes de su uso en producción.
