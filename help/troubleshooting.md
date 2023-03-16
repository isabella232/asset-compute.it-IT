---
title: Risoluzione dei problemi [!DNL Asset Compute Service]
description: Risolvere i problemi e eseguire il debug delle applicazioni personalizzate utilizzando [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# Risoluzione dei problemi {#troubleshoot}

Alcuni suggerimenti generici per la risoluzione dei problemi che possono essere utili per la risoluzione dei problemi con Asset compute Service sono:

* Assicurati che l&#39;applicazione JavaScript non si arresti all&#39;avvio. Tali arresti anomali sono solitamente correlati a una libreria mancante o a una dipendenza.
* Assicurati che tutte le dipendenze da installare siano referenziate nel `package.json` file.
* Assicurati che eventuali errori derivanti dalla pulizia in caso di errore non generino errori propri che nascondono il problema originale.

* Quando si avvia lo strumento di sviluppo per la prima volta con un nuovo [!DNL Asset Compute Service] integrazione, potrebbe non riuscire la prima richiesta di elaborazione se Asset compute Events Journal non è completamente configurato. Attendi l&#39;impostazione del journal prima di inviare un&#39;altra richiesta.
* Se si verificano degli errori durante l’invio di un Asset compute `/register` o `/process` , assicurati che tutte le API necessarie siano aggiunte al [!DNL Adobe I/O] Progetto e area di lavoro, ad Asset compute, [!DNL Adobe I/O] Eventi, [!DNL Adobe I/O] Gestione degli eventi e [!DNL Adobe I/O] Runtime.

## Problemi di accesso tramite [!DNL Adobe I/O] CLI {#login-via-aio-cli}

In caso di problemi di accesso al [!DNL Adobe Developer Console] [attraverso [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli), quindi aggiungi manualmente le credenziali necessarie per sviluppare, testare e distribuire l&#39;applicazione personalizzata:

1. Passa al progetto Adobe Developer App Builder e all’area di lavoro nel [Console Adobe Developer](https://console.adobe.io/) e premere **[!UICONTROL Scarica]** dall&#39;angolo in alto a destra. Apri questo file e salva questo JSON in un punto sicuro sul tuo computer.

1. Passa al file ENV nell’applicazione Adobe Developer App Builder.

1. Aggiungi il [!DNL Adobe I/O] Credenziali runtime. Ottieni il [!DNL Adobe I/O] Credenziali runtime dal JSON scaricato. Le credenziali sono sotto `project.workspace.services.runtime`. Aggiungi il [!DNL Adobe I/O] Credenziali runtime nel `AIO_runtime_XXX` variabili:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Aggiungi il percorso assoluto al JSON scaricato al passaggio 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Imposta il resto del [credenziali obbligatorie](develop-custom-application.md) necessario per lo strumento di sviluppo.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
