---
status: open
priority: normal
dateCreated: 2026-10-13T08:06:46.540-03:00
dateModified: 2026-10-13T08:07:07.768-03:00
tags:
  - task
---
### Notas
* Informe2N: https://wiki.roshka.com/es/platform-engineering/cliente/brosco/elk/informe/Infome-2026-ELK-Fase2
* Docu: https://wiki.roshka.com/es/platform-engineering/cliente/brosco/elk/Instructivo-Creacion-Logs-Dashboards-kibana
* ServerHml: ssh -p 6722 -l devops 10.69.0.20

### Tareas
-  Reunión 
	* [ ] Leer informe
	* [ ] Enviar correo de informe
	* [ ] Fecha limite? 
* **Auditoría SSH:** centralizar registros de acceso a servidores y generar alertas.
	* [x] Recuperar stack ELK
	* [x] Retirar agentes legacy
	* [ ] Conseguir conectividad desde aws a los servidores dev y hml
	* [ ] Generar convención para rsync a servidores
	* [ ] Crear alarmas en base a accesos luego de cierto horarios
	* [ ] Verifcar consumo
- **Métricas de infraestructura:** monitorizar CPU, RAM, disco y load en DEV y HML.
	- [x] Implementar Metcbeat
	- [x] Implementar script de verificacion de consumo 
	- [ ] Verificar consumo
- **Métricas de Tomcat/JVM:** monitorizar recursos y comportamiento de Tomcat.
	- [ ] Implementar Metricbeat
	- [ ] Verificar consumo
- **Análisis de consumo vs recursos**: proponer aumento de recursos
