---
title: Risoluzione dei problemi [!DNL Asset Compute Service].
description: Risoluzione dei problemi e debug delle applicazioni personalizzate utilizzando [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '306'
ht-degree: 1%

---


# Risoluzione dei problemi {#troubleshoot}

Alcuni suggerimenti generici per la risoluzione dei problemi che possono essere utili per risolvere eventuali problemi con Asset Compute Service sono:

* Verificare che l&#39;applicazione JavaScript non si arresti in modo anomalo all&#39;avvio. Tali arresti anomali sono in genere correlati a una libreria mancante o a una dipendenza.
* Verificate che nel `package.json` file dell&#39;applicazione venga fatto riferimento a tutte le dipendenze da installare.
* Assicurati che eventuali errori derivanti dalla pulizia in caso di errore non generino errori personalizzati che nascondano il problema originale.

* Quando si avvia lo strumento di sviluppo per la prima volta con una nuova [!DNL Asset Compute Service] integrazione, la prima richiesta di elaborazione potrebbe non riuscire, perché il giornale di registrazione eventi di calcolo risorse potrebbe non essere completamente configurato. Attendi che il journal venga configurato prima di inviare un&#39;altra richiesta.
* Se si verificano degli errori durante l&#39;invio di `/register` `/process` richieste o calcoli di risorse, assicurarsi che tutte le API necessarie siano aggiunte al progetto di I/O e all&#39;area di lavoro di I/O del Adobe , ovvero, calcolo delle risorse, eventi IO, gestione eventi IO e runtime.

## Problemi di accesso tramite  CLI I/O Adobe {#login-via-aio-cli}

In caso di problemi durante l&#39;accesso all&#39; [!DNL Adobe Developer Console] interfaccia CLI [I/O del Adobe , aggiungere manualmente le credenziali necessarie per lo sviluppo, il test e la distribuzione dell&#39;applicazione personalizzata](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)tramite l&#39;interfaccia CLIdi I/O del :

1. Andate al progetto e all&#39;area di lavoro Firefly nel [Adobe Developer Console](https://console.adobe.io/) e premete **[!UICONTROL Scarica]** dall&#39;angolo superiore destro. Apri questo file e salva il JSON in un luogo sicuro sul tuo computer.

1. Individuate il file ENV nell’applicazione Firefly.

1. Aggiungere le credenziali Adobe I/O Runtime. Ottenete le credenziali Adobe I/O Runtime dal JSON scaricato. Le credenziali sono scadute `project.workspace.services.runtime`. Aggiungere le credenziali di I/O Runtime nelle `AIO_runtime_XXX` variabili:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Aggiungi il percorso assoluto al JSON scaricato al passaggio 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configurate il resto delle credenziali [](develop-custom-application.md) necessarie per lo strumento di sviluppo.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
