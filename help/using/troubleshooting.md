---
title: Risoluzione dei problemi [!DNL Asset Compute Service]
description: Risolvere i problemi ed eseguire il debug di applicazioni personalizzate tramite [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# Risoluzione dei problemi {#troubleshoot}

Di seguito sono riportati alcuni suggerimenti generici per la risoluzione dei problemi relativi al servizio Asset compute:

* Assicurarsi che l&#39;applicazione JavaScript non si arresti in modo anomalo all&#39;avvio. Tali arresti anomali sono in genere correlati a una libreria mancante o a una dipendenza.
* Assicurarsi che tutte le dipendenze da installare siano indicate nel file `package.json` file.
* Assicurati che eventuali errori derivanti dalla pulizia in caso di errore non generino errori che nascondono il problema originale.

* Quando si avvia lo strumento per sviluppatori per la prima volta con un nuovo [!DNL Asset Compute Service] integrazione, la prima richiesta di elaborazione potrebbe non riuscire se il journal degli eventi di Asset compute non è completamente configurato. Attendi che il giornale di registrazione venga configurato prima di inviare un’altra richiesta.
* Se riscontri errori durante l’invio dell’Asset compute `/register` o `/process` , assicurati che tutte le API necessarie siano aggiunte al [!DNL Adobe I/O] Progetto e Workspace, ovvero Asset compute, [!DNL Adobe I/O] Eventi, [!DNL Adobe I/O] Gestione degli eventi e [!DNL Adobe I/O] Runtime.

## Problemi di accesso tramite [!DNL Adobe I/O] CLI {#login-via-aio-cli}

In caso di problemi di accesso a [!DNL Adobe Developer Console] [tramite [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli), quindi aggiungi manualmente le credenziali necessarie per sviluppare, testare e distribuire l’applicazione personalizzata:

1. Passa al progetto Adobe Developer App Builder e all’area di lavoro nella sezione [Console Adobe Developer](https://console.adobe.io/) e premere **[!UICONTROL Scarica]** dall&#39;angolo in alto a destra. Apri questo file e salva JSON in un luogo sicuro sul computer.

1. Passa al file ENV nell’applicazione Adobe Developer App Builder.

1. Aggiungi il [!DNL Adobe I/O] Credenziali runtime. Ottieni [!DNL Adobe I/O] Credenziali di runtime dal JSON scaricato. Le credenziali sono sotto `project.workspace.services.runtime`. Aggiungi il [!DNL Adobe I/O] Credenziali runtime in `AIO_runtime_XXX` Variabili:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Aggiungi il percorso assoluto al JSON scaricato nel passaggio 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Imposta il resto del [credenziali richieste](develop-custom-application.md) necessario per lo strumento per sviluppatori.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
