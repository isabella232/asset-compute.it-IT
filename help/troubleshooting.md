---
title: Risoluzione dei problemi [!DNL Asset Compute Service]
description: Risoluzione dei problemi e debug di applicazioni personalizzate utilizzando [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '288'
ht-degree: 1%

---


# Risoluzione dei problemi {#troubleshoot}

Alcuni suggerimenti generici per la risoluzione dei problemi che possono essere utili per la risoluzione dei problemi con  servizio Asset compute sono:

* Verificare che l&#39;applicazione JavaScript non si arresti in modo anomalo all&#39;avvio. Tali arresti anomali sono in genere correlati a una libreria mancante o a una dipendenza.
* Assicurarsi che tutti i riferimenti alle dipendenze da installare siano presenti nel file `package.json` dell&#39;applicazione.
* Assicurati che eventuali errori derivanti dalla pulizia in caso di errore non generino errori personalizzati che nascondano il problema originale.

* Quando si avvia lo strumento di sviluppo per la prima volta con una nuova [!DNL Asset Compute Service] integrazione, la prima richiesta di elaborazione potrebbe non riuscire, perché il giornale di registrazione eventi Asset compute  potrebbe non essere completamente configurato. Attendi che il journal venga configurato prima di inviare un&#39;altra richiesta.
* Se si verificano degli errori durante l&#39;invio  richieste di Asset compute `/register` o `/process`, assicurarsi che tutte le API necessarie siano aggiunte al [!DNL Adobe I/O] Project and Workspace, ovvero,  Asset compute, [!DNL Adobe I/O] Events, [!DNL Adobe I/O] Events Management e [!DNL Adobe I/O] Runtime.

## Problemi di accesso tramite [!DNL Adobe I/O] CLI {#login-via-aio-cli}

In caso di problemi durante l&#39;accesso a [!DNL Adobe Developer Console] [mediante la  [!DNL Adobe I/O] CLI](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli), aggiungere manualmente le credenziali necessarie per lo sviluppo, il test e la distribuzione dell&#39;applicazione personalizzata:

1. Andate al progetto e all&#39;area di lavoro Firefly nella [ Adobe Developer Console](https://console.adobe.io/) e premete **[!UICONTROL Download]** dall&#39;angolo in alto a destra. Apri questo file e salva il JSON in un luogo sicuro sul tuo computer.

1. Individuate il file ENV nell’applicazione Firefly.

1. Aggiungete le credenziali runtime [!DNL Adobe I/O]. Ottenete le credenziali [!DNL Adobe I/O] A Runtime dal JSON scaricato. Le credenziali sono inferiori a `project.workspace.services.runtime`. Aggiungete le credenziali [!DNL Adobe I/O] Runtime nelle variabili `AIO_runtime_XXX`:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Aggiungi il percorso assoluto al JSON scaricato al passaggio 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configurate il resto delle [credenziali ](develop-custom-application.md) necessarie per lo strumento di sviluppo.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
