---
title: Inviare e ricevere SMS usando Firebase Functions | One Dev Min
description: Questa panoramica ti mostrerà come creare un registro dei messaggi
  SMS ricevuti e come inviare una risposta al mittente utilizzando Firebase
  Cloud Functions e Firebase Real-Time Database insieme alle API Vonage SMS.
thumbnail: /content/blog/send-and-receive-sms-messages-with-firebase-functions-one-dev-minute/thumbnail-and-assets-for-one-dev-minute.jpg
author: amanda-cavallaro
published: true
published_at: 2022-03-25T16:06:52.389Z
updated_at: 2022-03-25T16:06:52.400Z
category: tutorial
tags:
  - italian
  - sms-api
  - firebase
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Benvenuti a [One Dev Minute](https://www.youtube.com/playlist?list=PLWYngsniPr_mwb65DDl3Kr6xeh6l7_pVY) ! L'obiettivo di questa serie di video è condividere informazioni in un formato velocemente consumabile. Puoi seguirla sul [canale YouTube di Vonage Dev](https://www.youtube.com/vonagedev).

Questa panoramica ti mostrerà come creare un registro dei messaggi SMS ricevuti e come inviare una risposta al mittente utilizzando Firebase Cloud Functions e Firebase Real-Time Database insieme alle API Vonage SMS.

<youtube id="c8gHy_KvQAE"></youtube>

## **Trascrizione**

Per inviare messaggi SMS utilizzando Cloud Functions per Firebase, dovrai creare un paio di account:

* uno per Firebase
* e uno dell'API Vonage.

Crea il progetto nella console Firebase e scegli se utilizzare Analytics o meno.

Attendi la creazione del tuo progetto.

Seleziona il piano di fatturazione Firebase, in questo caso è il pagamento in base al consumo, pay as you go.

Nella riga di comando, installa gli strumenti Firebase.

Accedi a Firebase e autenticati. Crea la cartella del progetto e naviga al suo interno.

Inizializza le Cloud Functions per Firebase.

Installa le dipendenze che useremo all'interno della cartella delle funzioni.

Crea un `.env` file e aggiungi lì le variabili d'ambiente per Vonage API.

All'interno del file `index.js`, aggiungi tutte le dipendenze e le variabili di ambiente richieste e inizializza Firebase.

Nello stesso file, crea la prima funzione che fungerà da webhook per acquisire e registrare i messaggi SMS in arrivo su un numero di telefono Vonage.

Creiamo quindi una funzione per Firebase per inviare l'SMS di risposta e per reagire agli aggiornamenti del database.

Rilascia la funzione e invia un messaggio SMS dal tuo telefono al numero di telefono dell'applicazione Vonage.

Riceverai quindi un messaggio SMS di risposta sul telefono e un aggiornamento al Firebase Real-Time Database.

Puoi trovare il codice completo su GitHub. Grazie per la visione e buono sviluppo!

## Links

[Il codice di questo tutorial su GitHub](https://github.com/nexmo-community/firebase-functions-sms-example).

[Trova il tutorial scritto qui](https://developer.vonage.com/blog/2020/01/24/send-and-receive-sms-messages-with-firebase-functions-dr).

[Dai un'occhiata alla documentazione per gli sviluppatori](https://developer.vonage.com/).

[Dettagli sulla funzionalità SMS di Vonage](https://developer.vonage.com/messaging/sms/overview).

[Guida introduttiva alle funzioni Firebase](https://firebase.google.com/docs/functions/get-started).