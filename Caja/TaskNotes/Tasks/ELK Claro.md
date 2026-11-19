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
	- {
		  "event_type": "payment_transaction_summary",
		  "transaction_id": "TX-12345",
		  "collector_id": "14",
		  "dtinbound": "2026-11-18T10:28:06.120-03:00",
		  "dtoutbound": "2026-11-18T10:28:06.365-03:00",
		  "duration_ms": 245,
		  "ps_returncode": 0,
		  "result": "success"
		}
- **Atributos dtoutboun inbound**

# Reu-02 — Seguimiento Claro PAYMENT / ELK

## 2. Temas a tratar en la reunión

### 2.1. Transacciones de Socket — Correlación de eventos

**Situación actual**

- Una transacción puede generar múltiples eventos de log.
    
- Actualmente no hemos identificado un parámetro de correlación común y consistente en todos los eventos necesarios para reconstruir una transacción completa.
    
- Aunque existen identificadores como `txId` y `transactionId`, no hemos demostrado que permitan correlacionar todo el flujo.
    

**Evidencia visual**

- `Pasted image 20261118150732.png`
    
- `Pasted image 20261118151106.png`
    

**Propuesta**

La solución más sencilla y mantenible sería que Desarrollo genere un **evento estructurado de resumen por transacción completada**, sin reemplazar los logs técnicos actuales.

Ejemplo de formato propuesto:

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

_Los nombres, valores y significados del ejemplo son ilustrativos; deben acordarse con Desarrollo._

**Puntos a definir**

- ¿Es posible generar un evento de resumen al completar cada transacción?
    
- ¿Qué identificador único debería utilizarse?
    
- ¿En qué componente se conoce el resultado definitivo?
    
- ¿Cómo se registrarían las transacciones fallidas o interrumpidas?
    

### 2.2. Atributos `dtinbound` y `dtoutbound`

**Resultado del relevamiento**

Se realizaron búsquedas en los logs originales de Testing:

- API REST: `claro-payment-api` (`payment.log`).
    
- Socket: `documenta` (`routlog.txt`).
    

No se encontraron `dtinbound`, `dtoutbound` ni variantes que contengan `bound` en los archivos activos analizados.

**Limitación**

Esto no descarta que los atributos existan en otros logs, históricos o componentes que no fueron revisados.

**Puntos a definir**

- ¿Qué componente genera actualmente estos atributos?
    
- ¿Se encuentran en otros archivos o servicios?
    
- ¿Representan la entrada y salida de una transacción completa o de una operación particular?
    

### 2.3. Tiempos de respuesta de las transacciones

**Situación actual**

- Encontramos eventos asociados al inicio y finalización de verificaciones PS.
    
- Los timestamps visibles de API REST tienen precisión de segundos.
    
- No conseguimos reconstruir pares de eventos de inicio y final correlacionados de manera fiable.
    
- No podemos calcular actualmente la duración total de una transacción utilizando únicamente la información validada.
    

**Propuesta**

Registrar explícitamente los timestamps de entrada y salida y la duración de la operación en el evento resumen.

**Puntos a definir**

- ¿Qué se considera inicio y fin de una transacción?
    
- ¿Se desea medir el tiempo total del pago o el tiempo de procesamiento de cada componente?
    
- ¿Se necesita precisión de milisegundos?
    
- ¿Cómo se identificarán los timeouts?
    

### 2.4. Códigos `ps_returncode` y `error_code`

**Resultado del relevamiento**

Se encontraron diferentes mecanismos de registro de resultados:

|Origen|Código observado|Contexto|
|---|---|---|
|API REST|`errorCode: 0`|Respuestas de API|
|API REST|`errorCode: -1`|Error de validación|
|Socket|`06`|Paquete inválido recibido desde collector|
|Socket|`30`|Errores de transacción o de base de datos|
|Operaciones PS|`ended returning 0`|Finalización de procedimiento PS|

En Elasticsearch:

- `payment.error_code` ya está implementado y se validó en API REST.
    
- `payment.result_code` existe como campo independiente.
    
- No encontramos `payment.ps_returncode` como campo específico.
    
- Se identificaron 53 documentos con la expresión `ended returning` en los índices consultados.
    

**Puntos a definir**

- ¿Qué representa exactamente `ps_returncode`?
    
- ¿Qué procedimientos PS deberían monitorearse?
    
- ¿El código `0` significa ejecución correcta del procedimiento o transacción exitosa?
    
- ¿Debemos conservar separados `error_code`, `result_code` y `ps_returncode`?
    

**Recomendación**

Mantener los códigos diferenciados hasta confirmar su significado funcional.

### 2.5. Identificación de collectors y procesadoras

**Resultado del relevamiento**

El campo `payment.collector_id` ya está implementado en Elasticsearch.

Se observaron identificadores como:

- `1`
    
- `11`
    
- `12`
    
- `14`
    

En los archivos originales de Testing, el collector `11` aparece tanto en API REST como en Socket.

**Pendiente**

No tenemos confirmada la correspondencia entre cada `collector_id` y su procesadora o entidad funcional.

**Puntos a definir**

- ¿Qué representa exactamente un collector?
    
- ¿Cada collector corresponde a una procesadora?
    
- ¿Existe un catálogo oficial de IDs y nombres?
    
- ¿Qué información debería mostrarse en Kibana: el ID, el nombre o ambos?
    

### 2.6. Estado de Elasticsearch y catálogo funcional

**Resultado**

- El pipeline `payment-logs-v1` v11 continúa operativo.
    
- Los campos `payment.error_code`, `payment.collector_id`, `payment.tx_id` y `payment.transaction_id` existen en Elasticsearch.
    
- Se validó la extracción de códigos de error y collectors en el índice del 18/11 para API REST.
    
- Los índices recientes utilizan mappings `keyword` para estos atributos; varios índices históricos tienen mappings `text`.
    
- La validación específica de extracción de errores Socket continúa pendiente.
    

**Propuesta**

Mantener la configuración actual y evaluar modificaciones del pipeline únicamente después de acordar las definiciones funcionales.

## 3. Decisiones que necesitamos obtener

|Tema|Decisión esperada|
|---|---|
|Correlación|Acordar si Desarrollo puede emitir un evento resumen por transacción|
|`dtinbound` / `dtoutbound`|Identificar origen, disponibilidad y significado|
|Duración|Definir qué intervalo se necesita medir|
|Códigos PS|Aclarar semántica y relación con otros códigos|
|Collectors|Obtener correspondencia entre ID y entidad funcional|
|Pipeline ELK|Determinar qué campos adicionales deben incorporarse|

## 4. Próximos pasos

Una vez definidas las decisiones con el interesado:

1. Actualizar el catálogo funcional con las definiciones aprobadas.
    
2. Coordinar con Desarrollo la emisión de nuevos campos o eventos, si corresponde.
    
3. Evaluar los cambios necesarios al pipeline de Elasticsearch.
    
4. Validar la información en Kibana antes de construir las visualizaciones funcionales.
    

**Estado:** relevamiento técnico inicial completado. Pendientes de definición funcional y aprobación del interesado.