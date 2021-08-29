---
title: Risoluzione dei problemi [!DNL Asset Compute Service]
description: Risolvere i problemi e eseguire il debug delle applicazioni personalizzate utilizzando [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '285'
ht-degree: 1%

---

# Risoluzione dei problemi {#troubleshoot}

Alcuni suggerimenti generici per la risoluzione dei problemi che possono essere utili per la risoluzione dei problemi con Asset compute Service sono:

* Assicurati che l&#39;applicazione JavaScript non si arresti all&#39;avvio. Tali arresti anomali sono solitamente correlati a una libreria mancante o a una dipendenza.
* Assicurati che nel file `package.json` dell&#39;applicazione siano presenti riferimenti a tutte le dipendenze da installare.
* Assicurati che eventuali errori derivanti dalla pulizia in caso di errore non generino errori propri che nascondono il problema originale.

* Quando si avvia lo strumento per sviluppatori per la prima volta con una nuova integrazione [!DNL Asset Compute Service], la prima richiesta di elaborazione potrebbe non riuscire se Asset compute Events Journal non è completamente configurato. Attendi l&#39;impostazione del journal prima di inviare un&#39;altra richiesta.
* Se si verificano degli errori durante l’invio di richieste di Asset compute `/register` o `/process`, assicurati che tutte le API necessarie siano aggiunte al progetto [!DNL Adobe I/O] e all’area di lavoro, ovvero, Asset compute, [!DNL Adobe I/O] Eventi, [!DNL Adobe I/O] Gestione eventi e [!DNL Adobe I/O] Runtime.

## Problemi di accesso tramite [!DNL Adobe I/O] CLI {#login-via-aio-cli}

In caso di problemi durante l&#39;accesso a [!DNL Adobe Developer Console] [tramite [!DNL Adobe I/O] CLI](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#3-signing-in-from-cli), aggiungi manualmente le credenziali necessarie per lo sviluppo, il test e la distribuzione dell&#39;applicazione personalizzata:

1. Passa al progetto Firefly e all&#39;area di lavoro in [Adobe Developer Console](https://console.adobe.io/) e premi **[!UICONTROL Download]** dall&#39;angolo in alto a destra. Apri questo file e salva questo JSON in un punto sicuro sul tuo computer.

1. Passa al file ENV nella tua applicazione Firefly.

1. Aggiungi le credenziali [!DNL Adobe I/O] Runtime. Ottieni le credenziali [!DNL Adobe I/O] A Runtime dal JSON scaricato. Le credenziali si trovano sotto `project.workspace.services.runtime`. Aggiungi le credenziali [!DNL Adobe I/O] Runtime nelle variabili `AIO_runtime_XXX` :

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Aggiungi il percorso assoluto al JSON scaricato al passaggio 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Imposta il resto delle [credenziali richieste](develop-custom-application.md) necessarie per lo strumento sviluppatore.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
