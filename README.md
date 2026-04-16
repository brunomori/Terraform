# Terraform + IAM Role – Guia Prático (Nimbus)

> Este documento explica **de forma simples, prática e direta** o que foi feito, **por que foi feito**, e **como usar** o módulo Terraform de IAM Role que você acabou de criar.

Ele serve como **material de consulta rápida** para fixar conceitos de Terraform, módulos, fluxo Git, Terragrunt e governança.

---

## 📌 Contexto Geral

Você estava trabalhando no **Projeto Nimbus**, com o objetivo de:

- Criar uma **IAM Role de desenvolvedor** com **mínimos privilégios**;
- Padronizar o modelo de acesso entre contas AWS;
- Substituir criação manual (CLI / Console) por **Infraestrutura como Código**;
- Permitir uso em escala via **Terraform + Terragrunt**;
- Manter tudo versionado e auditável no Bitbucket.

Para isso, foi criado um **módulo Terraform** chamado `aws-iam-role`.

---

## 🧠 Conceitos-Chave (bem resumidos)

### O que é Terraform?
Terraform é uma ferramenta para **criar e manter recursos de infraestrutura** (AWS, por exemplo) usando código.

👉 Em vez de clicar na Console ou rodar CLI, você descreve *como o recurso deve ser*.

---

### O que é um Módulo Terraform?

Um **módulo** é como uma **função reutilizável**:

- Ele **não executa nada sozinho**
- Ele apenas define *como* criar um recurso
- Ele é chamado por outro código (stack / ambiente)

📌 Importante: **módulo NÃO roda `terraform apply`**.

---

### Onde entra o Terragrunt?

- Terraform → cria recursos
- Terragrunt → **orquestra Terraform em escala**

Terragrunt decide:
- Em qual conta AWS
- Em qual ambiente (dev / prod)
- Onde fica o state (S3)
- Qual módulo usar

📌 Regra da empresa:
> Módulos = Terraform puro
> Apply = feito via Terragrunt em outro repositório

---

## 📂 Estrutura do Módulo criado

```text
aws-iam-role/
├── data.tf
├── locals.tf
├── main.tf
├── outputs.tf
├── variables.tf
└── versions.tf
```

Todos esses arquivos juntos formam **um único módulo**.

---

## 📄 Explicação arquivo por arquivo

### 1️⃣ `data.tf`

```hcl
# Reservado para data sources, se necessário futuramente
```

- Usado para **consultar dados da AWS** (se precisar)
- Neste módulo, **não é necessário agora**
- Manter vazio é normal e aceitável

---

### 2️⃣ `locals.tf`

```hcl
locals {
  default_tags = {
    AppID            = var.app_id
    CostString       = var.cost_string
    Environment      = var.environment == "prod" ? "prd" : var.environment
    BusinessServices = var.business_services
    CreatedBy        = "Terraform"
    BU               = "Agribusiness"
    ResourceType     = var.resource_type
  }
}
```

📌 Função desse arquivo:
- Centralizar **regras internas**
- Definir **tags corporativas padrão**
- Evitar duplicação

✅ Esse padrão já existe em outros módulos (SQS, SNS, S3).

---

### 3️⃣ `main.tf` (arquivo mais importante)

```hcl
resource "aws_iam_role" "this" {
  name               = var.role_name
  assume_role_policy = var.assume_role_policy
  tags               = local.default_tags
}
```

Cria a **IAM Role** com:
- Nome dinâmico
- Trust policy recebida por variável
- Tags padrão da empresa

---

```hcl
resource "aws_iam_role_policy_attachment" "managed" {
  for_each = toset(var.managed_policy_arns)

  role       = aws_iam_role.this.name
  policy_arn = each.value
}
```

Permite anexar **policies gerenciadas** (AWS ou corporativas).

---

```hcl
resource "aws_iam_role_policy" "inline" {
  for_each = var.inline_policies

  name   = each.key
  role   = aws_iam_role.this.id
  policy = each.value
}
```

Permite criar **policies inline**, como:
- Policy mínima de Dev
- Policy específica por role

---

### 4️⃣ `variables.tf`

Define tudo o que o módulo **espera receber de fora**:

```hcl
variable "role_name" { type = string }
variable "assume_role_policy" { type = string }
variable "managed_policy_arns" { type = list(string) default = [] }
variable "inline_policies" { type = map(string) default = {} }
```

E também as variáveis usadas nas tags:

```hcl
variable "app_id" {}
variable "cost_string" {}
variable "environment" {}
variable "business_services" {}
variable "resource_type" {}
```

📌 Sem essas variáveis, o `locals.tf` não funciona.

---

### 5️⃣ `outputs.tf`

```hcl
output "role_name" { value = aws_iam_role.this.name }
output "role_arn"  { value = aws_iam_role.this.arn }
```

Outputs servem para:
- Referenciar a role em outros módulos
- Auditoria
- Debug

---

### 6️⃣ `versions.tf`

```hcl
terraform {
  required_version = ">= 1.3.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
  }
}
```

Garante:
- Versão mínima do Terraform
- Versão mínima do provider AWS

Evita erro de execução no pipeline.

---

## 🔄 Fluxo completo do que aconteceu

```text
Você criou os arquivos
        ↓
Criou um módulo Terraform
        ↓
Subiu no Bitbucket (repo de módulos)
        ↓
Tag v1.0.0 (após merge)
        ↓
Outro repo (contas AWS) usa o módulo
        ↓
Terragrunt chama o módulo
        ↓
terragrunt apply
        ↓
IAM Role criada na AWS
```

---

## 🧪 Comandos Git usados (o essencial)

```bash
git clone <repo>
git checkout -b feat/SREPLATAFO-4425
git add aws-iam-role
git commit -m "feat(SREPLATAFO-4425): add aws iam role terraform module"
git push origin feat/SREPLATAFO-4425
```

---

## ⚠️ Pontos importantes para lembrar

- ✅ Módulo **não faz apply**
- ✅ Apply é feito com **Terragrunt em outro repo**
- ✅ IAM Role aceita tags (mesmo não sendo cobrável)
- ✅ Não misturar lógica de serviço (SQS, EC2) em módulo genérico
- ✅ Nome de arquivos importa (`main.tf`, `versions.tf`)

---

## ✅ O que você aprendeu aqui

- Como um módulo Terraform funciona
- Diferença entre Terraform e Terragrunt
- Para que serve cada arquivo `.tf`
- Como criar uma IAM Role de forma limpa
- Como subir código corretamente no Git
- Como isso entra no fluxo corporativo Nimbus

---

## ✅ Próximos passos naturais

1. Merge do PR
2. Criar tag `v1.0.0`
3. Usar o módulo no repo de contas AWS
4. Rodar `terragrunt apply`
5. Validar com dev
6. Evoluir governança IAM

---

📌 **Regra de ouro**:
Se você entende *este README*, você entende **80% do Terraform real usado na empresa**.
