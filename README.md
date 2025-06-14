# Roteiro sobre configuração e uso de um pipeline CI/CD

Este repositório apresenta um roteiro prático para configurar e utilizar um pipeline de **Integração Contínua e Delivery Contínuo (CI/CD)**. O objetivo é simular práticas reais de DevOps no contexto de desenvolvimento web, utilizando o GitHub Actions.

O GitHub Actions permite automatizar fluxos de trabalho (workflows) como testes, builds e deploys. Neste tutorial, aplicaremos esses princípios em um cenário semelhante ao de produção.

**Contexto**: Considere que você faz parte de uma equipe de desenvolvimento Web para uma Ong de serviço para animais, resposável pelo Adote Fácil, sistema Web para adoção de animais. Sua equipe é distribuída geograficamente e é composta por profissionais da área de projeto Web em que cada um está responsável por diferentes partes do sistema. Você percebe que a adoção de metodologias de integração e entrega contínuas neste projeto irá promover diversas melhorias, como: Automatizar a função de um dos contribuintes de analisar os Pull Requests e realizar os Merges, disponibilizando-o para outras funções, elevar o padrão do codigo produzido, incentivar a criação e atualização constante de documentação, permitir a coleta de avaliações dos usuários de forma mais ágil, entre outras vantagens. Cientes desses potenciais ganhos, vocês desenvolveram um guia para a instalação de um ambiente servidor focado nessas práticas de CI/CD.

## Tarefa #1: Configurar o GitHub Actions
#### Passo 1

Crie um Token pessoal no GitHub (i) e;  faça um Fork (ii) do projeto adote-facil. 

**(i)** Para criar um Token, clique no ícone do seu perfil, vá em **Settings →**

![image](https://github.com/user-attachments/assets/9b5d64e8-1e8c-4d63-88fd-765b8cd7aeab)

**Developer settings →**

![image](https://github.com/user-attachments/assets/ea6fc354-d661-46a3-9cf7-cdc59b620c76)

**Personal access tokens →**

![image](https://github.com/user-attachments/assets/ea6fc354-d661-46a3-9cf7-cdc59b620c76)


**Tokens (classic)** →

![image](https://github.com/user-attachments/assets/44645080-395c-4a8b-a37f-adb8148bb3c1)

**Generate new token (classic)**. 

![image](https://github.com/user-attachments/assets/cafc4d73-2941-40de-a8eb-c111c2beea52)


Dê um nome para ele e marque as opções `repo` e `workflows` para gerar o Token. Gere o token mínimo (7 dias) apenas para este experimento. 

![image](https://github.com/user-attachments/assets/da72310e-d94c-4ea9-9c9f-7053b286fddd)

Copie e guarde esse código (token) para usar quando o GitHub lhe pedir para usar uma senha (password). 

#

**(ii)** Entre no repositório [adote-facil](https://github.com/ArthurEnrique15/adote-facil) e clique no botão **Fork** no canto superior direito na página do projeto no GitHub.

![image](https://github.com/user-attachments/assets/913b1f5d-2d42-4355-9361-ed6cda99107b)


#### Passo 2

Clone o repositório para sua máquina local, usando o seguinte comando (onde `<USER>` deve ser substituído pelo seu usuário no GitHub):

```bash
git clone https://github.com/<USER>/adote-facil.git
```

Em seguida, no diretório clonado, copie o código a seguir para um arquivo com o seguinte nome e caminho: `.github/workflows/experimento-ci-cd.yml`. Isto é, crie diretórios `.github` e depois `workflows` e salve o código abaixo no arquivo `experimento-ci-cd.yml`. Utilize os comandos `mv`, `cd`, `ls`, `mkdir` e `touch` no seu terminal ou use a GUI para criar os diretórios e arquivo.

```yml
name: experimento-ci-cd

on:
  pull_request:
    branches:
      - main

jobs:
  unit-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Instalar dependências do backend
        run: |
          cd backend
          npm install

      - name: Executar testes unitários com Jest
        run: |
          cd backend
          npm test -- --coverage

  build:
    needs: unit-test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Configurar Docker QEMU
        uses: docker/setup-qemu-action@v2

      - name: Build das imagens Docker
        run: docker compose build

  up-containers:
    needs: build
    runs-on: ubuntu-latest
    env:
      POSTGRES_DB: adote_facil
      POSTGRES_HOST: adote-facil-postgres
      POSTGRES_USER: ${{ secrets.POSTGRES_USER }}
      POSTGRES_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
      POSTGRES_PORT: 5432
      POSTGRES_CONTAINER_PORT: 6500

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Criar arquivo .env
        working-directory: ./backend
        run: |
          echo "POSTGRES_DB=${{ env.POSTGRES_DB }}" > .env
          echo "POSTGRES_HOST=${{ env.POSTGRES_HOST }}" >> .env
          echo "POSTGRES_USER=${{ env.POSTGRES_USER }}" >> .env
          echo "POSTGRES_PASSWORD=${{ env.POSTGRES_PASSWORD }}" >> .env
          echo "POSTGRES_PORT=${{ env.POSTGRES_PORT }}" >> .env
          echo "POSTGRES_CONTAINER_PORT=${{ env.POSTGRES_CONTAINER_PORT }}" >> .env

      - name: Subir containers com Docker Compose
        working-directory: ./backend
        run: |
          docker compose up -d
          sleep 10
          docker compose down

  delivery:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Gerar arquivo ZIP do projeto completo
        run: zip -r adote-facil-projeto.zip . -x '*.git*' '*.github*' 'node_modules/*'

      - name: Upload do artefato
        uses: actions/upload-artifact@v4
        with:
          name: adote-facil-projeto
          path: adote-facil-projeto.zip

```

#### Passo 3

Entre no diretório criado (use o comando cd no terminal) ".../adote-facil/", realize um `add`, um `commit` e um `git push`, ou seja:

```bash
git add --all
git commit -m "Configurando o GitHub Actions"
git push origin main
```

## Tarefa #2: Configurar GitHub Secrets

Em muitos projetos, utilizamos variáveis sensíveis (como senhas e chaves) que não devem ser expostas no código. Para isso, o GitHub Actions permite o uso de Secrets — variáveis de ambiente criptografadas e seguras. Para configurar os Secrets para armazenar os dados de conexão com o banco de dados é preciso:

#### Passo 1

Acesse seu repositório adote-facil no GitHub, clique em Settings, depois vá em: **Secrets and variables** → **Actions** → **New repository secret**

![image](https://github.com/user-attachments/assets/0e21ceab-8058-4c60-9b13-103a90ef376c)
![image](https://github.com/user-attachments/assets/cfd70f52-b185-4b7c-9a4b-780a1b2bd92c)
![image](https://github.com/user-attachments/assets/fcc27f4a-109f-4808-95ba-cffc0d473c59)

#### Passo 2

Crie dois segredos (secrets) com os seguintes valores:

1. **POSTGRES_USER**
   - **Name**: `POSTGRES_USER`
   - **Secret**: `Postgres`

2. **POSTGRES_PASSWORD**
   - **Name**: `POSTGRES_PASSWORD`
   - **Secret**: `postgres`

Os valores simulam um cenário de acesso ao banco de dados. Eles serão utilizados automaticamente no workflow `.github/workflows/experimento-ci-cd.yml`.

## Tarefa #3: Criando um Pull Request (PR) com bug

Este caso simula um cenário onde o código contém um bug, fazendo com que o pipeline Ci/CD falhe e rejeite o merge na main.

#### Passo 1

Vamos simular que a função de atualizar o e-mail do usuário está retornando um valor diferente do esperado. Para isso, edite o arquivo /backend/src/services/user/update-user-spec.ts e:
- Comente a linha 92.
- Logo abaixo, insira:
```diff
expect(result).toEqual(Success.create({ ...updatedUser, email: 'email-errado@mail.com' }))
````

![image](https://github.com/user-attachments/assets/05cc58db-d6c9-4168-aa5e-16a4f34a3c16)
![image](https://github.com/user-attachments/assets/43298bfd-31fe-4732-a44b-321085fa0112)


#### Passo 2

Após modificar o código, você deve criar uma novo branch, realizar um `commit` e um `push`:

```bash
git checkout -b bug
git add --all
git commit -m "Alterando o arquivo update-user-spec.ts"
git push origin bug
```

#### Passo 3

Em seguida, crie um Pull Request (PR) com suas modificações. Para isso, acesse seu repositorio, você verá uma mensagem sugerindo criar um Pull Request da branch 'Bug'. Clique em **Compare & pull request**. Se não aparecer, vá na aba **Pull requests** e clique em **New pull request**

Selecione a **Base repository: <USER>/adote-facil**, em seguida **base: main** e **compare: bug** e crie um Pull Request. Nessa página, você poderá revisar as alterações feitas e insirir uma breve descrição do PR e confirme a criação clicando novamente em Create pull request.

Assim que o PR for criado, o pipeline configurado no arquivo experimento-ci-cd.yml será automaticamente iniciada pelo GitHub Actions. Como introduzimos um bug, os testes irão falhar — essa falha será exibida na tela de execução do pipeline. Você pode acompanhar o status da execução acessando a aba Actions do seu repositório.

Em suma, o servidor de CI/CD detectou automaticamente um problema no código enviado, impedindo que ele seja integrado ao branch principal do projeto.

## Tarefa #3: Criando um Pull Request (PR) com a correção

Para restaurar o arquivo ao seu estado original, descomente a linha 92 e exclua a linha 93. Assim, quando criarmos um novo PR, os testes serão executados com sucesso, sem apresentar falhas.

Após modificar o código, salve o arquivo e crie um novo branch para consertar o bug, realize um `add`, um `commit` e um `push`:

```bash
git checkout -b fixture
git add --all
git commit -m "Consertando a função Test"
git push origin fixture
```
Insira seu nome de usuário e senha (Token) do GH se for requerido.

Em seguida, crie novamente um Pull Request (PR) com a correção. Para isso, acesse seu repositorio, você verá uma mensagem sugerindo criar um Pull Request da branch 'Bug'. Clique em **Compare & pull request**. Se não aparecer, vá na aba **Pull requests** e clique em **New pull request**

Selecione a **Base repository: <USER>/adote-facil**, em seguida **base: main** e **compare: bug** e crie um Pull Request. Nessa página, você poderá revisar as alterações feitas e insirir uma breve descrição do PR e confirme a criação clicando novamente em Create pull request.

Você pode acompanhar o andamento da pipeline acessando a aba Actions do repositório e selecionando o nome do PR em execução.

Após a criação do PR, o GitHub Actions iniciará automaticamente o pipeline, que executará os testes, realizará o build e fará a entrega do artefato gerando um arquivo .zip do projeto. Quando concluído, o arquivo .zip estará disponível para download.

# FIM



