# 🧪 Terraform Lab - AWS VPC (Base DevOps/SRE)

## 🎯 Objetivo

Aprender o fluxo básico do Terraform criando infraestrutura na AWS.

------------------------------------------------------------------------

## 📁 Estrutura do projeto

``` bash
~/terraform-labs/aws-vpc
```

------------------------------------------------------------------------

## ⚙️ Comandos principais

### 1. Inicializar projeto

``` bash
tf init
```

📌 Baixa providers e prepara o ambiente local

------------------------------------------------------------------------

### 2. Validar configuração

``` bash
tf validate
```

📌 Verifica sintaxe do Terraform (não cria nada)

------------------------------------------------------------------------

### 3. Planejar mudanças

``` bash
tf plan
```

📌 Simula o que será criado na AWS

------------------------------------------------------------------------

### 4. Aplicar infraestrutura

``` bash
tf apply
```

📌 Cria recursos reais na AWS

------------------------------------------------------------------------

### 5. Destruir infraestrutura

``` bash
tf destroy
```

📌 Remove tudo criado pelo Terraform

------------------------------------------------------------------------

## 🧠 Fluxo DevOps

init → plan → apply → destroy

------------------------------------------------------------------------

## 💡 Conceitos aprendidos

-   Infraestrutura como Código (IaC)
-   Controle de estado (state)
-   Segurança com preview (plan)
-   Automação de infraestrutura

------------------------------------------------------------------------

## 🚀 Resultado esperado do lab

-   AWS S3 Bucket criado via Terraform
-   Entendimento do ciclo completo de IaC
