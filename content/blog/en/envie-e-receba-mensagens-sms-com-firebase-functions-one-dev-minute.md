---
title: Envie e Receba Mensagens SMS com Firebase Functions | One Dev Minute
description: Este passo a passo rápido mostrará como criar um registro de
  mensagens SMS e uma resposta ao remetente usando Firebase Cloud Functions
  assim como o Firebase Real Time Database juntamente a API de SMS da Vonage.
thumbnail: /content/blog/envie-e-receba-mensagens-sms-com-firebase-functions-one-dev-minute/thumbnail-and-assets-for-one-dev-minute.jpg
author: amanda-cavallaro
published: true
published_at: 2021-12-07T09:47:51.097Z
updated_at: 2022-01-20T09:47:51.127Z
category: tutorial
tags:
  - sms-api
  - firebase
  - javascript
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Boas vindas ao [One Dev Minute](https://www.youtube.com/playlist?list=PLWYngsniPr_mwb65DDl3Kr6xeh6l7_pVY)! Esta série está hospedada no [canal de YouTube VonageDev](https://www.youtube.com/vonagedev). O objetivo desta série de vídeos é compartilhar conhecimento de uma maneira breve.

Este passo a passo rápido mostrará como criar um registro de mensagens SMS e uma resposta ao remetente usando o Firebase Cloud Functions e o Banco de Dados Real Time Database juntamente a API de SMS da Vonage.

<youtube id="v2sh3KHyKXc"></youtube>

## Trancrição

Você pode enviar mensagens de SMS usando o Cloud Functions para Firebase. Você precisará de criar duas contas:

* Uma de Firebase
* E outra de APIs da Vonage

Crie o projeto no console do Firebase e escolha se você vai usar o Analytics.

Aguarde a criação do seu projeto.

Selecione o plano de faturamento do Firebase, neste caso é o pagamento conforme o uso.

A partir da linha de comandos, instale as ferramentas do Firebase.

Efetue o login e autentique-se.

Crie a pasta do projeto e altere o diretório dentro dela.

Inicialize o Cloud Functions para Firebase.

Instale as dependências que vamos usar dentro da pasta `functions`.

Crie um arquivo `.env` e adicione as variáveis de ambiente da Vonage lá.

Dentro do arquivo `index.js`, adicione todas as dependências e variáveis de ambiente necessárias e inicialize o Firebase.

No mesmo arquivo, crie a primeira função que funcionará como um webhook para capturar e registrar mensagens de SMS recebidas de um número de telefone da Vonage.

Vamos então criar uma função para o Firebase enviar o SMS de resposta e reagir às atualizações do banco de dados.

Implante a função, envie uma mensagem de SMS do seu telefone para o telefone do aplicativo da  Vonage.

Você receberá uma mensagem SMS de resposta em seu telefone e uma atualização no Banco de Dados Firebase Real-Time.

O código completo está no GitHub.

Obrigada por assistir e boa codificação!

## Links

[O código deste tutorial disponível no GitHub](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqazJ0UDFleGVwSnBfQU1ORTRLYkhDM0xrbkpZQXxBQ3Jtc0trcjJnY0E4QjRybFUwVk5GRWJQSVhMcnJERC1MbHQyWEpqaHNLSklyWjRiMFdZYmt2RzlaVVQ5UWRMYnVDa1V6SE1RcG5jTm5RSl9MbkRWNlhYZkRsYUtkc2JDXzZBM3p4NXRzNkZnTHp0LVMxbEdNUQ&q=https%3A%2F%2Fgithub.com%2Fnexmo-community%2Ffirebase-functions-sms-example).

[O tutorial em formato escrito aqui](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbUttd1Q0OHBsYU9fWlZyaHZlZ2JhN25FVE1LQXxBQ3Jtc0tsbFNxSVV1Q3ZtNzRUSkU4QUJwYVhHaENZZkJNYXZoemx0YkVjOUpWMmhMcXluRjBYVU4wNVcwdGU5SWFjU0FDUXRCUW1VNEd6U1ZjNTd5ZHl0V20xTW5fSUZfUXBzNldYUDltMlprOEhZRDBpMFMxMA&q=https%3A%2F%2Flearn.vonage.com%2Fblog%2F2020%2F01%2F24%2Fsend-and-receive-sms-messages-with-firebase-functions-dr%2F).

[A documentação para pessoas desenvolvedoras](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbkdBRVRBMDZsY05fYTJJeE14UmxsMFFGUWJGQXxBQ3Jtc0trY21SMGtEaGRsaVBKUmdpMkxDMlh6NWFrU2JtNjRNcHlGM200bGoyaVRiOGFnN2lYOUFFNnY3V1hZaVFaMWlEamtFOGU0eDdtWmxEVnlJLWlzWFptT3NJM2RpZFQtclg2Z09zVUpHcmZUXzM1T3BOTQ&q=https%3A%2F%2Fdeveloper.vonage.com).

[Detalhes sobre a funcionalidade de SMS da Vonage](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa3VKcWlvTTJqXzRTODh6SEdoNUlvdmJuMHo1d3xBQ3Jtc0tub3hvYlFpbnhQVktXdWZFcENEVHNlbFNfUmFZenNOVUFoTmUwWHBwekxrSDBLWW1LZDg5UFBnZ2t4UWpBaFlFazBIcDF2bjRLc1c1ZGVNRUhKblFFRmZDLTQtQXIxMnBVQ1RKR1dGTG5xd0dPRzdqZw&q=https%3A%2F%2Fdeveloper.vonage.com%2Fmessaging%2Fsms%2Foverview).

[Primeiros passos com Firebase Functions](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbUdadHRpUm4zZkNSSkdmQnMzUUEzdTFxR2ZPd3xBQ3Jtc0trSTc4S0tUbmNGVEFxaHk3Zk5CbmE5c3pQMzgzczd0QUF0M3Y3aTMzWFhiVlhHTkdDa3I3aUFxNGZqN05SZ09TUG1wcFd6UW1FRkl0THFJbWFBRHBTbXg5c1lwbG4zSjZzRXdGS0dGR2l3dHQ2LUQ0NA&q=https%3A%2F%2Ffirebase.google.com%2Fdocs%2Ffunctions%2Fget-started).