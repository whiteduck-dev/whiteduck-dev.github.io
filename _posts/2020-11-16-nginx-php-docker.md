---
layout: post
title: 'Como rodar um php-fpm server atrás de um nginx'
author: fernando
categories: [php, nginx, docker, Tutorial]
tags: [php, nginx, docker, Tutorial]
image: assets/images/nginx-logo.png
description: 'Como rodar um php-fpm server atrás de um nginx'
featured: true
hidden: true
---

## Problema atual

Eu precisava colocar nossos servers em PHP para rodar dentro de containers no Docker, para isso temos duas soluções uma usando a imagem do docker oficial que já vem com o apache instalado e configurado:

```dockerfile
FROM php:7.2-apache
```

ou utilizar uma imagem customizada com php+apache ou nginx.

Mas eu teimoso queria uma coisa mais simples que rodasse o PHP com o minimo de espaço ocupado pelas imagens e pudesse utilizar o nginx para gerenciar os sites/proxy reverso.

Entra em cena php-fpm.

```bash
$ docker images php
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
php                 7.2-fpm             03d449391aab        10 days ago         398MB
php                 7.2-apache          ea7f5666bfc1        10 days ago         410MB
```

#### O que é PHP-FPM

FPM é um gerenciador de processos para gerenciar o FastCGI SAPI (Server API) em PHP.

O PHP-FPM é um serviço e não um módulo. Este serviço é executado completamente independente do servidor web em um processo à parte e é suportado por qualquer servidor web compatível com FastCGI (Fast Common Gateway Interface).

PHP-FPM é consideravelmente mais rápido que os outros métodos de se processar scripts php, e também é escalável, ou seja é possível construir clusters e expandir a capacidade do PHP de receber requisições.

#### O que é NGINX

NGINX, pronunciado “engine-ex,” é um famoso software de código aberto para servidores web lançado originalmente para navegação HTTP. Hoje, porém, ele também funciona como proxy reverso, balanceador de carga HTTP, e proxy de email para os protocolos IMAP, POP3, e SMTP.

#### Resolução do meu problema?

Foi demorado conseguir chegar num exemplo que está funcionando 100%, muitas contradições em configurações, principalmente do nginx.

Mas após algum tempo procurando e juntando pedaços de respostas do `stackoverflow` consegui chegar num exemplo funcional, com 2 sites rodando php + 1 rodando node, atrás de um nginx.

#### Como testar você mesmo

Crie uma pasta onde irá rodar os testes e entre na mesma

```bash
$ mkdir teste
$ cd teste
```

Crie uma pasta chamada `site-a`

```bash
$ mkdir site-a
$ cd site-a
```

Dentro da pasta `site-a` crie uma pasta `php` e dentro dela crie um arquivo `index.php` com o seguinte conteúdo

```php
<?
echo "Bem vindo ao Site A";
```

Volte a pasta `site-a` e crie um arquivo chamado `Dockerfile` com o seguinte conteúdo

```dockerfile
FROM brilvio/php-fpm-mssql

COPY ./php /var/www/html/site-a
```

Se você é familiarizado com o Docker verá que é bem simples ele irá pegar como base a minha imagem já configurado com o conector para `mssql` e copiar os arquivos da pasta `./php` para a pasta dentro do container `/var/www/html/site-a` é importante que essa pasta dentro do container concida depois com o nome da pasta dentro do container do nginx.

Rode o comando na pasta `site-a` para gerar a imagem do docker com a tag correta

```bash
$ docker build -t site-a .
```

Crie agora mais uma pasta chamada `site-b` modificando o `Dockerfile` e o `index.php` de acordo.

Rode o comando na pasta `site-b` para gerar a imagem do docker com a tag correta

```bash
$ docker build -t site-b .
```

Para o `site-c` vai ser um pouco diferente pois ele vai rodar um servidor node+express dentro dele, crie a pasta normal, porém invés de uma pasta `php` crie uma pasta chamada `src` e dentro dela o arquivo `index.js` com o seguinte conteúdo.

```js
const express = require('express');
const app = express();
const port = 80;

app.get('/', (req, res) => {
  res.send('Hello  from express!');
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

Na pasta `site-c` crie um arquivo `package.json` com o seguinte conteúdo

```json
{
  "name": "site-c",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

E um arquivo `Dockerfile` com o seguinte conteúdo

```dockerfile
FROM node:12-alpine as compile-image
RUN apk update && apk add yarn curl bash python g++ make && rm -rf /var/cache/apk/*
RUN curl -sfL https://install.goreleaser.com/github.com/tj/node-prune.sh | bash -s -- -b /usr/local/bin

WORKDIR /usr/src/

COPY package.json ./
RUN yarn install

COPY src/ ./

RUN npm prune --production

RUN /usr/local/bin/node-prune

FROM node:12-alpine

WORKDIR  /usr/src/
COPY --from=compile-image /usr/src/node_modules ./node_modules

COPY --from=compile-image /usr/src/ ./

EXPOSE 80

CMD [ "node", "index.js" ]
```

Rode o comando na pasta `site-c` para gerar a imagem do docker com a tag correta

```bash
$ docker build -t site-c .
```

Estamos quase lá, vamos criar agora as configurações que o nginx vai usar para saber para onde enviar as requisições, crie uma pasta sites e dentro dela crie as seguintes configurações.

`site-a.conf`

```
server {
	root /var/www/html/site-a; # deve ser a mesma pasta dentro do Dockerfile
	index index.php index.html index.htm;

	server_name site-a.docker;

	 index index.php index.html;

    client_max_body_size 108M;
    gzip_static on;

    location / {
        if (!-e $request_filename) {
            rewrite ^.*$ /index.php last;
        }
    }

    location ~ [^/]\.php(/|$) {
            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            if (!-f $document_root$fastcgi_script_name)
            {
                    return 404;
            }
            # Mitigate https://httpoxy.org/ vulnerabilities
            fastcgi_param HTTP_PROXY "";
            fastcgi_read_timeout 150;
            fastcgi_buffers 4 256k;
            fastcgi_buffer_size 128k;
            fastcgi_busy_buffers_size 256k;
            fastcgi_pass site-a:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }

    location ~ /\.ht {
        deny all;
    }
}
```

`site-b.conf`

```
server {
	root /var/www/html/site-b; # deve ser a mesma pasta dentro do Dockerfile
	index index.php index.html index.htm;

	server_name site-b.docker;

	 index index.php index.html;

    client_max_body_size 108M;
    gzip_static on;

    location / {
        if (!-e $request_filename) {
            rewrite ^.*$ /index.php last;
        }
    }

    location ~ [^/]\.php(/|$) {
            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            if (!-f $document_root$fastcgi_script_name)
            {
                    return 404;
            }
            # Mitigate https://httpoxy.org/ vulnerabilities
            fastcgi_param HTTP_PROXY "";
            fastcgi_read_timeout 150;
            fastcgi_buffers 4 256k;
            fastcgi_buffer_size 128k;
            fastcgi_busy_buffers_size 256k;
            fastcgi_pass site-b:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }

    location ~ /\.ht {
        deny all;
    }
}
```

`site-c.conf`

```
upstream site-c {
    server site-c:80;
}

server {
    server_name site-c.docker;
    client_max_body_size 200M;
    location / {
        proxy_pass http://site-c/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forward-Proto http;
        proxy_set_header X-Nginx-Proxy true;

        proxy_redirect off;
    }
    listen 80;
}
```

Veja que o `site-c.conf` é mais simples, é por que ele é somente um proxy reverso.

Vamos agora a parte mais fácil, criar o `docker-compose.yml` que vai subir os nossos servidores.

```yml
version: '3.3'
services:
  nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    volumes:
      - type: volume
        source: site-a
        target: /var/www/html/site-a # deve ser a mesma pasta dentro do Dockerfile
      - type: volume
        source: site-a
        target: /var/www/html/site-b # deve ser a mesma pasta dentro do Dockerfile
      - ./sites:/etc/nginx/conf.d/
    ports:
      - '8080:80'
    links:
      - site-a
      - site-b
      - site-c
  site-a:
    image: site-a:latest
    container_name: site-a
    expose:
      - '9000'
    volumes:
      - type: volume
        source: site-a
        target: /var/www/html/site-a # deve ser a mesma pasta dentro do Dockerfile
  site-b:
    image: site-b:latest
    container_name: site-b
    expose:
      - '9000'
    volumes:
      - type: volume
        source: site-b
        target: /var/www/html/site-b # deve ser a mesma pasta dentro do Dockerfile
  site-c:
    image: site-c:latest
    container_name: site-c
    expose:
      - '80'

volumes:
  site-a:
  site-b:
```

Somente os site-a e site-b tem volumes configurados, é porque o nginx precisa ter os arquivos .php dentro dele também e nas mesmas pastas para poder passar para o servidor php-fpm interpretar.

Único problema é que se você quiser atualizar os arquivos php dentro dos containers tu vai ter que remover os volumes e recria-los novamente.

Configure no teu arquivo hosts `/etc/hosts`

```
127.0.0.1 site-a
127.0.0.1 site-b
127.0.0.1 site-c
```

Ao acessar no browser http://site-a:8080 http://site-b:8080 ou http://site-c:8080 verá que o nginx está redicionando para os servidores corretamente.
