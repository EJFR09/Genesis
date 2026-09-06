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