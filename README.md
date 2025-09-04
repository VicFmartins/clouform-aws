# clouform-aws

Infraestrutura Automatizada com AWS CloudFormation
0) Pré-requisitos

Conta AWS ativa com usuário IAM configurado.

Permissões para usar CloudFormation e os serviços que serão criados (ex.: S3, DynamoDB, EC2).

AWS CLI instalada e configurada:

aws --version
aws configure


ou, em ambientes corporativos, via SSO:

aws configure sso


Git instalado e conta no GitHub para versionamento.

1) Planejamento da Infraestrutura

Antes de escrever qualquer template, defina:

Ambiente de uso: dev, stg, prod.

Serviços que serão criados no MVP (por exemplo, S3 para armazenamento e DynamoDB para dados).

Região AWS em que a infraestrutura será provisionada, como us-east-1 ou outra adequada ao projeto.

2) Estrutura do Repositório

Organização recomendada:

infra-cloudformation/
├─ templates/
│  └─ infra.yml        # Template principal
├─ images/             # Prints ou diagramas (opcional)
├─ .github/
│  └─ workflows/
│     └─ deploy.yml    # Automação com GitHub Actions (opcional)
└─ README.md           # Documentação

3) Criação do Template CloudFormation

Crie o arquivo templates/infra.yml com a definição da infraestrutura. Exemplo inicial com S3 e DynamoDB:

AWSTemplateFormatVersion: '2010-09-09'
Description: 'Infraestrutura básica com S3 e DynamoDB'

Parameters:
  EnvironmentName:
    Type: String
    AllowedValues:
      - dev
      - stg
      - prod
    Default: dev

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'meu-bucket-${EnvironmentName}'
      VersioningConfiguration:
        Status: Enabled

  DynamoTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub 'meu-dynamo-${EnvironmentName}'
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

4) Validação do Template

Antes de criar o stack, valide o template:

aws cloudformation validate-template --template-body file://templates/infra.yml

5) Criação do Stack

Para provisionar a infraestrutura:

aws cloudformation create-stack \
  --stack-name infra-dev \
  --template-body file://templates/infra.yml \
  --parameters ParameterKey=EnvironmentName,ParameterValue=dev \
  --capabilities CAPABILITY_IAM

6) Atualização do Stack

Se houver modificações no template, atualize o stack existente:

aws cloudformation update-stack \
  --stack-name infra-dev \
  --template-body file://templates/infra.yml \
  --parameters ParameterKey=EnvironmentName,ParameterValue=dev \
  --capabilities CAPABILITY_IAM

7) Exclusão do Stack

Para remover toda a infraestrutura criada:

aws cloudformation delete-stack --stack-name infra-dev

8) Automação com GitHub Actions (opcional)

Crie .github/workflows/deploy.yml para automatizar o provisionamento a cada alteração no repositório:

name: Deploy CloudFormation

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout código
        uses: actions/checkout@v3

      - name: Configurar credenciais AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy CloudFormation
        run: |
          aws cloudformation deploy \
            --stack-name infra-dev \
            --template-file templates/infra.yml \
            --capabilities CAPABILITY_IAM

9) Boas Práticas

Usar parâmetros e variáveis para diferenciar ambientes (dev, stg, prod).

Centralizar valores em arquivos parameters.json para simplificar deploys.

Documentar alterações relevantes no README.

Utilizar controle de versão (GitHub) para rastrear mudanças na infraestrutura.
