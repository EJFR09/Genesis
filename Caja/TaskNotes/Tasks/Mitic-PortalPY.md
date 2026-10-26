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
  
  docker run -d --name ie-sustancial-core `
  --network ie-sustancial-core_default `
  -p 8080:8080 -p 8081:8081 `
  -e DB_IDENTIDAD_URL="jdbc:postgresql://postgres:5432/identidad_electronica" `
  -e DB_IDENTIDAD_USER=ie_local `
  -e DB_IDENTIDAD_PASSWORD=ie_local `
  -e MONGO_URL="mongodb://mongo:27017/identidad_electronica" `
  -e SII_SECURITY_URL=mock `
  -e SII_USERNAME=mock `
  -e SII_PASSWORD=mock `
  docker-dev.roshka.com/mitic/mitic-ie-sustancial-core:0.0.1-test

# Cosas a tener en cuenta
* Registry: docker-dev.roshka.com/mitic/
* Imagenes: 
	*  docker-dev.roshka.com/mitic/mitic-frc-gateway:0.0.1-test .
	*  docker-dev.roshka.com/mitic/mitic-ie-sustancial-core:0.0.1-test .
* Comandos usados:
	* docker build -f jenkins/Dockerfile.run -t docker-dev.roshka.com/mitic/mitic-frc-gateway:0.0.1-test .
	* docker build -f jenkins/Dockerfile.run -t docker-dev.roshka.com/mitic/mitic-ie-sustancial-core:0.0.1-test .
* En r