# Argo CD — Arquitectura y operación GitOps

## 1. Arquitectura general

**Argo CD centralizado en OKD**, administrado mediante Operator y configurado declarativamente desde GitHub.

```
GitHub — roshkadev/devops
       │
       │ GitOps / GitHub App
       ▼
Argo CD — OKD (argocd)
       │
       ├── Dex → OpenShift OAuth → LDAP
       ├── AppProject → RBAC por grupos
       ├── Sealed Secrets → Credenciales
       │
       ├── OKD local
       │   ├── dnit-sid-dev
       │   └── finnekta-sudameris
       │
       └── Kubernetes PROD
           └── finnekta-integracion
```

|Componente|Valor|
|---|---|
|Argo CD|v3.4.4|
|Namespace|`argocd`|
|Operator namespace|`argocd-operator`|
|URL|[https://argocd.roshka.com](https://argocd.roshka.com/)|
|Repositorio principal|[https://github.com/roshkadev/devops](https://github.com/roshkadev/devops)|
|Root Application|`argocd-config`|
|Proyecto administrativo|`argocd-admin`|
|Autenticación usuarios|OpenShift OAuth + Dex|
|Autorización|Grupos LDAP + AppProject RBAC|
|Autenticación Git|GitHub App|
|Credenciales|Sealed Secrets|
|Sincronización|Manual por defecto|

## 2. Instalación y configuración de Argo CD

### Operator

Instalado mediante OLM:

```
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

Manifiestos originales:

```
Operators/argocd/
├── Namespace.yaml
├── operatorgroup.yaml
├── Subscription.yaml
└── dex-token-rbac.yaml

manifests/argocd/
├── namespace.yaml
└── argocd.yaml
```

`dex-token-rbac.yaml` permite al ServiceAccount del Operator solicitar tokens para Dex.

### Instancia Argo CD

Configuración principal del recurso `ArgoCD`:

```
spec:
  server:
    host: argocd.roshka.com
    insecure: true
    route:
      enabled: true
      tls:
        termination: edge
        insecureEdgeTerminationPolicy: Allow

  rbac:
    defaultPolicy: ""
    scopes: '[groups]'
    policy: |
      g, argocd, role:admin

  sso:
    provider: dex
    dex:
      openShiftOAuth: true

  extraConfig:
    oidc.tls.insecure.skip.verify: "true"
    resource.respectRBAC: "normal"

  webhookSecrets:
    github:
      webhookSecretRef:
        name: github-webhook-credentials
        key: token
```

**Interpretación rápida:**

|Parámetro|Función|
|---|---|
|`server.route.enabled`|Publicar Argo CD en OpenShift|
|`tls.termination: edge`|TLS terminado en la Route|
|`server.insecure`|HTTP entre Route y servidor Argo CD|
|`rbac.defaultPolicy: ""`|Sin rol global predeterminado|
|`rbac.scopes: '[groups]'`|Utilizar grupos SSO para RBAC|
|`g, argocd, role:admin`|Grupo `argocd` como administrador|
|`openShiftOAuth: true`|Autenticación con OpenShift|
|`resource.respectRBAC: normal`|Descubrimiento de recursos considerando permisos RBAC|
|`webhookSecrets`|Referencia al secreto de GitHub Webhook|

**Verificación:**

```
oc get argocd argocd -n argocd
oc get pods -n argocd
oc get route -n argocd
oc get csv -n argocd-operator
```

## 3. Estructura GitOps en GitHub

Repositorio: [roshkadev/devops](https://github.com/roshkadev/devops)

```
gitops/
├── argocd/
│   ├── bootstrap/
│   │   ├── admin-project.yaml
│   │   └── root-application.yaml
│   │
│   └── managed/
│       ├── credentials/
│       │   ├── GitHub App SealedSecret
│       │   └── Kubernetes PROD SealedSecret
│       │
│       └── projects/
│           ├── dnit-sid-dev/
│           ├── finnekta-sudameris/
│           └── finnekta-integracion/
│
└── infrastructure/
    └── <CLUSTER>/
        └── argocd-access/
            ├── namespace.yaml
            ├── serviceaccount.yaml
            ├── clusterrole.yaml
            └── rolebinding-<NAMESPACE>.yaml
```

`infrastructure/` representa la ubicación definida para versionar accesos administrativos; comprobar los archivos efectivos en Git. Los nombres mostrados en `bootstrap/` son referencias.

### Bootstrap

|Recurso|Función|
|---|---|
|`argocd-admin`|AppProject administrativo|
|`argocd-config`|Application raíz que administra la configuración GitOps|

Configuración de `argocd-config`:

```
source:
  repoURL: https://github.com/roshkadev/devops.git
  targetRevision: main
  path: gitops/argocd/managed
  directory:
    recurse: true

destination:
  server: https://kubernetes.default.svc
  namespace: argocd
```

**Regla:** `bootstrap/` se aplica inicialmente de manera administrativa; `managed/` se administra mediante `argocd-config`.

## 4. GitHub App y webhooks

### Autenticación de repositorios

GitHub App instalada en la organización `roshkadev`, autorizada para los repositorios utilizados por Argo CD.

Permisos:

- Contents: Read-only
    
- Metadata: Read-only
    
- Evento: Push
    

Las credenciales compartidas se almacenan en un Secret con:

```
metadata:
  name: github-app-repo-creds
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repo-creds
stringData:
  type: git
  url: https://github.com/roshkadev/
  githubAppID: "<APP_ID>"
  githubAppInstallationID: "<INSTALLATION_ID>"
  githubAppPrivateKey: "<PRIVATE_KEY>"
```

El Secret se versiona como **SealedSecret**.

Las Applications utilizan repositorios HTTPS:

```
repoURL: https://github.com/roshkadev/<REPO>.git
```

### Webhooks

```
GitHub App
    │ POST /api/webhook
    ▼
https://argocd.roshka.com
    │
    ▼
Nginx — Katniss
    │ TLS público: Let's Encrypt
    │ Filtrado de IPs
    ▼
192.168.16.167:80
    │
    ▼
OpenShift Route → argocd-server
```

|Elemento|Configuración|
|---|---|
|Endpoint|`https://argocd.roshka.com/api/webhook`|
|Nginx host|Katniss|
|Virtual host|`/etc/nginx/sites-available/argocd.roshka.com`|
|Configuración común|`/etc/nginx/sites-available/argocd-common-config`|
|Origen IPs GitHub|`https://api.github.com/meta` → `.hooks[]`|
|Secreto|`argocd/github-webhook-credentials`|
|Key|`token`|
|Argo CD|`spec.webhookSecrets.github.webhookSecretRef`|

`/api/webhook` admite las IPs autorizadas; `/` permanece restringido a las redes corporativas configuradas.

El webhook **actualiza la comparación Git/Kubernetes**; no ejecuta Sync si la Application es manual.

**Verificaciones:**

```
# Katniss
sudo nginx -t
sudo systemctl reload nginx

# OKD: referencia del webhook
oc get argocd argocd -n argocd \
  -o jsonpath='{.spec.webhookSecrets.github.webhookSecretRef.name}{"\n"}'
```

Recepción: GitHub App → **Advanced → Recent Deliveries**.

## 5. SSO, LDAP y RBAC de usuarios

### Flujo de autenticación

```
Usuario
  → Argo CD
  → Dex
  → OpenShift OAuth
  → LDAP
```

Los grupos LDAP se importan a OpenShift y se utilizan en los AppProjects.

Directorio administrativo en el helper:

```
/home/roshka/okd/manifests/ldap-sync/
└── sync-config.yaml
```

### Sincronización de grupos

**Grupo nuevo:**

```
oc adm groups sync \
  --type=ldap \
  'cn=<GRUPO>,ou=okd,ou=groups,dc=roshka,dc=com,dc=py' \
  --sync-config=sync-config.yaml \
  --confirm
```

**Actualizar grupos importados:**

```
oc adm groups sync \
  --type=openshift \
  --sync-config=sync-config.yaml \
  --confirm
```

Antes de aplicar, ejecutar sin `--confirm` para revisar el resultado.

**Verificar:**

```
oc get group <GRUPO> -o yaml
```

### RBAC por AppProject

```
spec:
  roles:
    - name: deployer
      groups:
        - <GRUPO_LDAP>
      policies:
        - p, proj:<PROYECTO>:deployer, applications, get, <PROYECTO>/*, allow
        - p, proj:<PROYECTO>:deployer, applications, sync, <PROYECTO>/*, allow
```

|Grupo / rol|Permisos|
|---|---|
|`argocd`|Administrador global|
|`<PROYECTO>:deployer`|Ver y sincronizar Applications del proyecto|
|Usuarios sin rol|Sin permisos globales predeterminados|

El grupo reconocido debe aparecer en Argo CD → **User Info**.

## 6. Conexión de clústeres Kubernetes

Argo CD accede a los clústeres externos mediante una ServiceAccount dedicada y un Secret de conexión declarativo.

### Kubernetes PROD — Inventario

|Elemento|Valor|
|---|---|
|Nombre de conexión|`kubernetes-prod`|
|API|`https://k8s-master.iservers.roshka.com:443`|
|Kubernetes|v1.30.2|
|Namespace administrativo|`argocd-management`|
|ServiceAccount|`argocd-manager`|
|ClusterRole|`argocd-manager`|
|RoleBinding|`finnekta-integracion/argocd-manager`|
|Namespace autorizado|`finnekta-integracion`|
|Secret en OKD|`argocd/kubernetes-prod`|
|Autenticación|ServiceAccount token|
|TLS|`insecure: false` + `caData`|
|Alcance global|`clusterResources: "false"`|

### Recursos en Kubernetes remoto

ServiceAccount:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: argocd-manager
  namespace: argocd-management
automountServiceAccountToken: false
```

RoleBinding:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: argocd-manager
  namespace: finnekta-integracion
subjects:
  - kind: ServiceAccount
    name: argocd-manager
    namespace: argocd-management
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-manager
```

El `ClusterRole` define los permisos; el `RoleBinding` los concede dentro del namespace autorizado. El manifiesto completo de permisos debe conservarse en `gitops/infrastructure/<CLUSTER>/argocd-access/`.

### Secret de registro en Argo CD

```
metadata:
  name: kubernetes-prod
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
stringData:
  name: kubernetes-prod
  server: https://k8s-master.iservers.roshka.com:443
  namespaces: finnekta-integracion
  clusterResources: "false"
  config: |
    {
      "bearerToken": "<TOKEN>",
      "tlsClientConfig": {
        "insecure": false,
        "caData": "<CA_BASE64>"
      }
    }
```

El Secret se cifra mediante `kubeseal` y se versiona en:

```
gitops/argocd/managed/credentials/<CLUSTER>-sealedsecret.yaml
```

**Flujo de registro:**

```
Kubernetes remoto
  → Namespace argocd-management
  → ServiceAccount
  → ClusterRole + RoleBinding
  → Token + CA
  → SealedSecret en OKD
  → GitHub devops
  → argocd-config Sync
  → Argo CD Settings → Clusters
```

**Comprobar RBAC en el clúster remoto:**

```
SA=system:serviceaccount:argocd-management:argocd-manager

kubectl auth can-i get deployments \
  -n finnekta-integracion --as="$SA"

kubectl auth can-i delete deployments \
  -n default --as="$SA"
```

## 7. Convención de AppProjects y Applications

Directorio por proyecto:

```
gitops/argocd/managed/projects/<PROYECTO>/
├── appproject.yaml
├── application.yaml
└── <OTRA-APPLICATION>.yaml
```

### AppProject

Define el perímetro autorizado:

```
spec:
  sourceRepos:
    - https://github.com/roshkadev/<REPO>.git
  destinations:
    - server: <API_CLUSTER>
      namespace: <NAMESPACE>
  clusterResourceBlacklist:
    - group: "*"
      kind: "*"
```

Los roles LDAP se incluyen en `spec.roles`. La lista de recursos namespaced permitidos se define según el proyecto.

### Application

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <APPLICATION>
  namespace: argocd
spec:
  project: <PROYECTO>
  source:
    repoURL: https://github.com/roshkadev/<REPO>.git
    targetRevision: <RAMA>
    path: <PATH>
  destination:
    server: <API_CLUSTER>
    namespace: <NAMESPACE>
  syncPolicy:
    syncOptions:
      - CreateNamespace=false
```

Para Helm se agrega `source.helm.releaseName`, conservando el nombre de release correspondiente.

**Convención:** Sync manual; namespace creado previamente.

### Proyectos registrados

|Proyecto|Destino|
|---|---|
|`dnit-sid-dev`|OKD / `dnit-sid-dev`|
|`finnekta-sudameris`|OKD / `finnekta-sudameris`|
|`finnekta-integracion`|Kubernetes PROD / `finnekta-integracion`|

Finnekta utiliza Applications separadas para Core y Front.

## 8. Flujo operativo general

### A. Incorporar un proyecto nuevo

1. Identificar namespace, clúster, repositorio y rama.
    
2. Crear o identificar grupo LDAP y sincronizarlo.
    
3. Autorizar el repositorio en la instalación de GitHub App.
    
4. Crear `AppProject` con destinos, recursos y RBAC.
    
5. Crear una o varias `Applications`.
    
6. Guardar en `gitops/argocd/managed/projects/<PROYECTO>/`.
    
7. Crear PR hacia `main` de `roshkadev/devops`.
    
8. Ejecutar Refresh → Diff → Sync de `argocd-config`.
    
9. Comprobar que las Applications aparecen.
    
10. Revisar su Diff y sincronizar workloads cuando corresponda.
    

### B. Incorporar un clúster nuevo

1. Obtener API Server, CA y namespaces.
    
2. Crear `argocd-management` y `argocd-manager`.
    
3. Configurar ClusterRole y RoleBindings.
    
4. Preparar credencial de autenticación.
    
5. Crear Secret de clúster en OKD y convertirlo a SealedSecret.
    
6. Versionar RBAC en `gitops/infrastructure/<CLUSTER>/argocd-access/`.
    
7. Versionar SealedSecret en `gitops/argocd/managed/credentials/`.
    
8. Aplicar RBAC en clúster remoto y sincronizar `argocd-config`.
    
9. Validar en Settings → Clusters.
    

### C. Cambio habitual de aplicación

```
Commit / Push en repo aplicación
             ↓
GitHub Webhook
             ↓
Nginx Katniss → Argo CD
             ↓
Refresh / Compare
             ↓
OutOfSync (si hay diferencias)
             ↓
Revisión del Diff
             ↓
Sync manual
             ↓
Aplicación en Kubernetes
```

**Dos sincronizaciones distintas:**

- `argocd-config`: actualiza configuración de proyectos, Applications y credenciales.
    
- `Application` del proyecto: aplica los manifiestos de la aplicación al clúster de destino.
    

## 9. Comandos de operación rápida

**Helper OKD:**

```
# Estado general
oc get applications -n argocd \
  -o custom-columns='NAME:.metadata.name,SYNC:.status.sync.status,HEALTH:.status.health.status'

# Proyectos
oc get appprojects -n argocd

# Root Application
oc get application argocd-config -n argocd

# Detalle de una Application
oc describe application <APPLICATION> -n argocd

# Grupo LDAP sincronizado
oc get group <GRUPO> -o yaml

# Credenciales declarativas
oc get sealedsecrets -n argocd

# Conexión remota
oc get secret kubernetes-prod -n argocd --show-labels

# Controlador
oc logs -n argocd argocd-application-controller-0 --since=15m

# Argo CD server
oc logs -n argocd deployment/argocd-server --since=15m
```

## 10. Documentación detallada

|Tema|Playbook|
|---|---|
|Instalación de Argo CD en OKD|[[Argo CD — Arquitectura y operación GitOps]] — Nota actual como resumen|
|GitHub App, Nginx y webhooks|[GitHub App y Webhooks](https://wiki.roshka.com/e/es/platform-engineering/cliente/interno/cluster-okd/playbooks/githubapp-argocd-playbook)|
|Alta de proyectos, LDAP y RBAC|[Crear nuevo proyecto Argo CD](https://wiki.roshka.com/es/platform-engineering/cliente/interno/cluster-okd/playbooks/crear-nuevo-proyecto-argocd)|
|Registro de clústeres externos|[Conexiones de clústeres](https://wiki.roshka.com/es/platform-engineering/cliente/interno/cluster-okd/playbooks/conexiones-cluster-argocd)|
|Backup de configuración del helper|[Backup config helper](https://wiki.roshka.com/es/platform-engineering/cliente/interno/cluster-okd/playbooks/backup-config-helper)|