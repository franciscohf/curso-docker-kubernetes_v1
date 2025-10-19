# Clase 5: Seguridad y Optimización

## Objetivos de Aprendizaje

- Escanear imágenes Docker en busca de vulnerabilidades con Trivy
- Aplicar técnicas de optimización para reducir tamaño de imágenes
- Comprender niveles de severidad de vulnerabilidades (CRITICAL, HIGH, MEDIUM, LOW)
- Implementar buenas prácticas de seguridad en containers
- Analizar y comparar resultados antes y después de optimización

## Conceptos Clave

- **Trivy**: Herramienta de escaneo de vulnerabilidades para containers
- **CVE (Common Vulnerabilities and Exposures)**: Base de datos de vulnerabilidades conocidas
- **Multi-stage builds**: Técnica para reducir tamaño y superficie de ataque de imágenes
- **Alpine Linux**: Distribución mínima para containers con menor superficie de ataque
- **Non-root users**: Ejecución de containers con usuarios sin privilegios
- **Image layers**: Capas de una imagen Docker y su impacto en tamaño
- **Health checks**: Verificación automática del estado de un container
- **Attack surface**: Superficie de ataque, minimizada al reducir paquetes instalados

## Laboratorios de la clase

### [Lab 01: Escaneo de Vulnerabilidades con Trivy](labs/01-trivy-scan/)
Aprender a instalar y usar Trivy para escanear imágenes en busca de vulnerabilidades, aplicándolo al Proyecto Integrador.

### [Lab 02: Optimización de Imágenes Docker](labs/02-optimizacion/)
Aplicar técnicas de optimización comparando resultados antes y después: multi-stage builds, alpine, non-root user.

## Tareas

- [Desafío Rápido](tareas/desafio-rapido.md)
- [Tarea para Casa](tareas/tarea-casa.md)

## Cheatsheet

- [Comandos y Conceptos - Clase 5](cheatsheet.md)

## Recursos

- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [Docker Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Alpine Linux Docker Images](https://hub.docker.com/_/alpine)
- [CVE Database (NIST)](https://nvd.nist.gov/)
