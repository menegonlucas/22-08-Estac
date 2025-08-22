# 🚀 Aula 04 - Deploy (Implantação) de API Estacionamento

Implantação de aplicações **Web/API Back-end** com a [Vercel](https://vercel.com/).  
Para front-end, utilizamos o **GitHub Pages**. Para o back-end, usaremos a Vercel, que suporta **Node.js**, **Prisma** e **Postgres**.  
> **Observação:** O serviço é gratuito até um limite de uso.

---

## 🛠️ Ambiente Necessário

- [Visual Studio Code](https://code.visualstudio.com/)
- [Node.js](https://nodejs.org/)
- [Prisma](https://www.prisma.io/)
- [Postgres](https://www.postgresql.org/)
- [Vercel](https://vercel.com/)

### 📑 Contas Necessárias

- [GitHub](https://github.com/)
- [Vercel](https://vercel.com/)

---

## 1️⃣ Criando o Projeto Node.js + Prisma

### Estrutura do Projeto

- Crie uma pasta chamada `estacionamentoSeuNome` e abra no **VSCode**.
- Inicie um novo projeto Node.js:

```bash
npm init -y
```

- Edite o `package.json` conforme necessário:

```json
{
  "name": "estacionamentoapi",
  "version": "1.0.0",
  "main": "api/server.js",
  "scripts": {
    "dev": "npx nodemon api/server.js"
  },
  "author": "SeuNome",
  "license": "ISC",
  "description": "Projeto de Estacionamento API, para aulas de Node.js"
}
```

- Instale as dependências principais:

```bash
npm install express cors dotenv prisma
```

- Inicie o Prisma com Postgres:

```bash
npx prisma init --datasource-provider postgresql
```

### Modelo de Dados (`prisma/schema.prisma`)

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Veiculo {
  placa        String    @id
  tipo         Tipo
  proprietario String
  modelo       String
  marca        String
  cor          String?
  ano          Int?
  telefone     String
  estadias     Estadia[]
}

enum Tipo {
  CARRO
  MOTO
  VAN
  CAMINHAO
  ONIBUS
}

model Estadia {
  id         Int       @id @default(autoincrement())
  placa      String?
  entrada    DateTime  @default(now())
  saida      DateTime?
  valorHora  Float
  valorTotal Float?
  automovel  Veiculo?  @relation(fields: [placa], references: [placa], onUpdate: Cascade, onDelete: SetNull)
}
```

### Estrutura de Pastas

![Diagrama de Classes](./screenshots/uml-dc.png)  
![Pastas](./screenshots/pastas.png)

---

## 2️⃣ Código Base

### `api/server.js`

```js
const express = require('express');
const cors = require('cors');
const routes = require('../src/routes');

const port = process.env.PORT || 3001;
const app = express();
app.use(express.json());
app.use(cors());
app.use(routes);

app.listen(port, () => {
    console.log('API respondendo em http://localhost:' + port)
});
```

### `src/routes.js`

```js
const express = require('express');
const routes = express.Router();

const Veiculo = require('./controllers/veiculo');
const Estadia = require('./controllers/estadia');

routes.get('/', (req, res) => {
    const api = {
        titulo: 'API Estacionamento',
        versao: '1.0.0',
        rotas: [
            { metodo: 'GET', caminho: '/veiculos' },
            { metodo: 'GET', caminho: '/veiculos/:placa' },
            { metodo: 'POST', caminho: '/veiculos' },
            { metodo: 'PATCH', caminho: '/veiculos/:placa' },
            { metodo: 'DELETE', caminho: '/veiculos/:placa' },
            { metodo: 'GET', caminho: '/estadias' },
            { metodo: 'GET', caminho: '/estadias/:placa' },
            { metodo: 'POST', caminho: '/estadias' },
            { metodo: 'PATCH', caminho: '/estadias/:id' },
            { metodo: 'DELETE', caminho: '/estadias/:id' }
        ]
    }
    res.json(api);
});

routes.get('/veiculos', Veiculo.read);
routes.get('/veiculos/:placa', Veiculo.read);
routes.post('/veiculos', Veiculo.create);
routes.patch('/veiculos/:placa', Veiculo.update);
routes.delete('/veiculos/:placa', Veiculo.del);

routes.get('/estadias', Estadia.read);
routes.get('/estadias/:placa', Estadia.read);
routes.post('/estadias', Estadia.create);
routes.patch('/estadias/:id', Estadia.update);
routes.delete('/estadias/:id', Estadia.del);

module.exports = routes;
```

---

## 3️⃣ Testando Localmente

- Altere o banco de dados para MySQL em `prisma/schema.prisma`:

```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}
```

- Configure o `.env`:

```env
DATABASE_URL="mysql://root@localhost:3306/estacionamentoapi?schema=public&timezone=UTC"
```

- Execute as migrações e rode o projeto:

```bash
npm install @prisma/client
npx prisma migrate dev --name init
npm run dev
```

- Para abrir o Prisma Studio:

```bash
npx prisma studio
```

---

## 4️⃣ Deploy na Vercel

### Configuração

- Instale o CLI da Vercel:

```bash
npm i -g vercel@latest
```

- Link o projeto e baixe as variáveis de ambiente:

```bash
vercel link
vercel env pull .env
```

- Atualize o `package.json`:

```json
{
  "scripts": {
    "dev": "npx nodemon api/server.js",
    "postinstall": "prisma migrate dev --name init && prisma generate"
  },
  "dependencies": {
    "@prisma/client": "^6.14.0",
    "cors": "^2.8.5",
    "dotenv": "^17.2.1",
    "express": "^5.1.0",
    "prisma": "^6.14.0"
  },
  "devDependencies": {
    "prisma": "^6.14.0"
  }
}
```

- Crie o arquivo `vercel.json`:

```json
{
    "version": 2,
    "rewrites": [
        {
            "source": "/(.*)",
            "destination": "/api/server.js"
        }
    ]
}
```

- Faça o deploy:

```bash
vercel --prod
```

---

## 5️⃣ Banco de Dados na Vercel

- Crie um serviço de banco de dados com Prisma (Neon) na Vercel.
- Siga as telas abaixo para configurar:

![Prisma 1](./screenshots/neon1.png)  
![Prisma 2](./screenshots/neon2.png)

---

## 6️⃣ GitHub e .gitignore

- Crie um repositório no GitHub e envie o projeto.
- Adicione um `.gitignore`:

```
node_modules
.env
/generated/prisma
/prisma/migrations
```

---

## ✅ Atividades

Em grupos, elejam um **DevOps** para implantar a API na Vercel.  
Outro integrante deve criar a **UI** e publicar no **GitHub Pages**.

### [Formulário de Entregas](https://docs.google.com/forms/d/e/1FAIpQLSdV24fB9faivuHKaluB1J8EgxOYZCR63u1IIQQ72q7mWm0rGg/viewform?usp=dialog)

Preencha com:
- Link da API implantada na Vercel
- Link da UI no GitHub Pages

---
