---
status: open
priority: normal
dateCreated: 2026-10-28T07:51:17.807-03:00
dateModified: 2026-10-28T07:51:42.890-03:00
tags:
---
# 1. Resumen de la implementación

- Indices:  `es02-payment_py-transactions-*`
- Pipeline:  `payment-logs-v1`
- Template:  `payment-transactions-template`
- ILM:  `limpiaidx-15d`
- Space:  `payment-interfaces`
### 1.1 Arquitectura actual
![[Payment Testing to Elastic Cluster Flow.png]]
### 1.2 Trabajos realizados
**1. Relevamiento de infraestructura**
- Identificación del servidor PAYMENT de Testing.
- Inventario de contenedores Podman.
- Identificación de archivos, formatos, permisos y rotación de logs.
- Relevamiento de Elasticsearch y Kibana.
**2. Implementación de Filebeat**
- Filebeat 7.17.2 mediante Podman rootless.
- Administración mediante systemd de usuario.
- Captura de logs API REST y Socket `documenta`.
- Configuración de eventos multilínea.
- Registry persistente y mecanismos nativos de reintento.
- Autenticación mediante usuario `payment` y Filebeat Keystore.
**3. Ingesta y procesamiento**
- Envío de eventos hacia `es02-testing`.
- Implementación del pipeline `payment-logs-v1` v11.
- 42 procesadores.
- Extracción de timestamps, niveles y campos funcionales.
- Conservación del mensaje original.
**4. Catálogo funcional** 
- 28 campos implementados de API REST.
- 8 campos implementados de Socket.
- 34 atributos adicionales observados.
- 34 atributos no observados, retirados del catálogo operativo.
- Validación satisfactoria sobre 414 documentos API REST y 1.613 documentos Socket.
**5. Almacenamiento**
- Creación del Index Template `payment-transactions-template` v1.
- Mappings técnicos y funcionales explícitos.
- 1 shard primario y 1 réplica.
- Prioridad 100.
- Asociación con `limpiaidx-15d`.
- Simulación satisfactoria para índices futuros.
- Template guardado en GitHub.
**6. Kibana**
- Data View para `es02-payment_py-transactions-*`.
- Space `payment-interfaces`.
- Disponibilidad de eventos en Discover.
- Validación funcional completa de consultas todavía pendiente.
---

# 2. Temas a tratar en la reunión
### 2.1. Política ILM - Retención en Elasticsearch
**Situación actual:**
- Los índices PAYMENT utilizan la politica ilm `limpiaidx-15d`.
- No tenemos permisos sobre las politicas ilm, por lo que no sabemos si como funciona o si corresponde. (Podria dar errores si es necesario un alias)
- Tenemos: 
	- Usuario `payment`.
	- Rol `paymentADM`.
	- Permisos sobre los índices PAYMENT.
	- Permisos para administrar templates y pipelines.
- Sin privilegio `read_ilm`.

**Preguntas:**
- ¿Pueden darnos permisos para verificar esa policita?
- ¿Van a querer que se mantenga esa politica o van a querer una nueva para más días?
- ¿Cuantos días van a querer que elastic mantenga los logs?
### 2.2. Retención local de archivos
**Situación actual:**
- API REST utiliza rotación Log4j2.
- Socket `documenta` utiliza rotación por tamaño.
- No pudimos encontrar una configuración de retención de logs.
**Preguntas:**
- ¿Existe algún script, cron o politica de retención?
- ¿Cuántos días deben conservarse los archivos locales?
- ¿En producción tienen alguna convención de envio de logs a algun almacen ej: S3?

### 2.3. Validación del catálogo funcional
**Situación actual:**
- 36 campos funcionales implementados.
- 34 atributos adicionales observados.
- 34 atributos no observados retirados del catálogo operativo.
- Pipeline v11 validado.
- Seria más facil si los nuevos desarrollos realizan una convención de logeo en json

**Preguntas:**
- ¿Qué campos son obligatorios para las consultas?
- ¿Se sabe si algun dato es sensible y debe tener un trato especial? ¿Cuales?
### 2.4. Seguridad HTTP / HTTPS
**Situación actual:**
- Filebeat envía eventos mediante HTTP autenticado.
- Endpoint: `http://test-elastic.claro.com.ar:9200`.
- La contraseña está protegida localmente mediante Keystore.
- El transporte no está cifrado.

**Preguntas:**
- ¿Se acepta formalmente que HTTP va ser utilizado en testing o van a implementar más adelante HTTPS?
- ¿Utilizaremos una identidad dedicada para la ingesta en Producción?
### 2.5. Nuevas procesadoras Socket

**Situación actual:**
- `documenta` está incorporada.
- Las demás procesadoras requieren relevamiento particular.
- No debemos asumir que todas generan el mismo formato de logs.
**Preguntas:**
- ¿Qué procesadoras se incorporarán?


---

# 3. Comandos de Kibana — Dev Tools
### 3.1. Clúster

```
GET /
GET /_cluster/health
GET /_cat/nodes?v
GET /_cat/indices/es02-payment_py-transactions-*?v
```

### 3.2. Permisos

```
GET /_security/_authenticate
GET /_security/user/_privileges
```

### 3.3. Index Templates

```
# Template específico PAYMENT
GET /_index_template/payment-transactions-template

# Template general existente
GET /_index_template/limpiaidx-template-15d

# Otro template general
GET /_index_template/limpia
```

Simular configuración de un índice futuro:

```
POST /_index_template/_simulate_index/es02-payment_py-transactions-2026.11.18
```

### 3.4. Política ILM

```
# Consultar definición de ILM
GET /_ilm/policy/limpiaidx-15d

# Comprobar ILM aplicada
GET /es02-payment_py-transactions-*/_ilm/explain
```

La consulta de definición ILM puede devolver `403` debido a los permisos actuales.

### 3.5. Ingest Pipeline

```
GET /_ingest/pipeline/payment-logs-v1
```

### 3.6. Mappings y settings

```
GET /es02-payment_py-transactions-*/_mapping
```

```
GET /es02-payment_py-transactions-*/_settings?filter_path=*.settings.index.number_of_shards,*.settings.index.number_of_replicas,*.settings.index.lifecycle.name
```

### 3.7. Consulta de eventos recientes

```
GET /es02-payment_py-transactions-*/_search
{
  "size": 5,
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ],
  "_source": [
    "@timestamp",
    "service.name",
    "event.dataset",
    "labels.processor",
    "payment.parsing_status"
  ]
}
```

---

# 4. Comandos de terminal — Servidor PAYMENT

**Servidor:** `paymendb01ptl.claro.amx`  
**Usuario:** `payment`

### 4.1. Estado de Filebeat

```
systemctl --user status \
  filebeat-compose-payment-test.service --no-pager

podman ps --filter name=filebeat-payment-test

podman inspect filebeat-payment-test \
  --format '{{.State.Status}} {{.RestartCount}}'
```

### 4.2. Contenedores PAYMENT

```
podman ps --format '{{.Names}} {{.Status}}'
```

### 4.3. Errores recientes de Filebeat

```
podman logs --since 10m filebeat-payment-test 2>&1 \
  | grep -Ei 'error|failed|unauthorized|denied' \
  | tail -n 30
```

### 4.4. Espacio disponible

```
df -h /app/payment-test
```

### 4.5. Archivos de logs

```
ls -lh /app/payment-test/claro-payment-api/logs/

ls -lh /app/payment-test/documenta/logs/
```

Evitar mostrar mensajes completos de logs que puedan contener información sensible.

---



# Reu-02
## 2. Temas a tratar en la reunión
- **Transacciones de Socket**
	- ![[Pasted image 20261118150732.png]]
	- Como una transacción logea varios eventos, no podemos hacer una correlación en un solo evento ya que estos no tienen un parametro en comun
	- ![[Pasted image 20261118151106.png]]
	- La solución más sencilla y mantenible sería que Desarrollo genere un evento estructurado de resumen por transacción completada.
	- Ejemplo ilustrativo:
```
{
  "event_type": "payment_transaction_summary",
  "transaction_id": "TX-12345",
  "collector_id": "14",
  "dtinbound": "2026-11-18T10:28:06.120-03:00",
  "dtoutbound": "2026-11-18T10:28:06.365-03:00",
  "duration_ms": 245,
  "ps_returncode": 0,
  "result": "success"
}
```

- **Atributos dtoutboun inbound**
	- No se encontraron `dtinbound`, `dtoutbound` ni variantes que contengan `bound` en los archivos activos analizados.
	- no significa que no existan, pero por los menos en los logs de testing no lo encontramos
- Tiempos de respuestas de lsa transacciones
	- Los timestamps visibles de API REST tienen precisión de segundos.
	- No podemos calcular actualmente la duración total de una transacción utilizando únicamente la información validada.
	- No conseguimos reconstruir pares de eventos de inicio y final correlacionados de manera fiable.
- ps_returncode y error_code
	- ![[Pasted image 20261119113606.png]]
	- ![[Pasted image 20261119113620.png]]

# **Informe de avance — Claro PAYMENT / ELK**

**Fecha:** 25/11/2026  
**Ambiente:** Testing  
**Estado:** Infraestructura desplegada y operativa. Validación funcional pendiente.

## **1. Objetivo**

Implementar una infraestructura de ingesta centralizada y reproducible mediante Podman Compose, utilizando Filebeat y Logstash para procesar los logs de PAYMENT y almacenar **un documento consolidado por transacción** en Elasticsearch.

## **2. Arquitectura implementada**

```
PAYMENT (API REST + Socket)
          ↓
    Filebeat 7.17.2
          ↓
    Logstash 7.17.2
    ├── classify.rb
    ├── aggregate
    └── consolidate.rb
          ↓
 Elasticsearch 7.17.2
          ↓
        Kibana
```

**Directorio de configuración:**

`/home/payment/.config/ingestion/`

**Administración:** Podman rootless con el usuario `payment`.

## **3. Trabajo realizado**

|Componente|Implementación|Estado|
|---|---|---|
|Infraestructura anterior|Retirados los contenedores antiguos de Filebeat y Logstash|Completado|
|Podman Compose|Unificado con ambos servicios y volúmenes persistentes|Completado|
|Filebeat|Dos inputs `filestream`, API REST y Socket|Operativo|
|Logstash|Clasificación, correlación y consolidación mediante Ruby y `aggregate`|Operativo|
|Elasticsearch|Índices `es02-payment_py-transactions-*`|Operativo|
|Index Template|`payment-transactions-template` v2|Configurado|
|ILM|Política existente `limpiaidx-15d`|Configurada|
|Limpieza ELK|Retirados índices anteriores, template PoC e Ingest Pipeline antiguo|Completado|
|Permisos|Corregido acceso al volumen persistente de Filebeat|Resuelto|
|systemd|Servicio `claro-payment-ingestion.service`|Habilitado|

Los tres contenedores de las aplicaciones PAYMENT permanecieron intactos.

## **4. Resultados obtenidos**

Eventos recibidos por Logstash

# **1.845**

Documentos consolidados

# **20**

Indexaciones exitosas

# **20/20**

Los 20 documentos corresponden al servicio Socket (`claro-posjota`) y presentan `document.correlation.complete = true`.

No se detectaron duplicados entre los identificadores de transacción presentes en esta muestra.

**API REST todavía no generó documentos consolidados**, aunque su input Filebeat está configurado y activo.

## **5. Arranque automático**

Se creó y habilitó el servicio:

`claro-payment-ingestion.service`

Validaciones realizadas:

```
systemctl --user is-enabled → enabled
systemctl --user status     → active (exited)
ExecStart                  → 0/SUCCESS
Linger                     → yes
```

El Compose quedó configurado para iniciarse automáticamente al arrancar el servidor. La validación tras un reinicio real sigue pendiente.

## **6. Tareas pendientes**

1. **API REST:** determinar por qué no aparecen transacciones consolidadas y realizar una prueba funcional completa.
    
2. **Socket:** validar los campos extraídos, especialmente `READ`, `SENDING`, referencias y códigos de error.
    
3. **Calidad de datos:** revisar fechas de transacción, correlación e identificadores determinísticos para evitar duplicados.
    
4. **Cierre operativo:** guardar el template y las configuraciones definitivas en Git, documentar el despliegue y verificar el arranque automático después de un reinicio.
    

**Conclusión:** se completó el despliegue de la infraestructura definitiva de ingesta ELK, incluyendo su almacenamiento y automatización mediante systemd. El flujo técnico está funcionando y ya almacena transacciones Socket consolidadas.

**La siguiente fase es validar la calidad funcional de API REST y Socket antes de considerar finalizada la implementación.**

`74532342`