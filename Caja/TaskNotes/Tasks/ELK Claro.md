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
**Fecha de inicio:** 05/11/2026  
**Última actualización:** 06/11/2026  
**Ambiente:** Testing  
**Bloque:** 1 — Relevamiento de infraestructura y logs  
**Estado:** Relevamiento técnico local completado, con pendientes identificados para las siguientes etapas.

---

## 1. Objetivo

Relevar la infraestructura de PAYMENT en Testing, identificar los servicios desplegados mediante Podman y determinar dónde se generan y almacenan sus logs.

El relevamiento permitirá diseñar posteriormente la captura e ingesta de logs mediante Filebeat hacia el clúster Elasticsearch `es02-testing`.

### 1.1. Alcance

- **API REST:** `claro-payment-api`.
    
- **Servicios Socket:** `claro-posjota`, contemplando todas las procesadoras que formen parte del alcance de PAYMENT. Actualmente se identificó la instancia `documenta` en el servidor relevado.
    
- **Proxy Nginx:** componente identificado, pero fuera de las fuentes prioritarias de ingesta.
    

El relevamiento actual se limita al servidor `paymendb01ptl.claro.amx`. Otros servidores quedan fuera de esta etapa.

### 1.2. Consideraciones

La solución deberá contemplar:

- Captura de logs de los servicios prioritarios.
    
- Identificación de la aplicación y procesadora de origen.
    
- Tratamiento adecuado de eventos multilínea.
    
- Compatibilidad con los mecanismos de rotación existentes.
    
- Protección de información sensible.
    
- Centralización y posterior consulta desde Kibana.
    

---

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

|Contenedor|Imagen|Estado observado|
|---|---|---|
|`claro-payment-api-test`|`docker-dev.roshka.com/claro/payment/claro-payment-api:2.0.10`|Running|
|`documenta`|`docker.roshka.com/claro/payment/posjota:1.3.0`|Running|
|`claro-payment-proxy-test`|`nginx:1.27-alpine`|Running|

**Hallazgo:** se confirmó la existencia del servicio Socket `documenta` en Testing, aunque inicialmente se había informado que `claro-posjota` no estaba desplegado en este ambiente.

No se identificaron otras procesadoras en este servidor durante el relevamiento.

### 2.3. Repositorios relacionados

|Componente|Repositorio|
|---|---|
|API REST|[https://github.com/roshkadev/claro-payment-api](https://github.com/roshkadev/claro-payment-api)|
|Socket|[https://github.com/roshkadev/claro-posjota](https://github.com/roshkadev/claro-posjota)|
|Servicio de consulta de CI|[https://github.com/roshkadev/claro-paymentgw](https://github.com/roshkadev/claro-paymentgw)|

El servicio de consulta de CI no forma parte del alcance prioritario.

---

## 3. Montajes y almacenamiento de logs

Los servicios prioritarios utilizan bind mounts de Podman, mediante los cuales sus archivos de logs quedan disponibles en el filesystem del host.

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

Montajes identificados:

- `/app/payment-test/nginx/nginx.conf`
    
- `/app/payment-test/nginx/certs`
    

No se identificó un bind mount específico para los logs del proxy.

### 3.4. Implicación para Filebeat

Los archivos de las aplicaciones están disponibles en directorios del host.

Esto permite plantear una arquitectura donde Filebeat lea directamente los archivos desde el servidor, sin instalar agentes dentro de los contenedores.

La viabilidad definitiva dependerá de la instalación, identidad de ejecución y permisos efectivos de Filebeat.

---

## 4. Relevamiento de logs — API REST

### 4.1. Ubicación

**Directorio del host:**

`/app/payment-test/claro-payment-api/logs`

**Archivo activo:**

`/app/payment-test/claro-payment-api/logs/payment.log`

**Históricos:**

`/app/payment-test/claro-payment-api/logs/logs/YYYY-MM/payment-*.log.gz`

### 4.2. Características

|Propiedad|Resultado|
|---|---|
|Formato|Texto plano|
|Framework de logging|Log4j2|
|Archivo activo|`payment.log`|
|Última modificación observada del archivo activo|05/11/2026 14:07:39 (-03:00)|
|Tamaño observado del archivo activo|307.243 bytes|
|Históricos|Archivos `.log.gz`|
|Organización histórica|Directorios por año y mes|
|Tamaño total observado|Aproximadamente 13 MB|
|Permisos del directorio de logs|`777`|
|Propietario del directorio|`payment:users`|

### 4.3. Estructura de los eventos

Los registros contienen:

- Timestamp.
    
- Nivel de severidad.
    
- Hilo de ejecución.
    
- Clase o componente de aplicación.
    
- Mensaje.
    
- Identificadores de sesión y datos asociados a operaciones.
    

**Ejemplo de estructura, con datos ficticios:**

`[INFO] 2026-11-05 09:04:27 [http-nio-8081-exec-9] INFO Clase:61 - Mensaje`

Se identificaron clases como:

- `ClaroPacksClient`
    
- `ConsultaPackService`
    
- `PackOfferService`
    

Los logs no utilizan JSON como formato principal.

Se observaron mensajes relacionados con consultas y ofertas de packs, así como números de servicio e identificadores de sesión.

### 4.4. Configuración de logging

**Archivo identificado:**

`/app/payment-test/claro-payment-api/config/log4j2-spring.xml`

Se confirmó la configuración de un appender `RollingFile`, cuyo archivo activo es `payment.log`.

El archivo está configurado mediante una ruta relativa:

`logs/payment.log`

La ubicación efectiva se corroboró con los archivos presentes en el bind mount.

### 4.5. Rotación de logs

La configuración contiene:

```
<Policies>
    <TimeBasedTriggeringPolicy />
    <SizeBasedTriggeringPolicy size="250 MB"/>
</Policies>
```

**Política identificada:**

|Propiedad|Configuración|
|---|---|
|Mecanismo|Log4j2 RollingFile|
|Rotación temporal|Diaria, según el patrón de fecha|
|Rotación por tamaño|250 MB|
|Formato de históricos|`.log.gz`|
|Organización histórica|`logs/YYYY-MM/`|
|Índice de archivos|`%i`|
|Eliminación por antigüedad|No definida explícitamente en el XML|
|Límite total de almacenamiento|No definido explícitamente en el XML|

El patrón de archivado identificado es:

`${log-path}/logs/$${date:yyyy-MM}/payment-%d{MM-dd-yyyy}-%i.log.gz`

**Observación:** el directorio mensual utiliza una expresión de fecha evaluada de forma diferente a la fecha incorporada en el nombre del archivo. Esto es coherente con los históricos encontrados en algunos directorios cuyo mes no coincide con la fecha del nombre.

La retención efectiva sigue pendiente de confirmar porque podrían existir mecanismos de limpieza externos.

### 4.6. Eventos multilínea

Se confirmó la existencia de stack traces Java en `payment.log`.

Se observaron referencias a:

- Spring Framework.
    
- `DefaultRestClient`.
    
- `ClaroPacksClient`.
    
- `ConsultaPackService`.
    
- `PacksController`.
    

**Conclusión:** la futura ingesta deberá contemplar eventos multilínea para evitar que una excepción quede dividida en múltiples documentos independientes.

La expresión regular definitiva se definirá durante el diseño de Filebeat.

### 4.7. Volumen histórico de logs

Se analizaron archivos históricos comprimidos correspondientes al período del 31/10/2026 al 05/11/2026.

|Fecha correspondiente al log|Tamaño comprimido|Tamaño original|
|---|---|---|
|31/10/2026|144 B|445 B|
|01/11/2026|1.683 B|33.095 B|
|02/11/2026|34.181 B|516.654 B|
|03/11/2026|16.457 B|474.880 B|
|04/11/2026|1.737 B|13.908 B|
|05/11/2026|19.493 B|307.466 B|
|**Total**|**73.695 B**|**1.346.448 B**|

**Indicadores obtenidos:**

- Volumen original total: aproximadamente 1,35 MB.
    
- Promedio diario de la muestra: aproximadamente 224 KB.
    
- Compresión observada: aproximadamente 94,5 %.
    
- Mayor volumen diario de la muestra: aproximadamente 517 KB.
    

**Limitaciones:**

- El cálculo utiliza únicamente los archivos históricos identificados.
    
- No incluye el archivo activo ni posibles archivos adicionales.
    
- No representa necesariamente días de mayor carga.
    
- No debe utilizarse directamente como estimación de Producción.
    
- El almacenamiento requerido en Elasticsearch será diferente al tamaño original de los archivos.
    

### 4.8. Archivos atípicos

Se identificaron archivos adicionales con nombres que parecen fragmentos de contenido, entre ellos:

- `informacion",`
    
- `ion": 0,`
    
- `accion": "10",`
    

Su origen y finalidad no se determinaron.

**Recomendación:** no incluirlos automáticamente en la ingesta ni eliminarlos sin verificar su procedencia.

---

## 5. Relevamiento de logs — Socket documenta

### 5.1. Ubicación

**Directorio del host:**

`/app/payment-test/documenta/logs`

### 5.2. Archivos identificados

|Archivo|Tamaño|Última modificación observada|
|---|---|---|
|`routlog.txt`|208.061 bytes|03/11/2026 16:07|
|`outlog.txt`|526 bytes|03/11/2026 16:07|
|`test.log`|7 bytes|28/08/2026|

**Tamaño total observado:** aproximadamente 220 KB.

### 5.3. Características

Los archivos `routlog.txt` y `outlog.txt` contienen registros de texto plano.

Se identificaron:

- Timestamps con precisión de milisegundos.
    
- Niveles de severidad.
    
- Identificadores de hilo.
    
- Clases Java.
    
- Mensajes de inicialización, configuración y conexiones.
    

Entre las clases identificadas se encuentran:

- `PronetStreamCollectorServer`
    
- `CollectorSession`
    

El archivo `routlog.txt` fue identificado como texto UTF-8 con líneas largas.

### 5.4. Configuración de logging

**Archivo identificado dentro del contenedor:**

`/opt/payment/plainlog4jconfig.xml`

También se identificaron:

- `/opt/payment/lib/log4j.jar`
    
- `/opt/payment/lib/commons-logging.jar`
    

La configuración utiliza clases correspondientes a Log4j 1.x, aunque la versión exacta de la biblioteca no fue verificada.

### 5.5. Configuración de `routlog.txt`

Se identificó un appender de tipo `RollingFileAppender`.

Configuración relevante:

```
<param name="File" value="logs/routlog.txt"/>
<param name="Append" value="true"/>
<param name="MaxFileSize" value="15000KB"/>
<param name="MaxBackupIndex" value="15"/>
```

|Propiedad|Configuración|
|---|---|
|Appender|`RollingFileAppender`|
|Archivo|`routlog.txt`|
|Añadir al archivo existente|Sí|
|Rotación|Por tamaño|
|Tamaño máximo configurado|15.000 KB|
|Cantidad máxima de respaldos|15|
|Compresión|No configurada|
|Retención temporal|No configurada|

Los respaldos se generarían siguiendo la política de rotación de Log4j.

No se observaron archivos históricos en el inventario del directorio consultado.

### 5.6. Configuración de `outlog.txt`

Se identificó un appender de tipo `FileAppender`.

Configuración relevante:

```
<param name="File" value="logs/outlog.txt"/>
<param name="Append" value="false"/>
```

**Comportamiento:**

- No tiene rotación configurada.
    
- `Append=false` indica que se sobrescribe el contenido cuando se inicializa el appender.
    
- Los registros anteriores pueden perderse cuando la aplicación reinicializa ese mecanismo de logging.
    

Este comportamiento deberá contemplarse durante el diseño de ingesta.

### 5.7. Eventos multilínea

Se confirmó la existencia de stack traces Java en `routlog.txt`.

Se identificaron referencias a:

- `PronetStreamInputPackage`
    
- `CollectorSession`
    
- `NumberFormatException`
    

También se observaron estructuras como:

- `Caused by:`
    
- `at ...`
    
- `... 2 more`
    

**Conclusión:** los logs Socket requieren tratamiento multilínea.

Su estructura difiere de la API REST, por lo que deberá evaluarse una configuración específica para este servicio.

### 5.8. Permisos y propietarios

|Archivo|Permisos|UID|GID|
|---|---|---|---|
|`routlog.txt`|`644`|`690823`|`690823`|
|`outlog.txt`|`644`|`690823`|`690823`|

Los UID/GID no tienen una correspondencia de nombre visible en el host.

Esto podría estar relacionado con el mapeo de usuarios de Podman, aunque no fue verificado.

El directorio de logs tiene permisos `777`.

### 5.9. Actividad y volumen

La última modificación observada de ambos archivos `.txt` corresponde al 03/11/2026.

No contamos con mediciones suficientes para determinar la tasa diaria de generación de logs Socket.

La existencia del contenedor en estado Running no garantiza que esté recibiendo operaciones o escribiendo nuevos eventos continuamente.

**Pendientes específicos:**

- Confirmar qué archivos deben centralizarse.
    
- Verificar si el servicio continúa generando eventos.
    
- Determinar su volumen diario en condiciones representativas.
    
- Verificar si el XML encontrado es efectivamente la configuración cargada por el proceso.
    

---

## 6. Hallazgos técnicos consolidados

### H01 — Logs disponibles mediante bind mounts

Los servicios prioritarios utilizan directorios del host montados en contenedores.

Esto permite considerar la captura desde Filebeat instalado en el servidor, sin necesidad de ejecutar un agente por contenedor.

### H02 — Diferencias entre ambientes

Las rutas inicialmente proporcionadas correspondían a `/app/payment/`.

En Testing se confirmó el uso de `/app/payment-test/`.

Las configuraciones deberán contemplar las rutas específicas del ambiente.

### H03 — Servicio Socket disponible en Testing

Se identificó `documenta`, una instancia del servicio `claro-posjota`.

La solución deberá contemplar las demás procesadoras que se incorporen al alcance y estén disponibles en los servidores correspondientes.

### H04 — Diferencias en logging

La API REST utiliza Log4j2 y el servicio Socket presenta configuración de Log4j 1.x.

Sus formatos, mecanismos de rotación y archivos son diferentes.

### H05 — Eventos multilínea confirmados

Ambos servicios generan stack traces Java.

El diseño de Filebeat deberá incorporar un tratamiento adecuado para conservar la integridad de estos eventos.

### H06 — Permisos amplios

Los directorios de logs presentan permisos `777`.

Se registra como hallazgo de seguridad.

No se realizaron modificaciones durante el relevamiento.

### H07 — Información sensible en logs

Se observaron identificadores de sesión, números de servicio y datos relacionados con operaciones.

La centralización deberá incorporar controles para proteger información sensible.

Se deberá evaluar:

- Minimización de datos.
    
- Enmascaramiento o eliminación de campos sensibles cuando corresponda.
    
- Restricción de permisos de consulta.
    
- Protección de la comunicación hacia Elasticsearch.
    
- Retención adecuada.
    
- Evitar registrar credenciales, tokens y otros secretos.
    
- Revisión de los mensajes antes de incorporarlos al almacenamiento centralizado.
    

### H08 — Rotación diferente entre servicios

La API REST utiliza rotación diaria o por tamaño, con compresión GZIP.

`documenta` utiliza un archivo con rotación por tamaño y otro sin rotación que puede sobrescribirse.

Filebeat deberá considerar estos comportamientos para evitar pérdidas o duplicaciones.

### H09 — Volumen inicial bajo en la API REST

Se observó un promedio histórico aproximado de 224 KB diarios durante seis días.

Esta cifra es orientativa y no representa la carga esperada en Producción.

### H10 — Retención no completamente determinada

No se identificó una política explícita de eliminación por antigüedad en los archivos de configuración revisados.

Esto no descarta mecanismos externos de limpieza.

La retención en Elasticsearch deberá definirse de forma independiente durante el diseño de la solución.

---

## 7. Información pendiente y tratamiento

|ID|Pendiente|Tratamiento|
|---|---|---|
|P01|Verificar si existen mecanismos externos de limpieza de históricos|Revisión complementaria|
|P02|Confirmar archivos Socket que deben centralizarse|Validación funcional|
|P03|Verificar actividad reciente de `documenta`|Validación con responsables|
|P04|Estimar volumen diario representativo de Socket|Medición posterior|
|P05|Evaluar archivos atípicos de la API REST|Revisión complementaria|
|P06|Verificar permisos efectivos de Filebeat|Bloque 2|
|P07|Definir tratamiento de información sensible|Bloque 3|
|P08|Confirmar retención requerida en Elasticsearch|Bloque 3|
|P09|Validar configuración multilínea definitiva|Bloques 3 y 4|
|P10|Confirmar procesadoras incluidas y disponibles|Incorporación progresiva|
|P11|Identificar mecanismo de administración de contenedores|Revisión operativa complementaria|

Los otros servidores PAYMENT no forman parte del relevamiento actual.

No se considera necesario resolver todos estos pendientes para iniciar el Bloque 2. Los que afectan al diseño deberán resolverse antes de habilitar la ingesta.

---

## 8. Estado de las tareas

|Tarea|Estado|
|---|---|
|1. Relevar servidores y contenedores PAYMENT en Testing|Relevamiento del servidor objetivo realizado|
|2. Relevar rutas, formatos, rotación y permisos de logs|Relevamiento técnico principal completado|

### Validaciones realizadas

- Identificación del sistema operativo y versión de Podman.
    
- Inventario de contenedores en ejecución.
    
- Inspección de bind mounts.
    
- Identificación de rutas reales de logs.
    
- Inventario de archivos activos e históricos.
    
- Revisión de tamaños y permisos.
    
- Análisis general de formatos.
    
- Identificación de stack traces multilínea.
    
- Revisión de configuraciones Log4j2 y Log4j.
    
- Identificación de mecanismos de rotación.
    
- Estimación del volumen histórico de la API REST.
    

**No se modificaron aplicaciones, contenedores ni archivos de configuración durante el relevamiento.**

---

## 9. Próximos pasos

El siguiente bloque corresponde al **Bloque 2 — Relevamiento de Filebeat y ELK**.

### Tarea 3 — Filebeat

- Verificar instalación y versión.
    
- Identificar método de ejecución.
    
- Revisar configuración existente.
    
- Determinar inputs y outputs actuales.
    
- Revisar permisos de lectura.
    
- Analizar processors, colas y reintentos.
    
- Determinar compatibilidad con la infraestructura existente.
    

### Tarea 4 — Elasticsearch y Kibana

- Verificar estado y recursos del clúster `es02-testing`.
    
- Identificar índices, aliases y Data Streams.
    
- Revisar templates y mappings.
    
- Identificar pipelines existentes.
    
- Revisar ILM y retención.
    
- Verificar permisos y recursos de Kibana.
    

Los resultados servirán de base para el posterior diseño de ingesta.

---

## 10. Conclusión

El relevamiento confirmó que `claro-payment-api` y la instancia Socket `documenta` generan logs persistidos en directorios del host mediante bind mounts de Podman.

Se identificaron los archivos activos, sus formatos generales, configuraciones de logging, mecanismos de rotación, permisos y características de los registros.

También se confirmó que ambos servicios generan eventos multilínea y que los logs pueden contener información sensible.

**El Bloque 1 queda cerrado para el alcance técnico local definido**, conservando los pendientes documentados para su resolución durante el diseño y la implementación.

La siguiente etapa será verificar el estado actual de Filebeat y de la infraestructura Elasticsearch/Kibana antes de realizar cambios.