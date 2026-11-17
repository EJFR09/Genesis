---
status: open
priority: normal
dateCreated: 2026-10-28T07:51:17.807-03:00
dateModified: 2026-10-28T07:51:42.890-03:00
tags:
---
## 1. Resumen de la implementación

### Arquitectura actual


### Trabajos realizados
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
**4. Catálogo funcional*
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
## 2. Temas a tratar en la reunión

### 2.1. Política ILM — Retención en Elasticsearch
**Situación actual:**
- Los índices PAYMENT utilizan `limpiaidx-15d`.
- El nuevo template también referencia esa política.
- No tenemos permisos `read_ilm` para consultar su definición completa.
- No está confirmado que la política elimine índices después de 15 días.
**Preguntas:**
- ¿Cuántos días deben conservarse los logs en Testing y Producción?
- ¿La política `limpiaidx-15d` cumple ese requisito?
- ¿Podemos obtener `read_ilm` o su definición mediante el administrador?
- ¿Mantendremos la política compartida o crearemos una específica para PAYMENT?

### 2.2. Permisos de Elasticsearch

**Situación actual:**

- Usuario `payment`.
    
- Rol `paymentADM`.
    
- Permisos sobre los índices PAYMENT.
    
- Permisos para administrar templates y pipelines.
    
- Sin privilegio `read_ilm`.
    

**Preguntas:**

- ¿Podemos disponer de permisos de solo lectura sobre ILM?
    
- ¿Utilizaremos una identidad dedicada para la ingesta en Producción?
    
- ¿Quién administrará credenciales y permisos?
    

**Recomendación:** separar los privilegios administrativos de los permisos mínimos necesarios para Filebeat.

**Estado:** Pendiente  
**Decisión:**  
**Responsable:**

### 2.3. Seguridad HTTP / HTTPS

**Situación actual:**

- Filebeat envía eventos mediante HTTP autenticado.
    
- Endpoint: `http://test-elastic.claro.com.ar:9200`.
    
- La contraseña está protegida localmente mediante Keystore.
    
- El transporte no está cifrado.
    

**Preguntas:**

- ¿Existe un endpoint HTTPS disponible?
    
- ¿Se acepta formalmente HTTP en Testing?
    
- ¿Qué mecanismo TLS se utilizará en Producción?
    

**Recomendación:** utilizar HTTPS con validación de certificados y autenticación de privilegios mínimos.

**Estado:** Pendiente  
**Decisión:**  
**Responsable:**

### 2.4. Validación del catálogo funcional

**Situación actual:**

- 36 campos funcionales implementados.
    
- 34 atributos adicionales observados.
    
- 34 atributos no observados retirados del catálogo operativo.
    
- Pipeline v11 validado.
    

**Preguntas:**

- ¿Son suficientes los campos implementados?
    
- ¿Cuáles de los campos adicionales deberían incorporarse?
    
- ¿Qué campos son obligatorios para las consultas?
    
- ¿Los importes deben conservarse como `keyword` o convertirse en valores numéricos?
    
- ¿Existen requisitos diferentes entre Testing y Producción?
    

**Material de referencia:** PDF del catálogo final.

**Estado:** Pendiente  
**Decisión:**  
**Responsable:**

### 2.5. Tratamiento de datos sensibles

**Situación actual:**

- Los registros pueden contener identificadores de sesión, números de servicio y datos de transacciones.
    
- El pipeline conserva el campo `message` original.
    
- No existe todavía una aprobación funcional completa del tratamiento de información sensible.
    

**Preguntas:**

- ¿Qué datos deben enmascararse?
    
- ¿Qué atributos no deben enviarse a Elasticsearch?
    
- ¿Puede conservarse el campo `message` completo?
    
- ¿Qué usuarios tendrán acceso a los registros en Producción?
    

**Recomendación:** aplicar minimización de datos, controles de acceso y enmascaramiento cuando corresponda.

**Estado:** Pendiente  
**Decisión:**  
**Responsable:**

### 2.6. Retención local de archivos

**Situación actual:**

- API REST utiliza rotación Log4j2.
    
- Socket `documenta` utiliza rotación por tamaño.
    
- No está completamente confirmado el mecanismo de eliminación de archivos históricos.
    
- La retención local es independiente de ILM.
    

**Preguntas:**

- ¿Existe algún script, cron o proceso de limpieza?
    
- ¿Quién administra esa limpieza?
    
- ¿Cuántos días deben conservarse los archivos locales?
    
- ¿Existe almacenamiento adicional para auditoría?
    

**Recomendación:** definir conservación local considerando espacio disponible y tiempo suficiente para recuperar eventos pendientes durante interrupciones de ingesta.

**Estado:** Pendiente  
**Decisión:**  
**Responsable:**

### 2.7. Nuevas procesadoras Socket

**Situación actual:**

- `documenta` está incorporada.
    
- Las demás procesadoras requieren relevamiento particular.
    
- No debemos asumir que todas generan el mismo formato de logs.
    

**Preguntas:**

- ¿Qué procesadoras se incorporarán?
    
- ¿En qué servidores están desplegadas?
    
- ¿Cuál es el orden de prioridad?
    
- ¿Quién proporcionará rutas y formatos de logs?
    
- ¿Quién validará los campos funcionales?
    

**Recomendación:** incorporar nuevas procesadoras siguiendo un procedimiento de relevamiento, configuración y validación individual.

**Estado:** Pendiente  
**Decisión:**  
**Responsable:**

### 2.8. Preparación de Producción

**Situación actual:**

- Implementación operativa en Testing para API REST y `documenta`.
    
- Pipeline v11 y template v1 disponibles.
    
- Quedan decisiones de seguridad, retención y aprobación funcional.
    

**Preguntas:**

- ¿Quién aprobará el resultado de Testing?
    
- ¿Qué requisitos deben cumplirse antes de Producción?
    
- ¿Quién administrará los accesos y credenciales productivos?
    
- ¿Cuál será el alcance inicial de Producción?
    

**Estado:** Pendiente  
**Decisión:**  
**Responsable:**

---

## 3. Comandos de Kibana — Dev Tools

Todos los comandos siguientes son consultas de lectura, salvo la simulación, que no crea ni modifica recursos.

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

## 4. Comandos de terminal — Servidor PAYMENT

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

## 5. Herramientas necesarias

### Antes de la reunión

- GitHub abierto con las configuraciones PAYMENT–ELK.
    
- Kibana abierto en el Space `payment-interfaces`.
    
- Dev Tools disponible.
    
- Terminal SSH conectada a `paymendb01ptl`.
    
- PDF del catálogo de campos.
    
- Informes técnicos de los bloques realizados.
    
- Esta guía de reunión.
    
- Libreta o documento para registrar respuestas.
    

### Recursos importantes

**Kibana:**  
[http://test-kibana.claro.com.ar:5601](http://test-kibana.claro.com.ar:5601)

**Elasticsearch:**  
[http://test-elastic.claro.com.ar:9200](http://test-elastic.claro.com.ar:9200)

**Índices:**  
`es02-payment_py-transactions-*`

**Pipeline:**  
`payment-logs-v1`

**Template:**  
`payment-transactions-template`

**ILM:**  
`limpiaidx-15d`

**Space:**  
`payment-interfaces`

---

## 6. Resumen de acuerdos

Completar durante o después de la reunión.

|Tema|Estado|Acuerdo / próxima acción|Responsable|
|---|---|---|---|
|Política ILM|Pendiente|||
|Permisos Elasticsearch|Pendiente|||
|HTTP / HTTPS|Pendiente|||
|Catálogo funcional|Pendiente|||
|Datos sensibles|Pendiente|||
|Retención local|Pendiente|||
|Nuevas procesadoras|Pendiente|||
|Producción|Pendiente|||

**Estados posibles:** Resuelto / Pendiente / Bloqueado.

---

## 7. Cierre de la reunión

- Registrar las decisiones tomadas.
    
- Identificar responsables de cada pendiente.
    
- Determinar qué configuraciones requieren modificación.
    
- Separar los cambios de Testing de los requisitos para Producción.
    
- Actualizar la documentación técnica.
    
- Devolver los acuerdos al Main del proyecto.
    

**Objetivo final:** finalizar la reunión con las decisiones necesarias para completar el alcance actual de Testing y definir las condiciones para avanzar hacia Producción.