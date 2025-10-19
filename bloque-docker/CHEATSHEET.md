# Cheatsheet Consolidado - Bloque 1: Docker

Referencia rápida de comandos esenciales del Bloque Docker (Clases 1-5).

---

## Comandos Docker Básicos (Clase 1)

### Gestión de Containers

```bash
# Listar containers corriendo
docker ps

# Listar todos los containers (incluyendo detenidos)
docker ps -a

# Ejecutar container en modo detached
docker run -d nginx:alpine

# Ejecutar container interactivo
docker run -it ubuntu bash

# Mapear puertos
docker run -d -p 8080:80 nginx:alpine

# Detener container
docker stop <container-id>

# Eliminar container
docker rm <container-id>

# Eliminar container forzadamente
docker rm -f <container-id>

# Ver logs
docker logs <container-id>

# Ver logs en tiempo real
docker logs -f <container-id>

# Ejecutar comando en container corriendo
docker exec -it <container-id> bash
```

### Gestión de Imágenes

```bash
# Listar imágenes
docker images

# Descargar imagen
docker pull nginx:alpine

# Eliminar imagen
docker rmi nginx:alpine

# Ver capas de una imagen
docker history nginx:alpine

# Inspeccionar imagen
docker inspect nginx:alpine
```

### Limpieza

```bash
# Eliminar containers detenidos
docker container prune

# Eliminar imágenes sin usar
docker image prune

# Eliminar todo lo no usado
docker system prune

# Limpieza completa (incluye volúmenes)
docker system prune -a --volumes
```

---

## Dockerfiles y Build (Clase 2)

### Estructura Básica de Dockerfile

```dockerfile
# Imagen base
FROM node:18-alpine

# Metadata
LABEL maintainer="tu-nombre"
LABEL version="1.0.0"

# Crear usuario non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Directorio de trabajo
WORKDIR /app

# Copiar archivos
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN chown -R nodejs:nodejs /app

# Cambiar a usuario non-root
USER nodejs

# Puerto
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Comando por defecto
CMD ["node", "server.js"]
```

### Multi-Stage Build

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
COPY --from=build /app/package.json ./

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
RUN chown -R nodejs:nodejs /app
USER nodejs

EXPOSE 3000
CMD ["node", "src/server.js"]
```

### Comandos de Build

```bash
# Construir imagen
docker build -t mi-app:1.0 .

# Build sin cache
docker build --no-cache -t mi-app:1.0 .

# Build con archivo específico
docker build -f Dockerfile.prod -t mi-app:prod .

# Ver tamaño de imagen
docker images mi-app
```

### Docker Hub

```bash
# Login
docker login

# Tagear imagen
docker tag mi-app:1.0 usuario/mi-app:1.0

# Subir imagen
docker push usuario/mi-app:1.0

# Descargar imagen
docker pull usuario/mi-app:1.0
```

---

## Docker Compose (Clase 3)

### Comandos Esenciales

```bash
# Levantar servicios
docker compose up -d

# Levantar y reconstruir imágenes
docker compose up -d --build

# Ver estado
docker compose ps

# Ver logs
docker compose logs -f

# Ver logs de servicio específico
docker compose logs -f app

# Detener servicios
docker compose stop

# Detener y eliminar
docker compose down

# Detener y eliminar con volúmenes
docker compose down -v

# Reiniciar servicios
docker compose restart

# Ejecutar comando en servicio
docker compose exec app sh
```

### Estructura Básica docker-compose.yml

```yaml
services:
  app:
    build: .
    container_name: mi-app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db
    networks:
      - app-network
    volumes:
      - ./src:/app/src
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      - POSTGRES_DB=midb
      - POSTGRES_USER=usuario
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

### Tipos de Volúmenes

```yaml
# Named volume (recomendado para datos persistentes)
volumes:
  - db-data:/var/lib/postgresql/data

# Bind mount (desarrollo, hot reload)
volumes:
  - ./src:/app/src

# Anonymous volume (temporal)
volumes:
  - /app/node_modules
```

### Tipos de Redes

```yaml
# Bridge (default - comunicación entre containers)
networks:
  app-network:
    driver: bridge

# Host (usa red del host directamente)
network_mode: host

# None (sin red)
network_mode: none
```

---

## Microservicios (Clase 4)

### MongoDB

```bash
# Conectar a MongoDB
docker compose exec mongo mongosh

# Comandos dentro de mongosh
show dbs
use midb
show collections
db.users.find()
db.users.insertOne({nombre: "Juan", email: "juan@example.com"})
```

### Redis

```bash
# Conectar a Redis
docker compose exec redis redis-cli

# Comandos Redis
KEYS *
GET clave
SET clave "valor"
DEL clave
FLUSHALL
TTL clave
```

### Redis en Node.js (Cache Pattern)

```javascript
const { createClient } = require('redis');

const redis = createClient({
  socket: { host: 'redis', port: 6379 }
});

await redis.connect();

// Cache middleware
const cacheMiddleware = async (req, res, next) => {
  const cacheKey = `users:all`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    console.log('Cache HIT');
    return res.json({ source: 'cache', data: JSON.parse(cached) });
  }

  console.log('Cache MISS');
  req.cacheKey = cacheKey;
  next();
};

// Guardar en cache con TTL
await redis.setEx('users:all', 60, JSON.stringify(users));

// Invalidar cache
await redis.del('users:all');
```

### Kong API Gateway (Configuración Inicial)

```bash
# Crear Service
curl -X POST http://localhost:8001/services \
  --data name=mi-api \
  --data url=http://app:8080

# Crear Route
curl -X POST http://localhost:8001/services/mi-api/routes \
  --data "paths[]=/api" \
  --data "name=api-route"

# Habilitar CORS
curl -X POST http://localhost:8001/services/mi-api/plugins \
  --data "name=cors" \
  --data "config.origins=*"
```

---

## Seguridad y Optimización (Clase 5)

### Trivy - Escaneo de Vulnerabilidades

```bash
# Instalación como container (recomendado)
alias trivy="docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest"

# Escanear imagen
trivy image nginx:alpine

# Solo vulnerabilidades críticas y altas
trivy image --severity CRITICAL,HIGH mi-app:1.0

# Generar reporte JSON
trivy image -f json -o reporte.json mi-app:1.0

# Escanear Dockerfile
trivy config Dockerfile

# Escanear docker-compose
trivy config docker-compose.yml
```

### Remediación de Vulnerabilidades

```bash
# 1. Escanear e identificar
trivy image --severity CRITICAL,HIGH mi-app:1.0

# 2. Ver solo con fix disponible
trivy image --ignore-unfixed mi-app:1.0

# 3. Identificar imagen base
grep "^FROM" Dockerfile

# 4. Actualizar Dockerfile
# FROM node:18-alpine → FROM node:18.19-alpine

# 5. Rebuild
docker build -t mi-app:1.1 .

# 6. Re-escanear
trivy image --severity CRITICAL,HIGH mi-app:1.1
```

### Optimización de Imágenes

```bash
# Usar imagen base Alpine
FROM node:18-alpine      # ~150MB
FROM python:3.11-alpine  # ~50MB

# Multi-stage build (ver sección Dockerfiles arriba)

# .dockerignore
node_modules
.git
*.log
.env
```

### Análisis de Tamaño

```bash
# Ver tamaño de imagen
docker images mi-app

# Ver capas y tamaños
docker history mi-app:1.0

# Comparar antes/después
docker images | grep mi-app
```

---

## Workflow Completo (End-to-End)

### Desarrollo Local

```bash
# 1. Crear Dockerfile multi-stage con Alpine
# 2. Crear docker-compose.yml con servicios necesarios
# 3. Crear .dockerignore

# 4. Levantar stack
docker compose up -d --build

# 5. Ver logs
docker compose logs -f

# 6. Probar aplicación
curl http://localhost:3000/health

# 7. Ver estado
docker compose ps
```

### Escaneo de Seguridad

```bash
# 8. Escanear imagen
trivy image --severity CRITICAL,HIGH mi-app:latest

# 9. Si hay vulnerabilidades, remediar
# - Actualizar imagen base
# - Actualizar dependencias
# - Rebuild

# 10. Re-escanear
trivy image --severity CRITICAL,HIGH mi-app:latest
```

### Publicación

```bash
# 11. Tagear imagen
docker tag mi-app:latest usuario/mi-app:1.0

# 12. Login a Docker Hub
docker login

# 13. Subir imagen
docker push usuario/mi-app:1.0
```

### Limpieza

```bash
# 14. Detener servicios
docker compose down

# 15. Limpiar sistema
docker system prune -a
```

---

## Troubleshooting Común

### Error: "port is already allocated"

```bash
# Cambiar puerto en docker-compose.yml
ports:
  - "3001:3000"  # En lugar de 3000:3000

# O detener container que usa el puerto
docker ps
docker stop <container-id>
```

### Error: "permission denied" (Linux)

```bash
# Agregar usuario a grupo docker
sudo usermod -aG docker $USER
newgrp docker
```

### Error: Container se detiene inmediatamente

```bash
# Ver logs para diagnosticar
docker logs <container-id>

# Ver qué pasó
docker ps -a
```

### Error: "database connection failed"

```bash
# Verificar que servicios estén corriendo
docker compose ps

# Verificar logs de BD
docker compose logs db

# Verificar conectividad desde app
docker compose exec app ping db

# Verificar variables de entorno
docker compose exec app env | grep DB
```

### Cache de Docker no funciona

```bash
# Orden correcto en Dockerfile:
COPY package.json ./
RUN npm install         # Se cachea
COPY . .                # Código al final

# Forzar rebuild sin cache
docker build --no-cache -t mi-app .
```

---

## Buenas Prácticas - Resumen

### Dockerfiles

- Usar imágenes base Alpine
- Multi-stage builds
- Usuario non-root
- COPY específico (package.json antes que código)
- Labels de metadata
- Health checks
- .dockerignore configurado

### Docker Compose

- Named volumes para datos persistentes
- Custom networks
- depends_on para orden de inicio
- Variables de entorno centralizadas
- restart: unless-stopped para producción
- NO usar version: (deprecated en Compose v2)

### Seguridad

- Escanear con Trivy regularmente
- 0 vulnerabilidades CRITICAL antes de deploy
- Actualizar imágenes base frecuentemente
- No hardcodear secretos
- Usuario non-root siempre

### Optimización

- Imagen Alpine sobre Full
- Multi-stage para reducir tamaño
- Aprovechar cache de layers
- Limpiar en misma capa (RUN apt-get && clean)
- JRE en lugar de JDK (Java)

---

## Recursos Adicionales

### Documentación Oficial
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### Imágenes Oficiales
- [Docker Hub](https://hub.docker.com/)
- [Alpine Linux](https://hub.docker.com/_/alpine)
- [Node.js](https://hub.docker.com/_/node)
- [Python](https://hub.docker.com/_/python)
- [PostgreSQL](https://hub.docker.com/_/postgres)
- [MongoDB](https://hub.docker.com/_/mongo)
- [Redis](https://hub.docker.com/_/redis)
- [Nginx](https://hub.docker.com/_/nginx)

### Herramientas
- [Trivy - Vulnerability Scanner](https://aquasecurity.github.io/trivy/)
- [Docker Scout](https://docs.docker.com/scout/)
- [Dive - Image Layer Inspector](https://github.com/wagoodman/dive)

### Seguridad
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [CVE Database](https://nvd.nist.gov/)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)

---

## Cheatsheets por Clase

Para referencia detallada de cada clase:

- [Clase 1: Introducción](clase1-introduccion/cheatsheet.md)
- [Clase 2: Dockerfiles](clase2-dockerfiles/cheatsheet.md)
- [Clase 3: Docker Compose](clase3-compose/cheatsheet.md)
- [Clase 4: Microservicios](clase4-microservicios/cheatsheet.md)
- [Clase 5: Seguridad y Optimización](clase5-seguridad/cheatsheet.md)

---

**Autor:** Alejandro Fiengo (alefiengo)
**Curso:** Docker & Kubernetes - Contenedores y Orquestación en la Práctica
**Institución:** i-Quattro
