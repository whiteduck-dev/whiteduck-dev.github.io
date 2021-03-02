---
layout: post
title: "Recriando a interface do Twitter utilizando TypeScript, Prisma e Next.js."
author: Bruno Viganó
categories: [Next.js, Twitter, TypeScript]
tags: [Next.js, Twitter, TypeScript]
image: assets/images/twitter-clone/twitter-blog.jpeg
description: "Utilizando Next.js para recriar a interface do twitter."
featured: true
hidden: true
---

## Introdução

Ao final, nossa aplicação contará com os seguintes recursos:

- autenticação utilizando NextAuth e Twitter OAuth
- opção para publicar um novo tweet
- opção para visualizar uma lista de tweets
- opção para visualizar o perfil de um usuário e seus respectivos tweets

Você poderá encontrar o código final para nossa aplicação no GitHub. //TODO

#### Preparação

Next.js é um dos frameworks React.js mais populares. Ele disponibiliza um leque de recursos interessantes como server side rendering, suporte a TypeScript, otimização de imagens, suporte a i18n, criação de rotas baseada na estrutura de arquivos do sistema e muito mais.

Prisma é um ORM para Node.js e TypeScript. Ele também disponibiliza vários recursos como //TODO

#### Requerimentos

Precisaremos dos seguintes recursos para rodar o app:

- Docker
- npm
- yarn
- git

Estas tecnologias serão utilizadas no app:

- Next.js: para construir o app
- Prisma: para buscar e persistir dados na nossa base de dados
- Chakra UI: para fazer a estilização
- NextAuth: para lidar com questões de autenticação
- React Query: para buscar e atualizar dados na nossa aplicação

#### Criando uma nova aplicação Next.js

Chegou a hora de colocar a mão no código. Vamos começar criando um novo app Next executando o seguinte comando do nosso terminal:

`yarn create next-app`

Precisamos informar o nome do app quando solicitado. Se preferir poderá renomeá-lo quando quiser. De qualquer forma chamarei de twitter-clone. Você conseguirá visualizar algo parecido no seu terminal:

```bash
$ yarn create next-app

yarn create v1.22.5
[1/4] 🔍  Resolving packages...
[2/4] 🚚  Fetching packages...
[3/4] 🔗  Linking dependencies...
[4/4] 🔨  Building fresh packages...

success Installed "create-next-app@10.0.7" with binaries:
      - create-next-app
✔ What is your project named? twitter-clone
Creating a new Next.js app in /home/bruno/development/twitter-clone.

....

Initialized a git repository.

Success! Created twitter-clone at /twitter-clone
Inside that directory, you can run several commands:

  yarn dev
    Starts the development server.

  yarn build
    Builds the app for production.

  yarn start
    Runs the built app in production mode.

We suggest that you begin by typing:

  cd twitter-clone
  yarn dev
```

Estamos prontos para acessar a pasta twitter-clone e iniciar nosso app executando o comando:

```bash
cd twitter-clone && yarn dev
```

Nosso app Next.js deve estar acessível em http://localhost:3000. Acessando este link devemos ver a seguinte tela:

![](../assets/images/twitter-clone/next-localhost-home.png)

#### Adicionando uma base de dados PostgreSQL com Docker

Vamos utilizar o docker para adicionar trabalhar com um banco de dados Postgre, assim poderemos armazenar os usuários e seus respectivos tweets. Podemos criar um novo arquivo `docker-compose.yml` na raíz do projeto com o seguinte conteúdo:

```Docker
version: "3"

services:
  db:
    container_name: db
    image: postgres:11.3-alpine
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  db_data:
```

Se o Docker estiver rodando no seu computador, poderemos então partir para o seguinte comando para inicializar nosso container PostgreSQL:

```docker
docker-compose up
```

Este comando será responsável por iniciar o container PostgreSQL que ficará disponível através do caminho `postgresql://postgres:@localhost:5432/postgres`. Lembrando que se você preferir pode utilizar uma instalação local do Postgres no lugar do Docker.

#### Adicionando o Chackra UI

Chakra UI é uma biblioteca simples de React.js. Ela é muito popular e possui diversos recursos como acessibilidade, suporte para modo noturno e muito mais. Neste tutorial usaremos o Chakra UI para estilizar nossa interface. Podemos instalar o pacote rodando o comando a seguir:

```bash
yarn add @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

Vamos agora na pasta pages renomear nosso `_app.js` para `_app.tsx` e substituir seu conteúdo pelo trecho a seguir:

```node
// pages/_app.tsx

import { ChakraProvider } from "@chakra-ui/react";
import { AppProps } from "next/app";
import Head from "next/head";
import React from "react";

const App = ({ Component, pageProps }: AppProps) => {
  return (
    <>
      <Head>
        <link rel="shortcut icon" href="/images/favicon.ico" />
      </Head>
      <ChakraProvider>
        <Component {...pageProps} />
      </ChakraProvider>
    </>
  );
};

export default App;
```

Após adicionar um novo arquivo TypeScript, precisamos reiniciar nosso servidor Next.js. Uma vez reiniciado iremos no deparar com o seguinte erro:

```bash
❯ yarn dev                                                              
yarn run v1.22.5
$ next dev
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
It looks like you're trying to use TypeScript but do not have the required package(s) installed.

Please install typescript, @types/react, and @types/node by running:

yarn add --dev typescript @types/react @types/node

If you are not trying to use TypeScript, please remove the tsconfig.json file from your package root (and any TypeScript files in your pages directory).
```

Isso acontece pois adicionamos no projeto um novo arquivo TypeScript porém não adicionamos as suas depedências que são necessárias para executá-lo. Como sugere a mensagem devemos utilizar nosso gerenciador de pacotes para instalar o typescript e seus types:

```bash
yarn add --dev typescript @types/react @types/node
```

Agora sim podemos inicializar o projeto sem maiores problemas:

```bash
$ yarn dev

yarn run v1.22.5
$ next dev
ready - started server on http://localhost:3000
We detected TypeScript in your project and created a tsconfig.json file for you.

event - compiled successfully
```

#### Adicionando NextAuth

NextAuth é uma biblioteca de autenticação para Next.js. Vamo configurá-la em nosso app rodando o seguinte comando da raíz do nosso projeto:

```bash
yarn add next-auth
```

O próximo passo é atualizar nosso arquivo `pages/_app.tsx` importando a dependência recém instalada e envolvendo nosso app com o  `NextAuthProvider`. Ficando desta forma:

```node
// pages/_app.tsx

import { ChakraProvider } from "@chakra-ui/react";
import { Provider as NextAuthProvider } from "next-auth/client";
import { AppProps } from "next/app";
import Head from "next/head";
import React from "react";

const App = ({ Component, pageProps }: AppProps) => {
  return (
    <>
      <Head>
        <link rel="shortcut icon" href="/images/favicon.ico" />
      </Head>
      <NextAuthProvider session={pageProps.session}>
        <ChakraProvider>
          <Component {...pageProps} />
        </ChakraProvider>
      </NextAuthProvider>
    </>
  );
};

export default App;
```

Precisamos agora criar um novo arquivo chamado `[...nextauth].ts` dentro do diretório `pages/api/auth` com o seguinte conteúdo:

```node
// pages/api/auth/[...nextauth].ts

import { NextApiRequest, NextApiResponse } from "next";
import NextAuth from "next-auth";
import Providers from "next-auth/providers";

const options = {
  providers: [
    Providers.Twitter({
      clientId: process.env.TWITTER_KEY,
      clientSecret: process.env.TWITTER_SECRET,
    }),
  ],
};

export default NextAuth(options);
```

Esse arquivo será responsável por gerenciar nossa autenticação utilizando o sistema de rotas do Next.js. Na sequência vamos criar um novo arquivo chamado `.env` na raíz da aplicação para armazenar nossas variáveis de ambiente:

```node
DATABASE_URL="postgresql://postgres:@localhost:5432/postgres?synchronize=true"
NEXTAUTH_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:3000
TWITTER_KEY=""
TWITTER_SECRET=""
```

As variáveis de ambiente do Twitter serão geradas a partir da [API do Twitter](https://developer.twitter.com/en/portal/petition/use-case). Iremos criar um novo Twitter app da [Dashboard de desenvolvimento do Twitter](https://developer.twitter.com/en/portal/petition/use-case).

1. Se não tem um registro como desenvolvedor no Twitter será necessário seguir todos os passos abaixo, se já possui pode pular para a etapa 6.
 
2. Acesse o [portal de desenvolvedor do Twitter](https://developer.twitter.com/en/portal/petition/use-case). Selecione a opção Hobbyist > Exploring the API e clique em `Get Started`.
   
3. Informe como gostaria de ser chamado, seu país e seu nível se habilidade como desenvolvedor e clique em `Next`.

![](../assets/images/twitter-clone/twitter-developer-portal.png)

4. No próximo passo você encontrará uma caixa de texto para informar com suas palavras como deseja utilizar a API e submeter essas informações para aprovação.

5. Você receberá um e-mail para confirmar seu cadastro, clique no botão `Confirm your email`.

6. Ainda na Dashboard do Twitter clique no menu `Projects & Apps` > `Overview` e clique no botão `+ Create App`.

7. Escolha o nome da sua aplicação. No meu caso utilizarei o nome `twitter-clone-bruno`.
8. Nesse momento você terá acesso as chaves da API copie-as pois utilizaremos mais tarde na nossa integração.

![](../assets/images/twitter-clone/twitter-dashboard-api-key.png)

9. Na próxima tela altere as permissões do aplicativo de apenas leitura para leitura e escrita.

![](../assets/images/twitter-clone/app-permissions.png)

10. Clique no botão `Edit` na seção `Authentication settings` e habilite os recursos `3-legged OAuth` e `Request email address from users`. No campo Callback URLs informe a seguinte *URL* `http://localhost:3000/api/auth/callback/twitter`. Os campos `Website URL`, `Terms of service` e `Privacy policy` podem ser preenchidos com qualquer informação. (ex: http://seuwebsite.com.br)

![](../assets/images/twitter-clone/3-legged-oauth.png)

Cole as informações de `API Key` dentro da variável de ambiente `TWITTER_KEY` e o valor de `API secret key` dentro da var `TWITTER_SECRET`.

Agora reiniciaremos o servidor Next.js e acessaremos http://localhost:3000/api/auth/signin, deveremos visualizar o botão para iniciar sessão no Twitter.

Se clicarmos nele estaremos prontos para autorizar nosso Twitter app mas não estamos preparados para logar na aplicação. Nosso terminal deve apresentar o seguinte alerta:

```bash
[next-auth][warn][jwt_auto_generated_signing_key] 
https://next-auth.js.org/warnings#jwt_auto_generated_signing_key
```

Iremos corrigir esse problema na próxima etapa quando adicionaremos e configuraremos o `Prisma`.

