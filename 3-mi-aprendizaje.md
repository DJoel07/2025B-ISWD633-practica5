# Mi Aprendizaje - Práctica 5: Docker Compose
# Dorian Joel Alban Lucas

## Conocimientos Previos
Antes de realizar esta práctica, tenía conocimientos básicos sobre Docker y contenedores individuales tal y como se ha visto en las antreriores practicas realizadas. Sabía cómo ejecutar contenedores simples usando comandos de Docker, pero no había trabajado con aplicaciones multi-contenedor ni entendía completamente cómo hacer que múltiples servicios se comunicaran entre sí de manera eficiente.

## Principales Aprendizajes

### 1. Docker Compose y Orquestación de Contenedores
Aprendí que Docker Compose es una herramienta fundamental para definir y gestionar aplicaciones que requieren múltiples contenedores. En lugar de ejecutar varios comandos `docker run` con múltiples parámetros difíciles de recordar, puedo definir toda la configuración en un archivo YAML y levantar toda la aplicación con un solo comando `docker compose up -d`. Esto hace que el desarrollo sea más reproducible y fácil de compartir con otros miembros del equipo.

### 2. Sintaxis y Estructura de Archivos YAML
Durante la práctica comprendí la importancia de la indentación correcta en archivos YAML. Aprendí que YAML usa espacios (no tabuladores) y que la jerarquía de los elementos se define por el nivel de indentación. Un error común que cometí al inicio fue no alinear correctamente las propiedades bajo cada servicio, lo que causaba errores de validación.

### 3. Redes en Docker
Entendí cómo funcionan las redes de tipo bridge en Docker. Aprendí que cuando varios servicios pertenecen a la misma red, pueden comunicarse entre sí usando sus nombres de servicio como hostname. Por ejemplo, en la práctica con WordPress y MySQL, el servicio de WordPress puede conectarse a la base de datos simplemente usando `mysql-service` como host, sin necesidad de conocer la IP del contenedor. Esto simplifica mucho la configuración y hace que sea más portable.

### 4. Dependencias y Healthchecks
Uno de los conceptos más importantes que aprendí fue el uso de `depends_on` con la condición `service_healthy`. Esto garantiza que un servicio no intente iniciarse hasta que sus dependencias estén completamente funcionales. Los healthchecks son verificaciones periódicas que Docker realiza para determinar si un contenedor está funcionando correctamente. Por ejemplo:
- Para MySQL usamos `mysqladmin ping` para verificar que el servidor de base de datos responde
- Para PostgreSQL usamos `pg_isready` para verificar que está listo para aceptar conexiones
- Para servicios web como WordPress y SonarQube usamos `curl` o `wget` para verificar que el servidor HTTP responde

### 5. Volúmenes para Persistencia de Datos
Aprendí la diferencia entre volúmenes nombrados y bind mounts. Los volúmenes nombrados son manejados por Docker y persisten los datos incluso cuando los contenedores se eliminan. Esto es crucial para bases de datos y aplicaciones que necesitan mantener su estado. En la práctica configuré volúmenes para:
- Datos de MySQL y PostgreSQL
- Archivos de WordPress
- Datos, extensiones y logs de SonarQube

### 6. Variables de Entorno para Configuración
Comprendí cómo las variables de entorno permiten configurar los contenedores sin modificar las imágenes. Cada servicio tiene variables específicas que necesita, como credenciales de base de datos, URLs de conexión, etc. Esto hace que la configuración sea flexible y segura.

## Problemas Encontrados y Soluciones

### Problema 1: Error de Validación en depends_on
**Error encontrado:**
```
services.wordpress-service.depends_on.mysql-service.condition value must be one of 'service_started', 'service_healthy', 'service_completed_successfully'
```

**Causa:** La sintaxis en la sección `depends_on` no era correcta. Inicialmente tenía problemas con la indentación o posiblemente tenía comillas alrededor del valor `service_healthy`.

**Solución:** Corregí la estructura asegurándome de que:
- `depends_on:` tuviera la indentación correcta (4 espacios)
- El nombre del servicio `mysql-service:` estuviera indentado con 6 espacios
- La condición `condition: service_healthy` estuviera indentada con 8 espacios
- No hubiera comillas alrededor de `service_healthy`

### Problema 2: Red No Definida
**Error encontrado:**
```
service "wordpress-service" refers to undefined network wordpress-network: invalid compose project
```

**Causa:** En el archivo compose.yaml, los servicios hacían referencia a una red llamada `wordpress-network`, pero en la sección de redes estaba definida con un nombre diferente (`net-wp`). Además, había inconsistencias en los nombres de los volúmenes.

**Solución:** Revisé todo el archivo y aseguré la consistencia en los nombres:
- Cambié el nombre de la red de `net-wp` a `wordpress-network`
- Cambié el nombre del volumen de `mysql-data` a `mysql-vol`
- Me aseguré de que todos los servicios hicieran referencia al mismo nombre de red

### Problema 3: Conflicto de Nombres de Contenedores
**Error encontrado:**
```
The container name "/mysql-container" is already in use by container "0f042f07321b..."
```

**Causa:** Existía un contenedor previo con el mismo nombre debido a las anteriores practicas no se había eliminado correctamente.

**Solución:** Eliminé el contenedor existente usando `docker rm -f mysql-container` antes de volver a ejecutar `docker compose up -d`.

### Problema 4: Atributo version Obsoleto
**Advertencia encontrada:**
```
the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
```

**Causa:** En versiones recientes de Docker Compose, el atributo `version` en la parte superior del archivo ya no es necesario y está obsoleto.

**Solución:** Eliminé la línea `version: '3.8'` del archivo compose.yaml. Las versiones modernas de Docker Compose detectan automáticamente la versión del formato del archivo.
