---
status: open
priority: normal
scheduled: 2026-10-22
dateCreated: 2026-10-22T19:38:19.657-03:00
dateModified: 2026-10-26T07:39:15.691-03:00
tags:
  - devops
  - task
---


# Cosas a tener en cuenta
* Registry: docker-dev.roshka.com/mitic/
* Imagenes: 
	*  docker-dev.roshka.com/mitic/mitic-frc-gateway:0.0.1-test .
	*  docker-dev.roshka.com/mitic/mitic-ie-sustancial-core:0.0.1-test .
* Comandos usados:
	* docker build -f jenkins/Dockerfile.run -t docker-dev.roshka.com/mitic/mitic-frc-gateway:0.0.1-test .
	* docker build -f jenkins/Dockerfile.run -t docker-dev.roshka.com/mitic/mitic-ie-sustancial-core:0.0.1-test .
* En repo core no tenian el ejetable de maven añadí 
* En mi local me dió error al compilar core, por zonas horarias, en especifico con java enviando `America/Buenos_Aires` a los tests de postgres, solución de compilación, usar `-Duser.timezone=UTC`, luego al utilizar el contenedor con `America/Asuncion`, no volvió a dar el error sin necesidad de agregarle -Duser.timezone=UTC

Cmabio de ubicación de .yaml
**Sí, esa sería la organización recomendada:** configuración base versionada dentro del JAR y valores propios de cada ambiente suministrados externamente.

1. **Mover la configuración base**
    
    De:
    
    ```
    core/config/application.yaml
    ```
    
    A:
    
    ```
    core/src/main/resources/application.yaml
    ```
    
    Retirar el archivo anterior para evitar que sobrescriba la configuración del JAR. Gradle incluirá el nuevo recurso automáticamente; el Dockerfile seguirá copiando únicamente el JAR.
    
2. **Parametrizar lo que cambia por ambiente**
    
    Conexiones, credenciales, API keys, URLs de servicios y claves OIDC. Las constantes funcionales pueden permanecer en YAML.
    
    Por ejemplo:
    
    ```
    michi:
      server-info:
        api_key: ${MICHI_AUTH_API_KEY}
        url: ${MICHI_AUTH_URL}
    ```
    
    Usar valores predeterminados solo cuando sean apropiados; evitar claves o contraseñas predeterminadas.
    
3. **Agregar la importación del `.env`**
    
    Conservando la importación existente:
    
    ```
    spring:
      config:
        import:
          - optional:classpath:application-common.yml
          - "optional:file:./.env[.properties]"
    ```
    
    Spring cargará el `.env` del directorio de trabajo y resolverá las interpolaciones.
    
4. **Crear `.env.example` y el `.env` personal**
    
    El ejemplo versionado documentaría todas las variables necesarias, sin secretos:
    
    ```
    MICHI_AUTH_API_KEY=
    MICHI_AUTH_URL=http://localhost:8080/michi-auth/
    ```
    
    Cada desarrollador lo copiaría y completaría:
    
    ```
    Copy-Item .env.example .env
    ```
    
    Esas variables son ejemplos propuestos; hay que incluir también las demás que requiera Core.
    
5. **Excluir los secretos**
    
    En `.gitignore`:
    
    ```
    .env
    .env.*
    !.env.example
    ```
    
    Excluir los `.env` personales también del contexto Docker. **Gitignore no controla qué entra en una imagen.**
    
6. **Ejecutar o desplegar**
    
    Para desarrollo, desde `core`:
    
    ```
    .\gradlew.bat bootRun
    ```
    
    En Docker, proporcionar los valores mediante `environment`, `env_file` o secretos montados. El `.env` personal no se incluye en la imagen.
    

Esto permite usar **el mismo JAR y la misma imagen en distintos ambientes**, cambiando únicamente la configuración externa. No modifiqué archivos.