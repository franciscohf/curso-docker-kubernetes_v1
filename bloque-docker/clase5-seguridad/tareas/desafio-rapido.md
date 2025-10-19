# Desafío Rápido - Clase 5

Tiempo estimado: 5 minutos

## Objetivo

Escanear una imagen Docker con Trivy e identificar vulnerabilidades críticas.

## Instrucciones

1. Asegúrate de tener Trivy instalado:
```bash
trivy --version
```

Si no está instalado, sigue las instrucciones del [Lab 01](../labs/01-trivy-scan/).

2. Escanea la imagen oficial de nginx:alpine:
```bash
trivy image --severity CRITICAL,HIGH nginx:alpine
```

3. Escanea una imagen completa de nginx (no alpine):
```bash
trivy image --severity CRITICAL,HIGH nginx:latest
```

4. Compara los resultados:
   - ¿Cuántas vulnerabilidades CRITICAL tiene cada una?
   - ¿Cuántas vulnerabilidades HIGH tiene cada una?
   - ¿Qué diferencia de tamaño tienen?

5. Verifica el tamaño de ambas imágenes:
```bash
docker images | grep nginx
```

## Preguntas de Reflexión

1. ¿Cuál imagen tiene menos vulnerabilidades?
2. ¿Cuál es más pequeña?
3. ¿Por qué crees que hay esta diferencia?

## Resultado Esperado

Debes observar que:
- nginx:alpine tiene significativamente menos vulnerabilidades
- nginx:alpine es mucho más pequeña (~40MB vs ~180MB)
- La imagen alpine es más segura por tener menos paquetes instalados

## Entrega

No es necesario entregar este desafío, es solo para practicar en clase.
