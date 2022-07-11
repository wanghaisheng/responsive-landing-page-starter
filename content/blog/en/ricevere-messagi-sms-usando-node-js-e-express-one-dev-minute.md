---
title: Ricevere Messagi SMS Usando Node.js e Express | One Dev Minute
description: Vediamo come ricevere messaggi usando Node.JS, Express e l’API
  Messages di Vonage.
thumbnail: /content/blog/ricevere-messagi-sms-usando-node-js-e-express-one-dev-minute/ricevere-messagi-sms-usando-node.js-e-express.png
author: amanda-cavallaro
published: true
published_at: 2022-07-11T20:25:12.954Z
updated_at: 2022-07-11T20:25:12.970Z
category: tutorial
tags:
  - italian
  - node
  - messages-api
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Benvenuti a One Dev Minute! L'obiettivo di questa serie di video è condividere informazioni in un formato velocemente consumabile. Puoi seguirla sul canale YouTube di Vonage Dev.

## Trascrizione

Vediamo come ricevere messaggi usando Node.JS, Express e l’API Messages di Vonage.

Prima di cominciare, assicurati di avere:

* creato un account Vonage,
* instalato Node.JS, Ngrok e il CLI di Vonage globalmente.

Crea una cartella, e naviga al suo interno. Installa Express e il Vonage Server SDK Beta. Crea un nuovo file chiamato server.js e aprilo nel tuo ambiente di sviluppo preferito.

Procederemo a creare una applicazione basata su Express che userà i moduli per parsare JSON e codificare gli URL.

Useremo la porta 3000 per mettere il server in ascolto. Ora creeremo un handler per gestire la richiesta POST per il webhook associato all’URL in entrata, per poi loggare il body nella console.

Eseguiremo quindi il codice con il comando “node server.js” in una scheda del terminale mentre in un’altra scheda eseguiremo “ngrok http 3000”.

Dalla dashboard di Vonage, clicca su “Settings” nel menu sulla sinistra. Assicurati che la Messages API è impostata come default nei “SMS settings”, e infine clicca “Save” per confermare.

Nella dashboard di Vonage e seleziona la creazione di una nuova applicazione. Scegli il nome, scorri verso il basso finoa a “Capabilities” e seleziona “Messages” sulla destra.

Torna alla tua scheda del termina e copia l’URL HTTPS che è stato generato per usarlo in ngrok.

Per l’URL in entrata useremo l’URL aggiungendovi “/webhooks/inbound”, che è il percorso che abbiamo impostato nel nostro codice.

Naviga verso il basso e genera una nuova applicazione. Associa un numero di telefono. Se non ne hai uno, puoi comprarlo sul menu a sinistra.

Per vederlo all’opera, puoi inviare un messaggio dal tuo telefono al tuo numero virtuale. Dovresti vedere un messaggio loggato nella finestra del terminale.

Puoi approfondire l’argomento seguendo i link qui in basso.

## Links

[Leggi la versione scritta del tutorial](https://developer.vonage.com/blog/2019/09/16/how-to-send-and-receive-sms-messages-with-node-js-and-express-dr)

[Fai riferimento al codice su GitHub](https://github.com/nexmo-community/nexmo-sms-autoresponder-node/)

[Fai riferimento al codice su Glitch](https://glitch.com/edit/#!/whispering-rebel-ixia)

[Unisciti alla Community di Sviluppatori Vonage su Slack](https://developer.vonage.com/community/slack)