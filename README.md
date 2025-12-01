# Setup Deploy Action

Esta GitHub Action configura as credenciais e arquivos necessários para realizar deploys em ambientes AWS EKS.

## Descrição

A action realiza as seguintes operações:

1. **Checkout do repositório** - Faz o checkout do código fonte
2. **Configuração de credenciais AWS** - Configura as credenciais AWS usando role assumption
3. **Login no Amazon ECR** - Realiza autenticação no Elastic Container Registry
4. **Geração de versão** - Extrai os primeiros 7 caracteres do SHA do commit para usar como versão
5. **Configuração do kubeconfig** - Configura o acesso ao cluster EKS
6. **Instalação do kubectl** - Instala a ferramenta kubectl para gerenciamento do Kubernetes

## Inputs

| Nome | Descrição | Obrigatório | Padrão |
|------|-----------|-------------|--------|
| `aws-account-id` | O ID da conta AWS (necessário para login no ECR) | Não* | - |
| `aws-access-key` | A chave de acesso AWS | Sim | - |
| `aws-secret-access-key` | A chave secreta de acesso AWS | Sim | - |
| `aws-role-arn` | O ARN da role AWS para assumir | Sim | - |

> **Nota:** O `aws-account-id` é tecnicamente opcional na definição da action, mas é necessário para o login no ECR funcionar corretamente. Recomenda-se sempre fornecê-lo.

## Variáveis de Ambiente Necessárias

Esta action espera que as seguintes variáveis de ambiente estejam definidas:

| Variável | Descrição |
|----------|-----------|
| `AWS_DEFAULT_REGION` | A região AWS padrão (ex: `us-east-1`) |
| `K8S_CLUSTER` | O nome do cluster EKS |

## Variáveis de Ambiente Exportadas

Após a execução, a seguinte variável de ambiente estará disponível:

| Variável | Descrição |
|----------|-----------|
| `VERSION` | Os primeiros 7 caracteres do SHA do commit |

## Uso

### Exemplo Básico

```yaml
name: Deploy

on:
  push:
    branches:
      - main

env:
  AWS_DEFAULT_REGION: us-east-1
  K8S_CLUSTER: meu-cluster-eks
  AWS_ACCOUNT_ID: '123456789012'  # Substitua pelo seu AWS Account ID

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Deploy
        uses: i9cloud-tech/action-setup-deploy@main
        with:
          aws-account-id: ${{ env.AWS_ACCOUNT_ID }}
          aws-access-key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/deployment.yaml
```

### Exemplo Completo com Build de Imagem Docker

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main

env:
  AWS_DEFAULT_REGION: us-east-1
  K8S_CLUSTER: meu-cluster-eks
  ECR_REPOSITORY: minha-aplicacao
  AWS_ACCOUNT_ID: '123456789012'  # Substitua pelo seu AWS Account ID

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Deploy
        uses: i9cloud-tech/action-setup-deploy@main
        with:
          aws-account-id: ${{ env.AWS_ACCOUNT_ID }}
          aws-access-key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}

      - name: Build and Push Docker Image
        run: |
          docker build -t ${{ env.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_DEFAULT_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.VERSION }} .
          docker push ${{ env.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_DEFAULT_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.VERSION }}

      - name: Deploy to Kubernetes
        run: |
          # Substitua 'nome-do-deployment' pelo nome do seu deployment no Kubernetes
          kubectl set image deployment/nome-do-deployment \
            nome-do-container=${{ env.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_DEFAULT_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.VERSION }}
```

> **Nota:** No exemplo acima, substitua `nome-do-deployment` e `nome-do-container` pelos nomes corretos do seu deployment e container no Kubernetes.

## Pré-requisitos

Antes de usar esta action, certifique-se de que:

1. Você possui uma conta AWS com as permissões necessárias
2. O cluster EKS está configurado e acessível
3. A role AWS possui as políticas necessárias para:
   - Acessar o Amazon ECR
   - Gerenciar o cluster EKS
4. Os seguintes valores estão configurados (como secrets ou variáveis de ambiente):
   - `AWS_ACCESS_KEY_ID` (secret)
   - `AWS_SECRET_ACCESS_KEY` (secret)
   - `AWS_ROLE_ARN` (secret)
   - `AWS_ACCOUNT_ID` (pode ser variável de ambiente ou secret)

## Autor

Robson Andrade

## Licença

Este projeto é de uso interno da i9cloud-tech.
