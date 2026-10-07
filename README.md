# 🎁 Friend Secret API

API REST para gerenciamento e realização de **Amigo Secreto**, desenvolvida com **Node.js, TypeScript, Express, Prisma e PostgreSQL**.

A aplicação permite criar e administrar eventos de Amigo Secreto, organizar participantes em grupos, realizar o sorteio automaticamente e disponibilizar uma consulta pública para que cada participante descubra quem foi sorteado.

O projeto foi desenvolvido com foco em uma arquitetura simples e organizada, separando **rotas, controllers, services e persistência de dados**.

---

## ✨ Funcionalidades

### 🎉 Eventos

* Criar eventos de Amigo Secreto
* Consultar eventos
* Atualizar eventos
* Excluir eventos
* Ativar/desativar um evento
* Realizar o sorteio dos participantes

### 👥 Grupos

Os participantes podem ser organizados em grupos.

* Criar grupos dentro de um evento
* Listar grupos
* Atualizar grupos
* Excluir grupos
* Associar participantes aos grupos

O sistema possui uma opção de sorteio **por grupos**, impedindo que um participante sorteie alguém pertencente ao mesmo grupo.

### 👤 Participantes

* Adicionar participantes
* Consultar participantes
* Atualizar participantes
* Remover participantes
* Associar participantes a grupos
* Consultar participante através do CPF

### 🎲 Sorteio

O sistema realiza o sorteio automaticamente quando o evento é ativado.

Durante o sorteio, existem regras para:

* Impedir que uma pessoa sorteie a si mesma
* Evitar sorteios inválidos
* Respeitar a separação entre grupos quando o evento estiver configurado para isso
* Repetir as tentativas caso a combinação gerada seja inválida

O resultado do sorteio é armazenado no participante e posteriormente utilizado para a consulta pública.

### 🔐 Autenticação administrativa

As rotas administrativas são protegidas por autenticação.

O acesso ao painel administrativo utiliza:

* Login através de senha
* Token de autenticação
* Middleware de validação
* Variável de ambiente para o token base

As operações de gerenciamento de eventos, grupos e participantes exigem autenticação.

### 🔎 Consulta pública

Os participantes não precisam acessar o painel administrativo para descobrir o resultado.

A API disponibiliza uma rota pública que permite consultar um participante através do **CPF** e retornar o respectivo amigo secreto após o sorteio.

---

## 🏗️ Arquitetura

A aplicação segue uma organização baseada em responsabilidades:

```text
src/
├── controllers/
│   ├── auth.ts
│   ├── events.ts
│   ├── groups.ts
│   └── people.ts
│
├── routes/
│   ├── admin.ts
│   └── site.ts
│
├── services/
│   ├── auth.ts
│   ├── events.ts
│   ├── groups.ts
│   └── people.ts
│
├── utils/
│   ├── match.ts
│   ├── requestIntercepter.ts
│   └── ...
│
└── server.ts
```

O `server.ts` inicializa o Express, configura CORS e parsing de JSON/formulários e registra as rotas públicas e administrativas.

---

## 🧰 Tecnologias

### Backend

* **Node.js**
* **TypeScript**
* **Express**
* **Prisma ORM**
* **PostgreSQL**
* **Zod**
* **CORS**
* **dotenv**
* **Nodemon**

As dependências e scripts utilizados pelo projeto estão definidos no `package.json`.

### Banco de dados

O projeto utiliza **PostgreSQL** através do Prisma.

O modelo principal é dividido em três entidades:

```text
Event
  │
  ├── EventGroup
  │       │
  │       └── EventPeople
  │
  └── EventPeople
```

O schema possui os modelos `Event`, `EventGroup` e `EventPeople`, permitindo relacionar eventos, grupos e participantes.

---

## 🗄️ Modelo de dados

### Event

Representa um evento de Amigo Secreto.

Principais campos:

```text
id
status
title
description
grouped
```

O campo `grouped` determina se o sorteio deverá respeitar a separação entre grupos.

### EventGroup

Representa um grupo dentro de um evento.

```text
id
id_event
name
```

### EventPeople

Representa um participante.

```text
id
id_event
id_group
name
cpf
matched
```

O campo `matched` armazena a referência ao participante sorteado.

---

## 🔄 Fluxo da aplicação

O funcionamento principal pode ser representado da seguinte forma:

```text
                    ┌──────────────┐
                    │ Administrador│
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    Login     │
                    └──────┬───────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ Criar evento      │
                 │ Criar grupos      │
                 │ Adicionar pessoas │
                 └─────────┬─────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Sorteio    │
                    └──────┬───────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ Resultado salvo  │
                 │ no banco de dados│
                 └─────────┬─────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Participante │
                    │ consulta CPF │
                    └──────────────┘
```

---

# 🚀 Instalação

## 📋 Pré-requisitos

Antes de executar o projeto, você precisa ter instalado:

* [Node.js](https://nodejs.org/) 20+
* PostgreSQL
* npm

Você também precisará configurar as variáveis de ambiente utilizadas pela aplicação.

---

## 📥 Clone o repositório

```bash
git clone https://github.com/luan-junior/friend-secret-public.git
```

Entre no diretório:

```bash
cd friend-secret-public
```

---

## 📦 Instale as dependências

```bash
npm install
```

---

## 🔐 Configure as variáveis de ambiente

O projeto possui um arquivo `.env.example` com as variáveis necessárias.

Crie o arquivo `.env`:

```bash
cp .env.example .env
```

Configure os valores:

```env
NODE_ENV=development
PORT=9000

DATABASE_URL_POSTGRES="postgresql://USER:PASSWORD@localhost:5432/DATABASE"

DATABASE_URL_MYSQL=""

DEFAULT_TOKEN="seu_token"

SSL_KEY=""
SSL_CERT=""
```

> Para o funcionamento atual do Prisma, a variável `DATABASE_URL_POSTGRES` é a utilizada pelo datasource PostgreSQL.

---

# 🗃️ Banco de dados

Depois de configurar o PostgreSQL e o `.env`, execute as migrations:

```bash
npm run migrate:dev
```

Para aplicar migrations em um ambiente de produção:

```bash
npm run migrate:deploy
```

Os comandos de migration estão configurados diretamente no `package.json`.

---

# ▶️ Executando o projeto

## Desenvolvimento

Execute:

```bash
npm run dev
```

O servidor será iniciado utilizando a configuração do Nodemon.

Por padrão, quando nenhuma porta é informada, a aplicação utiliza a porta:

```text
9000
```

Esse comportamento está definido no servidor.

A API estará disponível em:

```text
http://localhost:9000
```

---

## 🏭 Produção

Primeiro compile o TypeScript:

```bash
npm run build
```

Depois execute:

```bash
npm start
```

O projeto gera o JavaScript compilado no diretório `build` e inicia o servidor através de:

```text
build/server.js
```

---

# 🔌 API

## Rotas públicas

### Health check

```http
GET /ping
```

Retorno:

```json
{
  "pong": true
}
```

### Consultar evento

```http
GET /events/:id
```

### Consultar participante

```http
GET /events/:id_event/search?cpf=CPF
```

Essa rota permite que o participante consulte seu resultado através do CPF após o sorteio.

---

# 🔒 Rotas administrativas

As rotas administrativas ficam agrupadas sob:

```text
/admin
```

### Autenticação

```http
POST /admin/login
```

Body:

```json
{
  "password": "sua_senha"
}
```

A API retorna um token que deverá ser utilizado nas requisições administrativas.

---

## Eventos

```http
GET    /admin/events
GET    /admin/events/:id
POST   /admin/events
PUT    /admin/events/:id
DELETE /admin/events/:id
```

---

## Grupos

```http
GET    /admin/events/:id_event/groups
GET    /admin/events/:id_event/groups/:id
POST   /admin/events/:id_event/groups
PUT    /admin/events/:id_event/groups/:id
DELETE /admin/events/:id_event/groups/:id
```

---

## Participantes

```http
GET    /admin/events/:id_event/groups/:id_group/people
GET    /admin/events/:id_event/groups/:id_group/people/:id
POST   /admin/events/:id_event/groups/:id_group/people
PUT    /admin/events/:id_event/groups/:id_group/people/:id
DELETE /admin/events/:id_event/groups/:id_group/people/:id
```

As rotas administrativas de eventos, grupos e participantes são protegidas pelo middleware de autenticação.

---

# 🎲 Algoritmo de sorteio

O sorteio é realizado pelo serviço de eventos.

O algoritmo cria combinações aleatórias entre os participantes e verifica se a combinação é válida.

Entre as regras aplicadas estão:

* Um participante não pode sortear a si próprio.
* Cada participante recebe um único participante.
* Quando o evento utiliza grupos, participantes do mesmo grupo não podem ser sorteados entre si.
* Caso uma combinação inválida seja encontrada, o algoritmo realiza novas tentativas.
* Caso não seja possível encontrar uma combinação válida, o sorteio é rejeitado.

Após um sorteio válido, o resultado é persistido no banco de dados.

---

# 🛡️ Validação de dados

A API utiliza **Zod** para validar os dados recebidos nas requisições.

Exemplo de criação de evento:

```typescript
const addEventSchema = z.object({
  title: z.string(),
  description: z.string(),
  grouped: z.boolean(),
});
```

Também existem validações para participantes e grupos antes que os dados sejam enviados ao banco.

---

# 🔐 HTTPS

Em ambiente de produção, a aplicação possui suporte para execução utilizando HTTPS.

As configurações de certificado são obtidas através das variáveis:

```env
SSL_KEY=
SSL_CERT=
```

Quando `NODE_ENV` está configurado como `production`, o servidor cria uma instância HTTP e outra HTTPS.

---

# 📂 Estrutura geral

```text
friend-secret-public/
│
├── prisma/
│   └── schema.prisma
│
├── src/
│   ├── controllers/
│   │   ├── auth.ts
│   │   ├── events.ts
│   │   ├── groups.ts
│   │   └── people.ts
│   │
│   ├── routes/
│   │   ├── admin.ts
│   │   └── site.ts
│   │
│   ├── services/
│   │   ├── auth.ts
│   │   ├── events.ts
│   │   ├── groups.ts
│   │   └── people.ts
│   │
│   ├── utils/
│   │   └── ...
│   │
│   └── server.ts
│
├── .env.example
├── .gitignore
├── nodemon.json
├── package.json
├── package-lock.json
└── tsconfig.json
```

---

# 📌 Scripts disponíveis

| Comando                  | Descrição                                 |
| ------------------------ | ----------------------------------------- |
| `npm run dev`            | Executa a API em desenvolvimento          |
| `npm run build`          | Compila o TypeScript                      |
| `npm start`              | Executa a versão compilada                |
| `npm run migrate:dev`    | Cria/aplica migrations em desenvolvimento |
| `npm run migrate:deploy` | Aplica migrations em produção             |

---

# 🎯 Objetivo do projeto

O **Friend Secret API** foi desenvolvido para praticar e demonstrar conceitos de desenvolvimento backend utilizando o ecossistema Node.js.

Entre os principais conceitos trabalhados estão:

* Desenvolvimento de APIs REST
* TypeScript
* Express
* Arquitetura baseada em Controllers e Services
* ORM com Prisma
* PostgreSQL
* Relacionamentos entre entidades
* Autenticação
* Validação de dados
* Variáveis de ambiente
* Migrations
* Algoritmos de sorteio
* Regras de negócio
* Separação entre endpoints públicos e administrativos

---

# 🚧 Possíveis melhorias

Algumas evoluções que podem ser implementadas futuramente:

* [x] JWT para autenticação
* [x] Hash seguro para senhas
* [x] Sistema de usuários administrativos
* [x] Expiração de tokens
* [x] Rate limiting
* [x] Testes unitários
* [x] Testes de integração
* [x] Documentação com Swagger/OpenAPI
* [x] Docker e Docker Compose
* [x] Logs estruturados
* [x] Tratamento centralizado de erros
* [x] Frontend para gerenciamento dos eventos
* [x] Sistema de envio do resultado por e-mail ou WhatsApp
* [x] Melhorias no algoritmo de sorteio
* [x] CI/CD

---

# 👨‍💻 Autor

**Luan Junior**

Desenvolvedor Full Stack com experiência em **React, Next.js, Node.js e TypeScript**.

---

## 📄 Licença

Este projeto está disponível sob a licença definida no repositório.
