# Playbook — Instalación y configuración de Argo CD en OKD

**Autor:** Elias Figueredo  
**Fecha:** 14/10/2026  
**Plataforma:** OKD / OpenShift  
**Componente:** Argo CD / Argo CD Operator  
**Entorno:** Clúster OKD actual  
**Estado:** Implementado, con verificaciones y mejoras pendientes

---

## 1. Introducción

### 1.1. Objetivo

Documentar el procedimiento de instalación y configuración de Argo CD en OKD, mediante un operador administrado por Operator Lifecycle Manager (OLM).

Incluye la creación de la instancia, exposición mediante OpenShift Route, autenticación con Dex/OpenShift OAuth y configuración de proyectos y aplicaciones GitOps.

**Resultado esperado:** una instancia funcional de Argo CD capaz de autenticar usuarios mediante OpenShift, conectarse a repositorios privados de GitHub y administrar recursos Kubernetes de forma declarativa.

### 1.2. Cuándo utilizarlo

- Para instalar Argo CD en un clúster OKD.
    
- Para reproducir la configuración implementada en otro entorno compatible.
    
- Para reconstruir o recuperar la plataforma de administración GitOps.
    
- Como referencia para agregar nuevos proyectos y aplicaciones.
    

## 2. Requisitos previos

|Requisito|Descripción / Valor|
|---|---|
|Plataforma|Clúster OKD operativo|
|Helper|Servidor administrativo con acceso al clúster|
|Herramientas|OpenShift CLI (`oc`) y cliente Git|
|Permisos|Administración de operadores, namespaces, RBAC y recursos Argo CD|
|Operador|Community Argo CD Operator|
|Namespace operador|`argocd-operator`|
|Namespace instancia|`argocd`|
|Catálogo|`community-operators`|
|Canal|`alpha`|
|CSV inicial declarado|`argocd-operator.v0.19.0`|
|Autenticación|OpenShift OAuth integrado con LDAP|
|Repositorio piloto|`git@github.com:roshkadev/dnit-sid-devops.git`|
|Namespace aplicación|`dnit-sid-dev`|
|Credenciales Git|SSH Deploy Key|
|Directorio de manifiestos|Repositorio declarativo de Argo CD|

**Nota:** validar la disponibilidad y compatibilidad del operador y de su CSV antes de reproducir esta instalación. La versión actualmente instalada no está confirmada únicamente por el manifiesto `Subscription`.

## 3. Procedimiento

Todos los comandos se ejecutan desde el **helper**, con una sesión `oc` autenticada contra el clúster OKD correspondiente y desde la raíz del repositorio declarativo.

### 3.1. Preparar la estructura de manifiestos

Organizar los archivos de instalación y configuración de la siguiente manera:

```
argocd/
├── Operators/
│   └── argocd/
│       ├── Namespace.yaml
│       ├── operatorgroup.yaml
│       ├── Subscription.yaml
│       └── dex-token-rbac.yaml
├── manifests/
│   └── argocd/
│       ├── namespace.yaml
│       └── argocd.yaml
└── projects/
    └── dnit-sid-dev/
        ├── appproject.yaml
        ├── application.yaml
        └── repo-secret.yaml
```

La estructura separa la instalación del operador, la configuración de la instancia y los proyectos GitOps.

**Validación inicial:**

```
oc whoami
oc config current-context
oc get nodes
```

Confirmar que el contexto corresponde al clúster donde se instalará Argo CD.

### 3.2. Instalar el operador Argo CD

**Objetivo:** instalar el controlador responsable de administrar la instancia Argo CD.

Crear los manifiestos en `Operators/argocd/`.

**Namespace.yaml**

Define el namespace `argocd-operator`, dedicado al operador.

**operatorgroup.yaml**

Crear un `OperatorGroup` denominado `argocd-operator`, dentro del mismo namespace, con `spec: {}`.

Esta configuración no declara `targetNamespaces`, por lo que debe verificarse el alcance de instalación admitido por el operador.

**Subscription.yaml**

Declarar la suscripción con la configuración utilizada:

```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: argocd-operator
  namespace: argocd-operator
spec:
  channel: alpha
  name: argocd-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
  installPlanApproval: Manual
  startingCSV: argocd-operator.v0.19.0
  config:
    env:
      - name: ARGOCD_CLUSTER_CONFIG_NAMESPACES
        value: "argocd"
```

La aprobación manual permite revisar los planes de instalación y actualización antes de ejecutarlos.

**Aplicación:**

```
oc apply -f Operators/argocd/Namespace.yaml
oc apply -f Operators/argocd/operatorgroup.yaml
oc apply -f Operators/argocd/Subscription.yaml
```

**Validación:**

```
oc get subscription -n argocd-operator
oc get installplan -n argocd-operator
oc get csv -n argocd-operator
oc get pods -n argocd-operator
```

Si existe un `InstallPlan` pendiente, revisar su contenido y aprobarlo desde OpenShift únicamente después de verificar la versión y los permisos solicitados.

**Resultado esperado:** operador instalado, CSV en estado `Succeeded` y controlador funcionando.

### 3.3. Crear el namespace de Argo CD

**Objetivo:** separar la instancia de Argo CD de los componentes de su operador.

En `manifests/argocd/namespace.yaml` declarar:

```
apiVersion: v1
kind: Namespace
metadata:
  name: argocd
```

**Aplicar:**

```
oc apply -f manifests/argocd/namespace.yaml
```

**Validar:**

```
oc get namespace argocd
```

**Resultado esperado:** namespace `argocd` en estado `Active`.

### 3.4. Configurar RBAC para Dex

**Objetivo:** permitir al operador solicitar el token del ServiceAccount utilizado por Dex.

En `Operators/argocd/dex-token-rbac.yaml` declarar:

- Un `Role` denominado `argocd-operator-dex-token` en `argocd`.
    
- Permiso `create` sobre `serviceaccounts/token`.
    
- Restricción `resourceNames` al ServiceAccount `argocd-argocd-dex-server`.
    
- Un `RoleBinding` que vincule ese permiso al ServiceAccount `argocd-operator-controller-manager` del namespace `argocd-operator`.
    

**Aplicar:**

```
oc apply -f Operators/argocd/dex-token-rbac.yaml
```

**Validar:**

```
oc get role argocd-operator-dex-token -n argocd

oc get rolebinding argocd-operator-dex-token -n argocd
```

Este permiso es específico para la solicitud de tokens; no concede acceso general a todos los Secrets del namespace.

### 3.5. Crear la instancia de Argo CD

**Objetivo:** desplegar la instancia administrada por el operador, con interfaz web y autenticación SSO.

En `manifests/argocd/argocd.yaml` declarar:

```
apiVersion: argoproj.io/v1beta1
kind: ArgoCD
metadata:
  name: argocd
  namespace: argocd
spec:
  server:
    route:
      enabled: true
  sso:
    provider: dex
    dex:
      openShiftOAuth: true
  extraConfig:
    oidc.tls.insecure.skip.verify: "true"
```

**Configuración realizada:**

- `server.route.enabled`: habilita la exposición de Argo CD mediante OpenShift Route.
    
- `sso.provider: dex`: configura Dex como proveedor SSO.
    
- `dex.openShiftOAuth: true`: integra Dex con OpenShift OAuth.
    
- `oidc.tls.insecure.skip.verify`: deshabilita la verificación TLS en el contexto OIDC configurado. Es una excepción existente que debe corregirse.
    

**Aplicar:**

```
oc apply -f manifests/argocd/argocd.yaml
```

**Validar:**

```
oc get argocd argocd -n argocd
oc get pods -n argocd
oc get svc -n argocd
oc get route -n argocd
```

**Resultado esperado:** componentes de Argo CD operativos y una Route disponible para acceder a la interfaz.

### 3.6. Validar la autenticación con OpenShift OAuth

**Objetivo:** permitir a los usuarios iniciar sesión en Argo CD mediante la identidad administrada por OpenShift.

El flujo configurado es:

```
Usuario
   ↓
Interfaz Argo CD
   ↓
Dex
   ↓
OpenShift OAuth
   ↓
Proveedor de identidad LDAP
```

La integración LDAP pertenece a la configuración de autenticación existente en OpenShift; no se configuró un conector LDAP directo en Dex.

**Acciones:**

1. Obtener la Route de Argo CD.
    
2. Acceder a la interfaz HTTPS desde el navegador.
    
3. Iniciar sesión utilizando OpenShift.
    
4. Confirmar la autenticación del usuario y revisar los permisos efectivos.
    

**Validación:**

```
oc get route -n argocd
oc get pods -n argocd
```

**Resultado esperado:** acceso exitoso mediante OpenShift OAuth.

**Nota:** la autenticación no equivale a autorización administrativa. Los roles y permisos de usuarios en Argo CD requieren una revisión RBAC independiente.

### 3.7. Configurar el primer AppProject

**Objetivo:** delimitar los repositorios, destinos y tipos de recursos que podrá administrar la aplicación.

Crear `projects/dnit-sid-dev/appproject.yaml`.

Configuración principal:

```
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: dnit-sid-dev
  namespace: argocd
spec:
  description: GitOps project for DNIT SID DEV

  sourceRepos:
    - git@github.com:roshkadev/dnit-sid-devops.git

  destinations:
    - server: https://kubernetes.default.svc
      namespace: dnit-sid-dev

  clusterResourceBlacklist:
    - group: "*"
      kind: "*"
```

Además, configurar `namespaceResourceWhitelist` con los siguientes recursos permitidos, tal como se definió en el manifiesto original:

|API Group|Recursos|
|---|---|
|Core (`""`)|ConfigMap, Service, ServiceAccount, PersistentVolumeClaim|
|`apps`|Deployment, StatefulSet, DaemonSet|
|`batch`|Job, CronJob|
|`networking.k8s.io`|Ingress|
|`route.openshift.io`|Route|
|`bitnami.com`|SealedSecret|

La lista blanca limita los tipos de recursos namespaced administrables, mientras que `clusterResourceBlacklist` bloquea recursos de alcance de clúster.

**Aplicar:**

```
oc apply -f projects/dnit-sid-dev/appproject.yaml
```

**Validar:**

```
oc get appproject dnit-sid-dev -n argocd
```

**Resultado esperado:** AppProject disponible con restricciones explícitas de repositorio, destino y recursos.

### 3.8. Configurar acceso al repositorio privado

**Objetivo:** permitir que Argo CD descargue los manifiestos GitOps desde GitHub.

Crear `projects/dnit-sid-dev/repo-secret.yaml` con un Secret Kubernetes de tipo `Opaque`.

Configuración estructural:

```
apiVersion: v1
kind: Secret
metadata:
  name: repo-dnit-sid-devops
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  type: git
  url: git@github.com:roshkadev/dnit-sid-devops.git
  sshPrivateKey: |
    <CLAVE_PRIVADA_SSH>
```

La clave debe corresponder a una Deploy Key autorizada sobre el repositorio GitHub.

**Seguridad:** no almacenar ni publicar este manifiesto con la clave privada en texto plano. Generar el Secret desde una fuente segura o utilizar una alternativa cifrada compatible con el procedimiento de recuperación.

**Aplicar**, únicamente desde una ubicación segura:

```
oc apply -f projects/dnit-sid-dev/repo-secret.yaml
```

**Validar sin mostrar el secreto:**

```
oc get secret repo-dnit-sid-devops -n argocd
```

Posteriormente, verificar en la interfaz de Argo CD que la conexión al repositorio sea exitosa.

### 3.9. Crear la primera Application

**Objetivo:** vincular el repositorio Git al namespace de destino del clúster OKD.

Crear `projects/dnit-sid-dev/application.yaml`:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: dnit-sid-dev
  namespace: argocd
spec:
  project: dnit-sid-dev

  source:
    repoURL: git@github.com:roshkadev/dnit-sid-devops.git
    targetRevision: main
    path: k8s
    directory:
      recurse: true
      exclude: 'secrets/plantillas/**'

  destination:
    server: https://kubernetes.default.svc
    namespace: dnit-sid-dev

  syncPolicy:
    syncOptions:
      - CreateNamespace=false
```

**Configuración utilizada:**

- Repositorio GitHub privado mediante SSH.
    
- Rama `main`.
    
- Manifiestos ubicados dentro de `k8s/`.
    
- Recorrido recursivo de los manifiestos.
    
- Exclusión de `secrets/plantillas/**`.
    
- Destino: namespace `dnit-sid-dev` del clúster local.
    
- Sincronización manual, sin `syncPolicy.automated`.
    
- Creación automática del namespace deshabilitada.
    

**Requisito:** el namespace `dnit-sid-dev` debe existir antes de sincronizar.

**Aplicar:**

```
oc apply -f projects/dnit-sid-dev/application.yaml
```

**Validar:**

```
oc get application dnit-sid-dev -n argocd

oc get application dnit-sid-dev -n argocd -o wide
```

**Resultado esperado:** Application visible en Argo CD, con acceso al repositorio y capacidad para comparar el estado deseado contra el estado del clúster.

### 3.10. Verificar la sincronización

**Objetivo:** comprobar el funcionamiento del primer proyecto GitOps.

Desde la interfaz web de Argo CD:

1. Abrir la Application `dnit-sid-dev`.
    
2. Verificar que Argo CD puede recuperar los manifiestos de Git.
    
3. Revisar el estado de sincronización y salud.
    
4. Confirmar que los recursos están permitidos por el AppProject.
    
5. Revisar las diferencias antes de realizar una sincronización manual.
    
6. Ejecutar la sincronización únicamente si los cambios esperados son correctos.
    

**Validación desde el helper:**

```
oc get application dnit-sid-dev -n argocd

oc get deployments -n dnit-sid-dev

oc get services -n dnit-sid-dev

oc get routes -n dnit-sid-dev
```

**Resultado esperado:** los recursos administrados por Argo CD corresponden a los manifiestos declarados en Git, sin errores de acceso, generación o permisos.

### 3.11. Validación final

- Operador instalado y CSV en estado `Succeeded`.
    
- Instancia Argo CD operativa.
    
- Route accesible mediante HTTPS.
    
- Autenticación con OpenShift OAuth funcional.
    
- RBAC de Dex aplicado correctamente.
    
- AppProject `dnit-sid-dev` creado.
    
- Repositorio GitHub conectado mediante SSH.
    
- Application `dnit-sid-dev` configurada.
    
- Manifiestos Git recuperados correctamente.
    
- Sincronización manual comprobada.
    
- Restricciones del AppProject verificadas.
    

### 3.12. Rollback y recuperación

Ante problemas durante la instalación o configuración:

1. **Subscription/operador:** revisar el InstallPlan, el CSV y los logs del controlador antes de modificar la instalación.
    
2. **Instancia Argo CD:** restaurar la versión funcional anterior de `argocd.yaml` y comprobar la reconciliación del operador.
    
3. **Dex/OAuth:** recuperar la configuración SSO anterior y validar nuevamente el inicio de sesión.
    
4. **AppProject/Application:** restaurar desde Git los manifiestos funcionales y revisar las diferencias antes de sincronizar.
    
5. **Repositorio Git:** mantener disponible la Deploy Key mientras se prueban mecanismos de autenticación alternativos.
    

No eliminar automáticamente el namespace `argocd`, el recurso ArgoCD ni los recursos de aplicación como mecanismo de rollback. Su eliminación puede afectar la disponibilidad, las credenciales y las cargas de trabajo.

## 4. Notas y consideraciones

### 4.1. Arquitectura implementada

```
OKD / OpenShift
│
├── argocd-operator
│   └── Argo CD Operator (OLM)
│
├── argocd
│   ├── Argo CD
│   ├── OpenShift Route
│   ├── Dex / OpenShift OAuth
│   ├── AppProject: dnit-sid-dev
│   ├── Application: dnit-sid-dev
│   └── SSH Repository Secret
│
└── dnit-sid-dev
    └── Recursos administrados por Argo CD
         ▲
         │
    GitHub: roshkadev/dnit-sid-devops
    Rama: main / Directorio: k8s
```

### 4.2. Consideraciones de seguridad

- **TLS/OIDC:** la instancia utiliza `oidc.tls.insecure.skip.verify: "true"`. Se debe investigar la cadena de certificados y establecer confianza TLS correctamente antes de retirar la excepción.
    
- **Operador:** validar el mantenimiento y compatibilidad del Community Argo CD Operator, su canal `alpha` y la versión efectiva instalada.
    
- **Credenciales:** proteger la clave SSH privada y evitar Secrets en texto plano dentro de Git.
    
- **RBAC:** revisar los permisos de usuarios y grupos autenticados desde OpenShift.
    
- **AppProject:** mantener restricciones por repositorio, clúster, namespace y tipos de recurso.
    
- **Sincronización:** mantener inicialmente sincronización manual para conservar el control de los despliegues.
    

### 4.3. Mejoras pendientes

|Prioridad|Tarea|Estado|
|---|---|---|
|1|Validar versiones y compatibilidad del operador|Pendiente|
|2|Corregir la verificación TLS de OIDC|Pendiente|
|3|Migrar credenciales SSH a GitHub App|En planificación/configuración|
|4|Crear repositorio central GitOps para Argo CD|Pendiente|
|5|Incorporar clústeres OKD adicionales|Pendiente|
|6|Agregar nuevos AppProjects y Applications|Pendiente|
|7|Evaluar ApplicationSet y políticas de sincronización|Pendiente|
|8|Definir monitoreo y recuperación operativa|Pendiente|

La gestión de secretos mediante Sealed Secrets ya forma parte del entorno, pero no se documenta aquí su instalación porque no se proporcionaron los manifiestos ni el procedimiento original.

### 4.4. Alcance de la documentación

Este playbook fue reconstruido mediante ingeniería inversa de los manifiestos existentes y del contexto técnico disponible.

Los archivos confirman la configuración declarada y el responsable de la plataforma confirmó el funcionamiento de la instancia. Sin embargo, no disponemos de registros históricos suficientes para afirmar el orden cronológico exacto de cada operación ni el resultado individual de todas las pruebas.

### 4.5. Referencias oficiales

- [Argo CD — Declarative Setup](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)
    
- [Argo CD — Projects](https://argo-cd.readthedocs.io/en/stable/user-guide/projects/)
    
- [Argo CD — Private Repositories](https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/)
    
- [Argo CD Operator — Documentation](https://argocd-operator.readthedocs.io/)
    
- [OpenShift — Operators y OLM](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/observability/operators/understanding-operators)
