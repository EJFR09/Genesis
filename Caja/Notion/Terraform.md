**Links**

- [https://youtu.be/_84CxYRv9Ik?si=ct_KdjIECnle6fTJ](https://youtu.be/_84CxYRv9Ik?si=ct_KdjIECnle6fTJ)

---

### 1. Introducción

- 1.1 ¿Qué es Terraform?
    
    Es una herramienta de Infraestructura como Código (IaC) que permite crear, gestionar y configurar infraestructura de forma declarativa. Su gran ventaja es que es multi-proveedor (AWS, Azure, GCP, etc).
    
- 1.2 Infraestructura como Código (IaC)
    
    IaC es el principio detrás de Terraform. La idea es tratar la infraestructura exactamente igual que el código de una aplicación
    
    - **Reproducibilidad** — el mismo código genera exactamente la misma infraestructura en cualquier cuenta o región
    - **Auditabilidad** — Git guarda quién cambió qué y cuándo
    - **Colaboración** — todo el equipo puede ver y proponer cambios a la infraestructura
    - **Automatización** — se puede integrar en pipelines de CI/CD para despliegues automáticos y tests
- 1.3 Casos de uso reales
    
    **Levantar entornos completos de forma consistente**  
    Un equipo de DevOps define en Terraform toda la infraestructura de producción: VPC, subnets, instancias EC2, base de datos RDS, load balancer, certificados SSL. Cualquier persona del equipo puede recrear ese entorno completo corriendo `terraform apply`.
    
    **Gestión de múltiples entornos**  
    Con el mismo código base y distintos archivos de variables podés tener un entorno de desarrollo, uno de QA y uno de producción, todos con la misma estructura pero con distintos tamaños de instancias, distintos nombres y distintas configuraciones.
    
    **Infraestructura efímera para testing**  
    Un pipeline de CI crea infraestructura temporal para correr tests de integración, y al finalizar corre `terraform destroy` para eliminarla. Pagás solo por el tiempo que la usás.
    
    **Onboarding de nuevos proyectos**  
    Cuando se arranca un proyecto nuevo, en lugar de configurar manualmente decenas de recursos en la consola, se corre el Terraform del equipo y en minutos tenés todo listo.
    
- 1.4 Instalación (Windows, Mac, Linux)
    
    **Windows**
    
    ```powershell
    winget install HashiCorp.Terraform
    ```
    
    **Mac**
    
    ```bash
    brew tap hashicorp/tap
    brew install hashicorp/tap/terraform
    ```
    
    **Linux (Ubuntu/Debian)**
    
    ```bash
    sudo apt update && sudo apt install terraform
    ```
    
    **Verificar instalación**
    
    ```powershell
    terraform --version
    ```
    

### 2. HCL — HashiCorp Configuration Language

- 2.1 Sintaxis básica
    
    HCL es el lenguaje con el que se escriben los archivos de Terraform. Todos los archivos tienen extensión `.tf`. La estructura general de cualquier bloque en HCL es siempre la misma:
    
    ```bash
    tipo_de_bloque "provider_tipo_recurso" "nombre_recurso" {
      argumento_1 = valor_1
      argumento_2 = valor_2
    }
    ```
    
- 2.2 Tipos de datos (string, number, bool, list, map, object)
    
    **String**
    
    ```bash
    nombre = "servidor-web"
    ```
    
    **Number**
    
    ```
    puerto   = 8080
    replicas = 3
    ```
    
    **Bool**
    
    ```
    habilitado = true
    publico    = false
    ```
    
    **List — lista ordenada de valores del mismo tip**
    
    ```
    zonas = ["us-east-1a", "us-east-1b", "us-east-1c"]
    ```
    
    **Map — pares clave/valor**
    
    ```
    tags = {
      ambiente = "produccion"
      equipo   = "devops"
      proyecto = "mi-app"
    }
    ```
    
    **Object — similar al map pero cada clave puede tener un tipo distinto**
    
    ```
    configuracion = {
      nombre     = "web"
      puerto     = 80
      habilitado = true
    }
    ```
    
    **Tuple — lista donde cada elemento puede ser de tipo distinto**
    
    ```
    mixto = ["texto", 42, true]
    ```
    
- 2.3 Expresiones e interpolación
    
    La interpolación te permite construir strings dinámicos combinando texto con valores de variables u otros recursos.
    
    **Sintaxis:** `"${expresion}"`
    
    ```
    variable "ambiente" {
      default = "dev"
    }
    
    variable "proyecto" {
      default = "mi-app"
    }
    
    locals {
      nombre_servidor = "${var.proyecto}-${var.ambiente}-server"
      # resultado: "mi-app-dev-server"
    }
    ```
    
    También podés referenciar atributos de otros recursos:
    
    ```
    resource "aws_instance" "servidor" {
      ami           = "ami-12345"
      instance_type = "t3.micro"
    }
    
    output "ip" {
      value = aws_instance.servidor.public_ip
      # accede al atributo public_ip del recurso creado
    }
    ```
    
    La sintaxis para referenciar un recurso siempre es:
    
    `tipo_recurso.nombre_recurso.atributo`
    
- 2.4 Funciones integradas útiles
    
    Terraform trae un conjunto de funciones que podés usar directamente en tus configuraciones. No podés crear funciones propias, solo usar las que provee HCL.
    
    Para probarlas sin tocar infraestructura real podés usar la consola interactiva:
    
    `terraform console`
    
    **Funciones de strings:**
    
    ```
    # Convertir a mayúsculas / minúsculas
    upper("hola")   # "HOLA"
    lower("HOLA")   # "hola"
    
    # Reemplazar texto
    replace("hola mundo", "mundo", "terraform")  # "hola terraform"
    
    # Partir un string
    split(",", "a,b,c")  # ["a", "b", "c"]
    
    # Unir una lista en string
    join("-", ["dev", "web", "01"])  # "dev-web-01"
    
    # Longitud de un string o lista
    length("hola")      # 4
    length(["a", "b"])  # 2
    ```
    
    **Funciones de archivos:**
    
    ```
    # Leer el contenido de un archivo
    file("./scripts/init.sh")
    
    # Codificar en base64 (muy usado para user_data en AWS)
    filebase64("./scripts/init.sh")
    ```
    
    **Funciones de listas:**
    
    ```
    # Obtener elemento por índice
    element(["a", "b", "c"], 1)  # "b"
    
    # Aplanar listas anidadas
    flatten([["a", "b"], ["c"]])  # ["a", "b", "c"]
    
    # Convertir lista a set (elimina duplicados)
    toset(["a", "b", "a"])  # {"a", "b"}
    ```
    
    **Funciones de mapas:**
    
    ```
    # Obtener todas las claves
    keys({ nombre = "web", puerto = 80 })    # ["nombre", "puerto"]
    
    # Obtener todos los valores
    values({ nombre = "web", puerto = 80 })  # ["web", 80]
    
    # Fusionar dos mapas
    merge({ a = 1 }, { b = 2 })             # { a = 1, b = 2 }
    ```
    
    **Funciones de codificación:**
    
    ```
    # Convertir a JSON
    jsonencode({ nombre = "web", puerto = 80 })
    # resultado: {"nombre":"web","puerto":80}
    
    # Parsear JSON
    jsondecode("{\"nombre\":\"web\"}")
    ```
    
- 2.5 El registro oficial: [registry.terraform.io](http://registry.terraform.io)
    
    [https://registry.terraform.io/browse/providers](https://registry.terraform.io/browse/providers)
    
    El registry es la referencia que vas a consultar constantemente mientras trabajás con Terraform. Tiene tres secciones principales:
    
    **Providers** — `registry.terraform.io/browse/providers`  
    Listado de todos los providers disponibles. Cada provider documenta exactamente qué recursos soporta y qué argumentos acepta cada uno. Antes de crear cualquier recurso, conviene consultar acá la documentación del recurso específico para saber qué argumentos son obligatorios y cuáles opcionales.
    
    **Modules** — `registry.terraform.io/browse/modules`  
    Módulos públicos creados por la comunidad y por los propios proveedores. Por ejemplo HashiCorp publica módulos oficiales de AWS para crear VPCs, clusters de EKS, etc. En lugar de escribir todo desde cero podés usar estos módulos como punto de partida.
    
    **Policies** — para Terraform Cloud/Enterprise, por ahora no es relevante.
    
    Un ejemplo de cómo leer la documentación del registry para el resource `aws_instance`:
    
    ```
    Proveedor: hashicorp/aws
    Recurso:   aws_instance
    
    Argumentos requeridos:
      - ami           (string)
      - instance_type (string)
    
    Argumentos opcionales:
      - key_name               (string)
      - vpc_security_group_ids (list of strings)
      - tags                   (map of strings)
      ...
    
    Atributos exportados:
      - id
      - public_ip
      - private_ip
      - public_dns
      ...
    ```
    
    La sección de _atributos exportados_ es especialmente importante porque son los valores que podés usar en outputs o para referenciar desde otros recursos.
    

### 3. Tipos de bloques

- 3.1 `terraform`
    
    Es el bloque de configuración global de Terraform. Define cosas como la versión mínima requerida de Terraform, los providers que se van a usar y dónde guardar el estado remoto.
    
    ```
    terraform {
      required_version = ">= 1.7.0"
    
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
      }
    
      backend "s3" {
        bucket = "mi-terraform-state"
        key    = "prod/terraform.tfstate"
        region = "us-east-1"
      }
    }
    ```
    
    El versionado de providers merece atención. Los operadores que podés usar son:
    
    |Operador|Significado|
    |---|---|
    |`= 5.0.0`|Exactamente esa versión|
    |`>= 5.0.0`|Esa versión o mayor|
    |`~> 5.0`|La 5.x más reciente, nunca la 6.0|
    |`>= 5.0, < 6.0`|Entre esos rangos|
    
    El más usado en la práctica es `~>` porque te da actualizaciones de parche y minor sin riesgo de romper cosas con un cambio de versión mayor.
    
- 3.2 `provider`
    
    Configura la conexión con el servicio externo. Cada provider tiene sus propios argumentos, que podés consultar en el registry.
    
    ```
    provider "aws" {
      region  = "us-east-1"
      profile = "mi-perfil"  # perfil de AWS CLI
    }
    ```
    
    Podés definir múltiples instancias del mismo provider usando `alias`, útil cuando trabajás con múltiples regiones o múltiples cuentas al mismo tiempo:
    
    ```
    provider "aws" {
      region = "us-east-1"
      alias  = "virginia"
    }
    
    provider "aws" {
      region = "eu-west-1"
      alias  = "irlanda"
    }
    
    # Para usarlo en un resource:
    resource "aws_instance" "servidor_irlanda" {
      provider      = aws.irlanda
      ami           = "ami-12345"
      instance_type = "t3.micro"
    }
    ```
    
- 3.3 `resource`
    
    El bloque más importante. Representa cualquier pieza de infraestructura que Terraform va a crear, gestionar y destruir.
    
    ```
    resource "aws_instance" "servidor_web" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t3.micro"
    
      tags = {
        Name = "servidor-web"
      }
    }
    ```
    
    La estructura siempre es:
    
    `resource "tipo_recurso" "nombre_local" { }`
    
    Para referenciar atributos de un resource desde otro lugar:
    
    ```
    resource "aws_instance" "servidor" {
      ami           = "ami-12345"
      instance_type = "t3.micro"
    }
    
    resource "aws_eip" "ip_elastica" {
      instance = aws_instance.servidor.id
      # referencia el ID del servidor creado arriba
    }
    ```
    
- 3.4 `variable`
    
    Declara las variables de entrada del módulo o configuración. Permite parametrizar el código para no tener valores hardcodeados.
    
    ```
    variable "ambiente" {
      type        = string
      description = "Ambiente de despliegue (dev, qa, prod)"
      default     = "dev"
    }
    
    variable "tipo_instancia" {
      type        = string
      description = "Tipo de instancia EC2"
      default     = "t3.micro"
    }
    
    variable "puerto" {
      type    = number
      default = 80
    }
    
    variable "habilitar_monitoreo" {
      type    = bool
      default = false
    }
    
    variable "zonas_disponibles" {
      type    = list(string)
      default = ["us-east-1a", "us-east-1b"]
    }
    
    variable "tags_comunes" {
      type = map(string)
      default = {
        equipo   = "devops"
        proyecto = "mi-app"
      }
    }
    ```
    
    Para usar una variable dentro del código se usa el prefijo `var.`:
    
    ```
    resource "aws_instance" "servidor" {
      ami           = "ami-12345"
      instance_type = var.tipo_instancia
    
      tags = merge(var.tags_comunes, {
        Name = "servidor-${var.ambiente}"
      })
    }
    ```
    
    También podés agregar validaciones para asegurarte de que el valor ingresado sea correcto:
    
    ```
    variable "ambiente" {
      type        = string
      description = "Ambiente de despliegue"
    
      validation {
        condition     = contains(["dev", "qa", "prod"], var.ambiente)
        error_message = "El ambiente debe ser dev, qa o prod."
      }
    }
    ```
    
- 3.5 `output`
    
    Expone valores resultantes de la infraestructura para que los puedas ver en la terminal o usar desde otros módulos.
    
    ```
    output "ip_publica" {
      description = "IP pública del servidor"
      value       = aws_instance.servidor.public_ip
    }
    
    output "dns_publico" {
      description = "DNS público del servidor"
      value       = aws_instance.servidor.public_dns
    }
    ```
    
    Después de un `terraform apply` los outputs aparecen automáticamente en la terminal. También los podés consultar en cualquier momento con:
    
    ```bash
    terraform output
    terraform output ip_publica  # solo uno específico
    ```
    
    Para marcar un output como sensible (no se muestra en la terminal):
    
    ```
    output "password_bd" {
      value     = aws_db_instance.base_datos.password
      sensitive = true
    }
    ```
    
- 3.6 `locals`
    
    Define valores calculados internamente dentro del módulo. La diferencia con `variable` es que los locals no los puede sobreescribir quien llama al módulo, son puramente internos.
    
    ```
    locals {
      nombre_completo = "${var.proyecto}-${var.ambiente}"
      es_produccion   = var.ambiente == "prod"
    
      tags_comunes = {
        Name        = local.nombre_completo
        Environment = var.ambiente
        ManagedBy   = "terraform"
        Owner       = "devops@empresa.com"
      }
    }
    ```
    
    Para usarlos se usa el prefijo `local.` (sin s):
    
    ```
    resource "aws_instance" "servidor" {
      ami           = "ami-12345"
      instance_type = local.es_produccion ? "t3.large" : "t3.micro"
    
      tags = local.tags_comunes
    }
    ```
    
    El operador `condicion ? valor_si_true : valor_si_false` es el operador ternario, muy usado en locals para cambiar valores según el ambiente.
    
- 3.7 `module`
    
    Llama a un conjunto de recursos empaquetados en una carpeta o en el registry público.
    
    ```
    # Módulo local
    module "servidor_web" {
      source = "./modulos/servidor"
    
      ambiente       = var.ambiente
      tipo_instancia = "t3.micro"
      nombre         = "web"
    }
    
    # Módulo público del registry
    module "vpc" {
      source  = "terraform-aws-modules/vpc/aws"
      version = "~> 5.0"
    
      name = "mi-vpc"
      cidr = "10.0.0.0/16"
    }
    ```
    
    Los outputs del módulo se acceden así:
    
    ```
    output "ip_servidor" {
      value = module.servidor_web.ip_publica
      # módulo.nombre_módulo.nombre_output
    }
    ```
    
    Cada vez que agregás un nuevo módulo hay que volver a correr `terraform init` para que Terraform lo descargue y registre.
    
- 3.8 `data`
    
    Consulta información de recursos que ya existen en el proveedor sin crearlos ni modificarlos. Es de solo lectura.
    
    ```
    # Buscar la AMI más reciente de Ubuntu
    data "aws_ami" "ubuntu" {
      most_recent = true
      owners      = ["099720109477"]  # Canonical
    
      filter {
        name   = "name"
        values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
      }
    }
    
    # Usar ese dato en un resource
    resource "aws_instance" "servidor" {
      ami           = data.aws_ami.ubuntu.id
      instance_type = "t3.micro"
    }
    ```
    
    Otros usos comunes del bloque `data`:
    
    ```
    # Obtener información de la cuenta AWS actual
    data "aws_caller_identity" "actual" {}
    
    output "account_id" {
      value = data.aws_caller_identity.actual.account_id
    }
    
    # Leer un secret de AWS Secrets Manager
    data "aws_secretsmanager_secret_version" "db_password" {
      secret_id = "mi-app/db-password"
    }
    ```
    

### 4. Comandos esenciales

- 4.1 `terraform init`
    
    Es siempre el primer comando que corrés en cualquier proyecto nuevo o cuando agregás un nuevo provider o módulo. Se encarga de preparar el directorio de trabajo.
    
    - Descarga los plugins de los providers declarados
    - Descarga los módulos externos referenciados
    - Configura el backend donde se va a guardar el estado
    - Crea la carpeta oculta `.terraform/` con todo lo descargado
    
    La carpeta `.terraform/` no se sube a Git, igual que `node_modules` en un proyecto Node. Se agrega al `.gitignore`
    
- 4.2 `terraform plan`
    
    Genera un plan de ejecución mostrando exactamente qué va a pasar si corrés `apply`. No modifica nada en la infraestructura real.
    
    `terraform plan`
    
    La salida usa símbolos para indicar qué acción se va a tomar sobre cada recurso:
    
    |Símbolo|Acción|
    |---|---|
    |`+`|Crear|
    |`-`|Destruir|
    |`~`|Modificar en sitio|
    |`-/+`|Destruir y volver a crear|
    
    Los valores que dicen `(known after apply)` son atributos que Terraform no puede saber hasta que el recurso exista, como el ID o la IP pública.
    
    **Guardar el plan en un archivo:**
    
    `terraform plan -out=mi-plan.tfplan`
    
    Esto es muy útil en pipelines de CI/CD: generás el plan, lo revisás, y después lo aplicás exactamente tal cual sin que nadie tenga que confirmar nada:
    
    `terraform apply mi-plan.tfplan`
    
- 4.3 `terraform apply`
    
    Aplica los cambios en la infraestructura real. Por defecto primero muestra el plan y pide confirmación.
    
    **Con archivo de plan previo:**
    
    ```bash
    terraform apply mi-plan.tfplan
    # No pide confirmación porque el plan ya fue revisado
    ```
    
    **Saltarse la confirmación manual:**
    
    `terraform apply -auto-approve`
    
    Útil en pipelines automatizados, pero nunca lo uses manualmente en producción sin revisar el plan primero.
    
    **Aplicar solo un recurso específico:**
    
    `terraform apply -target=aws_instance.servidor`
    
    Útil cuando querés crear o actualizar un solo recurso sin tocar el resto. Úsalo con cuidado porque puede dejar el estado inconsistente si hay dependencias.
    
- 4.4 `terraform destroy`
    
    Destruye toda la infraestructura gestionada por Terraform en el directorio actual. Al igual que `apply`, primero muestra el plan y pide confirmación.
    
    Muy útil para infraestructura efímera de testing. En un pipeline podés levantar todo con `apply`, correr los tests y destruir todo con `destroy -auto-approve` al final.
    
- 4.5 `terraform fmt` y `terraform validate`
    
    Dos comandos que conviene correr siempre antes de hacer commit.
    
    **`terraform fmt`** — formatea automáticamente todos los archivos `.tf` siguiendo las convenciones oficiales de HCL.
    
    ```bash
    terraform fmt             # formatea el directorio actual
    terraform fmt -recursive  # formatea también subdirectorios
    terraform fmt -check      # solo verifica sin modificar (útil en CI)
    ```
    
    **`terraform validate`** — verifica que la sintaxis y la configuración sean correctas sin conectarse a ningún proveedor. Detecta referencias a variables que no existen, tipos incorrectos o argumentos mal escritos.
    
    `terraform validate`
    
    El flujo recomendado antes de cualquier commit:
    
    ```bash
    terraform fmt
    terraform validate
    terraform plan
    ```
    
- 4.6 `terraform state` (list, show)
    
    Conjunto de subcomandos para inspeccionar y manipular el archivo de estado. En el día a día los más usados son `list` y `show`.
    
    **Listar todos los recursos gestionados:**
    
    `terraform state list`
    
    **Ver el detalle completo de un recurso:**
    
    `terraform state show aws_instance.servidor`
    
    **Mover un recurso en el estado (útil al refactorizar):**
    
    `terraform state mv aws_instance.servidor aws_instance.servidor_web`
    
    **Eliminar un recurso del estado sin destruirlo:**
    
    `terraform state rm aws_instance.servidor`
    
- 4.7 `terraform output`
    
    Muestra los valores de los outputs definidos en la configuración.
    
    `terraform output # todos los outputs terraform output ip_publica # uno específico terraform output -json # en formato JSON terraform output -raw ip_publica # solo el valor sin formato`
    
    El flag `-raw` es especialmente útil en scripts de shell porque devuelve el valor limpio sin comillas:
    
    `IP=$(terraform output -raw ip_publica) ssh ec2-user@$IP`
    
- 4.8 `terraform import`
    
    Trae a Terraform un recurso que ya existe en el proveedor pero que no fue creado por Terraform. Permite gestionar infraestructura legacy sin destruirla y recrearla.
    
    `terraform import <tipo_recurso>.<nombre_local> <id_del_recurso>`
    
    **Flujo básico:**
    
    1. Declarar el bloque vacío del recurso en el `.tf`
    2. Correr `terraform import`
    3. Revisar con `terraform state show`
    4. Completar el `.tf` con los atributos relevantes
    5. Correr `terraform plan` hasta llegar a `No changes`
    
    Ver capítulo 11 para el detalle completo.
    

### 5. Providers

- 5.1 ¿Qué es un provider?
    
    Un provider es un plugin que le permite a Terraform comunicarse con una API externa. Puede ser una nube pública como AWS, Azure o GCP, pero también servicios como GitHub, Datadog, Cloudflare, o incluso el sistema de archivos local.  
    Sin un provider Terraform no puede hacer nada, porque no sabe con quién hablar. El provider es el traductor entre tu código HCL y la API del servicio.
    
- 5.2 Configuración básica
    
    La configuración mínima de un provider es simplemente declararlo:
    
    `provider "aws" { region = "us-east-1" }`
    
    Pero cada provider tiene sus propios argumentos.
    
- 5.3 Versionado de providers
    
    El versionado es importante para garantizar que tu código funcione igual en todas las máquinas del equipo y en el futuro. Se declara en el bloque terraform.
    
    ```
    terraform {
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
        datadog = {
          source  = "DataDog/datadog"
          version = ">= 3.20.0"
        }
      }
    }
    ```
    
- 5.4 Providers más usados (AWS, Azure, GCP, local)
    
    **AWS — `hashicorp/aws`**
    
    El más popular. Da acceso a prácticamente todos los servicios de Amazon Web Services: EC2, S3, RDS, Lambda, VPC, IAM, Route53, y cientos más.
    
    ```
    terraform {
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
      }
    }
    
    provider "aws" {
      region = "us-east-1"
    }
    
    resource "aws_s3_bucket" "mi_bucket" {
      bucket = "mi-bucket-unico-2026"
    }
    ```
    
    Las credenciales de AWS las podés pasar de varias formas, en orden de preferencia:
    
    1. Variables de entorno (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
    2. Perfil de AWS CLI configurado localmente
    3. IAM Role (cuando Terraform corre dentro de AWS)

### 6. Variables

- 6.1 Declaración de variables
    
    Las variables se declaran con el bloque `variable` y se usan con el prefijo `var.` La declaración mínima es solo el nombre, pero se recomienda siempre agregar `type` y `description`.
    
    ```
    variable "ambiente" {
      type        = string
      description = "Ambiente de despliegue (dev, qa, prod)"
      default     = "dev"
    }
    ```
    
    Si la variable no tiene `default` es obligatoria y Terraform la pedirá interactivamente si no la encontró por otro medio.
    
- 6.2 Pasar variables por CLI, archivo y entorno
    
    Hay cuatro formas de pasarle un valor a una variable, en orden de precedencia de mayor a menor:
    
    **1. Flag `-var` en la línea de comandos**
    
    ```bash
    terraform apply -var="ambiente=prod" -var="puerto=8080"
    ```
    
    **2. Archivo `.tfvars` con flag `-var-file`**
    
    ```bash
    terraform apply -var-file="prod.tfvars"
    ```
    
    **3. Variables de entorno con prefijo `TF_VAR_`**
    
    ```bash
    export TF_VAR_ambiente="prod"
    export TF_VAR_puerto="8080"
    terraform apply
    ```
    
    Muy útil en pipelines de CI/CD para no exponer valores sensibles en el código.
    
    **4. Archivos cargados automáticamente**
    
    Terraform carga automáticamente sin necesidad de `-var-file` los archivos que se llamen exactamente:
    
    - `terraform.tfvars`
    - `terraform.tfvars.json`
    - Cualquier archivo terminado en `.auto.tfvars`
    
    ```
    mi-proyecto/
    ├── main.tf
    ├── variables.tf
    ├── terraform.tfvars        # cargado automáticamente
    └── produccion.auto.tfvars  # cargado automáticamente
    ```
    
    Si ninguna de estas formas provee un valor y la variable no tiene `default`, Terraform lo pide interactivamente:
    
    ```
    var.ambiente
      Ambiente de despliegue (dev, qa, prod)
    
      Enter a value:
    ```
    
- 6.3 `terraform.tfvars` y archivos `.tfvars` personalizados
    
    El archivo `terraform.tfvars` es donde centralizás los valores de las variables para un ambiente determinado. La estructura es simple:
    
    ```
    # terraform.tfvars
    ambiente       = "dev"
    tipo_instancia = "t3.micro"
    puerto         = 8080
    zonas          = ["us-east-1a", "us-east-1b"]
    
    tags = {
      equipo   = "devops"
      proyecto = "mi-app"
    }
    ```
    
    Para múltiples entornos lo más limpio es tener un archivo por ambiente:
    
    ```
    mi-proyecto/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── dev.tfvars
    ├── qa.tfvars
    └── prod.tfvars
    ```
    
- 6.4 Variables sensibles
    
    Algunas variables contienen información sensible como passwords, tokens o claves de API. Para esas existe el argumento `sensitive = true`, que oculta el valor en toda la salida de la terminal.
    
    ```
    variable "db_password" {
      type      = string
      sensitive = true
    }
    ```
    
    Aunque el valor se oculta en la terminal, **sí queda guardado en el `tfstate` en texto plano**. Por eso es crucial proteger el archivo de estado. La forma más segura de pasarlas es via variable de entorno:
    
    ```bash
    export TF_VAR_db_password="mi-password-secreto"
    ```
    

### 7. Outputs y Locals

- 7.1 ¿Para qué sirven los outputs?
    
    Los outputs son la forma que tiene Terraform de exponer información resultante de la infraestructura creada. Tienen tres usos principales:
    
    **1. Ver valores en la terminal después de un apply**
    
    Sin outputs tendrías que entrar a la consola de AWS a buscar la IP de tu servidor. Con outputs Terraform te la muestra directamente al terminar:
    
    ```
    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
    
    Outputs:
    
    ip_publica  = "54.123.45.67"
    dns_publico = "ec2-54-123-45-67.compute-1.amazonaws.com"
    ```
    
    **2. Compartir valores entre módulos**
    
    Cuando un módulo necesita usar un valor creado por otro módulo, lo hace a través de outputs. Es la única forma de pasar información de un módulo a otro.
    
    **3. Usar valores en scripts o pipelines**
    
    Con `terraform output -raw` podés capturar un valor y usarlo en un script de shell, en un pipeline de CI/CD o en cualquier otro proceso automatizado.
    
- 7.2 Outputs entre módulos
    
    Este es el uso más importante de los outputs en proyectos reales. Cuando tenés tu infraestructura separada en módulos, la única forma de que un módulo use información de otro es a través de outputs.
    
    **Estructura del proyecto:**
    
    ```
    mi-proyecto/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── modulos/
        ├── red/
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf      # expone el ID de la VPC y subnets
        └── servidor/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf      # expone la IP del servidor
    ```
    
- 7.3 Locals: variables internas calculadas
    
    Los locals son valores que calculás una sola vez y reutilizás en múltiples lugares dentro del mismo módulo. La diferencia con las variables es que los locals no los puede sobreescribir quien llama al módulo, son completamente internos. Se usan con el prefijo `local.` (sin s).
    
    **Cuándo usar locals en lugar de variables:**
    
    - Cuando el valor se deriva de otras variables o recursos
    - Cuando querés evitar repetir la misma expresión en múltiples lugares
    - Cuando querés nombrar un valor complejo para que el código sea más legible
    
    ```
    locals {
      nombre_base    = "${var.proyecto}-${var.ambiente}"
      es_produccion  = var.ambiente == "prod"
      tipo_instancia = local.es_produccion ? "t3.large" : "t3.micro"
    
      tags_comunes = {
        Proyecto  = var.proyecto
        Ambiente  = var.ambiente
        ManagedBy = "terraform"
      }
    }
    ```
    
- 7.4 Outputs sensibles
    
    Al igual que las variables, los outputs pueden marcarse como sensibles para que no se muestren en la terminal. Si un output referencia una variable o recurso sensible, Terraform te obliga a marcarlo también.
    
    ```
    output "db_password" {
      description = "Password de la base de datos"
      value       = aws_db_instance.base_datos.password
      sensitive   = true
    }
    ```
    

### 8. Organización del código

- 8.1 Estructura de archivos recomendada
    
    Terraform lee todos los archivos `.tf` del directorio como si fueran uno solo, por lo que técnicamente podrías poner todo en un único `main.tf` y funcionaría. Sin embargo a medida que el proyecto crece eso se vuelve imposible de mantener.
    
    La convención de la industria es separar el código por responsabilidad. El conjunto mínimo recomendado para cualquier proyecto es:
    
    ```
    mi-proyecto/
    ├── main.tf           # recursos principales
    ├── variables.tf      # declaración de todas las variables
    ├── outputs.tf        # todos los outputs
    ├── providers.tf      # configuración de providers
    ├── locals.tf         # locals (si hay muchos)
    ├── versions.tf       # bloque terraform con versiones
    ├── terraform.tfvars  # valores por defecto de las variables
    └── README.md         # documentación del módulo
    ```
    
    No es una regla estricta de Terraform, es una convención que adopta casi toda la industria y que vas a ver en prácticamente todos los proyectos open source.
    
- 8.2 Estructura de archivos para proyecto grande
    
    Cuando el proyecto crece `main.tf` se divide en archivos por funcionalidad:
    
    `mi-proyecto/ ├── versions.tf ├── providers.tf ├── variables.tf ├── locals.tf ├── outputs.tf ├── network.tf # VPC, subnets, route tables ├── security.tf # security groups, IAM roles ├── compute.tf # instancias EC2, auto scaling ├── database.tf # RDS, ElastiCache ├── storage.tf # S3 buckets ├── dns.tf # Route53 ├── terraform.tfvars └── README.md`
    
- 8.3 Múltiples entornos con `var-file`
    
    La forma más simple de manejar múltiples entornos es tener un archivo `.tfvars` por ambiente y aplicar el que corresponde.
    
    **Estructura:**
    
    ```
    mi-proyecto/
    ├── versions.tf
    ├── providers.tf
    ├── variables.tf
    ├── locals.tf
    ├── main.tf
    ├── outputs.tf
    ├── envs/
    │   ├── dev.tfvars
    │   ├── qa.tfvars
    │   └── prod.tfvars
    └── README.md
    ```
    

### 9. Módulos

- 9.1 ¿Qué es un módulo?
    
    Un módulo es un conjunto de archivos `.tf` agrupados en una carpeta que representan una unidad reutilizable de infraestructura. Es el mecanismo de Terraform para empaquetar y reutilizar configuraciones.
    
    Todo proyecto de Terraform tiene al menos un módulo: el **módulo raíz**, que es el directorio donde corrés los comandos de Terraform. Cuando empezás a crear subcarpetas con archivos `.tf` y los llamás desde el módulo raíz, esas subcarpetas son **módulos hijos**.
    
    La idea es la misma que una función en programación: escribís la lógica una vez, la parametrizás con inputs, y la reutilizás todas las veces que necesitás con distintos valores.
    
    ```
    Sin módulos:              Con módulos:
    
    main.tf (500 líneas)      main.tf (50 líneas)
                              modulos/
                              ├── red/
                              ├── servidor/
                              └── base-datos/
    ```
    
- 9.2 Crear un módulo local
    
    Un módulo es simplemente una carpeta con archivos `.tf`. La estructura mínima recomendada es:
    
    ```
    modulos/servidor/
    ├── main.tf       # recursos que crea el módulo
    ├── variables.tf  # inputs del módulo
    └── outputs.tf    # valores que expone hacia afuera
    ```
    
    Desde el módulo raíz llamás al módulo con el bloque `module` y pasás los valores de las variables:
    
    **`main.tf` (raíz)**
    
    ```
    module "servidor_dev" {
      source = "./modulos/servidor"
    
      proyecto       = "mi-app"
      ambiente       = "dev"
      ami            = "ami-0c55b159cbfafe1f0"
      tipo_instancia = "t3.micro"
      puerto_app     = 80
    }
    
    module "servidor_prod" {
      source = "./modulos/servidor"
    
      proyecto       = "mi-app"
      ambiente       = "prod"
      ami            = "ami-0c55b159cbfafe1f0"
      tipo_instancia = "t3.large"
      puerto_app     = 80
    }
    ```
    
    Con el mismo módulo creaste dos servidores completamente distintos con una configuración diferente cada uno. Si necesitás cambiar algo en la lógica del servidor lo cambiás en un solo lugar y afecta a todos los ambientes.
    
    Cada vez que agregás un módulo nuevo hay que correr `terraform init`:
    
    `terraform init`
    
- 9.3 Usar módulos públicos del registry
    
    El registry de Terraform tiene miles de módulos públicos listos para usar, muchos mantenidos por HashiCorp o por los propios proveedores. En lugar de escribir desde cero la configuración de una VPC compleja podés usar el módulo oficial:
    
    ```
    module "vpc" {
      source  = "terraform-aws-modules/vpc/aws"
      version = "~> 5.0"
    
      name = "mi-vpc"
      cidr = "10.0.0.0/16"
    
      azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
      private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
      public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
    
      enable_nat_gateway = true
      enable_vpn_gateway = false
    
      tags = {
        Proyecto  = "mi-app"
        ManagedBy = "terraform"
      }
    }
    ```
    
    La sintaxis del source para módulos públicos es:
    
    `"namespace/nombre-modulo/provider"`
    
- 9.4 Inputs y outputs de módulos
    
    Los inputs de un módulo son simplemente sus variables. Los outputs son lo que expone hacia afuera. La comunicación entre módulos siempre fluye así:
    
    ```
    módulo raíz
    │
    ├── pasa variables como inputs ──────────────> módulo hijo
    │                                                   │
    └── recibe outputs ◄────────────────────────────────┘
    ```
    
- 9.5 Buenas prácticas con módulos
    
    **Mantener los módulos pequeños y enfocados**
    
    Cada módulo debería tener una responsabilidad clara. Un módulo `servidor` crea un servidor. Un módulo `red` crea la red. No un módulo `todo` que crea todo junto.
    
    ```
    ✅ módulo servidor   — EC2 + Security Group + Key Pair
    ✅ módulo red        — VPC + Subnets + Route Tables + IGW
    ✅ módulo base-datos — RDS + Security Group + Subnet Group
    
    ❌ módulo infraestructura-completa — todo lo anterior junto
    ```
    
    **Siempre versionar los módulos externos**
    
    ```
    # ❌ Sin versión — puede romperse con actualizaciones
    module "vpc" {
      source = "terraform-aws-modules/vpc/aws"
    }
    
    # ✅ Con versión fija
    module "vpc" {
      source  = "terraform-aws-modules/vpc/aws"
      version = "~> 5.0"
    }
    ```
    
    **Documentar inputs y outputs**
    
    Siempre agregar `description` a todas las variables y outputs del módulo. Cuando alguien más use el módulo va a necesitar saber qué hace cada input.
    
    **Proveer defaults razonables**
    
    Las variables que tienen un valor sensato por defecto deberían tenerlo declarado. Solo las variables verdaderamente requeridas deberían no tener default:
    
    ```
    # ✅ Requerida sin default — el caller debe proveerla
    variable "proyecto" {
      type        = string
      description = "Nombre del proyecto"
    }
    
    # ✅ Opcional con default razonable
    variable "tipo_instancia" {
      type        = string
      description = "Tipo de instancia EC2"
      default     = "t3.micro"
    }
    ```
    
    **No hardcodear valores específicos del ambiente**
    
    Los módulos deben ser agnósticos al ambiente. Los valores que cambian por ambiente entran como variables:
    
    ```
    # ❌ Hardcodeado — no reutilizable
    resource "aws_instance" "servidor" {
      instance_type = "t3.large"  # siempre prod size
    }
    
    # ✅ Parametrizado — reutilizable en cualquier ambiente
    resource "aws_instance" "servidor" {
      instance_type = var.tipo_instancia
    }
    ```
    
    **Estructura de un proyecto real con módulos:**
    
    ```
    mi-proyecto/
    ├── main.tf           # llama a los módulos
    ├── variables.tf
    ├── outputs.tf
    ├── providers.tf
    ├── versions.tf
    ├── locals.tf
    ├── envs/
    │   ├── dev.tfvars
    │   ├── qa.tfvars
    │   └── prod.tfvars
    └── modulos/
        ├── red/
        │   ├── main.tf
        │   ├── variables.tf
        │   ├── outputs.tf
        │   └── README.md
        ├── servidor/
        │   ├── main.tf
        │   ├── variables.tf
        │   ├── outputs.tf
        │   └── README.md
        └── base-datos/
            ├── main.tf
            ├── variables.tf
            ├── outputs.tf
            └── README.md
    ```
    

### 10. Estado (tfstate)

- 10.1 ¿Qué es el archivo de estado?
    
    El archivo de estado (`terraform.tfstate`) es el mecanismo que usa Terraform para saber qué infraestructura está gestionando. Es un archivo JSON que mapea cada bloque `resource` de tu código con el recurso real que existe en el proveedor.  
    Cada vez que corrés `terraform apply` o `terraform destroy` Terraform actualiza este archivo para reflejar el estado actual de la infraestructura real.
    
    El flujo de trabajo de Terraform siempre involucra tres elementos:
    
    ```
    Código .tf          Estado tfstate        Infraestructura real
    (lo que querés)     (lo que Terraform     (lo que existe)
                         cree que existe)
    ```
    
    Cuando corrés `terraform plan` Terraform hace lo siguiente:
    
    1. Lee tu código `.tf` — el estado deseado
    2. Lee el `tfstate` — el estado conocido
    3. Consulta al proveedor — el estado real actual
    4. Calcula la diferencia y muestra qué hay que hacer
    
    Esto significa que si alguien modifica un recurso manualmente desde la consola de AWS sin pasar por Terraform, el `tfstate` va a quedar desincronizado con la realidad. Para detectar eso existe el comando:
    
    `terraform refresh`
    
    Pero hoy en dia ya esta implementando en el comando plan.
    
- 10.2 Por qué no guardarlo en Git
    
    - **Contiene información sensible en texto plano** — passwords, tokens y claves privadas quedan en el archivo sin encriptar. Cualquier persona con acceso al repo puede leerlos.
    - **Genera conflictos constantemente** — si dos personas corren `terraform apply` al mismo tiempo cada una genera su propia versión del estado y Git no puede resolver el conflicto automáticamente.
    - **No tiene locking** — sin un mecanismo de bloqueo, dos personas pueden modificar el estado simultáneamente y corromperlo.
    
    Por esto el `tfstate` va en el `.gitignore` desde el primer día.
    
- 10.3 Remote state en S3
    
    La solución estándar en AWS es guardar el estado en un bucket de S3 con locking a través de DynamoDB. Esta combinación resuelve los tres problemas del punto anterior:
    
    - **S3** — almacena el archivo encriptado en reposo, con versionado habilitado para poder recuperar estados anteriores
    - **DynamoDB** — provee el mecanismo de locking para que solo un proceso pueda modificar el estado a la vez
    - **Acceso compartido** — todo el equipo lee y escribe en el mismo lugar
    
    Ver 10.4 para la configuración completa.
    
- 10.4 State locking con DynamoDB
    
    **Paso 1 — Crear el bucket S3 y la tabla DynamoDB**
    
    Lo ideal es crear estos recursos con Terraform también, pero en un directorio separado de bootstrap que solo se aplica una vez:
    
    ```powershell
    # bootstrap/main.tf
    
    resource "aws_s3_bucket" "terraform_state" {
      bucket = "mi-empresa-terraform-state"
    
      lifecycle {
        prevent_destroy = true  # nunca destruir accidentalmente
      }
    
      tags = {
        Name      = "terraform-state"
        ManagedBy = "terraform"
      }
    }
    
    # Habilitar versionado — permite recuperar versiones anteriores del estado
    resource "aws_s3_bucket_versioning" "terraform_state" {
      bucket = aws_s3_bucket.terraform_state.id
    
      versioning_configuration {
        status = "Enabled"
      }
    }
    
    # Encriptar el estado en reposo
    resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
      bucket = aws_s3_bucket.terraform_state.id
    
      rule {
        apply_server_side_encryption_by_default {
          sse_algorithm = "AES256"
        }
      }
    }
    
    # Bloquear acceso público al bucket
    resource "aws_s3_bucket_public_access_block" "terraform_state" {
      bucket = aws_s3_bucket.terraform_state.id
    
      block_public_acls       = true
      block_public_policy     = true
      ignore_public_acls      = true
      restrict_public_buckets = true
    }
    
    # Tabla DynamoDB para el locking
    resource "aws_dynamodb_table" "terraform_locks" {
      name         = "terraform-locks"
      billing_mode = "PAY_PER_REQUEST"
      hash_key     = "LockID"
    
      attribute {
        name = "LockID"
        type = "S"
      }
    
      lifecycle {
        prevent_destroy = true
      }
    
      tags = {
        Name      = "terraform-locks"
        ManagedBy = "terraform"
      }
    }
    ```
    
    **Paso 2 — Configurar el backend en tu proyecto**
    
    ```powershell
    # versions.tf
    
    terraform {
      required_version = ">= 1.7.0"
    
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
      }
    
      backend "s3" {
        bucket         = "mi-empresa-terraform-state"
        key            = "mi-proyecto/dev/terraform.tfstate"
        region         = "us-east-1"
        dynamodb_table = "terraform-locks"
        encrypt        = true
      }
    }
    ```
    
    La propiedad `key` es la ruta dentro del bucket donde se guarda el archivo. La convención es usar una estructura que identifique el proyecto y el ambiente:
    
    ```powershell
    mi-proyecto/dev/terraform.tfstate
    mi-proyecto/qa/terraform.tfstate
    mi-proyecto/prod/terraform.tfstate
    otro-proyecto/dev/terraform.tfstate
    ```
    
    **Paso 3 — Correr `terraform init`**
    
    Al cambiar el backend hay que reinicializar. Terraform va a preguntar si querés migrar el estado local al remoto:
    
    ```powershell
    terraform init
    
    # Initializing the backend...
    # Do you want to copy existing state to the new backend?
    # Enter a value: yes
    
    # Successfully configured the backend "s3"!
    ```
    
    A partir de ese momento el `terraform.tfstate` local desaparece y todo se guarda en S3.
    
    Cuando el backend es S3 con DynamoDB, Terraform automáticamente bloquea el estado antes de cualquier operación que lo modifique (`apply`, `destroy`, `import`). Si otra persona intenta correr una operación al mismo tiempo ve este error:
    
- 10.5 Comandos de gestión del estado
    
    `terraform state list` — lista todos los recursos gestionados
    
    `terraform state show <recurso>` — detalle completo de un recurso
    
    `terraform state mv <origen> <destino>` — renombra un recurso en el estado
    
    `terraform state rm <recurso>` — elimina del estado sin destruir el recurso real
    
    `terraform state pull` — descarga el estado remoto y lo muestra en la terminal
    
    `terraform force-unlock <lock-id>` — libera un lock trabado manualmente
    

### 11. `terraform import`

- 11.1 ¿Cuándo usar import?
    
    Resuelve un problema muy común: tenés infraestructura que ya existe (creada manualmente o por otra herramienta) y querés que Terraform la gestione sin destruirla y recrearla.
    
    **Casos típicos:**
    
    - **Migración a Terraform** — una empresa lleva años creando infraestructura a mano y decide adoptar Terraform
    - **Recurso creado por emergencia** — alguien creó un servidor manualmente a las 3am durante un incidente y hay que incorporarlo al código
    - **Estado perdido** — el `tfstate` se corrompió o se perdió pero la infraestructura real sigue existiendo
- 11.2 Cómo funciona internamente
    
    Cuando corrés `terraform import` pasan tres cosas:
    
    1. Terraform llama a la API del proveedor y obtiene todos los atributos del recurso real
    2. Guarda esa información en el `tfstate`
    3. No modifica el recurso real ni genera código `.tf`
    
    **Lo que NO hace:**
    
    - No genera el código `.tf` automáticamente — eso lo escribís vos
    - No valida que tu código coincida con el recurso real
    - No importa dependencias — cada recurso hay que importarlo por separado
- 11.3 Sintaxis y flujo paso a paso
    
    **Sintaxis del comando:**
    
    `terraform import <tipo_recurso>.<nombre_local> <id_del_recurso>`
    
    El `<id>` varía según el tipo de recurso. Siempre está documentado en el Terraform Registry, sección **Import** al final de cada recurso.
    
    |Recurso|ID para import|
    |---|---|
    |`local_file`|path del archivo|
    |`aws_instance`|instance ID (`i-0a1b2c3d4e5f`)|
    |`aws_s3_bucket`|nombre del bucket|
    |`aws_security_group`|security group ID (`sg-12345`)|
    |`aws_iam_role`|nombre del rol|
    
    **Flujo completo:**
    
    **1.** Escribir el bloque vacío en el `.tf`
    
    ```
    resource "aws_instance" "servidor" {}
    ```
    
    **2.** Correr el import
    
    ```bash
    terraform import aws_instance.servidor i-0a1b2c3d4e5f
    ```
    
    **3.** Ver qué importó Terraform
    
    `terraform state show aws_instance.servidor`
    
    **4.** Completar el `.tf` con los atributos relevantes
    
    ```
    resource "aws_instance" "servidor" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t3.micro"
      tags = {
        Name = "servidor-web"
      }
    }
    ```
    
    **5.** Verificar que no haya diferencias
    
    ```bash
    terraform plan
    # Objetivo: "No changes"
    ```
    
- 11.4 Import declarativo (Terraform >= 1.5)
    
    A partir de Terraform 1.5 existe una forma más limpia: declarar el import en el código con un bloque `import`. Queda documentado y se ejecuta con el flujo normal de plan/apply.
    
    ```
    # import.tf
    import {
      id = "i-0a1b2c3d4e5f"
      to = aws_instance.servidor
    }
    
    import {
      id = "sg-12345"
      to = aws_security_group.web
    }
    ```
    
    ```bash
    terraform plan
    terraform apply
    ```
    
    Una vez aplicado, eliminás los bloques `import` del código.
    
    **Generar el código `.tf` automáticamente:**
    
    `terraform plan -generate-config-out=recursos_importados.tf`
    
    Terraform lee los bloques `import`, consulta el proveedor y genera el HCL completo. El archivo generado va a tener decenas de atributos — usalo como punto de partida y limpiá todo lo que no necesitás gestionar.
    
- 11.5 Limitaciones y consideraciones
    
    **No todos los recursos son importables** — en la documentación del registry cada recurso tiene una sección _Import_ al final. Si no existe, ese recurso no soporta import.
    
    **Si el plan muestra destroy después del import** — significa que algún atributo en tu código no coincide con el recurso real. Revisá y corregí antes de aplicar; nunca aplicar a ciegas después de un import.
    
    **Atributos write-only** — algunos atributos (como passwords) se pueden escribir al crear el recurso pero no se pueden leer después. Después del import esos campos quedan vacíos y hay que manejarlos con `ignore_changes`:
    
    ```
    lifecycle {
      ignore_changes = [password]
    }
    ```
    
    **Las dependencias no se importan automáticamente** — si importás una instancia EC2 con un Security Group, hay que importar el Security Group por separado.
    

### 12. Buenas prácticas

- 12.1 Estructura de proyectos reales
    
    **Patrón 1 — Monorepo con módulos internos**
    
    Todo el código de infraestructura en un solo repositorio separado por carpetas:
    
    ```
    infraestructura/
    ├── modulos/                    # módulos reutilizables internos
    │   ├── red/
    │   ├── servidor/
    │   ├── base-datos/
    │   └── kubernetes/
    ├── entornos/                   # configuración por ambiente
    │   ├── dev/
    │   │   ├── main.tf
    │   │   ├── variables.tf
    │   │   ├── outputs.tf
    │   │   └── terraform.tfvars
    │   ├── qa/
    │   │   ├── main.tf
    │   │   ├── variables.tf
    │   │   ├── outputs.tf
    │   │   └── terraform.tfvars
    │   └── prod/
    │       ├── main.tf
    │       ├── variables.tf
    │       ├── outputs.tf
    │       └── terraform.tfvars
    └── README.md
    ```
    
    Cada carpeta en `entornos/` tiene su propio estado y se aplica de forma independiente:
    
    `cd entornos/dev && terraform apply cd entornos/prod && terraform apply`
    
- 12.3 Uso de tags en todos los recursos
    
    Los tags son metadatos que adjuntás a los recursos del proveedor. Son gratuitos y aportan un valor enorme para la gestión operativa. Sin tags es imposible saber a qué proyecto pertenece un recurso, quién lo creó o cuánto cuesta.
    
    **Tags mínimos recomendados:**
    
    ```
    locals {
      tags_comunes = {
        Proyecto    = var.proyecto
        Ambiente    = var.ambiente
        Equipo      = var.equipo
        Owner       = var.owner_email
        ManagedBy   = "terraform"
        Repositorio = "github.com/mi-empresa/infra"
        CostCenter  = var.cost_center
      }
    }
    ```
    
    **Aplicarlos a todos los recursos con `default_tags` en AWS:**
    
    En lugar de agregar `tags = local.tags_comunes` en cada resource, podés configurar tags por defecto directamente en el provider de AWS y se aplican automáticamente a todos los recursos:
    
    ```
    provider "aws" {
      region = var.region
    
      default_tags {
        tags = local.tags_comunes
      }
    }
    ```
    
    Con esto todos los recursos creados por ese provider van a tener esos tags sin que tengas que escribirlos explícitamente en cada resource. Si un resource necesita tags adicionales los agregás normalmente y se van a mergear con los default:
    
- 12.4 Seguridad
    
    **Nunca hardcodear credenciales en el código:**
    
    ```
    # ❌ Nunca hacer esto
    provider "aws" {
      access_key = "AKIAIOSFODNN7EXAMPLE"
      secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
    }
    
    # ✅ Usar variables de entorno o perfil de AWS CLI
    provider "aws" {
      region  = var.region
      profile = var.aws_profile
    }
    ```
    
    **Usar `sensitive = true` para variables con datos sensibles:**
    
    ```
    variable "db_password" {
      type      = string
      sensitive = true
    }
    
    variable "api_key" {
      type      = string
      sensitive = true
    }
    ```
    
    **Pasar secrets desde el gestor de secretos del proveedor:**
    
    En lugar de pasar passwords como variables, leerlos directamente desde AWS Secrets Manager o Parameter Store:
    
    ```
    data "aws_secretsmanager_secret_version" "db_credentials" {
      secret_id = "mi-app/prod/db-credentials"
    }
    
    locals {
      db_credentials = jsondecode(
        data.aws_secretsmanager_secret_version.db_credentials.secret_string
      )
    }
    
    resource "aws_db_instance" "base_datos" {
      identifier = "mi-db-prod"
      engine     = "postgres"
      username   = local.db_credentials["username"]
      password   = local.db_credentials["password"]
    
      lifecycle {
        ignore_changes = [password]
      }
    }
    ```
    
    **Proteger el bucket de estado:**
    
    ```
    resource "aws_s3_bucket_policy" "terraform_state" {
      bucket = aws_s3_bucket.terraform_state.id
    
      policy = jsonencode({
        Version = "2012-10-17"
        Statement = [
          {
            Effect    = "Deny"
            Principal = "*"
            Action    = "s3:*"
            Resource  = [
              aws_s3_bucket.terraform_state.arn,
              "${aws_s3_bucket.terraform_state.arn}/*"
            ]
            Condition = {
              Bool = {
                "aws:SecureTransport" = "false"
              }
            }
          }
        ]
      })
    }
    ```
    
- 12.5 Terraform en pipelines CI/CD
    
    Integrar Terraform en un pipeline automatizado es la práctica más importante para equipos que trabajan en conjunto. Elimina los problemas de "funciona en mi máquina" y garantiza que todos los cambios pasan por el mismo proceso.
    
    **Flujo típico de un pipeline:**
    
    ```
    Push a branch
          │
          ▼
    terraform fmt -check     ← falla si el código no está formateado
          │
          ▼
    terraform validate       ← falla si hay errores de sintaxis
          │
          ▼
    terraform plan           ← muestra los cambios, guarda el plan
          │
          ▼
    Pull Request Review      ← un humano revisa el plan
          │
          ▼
    Merge a main
          │
          ▼
    terraform apply          ← aplica el plan guardado
    ```
    
    ```yaml
    # .github/workflows/terraform.yml
    name: Terraform
    
    on:
      push:
        branches: [main]
      pull_request:
        branches: [main]
    
    env:
      TF_VERSION: "1.7.0"
      AWS_REGION: "us-east-1"
    
    jobs:
      terraform:
        runs-on: ubuntu-latest
        permissions:
          contents: read
          pull-requests: write
        steps:
          - name: Checkout
            uses: actions/checkout@v4
          - name: Setup Terraform
            uses: hashicorp/setup-terraform@v3
            with:
              terraform_version: ${{ env.TF_VERSION }}
          - name: Configure AWS credentials
            uses: aws-actions/configure-aws-credentials@v4
            with:
              aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
              aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
              aws-region: ${{ env.AWS_REGION }}
          - name: Terraform Init
            run: terraform init
          - name: Terraform Format Check
            run: terraform fmt -check -recursive
          - name: Terraform Validate
            run: terraform validate
          - name: Terraform Plan
            id: plan
            run: terraform plan -no-color -out=tfplan
          - name: Terraform Apply
            if: github.ref == 'refs/heads/main' && github.event_name == 'push'
            run: terraform apply -auto-approve tfplan
    ```
    
- 12.6 `-auto-approve`: cuándo sí y cuándo no
    
    El flag `-auto-approve` salta la confirmación manual del `apply`. Es necesario en pipelines automatizados porque no hay un humano para escribir `yes`, pero en manos equivocadas puede ser destructivo.
    
    **Cuándo usarlo:**
    
    ```bash
    # ✅ En un pipeline de CI/CD donde el plan ya fue revisado y aprobado
    terraform apply -auto-approve tfplan
    
    # ✅ En entornos efímeros de desarrollo o testing
    terraform destroy -auto-approve
    
    # ✅ Para infraestructura de bajo riesgo que se crea y destruye frecuentemente
    terraform apply -auto-approve -var-file="dev.tfvars"
    ```
    
    **Cuándo no usarlo:**
    
    ```bash
    # ❌ Nunca en producción sin haber revisado el plan primero
    terraform apply -auto-approve -var-file="prod.tfvars"
    
    # ❌ Nunca cuando el plan muestra recursos que se van a destruir
    # y no estás seguro de por qué
    ```
    
    **La regla de oro:**
    
    Siempre correr `terraform plan` primero y leer el output completo. Prestar especial atención a las líneas que dicen `destroy` o `-/+` (destroy + create). Si algo va a ser destruido y no esperabas eso, no aplicar hasta entender por qué.
    
    **Usar `terraform fmt` antes de cada commit**
    
    Idealmente configurarlo como un pre-commit hook para que se corra automáticamente.
    
    **Pinear versiones de todo**
    
    Tanto la versión de Terraform como la de cada provider y módulo externo deben estar fijadas. Nunca usar `latest` implícito:
    
    ```
    terraform {
      required_version = "= 1.7.0"
    
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "= 5.31.0"
        }
      }
    }
    
    module "vpc" {
      source  = "terraform-aws-modules/vpc/aws"
      version = "= 5.5.1"
    }
    ```
    
    **No usar `terraform apply` directamente en producción desde una máquina local**
    
    Todos los cambios a producción deben pasar por el pipeline de CI/CD. Esto garantiza que:
    
    - El código que se aplica es el que está en el repositorio
    - Hay un registro de quién aprobó el cambio y cuándo
    - Se corrieron todos los checks de validación
    
    **Revisar periódicamente con `terraform plan`**
    
    Aunque nadie haya tocado el código, correr un plan periódicamente detecta si alguien modificó algo manualmente en la consola del proveedor (drift de configuración):
    
    ```bash
    terraform plan -detailed-exitcode
    # exit code 0: no hay cambios
    # exit code 1: error
    # exit code 2: hay cambios — alguien tocó algo manualmente
    ```
    
    **Documentar decisiones de arquitectura**
    
    Cuando tomás una decisión de infraestructura que no es obvia documentala en un comentario en el código:
    
    ```
    # Usamos t3.large en vez de t3.medium porque el análisis de memoria
    # de marzo 2026 mostró que el proceso de indexación usa hasta 5GB RAM.
    resource "aws_instance" "indexador" {
      ami           = var.ami
      instance_type = "t3.large"
    }
    ```