# 1. Primero, entendamos qué construimos

El objetivo del proyecto es centralizar los logs de Claro PAYMENT en Elasticsearch para poder consultarlos y analizarlos desde Kibana, sin tener que entrar manualmente a cada servidor y revisar archivos.

Esta es nuestra arquitectura actual:

Servidor PAYMENT — Testing

`paymendb01ptl`

API REST

payment.log

Socket

routlog.txt

Filebeat 7.17.2

Lee archivos, agrupa multilínea, identifica servicios y envía eventos

Elasticsearch — es02-testing

Ingest Pipeline `payment-logs-v1` v11

Extrae campos técnicos y funcionales mediante Grok

Índices diarios

es02-payment_py-transactions-*

Kibana — Data View y Discover

Consultas, filtros y visualización de logs

Una pequeña corrección a tu explicación: el pipeline no se ejecuta dentro de Kibana, sino dentro de Elasticsearch. Kibana es la interfaz desde la que podemos crearlo, modificarlo, probarlo y consultar sus resultados.

Filebeat indica qué pipeline utilizar, Elasticsearch procesa el documento antes de indexarlo y Kibana permite visualizarlo después.

## 2. El proceso completo explicado por etapas

|Etapa|Qué hicimos|Para qué|
|---|---|---|
|1. Relevamiento|Identificamos servidores, aplicaciones, logs, formatos y permisos|Saber qué necesitamos capturar|
|2. Instalación|Desplegamos Filebeat mediante Podman y systemd|Tener un agente que lea los logs|
|3. Configuración|Definimos inputs, multiline, identificadores y output|Decirle a Filebeat qué leer y dónde enviarlo|
|4. Seguridad|Configuramos el keystore y la autenticación|Evitar almacenar la contraseña directamente en el YAML|
|5. Ingesta|Conectamos Filebeat con Elasticsearch|Centralizar los eventos|
|6. Procesamiento|Construimos y validamos el pipeline Grok|Transformar mensajes en campos consultables|
|7. Visualización|Creamos el Data View en Kibana|Consultar y filtrar los datos|
|8. Estabilización|Estamos revisando reintentos, almacenamiento y recuperación|Preparar una solución operativamente confiable|

Ese es el recorrido que conviene memorizar.

# 3. Lo más importante: ¿cómo extrajimos los campos funcionales?

Este fue uno de los trabajos centrales del proyecto y vale la pena entenderlo bien.

No comenzamos escribiendo expresiones Grok al azar. Primero investigamos cómo estaban construidos los mensajes reales.

## El procedimiento que seguimos

1. Identificar los datos que queríamos obtener
    
    Separamos los atributos útiles en categorías: identificadores de transacción, identificación de servicios o procesadoras, estados, errores e importes.
    
    Por ejemplo, queríamos extraer `payment.tx_id` para poder buscar una transacción sin revisar manualmente el texto completo.
    
2. Examinar los mensajes originales
    
    Analizamos muestras de los logs API REST y Socket para identificar dónde aparecían esos atributos.
    
    Descubrimos que un mismo concepto podía aparecer con distintos formatos, dependiendo del componente o método que generaba el mensaje.
    
3. Identificar las variantes de escritura
    
    Por ejemplo, un identificador podía aparecer así:
    
    ```
    txId=74532312
    txId: 74532312
    txId -> 74532312
    ```
    
    Los tres expresan el mismo atributo, pero no tienen la misma estructura.
    
    También encontramos mensajes con valores `null`, texto libre o menciones de atributos sin un valor que pudiera extraerse correctamente.
    
4. Diseñar patrones Grok para las variantes válidas
    
    Grok identifica estructuras de texto mediante patrones. El procesamiento asigna el valor reconocido a un campo del documento.
    
    Por ejemplo, un patrón puede identificar `txId=74532312` y generar:
    
    ```
    {
      "payment": {
        "tx_id": "74532312"
      }
    }
    ```
    
    También agregamos condiciones mediante `event.dataset` para que cada extractor se aplique al servicio correspondiente.
    
5. Probar los patrones antes de aplicarlos
    
    Utilizamos simulaciones del pipeline para comprobar si se extraía exactamente el valor esperado.
    
    Si un patrón no coincidía, examinábamos el mensaje y determinábamos si faltaba una variante o si el valor realmente no tenía el formato esperado.
    
    No considerábamos todo mensaje sin coincidencia como un error del pipeline.
    
6. Validar con eventos reales
    
    Después de las simulaciones, reprocesamos grupos pequeños de documentos de Elasticsearch.
    
    Comprobamos cuántos obtenían el campo, si mantenían el parsing correcto y si el resto de atributos permanecía intacto.
    
7. Reprocesar el histórico y medir la cobertura
    
    Cuando los patrones quedaron validados, aplicamos la versión 11 al histórico Socket de 1.613 documentos.
    
    Finalmente comparamos el número de atributos extraídos con lo identificado durante el relevamiento.
    

### ¿Por qué no se pudieron extraer todas las menciones?

Porque una mención a un atributo no siempre representa un valor válido para extraer.

Por ejemplo:

|Mensaje|¿Extraer?|Motivo|
|---|---|---|
|`Importe: 15000`|Sí|Contiene un valor numérico reconocible|
|`Importe: XXXXX`|No|No contiene un importe numérico válido|
|`Colector 11`|Sí|Contiene un identificador reconocible|
|`Error al consultar colector`|No|Es una descripción, no un identificador|
|`errorCode=null`|No|No hay valor funcional que extraer|

Por eso analizamos candidatos, verificamos sus formatos y decidimos qué patrones incorporar.

Para Socket identificamos 214 menciones candidatas, de las cuales 189 cumplían los formatos que decidimos extraer. El pipeline logró recuperar esas 189 coincidencias después del reprocesamiento.

Esto no significa que el pipeline interprete el 88,3 % de las transacciones: significa que cubre el 88,3 % de las menciones candidatas de esos ocho atributos.

### La idea que deberías anotar

> Para extraer campos funcionales de logs, primero identificamos los atributos útiles, estudiamos cómo aparecen en mensajes reales, clasificamos sus formatos, construimos patrones Grok, validamos las extracciones y finalmente las aplicamos al histórico.

Ese procedimiento es reutilizable para futuros servicios, no solamente para Claro PAYMENT.

# 4. ¿En qué estado están nuestros siete bloques?

Tomando tus bloques, los resultados de nuestras pruebas y la documentación existente:

|Bloque|Objetivo|Estado actual|
|---|---|---|
|1. Relevamiento|Infraestructura, archivos, formatos, permisos|Completado para Testing inicial|
|2. Filebeat|Instalación, Podman, systemd, configuración|Implementado|
|3. Ingesta ELK|Envío, autenticación, identificación|Operativo; TLS pendiente|
|4. Procesamiento|Parsing técnico y extracción funcional|Implementación v11 validada|
|5. Almacenamiento|Mappings, templates, ILM|Parcial|
|6. Visualización|Data View, consultas funcionales|Parcial|
|7. Pruebas y cierre|Recuperación, integridad, documentación, Producción|Pendiente de cierre|

Hay una diferencia importante entre el estado que compartiste y nuestro estado más reciente: el Bloque 4 ya no está en curso en lo relativo a la extracción funcional acordada. Tenemos `payment-logs-v1` v11, ocho extractores Socket validados, 1.613 eventos Socket con parsing correcto y una comprobación de regresión satisfactoria sobre los campos seleccionados de API REST.

No significa que no vayamos a ampliar campos en el futuro. Significa que el alcance que definimos para esta iteración está implementado.

## 5. ¿Qué queda por hacer y qué está bloqueado?

|Pendiente|¿Podemos avanzar?|Qué necesitamos|
|---|---|---|
|Guardar v11 y la configuración final de Filebeat en GitHub|Sí|Verificar cambios, commit y push|
|Revisar mappings de identificadores e importes|Sí|Consultas de solo lectura a Elasticsearch|
|Revisar templates y política ILM|Parcialmente|Consultar la configuración disponible; solicitar permisos si faltan|
|Validar campos funcionales en Kibana Discover|Sí|Acceso actual al Space y refrescar campos del Data View si corresponde|
|Probar recuperación ante cortes|Sí, con coordinación|Preparar una prueba segura que afecte solamente a Filebeat|
|Verificar rotación y conservación de archivos en origen|Sí|Relevar la configuración efectiva y los archivos rotados|
|Revisar seguridad de transporte|Depende de Claro|Confirmar disponibilidad o habilitación de HTTPS/TLS|
|Definir tratamiento definitivo de datos sensibles|Depende de los responsables|Criterios de conservación, enmascaramiento y acceso|
|Consolidar documentación|Sí|Organizar los informes y archivos de configuración|
|Preparar Producción|Parcialmente|Validar diferencias de infraestructura y decisiones pendientes|

Un punto relevante: la retención de 15 días que identificamos pertenece a la política de Elasticsearch. No hemos confirmado que los archivos originales de las aplicaciones se conserven durante esos mismos 15 días.

También debemos distinguir la configuración de reintentos que acabamos de preparar de su aplicación efectiva: todavía no tenemos una prueba posterior al reinicio de Filebeat ni una simulación de recuperación ante corte.

# 6. Cómo organizaría la documentación

Te recomiendo mantener una estructura sencilla en el repositorio, separando configuraciones ejecutables de documentación.

```
payment-elk/
│
├── README.md
│
├── filebeat/
│   ├── compose.yml
│   └── filebeat.yml
│
├── elasticsearch/
│   └── pipelines/
│       └── payment-logs-v1.json
│
└── docs/
    ├── 01-relevamiento.md
    ├── 02-arquitectura.md
    ├── 03-filebeat.md
    ├── 04-ingesta-y-procesamiento.md
    ├── 05-campos-funcionales.md
    ├── 06-almacenamiento-y-kibana.md
    ├── 07-validaciones.md
    └── 08-pendientes-produccion.md
```

Es una estructura propuesta, no una estructura que hayamos confirmado actualmente en GitHub.

El documento `05-campos-funcionales.md` contendría la matriz que mencionaste que quieres preparar más adelante: campos propuestos, implementados, descartados o pendientes, separados por API REST y Socket, con su justificación.

No necesitamos redactar todo otra vez desde cero. Ya tenemos varios informes; el trabajo sería consolidarlos y eliminar las versiones que describen estados anteriores como si todavía fueran los actuales.

# 7. ¿Qué haría ahora?

Mi recomendación es seguir esta secuencia corta:

1. Cerrar Filebeat: verificar que los cuatro parámetros estén aplicados, realizar la prueba controlada de recuperación y registrar el resultado.
    
2. Consolidar almacenamiento: revisar mappings, templates y retención efectiva.
    
3. Cerrar visualización: comprobar en Kibana los campos funcionales de ambos servicios.
    
4. Versionar y documentar: guardar la configuración final y los informes consolidados, incluyendo las decisiones y limitaciones.
    

El versionado en GitHub puede hacerse incluso antes de completar todas las pruebas, registrando claramente lo que está validado y lo que sigue pendiente.

## Resumen para tu cuaderno

Qué aprendimos y construimos

- Filebeat: lee, agrupa e identifica los logs y los envía a Elasticsearch.
    
- Keystore: almacena la contraseña utilizada para autenticarse; no cifra por sí solo la comunicación.
    
- Ingest Pipeline: transforma los mensajes y extrae campos mediante Grok.
    
- Índices: almacenan los documentos procesados, en nuestro caso bajo una convención diaria.
    
- Mappings: determinan cómo Elasticsearch interpreta y permite consultar cada campo.
    
- ILM: administra el ciclo de vida de los índices según una política.
    
- Data View: permite a Kibana consultar los índices que coinciden con un patrón.
    
- Reintentos: Filebeat intenta recuperar la comunicación automáticamente, pero la garantía de recuperación depende también del almacenamiento y la disponibilidad de los archivos originales.
    

Dónde estamos: ya tenemos un flujo funcional completo desde las aplicaciones PAYMENT hasta Kibana. El mayor trabajo de instalación y parsing está realizado.

Qué estamos haciendo ahora: pasar de una implementación que funciona en Testing a una implementación verificada, documentada, reproducible y preparada para futuras incorporaciones y para Producción.

Y tu intuición sobre el proceso de extracción fue correcta: relevar, analizar formatos, definir campos, construir patrones, probar y validar. Es justamente la metodología que seguimos, y probablemente sea una de las partes más valiosas de este proyecto para que puedas repetirla en otros entornos.