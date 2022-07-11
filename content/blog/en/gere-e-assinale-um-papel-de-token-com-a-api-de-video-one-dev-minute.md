---
title: Gere e Assinale um Papel de Token com a API de Vídeo | One Dev Minute
description: Veja como as funções de token permitem identificar usuários e
  alterar as funções de permissão usando a API de Vídeo da Vonage.
thumbnail: /content/blog/gere-e-assinale-um-papel-de-token-com-a-api-de-vídeo-one-dev-minute/thumbnail.png
author: amanda-cavallaro
published: true
published_at: 2022-07-08T10:09:05.915Z
updated_at: 2022-07-08T10:09:05.927Z
category: tutorial
tags:
  - javascript
  - portuguese
  - voice-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Boas vindas ao [One Dev Minute](https://www.youtube.com/playlist?list=PLWYngsniPr_mwb65DDl3Kr6xeh6l7_pVY)! Esta série está hospedada no [canal de YouTube VonageDev](https://www.youtube.com/vonagedev). O objetivo desta série de vídeos é compartilhar conhecimento de uma maneira breve.

Veja como as funções de token permitem identificar usuários e alterar as funções de permissão usando a API de Vídeo da Vonage.

<youtube id="EMkGCDKup2Q"></youtube>

## Transcrição

Oi! Sou a Amanda Cavallaro da Vonage, e hoje falarei sobre geração e atribuição de papeis de token.

Para autenticar um usuário que se conecta a uma sessão de API de vídeo, devemos usar uma chave de autenticação exclusiva chamada token.

Existem três funções de token possíveis: subscriber, publisher, e moderator.

Vamos gerar um token usando a biblioteca do lado do servidor de Node.js da API de vídeo.

Primeiro instalamos o Opentok à partir do terminal.
Em seu editor de código, defina as constantes com a chave de API e o segredo de API que recebe ao se inscrever para usar a API de vídeo.

Você pode criar ou usar um ID de sessão existente. Em seguida, chamamos o método `generateToken` que retornará um token no formato string.

Você pode gerar tokens para os clientes usarem ao se conectarem à sessão. Neste exemplo, estamos criando um nome de usuário "Amanda" de função "publisher".

O parâmetro *options* é um objeto opcional usado para definir a função, o tempo de expiração e os dados de conexão do token. 

Em seguida, adicionamos as opções ao token gerado, executamos o Node e o nome do arquivo que criamos e em seu terminal, você pode ver seu token.

Há várias coisas que você pode fazer com seu token gerado, dependendo de sua função você tem permissões adicionais se receber um token de moderador. Você pode, por exemplo, desconectar, silenciar outros usuários ou até mesmo parar de publicar seus fluxos.

Neste vídeo, você viu como gerar e assinalar um token. Você pode aprender mais nos links abaixo!

## Links

[Código presente no vídeo](https://tokbox.com/developer/guides/create-token/node/)

[Capacidades](https://tokbox.com/developer/sdks/js/reference/Capabilities.html)

[Criar um Token](https://tokbox.com/developer/guides/create-token/)

Junte-se à [comunidade de pessoas desenvolvedoras da Vonage no Slack](https://developer.vonage.com/community/slack)