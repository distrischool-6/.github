# Como Usar o Workflow de CI Reutilizável

Para habilitar o pipeline de Integração Contínua (CI) padrão em um dos seus microsserviços, siga os passos abaixo.

## Passo 1: Crie o Arquivo de Workflow no Repositório do Serviço

Em cada um dos seus repositórios de serviço (ex: `ds6-student-service`, `ds6-auth-service`, etc.), crie um arquivo no seguinte caminho:

`.github/workflows/main-ci.yml`

## Passo 2: Cole o Conteúdo Abaixo no Arquivo

Este código irá "chamar" o nosso workflow reutilizável que está no repositório `.github`.

```yaml
# .github/workflows/main-ci.yml
name: CI Pipeline

# Gatilhos: Executa no push para a main ou na abertura/atualização de um PR
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  # Nome do job, pode ser qualquer um
  build-and-test:
    # A permissão 'contents: read' é uma boa prática de segurança
    permissions:
      contents: read

    # 1. A palavra-chave 'uses' aponta para o workflow reutilizável
    # Formato: ORG/REPO/.github/workflows/ARQUIVO.yml@BRANCH
    uses: distrischool-6/.github/.github/workflows/reusable-java-ci.yml@main

    # 2. A palavra-chave 'with' fornece os valores para as 'entradas' (inputs)
    with:
      # IMPORTANTE: Mude este valor para o nome do serviço atual
      service-name: 'ds6-student-service'
      java-version: '17' # Opcional, o padrão já é '17'

    # 3. A palavra-chave 'secrets' passa os segredos necessários
    # 'inherit' é a forma mais fácil de passar todos os segredos do repositório
    # para o workflow reutilizável.
    secrets: inherit
```

## Passo 3: Configure as Permissões da Organização (Apenas uma vez)

Para que isso funcione, o GitHub precisa permitir que os workflows sejam compartilhados entre os repositórios da organização.

1.  Vá para as **Configurações (Settings)** da sua organização `distrischool-6`.
2.  No menu lateral, vá para **Actions > General**.
3.  Na seção **Policies**, encontre a opção **"Reusable workflows"**.
4.  Selecione **"Allow all reusable workflows from the distrischool-6 organization"**.

E é isso! Agora, sempre que você atualizar o `reusable-java-ci.yml` no repositório `.github`, todos os seus microsserviços que o utilizam receberão a atualização automaticamente.
