---
layout: post
title: "Transforme seu texto simples em sites e blogs estáticos com Jekyll."
author: eduardo
categories: [Jekyll, Blog, Tutorial]
tags: [Jekyll, Blog, Tutorial]
image: assets/images/jekyll.png
description: "Transforme seu texto simples em sites e blogs estáticos."
featured: true
hidden: true
---

## Jekyll

Jekyll é um gerador de sites estáticos que transforma arquivos em um blog, site pessoal ou de uma empresa.

Escrito em Ruby por Tom Preston-Werner, co-fundador do GitHub, ele é distribuído sob a licença MIT de código aberto.

Com Jekyll é possível fazer tudo que um blog faz, como posts, paginação, categorias, tags, possibilidade de adicionar comentários, entre outras funcionalidades.

#### Instalação

```bash
gem install bundler jekyll
```

#### Criação do Site

> Após ter realizado a instalação do Jekyll, executar:

```bash
jekyll new my-awesome-site
cd my-awesome-site
```

#### Estrutura do Projeto

```bash
_posts
_config.yml
.gitignore
404.html
about.markdown
Gemfile
index.markdown
```

#### Sobre a Estrutura do Projeto

> Cada projeto pode ter sua própria estrutura, entretanto alguns arquivos são padrões:

```bash
_config.yml
```

É nesse arquivo que algumas informações importantes do seu blog serão descritas como:

- Nome do site;
- Descrição;
- URL;
- Tema;
- Entre outros;

```bash
Gemfile
```

Em projetos Ruby, o Gemfile é o gerenciamento de dependências. Cada pacote/biblioteca que você instala para ser usado pelo seu projeto é uma gem. Sendo assim você precisar por exemplo de plugins Jekyll para fazer coisas no seu site como paginar as postagens, incluir SEO ou construir automaticamente um feed XML.

#### Executando o Site

> No diretório "my-awesome-site", executar:

```bash
bundle exec jekyll serve
```

#### Vantagens

> Hospedagem do Site no [GitHub](https://github.com/).

#### Desvantagens

> Se comparar com o [Wordpress](https://br.wordpress.org/) ou outros CMS o mesmo não possui um site administrativo para gerenciar seus posts, por esse motivo uma pessoa leiga não conseguiria utilizar.
