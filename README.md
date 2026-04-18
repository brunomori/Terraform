# 🚀 Terraform - Guia de Referência Rápida

> Referência de consulta rápida com comandos essenciais, HCL patterns, troubleshooting e melhores práticas.
> Exemplos prontos para usar em infraestrutura como código.

---

## 📑 Índice

| # | Tema |
|---|------|
| 1 | [Comandos Essenciais](#-comandos-essenciais) |
| 2 | [Inicialização e Providers](#-inicialização-e-providers) |
| 3 | [Variáveis e Outputs](#-variáveis-e-outputs) |
| 4 | [Resources e Data Sources](#-resources-e-data-sources) |
| 5 | [State Management](#-state-management) |
| 6 | [Workspaces](#-workspaces) |
| 7 | [Módulos](#-módulos) |
| 8 | [Provisioners](#-provisioners) |
| 9 | [Funções Built-in](#-funções-built-in) |
| 10 | [Loops e Condicionais](#-loops-e-condicionais) |
| 11 | [Remote State](#-remote-state) |
| 12 | [Import e Taint](#-import-e-taint) |
| 13 | [HCL Patterns](#-hcl-patterns) |
| 14 | [Troubleshooting](#-troubleshooting) |
| 15 | [Flags Úteis](#-flags-úteis) |
| 16 | [Melhores Práticas](#-melhores-práticas) |

---

## ⚡ Comandos Essenciais

> Base para qualquer workflow Terraform

```bash
terraform version                         # versão instalada
terraform init                            # inicializar diretório (download de providers)
terraform init -upgrade                   # atualizar providers
terraform validate                        # validar sintaxe
terraform fmt                             # formatar código
terraform fmt -recursive                  # formatar todos os arquivos
terraform plan                            # visualizar mudanças
terraform plan -out=plan.tfplan           # salvar plano
terraform apply                           # aplicar mudanças (pede confirmação)
terraform apply -auto-approve             # aplicar sem confirmação
terraform apply plan.tfplan               # aplicar plano salvo
terraform destroy                         # destruir toda infraestrutura
terraform destroy -auto-approve           # destruir sem confirmação
terraform show                            # mostrar estado atual
terraform output                          # listar outputs
terraform output nome_output              # valor específico
```

> ⚠️ Nunca use `-auto-approve` em produção sem revisão do plano!

---

## 🔧 Inicialização e Providers

> Aula base — configurar providers e backend

```bash
terraform init                            # primeira execução
terraform init -backend-config=backend.hcl  # configurar backend dinâmico
terraform init -reconfigure               # reconfigurar backend
terraform init -migrate-state             # migrar state para novo backend
terraform providers                       # listar providers instalados
terraform providers schema -json          # schema completo dos providers
```

### Provider AWS básico

```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  }
}
```

### Provider Azure básico

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}
```

### Provider Google Cloud básico

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}
```

---

## 📝 Variáveis e Outputs

> Parametrização e exposição de valores

### Declaração de variáveis (variables.tf)

```hcl
# Variável simples
variable "aws_region" {
  description = "AWS region para recursos"
  type        = string
  default     = "us-east-1"
}

# Variável com validação
variable "environment" {
  description = "Ambiente de deploy"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment deve ser dev, staging ou prod."
  }
}

# Variável de número
variable "instance_count" {
  description = "Número de instâncias"
  type        = number
  default     = 2
}

# Variável booleana
variable "enable_monitoring" {
  description = "Habilitar monitoramento"
  type        = bool
  default     = true
}

# Variável tipo lista
variable "availability_zones" {
  description = "AZs para deploy"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

# Variável tipo mapa
variable "instance_types" {
  description = "Tipos de instância por ambiente"
  type        = map(string)
  default = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.medium"
  }
}

# Variável tipo objeto
variable "vpc_config" {
  description = "Configuração da VPC"
  type = object({
    cidr_block           = string
    enable_dns_hostnames = bool
    enable_dns_support   = bool
  })
  default = {
    cidr_block           = "10.0.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support   = true
  }
}

# Variável sensível
variable "database_password" {
  description = "Senha do banco de dados"
  type        = string
  sensitive   = true
}
```

### Formas de passar valores

```bash
# Via arquivo terraform.tfvars (carregado automaticamente)
aws_region  = "us-west-2"
environment = "prod"

# Via arquivo custom
terraform apply -var-file="prod.tfvars"

# Via linha de comando
terraform apply -var="environment=dev" -var="instance_count=5"

# Via variável de ambiente
export TF_VAR_aws_region="eu-west-1"
terraform apply
```

### Outputs (outputs.tf)

```hcl
# Output simples
output "instance_public_ip" {
  description = "IP público da instância"
  value       = aws_instance.web.public_ip
}

# Output sensível (não aparece no console)
output "database_password" {
  description = "Senha do RDS"
  value       = aws_db_instance.main.password
  sensitive   = true
}

# Output de lista
output "subnet_ids" {
  description = "IDs das subnets criadas"
  value       = aws_subnet.private[*].id
}

# Output de mapa
output "instance_ips" {
  description = "IPs por ambiente"
  value = {
    for k, v in aws_instance.servers : k => v.private_ip
  }
}
```

```bash
# Ver outputs
terraform output                          # todos
terraform output instance_public_ip       # específico
terraform output -json                    # formato JSON
terraform output -raw instance_public_ip  # valor raw (sem aspas)
```

---

## 🏗 Resources e Data Sources

> Criar e consultar recursos

### Resource — criar recurso

```hcl
# EC2 Instance
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_types[var.environment]
  
  tags = {
    Name = "${var.project_name}-web-${var.environment}"
  }
  
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [tags]
  }
}

# S3 Bucket
resource "aws_s3_bucket" "data" {
  bucket = "${var.project_name}-data-${var.environment}"
  
  tags = local.common_tags
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_config.cidr_block
  enable_dns_hostnames = var.vpc_config.enable_dns_hostnames
  
  tags = {
    Name = "${var.project_name}-vpc"
  }
}
```

### Data Source — consultar recurso existente

```hcl
# Buscar AMI mais recente
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Buscar VPC existente
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["production-vpc"]
  }
}

# Buscar availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Buscar região atual
data "aws_region" "current" {}

# Buscar account ID
data "aws_caller_identity" "current" {}
```

### Uso de data sources

```hcl
resource "aws_instance" "app" {
  ami               = data.aws_ami.ubuntu.id
  availability_zone = data.aws_availability_zones.available.names[0]
  
  tags = {
    Region    = data.aws_region.current.name
    AccountID = data.aws_caller_identity.current.account_id
  }
}
```

---

## 💾 State Management

> Gerenciamento do arquivo de estado

```bash
# Listar recursos no state
terraform state list

# Mostrar detalhes de um recurso
terraform state show aws_instance.web

# Mover recurso no state (renomear)
terraform state mv aws_instance.old aws_instance.new

# Remover recurso do state (não deleta o recurso real)
terraform state rm aws_instance.web

# Puxar state remoto
terraform state pull > terraform.tfstate.backup

# Atualizar state sem aplicar mudanças
terraform refresh

# Substituir provider no state
terraform state replace-provider hashicorp/aws registry.terraform.io/hashicorp/aws
```

> ⚠️ Sempre faça backup antes de manipular state: `terraform state pull > backup.tfstate`

### Local State (desenvolvimento)

```hcl
# terraform.tf
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

### Remote State S3 (produção)

```hcl
terraform {
  backend "s3" {
    bucket         = "meu-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

### Remote State Azure

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

---

## 🔀 Workspaces

> Múltiplos ambientes com mesmo código

```bash
terraform workspace list                  # listar workspaces
terraform workspace show                  # workspace atual
terraform workspace new dev               # criar workspace
terraform workspace new staging
terraform workspace new prod
terraform workspace select dev            # trocar workspace
terraform workspace delete dev            # deletar workspace
```

### Uso em código

```hcl
# Usar workspace no código
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
  
  tags = {
    Name        = "web-${terraform.workspace}"
    Environment = terraform.workspace
  }
}

# Locals baseados em workspace
locals {
  instance_counts = {
    dev     = 1
    staging = 2
    prod    = 5
  }
  
  instance_count = local.instance_counts[terraform.workspace]
}
```

> 💡 Workspaces são úteis para dev/staging, mas para produção considere usar diretórios separados.

---

## 📦 Módulos

> Reutilização e organização de código

### Estrutura de módulo

```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
    README.md
  ec2/
    main.tf
    variables.tf
    outputs.tf
```

### Criar módulo (modules/vpc/main.tf)

```hcl
# modules/vpc/variables.tf
variable "vpc_cidr" {
  type = string
}

variable "project_name" {
  type = string
}

# modules/vpc/main.tf
resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  
  tags = {
    Name = "${var.project_name}-vpc"
  }
}

resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.this.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
  }
}

# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.this.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}
```

### Usar módulo

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr     = "10.0.0.0/16"
  project_name = var.project_name
}

# Referenciar outputs do módulo
resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_ids[0]
  vpc_id    = module.vpc.vpc_id
}
```

### Módulo do Registry

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.2"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
}
```

```bash
terraform get                             # download de módulos
terraform init                            # também faz download
```

---

## 🔨 Provisioners

> Executar scripts após criação de recursos (último recurso!)

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  # Provisioner local-exec (executa na máquina que roda terraform)
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
  
  # Provisioner remote-exec (executa na instância)
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo systemctl start nginx"
    ]
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
  
  # Provisioner file (copia arquivo)
  provisioner "file" {
    source      = "app.conf"
    destination = "/tmp/app.conf"
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
  
  # Executar apenas na destruição
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance destroyed' >> log.txt"
  }
}
```

> ⚠️ Provisioners são último recurso. Prefira user_data, cloud-init ou ferramentas como Ansible.

---

## 🧮 Funções Built-in

> Manipulação de strings, listas, mapas e mais

### Strings

```hcl
# Concatenar
"${var.project}-${var.environment}"       # Interpolação

# Maiúscula/Minúscula
upper("hello")                            # "HELLO"
lower("WORLD")                            # "world"
title("hello world")                      # "Hello World"

# Substring
substr("hello world", 0, 5)               # "hello"

# Replace
replace("hello world", "world", "terraform")  # "hello terraform"

# Split/Join
split(",", "a,b,c")                       # ["a", "b", "c"]
join("-", ["a", "b", "c"])                # "a-b-c"

# Trim
trimspace("  hello  ")                    # "hello"
```

### Listas

```hcl
# Tamanho
length(["a", "b", "c"])                   # 3

# Elementos
element(["a", "b", "c"], 1)               # "b"
index(["a", "b", "c"], "b")               # 1

# Manipulação
concat(["a", "b"], ["c", "d"])            # ["a", "b", "c", "d"]
distinct(["a", "b", "a", "c"])            # ["a", "b", "c"]
sort(["c", "a", "b"])                     # ["a", "b", "c"]
reverse(["a", "b", "c"])                  # ["c", "b", "a"]

# Slice
slice(["a", "b", "c", "d"], 1, 3)         # ["b", "c"]

# Contains
contains(["a", "b", "c"], "b")            # true
```

### Mapas

```hcl
# Acessar valores
lookup(var.instance_types, "prod", "t3.micro")  # valor ou default

# Keys/Values
keys({a = 1, b = 2})                      # ["a", "b"]
values({a = 1, b = 2})                    # [1, 2]

# Merge
merge({a = 1}, {b = 2}, {c = 3})          # {a = 1, b = 2, c = 3}
```

### Números

```hcl
min(1, 2, 3)                              # 1
max(1, 2, 3)                              # 3
ceil(5.1)                                 # 6
floor(5.9)                                # 5
```

### Datas

```hcl
timestamp()                               # "2024-04-18T10:30:00Z"
formatdate("DD MMM YYYY hh:mm ZZZ", timestamp())  # "18 Apr 2024 10:30 UTC"
```

### Networking

```hcl
cidrsubnet("10.0.0.0/16", 8, 2)           # "10.0.2.0/24"
cidrhost("10.0.0.0/24", 5)                # "10.0.0.5"
```

### Encoding

```hcl
base64encode("hello")                     # "aGVsbG8="
base64decode("aGVsbG8=")                  # "hello"
jsonencode({a = 1, b = 2})                # '{"a":1,"b":2}'
jsondecode('{"a":1}')                     # {a = 1}
```

### Filesystem

```hcl
file("${path.module}/script.sh")          # lê arquivo
fileexists("config.json")                 # true/false
templatefile("template.tpl", {name = "John"})  # renderiza template
```

---

## 🔁 Loops e Condicionais

> Recursos dinâmicos

### count — criar múltiplos recursos

```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  tags = {
    Name = "web-${count.index + 1}"
  }
}

# Referenciar
aws_instance.web[0].id
aws_instance.web[*].id  # todos
```

### for_each — criar com mapa ou set

```hcl
# Com mapa
variable "instances" {
  type = map(string)
  default = {
    web = "t3.micro"
    api = "t3.small"
    db  = "t3.medium"
  }
}

resource "aws_instance" "servers" {
  for_each = var.instances
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = each.value
  
  tags = {
    Name = each.key
  }
}

# Referenciar
aws_instance.servers["web"].id

# Com set
resource "aws_iam_user" "developers" {
  for_each = toset(["alice", "bob", "charlie"])
  name     = each.key
}
```

### for expressions — transformar listas/mapas

```hcl
# Lista
[for s in var.list : upper(s)]

# Mapa
{for k, v in var.map : k => upper(v)}

# Com condição
[for s in var.list : upper(s) if length(s) > 5]

# Exemplo real
locals {
  instance_ips = [for i in aws_instance.web : i.private_ip]
  
  uppercase_tags = {
    for k, v in var.tags : k => upper(v)
  }
}
```

### Condicionais — ternário

```hcl
# condition ? true_val : false_val
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

# count com condicional (criar ou não criar)
resource "aws_instance" "bastion" {
  count = var.enable_bastion ? 1 : 0
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}

# Referenciar recurso condicional
bastion_ip = var.enable_bastion ? aws_instance.bastion[0].public_ip : null
```

### dynamic blocks

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

# variables.tf
variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}
```

---

## 🌐 Remote State

> Compartilhar dados entre projetos

### Configurar data source

```hcl
# Projeto B lê state do Projeto A
data "terraform_remote_state" "vpc" {
  backend = "s3"
  
  config = {
    bucket = "meu-terraform-state"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

# Usar outputs do remote state
resource "aws_instance" "web" {
  subnet_id = data.terraform_remote_state.vpc.outputs.public_subnet_id
  vpc_id    = data.terraform_remote_state.vpc.outputs.vpc_id
}
```

### Expor outputs para consumo

```hcl
# Projeto A - outputs.tf
output "vpc_id" {
  value       = aws_vpc.main.id
  description = "ID da VPC para uso em outros projetos"
}

output "public_subnet_id" {
  value = aws_subnet.public.id
}
```

---

## 📥 Import e Taint

> Gerenciar recursos existentes

### Import — trazer recurso existente para Terraform

```bash
# 1. Criar resource vazio no código
resource "aws_instance" "imported" {
  # Deixe vazio inicialmente
}

# 2. Importar
terraform import aws_instance.imported i-1234567890abcdef0

# 3. Ver configuração atual
terraform show

# 4. Copiar configuração para o .tf
# 5. Executar plan para ajustar
terraform plan
```

### Taint — forçar recriação

```bash
# Marcar para recriação (deprecated em Terraform 1.5+)
terraform taint aws_instance.web

# Forma atual (Terraform 1.5+)
terraform apply -replace="aws_instance.web"

# Desfazer taint
terraform untaint aws_instance.web
```

---

## 📐 HCL Patterns

> Padrões comuns de código

### Locals — valores computados

```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "Terraform"
    CreatedAt   = timestamp()
  }
  
  name_prefix = "${var.project_name}-${var.environment}"
  
  subnet_count = length(data.aws_availability_zones.available.names)
  
  # Condicional complexa
  instance_type = (
    var.environment == "prod" ? "t3.large" :
    var.environment == "staging" ? "t3.medium" :
    "t3.micro"
  )
}

resource "aws_instance" "web" {
  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-web"
    }
  )
}
```

### Moved block — refatoração sem destruição

```hcl
# Renomear recurso sem destruir
moved {
  from = aws_instance.web
  to   = aws_instance.webserver
}
```

### Lifecycle — controle de criação/destruição

```hcl
resource "aws_instance" "web" {
  # ... configuração ...
  
  lifecycle {
    create_before_destroy = true   # criar novo antes de destruir antigo
    prevent_destroy       = true   # bloquear destruição acidental
    ignore_changes        = [       # ignorar mudanças em campos específicos
      tags,
      user_data
    ]
  }
}
```

### Depends_on — dependências explícitas

```hcl
resource "aws_instance" "web" {
  # ... configuração ...
  
  # Terraform detecta dependências automaticamente,
  # mas às vezes precisa ser explícito
  depends_on = [
    aws_iam_role_policy.s3_access,
    aws_security_group.web
  ]
}
```

### Terraform console — testar expressões

```bash
terraform console

# Dentro do console:
> var.environment
"prod"

> upper(var.environment)
"PROD"

> length(var.availability_zones)
2

> [for az in var.availability_zones : upper(az)]
["US-EAST-1A", "US-EAST-1B"]
```

---

## 🔍 Troubleshooting

### Checklist — terraform apply falha

```bash
terraform validate                        # validar sintaxe
terraform fmt -check                      # verificar formatação
terraform plan                            # ver o que será criado
terraform plan -out=plan.tfplan           # salvar plano
terraform show plan.tfplan                # inspecionar plano salvo
```

### Erros comuns e soluções

| Erro | Causa | Solução |
|------|-------|---------|
| `Error: Duplicate resource` | Resource declarado 2x | Verificar código duplicado |
| `Error: Reference to undeclared variable` | Variável não declarada | Adicionar em `variables.tf` |
| `Error: Unsupported argument` | Campo inválido no resource | Consultar documentação do provider |
| `Error: Cycle` | Dependência circular | Revisar `depends_on` e referências |
| `Error acquiring state lock` | Outro processo usando state | Aguardar ou fazer force-unlock |
| `Error: Plugin did not respond` | Provider crashou | `terraform init -upgrade` |
| `Error: Invalid provider configuration` | Credenciais inválidas | Verificar AWS_ACCESS_KEY_ID, etc |

### Debug avançado

```bash
# Habilitar logs detalhados
export TF_LOG=TRACE
export TF_LOG_PATH=terraform.log
terraform apply

# Níveis de log: TRACE, DEBUG, INFO, WARN, ERROR

# Desabilitar
unset TF_LOG

# Ver plano em JSON
terraform show -json plan.tfplan | jq

# Gráfico de dependências
terraform graph | dot -Tpng > graph.png
```

### State lock travado

```bash
# Ver informações do lock
terraform force-unlock <LOCK_ID>

# ⚠️ Use apenas se tiver certeza que ninguém está executando terraform
```

### Recuperação de state

```bash
# Fazer backup do state
terraform state pull > backup.tfstate

# Restaurar backup
terraform state push backup.tfstate

# Remover state lock no S3 (emergência)
aws dynamodb delete-item \
  --table-name terraform-state-lock \
  --key '{"LockID":{"S":"<bucket>/<path>"}}'
```

---

## 🏴‍☠️ Flags Úteis

### Plan e Apply

```bash
# Plan
terraform plan -out=plan.tfplan           # salvar plano
terraform plan -refresh-only              # apenas atualizar state
terraform plan -target=aws_instance.web   # planejar recurso específico
terraform plan -var="environment=prod"    # passar variável
terraform plan -var-file="prod.tfvars"    # arquivo de variáveis
terraform plan -destroy                   # ver o que seria destruído

# Apply
terraform apply -auto-approve             # sem confirmação
terraform apply plan.tfplan               # aplicar plano salvo
terraform apply -target=aws_instance.web  # aplicar recurso específico
terraform apply -parallelism=20           # ajustar paralelismo (padrão: 10)
terraform apply -lock=false               # desabilitar lock (perigoso!)
terraform apply -replace="aws_instance.web"  # forçar recriação

# Destroy
terraform destroy -auto-approve
terraform destroy -target=aws_instance.web
```

### State

```bash
terraform state list                      # listar recursos
terraform state show aws_instance.web     # detalhes de recurso
terraform state mv SOURCE DEST            # mover/renomear
terraform state rm aws_instance.web       # remover do state
terraform state pull                      # baixar state remoto
terraform state push                      # enviar state
terraform state replace-provider OLD NEW  # trocar provider
```

### Formatação e validação

```bash
terraform fmt                             # formatar todos os arquivos
terraform fmt -check                      # verificar sem alterar
terraform fmt -recursive                  # incluir subdiretórios
terraform fmt -diff                       # mostrar diferenças

terraform validate                        # validar sintaxe
terraform validate -json                  # output JSON
```

### Outros

```bash
terraform init -upgrade                   # atualizar providers
terraform init -reconfigure               # reconfigurar backend
terraform init -backend-config=PATH       # configuração externa

terraform refresh                         # atualizar state (deprecated)
terraform providers                       # listar providers
terraform providers lock                  # criar lockfile
terraform version                         # versão do terraform
terraform -version                        # mesma coisa

# Gerar JSON schema
terraform providers schema -json > schema.json
```

---

## ✅ Melhores Práticas

### Estrutura de diretórios

```
project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   └── prod/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
├── .gitignore
└── README.md
```

### Arquivos essenciais

```hcl
# main.tf — recursos principais
# variables.tf — declaração de variáveis
# outputs.tf — outputs do módulo/projeto
# terraform.tf — configuração terraform e providers
# locals.tf — valores computados locais
# data.tf — data sources
# versions.tf — versões de providers
```

### .gitignore para Terraform

```gitignore
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate
*.tfstate.*

# Crash log files
crash.log
crash.*.log

# Exclude all .tfvars files (podem conter secrets)
*.tfvars
*.tfvars.json

# Ignore override files
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Ignore CLI configuration files
.terraformrc
terraform.rc

# Ignore plan files
*.tfplan
```

### Nomenclatura

```hcl
# Recursos: snake_case
resource "aws_instance" "web_server" {}

# Variáveis: snake_case
variable "instance_type" {}

# Outputs: snake_case
output "public_ip" {}

# Módulos: kebab-case
module "vpc-network" {
  source = "./modules/vpc-network"
}

# Tags: PascalCase
tags = {
  Name        = "WebServer"
  Environment = "Production"
}
```

### Segurança

```hcl
# ✅ BOM
variable "database_password" {
  type      = string
  sensitive = true  # Não aparece em logs
}

# Usar AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/database/password"
}

# ❌ RUIM — nunca commitar secrets
database_password = "senha123"  # NUNCA FAÇA ISSO
```

### State management

```hcl
# ✅ Sempre usar remote state em produção
terraform {
  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "vpc/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# ✅ Separar states por ambiente/componente
# dev/vpc/terraform.tfstate
# dev/app/terraform.tfstate
# prod/vpc/terraform.tfstate
# prod/app/terraform.tfstate
```

### Versionamento

```hcl
# ✅ Sempre fixar versões de providers
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # permite 5.x mas não 6.0
    }
  }
}

# ✅ Usar lockfile
# .terraform.lock.hcl — commitar no git
```

### Code review checklist

- [ ] `terraform fmt` aplicado
- [ ] `terraform validate` passou
- [ ] Variáveis sensíveis marcadas como `sensitive = true`
- [ ] Secrets não estão hardcoded
- [ ] Nomes de recursos seguem padrão
- [ ] Tags obrigatórias presentes
- [ ] Remote state configurado
- [ ] Versões de providers fixadas
- [ ] Plan revisado antes do apply
- [ ] Documentação atualizada

### Comandos antes de commitar

```bash
terraform fmt -recursive                  # formatar
terraform validate                        # validar
terraform plan                            # ver mudanças
# Revisar plan
git add .
git commit -m "feat: add RDS instance"
git push
```

---

## 📚 Exemplos Completos

### VPC completa com subnets

```hcl
# main.tf
data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(
    local.common_tags,
    {
      Name = "${var.project_name}-vpc"
    }
  )
}

resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
  }
}

resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "${var.project_name}-private-${count.index + 1}"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.project_name}-igw"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "${var.project_name}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

### EC2 com Auto Scaling

```hcl
resource "aws_launch_template" "web" {
  name_prefix   = "${var.project_name}-web-"
  image_id      = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(templatefile("${path.module}/user-data.sh", {
    environment = var.environment
  }))
  
  tag_specifications {
    resource_type = "instance"
    tags = merge(
      local.common_tags,
      {
        Name = "${var.project_name}-web"
      }
    )
  }
}

resource "aws_autoscaling_group" "web" {
  name                = "${var.project_name}-web-asg"
  vpc_zone_identifier = aws_subnet.private[*].id
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"
  min_size            = var.asg_min
  max_size            = var.asg_max
  desired_capacity    = var.asg_desired
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "${var.project_name}-web"
    propagate_at_launch = true
  }
}

resource "aws_autoscaling_policy" "scale_up" {
  name                   = "${var.project_name}-scale-up"
  autoscaling_group_name = aws_autoscaling_group.web.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = 1
  cooldown               = 300
}

resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "${var.project_name}-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 60
  statistic           = "Average"
  threshold           = 70
  alarm_actions       = [aws_autoscaling_policy.scale_up.arn]
  
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web.name
  }
}
```

---

## 🔗 Referências

- [Documentação oficial do Terraform](https://www.terraform.io/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Google Cloud Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [HCL Language](https://github.com/hashicorp/hcl)

---

**Criado por:** Guia de Referência Rápida  
**Versão:** 1.0  
**Última atualização:** Abril 2026
