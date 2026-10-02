Genera un manual de usuario para un pipeline de Jenkins en formato Markdown.
El documento debe seguir esta estructura y convenciones:

- Título principal con el nombre del proyecto entre corchetes y el tipo de pipeline
- Sección de metadatos: autores, links a pipelines y links a repositorios
- Introducción breve: qué hace el pipeline, qué tipo de job de Jenkins usa, y qué función/archivo lo implementa
- Sección "Cómo funciona": cómo se dispara, qué stages hay que monitorear, y qué parámetros opcionales acepta
- Resumen general en lista: tipo, uso, tecnología y flujo de trabajo resumido
- Flujo de trabajo detallado por stage: descripción breve + bloque de código relevante
- Configuración requerida en cada repositorio: qué archivos o ajustes necesita el repo que consume el pipeline, con bloques de código de ejemplo
- Consideraciones finales: solo lo que no se haya dicho antes

Convenciones:
- Los nombres de comandos, parámetros, funciones y archivos siempre en backticks
- Los bloques de código con el lenguaje indicado (groovy, kotlin, properties, etc.)
- Nada de redundancias: si algo ya se explicó, no repetirlo
- Tono técnico pero directo, sin relleno
  
  
  Ejemplo Plantilla: 
  # [PROYECTO] Nombre del Pipeline - Manual de Usuario

**Autores:**
- Nombre Apellido

**Pipelines:**
- [Nombre descriptivo](https://link-al-pipeline)

**Repos:**
- [nombre-del-repo](https://github.com/org/nombre-del-repo)

---

## Introducción

Descripción breve de qué hace el pipeline y para qué proyecto.

---

## Cómo funciona

- **Activación automática:** describir cuándo y cómo se dispara el pipeline.
- **Stages a monitorear:** listar los stages críticos que deben terminar correctamente.
- **Parámetros opcionales:** describir si el pipeline acepta parámetros desde el Jenkinsfile.
- **Comportamientos especiales:** describir cualquier lógica de skip u otras condiciones.

---

## 1. Resumen general

- **Tipo:** ...
- **Uso:** ...
- **Tecnología:** ...
- **Flujo de trabajo:**
  - **Stage 1:** descripción breve
  - **Stage 2:** descripción breve
  - **Stage N:** descripción breve

---

## 2. Flujo de trabajo

### 2.1 Nombre del Stage

Descripción de qué hace este stage.

```groovy
// código relevante
```

### 2.2 Nombre del Stage

Descripción de qué hace este stage.

```groovy
// código relevante
```

### 2.N Nombre del Stage Final

Descripción de qué hace este stage.

```groovy
// código relevante
```

---

## 3. Configuración requerida en cada repositorio

Para que el pipeline funcione, cada repositorio debe contar con lo siguiente:

- **Nombre del ajuste:** descripción de por qué es necesario.

```kotlin
// ejemplo de configuración
```

- **Jenkinsfile:** debe invocar la shared library y llamar a la función correspondiente.

```groovy
@Library('nombre-libreria') _
nombrefuncion(PARAMETRO: true)
```

---

## 4. Consideraciones finales

- Alcance del pipeline: qué hace y qué **no** hace.
- Para dudas, contactar al equipo DevOps.