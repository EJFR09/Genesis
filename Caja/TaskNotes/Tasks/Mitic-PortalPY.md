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