---
title: Comprendere il funzionamento di un'applicazione personalizzata.
description: Utilizzo interno [!DNL Asset Compute Service] dell'applicazione personalizzata per agevolare la comprensione del funzionamento.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '774'
ht-degree: 0%

---


# Interni di un&#39;applicazione personalizzata {#how-custom-application-works}

Utilizzate l&#39;illustrazione seguente per comprendere il flusso di lavoro end-to-end quando una risorsa digitale viene elaborata utilizzando un&#39;applicazione personalizzata da un client.

![Flusso di lavoro applicazione personalizzato](assets/customworker.png)

*Figura: Procedura per elaborare una risorsa con [!DNL Asset Compute Service].*

## Registrazione {#registration}

Il client deve chiamare [`/register`](api.md#register) una volta prima della prima richiesta a [`/process`](api.md#process-request) per impostare e recuperare l&#39;URL del giornale di registrazione per la ricezione  eventi I/O Adobe per  calcolo risorse Adobe.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

La libreria [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript può essere utilizzata nelle applicazioni NodeJS per gestire tutti i passaggi necessari, dalla registrazione all&#39;elaborazione alla gestione degli eventi asincrona. Per ulteriori informazioni sulle intestazioni richieste, vedere [Autenticazione e Autorizzazione](api.md).

## Elaborazione {#processing}

Il client invia una richiesta di [elaborazione](api.md#process-request) .

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Il client è responsabile della corretta formattazione delle rappresentazioni con URL pre-firmati. La libreria [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript può essere utilizzata nelle applicazioni NodeJS per pre-firmare gli URL. Attualmente la libreria supporta solo Azure Blob Storage e AWS S3 Containers.

La richiesta di elaborazione restituisce un elemento `requestId` che può essere utilizzato per il polling  eventi I/O Adobe.

Segue un esempio di richiesta di elaborazione personalizzata dell&#39;applicazione.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

Invia [!DNL Asset Compute Service] le richieste di rappresentazione dell&#39;applicazione personalizzata all&#39;applicazione personalizzata. Utilizza un POST HTTP per l’URL dell’applicazione fornito, ovvero l’URL dell’azione Web protetta da Project Firefly. Tutte le richieste utilizzano il protocollo HTTPS per massimizzare la sicurezza dei dati.

L’SDK [per elaborazione](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) risorse utilizzato da un’applicazione personalizzata gestisce la richiesta di POST HTTP. Gestisce inoltre il download dell’origine, il caricamento di rappresentazioni, l’invio di eventi I/O e la gestione degli errori.

<!-- TBD: Add the application diagram. -->

### Application code {#application-code}

Solo il codice personalizzato deve fornire un callback che prende il file di origine (`source.path`) disponibile localmente. La posizione `rendition.path` è la posizione in cui inserire il risultato finale di una richiesta di elaborazione risorse. L&#39;applicazione personalizzata utilizza il callback per trasformare i file sorgente disponibili localmente in un file di rappresentazione utilizzando il nome passato (`rendition.path`). Per creare una rappresentazione, un&#39;applicazione personalizzata deve scrivere `rendition.path` in:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Scaricare i file sorgente {#download-source}

Un&#39;applicazione personalizzata si occupa solo dei file locali. Il download del file sorgente viene gestito dall’SDK [per elaborazione](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)risorse.

### Creazione di rappresentazioni {#rendition-creation}

L’SDK chiama una funzione [di callback di](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) rappresentazione asincrona per ciascuna rappresentazione.

La funzione di callback ha accesso agli oggetti [di origine](https://github.com/adobe/asset-compute-sdk#source) e di [rappresentazione](https://github.com/adobe/asset-compute-sdk#rendition) . Esiste `source.path` già ed è il percorso della copia locale del file di origine. Percorso `rendition.path` in cui deve essere memorizzata la rappresentazione elaborata. A meno che il flag [](https://github.com/adobe/asset-compute-sdk#worker-options-optional) disableSourceDownload non sia impostato, l&#39;applicazione deve utilizzare esattamente il `rendition.path`. In caso contrario, l’SDK non è in grado di individuare o identificare il file di rappresentazione e non riesce.

La semplificazione eccessiva dell&#39;esempio viene fatta per illustrare e concentrarsi sull&#39;anatomia di un&#39;applicazione personalizzata. L&#39;applicazione copia semplicemente il file di origine nella destinazione di rappresentazione.

Per ulteriori informazioni sui parametri di callback di rappresentazione, consultate API [SDK per elaborazione](https://github.com/adobe/asset-compute-sdk#api-details)risorse.

### Caricare le rappresentazioni {#upload-rendition}

Dopo che ciascuna rappresentazione è stata creata e memorizzata in un file con il percorso fornito da `rendition.path`, [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carica ciascuna rappresentazione in un archivio cloud (AWS o Azure). Un&#39;applicazione personalizzata ottiene più rappresentazioni contemporaneamente se, e solo se, la richiesta in entrata dispone di più rappresentazioni che puntano allo stesso URL dell&#39;applicazione. Il caricamento nell’archiviazione cloud viene eseguito dopo ciascuna rappresentazione e prima di eseguire il callback per la rappresentazione successiva.

Il comportamento `batchWorker()` è diverso, in quanto elabora effettivamente tutte le rappresentazioni e solo dopo che sono state elaborate, le carica.

##  eventi I/O Adobe {#aio-events}

L’SDK invia  eventi I/O Adobe per ogni rappresentazione. Questi eventi possono essere di tipo `rendition_created` o `rendition_failed` a seconda del risultato. Consultate Eventi [asincroni di calcolo delle](api.md#asynchronous-events) risorse per i dettagli degli eventi.

## Ricezione  eventi I/O Adobe {#receive-aio-events}

Il client esegue il polling del giornale di registrazione [eventi I/O](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) Adobe in base alla logica di consumo. L&#39;URL del journal iniziale è quello fornito nella risposta `/register` API. Gli eventi possono essere identificati utilizzando gli eventi `requestId` presenti negli eventi ed è lo stesso di quelli restituiti in `/process`. Ogni rappresentazione dispone di un evento separato che viene inviato non appena la rappresentazione è stata caricata (o non è riuscita). Una volta ricevuto un evento corrispondente, il client può visualizzare o gestire in altro modo le rappresentazioni risultanti.

La libreria JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) semplifica il polling delle scritture contabili utilizzando il `waitActivation()` metodo per ottenere tutti gli eventi.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Per informazioni dettagliate su come ottenere gli eventi del diario, consultate [Adobe API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml)eventi I/O.

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
