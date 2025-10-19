# Tarea 5 - Seguridad y Optimización de Imágenes

## Objetivo

Aplicar técnicas de seguridad y optimización a una aplicación real, escaneándola con Trivy y mejorando su Dockerfile.

---

## Parte 1: Selección de Aplicación

### Opción A: Optimizar tu Tarea 4

Si completaste la Tarea 4 (microservicios), mejora la seguridad y optimización de tus servicios backend.

### Opción B: Crear Nueva Aplicación

Crea una aplicación simple con:
- Backend en Node.js/Python/Go
- Base de datos (MongoDB/PostgreSQL)
- docker-compose.yml

**Mínimo requerido:**
- 1 servicio backend con Dockerfile
- 1 base de datos
- docker-compose.yml funcional

---

## Parte 2: Análisis de Línea Base

### 2.1 Construir Imagen Inicial

```bash
cd tu-proyecto/
docker build -t mi-app:baseline .
```

### 2.2 Escanear con Trivy

```bash
# Escaneo completo
trivy image mi-app:baseline

# Solo vulnerabilidades críticas y altas
trivy image --severity CRITICAL,HIGH mi-app:baseline

# Generar reporte JSON
trivy image -f json -o baseline-scan.json mi-app:baseline
```

### 2.3 Documentar Línea Base

En tu README.md, documenta:

**Antes de optimización:**
- Tamaño de imagen: `docker images mi-app:baseline`
- Número de vulnerabilidades CRITICAL: X
- Número de vulnerabilidades HIGH: Y
- Usuario que ejecuta el proceso: `docker run mi-app:baseline whoami`
- Captura de pantalla del escaneo de Trivy

---

## Parte 3: Aplicar Optimizaciones

### 3.1 Mejoras Requeridas (Obligatorias)

Modifica tu Dockerfile para incluir:

#### a) Multi-Stage Build

```dockerfile
# Stage 1: Build
FROM node:18 AS build
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
# ... resto de configuración
```

#### b) Imagen Base Alpine

Cambia de imagen completa a alpine:
- `node:18` → `node:18-alpine`
- `python:3.11` → `python:3.11-alpine`
- Usar eclipse-temurin JRE alpine para Java

#### c) Usuario Non-Root

```dockerfile
RUN addgroup -g 1001 -S appuser && \
    adduser -S appuser -u 1001
RUN chown -R appuser:appuser /app
USER appuser
```

#### d) Labels de Metadata

```dockerfile
LABEL maintainer="tu-nombre" \
      version="1.0-optimizado" \
      description="Descripción de tu app" \
      security.scan="trivy" \
      security.non-root="true"
```

#### e) Health Check

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=30s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
```

### 3.2 Mejoras Opcionales (Puntos Extra)

- Variables de entorno para configuración
- .dockerignore para reducir contexto de build
- Optimización de instalación de dependencias
- Read-only filesystem donde sea posible

---

## Parte 4: Análisis Post-Optimización

### 4.1 Construir Imagen Optimizada

```bash
docker build -t mi-app:optimizado .
```

### 4.2 Escanear con Trivy

```bash
trivy image --severity CRITICAL,HIGH mi-app:optimizado
trivy image -f json -o optimizado-scan.json mi-app:optimizado
```

### 4.3 Comparar Resultados

Crea una tabla comparativa en tu README.md:

| Métrica | Baseline | Optimizado | Mejora |
|---------|----------|------------|--------|
| Tamaño imagen | 1.2GB | 350MB | -71% |
| Vulnerabilidades CRITICAL | 5 | 0 | -100% |
| Vulnerabilidades HIGH | 12 | 3 | -75% |
| Usuario | root | appuser | ✓ |
| Multi-stage | ✗ | ✓ | ✓ |
| Health check | ✗ | ✓ | ✓ |

---

## Parte 5: Verificación de Funcionalidad

### 5.1 Probar Imagen Optimizada

```bash
# Levantar con docker-compose
docker compose up -d

# Verificar que funciona correctamente
curl http://localhost:3000/health

# Verificar usuario non-root
docker exec <container-id> whoami
# Debe mostrar: appuser (no root)

# Verificar health check
docker ps
# Debe mostrar "healthy" en la columna STATUS
```

### 5.2 Capturas de Pantalla

Incluye en `docs/screenshots/`:
1. `trivy-baseline.png` - Escaneo antes de optimizar
2. `trivy-optimizado.png` - Escaneo después de optimizar
3. `docker-images.png` - Comparación de tamaños
4. `docker-ps-healthy.png` - Contenedor con status healthy
5. `whoami-nonroot.png` - Verificación de usuario non-root

---

## Parte 6: Documentación en README.md

Tu README.md debe incluir:

### 1. Título y Descripción
- Nombre del proyecto
- Descripción de la aplicación
- Objetivo de optimización

### 2. Tecnologías Utilizadas
```markdown
- Node.js 18 (alpine)
- MongoDB 6
- Docker & Docker Compose
- Trivy para escaneo de seguridad
```

### 3. Mejoras Aplicadas

Lista detallada de cada optimización:

```markdown
## Mejoras de Seguridad y Optimización

### 1. Multi-Stage Build
- **Antes**: Imagen única con herramientas de build
- **Después**: Build separado, solo archivos necesarios en producción
- **Beneficio**: Reducción de ~800MB

### 2. Imagen Base Alpine
- **Antes**: node:18 (1.1GB)
- **Después**: node:18-alpine (150MB)
- **Beneficio**: -85% tamaño, menos vulnerabilidades

### 3. Usuario Non-Root
- **Antes**: Ejecuta como root (riesgo de seguridad)
- **Después**: Usuario appuser (UID 1001)
- **Beneficio**: Previene escalación de privilegios

### 4. Health Check
- **Implementación**: Verificación cada 30s
- **Beneficio**: Docker detecta containers no saludables

### 5. Labels de Seguridad
- **Implementación**: Metadata en imagen
- **Beneficio**: Trazabilidad y auditoría
```

### 4. Tabla Comparativa

Incluye la tabla de comparación antes/después.

### 5. Análisis de Vulnerabilidades

```markdown
## Análisis de Vulnerabilidades

### Vulnerabilidades Críticas Resueltas

1. **CVE-2024-XXXXX** - Vulnerability en libssl
   - Severidad: CRITICAL
   - Fix: Actualización a alpine:3.19

2. **CVE-2024-YYYYY** - Buffer overflow en biblioteca X
   - Severidad: HIGH
   - Fix: Removida al usar imagen alpine minimal
```

### 6. Instrucciones de Uso

```bash
# Clonar repositorio
git clone <tu-repo>

# Construir imagen optimizada
docker build -t mi-app:optimizado .

# Escanear con Trivy
trivy image mi-app:optimizado

# Levantar servicios
docker compose up -d

# Verificar salud
docker ps
curl http://localhost:3000/health
```

### 7. Verificación de Seguridad

Checklist que puedes copiar:

```markdown
## Checklist de Seguridad

- [x] Escaneo con Trivy (0 CRITICAL)
- [x] Usuario non-root verificado
- [x] Multi-stage build implementado
- [x] Imagen base alpine utilizada
- [x] Health check funcional
- [x] Labels de metadata agregados
- [x] .dockerignore configurado
- [x] Sin secretos hardcoded en código
```

---

## Parte 7: Criterios de Evaluación

**Aplicación de Técnicas (40%)**
- Multi-stage build implementado correctamente
- Imagen base alpine utilizada
- Usuario non-root configurado
- Health check funcional
- Labels apropiados

**Análisis con Trivy (30%)**
- Escaneo baseline documentado
- Escaneo optimizado documentado
- Comparación clara de resultados
- Reducción significativa de vulnerabilidades (mínimo 50%)

**Documentación (20%)**
- README completo y claro
- Tabla comparativa incluida
- Capturas de pantalla relevantes
- Análisis de mejoras detallado

**Reducción de Tamaño (10%)**
- Reducción mínima de 30% en tamaño de imagen
- Reducción de 50%+ otorga puntos extra

---

## Parte 8: Entrega

### Formato de Entrega

1. Repositorio público en GitHub o GitLab
2. Nombre del repositorio: `docker-seguridad-clase5`
3. README.md completo con todas las secciones
4. Capturas de pantalla en `docs/screenshots/`
5. Archivos JSON de escaneos Trivy incluidos

### Estructura del Repositorio

```
docker-seguridad-clase5/
├── README.md
├── Dockerfile (optimizado)
├── Dockerfile.baseline (opcional, para referencia)
├── docker-compose.yml
├── .dockerignore
├── src/
│   └── ... (código de tu aplicación)
├── docs/
│   └── screenshots/
│       ├── trivy-baseline.png
│       ├── trivy-optimizado.png
│       ├── docker-images.png
│       ├── docker-ps-healthy.png
│       └── whoami-nonroot.png
└── scans/
    ├── baseline-scan.json
    └── optimizado-scan.json
```

### Cómo Entregar

1. Completa todas las optimizaciones
2. Documenta todo en README.md
3. Sube a GitHub/GitLab (repo público)
4. Verifica que el repo sea accesible
5. Entrega URL en Moodle

### Antes de Entregar, Verifica

- `docker build` completa sin errores
- Escaneo con Trivy muestra mejora significativa
- `docker compose up -d` levanta todos los servicios
- Health check muestra "healthy" en `docker ps`
- `docker exec <container> whoami` NO muestra "root"
- README.md incluye todas las secciones requeridas
- Capturas de pantalla incluidas y legibles
- Tabla comparativa muestra mejoras claras

---

## Recursos Adicionales

- [Lab 01: Trivy Scan](../labs/01-trivy-scan/)
- [Lab 02: Optimización](../labs/02-optimizacion/)
- [Cheatsheet Clase 5](../cheatsheet.md)
- [Docker Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Alpine Linux Docker Images](https://hub.docker.com/_/alpine)
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

---

## Ayuda y Dudas

Si tienes dudas:
1. Revisa los labs de la clase 5
2. Consulta el cheatsheet
3. Revisa la documentación de Trivy
4. Pregunta en el foro de Moodle
5. Asiste a horarios de consulta

---

## Fecha de Entrega

**Antes de la Clase 6**

Consulta la fecha exacta en Moodle.
