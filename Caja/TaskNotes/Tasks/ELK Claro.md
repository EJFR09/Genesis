---
status: open
priority: normal
dateCreated: 2026-10-28T07:51:17.807-03:00
dateModified: 2026-10-28T07:51:42.890-03:00
tags:
  - task
---
# Informe técnico — Relevamiento de PAYMENT en Testing

**Proyecto:** Claro PAYMENT — Centralización de logs en ELK  
**Fecha:** 05/11/2026  
**Ambiente:** Testing  
**Bloque:** 1 — Relevamiento de infraestructura y logs  
**Estado:** Relevamiento inicial realizado, con verificaciones pendientes.

---

## 1. Objetivo

Relevar la infraestructura de PAYMENT en Testing, identificar los servicios desplegados mediante Podman y determinar dónde se generan y almacenan sus logs.

El relevamiento permitirá diseñar posteriormente la captura e ingesta de logs mediante Filebeat hacia el clúster Elasticsearch `es02-testing`.

**Alcance inicial:**

- API REST: `claro-payment-api`.
    
- Servicios Socket: `claro-posjota`, inicialmente la instancia `documenta`.
    
- El proxy Nginx se identifica como componente de infraestructura, pero no forma parte de las fuentes prioritarias de ingesta.
    

## 2. Infraestructura identificada

### 2.1. Servidor

|Propiedad|Valor|
|---|---|
|Hostname|`paymendb01ptl.claro.amx`|
|IP 1|`10.202.42.71`|
|IP 2|`10.202.6.58`|
|Sistema operativo|Red Hat Enterprise Linux 8.10|
|Kernel|`4.18.0-553.125.1.el8_10.x86_64`|
|Podman|`4.9.4-rhel`|
|Usuario de acceso|`payment`|

### 2.2. Contenedores identificados

|Contenedor|Imagen|Estado|
|---|---|---|
|`claro-payment-api-test`|`docker-dev.roshka.com/claro/payment/claro-payment-api:2.0.10`|Running|
|`documenta`|`docker.roshka.com/claro/payment/posjota:1.3.0`|Running|
|`claro-payment-proxy-test`|`nginx:1.27-alpine`|Running|

**Hallazgo:** se confirmó la existencia del servicio Socket `documenta` en Testing, a pesar de que inicialmente se había informado que `claro-posjota` no estaba desplegado en este ambiente.

No se ha confirmado el despliegue de las demás procesadoras Socket.

## 3. Montajes y almacenamiento de logs

Los servicios principales utilizan bind mounts de Podman, por lo que los archivos de logs son accesibles desde el filesystem del host.

### 3.1. API REST

**Contenedor:** `claro-payment-api-test`

|Origen en el host|Destino en el contenedor|
|---|---|
|`/app/payment-test/claro-payment-api/config`|`/opt/payment/config`|
|`/app/payment-test/claro-payment-api/logs`|`/opt/payment/logs`|
|`/etc/localtime`|`/etc/localtime`|

### 3.2. Servicio Socket — documenta

**Contenedor:** `documenta`

|Origen en el host|Destino en el contenedor|
|---|---|
|`/app/payment-test/documenta/logs`|`/opt/payment/logs`|
|`/app/payment-test/documenta/collector.properties`|`/opt/payment/collector.properties`|
|`/etc/localtime`|`/etc/localtime`|

### 3.3. Proxy Nginx

**Contenedor:** `claro-payment-proxy-test`

Se identificaron montajes de configuración y certificados:

- `/app/payment-test/nginx/nginx.conf`
    
- `/app/payment-test/nginx/certs`
    

No se identificó un bind mount específico para logs del proxy.

## 4. Relevamiento de logs — API REST

### 4.1. Ubicación

**Ruta en el host:**

`/app/payment-test/claro-payment-api/logs`

**Archivo activo:**

`/app/payment-test/claro-payment-api/logs/payment.log`

**Históricos:**

`/app/payment-test/claro-payment-api/logs/logs/YYYY-MM/payment-*.log.gz`

### 4.2. Características

|Propiedad|Resultado|
|---|---|
|Formato|Texto plano|
|Archivo activo|`payment.log`|
|Última modificación observada|05/11/2026 14:07:39 (-03:00)|
|Tamaño observado|307.243 bytes|
|Históricos|Archivos `.log.gz` organizados por año y mes|
|Tamaño total del directorio|Aproximadamente 13 MB|
|Permisos del directorio de logs|`777`|
|Propietario del directorio|`payment:users`|

### 4.3. Estructura de los eventos

Los registros presentan elementos como:

- Timestamp.
    
- Nivel de severidad.
    
- Hilo de ejecución.
    
- Clase o componente de aplicación.
    
- Mensaje.
    
- Identificadores de sesión y datos asociados a operaciones.
    

**Ejemplo de estructura:**

`[INFO] 2026-11-05 09:04:27 [http-nio-8081-exec-9] INFO Clase:línea - Mensaje`

Se identificaron clases como `ClaroPacksClient`, `ConsultaPackService` y `PackOfferService`.

**Observaciones:**

- Los logs no utilizan JSON como formato principal.
    
- Se observan registros relacionados con consultas y ofertas de packs.
    
- Algunos mensajes contienen números de servicio y session IDs.
    
- Debe verificarse si existen eventos multilínea.
    
- La presencia de archivos históricos comprimidos evidencia un mecanismo de archivado, pero falta verificar la configuración exacta de rotación.
    

También se identificaron archivos con nombres atípicos en el directorio de logs. Su origen y finalidad no fueron determinados.

## 5. Relevamiento de logs — Socket documenta

### 5.1. Ubicación

**Ruta en el host:**

`/app/payment-test/documenta/logs`

### 5.2. Archivos identificados

|Archivo|Tamaño|Última modificación observada|
|---|---|---|
|`routlog.txt`|208.061 bytes|03/11/2026 16:07|
|`outlog.txt`|526 bytes|03/11/2026 16:07|
|`test.log`|7 bytes|28/08/2026|

**Tamaño total:** aproximadamente 220 KB.

### 5.3. Características

Los archivos `routlog.txt` y `outlog.txt` contienen registros en texto plano.

Se identificaron:

- Timestamps con precisión de milisegundos.
    
- Niveles de severidad.
    
- Identificadores de hilo.
    
- Nombres de clases Java.
    
- Mensajes relacionados con el inicio del servicio, configuración y conexiones.
    

Entre las clases observadas se encuentran:

- `PronetStreamCollectorServer`.
    
- `CollectorSession`.
    

El archivo `routlog.txt` fue identificado como texto UTF-8 con líneas largas.

### 5.4. Permisos

Los archivos `.txt` presentan permisos `644`.

El propietario y grupo aparecen como `UNKNOWN:UNKNOWN` en el host, por lo que se requiere revisar sus UID/GID numéricos si fuera necesario.

El directorio de logs tiene permisos `777`.

### 5.5. Observaciones

- No se identificaron archivos históricos comprimidos.
    
- No se ha confirmado un mecanismo de rotación.
    
- La última escritura observada fue el 03/11/2026.
    
- Debe verificarse si los archivos continúan recibiendo eventos.
    
- No se ha confirmado si ambos archivos deben ser ingeridos.
    

## 6. Hallazgos técnicos

### H01 — Logs disponibles mediante bind mounts

Los servicios prioritarios utilizan directorios del host montados en los contenedores.

Esto permite considerar una arquitectura en la que Filebeat lea directamente los archivos del host, sin necesidad de instalar agentes dentro de los contenedores.

### H02 — Diferencias entre ambientes

Las rutas proporcionadas inicialmente correspondían a una estructura `/app/payment/`.

En Testing se confirmó el uso de `/app/payment-test/`.

Por tanto, las configuraciones deben utilizar rutas específicas del ambiente.

### H03 — Servicio Socket disponible en Testing

Se identificó el contenedor `documenta`, correspondiente a una instancia de `claro-posjota`.

Debe actualizarse el inventario inicial, ya que existe al menos un servicio Socket desplegado.

### H04 — Formatos diferentes entre servicios

La API REST y el servicio Socket generan logs de texto plano, pero presentan diferencias en timestamps, nombres de archivos y estructura de mensajes.

La ingesta y el procesamiento deberán contemplar estas diferencias.

### H05 — Permisos amplios en directorios

Los directorios de logs presentan permisos `777`.

Se registra como hallazgo de seguridad para evaluación posterior. No se realizaron modificaciones durante el relevamiento.

### H06 — Información potencialmente sensible

Los logs de la API REST pueden contener identificadores de sesión y números asociados a servicios o transacciones.

Se requiere evaluar la política de protección de datos antes de centralizarlos.

## 7. Información pendiente

|ID|Pendiente|Importancia|
|---|---|---|
|P01|Confirmar mecanismo y configuración de rotación de `payment.log`|Alta|
|P02|Determinar si `documenta` utiliza rotación|Alta|
|P03|Comprobar si existen eventos multilínea|Alta|
|P04|Confirmar qué archivos Socket deben centralizarse|Alta|
|P05|Verificar actividad de escritura actual en `documenta`|Media|
|P06|Determinar volumen diario aproximado de logs|Media|
|P07|Confirmar si existen otros servidores PAYMENT en Testing|Media|
|P08|Evaluar archivos con nombres atípicos en la API REST|Baja|
|P09|Verificar permisos efectivos del usuario de Filebeat|Alta, durante el bloque 2|
|P10|Confirmar el tratamiento requerido para información sensible|Alta, durante el diseño|

## 8. Estado de las tareas

|Tarea|Estado|
|---|---|
|1. Relevar servidores y contenedores PAYMENT en Testing|Completada para el servidor identificado|
|2. Relevar rutas, formatos, rotación y permisos de logs|Parcialmente completada|

No se modificaron aplicaciones, contenedores ni configuraciones durante este bloque.

## 9. Próximos pasos

1. Completar las verificaciones pendientes de rotación y registros multilínea.
    
2. Continuar con el Bloque 2: relevamiento de Filebeat y Elasticsearch/Kibana.
    
3. Confirmar qué configuraciones existentes pueden reutilizarse.
    
4. Utilizar los hallazgos para definir posteriormente el diseño de ingesta.
    

**Conclusión:** se confirmó que la infraestructura de Testing dispone de una API REST y una instancia Socket con logs persistidos en directorios del host mediante bind mounts. Esto proporciona una base adecuada para estudiar una ingesta mediante Filebeat, pendiente de validar su instalación, permisos y configuración.