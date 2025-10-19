# Clase 4: Microservicios, Cache y API Gateway

## Objetivos de Aprendizaje

- Construir aplicaciones multi-servicio con bases de datos
- Implementar cache (caché) con Redis para mejorar rendimiento
- Configurar un API Gateway con Nginx para centralizar requests
- Comprender patrones de arquitectura de microservicios
- Gestionar comunicación entre servicios mediante redes Docker

## Conceptos Clave

- **Microservicios**: Arquitectura donde la aplicación se divide en servicios independientes que se comunican entre sí
- **Cache (Caché)**: Almacenamiento temporal de datos frecuentemente accedidos para mejorar rendimiento
- **API Gateway**: Punto de entrada único que distribuye requests a diferentes servicios backend
- **Service Discovery**: Resolución de nombres de servicios mediante DNS interno de Docker
- **Data Persistence**: Estrategias para persistir datos en aplicaciones distribuidas

## Buenas Prácticas Aplicadas

**Todos los labs de esta clase aplican las buenas prácticas aprendidas en Clase 2:**

- **Multi-stage builds**: Separación de etapas build y production para imágenes más livianas
- **Non-root users**: Todos los contenedores ejecutan con usuario `nodejs` (UID 1001) por seguridad
- **npm ci**: Uso de `npm ci --only=production` para instalaciones determinísticas
- **Cache cleaning**: `npm cache clean --force` para reducir tamaño de imágenes
- **HEALTHCHECK**: Checks automáticos de salud en todos los servicios
- **Variables de entorno**: ENV explícitas para mejor documentación
- **Ownership correcto**: `chown` para permisos apropiados del usuario no-root

Estas prácticas refuerzan lo aprendido y preparan para Clase 5 (Seguridad y Optimización).

## Laboratorios de la clase

### [Lab 01: Node.js + MongoDB](labs/01-nodejs-mongodb/)
Aplicación full-stack con API REST y base de datos MongoDB. Lab recuperado de la clase anterior.

### [Lab 02: Redis como Cache](labs/02-redis-cache/)
Implementar cache con Redis para optimizar consultas a base de datos.

### [Lab 03: Nginx como API Gateway](labs/03-nginx-gateway/)
Configurar Nginx como gateway para distribuir tráfico entre servicios frontend y backend.

## Tareas

- [Desafío Rápido](tareas/desafio-rapido.md)
- [Tarea para Casa](tareas/tarea-casa.md)

## Cheatsheet

- [Comandos y Conceptos - Clase 4](cheatsheet.md)

## Recursos

- [Docker Compose Networking](https://docs.docker.com/compose/networking/)
- [Redis Official Image](https://hub.docker.com/_/redis)
- [Nginx Official Image](https://hub.docker.com/_/nginx)
- [MongoDB Official Image](https://hub.docker.com/_/mongo)
- [Best Practices for Microservices with Docker](https://docs.docker.com/get-started/microservices/)
