# Titulo

Este repositório descreve um roteiro prático para configuração e uso de um **Servidor de Integração Contínua**. O Objetivo é simular práticas reais do DevOps no contexto de desenvolvimento Web.

O GitHub Actions é uma ferramenta que possibilita automatizar fluxos de trabalhos (workflows) de todo software. Neste tutorial, aplicaremos os princípios de CI/CD para testes, build e deploys automatiados, em um contexto real de produção.

## 1. Configurar o GitHub Actions

### 1.1

Crie um Token pessoal no GitHub (i) e;  faça um Fork (ii) deste projeto. (i) Para criar um Token, clique no ícone do seu perfil, vá em **Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**. Marque as opções `repo` e `workflows` para gerar o Token. Gere o token mínimo (7 dias) apenas para este experimento. Copie e guarde esse código (token) para usar quando o GitHub lhe pedir para usar uma senha (password). (ii) Entre no repositório [adote-facil](https://github.com/ArthurEnrique15/adote-facil) e clique no botão **Fork** no canto superior direito na página do projeto no GitHub.

#### Opcional, mas recomendado

Leia o README do projeto adote-facil e siga o tutorial de implantação para conhecer e entender o projeto em questão.

### 1.2

Clone o repositório para sua máquina local, usando o seguinte comando (onde `<USER>` deve ser substituído pelo seu usuário no GitHub):
```bash
git clone https://github.com/<USER>/adote-facil.git
```

Crie em seu diretório clonado, uma pasta `.github`, uma pasta `workflows` dentro de `.github`, e um arquivo `nome.yml` dentro de `workflows`. Se tudo estiver correto terá um caminho `.github/workflows/nome.yml`. Em seu arquivo `nome.yml` copie e cole o seguinte código em seu arquivo .yml:

```yml
name: ci-cd

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
      POSTGRES_USER:
      POSTGRES_PASSWORD:
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
          sleep 10  # aguardar containers inicializarem
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

### 1.3

Entre no diretório criado (use o comando cd no terminal) ".../adote-facil/¨ e dê um commit nos arquivos criados:

```bash
git add .
git commit -m "Configurando o GitHub Actions"
git push origin main
```

## 2. Configurando Secrets

Em diversos sistemas, teremos variáveis com informações confidenciais, para mantermos a privacidade delas, o GitHub Actions, possibilita o uso de Secrets.
Os Secrets são variáveis de ambiente criptografadas que permitem armazenar informações sensíveis, como senhas, chaves de API, etc. de forma segura dentro do GitHub. Em nosso contexto, usaremos o Secrets para inicializar o Banco de Dados.

### 2.1

Em seu repósitorio adote-facil no GitHub, clique no botão **Settings**, em seguida clique em **Secrets and variables** -> **Actions**. Selecione a opção **New repository secret**.


### Criar Pull Request com bug

criar branch + criar bug + pull request + vizualizar ações do github actions

### Criar Pull Request com correção

corrigir bug + pull request + vizualizar ações do github actions

### Acessar arquivo criado com o yaml

baixar arquivo + acessa-lo

