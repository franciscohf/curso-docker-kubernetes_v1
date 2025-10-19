# Bloque 2: Kubernetes

Este bloque cubre la orquestación de containers (contenedores) con Kubernetes, desde los conceptos fundamentales hasta el deployment (despliegue), escalado y observabilidad de aplicaciones en producción.

---

## Objetivos del bloque

Al finalizar este bloque, serás capaz de:

- Comprender la arquitectura y componentes de Kubernetes
- Desplegar y gestionar aplicaciones con Pods, Deployments y Services
- Configurar aplicaciones con ConfigMaps y Secrets
- Implementar health checks con Probes
- Configurar escalado automático (HPA) e Ingress
- Implementar observabilidad con logs, métricas, Prometheus y Grafana
- Aplicar buenas prácticas de deployment en Kubernetes

---

## Contenido

### [Clase 6: Introducción a Kubernetes](clase6-introduccion/)

Fundamentos de Kubernetes, arquitectura del cluster, y deployment básico de aplicaciones.

**Temas:**
- Arquitectura de Kubernetes (Control Plane y Worker Nodes)
- Instalación y configuración de minikube y kubectl
- Pods: Unidad mínima de despliegue
- Deployments: Gestión declarativa de aplicaciones
- Services: Exposición de aplicaciones (ClusterIP, NodePort, LoadBalancer)
- Labels, selectors y namespaces

**Labs:**
- Lab 01: Verificación del setup
- Lab 02: Primer Pod
- Lab 03: Deployments
- Lab 04: Service ClusterIP
- Lab 05: Service NodePort
- Lab 06: Service LoadBalancer con MetalLB

---

### [Clase 7: Namespaces, Configuración y Persistencia](clase7-configuracion-persistencia/)

Organización de recursos, externalización de configuración y gestión de aplicaciones con estado.

**Temas:**
- Namespaces para organización de recursos
- ConfigMaps para configuración no sensible
- Secrets para credenciales y datos sensibles
- StatefulSets para aplicaciones con estado
- PersistentVolumeClaims para persistencia de datos
- Headless Services para DNS estable

**Labs:**
- Lab 01: Namespaces - Organización y Aislamiento
- Lab 02: ConfigMaps y Secrets
- Lab 03: StatefulSet y Persistencia con PostgreSQL

---

### [Clase 8: Ingress, Health Probes y Escalado Automático](clase8-ingress-escalado/)

Routing HTTP avanzado, health checks automáticos y escalado dinámico de aplicaciones.

**Temas:**
- Ingress para routing HTTP/HTTPS
- NGINX Ingress Controller
- Health Probes (Liveness, Readiness, Startup)
- Horizontal Pod Autoscaler (HPA)
- Metrics Server
- Observabilidad con Prometheus, Grafana y Loki (demo)

**Labs:**
- Lab 01: Ingress - Routing HTTP
- Lab 02: Health Probes
- Lab 03: HPA - Horizontal Pod Autoscaler

---

## Proyecto Integrador - Evolución en Kubernetes

El Proyecto Integrador evoluciona en este bloque migrando la arquitectura de microservicios de Docker Compose a Kubernetes:

| Versión | Clases | Stack |
|---------|--------|-------|
| v2.0-clases6-7-8 | 6, 7, 8 | Migración completa a K8s: Deployments, Services, ConfigMaps, Secrets, StatefulSet, Ingress, HPA |

**Nota:** La versión v2.0 consolida todos los conceptos de Kubernetes en una sola versión funcional que se presenta progresivamente durante las Clases 6, 7 y 8.

---

## Requisitos Previos

Antes de comenzar este bloque, asegúrate de:

- Tener completado el Bloque 1 (Docker)
- Instalar minikube y kubectl (ver [INSTALL_KUBERNETES.md](../INSTALL_KUBERNETES.md))
- Verificar que minikube inicia correctamente
- Tener Docker funcionando (requerido por minikube)
- Tener conocimientos de YAML y línea de comandos

---

## Herramientas Utilizadas

**Cluster local:**
- minikube (Kubernetes local)
- kubectl (CLI de Kubernetes)

**Opcional:**
- k9s (Terminal UI para Kubernetes)
- Lens (Kubernetes IDE)
- Helm (Package manager para Kubernetes)

---

## Recursos adicionales

- [Documentación oficial de Kubernetes](https://kubernetes.io/docs/)
- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [CNCF Cloud Native Interactive Landscape](https://landscape.cncf.io/)

---

[← Volver al curso](../README.md)
