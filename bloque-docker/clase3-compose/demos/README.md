# Fundamentos - Clase 3

Ejercicios conceptuales que aclaran fundamentos de volúmenes, redes y Docker Compose.

Estos ejercicios son diferentes a los labs - están diseñados para entender conceptos clave **antes** de trabajar con aplicaciones completas.

---

## Contenido

### [01: Tipos de Volúmenes](01-tipos-volumenes/)

Exploración práctica de los 3 tipos de volúmenes en Docker:
- Named volumes (volúmenes nombrados)
- Bind mounts (montajes enlazados)
- Anonymous volumes (volúmenes anónimos)

Comparación, casos de uso y diferencias clave.

---

### [02: Tipos de Redes](02-tipos-redes/)

Exploración de tipos de redes y operaciones:
- Bridge (por defecto)
- Host (red del anfitrión)
- None (sin red)
- Attach/Detach dinámico de contenedores

---

### [03: Volúmenes y Redes - Contenedores vs Imágenes](03-volumenes-redes-conceptual/)

Aclaración conceptual importante:
- ¿Se configuran volúmenes/redes en imágenes o contenedores?
- ¿Puedo incluir volúmenes en un Dockerfile?
- Diferencia entre tiempo de build vs tiempo de ejecución

---

### [04: Anatomía de docker-compose.yml](04-anatomia-compose/)

Disección detallada de un archivo docker-compose.yml:
- Secciones principales (services, networks, volumes)
- Campos más comunes por servicio
- Sintaxis YAML básica
- Comentarios línea por línea

---

## Diferencia con los Labs

| Aspecto | Fundamentos | Labs |
|---------|-------------|------|
| **Herramienta** | Comandos `docker` directos | Docker Compose |
| **Enfoque** | Un solo concepto | Aplicación completa |
| **Objetivo** | Entender fundamentos | Aplicar conocimientos |

**Recomendación:** Revisar estos fundamentos antes de comenzar los labs para una mejor comprensión.

---

[← Volver a Clase 3](../README.md)
