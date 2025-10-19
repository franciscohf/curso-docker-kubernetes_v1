# Cheatsheet - Clase 5: Seguridad y Optimización

## Comandos Trivy

### Instalación

**Opción A: Instalación local**

```bash
# Ubuntu/Debian
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy

# macOS
brew install trivy

# Verificar instalación
trivy --version
```

**Opción B: Usar como container (recomendado)**

```bash
# Uso directo sin instalación
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image nginx:alpine

# Crear alias permanente
echo 'alias trivy="docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest"' >> ~/.bashrc
source ~/.bashrc

# Usar normalmente
trivy image nginx:alpine

# Escanear con volumen para reportes
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/output \
  aquasec/trivy:latest image -f json -o /output/report.json nginx:alpine
```

### Escaneo de Imágenes

```bash
# Escanear imagen completa
trivy image nombre-imagen:tag

# Solo vulnerabilidades críticas y altas
trivy image --severity CRITICAL,HIGH nombre-imagen:tag

# Escanear solo vulnerabilidades con fix disponible
trivy image --ignore-unfixed nombre-imagen:tag

# Generar reporte JSON
trivy image -f json -o reporte.json nombre-imagen:tag

# Generar reporte en tabla
trivy image -f table -o reporte.txt nombre-imagen:tag

# Generar reporte SARIF (para CI/CD)
trivy image -f sarif -o reporte.sarif nombre-imagen:tag
```

### Escaneo de Configuración

```bash
# Escanear Dockerfile
trivy config Dockerfile

# Escanear directorio completo
trivy config .

# Escanear docker-compose.yml
trivy config docker-compose.yml

# Escanear configuración de Kubernetes
trivy config k8s-manifests/
```

### Gestión de Base de Datos

```bash
# Actualizar base de datos de vulnerabilidades
trivy image --download-db-only

# Limpiar cache
trivy image --clear-cache

# Ver versión de base de datos
trivy --version
```

### Filtros Avanzados

```bash
# Ignorar vulnerabilidades específicas
trivy image --ignorefile .trivyignore nombre-imagen:tag

# Escanear solo dependencias de aplicación (no OS)
trivy image --vuln-type library nombre-imagen:tag

# Escanear solo dependencias del OS
trivy image --vuln-type os nombre-imagen:tag

# Timeout personalizado
trivy image --timeout 10m nombre-imagen:tag
```

### Archivo .trivyignore

Crea `.trivyignore` en la raíz del proyecto:

```
# Ignorar CVEs específicos
CVE-2024-12345
CVE-2024-67890

# Ignorar por paquete
pkg:golang/github.com/example/vulnerable-lib

# Comentarios
# Esta vulnerabilidad no aplica a nuestro caso de uso
```

## Niveles de Severidad

| Severidad | Descripción | Acción |
|-----------|-------------|--------|
| **CRITICAL** | Requiere acción inmediata | Bloquear deploy |
| **HIGH** | Requiere atención prioritaria | Resolver antes de producción |
| **MEDIUM** | Debe corregirse en próximo release | Planificar fix |
| **LOW** | Puede esperar | Monitorear |
| **UNKNOWN** | Sin clasificación | Investigar |

## Remediación de Vulnerabilidades

### Workflow Completo

```bash
# 1. Escanear e identificar problemas
trivy image --severity CRITICAL,HIGH mi-app:1.0

# 2. Ver solo vulnerabilidades con fix
trivy image --severity CRITICAL,HIGH --ignore-unfixed=false mi-app:1.0

# 3. Identificar imagen base
grep "^FROM" Dockerfile

# 4. Buscar versión más reciente en Docker Hub
# https://hub.docker.com

# 5. Actualizar Dockerfile
# FROM node:18-alpine    →    FROM node:18.19-alpine

# 6. Rebuild
docker build -t mi-app:1.1 .

# 7. Re-escanear y comparar
trivy image --severity CRITICAL,HIGH mi-app:1.0
trivy image --severity CRITICAL,HIGH mi-app:1.1

# 8. Verificar funcionalidad
docker run -d -p 8080:8080 mi-app:1.1
curl http://localhost:8080/health
```

### Comparar Imágenes Base

```bash
# Escanear diferentes variantes
trivy image --severity CRITICAL,HIGH node:18-alpine
trivy image --severity CRITICAL,HIGH node:18-slim
trivy image --severity CRITICAL,HIGH node:18

# Generar reportes para comparar
trivy image --severity CRITICAL,HIGH -f json node:18-alpine > alpine.json
trivy image --severity CRITICAL,HIGH -f json node:18-slim > slim.json
trivy image --severity CRITICAL,HIGH -f json node:18 > full.json

# Comparar total de vulnerabilidades
grep '"Total":' alpine.json slim.json full.json
```

### Actualizar Imagen Base

```bash
# Ver versiones actuales
docker images | grep mi-app

# Pull de versión más reciente
docker pull node:18.19-alpine

# Actualizar Dockerfile
sed -i 's/FROM node:18-alpine/FROM node:18.19-alpine/' Dockerfile

# Rebuild con --pull para asegurar última versión
docker build --pull -t mi-app:latest .

# Verificar mejora
trivy image --severity CRITICAL mi-app:latest
```

### Resolver Vulnerabilidades Específicas

```bash
# Identificar paquete vulnerable
trivy image mi-app:1.0 | grep "CVE-2024-XXXXX"

# Ver detalles de la vulnerabilidad
trivy image --severity CRITICAL mi-app:1.0 | grep -A 5 "CVE-2024-XXXXX"

# Buscar en qué versión se resolvió
# Visitar: https://nvd.nist.gov/vuln/detail/CVE-2024-XXXXX
```

### Vulnerabilidades Sin Fix

```bash
# Listar vulnerabilidades sin fix
trivy image --severity CRITICAL,HIGH mi-app:1.0 | grep "fixed: false"

# Opciones:
# 1. Ignorar temporalmente con .trivyignore
echo "CVE-2024-XXXXX" >> .trivyignore
echo "# Reason: No fix available, low impact in our use case" >> .trivyignore

# 2. Evaluar imagen base alternativa
trivy image --severity CRITICAL,HIGH python:3.11-alpine
trivy image --severity CRITICAL,HIGH python:3.11-slim
trivy image --severity CRITICAL,HIGH python:3.11-slim-bookworm

# 3. Monitorear periódicamente
trivy image --severity CRITICAL,HIGH mi-app:1.0 > scan-$(date +%Y%m%d).txt
```

### Integración en CI/CD

```bash
# Fallar build si hay vulnerabilidades CRITICAL
trivy image --exit-code 1 --severity CRITICAL mi-app:latest

# Fallar solo si hay fix disponible
trivy image --exit-code 1 --severity CRITICAL,HIGH --ignore-unfixed mi-app:latest

# Generar reporte y continuar
trivy image --severity CRITICAL,HIGH -f json -o report.json mi-app:latest || true
```

### Archivo .trivyignore

Crear `.trivyignore` en la raíz del proyecto:

```
# Ignorar CVE específico (con justificación)
CVE-2024-12345  # No aplica: No usamos la función vulnerable

# Ignorar por paquete
pkg:golang/github.com/example/pkg@1.0.0

# Ignorar temporalmente hasta que haya fix
CVE-2024-67890  # Monitorear: Sin fix disponible, mitigado con firewall

# Ignorar por severidad en paquete específico
CVE-2024-XXXXX  # LOW severity, no afecta producción
```

### Automatización de Escaneos

```bash
# Script de escaneo periódico
#!/bin/bash
DATE=$(date +%Y%m%d)
IMAGES=("mi-app:latest" "nginx:alpine" "postgres:15")

for IMAGE in "${IMAGES[@]}"; do
  echo "Scanning $IMAGE..."
  trivy image --severity CRITICAL,HIGH \
    -f json -o "scans/${IMAGE//\//_}-$DATE.json" \
    "$IMAGE"
done

# Comparar con escaneo anterior
# diff scans/mi-app_latest-20240101.json scans/mi-app_latest-20240108.json
```

## Optimización de Imágenes

### Selección de Imagen Base

```dockerfile
# EVITAR: Imagen completa (muy pesada)
FROM node:18                  # ~1GB
FROM python:3.11              # ~900MB
FROM eclipse-temurin:17       # ~450MB

# PREFERIR: Imagen slim (reducida)
FROM node:18-slim             # ~250MB
FROM python:3.11-slim         # ~150MB

# PREFERIR: Imagen alpine (mínima)
FROM node:18-alpine           # ~150MB
FROM python:3.11-alpine       # ~50MB
FROM eclipse-temurin:17-jre-alpine  # ~200MB
```

### Multi-Stage Build

#### Node.js

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

#### Python

```dockerfile
# Stage 1: Build
FROM python:3.11-alpine AS build
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Production
FROM python:3.11-alpine
WORKDIR /app
COPY --from=build /root/.local /root/.local
COPY . .

RUN addgroup -g 1001 -S appuser && \
    adduser -S appuser -u 1001
RUN chown -R appuser:appuser /app
USER appuser

ENV PATH=/root/.local/bin:$PATH
EXPOSE 8000
CMD ["python", "app.py"]
```

#### Java (Spring Boot)

```dockerfile
# Stage 1: Build
FROM maven:3.9.6-eclipse-temurin-17-alpine AS build
WORKDIR /app
COPY pom.xml ./
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Production
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

RUN addgroup -g 1001 -S spring && \
    adduser -S spring -u 1001
RUN chown -R spring:spring /app
USER spring

ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -XX:+UseG1GC -XX:+DisableExplicitGC"

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### Usuario Non-Root

#### Alpine Linux

```dockerfile
# Crear grupo y usuario
RUN addgroup -g 1001 -S appuser && \
    adduser -S appuser -u 1001

# Cambiar ownership de archivos
RUN chown -R appuser:appuser /app

# Cambiar a usuario non-root
USER appuser
```

#### Debian/Ubuntu

```dockerfile
# Crear grupo y usuario
RUN groupadd -r appuser -g 1001 && \
    useradd -r -g appuser -u 1001 appuser

# Cambiar ownership
RUN chown -R appuser:appuser /app

# Cambiar a usuario non-root
USER appuser
```

#### Verificar Usuario

```bash
# Ver usuario actual en container
docker exec <container-id> whoami

# Debe mostrar: appuser (no root)

# Ver proceso principal
docker exec <container-id> ps aux

# USER debe ser appuser, no root
```

### Labels de Metadata

```dockerfile
LABEL maintainer="tu-nombre" \
      version="1.0.0" \
      description="Descripción de la aplicación" \
      security.scan="trivy" \
      security.non-root="true" \
      org.opencontainers.image.source="https://github.com/usuario/repo" \
      org.opencontainers.image.licenses="MIT"
```

#### Ver Labels de una Imagen

```bash
# Inspeccionar labels
docker inspect nombre-imagen:tag | grep -A 10 Labels

# Filtrar labels específicos
docker inspect -f '{{.Config.Labels}}' nombre-imagen:tag
```

### Health Checks

#### En Dockerfile

```dockerfile
# Node.js con wget
HEALTHCHECK --interval=30s --timeout=3s --start-period=30s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Node.js con curl
HEALTHCHECK --interval=30s --timeout=3s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Python
HEALTHCHECK --interval=30s --timeout=3s --start-period=30s \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

# Java (Spring Boot Actuator)
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1
```

#### En docker-compose.yml

```yaml
services:
  app:
    build: .
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      start_period: 30s
      retries: 3
```

#### Verificar Health Status

```bash
# Ver estado de salud
docker ps

# STATUS debe mostrar "healthy" después del start_period

# Ver detalles del health check
docker inspect <container-id> | grep -A 20 Health

# Ver logs del health check
docker inspect <container-id> --format='{{json .State.Health}}' | jq
```

### Optimización JVM (Java)

```dockerfile
# Variables de entorno para JVM
ENV JAVA_OPTS="-XX:+UseContainerSupport \
               -XX:MaxRAMPercentage=75.0 \
               -XX:+UseG1GC \
               -XX:+DisableExplicitGC \
               -Xlog:gc*:file=/tmp/gc.log"

# Usar en ENTRYPOINT
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

#### Explicación de Flags

| Flag | Descripción |
|------|-------------|
| `+UseContainerSupport` | JVM detecta límites del container |
| `MaxRAMPercentage=75.0` | Usa máximo 75% de RAM asignada |
| `+UseG1GC` | Garbage Collector optimizado para containers |
| `+DisableExplicitGC` | Previene `System.gc()` explícito |
| `Xlog:gc*` | Log de Garbage Collection |

### .dockerignore

Crea `.dockerignore` en la raíz del proyecto:

```
# Node.js
node_modules
npm-debug.log
.npm

# Python
__pycache__
*.pyc
*.pyo
*.pyd
.Python
venv/
.venv/

# Java
target/
*.class
*.jar
*.war

# Git
.git
.gitignore

# IDE
.vscode/
.idea/
*.swp
*.swo

# Logs
*.log

# OS
.DS_Store
Thumbs.db

# Docker
Dockerfile*
docker-compose*

# CI/CD
.github/
.gitlab-ci.yml

# Documentación
README.md
docs/
*.md

# Tests
tests/
*.test.js
*.spec.js
```

## Cache de Docker Layers

### Cómo Funciona el Cache

Docker construye imágenes en capas (layers). Cada instrucción en el Dockerfile crea una nueva capa:

```dockerfile
FROM node:18-alpine        # Layer 1: Base image
WORKDIR /app               # Layer 2: Working directory
COPY package.json ./       # Layer 3: Package manifest
RUN npm install            # Layer 4: Dependencies installed
COPY . .                   # Layer 5: Application code
CMD ["node", "server.js"]  # Layer 6: Default command
```

**Regla clave:** Si una capa no cambia, Docker la reutiliza del cache. Si cambia, esa capa y **todas las siguientes** se reconstruyen.

### Invalidación del Cache

El cache se invalida cuando:
1. El contenido de un archivo copiado cambia
2. Un comando RUN produce resultado diferente
3. Una variable ARG cambia
4. Se modifica la instrucción del Dockerfile

**Ejemplo:**

```dockerfile
# Si cambias server.js:
COPY package.json ./       # Cache HIT (no cambió)
RUN npm install            # Cache HIT (package.json igual)
COPY . .                   # Cache MISS (server.js cambió)
CMD ["node", "server.js"]  # Se reconstruye (layer anterior cambió)
```

### Orden Óptimo de Instrucciones

**Principio:** Colocar lo que cambia **menos frecuente** al principio, lo que cambia **más frecuente** al final.

#### Frecuencia de Cambios (de menor a mayor)

1. Imagen base (casi nunca cambia)
2. Dependencias del sistema (rara vez cambian)
3. Dependencias de aplicación (cambian ocasionalmente)
4. Código fuente (cambia frecuentemente)
5. Variables de entorno dinámicas (pueden cambiar)

#### Ejemplo: Node.js

```dockerfile
# EVITAR: Cache se invalida con cualquier cambio de código
FROM node:18-alpine
WORKDIR /app
COPY . .                    # Copia TODO (package.json + código)
RUN npm install             # Se reinstala siempre que cambies código
CMD ["node", "server.js"]

# PREFERIR: Cache se aprovecha al máximo
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./  # Solo manifiestos
RUN npm ci --only=production            # Cache persiste si no cambias deps
COPY . .                                # Código al final
CMD ["node", "server.js"]
```

**Resultado:**
- Cambias código → Solo se reconstruye COPY y CMD (segundos)
- Cambias dependencias → Se reconstruye desde npm install (minutos)

#### Ejemplo: Python

```dockerfile
# EVITAR
FROM python:3.11-alpine
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

# PREFERIR
FROM python:3.11-alpine
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

#### Ejemplo: Java (Maven)

```dockerfile
# EVITAR
FROM maven:3.9-eclipse-temurin-17-alpine AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# PREFERIR
FROM maven:3.9-eclipse-temurin-17-alpine AS build
WORKDIR /app
COPY pom.xml ./
RUN mvn dependency:go-offline -B    # Descarga deps (cacheable)
COPY src ./src
RUN mvn clean package -DskipTests
```

### Estrategias Avanzadas de Cache

#### 1. Separar Instalación de Sistema

```dockerfile
# Dependencias de sistema (cambian rara vez)
RUN apk add --no-cache \
    python3 \
    make \
    g++

# Dependencias de aplicación (cambian ocasionalmente)
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Código (cambia frecuentemente)
COPY . .
```

#### 2. Multi-COPY para Archivos Críticos

```dockerfile
# Copiar archivos de configuración primero
COPY package.json package-lock.json ./
COPY tsconfig.json ./
COPY .eslintrc.js ./

RUN npm ci --only=production

# Copiar código después
COPY src ./src
```

#### 3. BuildKit Cache Mounts (Avanzado)

```dockerfile
# syntax=docker/dockerfile:1
FROM node:18-alpine

WORKDIR /app
COPY package.json package-lock.json ./

# Montar cache de npm (persiste entre builds)
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

COPY . .
```

**Habilitar BuildKit:**
```bash
export DOCKER_BUILDKIT=1
docker build -t mi-app .
```

### Forzar Rebuild Sin Cache

```bash
# Rebuild completo sin usar cache
docker build --no-cache -t mi-app:latest .

# Rebuild desde una instrucción específica
docker build --no-cache-filter=npm -t mi-app:latest .

# Actualizar imagen base y rebuild
docker build --pull --no-cache -t mi-app:latest .
```

### Casos de Uso: Cuándo Usar --no-cache

| Situación | Comando | Razón |
|-----------|---------|-------|
| Desarrollo normal | `docker build -t app .` | Aprovechar cache |
| Cambié package.json | `docker build -t app .` | Cache se invalida automáticamente |
| Problemas raros/inconsistencias | `docker build --no-cache -t app .` | Limpiar cache corrupto |
| Deploy a producción | `docker build --pull --no-cache -t app .` | Asegurar versiones más recientes |
| CI/CD pipeline | `docker build --pull -t app .` | Imagen base actualizada |

### Verificar Uso de Cache

Durante el build, observa los mensajes:

```bash
docker build -t mi-app .

# Salida con cache:
#2 [internal] load build context
#3 [1/5] FROM docker.io/library/node:18-alpine
#4 [2/5] WORKDIR /app
#5 [3/5] COPY package*.json ./
#6 CACHED [4/5] RUN npm install              # Cache HIT
#7 [5/5] COPY . .
```

**CACHED** = Docker reutilizó la capa del cache

### Ejemplo Comparativo Completo

#### Build Inicial (sin cache)

```bash
docker build -t app:v1 .

[+] Building 156.3s (12/12) FINISHED
 => [1/6] FROM node:18-alpine                     5.2s
 => [2/6] WORKDIR /app                            0.3s
 => [3/6] COPY package*.json ./                   0.2s
 => [4/6] RUN npm install                       145.6s  # Lento
 => [5/6] COPY . .                                1.5s
 => [6/6] CMD ["node", "server.js"]               0.1s
```

#### Build Después de Cambiar Código (con cache)

```bash
# Modificas server.js
docker build -t app:v2 .

[+] Building 3.1s (12/12) FINISHED
 => [1/6] FROM node:18-alpine                     0.0s
 => [2/6] WORKDIR /app                            0.0s
 => [3/6] COPY package*.json ./                   0.0s
 => CACHED [4/6] RUN npm install                  0.0s  # Rápido!
 => [5/6] COPY . .                                1.8s
 => [6/6] CMD ["node", "server.js"]               0.1s
```

**Tiempo:** 156s → 3s (98% más rápido)

#### Build Después de Cambiar Dependencias (cache parcial)

```bash
# Modificas package.json
docker build -t app:v3 .

[+] Building 98.7s (12/12) FINISHED
 => [1/6] FROM node:18-alpine                     0.0s
 => [2/6] WORKDIR /app                            0.0s
 => [3/6] COPY package*.json ./                   0.2s
 => [4/6] RUN npm install                        95.3s  # Se reconstruye
 => [5/6] COPY . .                                1.5s
 => [6/6] CMD ["node", "server.js"]               0.1s
```

### Tips Prácticos

1. **Ordenar instrucciones por frecuencia de cambio**
   - Base image → Deps sistema → Deps app → Código

2. **Usar COPY específico en lugar de COPY . .**
   ```dockerfile
   # Mejor:
   COPY package.json package-lock.json ./

   # Que:
   COPY . .
   ```

3. **Combinar RUN cuando tiene sentido**
   ```dockerfile
   # PREFERIR: Una capa, limpieza efectiva
   RUN apt-get update && \
       apt-get install -y curl && \
       apt-get clean

   # EVITAR: Tres capas, limpieza no reduce tamaño
   RUN apt-get update
   RUN apt-get install -y curl
   RUN apt-get clean
   ```

4. **Usar .dockerignore para evitar invalidaciones innecesarias**
   ```
   node_modules
   .git
   *.log
   ```

5. **BuildKit para cache avanzado**
   ```bash
   DOCKER_BUILDKIT=1 docker build -t app .
   ```

## Comandos de Análisis

### Tamaño de Imágenes

```bash
# Listar imágenes con tamaños
docker images

# Ver tamaño específico
docker images nombre-imagen:tag

# Ver capas de una imagen
docker history nombre-imagen:tag

# Ver capas con tamaños sin truncar
docker history --no-trunc --format "table {{.Size}}\t{{.CreatedBy}}" nombre-imagen:tag

# Comparar tamaños
docker images | grep nombre-imagen
```

### Análisis de Capas

```bash
# Ver todas las capas
docker history nombre-imagen:tag

# Ver solo las capas grandes (>10MB)
docker history nombre-imagen:tag | awk '$2 ~ /MB/ {print}'

# Usar dive para análisis interactivo (requiere instalación)
dive nombre-imagen:tag
```

### Inspeccionar Imagen

```bash
# Ver toda la información
docker inspect nombre-imagen:tag

# Ver solo configuración
docker inspect -f '{{json .Config}}' nombre-imagen:tag | jq

# Ver variables de entorno
docker inspect -f '{{.Config.Env}}' nombre-imagen:tag

# Ver usuario
docker inspect -f '{{.Config.User}}' nombre-imagen:tag

# Ver CMD y ENTRYPOINT
docker inspect -f '{{.Config.Cmd}}' nombre-imagen:tag
docker inspect -f '{{.Config.Entrypoint}}' nombre-imagen:tag
```

## Checklist de Seguridad

### Antes de Deploy a Producción

```bash
# 1. Escanear con Trivy (0 CRITICAL)
trivy image --severity CRITICAL nombre-imagen:tag

# 2. Verificar usuario non-root
docker run --rm nombre-imagen:tag whoami
# Debe mostrar: appuser (no root)

# 3. Verificar health check funciona
docker run -d -p 8080:8080 nombre-imagen:tag
sleep 40  # Esperar start_period
docker ps
# STATUS debe mostrar "healthy"

# 4. Verificar tamaño de imagen
docker images nombre-imagen:tag
# Debe ser razonable (< 500MB para apps simples)

# 5. Verificar no hay secretos
trivy config Dockerfile
grep -r "password\|secret\|key" . --exclude-dir=node_modules

# 6. Verificar logs no contienen datos sensibles
docker logs <container-id> | grep -i "password\|secret\|token"
```

### Lista de Verificación

- Imagen base alpine o slim
- Multi-stage build implementado
- Usuario non-root configurado
- Health check funcional
- Labels de metadata agregados
- .dockerignore configurado
- Escaneo Trivy (0 CRITICAL)
- Variables de entorno para configuración
- Sin secretos hardcoded
- Logs no contienen datos sensibles
- Tamaño de imagen optimizado

## Buenas Prácticas

### 1. Multi-Stage Build

**Por qué:** Reduce tamaño y superficie de ataque

```dockerfile
# PREFERIR: Multi-stage
FROM node:18-alpine AS build
# ... build steps ...
FROM node:18-alpine
COPY --from=build /app/dist ./dist

# EVITAR: Single stage
FROM node:18-alpine
# Todo queda en imagen final
```

### 2. Imagen Base Mínima

**Por qué:** Menos vulnerabilidades, menor tamaño

```dockerfile
# PREFERIR: Alpine
FROM node:18-alpine

# ACEPTABLE: Slim
FROM node:18-slim

# EVITAR: Full
FROM node:18
```

### 3. Usuario Non-Root

**Por qué:** Previene escalación de privilegios

```dockerfile
# PREFERIR: Non-root
USER appuser

# EVITAR: Root
USER root  # o no especificar USER
```

### 4. Orden de Layers

**Por qué:** Aprovecha cache de Docker

```dockerfile
# PREFERIR: Dependencias primero
COPY package.json ./
RUN npm install
COPY . .

# EVITAR: Todo junto
COPY . .
RUN npm install
```

### 5. Limpieza en Misma Capa

**Por qué:** Reduce tamaño final

```dockerfile
# PREFERIR: Limpia en misma capa
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# EVITAR: Limpia en capa separada
RUN apt-get update
RUN apt-get install -y package
RUN apt-get clean  # No reduce tamaño de capas anteriores
```

## Troubleshooting

### Error: "permission denied" con non-root user

```bash
# Problema: Archivos no tienen permisos correctos
# Solución: Cambiar ownership ANTES de USER

# CORRECTO:
RUN chown -R appuser:appuser /app
USER appuser

# INCORRECTO:
USER appuser
RUN chown -R appuser:appuser /app  # No tiene permisos para hacer esto
```

### Error: Health check siempre "unhealthy"

```bash
# Ver logs del health check
docker inspect <container-id> --format='{{json .State.Health}}' | jq

# Probar manualmente dentro del container
docker exec <container-id> wget --no-verbose --tries=1 --spider http://localhost:3000/health

# Verificar que:
# 1. Puerto correcto
# 2. Ruta correcta (/health existe)
# 3. start_period suficiente (app tarda en iniciar)
# 4. wget o curl instalado en la imagen
```

### Alpine: "module not found" o "library not found"

```bash
# Problema: Alpine usa musl en lugar de glibc
# Solución: Instalar dependencias necesarias

RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    # Otras dependencias nativas

# O usar imagen slim en lugar de alpine
FROM node:18-slim
```

### Trivy: "database download failed"

```bash
# Actualizar base de datos manualmente
trivy image --download-db-only

# Limpiar cache y reintentar
trivy image --clear-cache
trivy image nombre-imagen:tag

# Verificar conectividad
curl -I https://ghcr.io
```

## Comparación de Tamaños

### Imágenes Base

| Tecnología | Full | Slim | Alpine |
|------------|------|------|--------|
| Node.js 18 | 1.1GB | 250MB | 150MB |
| Python 3.11 | 900MB | 150MB | 50MB |
| Java 17 (JDK) | 450MB | N/A | 350MB |
| Java 17 (JRE) | 300MB | N/A | 200MB |

### Reducción Típica con Optimización

| Optimización | Reducción |
|--------------|-----------|
| Full → Alpine | 70-85% |
| Single → Multi-stage | 40-60% |
| Con .dockerignore | 10-30% |
| JDK → JRE (Java) | 30-40% |

## Alternativas a Trivy (Referencia)

### Docker Scout

```bash
# Escaneo básico
docker scout quickview nginx:alpine

# Ver vulnerabilidades
docker scout cves nginx:alpine

# Solo CRITICAL y HIGH
docker scout cves --only-severity critical,high nginx:alpine

# Comparar versiones
docker scout compare --to nginx:alpine nginx:latest
```

**Limitaciones:** 1 repositorio gratuito, requiere Docker Hub account.

### Comparación de Herramientas

| Herramienta | Costo | Limitaciones | Mejor para |
|-------------|-------|--------------|------------|
| **Trivy** | Gratis ilimitado | Ninguna | Uso general, CI/CD, educación |
| **Docker Scout** | 1 repo gratis | Login requerido | Uso casual con Docker Desktop |
| **Snyk** | Plan gratuito limitado | Rate limits | Análisis de dependencias |
| **Grype** | Gratis ilimitado | Menos formatos | Alternativa open source |

**Por qué Trivy:**
- Sin límites ni cuentas requeridas
- Open source y estándar de industria
- Ejecutable como container

## Recursos Adicionales

- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Docker Scout Documentation](https://docs.docker.com/scout/)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [CVE Database](https://nvd.nist.gov/)
