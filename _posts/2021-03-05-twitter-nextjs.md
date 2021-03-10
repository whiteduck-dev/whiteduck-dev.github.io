---
layout: post
title: "Criando uma aplicação estilo Twitter com Next.js."
author: bruno
categories: [Next.js, Twitter, TypeScript]
tags: [Next.js, Twitter, TypeScript]
image: assets/images/twitter-clone/twitter-blog.jpeg
description: "Utilizando Next.js para criar uma interface semelhante ao twitter."
featured: true
hidden: true
---

Faz algum tempo que tenho curiosidade a respeito do hype envolvendo o Next.js, o framework front-end "queridinho" atualmente. Por isso resolvi buscar conhecer um pouco mais a respeito e com a ajuda de alguns posts e blogs na internet resolvi criar uma aplicação básica, com a finalidade de estudo.

Ao término, nossa aplicação deve contar com alguns recursos como:

- publicar um novo tweet
- visualizar uma lista de tweets
- visualizar o perfil de um usuário e seus respectivos tweets
- autenticação utilizando NextAuth e Twitter OAuth

Você poderá encontrar o código final para nossa aplicação no [GitHub](https://github.com/brunovigano/twitter-clone).

## 🚀 Começando

Next.js é um dos frameworks React.js mais populares. Ele possui um leque de recursos interessantes como server side rendering, suporte a TypeScript, otimização de imagens, suporte a i18n, criação de rotas baseada na estrutura de arquivos do sistema e muito mais.

Prisma é um ORM moderno para Node.js e TypeScript. Ele também disponibiliza vários recursos como geração automática de SQL, migrations, interface para manipulação dos dados no browser, etc.

#### Requerimentos

Para desenvolver essa gloriosa aplicação será preciso ter instalado:

- Docker
- npm
- yarn
- git

Tecnologias que usaremos:

- Next.js: para construir o app
- Prisma: para buscar e persistir dados na nossa base de dados
- NextAuth: para lidar com questões de autenticação
- Chakra UI: para fazer a estilização
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

Estamos prontos para acessar a pasta twitter-clone e iniciar nosso app executando o comando:

```bash
cd twitter-clone && yarn dev
```

Nosso app Next.js deve estar acessível em http://localhost:3000. Acessando este link devemos ver a seguinte tela:

![](../assets/images/twitter-clone/next-localhost-home.png)

#### Adicionando uma base de dados PostgreSQL com Docker

Vamos utilizar o docker para adicionar trabalhar com um banco de dados Postgre, assim poderemos armazenar os usuários e seus respectivos tweets. Podemos criar um novo arquivo ```
docker-compose.yml``` na raíz do projeto com o seguinte conteúdo:

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

Este comando será responsável por iniciar o container PostgreSQL que ficará disponível através do caminho ```postgresql://postgres:@localhost:5432/postgres```. Lembrando que se você preferir pode utilizar uma instalação local do Postgres no lugar do Docker.

#### Adicionando o Chackra UI

Chakra UI é uma biblioteca simples de React.js. Ela é muito popular e possui diversos recursos como acessibilidade, suporte para modo noturno e muito mais. Neste tutorial usaremos o Chakra UI para estilizar nossa interface. Podemos instalar o pacote rodando o comando a seguir:

```bash
yarn add @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

Vamos agora na pasta pages renomear nosso `_app.js` para `_app.tsx` e substituir seu conteúdo pelo trecho a seguir:

```
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

#### Adicionando o Prisma

Pra começar precisamos instalar o Prisma:

```bash
yarn add prisma @prisma/client
```

A seguir vamos criar um novo arquivo chamado `prisma.ts` dentro de `lib/clients` com o seguinte código:

```node
// lib/clients/prisma.ts

import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export default prisma;
```

O `PrismaClient` será reutilizado entre vários arquivos. Portanto devemos adicioná-lo ao nosso `pages/api/auth/[...nextauth].ts` com o seguinte código:

```node
....

import prisma from "../../../lib/clients/prisma";
import Adapters from "next-auth/adapters";

....

const options = {
  providers: [
    ....
  ],
  adapter: Adapters.Prisma.Adapter({ prisma }),
};

```

Se visitarmos o http://localhost:3000/api/auth/signin receberemos um erro no terminal:

```bash
Error: @prisma/client did not initialize yet. Please run "prisma generate" and try to import it again.
```

Para corrigir esse erro vamos seguir os passos:

1. Rodar `npx prisma init` da raíz do projeto:
   
```bash
$ npx prisma init

   Environment variables loaded from .env

   ✔ Your Prisma schema was created at prisma/schema.prisma.
     You can now open it in your favorite editor.

   warn Prisma would have added DATABASE_URL="postgresql://usuario:senha@localhost:5432/mydb?schema=public" but it already exists in .env

   Next steps:
   1. Set the DATABASE_URL in the .env file to point to your existing database. If your database has no tables yet, read https://pris.ly/d/getting-started.
   2. Set the provider of the datasource block in schema.prisma to match your database: postgresql, mysql or sqlite.
   3. Run prisma introspect to turn your database schema into a Prisma data model.
   4. Run prisma generate to install Prisma Client. You can then start querying your database.

   More information in our documentation:
   https://pris.ly/d/getting-started
```

2. Rode `npx prisma generate` da raíz do projeto:

```bash
 $ npx prisma generate
                               4s
   Environment variables loaded from .env
   Prisma schema loaded from prisma/schema.prisma
   Error:
   You don't have any models defined in your schema.prisma, so nothing will be generated.
   You can define a model like this:

   model User {
     id    Int     @id @default(autoincrement())
     email String  @unique
     name  String?
   }

   More information in our documentation:
   https://pris.ly/d/prisma-schema
```

3. Atualize o arquivo `prisma/schema.prisma` com o schema que o `NextAuth` espera:
   
   ```bash
    $ npx prisma generate
                               4s
   Environment variables loaded from .env
   Prisma schema loaded from prisma/schema.prisma
   Error:
   You don't have any models defined in your schema.prisma, so nothing will be generated.
   You can define a model like this:

   model User {
     id    Int     @id @default(autoincrement())
     email String  @unique
     name  String?
   }

   More information in our documentation:
   https://pris.ly/d/prisma-schema
   ```

4. Adicione o schema para o Tweet dentro de `prisma/schema.prisma`:
   ```bash
   // prisma/schema.prisma

   ....

   model Tweet {
     id        Int      @id @default(autoincrement())
     body      String
     userId    Int
     createdAt DateTime @default(now()) @map("created_at")
     updatedAt DateTime @default(now()) @map("updated_at")
     author    User     @relation(fields: [userId], references: [id])

     @@map("tweets")
   }
   ```

   5. Rode `npx prisma migrate dev --preview-feature` da raíz do projeto pra criar uma nova migration. Informe um nome pra ela quando solicitado.

A partir de agora se revisitarmos http://localhost:3000/api/auth/signin e clicarmos em `Sign in with Twitter`, seremos autenticados no nosso app usando o Twitter.

#### Adicionando uma `seed`

A melhor forma de dar vida a uma aplicação de estudos ou teste é ter dados aparentemente reais nas tabelas do seu banco de dados. Podemos fazer isso automaticamente utilizando a lib `faker` que foi criada justamente pensando nisso. Então mais do que depressa vamos adicioná-la ao nosso projeto para ver a mágica acontecendo.

`yarn add -D faker ts-node`

Agora precisamos criar um arquivo chamado `seed.ts` dentro da pasta `prisma` e adicionar o código a seguir:

```node
import faker from "faker";
import prisma from "../lib/clients/prisma";

async function main() {
  const listOfNewUsers = [...new Array(5)].map(() => {
    return {
      email: faker.internet.email(),
      name: faker.name.findName(),
      image: faker.image.image(),
      tweets: {
        create: {
          body: faker.lorem.sentence(),
        },
      },
    };
  });

  for (let data of listOfNewUsers) {
    const user = await prisma.user.create({
      data,
    });

    console.log(user);
  }
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Também precisamos atualizar nossa configuração base do typescript no arquivo `tsconfig.json` conforme informado abaixo:

```node
{
  "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": false,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "commonjs",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "baseUrl": ".",
    "paths": {
      "*": [
        "/*"
      ],
      "components/*": [
        "components/*"
      ],
      "pages/*": [
        "pages/*"
      ],
      "types/*": [
        "types/*"
      ],
      "lib/*": [
        "lib/*"
      ],
    },
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

Finalmente estamos prontos para executar `npx prisma db seed --preview-feature` para 'semear' o nosso banco de dados com alguns dados teste.

#### Adicionando o React Query

`React Query` é uma forma popular e performática de buscar dados nas aplicações em React.js. Bora adicioná-lo na nossa aplicação:

`yarn add react-query`

Agora vamos criar um arquivo chamado `react-query.ts` dentro de `lib/clients` com o seguinte código:

```node
// lib/clients/react-query.ts

import { QueryClient } from "react-query";

const queryClient = new QueryClient();

export default queryClient;
```

Também será preciso atualizar nosso arquivo `pages/_app.tsx` com o seguinte código:

```node
// pages/_app.tsx

....

import { QueryClientProvider } from "react-query";
import { Hydrate } from "react-query/hydration";
import queryClient from "../lib/clients/react-query";

const App = ({ Component, pageProps }: AppProps) => {
  return (
    <QueryClientProvider client={queryClient}>
      <Hydrate state={pageProps.dehydratedState}>
        <Head>
          <link rel="shortcut icon" href="/images/favicon.ico" />
        </Head>
        <NextAuthProvider session={pageProps.session}>
          <ChakraProvider>
            <Component {...pageProps} />
          </ChakraProvider>
        </NextAuthProvider>
      </Hydrate>
    </QueryClientProvider>
  );
};

export default App;
```

Agora vamos envolver nosso app com `QueryClientProvider`, que proverá um `QueryClient` na nossa aplicação.

#### Visualizar uma lista de Tweets

Vamos criar mais um arquivo, dessa vez chamado `fetch-tweets.ts` dentro de `lib/queries` com o seguinte código:

```node
// lib/queries/fetch-tweets.ts

const fetchTweets = async () => {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/tweets`);
  const data = await res.json();

  return data;
};

export default fetchTweets;
```

Essa função será responsável por buscar todos os tweets na nossa aplicação. Agora vamos criar um novo arquivo chamado `tweets.tsx` dentro de `pages` com o seguinte código:

```node
// pages/tweets.tsx

import fetchTweets from "../lib/queries/fetch-tweets";
import queryClient from "../lib/clients/react-query";
import { GetServerSideProps, InferGetServerSidePropsType } from "next";
import { useSession } from "next-auth/client";
import Head from "next/head";
import React from "react";
import { useQuery } from "react-query";
import { dehydrate } from "react-query/hydration";

const TweetsPage: InferGetServerSidePropsType<
  typeof getServerSideProps
> = ({}) => {
  const { data } = useQuery("tweets", fetchTweets);
  const [session] = useSession();

  if (!session) {
    return <div>Not authenticated.</div>;
  }

  return (
    <>
      <Head>
        <title>All tweets</title>
      </Head>
      {console.log(JSON.stringify(data, null, 2))}
    </>
  );
};

export const getServerSideProps: GetServerSideProps = async ({ req }) => {
  await queryClient.prefetchQuery("tweets", fetchTweets);

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
};

export default TweetsPage;
```

`getServerSideProps` é uma das principais funções do Next.js e ela auxilia a buscar dados no servidor. Vamos também criar um arquivo chamado `index.ts` dentro de `pages/api/tweets` com o seguinte conteúdo:

```node
// pages/api/tweets/index.ts

import prisma from "../../../lib/clients/prisma";
import type { NextApiRequest, NextApiResponse } from "next";

export default async (req: NextApiRequest, res: NextApiResponse) => {
  if (req.method === "POST") {
    try {
      const { body } = req;
      const tweet = await prisma.tweet.create({ data: JSON.parse(body) });

      return res.status(200).json(tweet);
    } catch (error) {
      return res.status(422).json(error);
    }
  } else if (req.method === "GET") {
    try {
      const tweets = await prisma.tweet.findMany({
        include: {
          author: true,
        },
        orderBy: [
          {
            createdAt: "desc",
          },
        ],
      });

      return res.status(200).json(tweets);
    } catch (error) {
      return res.status(422).json(error);
    }
  }

  res.end();
};
```

Nesse momento podemos testar nossa requisição. Se for `POST` vamos criar um novo Tweet. Se for `GET` vamos enviar todos os Tweets. Agora se visitarmos http://localhost:3000/tweets iremos visualizar uma lista de tweets no console do nosso navegador.

![faker.js](../assets/images/twitter-clone/tweet-fetch.jpg)

Observe que como o `faker.js` gerou dados randômicos o que você verá no seu console será diferente dos dados apresentados na screenshot acima.

Agora vamos construir a interface do usuário para mostrar a lista de tweets. Vamos criar um novo arquivo chamado `index.tsx` dentro de `components/pages/tweets` com o seguinte código:

```node
// components/pages/tweets/index.tsx

import { Box, Grid, Stack } from "@chakra-ui/react";
import Tweet from "./tweet";
import React from "react";
import ITweet from "types/tweet";

const TweetsPageComponent = ({ tweets }) => {
  return (
    <Stack spacing={8}>
      <Grid templateColumns={["1fr", "1fr", "repeat(2, 1fr)"]} gap={8}>
        {tweets?.map((tweet: ITweet) => {
          return (
            <Box key={tweet.id}>
              <Tweet tweet={tweet} />
            </Box>
          );
        })}
      </Grid>
    </Stack>
  );
};

export default TweetsPageComponent;
```

Vamos criar mais um arquivo chamado `tweet.tsx` dentro do mesmo diretório `components/pages/tweets` com o seguinte código:

```node
// components/pages/tweets/tweet.tsx

import { Avatar, Box, Stack, Text } from "@chakra-ui/react";
import React, { FC } from "react";

const Tweet: FC = ({ tweet }) => {
  const authorNode = () => {
    return (
      <Stack
        spacing={4}
        isInline
        alignItems="center"
        p={4}
        borderBottomWidth={1}
      >
        <Avatar name={tweet.author.name} src={tweet.author.image} />
        <Stack>
          <Text fontWeight="bold">{tweet.author.name}</Text>
        </Stack>
      </Stack>
    );
  };

  const bodyNode = () => {
    return (
      <Text fontSize="md" p={4}>
        {tweet.body}
      </Text>
    );
  };

  return (
    <Box shadow="lg" rounded="lg">
      <Stack spacing={0}>
        {authorNode()}
        {bodyNode()}
      </Stack>
    </Box>
  );
};

export default Tweet;
```

Agora vamos atualizar nosso arquivo `pages/tweets.tsx` com o seguinte código:

```node
// pages/tweets.tsx

....

import Page from "../components/pages/tweets";

....

const TweetsPage: InferGetServerSidePropsType<
  typeof getServerSideProps
> = ({}) => {

....

  return (
    <>
      <Head>
        <title>All tweets</title>
      </Head>
      <Page tweets={data} />
    </>
  );

....

}

....
```

Aqui nós modificamos a interface da nossa aplicação. Se rodarmos o projeto e visitarmos o link http://localhost:3000/tweets poderemos ver algo nesse sentido:

![lista de tweets](../assets/images/twitter-clone/tweets-list.jpg)

#### Adicionar um novo Tweet

Vamos adicionar na nossa interface um campo para adicionar um novo Tweet. Pra fazer isso vamos criar um novo arquivo chamado `add-new-tweet-form.tsx` dentro de `components/pages/tweets` com o seguinte conteúdo:

```node
// components/pages/tweets/add-new-tweet-form.tsx

import {
  Box,
  Button,
  FormControl,
  FormLabel,
  Stack,
  Textarea,
} from "@chakra-ui/react";
import saveTweet from "../../../lib/mutations/save-tweet";
import fetchTweets from "../../../lib/queries/fetch-tweets";
import queryClient from "../../../lib/clients/react-query";
import { useSession } from "next-auth/client";
import React, { ChangeEvent, useState } from "react";
import { useMutation, useQuery } from "react-query";

const AddNewTweetForm = () => {
  const [body, setBody] = useState("");
  const [session] = useSession();
  const { refetch } = useQuery("tweets", fetchTweets);
  const mutation = useMutation(saveTweet, {
    onSuccess: async () => {
      await queryClient.invalidateQueries("tweets");

      refetch();
    },
  });

  if (!session) {
    return <div>Not authenticated.</div>;
  }

  const handleSubmit = () => {
    const data = {
      body,
      author: {
        connect: { email: session.user.email },
      },
    };

    mutation.mutate(data);

    if (!mutation.error) {
      setBody("");
    }
  };

  return (
    <Stack spacing={4}>
      <Box p={4} shadow="lg" rounded="lg">
        <Stack spacing={4}>
          <FormControl isRequired>
            <FormLabel htmlFor="body">What's on your mind?</FormLabel>
            <Textarea
              id="body"
              value={body}
              onChange={(e: ChangeEvent<HTMLTextAreaElement>) =>
                setBody(e.currentTarget.value)
              }
            />
          </FormControl>
          <FormControl>
            <Button
              loadingText="Posting..."
              onClick={handleSubmit}
              isDisabled={!body.trim()}
            >
              Post
            </Button>
          </FormControl>
        </Stack>
      </Box>
    </Stack>
  );
};

export default AddNewTweetForm;
```

A função mutation é responsável por fazer a requisição `POST` no server. Ele também buscará os dados assim que a requisição for feita com sucesso. Também vamos criar um novo arquivo `save-tweet.ts` dentro de `lib/mutations` com o seguinte conteúdo:

```node
// lib/mutations/save-tweet.ts

const saveTweet = async (body: any) => {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/tweets`, {
    method: "POST",
    body: JSON.stringify(body),
  });
  const data = await res.json();

  return data;
};

export default saveTweet;
```

Nós também precisamos alterar nosso `components/pages/tweets/index.tsx` com esse conteúdo:

```node
// components/pages/tweets/index.tsx

....

import AddNewTweetForm from "./add-new-tweet-form";

....

const TweetsPageComponent = ({ tweets }) => {
  return (
    <Stack spacing={8}>
      <Box>
        <AddNewTweetForm />
      </Box>

      ....

    </Stack>
  );
};

export default TweetsPageComponent;
```

A partir de agora ao visitar o link http://localhost:3000/tweets poderemos visualizar algo nesse sentido:

![new tweet](../assets/images/twitter-clone/new-tweet.jpg)

Nós também estamos prontos para enviar um novo tweet usando nosso incrível campo de texto. (**atenção: esta ação não publicará nada na sua conta original do Twitter**)

#### Visualizar o perfil de um usuário e seus respectivos tweets

Precisamos criar uma página que será responsável por exibir uma lista de todos os usuários. Você já sabe como fazer isso, basta criar um novo arquivo chamado `index.tsx` dentro de `pages/users` com o seguinte código:

```node
// pages/users/index.tsx

import { GetServerSideProps, InferGetServerSidePropsType } from "next";
import { useSession } from "next-auth/client";
import Head from "next/head";
import React from "react";
import { useQuery } from "react-query";
import { dehydrate } from "react-query/hydration";
import Page from "../../components/pages/users";
import queryClient from "../../lib/clients/react-query";
import fetchUsers from "../../lib/queries/fetch-users";

const MyAccountPage: InferGetServerSidePropsType<
  typeof getServerSideProps
> = ({}) => {
  const { data } = useQuery("users", fetchUsers);
  const [session] = useSession();

  if (!session) {
    return <div>Not authenticated.</div>;
  }

  return (
    <>
      <Head>
        <title>All users</title>
      </Head>
      <Page users={data} />
    </>
  );
};

export const getServerSideProps: GetServerSideProps = async ({ req }) => {
  await queryClient.prefetchQuery("users", fetchUsers);

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
};

export default MyAccountPage;
```

Também será preciso criar um novo arquivo chamado `fetch-users.ts` dentro de `lib/queries` com o seguinte conteúdo:

```node
// lib/queries/fetch-users.ts

const fetchUsers = async () => {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/users`);
  const data = await res.json();

  return data;
};

export default fetchUsers;
```

A função criada nesse arquivo ficará responsável por buscar todos os usuários da nossa API. Também iremos criar um novo arquivo chamado `index.tsx` dentro de `components/pages/users` com o seguinte conteúdo:

```node
// components/pages/users/index.tsx

import { Box, Grid, Stack } from "@chakra-ui/react";
import React from "react";
import User from "./user";

const UsersPageComponent = ({ users }) => {
  return (
    <Stack spacing={8}>
      <Grid templateColumns={["1fr", "1fr", "repeat(2, 1fr)"]} gap={8}>
        {users?.map((user) => {
          return (
            <Box key={user.id}>
              <User user={user} />
            </Box>
          );
        })}
      </Grid>
    </Stack>
  );
};

export default UsersPageComponent;
```

Agora vou criar um novo arquivo chamado `users.tsx` dentro do mesmo diretório `components/pages/users` com o seguinte conteúdo:

```node
// components/pages/users/user.tsx

import { Avatar, Box, Stack, Text, Button } from "@chakra-ui/react";
import Link from "next/link";
import React, { FC } from "react";

const User: FC = ({ user }) => {
  const authorNode = () => {
    return (
      <Stack
        spacing={4}
        isInline
        alignItems="center"
        p={4}
        borderBottomWidth={1}
      >
        <Avatar name={user.name} src={user.image} />
        <Stack>
          <Text fontWeight="bold">{user.name}</Text>
        </Stack>
      </Stack>
    );
  };

  const bodyNode = () => {
    return (
      <Text fontSize="md" p={4}>
        {user.email}
      </Text>
    );
  };

  const buttonNode = () => {
    return (
      <Box p={4} borderTopWidth={1}>
        <Link href={`/users/${user.id}`}>
          <Button>View profile</Button>
        </Link>
      </Box>
    );
  };

  return (
    <Box shadow="lg" rounded="lg">
      <Stack spacing={0}>
        {authorNode()}
        {bodyNode()}
        {buttonNode()}
      </Stack>
    </Box>
  );
};

export default User;
```

E mais um arquivo chamado `index.ts` dentro de `pages/api/users` com o seguinte conteúdo:

```node
// pages/api/users/index.ts

import prisma from "../../../lib/clients/prisma";
import type { NextApiRequest, NextApiResponse } from "next";

export default async (req: NextApiRequest, res: NextApiResponse) => {
  if (req.method === "GET") {
    try {
      const users = await prisma.user.findMany({
        orderBy: [
          {
            createdAt: "desc",
          },
        ],
      });

      return res.status(200).json(users);
    } catch (error) {
      return res.status(422).json(error);
    }
  }

  res.end();
};
```

O arquivo criado acima tem uma função que é responsável por enviar os detalhes de todos os usuários. Agora se visitarmos o link http://localhost:3000/users certamente veremos algo nesse sentido:

![all users](../assets/images/twitter-clone/all-users.jpg)

Agora vamos criar uma página para exibir os detalhes de um único usuário. Para isso vamos precisar criar outro arquivo, desta vez chamado `[id].tsx` dentro de `pages/users` com o código a seguir:

```node
// pages/users/[id].tsx

import Page from "../../components/pages/users/[id]";
import queryClient from "../../lib/clients/react-query";
import fetchUser from "../../lib/queries/fetch-user";
import { GetServerSideProps, InferGetServerSidePropsType } from "next";
import { getSession, useSession } from "next-auth/client";
import Head from "next/head";
import React from "react";
import { useQuery } from "react-query";
import { dehydrate } from "react-query/hydration";

const MyAccountPage: InferGetServerSidePropsType<typeof getServerSideProps> = ({
  id,
}) => {
  const { data } = useQuery("user", () => fetchUser(parseInt(id as string)));
  const [session] = useSession();

  if (!session) {
    return <div>Not authenticated.</div>;
  }

  return (
    <>
      <Head>
        <title>{session.user.name}'s profile</title>
      </Head>
      <Page user={data} />
    </>
  );
};

export const getServerSideProps: GetServerSideProps = async ({ query }) => {
  await queryClient.prefetchQuery("user", () =>
    fetchUser(parseInt(query.id as string))
  );

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
      id: query.id,
    },
  };
};

export default MyAccountPage;
```

O valor de `query.id` determina o `id` do usuário atual. Nós iremos também criar um arquivo que podemos chamar de `fetch-user.ts` dentro de `lib/queries` com o código a seguir:

```node
// lib/queries/fetch-user.ts

const fetchUser = async (userId: number) => {
  const res = await fetch(
    `${process.env.NEXT_PUBLIC_API_URL}/api/users/${userId}`
  );
  const data = await res.json();

  return data;
};

export default fetchUser;
```

A função acima fica responsável por executar a requisição `GET` no nosso endpoint. Agora precisamos de mais um arquivo, desta vez vamos chamá-lo de `index.tsx` dentro de `components/pages/users/[id]` com o código a seguir:

```node
// components/pages/users/[id]/index.tsx

import { Avatar, Box, Grid, Stack, Text } from "@chakra-ui/react";
import Tweet from "./tweet";
import React, { FC } from "react";

const UsersPageComponent: FC = ({ user }) => {
  const authorNode = () => {
    return (
      <Stack spacing={4} isInline alignItems="center">
        <Avatar name={user?.name} src={user?.image} />
        <Stack>
          <Text fontWeight="bold" fontSize="4xl">
            {user?.name}
          </Text>
        </Stack>
      </Stack>
    );
  };

  return (
    <Stack spacing={8}>
      {authorNode()}
      <Grid templateColumns={["1fr", "1fr", "repeat(2, 1fr)"]} gap={8}>
        {user?.tweets.map((tweet) => {
          return (
            <Box key={tweet.id}>
              <Tweet tweet={tweet} />
            </Box>
          );
        })}
      </Grid>
    </Stack>
  );
};

export default UsersPageComponent;
```

O próximo passo é criar um arquivo chamado `tweet.tsx` dentro do mesmo diretório `components/pages/users/[id]` com o conteúdo a seguir:

```node
// components/pages/users/[id]/tweet.tsx

import { Box, Stack, Text } from "@chakra-ui/react";
import React, { FC } from "react";

const Tweet: FC = ({ tweet }) => {
  const bodyNode = () => {
    return (
      <Text fontSize="md" p={4}>
        {tweet.body}
      </Text>
    );
  };

  return (
    <Box shadow="lg" rounded="lg">
      <Stack spacing={0}>{bodyNode()}</Stack>
    </Box>
  );
};

export default Tweet;
```

Finalmente vamos criar mais um arquivo chamado `[id].ts` dentro de `pages/api/users` com o seguinte código:

```node
// pages/api/users/[id].ts

import prisma from "../../../lib/clients/prisma";
import type { NextApiRequest, NextApiResponse } from "next";

export default async (req: NextApiRequest, res: NextApiResponse) => {
  if (req.method === "GET") {
    const userId = parseInt(req.query.id as string);

    try {
      const tweets = await prisma.user.findUnique({
        include: {
          tweets: true,
        },
        where: {
          id: userId,
        },
      });

      return res.status(200).json(tweets);
    } catch (error) {
      console.log(error);

      return res.status(422).json(error);
    }
  }

  res.end();
};
```

A função representada acima tem a responsabilidade de enviar os detalhes do usuário cujo `id` é o mesmo de `req.query.id`. Podemos agora visitar o link http://localhost:3000/users e clicar no botão **View Profile** e então veremos uma lista de tweets publicados por esse usuário.

![user-tweets](../assets/images/twitter-clone/user-tweets.jpg)

## Finalizando

Fazendo esse tutorial eu aprendi um pouco sobre Next.js e sua integração com o Prisma. Obviamente a aplicação real do Twitter contempla uma série de outras funcionalidades não abordadas nesse tutorial, visto que o objetivo aqui é puramente acadêmico.

Como mencionado acima o código dessa aplicação está disponível no meu [GitHub](https://github.com/brunovigano/twitter-clone).

Um agradecimento especial a Nirmalya Ghosh por ter feito o [post](https://www.sitepoint.com/nextjs-prisma-twitter-clone/) que serviu como base para os meus estudos.
Até a próxima!
