# [PROYECTO] Nombre del Pipeline (Release/Manual) - Manual de Usuario

**Autores:**
- Nombre Apellido

**Pipelines:**
- [Nombre descriptivo](https://link-al-pipeline)

**Repos:**
- [nombre-del-repo](https://github.com/org/nombre-del-repo)

## Introducción

Descripción breve de qué hace el pipeline y para qué proyecto.

- Indicar qué función o archivo implementa el pipeline.
- Indicar qué tipo de job de Jenkins utiliza (pipeline manual con `parameters`, etc.).

## Cómo funciona

1. Dirigirse a la url del pipeline y leer la descripción y funcionalidades.
2. Dar clic en "build with parameters" / "construir con parámetros".
3. Completar los parámetros:
   - **PARAMETRO_1:** descripción, formato esperado, ejemplo.
   - **PARAMETRO_2 (checkbox/selección múltiple):** qué determina, opciones disponibles.
   - **PARAMETRO_3 (condicional):** solo se habilita/usa si `PARAMETRO_X` tiene cierto valor.
   - **PARAMETRO_N:** ...
4. Dar clic en "run" / "ejecución".
5. Aguardar la finalización de todos los stages. Indicar cuáles son condicionales y bajo qué condición corren.

> OBS: describir qué pasa si un stage falla (aborta el pipeline, continúa igual, etc.)

## 1. Resumen general

- **Tipo:** pipeline manual de release / build / despliegue.
- **Uso:** para qué se ejecuta y quién lo dispara.
- **Tecnología:** lenguaje, framework, build tool, herramientas externas (Docker, Artifactory, SonarQube, etc.).
- **Flujo de trabajo:**
  - **Stage 1:** descripción breve
  - **Stage 2:** descripción breve
  - **Stage N:** descripción breve

## 2. Flujo de trabajo

### 2.1 Nombre del Stage

Descripción de qué hace este stage, y si es condicional, bajo qué expresión `when` corre.

```groovy
// código relevante
```

### 2.2 Nombre del Stage

Descripción de qué hace este stage.

```groovy
// código relevante
```

### 2.N Nombre del Stage Final

Descripción de qué hace este stage (normalmente el post/notificación).

```groovy
// código relevante
```

## 3. Configuración requerida en cada repositorio

Para que el pipeline funcione, cada repositorio consumidor debe contar con lo siguiente:

- **Nombre del ajuste/task/plugin:** descripción de por qué es necesario.

```kotlin
// ejemplo de configuración
```

- **Estructura de directorios / convención de nombres:** si el pipeline asume rutas o nombres fijos, documentarlo aquí.

- **`Jenkinsfile`:** debe invocar la shared library y llamar a la función correspondiente.

```groovy
@Library('nombre-libreria') _

nombrefuncion()
```

## 4. Consideraciones finales

- **Alcance del pipeline:** qué hace y qué no hace, diferencias con pipelines similares del mismo proyecto.
- **Credenciales/recursos preconfigurados:** cuáles deben existir ya en Jenkins para que el pipeline corra.
- Para dudas, contactar al equipo DevOps.