---
status: open
priority: normal
scheduled:
dateCreated: 2026-10-12T17:48:23.502-03:00
dateModified: 2026-10-13T09:08:09.972-03:00
tags:
  - devops
  - task
timeEntries:
  - startTime: 2026-10-13T12:00:00.000Z
    endTime: 2026-10-13T13:00:00.000Z
    description: ""
---

### Tareas
* [ ] TLS
	* [ ] https://wiki.roshka.com/es/platform-engineering/cliente/interno/cluster-okd/playbooks/playbook-iserver-okd
* [ ] Migrar a kubernetes con helmchart o manifests
* [ ] Repo para las configuraciones de argoCD
	* [ ] Automatizar, un cron o script que haga el push de los archivos en un folder de devops
* [ ] Investigar cuantos recursos consume
# Informe breve — Implementación de Argo CD en OKD

Objetivo: Implementar Argo CD en OKD mediante un operador, habilitar autenticación con OpenShift OAuth y configurar proyectos y aplicaciones para gestionar despliegues mediante GitOps.

Este procedimiento reconstruye la implementación realizada a partir de los manifiestos existentes.

## 1. Instalación del operador Argo CD

Crear el directorio `Operators/argocd/` con los siguientes manifiestos:

|Archivo|Configuración|
|---|---|
|`Namespace.yaml`|Crear el namespace `argocd-operator`.|
|`operatorgroup.yaml`|Definir el `OperatorGroup` del operador.|
|`Subscription.yaml`|Configurar el operador desde `community-operators`, canal `alpha`, aprobación manual y versión inicial `argocd-operator.v0.19.0`.|
|`dex-token-rbac.yaml`|Autorizar al ServiceAccount del operador a solicitar el token del ServiceAccount de Dex en `argocd`.|

Acción: aplicar los manifiestos, aprobar el InstallPlan correspondiente y verificar que el operador quede instalado y funcionando.

## 2. Creación de la instancia Argo CD

Crear el directorio `manifests/argocd/`:

|Archivo|Configuración|
|---|---|
|`namespace.yaml`|Crear el namespace `argocd`.|
|`argocd.yaml`|Declarar el recurso `ArgoCD` con nombre `argocd`, habilitar la Route y configurar SSO mediante Dex y OpenShift OAuth.|

La configuración principal utilizada fue:

```
spec:
  server:
    route:
      enabled: true
  sso:
    provider: dex
    dex:
      openShiftOAuth: true
```

Acción: aplicar la instancia, verificar sus pods y comprobar el acceso a través de la Route generada por OpenShift.

Observación: la implementación actual también utiliza `oidc.tls.insecure.skip.verify: "true"`. Es una excepción de seguridad pendiente de revisión; no debería asumirse como requisito permanente.

## 3. Configuración de proyectos GitOps

Crear el directorio `projects/dnit-sid-dev/`, separando las configuraciones por proyecto o entorno.

|Archivo|Configuración|
|---|---|
|`appproject.yaml`|Crear el `AppProject`, definir repositorios autorizados, clúster y namespace de destino y recursos Kubernetes permitidos.|
|`application.yaml`|Crear la `Application` vinculada al proyecto, especificando repositorio, rama, directorio de manifiestos y destino del despliegue.|
|`repo-secret.yaml`|Configurar el acceso al repositorio privado mediante clave SSH de despliegue.|

En el caso implementado:

- Proyecto y aplicación: `dnit-sid-dev`.
    
- Repositorio: `roshkadev/dnit-sid-devops`.
    
- Rama: `main`.
    
- Ruta de manifiestos: `k8s/`.
    
- Destino: namespace `dnit-sid-dev` del clúster local.
    
- Sincronización: manual.
    
- Exclusión: `secrets/plantillas/**`.
    

Acción: aplicar el `AppProject`, registrar las credenciales del repositorio y crear la `Application`. Verificar que Argo CD acceda a Git y pueda comparar y sincronizar los recursos con OKD.

## 4. Validación de la implementación

Una vez aplicados los manifiestos, comprobar:

1. Que el operador y los componentes de Argo CD estén operativos.
    
2. Que la interfaz web sea accesible mediante la Route.
    
3. Que los usuarios puedan autenticarse con sus credenciales de OpenShift.
    
4. Que el repositorio Git aparezca conectado.
    
5. Que la Application `dnit-sid-dev` pueda sincronizar correctamente sus recursos.
    
6. Que las restricciones del `AppProject` se respeten.
    

## 5. Estructura declarativa final

```
argocd/
├── Operators/
│   └── argocd/
│       ├── Namespace.yaml
│       ├── operatorgroup.yaml
│       ├── Subscription.yaml
│       └── dex-token-rbac.yaml
│
├── manifests/
│   └── argocd/
│       ├── namespace.yaml
│       └── argocd.yaml
│
└── projects/
    └── dnit-sid-dev/
        ├── appproject.yaml
        ├── application.yaml
        └── repo-secret.yaml
```

## 6. Próximas tareas

|Prioridad|Tarea|
|---|---|
|1|Revisar versiones, compatibilidad y mantenimiento del operador.|
|2|Corregir la configuración TLS insegura de OIDC.|
|3|Migrar la autenticación SSH de repositorios a GitHub App.|
|4|Crear un repositorio GitOps para autogestionar Projects y Applications de Argo CD.|
|5|Registrar los clústeres OKD adicionales y configurar sus permisos.|
|6|Incorporar nuevos proyectos y aplicaciones.|
|7|Definir políticas de sincronización y evaluar ApplicationSet para despliegues multiclúster.|

Resultado actual: disponemos de una instancia funcional de Argo CD sobre OKD, administrada mediante operador, con acceso web, autenticación integrada con OpenShift y un primer proyecto y aplicación conectados a GitHub.

La siguiente etapa consiste en reforzar la seguridad y ampliar la arquitectura hacia una administración declarativa centralizada y multiclúster.

Referencia: Documentación oficial de configuración declarativa de Argo CD.
