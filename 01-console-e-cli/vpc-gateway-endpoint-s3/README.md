# Lab: AWS VPC Gateway Endpoint para Amazon S3

## Objetivo
Demonstrar a configuração de um **VPC Gateway Endpoint** para o Amazon S3, permitindo que instâncias EC2 acessem recursos do S3 de forma 100% privada através do backbone interno da AWS, sem tráfego exposto na internet pública.

## Arquitetura
* **VPC:** 10.1.0.0/16
* **Subnet:** 10.1.1.0/24 (Subnet Privada/Pública com IGW para SSH)
* **Recursos:** EC2 (Amazon Linux 2023), IAM Role com `AmazonS3FullAccess`, Gateway Endpoint (`com.amazonaws.us-east-1.s3`).

## Passos Executados
1. Criação da VPC, Subnet e tabela de rotas com acesso SSH configurado via Security Group.
2. Associação de IAM Instance Profile (Role) à EC2 para permissões de API no S3 sem credenciais estáticas.
3. Criação do **VPC Gateway Endpoint (Type Gateway)** associado à Route Table da Subnet.
4. Validação da rota automática injetada na Route Table apontando a Prefix List do S3 para o Endpoint (`vpce-xxx`).
5. Validação prática do upload de objetos via AWS CLI direto pelo terminal.

## Comandos Utilizados
`bash
# Listar buckets
a

# Criar e enviar arquivo de teste via rede privada
echo "Rota privada do Endpoint Funcionando!" > test.txt
aws s3 cp test.txt s3://lab-vpc-endppoint-austriaco-12345/# AWS-Labs
Repository of hands-on labs for AWS, Terraform, and infrastructure projects.
