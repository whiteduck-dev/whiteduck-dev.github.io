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

Ao final nossa aplicação terá os seguintes recursos:

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

```bash
yarn create next-app
```

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

Podemos agora acessar a pasta twitter-clone e iniciar nosso app executando o comando:

```bash
cd twitter-clone && yarn dev
```

Nosso app Next.js deve estar acessível em http://localhost:3000. Acessando este link devemos ver a seguinte tela:

![alt text](assets/images/twitter-clone/next-localhost-home.png)
