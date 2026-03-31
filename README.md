# GymPass API

API REST no estilo GymPass construída com Node.js, Fastify, Prisma e TypeScript, seguindo os princípios SOLID e arquitetura em camadas.

## Tecnologias

- **[Node.js](https://nodejs.org/)** — Runtime JavaScript
- **[Fastify](https://fastify.dev/)** — Framework HTTP de alta performance
- **[TypeScript](https://www.typescriptlang.org/)** — Tipagem estática
- **[Prisma](https://www.prisma.io/)** — ORM para PostgreSQL
- **[PostgreSQL](https://www.postgresql.org/)** — Banco de dados relacional
- **[Docker](https://www.docker.com/)** — Containerização do banco de dados
- **[Zod](https://zod.dev/)** — Validação de schemas
- **[JWT](https://jwt.io/)** + **Refresh Token** — Autenticação stateless
- **[bcryptjs](https://github.com/dcodeIO/bcrypt.js)** — Hash de senhas
- **[Vitest](https://vitest.dev/)** — Testes unitários e E2E

---

## Requisitos

### RF (Requisitos Funcionais)

- [x] Deve ser possível se cadastrar;
- [x] Deve ser possível se autenticar;
- [x] Deve ser possível obter o perfil de um usuário logado;
- [x] Deve ser possível obter o número de check-ins realizados pelo usuário logado;
- [x] Deve ser possível o usuário obter o seu histórico de check-ins;
- [x] Deve ser possível o usuário buscar academias próximas (até 10km);
- [x] Deve ser possível o usuário buscar academias pelo nome;
- [x] Deve ser possível o usuário realizar check-in em uma academia;
- [x] Deve ser possível validar o check-in de um usuário;
- [x] Deve ser possível cadastrar uma academia;

### RN (Regras de Negócio)

- [x] O usuário não deve poder se cadastrar com um e-mail duplicado;
- [x] O usuário não pode fazer 2 check-ins no mesmo dia;
- [x] O usuário não pode fazer check-in se não estiver perto (100m) da academia;
- [x] O check-in só pode ser validado até 20 minutos após ser criado;
- [x] O check-in só pode ser validado por administradores;
- [x] A academia só pode ser cadastrada por administradores;

### RNF (Requisitos Não-Funcionais)

- [x] A senha do usuário precisa estar criptografada;
- [x] Os dados da aplicação precisam estar persistidos em um banco PostgreSQL;
- [x] Todas listas de dados precisam estar paginadas com 20 itens por página;
- [x] O usuário deve ser identificado por um JWT (JSON Web Token);

---

## Rotas da API

### Usuários

| Método  | Rota             | Descrição                                       | Auth |
| ------- | ---------------- | ----------------------------------------------- | ---- |
| `POST`  | `/users`         | Cadastrar novo usuário                          | —    |
| `POST`  | `/sessions`      | Autenticar (login)                              | —    |
| `PATCH` | `/token/refresh` | Renovar access token via refresh token (cookie) | —    |
| `GET`   | `/me`            | Obter perfil do usuário autenticado             | JWT  |

### Academias

| Método | Rota                                | Descrição                        | Auth        |
| ------ | ----------------------------------- | -------------------------------- | ----------- |
| `POST` | `/gyms`                             | Cadastrar academia               | JWT (ADMIN) |
| `GET`  | `/gyms/search?q=&page=`             | Buscar academias pelo nome       | JWT         |
| `GET`  | `/gyms/nearby?latitude=&longitude=` | Buscar academias próximas (10km) | JWT         |

### Check-ins

| Método  | Rota                             | Descrição                         | Auth        |
| ------- | -------------------------------- | --------------------------------- | ----------- |
| `POST`  | `/gyms/:gymId/check-ins`         | Realizar check-in em uma academia | JWT         |
| `GET`   | `/check-ins/history?page=`       | Histórico de check-ins do usuário | JWT         |
| `GET`   | `/check-ins/metrics`             | Total de check-ins do usuário     | JWT         |
| `PATCH` | `/check-ins/:checkInId/validate` | Validar check-in                  | JWT (ADMIN) |

---

## Arquitetura

O projeto segue os princípios SOLID com separação em camadas:

```
src/
├── http/
│   ├── controllers/       # Camada de apresentação (rotas + validação de entrada)
│   │   ├── users/
│   │   ├── gyms/
│   │   └── check-ins/
│   └── middlewares/       # Middlewares (ex: verifyJWT, verifyUserRole)
├── use-cases/             # Camada de domínio (regras de negócio)
│   ├── factories/         # Factory functions para instanciar use-cases
│   └── errors/            # Erros de domínio tipados
├── repositories/          # Interfaces e implementações de repositórios
│   ├── prisma/            # Implementação com Prisma (produção)
│   └── in-memory/         # Implementação em memória (testes unitários)
├── lib/
│   └── prisma.ts          # Instância global do Prisma Client
└── env/
    └── index.ts           # Validação das variáveis de ambiente com Zod
```

### Autenticação

A API usa dois tokens:

- **Access Token (JWT)** — expira em 10 minutos, enviado no header `Authorization: Bearer <token>`
- **Refresh Token** — expira em 7 dias, armazenado como cookie `httpOnly`

---

## Como executar

### Pré-requisitos

- [Node.js](https://nodejs.org/) v18+
- [Docker](https://www.docker.com/) e Docker Compose

### Passo a passo

1. Clone o repositório:

```bash
git clone <url-do-repositorio>
cd 03-api-solid
```

2. Instale as dependências:

```bash
npm install
```

3. Configure as variáveis de ambiente criando um arquivo `.env` na raiz:

```env
NODE_ENV=dev
PORT=3333
DATABASE_URL="postgresql://docker:docker@localhost:5432/apisolid?schema=public"
JWT_SECRET=sua-chave-secreta
```

4. Suba o banco de dados com Docker:

```bash
docker compose up -d
```

5. Execute as migrations do banco:

```bash
npx prisma migrate dev
```

6. Inicie o servidor em modo de desenvolvimento:

```bash
npm run start:dev
```

A API estará disponível em `http://localhost:3333`.

---

## Testes

O projeto possui dois tipos de teste:

### Testes Unitários

Testam os use-cases de forma isolada, usando repositórios in-memory (sem banco de dados).

```bash
# Executar uma vez
npm test

# Modo watch
npm run test:watch
```

### Testes E2E

Testam os controllers com requisições HTTP reais e banco de dados isolado por ambiente de teste.

```bash
# Executar uma vez
npm run test:e2e

# Modo watch
npm run test:e2e:watch
```

### Cobertura de Testes

```bash
npm run test:coverage
```

### Interface Visual

```bash
npm run test:ui
```

---

## Scripts disponíveis

| Script                  | Descrição                                     |
| ----------------------- | --------------------------------------------- |
| `npm run start:dev`     | Inicia em modo desenvolvimento com hot-reload |
| `npm start`             | Inicia a versão de produção (build)           |
| `npm run build`         | Compila o projeto com `tsup`                  |
| `npm test`              | Executa os testes unitários                   |
| `npm run test:e2e`      | Executa os testes E2E                         |
| `npm run test:coverage` | Gera relatório de cobertura                   |
| `npm run test:ui`       | Abre interface visual do Vitest               |
