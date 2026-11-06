---
status: open
priority: normal
dateCreated: 2026-10-28T07:51:17.807-03:00
dateModified: 2026-10-28T07:51:42.890-03:00
tags:
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


# Informe técnico — Relevamiento de Filebeat y ELK en Testing

**Proyecto:** Claro PAYMENT — Centralización de logs en ELK  
**Fecha de inicio:** 06/11/2026  
**Última actualización:** 06/11/2026  
**Ambiente:** Testing  
**Bloque:** 2 — Relevamiento de Filebeat y ELK  
**Estado:** Relevamiento técnico completado, con restricciones de acceso y decisiones pendientes para las siguientes etapas.

---

## 1. Objetivo

Relevar el estado actual de Filebeat en el servidor PAYMENT de Testing y analizar la infraestructura Elasticsearch/Kibana existente, identificando configuraciones, recursos, permisos y restricciones que deben considerarse antes de implementar la centralización de logs.

El relevamiento permitirá diseñar posteriormente la captura e ingesta de logs de los servicios PAYMENT hacia el clúster Elasticsearch `es02-testing`.

### 1.1. Alcance

El bloque comprende:

- **Tarea 3:** Relevar instalación y configuración actual de Filebeat.
    
- **Tarea 4:** Relevar Elasticsearch y Kibana: índices, templates, pipelines y permisos.
    

**Servidor PAYMENT relevado:**

`paymendb01ptl.claro.amx`

**Clúster Elasticsearch:**

`es02-testing`

**Servicios prioritarios identificados en el Bloque 1:**

- API REST: `claro-payment-api-test`.
    
- Servicio Socket: `documenta`, instancia de `claro-posjota`.
    

El relevamiento de Filebeat se limita al servidor PAYMENT identificado. No se realizaron inspecciones de instalaciones en otros servidores.

### 1.2. Consideraciones

La futura implementación deberá contemplar:

- Captura de archivos de logs existentes mediante Filebeat.
    
- Conectividad y autenticación hacia Elasticsearch.
    
- Protección de información sensible.
    
- Compatibilidad entre versiones.
    
- Estrategia de almacenamiento e indexación.
    
- Políticas de retención.
    
- Configuración de recursos de procesamiento.
    
- Visualización desde Kibana.
    
- Separación e identificación de aplicaciones y procesadoras.
    

---

## 2. Relevamiento de Filebeat

### 2.1. Servidor inspeccionado

|Propiedad|Valor|
|---|---|
|Hostname|`paymendb01ptl.claro.amx`|
|Sistema operativo|Red Hat Enterprise Linux 8.10|
|Usuario de acceso|`payment`|
|Plataforma de contenedores|Podman 4.9.4-rhel|
|Instalación de Filebeat|No identificada|

### 2.2. Verificación de instalación

Se realizaron las siguientes comprobaciones:

**Identificación del ejecutable:**

```
command -v filebeat
```

Resultado: no se encontró el ejecutable en el `PATH` del usuario.

**Verificación mediante RPM:**

```
rpm -q filebeat
```

Resultado:

```
package filebeat is not installed
```

**Verificación de servicios systemd:**

```
systemctl is-active filebeat
systemctl is-enabled filebeat
```

Resultados:

```
inactive
Failed to get unit file state for filebeat.service:
No such file or directory
```

Se consultaron adicionalmente las propiedades del servicio:

```
LoadState=not-found
ActiveState=inactive
SubState=dead
MainPID=0
```

No se identificó una unidad `filebeat.service` registrada.

### 2.3. Directorios y ejecutables

Se verificaron las ubicaciones habituales:

```
/etc/filebeat
/var/lib/filebeat
/var/log/filebeat
/opt/filebeat
/usr/share/filebeat
```

No se encontraron los directorios indicados.

También se verificaron rutas habituales de ejecutables:

```
/usr/bin/filebeat
/usr/local/bin/filebeat
/opt/filebeat/filebeat
/usr/share/filebeat/bin/filebeat
```

No se encontraron ejecutables en esas ubicaciones.

### 2.4. Procesos y contenedores

Se realizó una búsqueda de procesos relacionados con Filebeat y Elastic Agent.

No se identificaron procesos coincidentes en la consulta realizada.

Adicionalmente, se consultaron los contenedores Podman visibles para el usuario `payment`.

|Contenedor|Imagen|Estado observado|
|---|---|---|
|`claro-payment-api-test`|`docker-dev.roshka.com/claro/payment/claro-payment-api:2.0.10`|Running|
|`documenta`|`docker.roshka.com/claro/payment/posjota:1.3.0`|Running|
|`claro-payment-proxy-test`|`nginx:1.27-alpine`|Running|

No se encontró ningún contenedor de Filebeat o Elastic Agent entre los contenedores identificados.

**Limitación:** la inspección de Podman se realizó con el usuario `payment`. No descarta contenedores administrados por otros usuarios o por root.

### 2.5. Restricciones de permisos

Se intentó consultar los directorios habituales de Filebeat utilizando `sudo`.

El sistema devolvió una denegación de autorización para el usuario `payment`.

Esto confirma que el usuario no dispone de permisos sudo generales para realizar las operaciones consultadas.

No se modificaron permisos ni se solicitaron elevaciones de privilegios durante el relevamiento.

### 2.6. Conclusión del relevamiento de Filebeat

**No se identificó una instalación o ejecución convencional de Filebeat en** `**paymendb01ptl.claro.amx**`**.**

Las verificaciones abarcaron:

- Instalación RPM.
    
- Ejecutable disponible en el `PATH`.
    
- Servicio systemd.
    
- Directorios habituales.
    
- Ejecutables en rutas conocidas.
    
- Procesos activos.
    
- Contenedores Podman visibles para `payment`.
    

No se puede descartar completamente una instalación no convencional administrada por otro usuario.

De acuerdo con el Bloque 1, los logs de los servicios PAYMENT están disponibles en el filesystem del host mediante bind mounts de Podman.

Por tanto, se considera viable evaluar una instalación de Filebeat directamente sobre el servidor, sujeta a la validación de permisos, compatibilidad y diseño técnico.

**La instalación y configuración del agente quedan pendientes para el Bloque 4.**

---

## 3. Infraestructura Elasticsearch identificada

### 3.1. Identificación del clúster

Se consultó la API de Elasticsearch desde Kibana Dev Tools:

```
GET /
```

**Resultado:**

|Propiedad|Valor|
|---|---|
|Cluster name|`es02-testing`|
|Cluster UUID|`y_c0ETLUSlSkh1SJc8_TIg`|
|Elasticsearch|`7.17.2`|
|Build type|RPM|
|Build flavor|Default|
|Lucene|`8.11.1`|
|Nodo que respondió|`elk02atl.claro.amx`|

Se confirmó que el ambiente utiliza Elasticsearch 7.17.2.

### 3.2. Estado general del clúster

Se ejecutó:

```
GET /_cluster/health
```

Resultados:

|Indicador|Valor|
|---|---|
|Estado|`green`|
|Nodos totales|5|
|Nodos de datos|3|
|Shards primarios activos|311|
|Shards activos totales|327|
|Shards en reubicación|0|
|Shards en inicialización|0|
|Shards sin asignar|0|
|Tareas pendientes|0|
|Porcentaje de shards activos|100%|

**Conclusión:**

El clúster se encontraba operativo, con todos los shards asignados y sin tareas pendientes al momento de la consulta.

El estado `green` no garantiza por sí solo la ausencia de problemas de rendimiento ni determina la capacidad disponible para futuras ingestas.

### 3.3. Inventario de nodos

Se consultó:

```
GET /_cat/nodes?v&h=name,ip,roles,master,heap.percent,ram.percent,cpu,disk.used_percent,disk.avail
```

|Nodo|IP|Heap JVM|RAM|CPU|Disco utilizado|
|---|---|---|---|---|---|
|`elk03adl.claro.amx`|`10.202.50.102`|38%|95%|1%|2,24%|
|`elk02atl.claro.amx`|`10.202.48.105`|20%|75%|0%|0,70%|
|`elk03atl.claro.amx`|`10.202.48.106`|11%|65%|1%|0,70%|
|`elk01atl.claro.amx`|`10.202.48.104`|58%|91%|0%|2,22%|
|`elk02adl.claro.amx`|`10.202.50.101`|72%|96%|9%|14,18%|

El nodo `elk02atl.claro.amx` aparecía como master electo.

**Observaciones:**

- Existen nodos con porcentajes elevados de utilización de RAM física.
    
- El heap JVM más elevado en la muestra corresponde a `elk02adl`, con 72%.
    
- La utilización de CPU observada fue baja en la mayoría de los nodos.
    
- Una única muestra no permite determinar presión sostenida de memoria.
    

No se identificó evidencia suficiente para recomendar modificaciones de recursos o parámetros JVM.

### 3.4. Distribución de almacenamiento

Se ejecutó:

```
GET /_cat/allocation?v
```

|Nodo|Shards|Almacenamiento de índices|Disco disponible|
|---|---|---|---|
|`elk02adl.claro.amx`|109|203 GB|1,2 TB|
|`elk01atl.claro.amx`|109|22,4 GB|1,4 TB|
|`elk03adl.claro.amx`|109|22,5 GB|1,4 TB|

Los tres nodos de datos presentan 109 shards cada uno.

Se observó una diferencia importante entre el volumen almacenado en `elk02adl` y los otros dos nodos.

Esta distribución no demuestra por sí sola un problema, debido a que los shards pueden tener tamaños distintos.

**Conclusión:** existe capacidad de almacenamiento aparente en los nodos consultados, pero será necesario definir el volumen esperado y la retención de PAYMENT antes de dimensionar la nueva ingesta.

---

## 4. Índices, aliases y Data Streams

### 4.1. Índices PAYMENT

Se realizaron consultas utilizando distintos patrones:

```
GET /_cat/indices/*payment*?v&expand_wildcards=all

GET /_cat/indices/es02-*?v

GET /_cat/indices?v&expand_wildcards=all
```

Las respuestas devolvieron únicamente las cabeceras de las tablas.

No se identificaron índices visibles correspondientes a PAYMENT.

**Consideración:** las consultas se realizaron con el usuario `payment`, cuyos permisos están limitados a un patrón de índices. Por tanto, no permiten obtener un inventario completo de los índices de Elasticsearch.

El patrón destinado al proyecto es:

```
es02-payment_py-transactions-*
```

No se identificaron índices coincidentes en las consultas realizadas.

### 4.2. Aliases

Se consultó:

```
GET /_cat/aliases/*payment*?v
```

La respuesta no mostró aliases coincidentes.

No se identificaron aliases PAYMENT visibles para el usuario.

### 4.3. Data Streams

Se ejecutó:

```
GET /_data_stream
```

Resultado:

```
{
  "data_streams": []
}
```

No se identificaron Data Streams visibles en la consulta.

Esto no significa que Elasticsearch 7.17.2 carezca de soporte para Data Streams.

### 4.4. Conclusión sobre almacenamiento

No se encontró evidencia de índices, aliases o Data Streams destinados a PAYMENT dentro del alcance consultable.

Durante el diseño se deberá determinar si corresponde utilizar:

- Índices tradicionales.
    
- Índices asociados a aliases.
    
- Data Streams.
    

La decisión deberá considerar la nomenclatura acordada con Claro, los templates existentes, las políticas de retención y el funcionamiento de Filebeat.

---

## 5. Index Templates y Component Templates

### 5.1. Inventario

Se consultó el catálogo de templates existentes mediante:

```
GET /_cat/templates?v
```

Se identificaron templates generales del clúster y otros relacionados con Elastic Stack.

Entre los relevantes se encuentran:

- `limpia`
    
- `limpiaidx-template-15d`
    
- `claropay`
    

También se observaron definiciones para patrones como `logs-*-*`, `metrics-*-*` y distintos componentes de Elastic Agent.

### 5.2. Template `limpia`

Se consultó:

```
GET /_index_template/limpia
```

Configuración relevante:

```
index_patterns:
  - "es02*"
  - "idx_*"
  - "idx-*"

settings:
  index:
    lifecycle:
      name: limpiaidx
      rollover_alias: idx
    refresh_interval: 5s
    number_of_shards: 1
    number_of_replicas: 0
```

El template es composable y no tiene una prioridad explícita, por lo que utiliza la prioridad predeterminada de 0.

### 5.3. Template `limpiaidx-template-15d`

Se consultó:

```
GET /_index_template/limpiaidx-template-15d
```

Configuración relevante:

```
index_patterns:
  - "*idx_*"
  - "*idx-*"
  - "*es02-*"

priority: 20

settings:
  index:
    lifecycle:
      name: limpiaidx-15d
```

Este template establece una política ILM denominada `limpiaidx-15d`.

No se identificaron mappings ni aliases personalizados en su definición.

### 5.4. Template `claropay`

Se consultó:

```
GET /_template/claropay
```

Resultado:

```
{
  "claropay": {
    "order": 0,
    "index_patterns": [
      "es02-claropay*"
    ],
    "settings": {},
    "mappings": {},
    "aliases": {}
  }
}
```

Se trata de un template legacy.

Su patrón no coincide con:

```
es02-payment_py-transactions-*
```

### 5.5. Simulación de template para PAYMENT

Se ejecutó una simulación de solo lectura:

```
POST /_index_template/_simulate_index/es02-payment_py-transactions-2026.11.06
```

La configuración efectiva devuelta fue:

```
{
  "template": {
    "settings": {
      "index": {
        "lifecycle": {
          "name": "limpiaidx-15d"
        }
      }
    },
    "aliases": {}
  }
}
```

**Resultado:**

El índice simulado heredaría el template composable de mayor prioridad, `limpiaidx-template-15d`.

Por tanto:

- Se aplicaría la política ILM `limpiaidx-15d`.
    
- No heredaría automáticamente las configuraciones específicas del template `limpia`.
    
- No se definirían aliases mediante el template efectivo.
    
- No se identificaron mappings personalizados en la simulación.
    

La simulación no creó índices ni modificó recursos.

### 5.6. Implicaciones

La nomenclatura propuesta para PAYMENT ya coincide con templates generales del clúster.

Esto debe considerarse antes de crear índices o introducir un template específico.

**Pendiente:** confirmar el comportamiento real de la política `limpiaidx-15d` y su adecuación a los requisitos de retención de PAYMENT.

---

## 6. Políticas ILM

### 6.1. Políticas identificadas

Se identificaron referencias a:

- `limpiaidx`
    
- `limpiaidx-15d`
    

Estas referencias aparecen en los templates composables existentes.

### 6.2. Verificación de políticas

Se intentó consultar:

```
GET /_ilm/policy/limpiaidx-15d
```

El servidor respondió:

```
403 security_exception
```

El error indicó que la acción requiere alguno de los siguientes privilegios:

```
read_ilm
manage_ilm
manage
all
```

El usuario `payment` no tiene esos privilegios a nivel de clúster.

### 6.3. Información pendiente

No fue posible verificar:

- Si la política `limpiaidx-15d` realmente elimina índices a los 15 días.
    
- Si contempla rollover.
    
- Qué fases ILM tiene configuradas.
    
- Qué condiciones activan sus acciones.
    
- Si la política `limpiaidx` es apropiada para algún recurso PAYMENT.
    

**Conclusión:** el contenido y comportamiento de las políticas ILM permanecen sin verificar.

Para completar esta información se deberá solicitar el privilegio de solo lectura `read_ilm` o las definiciones correspondientes al administrador del clúster.

No se considera necesario solicitar `manage_ilm` durante el relevamiento.

---

## 7. Ingest Pipelines

### 7.1. Inventario general

Se consultó:

```
GET /_ingest/pipeline
```

La respuesta confirmó que el clúster contiene diversos pipelines de procesamiento.

Se identificó, entre otros:

```
PROV-FPLMN
```

Descripción:

```
Reingenieria FPLMN
```

Su configuración utiliza processors como:

- `rename`
    
- `gsub`
    
- `grok`
    
- `date`
    
- `convert`
    
- `remove`
    
- `set`
    
- `script`
    

También dispone de procesamiento de errores mediante `on_failure`.

Estos componentes demuestran que el clúster utiliza mecanismos de transformación de eventos mediante Elasticsearch Ingest Pipelines.

### 7.2. Pipelines relacionados con PAYMENT

Se ejecutaron las siguientes búsquedas:

```
GET /_ingest/pipeline/*payment*

GET /_ingest/pipeline/*claropay*

GET /_ingest/pipeline/*transactions*
```

Las tres consultas devolvieron:

```
{}
```

No se identificaron pipelines cuyos nombres coincidan con los patrones consultados.

Esto no descarta la existencia de pipelines genéricos o con otra nomenclatura que pudieran resultar reutilizables.

### 7.3. Conclusión

No se identificó un Ingest Pipeline específico para PAYMENT.

Durante el Bloque 3 se deberá evaluar si el formato de los logs, la extracción de campos y el tratamiento de información sensible justifican crear un pipeline específico.

La existencia de otros pipelines no implica que sean compatibles con los logs de PAYMENT.

---

## 8. Seguridad y permisos de Elasticsearch

### 8.1. Usuario autenticado

Se consultó:

```
GET /_security/_authenticate
```

|Propiedad|Valor|
|---|---|
|Usuario|`payment`|
|Rol|`paymentADM`|
|Realm|`default_native`|
|Tipo de autenticación|Native Realm|
|Estado|Habilitado|

### 8.2. Privilegios efectivos

Se consultó:

```
GET /_security/user/_privileges
```

**Privilegios de clúster:**

```
manage_index_templates
manage_ingest_pipelines
manage_pipeline
monitor
```

**Privilegios de índices:**

```
Patrón: es02-payment_py-transactions*
Privilegio: all
```

**Permisos de Kibana:**

El usuario tiene privilegios sobre el Space:

```
payment-interfaces
```

Entre los permisos identificados se encuentran:

- Discover: `all`.
    
- Dashboard: `all`.
    
- Dev Tools: `all`.
    
- Index Patterns: `all`.
    
- Saved Objects Management: `all`.
    
- Visualize: `read`.
    

### 8.3. Interpretación

El usuario `payment` dispone de permisos administrativos relevantes sobre los recursos destinados a PAYMENT.

Sin embargo:

- No dispone de visibilidad general sobre todos los índices del clúster.
    
- No tiene el privilegio `read_ilm`.
    
- No tiene el privilegio `manage_ilm`.
    
- No se ha validado una identidad específica para el agente Filebeat.
    

### 8.4. Consideración de seguridad

La cuenta `payment` tiene privilegios amplios dentro de su patrón autorizado.

Para la implementación se recomienda definir una identidad de servicio específica para Filebeat, con los permisos mínimos necesarios para enviar eventos.

Esto evitará utilizar una cuenta administrativa para tareas automáticas de ingesta.

---

## 9. Relevamiento de Kibana

### 9.1. Acceso

Se utilizó la interfaz:

`http://test-kibana.claro.com.ar:5601`

Se confirmó el acceso al Space:

```
payment-interfaces
```

### 9.2. Discover

El usuario pudo acceder a Discover.

Sin embargo, Kibana solicitó crear un Index Pattern antes de utilizar la funcionalidad.

### 9.3. Index Patterns

No se identificaron Index Patterns relacionados con PAYMENT dentro del Space consultado.

Tampoco se identificó un Index Pattern correspondiente a:

```
es02-payment_py-transactions-*
```

### 9.4. Index Templates y Component Templates

La interfaz de administración presenta recursos de Index Templates y Component Templates.

No se identificaron recursos específicos para el patrón PAYMENT solicitado.

Los Index Templates no sustituyen a los Index Patterns utilizados para consultar datos desde Discover.

### 9.5. Conclusión

Kibana es accesible y el usuario dispone de permisos para administrar Index Patterns dentro del Space correspondiente.

No se creó ningún Index Pattern durante el relevamiento.

Su configuración queda pendiente hasta disponer de la estrategia definitiva de indexación y los primeros índices o Data Streams de PAYMENT.

La versión exacta de Kibana no fue comprobada directamente durante este bloque.

---

## 10. Conectividad desde PAYMENT hacia Elasticsearch

### 10.1. Endpoint verificado

**Origen:**

`paymendb01ptl.claro.amx`

**Destino:**

`test-elastic.claro.com.ar:9200`

Las pruebas se ejecutaron desde el servidor PAYMENT de Testing.

### 10.2. Resolución DNS

Se ejecutó:

```
getent hosts test-elastic.claro.com.ar
```

Resultado:

```
10.202.48.104
10.202.50.102
10.202.50.101
```

Las direcciones corresponden a los nodos:

|IP|Nodo|
|---|---|
|`10.202.48.104`|`elk01atl.claro.amx`|
|`10.202.50.102`|`elk03adl.claro.amx`|
|`10.202.50.101`|`elk02adl.claro.amx`|

El endpoint dispone de múltiples registros DNS.

No se comprobó individualmente la conectividad hacia cada una de las direcciones.

### 10.3. Conectividad TCP

Se ejecutó:

```
nc -vz -w 5 test-elastic.claro.com.ar 9200
```

Resultado relevante:

```
Connected to 10.202.50.101:9200
```

Se confirmó la conectividad TCP al puerto 9200 del nodo `elk02adl`.

### 10.4. Respuesta HTTP

Se ejecutó una petición HTTP sin credenciales:

```
curl -sS -o /dev/null \
  -w 'HTTP %{http_code}\n' \
  --connect-timeout 5 \
  --max-time 10 \
  http://test-elastic.claro.com.ar:9200/
```

Resultado:

```
HTTP 401
```

La respuesta confirma que el endpoint es accesible mediante HTTP y que la petición sin credenciales requiere autenticación.

No se validó la escritura de documentos mediante este procedimiento.

### 10.5. Verificación HTTPS/TLS

Se ejecutó:

```
curl -vk --connect-timeout 5 --max-time 10 \
  https://test-elastic.claro.com.ar:9200/
```

La conexión TCP se estableció con:

```
10.202.50.101:9200
```

La negociación TLS falló con:

```
curl: (35) SSL routines:ssl3_get_record:wrong version number
```

**Interpretación:**

El error es compatible con un endpoint que atiende HTTP sin TLS en el puerto consultado.

No se consiguió establecer una sesión HTTPS ni verificar certificados.

Esta prueba no permite afirmar que toda la infraestructura Elasticsearch carezca de un endpoint HTTPS alternativo.

### 10.6. Resultado de conectividad

|Verificación|Estado|
|---|---|
|Resolución DNS|Correcta|
|Conexión TCP 9200|Confirmada hacia un nodo|
|Respuesta HTTP|Confirmada|
|Autenticación requerida|Confirmada para la petición|
|Negociación HTTPS|Fallida en el nodo probado|
|Validación de certificado TLS|No realizada|
|Envío autenticado de eventos|No realizado|

**Conclusión:** existe conectividad desde el servidor PAYMENT hacia Elasticsearch mediante el endpoint probado.

Antes de habilitar el envío de logs deberá confirmarse el mecanismo autorizado de transporte seguro y autenticación.

---

## 11. Hallazgos técnicos consolidados

### H01 — Filebeat no identificado en el servidor PAYMENT

No se encontró una instalación convencional ni ejecución visible de Filebeat en `paymendb01ptl.claro.amx`.

La implementación probablemente requerirá instalar el agente.

### H02 — Clúster Elasticsearch operativo

El clúster `es02-testing` respondió correctamente y presentó estado `green`.

### H03 — Infraestructura de cinco nodos

Se identificaron cinco nodos, de los cuales tres son nodos de datos.

### H04 — Distribución de shards correcta en la muestra

Los 327 shards activos estaban asignados, sin shards pendientes ni tareas en reubicación.

### H05 — Diferencias de utilización de recursos

Se observaron porcentajes elevados de RAM física en algunos nodos y diferencias de almacenamiento entre los nodos de datos.

No se identificó evidencia suficiente para recomendar ajustes de capacidad.

### H06 — Índices PAYMENT no identificados

No se encontraron índices coincidentes con los patrones PAYMENT consultados dentro del alcance visible para el usuario.

### H07 — Data Streams no identificados

La consulta correspondiente devolvió una colección vacía.

### H08 — Templates generales aplicables a PAYMENT

Los templates `limpia` y `limpiaidx-template-15d` coinciden con el patrón de índices previsto.

### H09 — Política ILM heredada por el patrón PAYMENT

La simulación confirmó que el índice propuesto heredaría la política `limpiaidx-15d`.

Su comportamiento efectivo no pudo verificarse.

### H10 — Restricción de lectura ILM

El usuario `payment` no dispone del privilegio necesario para consultar las políticas ILM.

### H11 — Ingest Pipelines disponibles

El clúster contiene pipelines de procesamiento existentes.

No se identificaron pipelines con nombres específicos de PAYMENT.

### H12 — Permisos de índices delimitados

El usuario `payment` tiene privilegios `all` sobre `es02-payment_py-transactions*`, pero no visibilidad general sobre los índices del clúster.

### H13 — Kibana accesible

Se confirmó el acceso al Space `payment-interfaces` y a Discover.

### H14 — Index Pattern de PAYMENT no identificado

No se encontró un Index Pattern relacionado con PAYMENT en el Space consultado.

### H15 — Conectividad Elasticsearch confirmada

Desde `paymendb01ptl` se verificó resolución DNS, conexión TCP y respuesta HTTP del endpoint Elasticsearch.

### H16 — HTTPS no operativo en la prueba realizada

La conexión HTTPS al puerto 9200 del nodo seleccionado falló durante la negociación TLS.

Se requiere confirmar la disponibilidad de un endpoint seguro para la ingesta.

---

## 12. Información pendiente y tratamiento

|ID|Pendiente|Tratamiento|
|---|---|---|
|P01|Confirmar identidad y permisos de ejecución del futuro Filebeat|Bloque 3 y 4|
|P02|Definir versión y método de instalación de Filebeat|Bloque 3|
|P03|Confirmar el endpoint oficial de ingesta|Validación con administradores ELK|
|P04|Verificar conectividad hacia todos los destinos DNS si se requiere|Validación complementaria|
|P05|Confirmar disponibilidad de HTTPS/TLS|Validación con administradores ELK|
|P06|Definir credenciales e identidad de servicio para Filebeat|Bloque 3|
|P07|Obtener contenido de `limpiaidx-15d` y `limpiaidx`|Solicitar `read_ilm` o definiciones|
|P08|Confirmar período de retención requerido|Bloque 3|
|P09|Definir índices tradicionales o Data Streams|Bloque 3|
|P10|Definir templates, mappings y campos|Bloque 3|
|P11|Determinar necesidad de Ingest Pipeline|Bloque 3|
|P12|Definir tratamiento de información sensible|Bloque 3|
|P13|Configurar Index Pattern de PAYMENT|Bloque 5|
|P14|Verificar versión exacta de Kibana|Verificación complementaria|
|P15|Validar permisos efectivos de escritura con identidad de ingesta|Bloque 4|
|P16|Verificar asignación de pipelines por configuraciones generales|Bloque 3|

No se considera necesario resolver todos estos pendientes para cerrar el relevamiento.

Las decisiones que afectan a la arquitectura y seguridad deberán resolverse antes de habilitar la ingesta.

---

## 13. Estado de las tareas

|Tarea|Estado|
|---|---|
|3. Relevar instalación y configuración actual de Filebeat|Completada para el servidor objetivo|
|4. Relevar Elasticsearch y Kibana: índices, templates, pipelines y permisos|Relevamiento principal completado, con limitaciones documentadas|

### 13.1. Validaciones realizadas

- Verificación de instalación RPM de Filebeat.
    
- Inspección de servicios systemd.
    
- Revisión de rutas habituales.
    
- Búsqueda de procesos relacionados.
    
- Inventario de contenedores Podman.
    
- Identificación del clúster Elasticsearch.
    
- Verificación de salud y nodos.
    
- Consulta de distribución de almacenamiento.
    
- Búsqueda de índices y aliases PAYMENT.
    
- Consulta de Data Streams.
    
- Inventario de Index Templates.
    
- Inspección de templates composables y legacy.
    
- Simulación de template para PAYMENT.
    
- Intento de consulta de políticas ILM.
    
- Inventario general de Ingest Pipelines.
    
- Búsqueda de pipelines específicos.
    
- Identificación de roles y privilegios.
    
- Validación de acceso a Kibana y Discover.
    
- Verificación de resolución DNS.
    
- Prueba de conectividad TCP.
    
- Prueba de respuesta HTTP.
    
- Prueba de negociación HTTPS.
    

**No se instalaron agentes, crearon índices, modificaron templates, cambiaron permisos ni alteraron recursos del clúster durante el relevamiento.**

---

## 14. Próximos pasos

El siguiente bloque corresponde al **Bloque 3 — Diseño técnico de la solución**.

### Tarea 5 — Definir diseño de ingesta, campos, filtros y estrategia de indexación

Se deberá definir:

**1. Arquitectura de captura**

- Instalación y versión de Filebeat.
    
- Inputs y rutas de lectura.
    
- Acceso a archivos mediante permisos mínimos.
    
- Tratamiento multilínea.
    
- Manejo de rotación y sobrescritura.
    
- Colas y recuperación ante interrupciones.
    

**2. Procesamiento de eventos**

- Campos identificadores.
    
- Extracción de timestamps y severidad.
    
- Parsing y filtrado.
    
- Tratamiento de información sensible.
    
- Necesidad de Ingest Pipelines.
    

**3. Almacenamiento**

- Índices tradicionales o Data Streams.
    
- Convención de nombres.
    
- Index Templates y mappings.
    
- Políticas ILM y retención.
    
- Estrategia de rollover y crecimiento.
    

**4. Seguridad**

- Endpoint oficial.
    
- TLS y autenticación.
    
- Identidad de servicio Filebeat.
    
- Privilegios mínimos.
    
- Acceso de lectura y visualización en Kibana.
    

**5. Validación**

- Criterios de aceptación.
    
- Verificación de integridad de eventos.
    
- Recuperación ante interrupciones.
    
- Control de duplicaciones y pérdidas.
    

La implementación comenzará con los servicios disponibles en Testing, incluyendo la API REST y contemplando la incorporación de los servicios Socket conforme al alcance acordado.

---

## 15. Conclusión

El Bloque 2 permitió determinar el estado actual de Filebeat y de la infraestructura Elasticsearch/Kibana destinada a PAYMENT.

**Principales resultados:**

1. No se identificó una instalación convencional de Filebeat en el servidor `paymendb01ptl.claro.amx`.
    
2. El clúster `es02-testing`, con Elasticsearch 7.17.2, se encontraba operativo y sin problemas de asignación de shards en la consulta realizada.
    
3. No se identificaron índices, aliases, Data Streams ni Index Patterns específicos para PAYMENT dentro del alcance consultable.
    
4. Se confirmó la existencia de templates generales que afectarían a los futuros índices `es02-payment_py-transactions-*`.
    
5. La simulación indicó que estos índices heredarían la política ILM `limpiaidx-15d`, cuyo contenido no pudo verificarse debido a restricciones de permisos.
    
6. El clúster dispone de Ingest Pipelines, pero no se encontraron pipelines identificados por nombres relacionados directamente con PAYMENT.
    
7. Se confirmaron los permisos actuales del usuario `payment`, incluyendo administración de índices dentro del patrón autorizado y acceso al Space `payment-interfaces`.
    
8. Se verificó conectividad DNS, TCP y HTTP desde el servidor PAYMENT hacia Elasticsearch. El endpoint solicita autenticación, mientras que la prueba HTTPS no logró establecer una sesión TLS.
    

**El Bloque 2 queda cerrado como relevamiento técnico**, conservando las restricciones y verificaciones pendientes debidamente registradas.

La siguiente etapa será definir el diseño técnico de ingesta, almacenamiento, procesamiento y seguridad antes de realizar modificaciones en Filebeat o Elasticsearch.