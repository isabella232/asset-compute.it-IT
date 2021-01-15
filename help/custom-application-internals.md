---
title: Comprendere il funzionamento di un'applicazione personalizzata.
description: Uso interno dell'applicazione personalizzata  [!DNL Asset Compute Service] per capire come funziona.
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---


# Interni di un&#39;applicazione personalizzata {#how-custom-application-works}

Utilizzate l&#39;illustrazione seguente per comprendere il flusso di lavoro end-to-end quando una risorsa digitale viene elaborata utilizzando un&#39;applicazione personalizzata da un client.

![Flusso di lavoro applicazione personalizzato](assets/customworker.png)

*Figura: Procedura per elaborare una risorsa con  [!DNL Asset Compute Service].*

## Registrazione {#registration}

Il client deve chiamare [`/register`](api.md#register) una volta prima della prima richiesta a [`/process`](api.md#process-request) per impostare e recuperare l&#39;URL del giornale di registrazione per la ricezione di eventi [!DNL Adobe I/O] per  Adobe  Asset compute.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

La [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) libreria JavaScript può essere utilizzata nelle applicazioni NodeJS per gestire tutti i passaggi necessari, dalla registrazione all&#39;elaborazione alla gestione degli eventi asincroni. Per ulteriori informazioni sulle intestazioni richieste, vedere [Autenticazione e autorizzazione](api.md).

## Elaborazione {#processing}

Il client invia una richiesta di [elaborazione](api.md#process-request).

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Il client è responsabile della corretta formattazione delle rappresentazioni con URL pre-firmati. La [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) libreria JavaScript può essere utilizzata nelle applicazioni NodeJS per pre-firmare gli URL. Attualmente la libreria supporta solo Azure Blob Storage e AWS S3 Containers.

La richiesta di elaborazione restituisce un `requestId` che può essere utilizzato per il polling di eventi [!DNL Adobe I/O].

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

[!DNL Asset Compute Service] invia le richieste di rappresentazione dell&#39;applicazione personalizzata all&#39;applicazione personalizzata. Utilizza un POST HTTP per l’URL dell’applicazione fornito, ovvero l’URL dell’azione Web protetta da Project Firefly. Tutte le richieste utilizzano il protocollo HTTPS per massimizzare la sicurezza dei dati.

L&#39;SDK [ Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) utilizzato da un&#39;applicazione personalizzata gestisce la richiesta di POST HTTP. Gestisce inoltre il download dell&#39;origine, il caricamento di rappresentazioni, l&#39;invio di eventi [!DNL Adobe I/O] e la gestione degli errori.

<!-- TBD: Add the application diagram. -->

### Codice applicazione {#application-code}

Solo il codice personalizzato deve fornire un callback che prende il file di origine localmente disponibile (`source.path`). `rendition.path` è il percorso in cui inserire il risultato finale di una richiesta di elaborazione risorse. L&#39;applicazione personalizzata utilizza il callback per trasformare i file sorgente disponibili localmente in un file di rappresentazione utilizzando il nome passato (`rendition.path`). Per creare una rappresentazione, un&#39;applicazione personalizzata deve scrivere in `rendition.path`:

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

### Download dei file sorgente {#download-source}

Un&#39;applicazione personalizzata si occupa solo dei file locali. Il download del file di origine viene gestito dall&#39;SDK del Asset compute [](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Creazione di rappresentazioni {#rendition-creation}

L&#39;SDK chiama una funzione di callback di rappresentazione [asincrona](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) per ogni rappresentazione.

La funzione di callback ha accesso agli oggetti [source](https://github.com/adobe/asset-compute-sdk#source) e [rappresentazione](https://github.com/adobe/asset-compute-sdk#rendition). `source.path` esiste già ed è il percorso della copia locale del file di origine. `rendition.path` è il percorso in cui deve essere memorizzata la rappresentazione elaborata. A meno che il flag [disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) non sia impostato, l&#39;applicazione deve utilizzare esattamente il flag `rendition.path`. In caso contrario, l’SDK non è in grado di individuare o identificare il file di rappresentazione e non riesce.

La semplificazione eccessiva dell&#39;esempio viene fatta per illustrare e concentrarsi sull&#39;anatomia di un&#39;applicazione personalizzata. L&#39;applicazione copia semplicemente il file di origine nella destinazione di rappresentazione.

Per ulteriori informazioni sui parametri di callback di rappresentazione, consultate [ Asset compute SDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### Caricare le rappresentazioni {#upload-rendition}

Dopo che ciascuna rappresentazione è stata creata e memorizzata in un file con il percorso fornito da `rendition.path`, l&#39;SDK [ Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carica ciascuna rappresentazione in un archivio cloud (AWS o Azure). Un&#39;applicazione personalizzata ottiene più rappresentazioni contemporaneamente se, e solo se, la richiesta in entrata dispone di più rappresentazioni che puntano allo stesso URL dell&#39;applicazione. Il caricamento nell’archiviazione cloud viene eseguito dopo ciascuna rappresentazione e prima di eseguire il callback per la rappresentazione successiva.

Il `batchWorker()` ha un comportamento diverso, in quanto elabora effettivamente tutte le rappresentazioni e solo dopo che sono state elaborate, le carica.

## [!DNL Adobe I/O] Eventi {#aio-events}

L&#39;SDK invia [!DNL Adobe I/O] eventi per ogni rappresentazione. Questi eventi possono essere di tipo `rendition_created` o `rendition_failed` a seconda del risultato. Per i dettagli sugli eventi, vedere [ eventi asincroni di Asset compute](api.md#asynchronous-events).

## Ricevi eventi [!DNL Adobe I/O] {#receive-aio-events}

Il client esegue il polling di [[!DNL Adobe I/O] Events Journal](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) in base alla logica di consumo. L&#39;URL del journal iniziale è quello fornito nella risposta API `/register`. Gli eventi possono essere identificati utilizzando il simbolo `requestId` presente negli eventi ed è uguale a quello restituito in `/process`. Ogni rappresentazione dispone di un evento separato che viene inviato non appena la rappresentazione è stata caricata (o non è riuscita). Una volta ricevuto un evento corrispondente, il client può visualizzare o gestire in altro modo le rappresentazioni risultanti.

La libreria JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) semplifica il polling delle scritture contabili mediante il metodo `waitActivation()` per ottenere tutti gli eventi.

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

Per informazioni dettagliate su come ottenere gli eventi del journal, vedere [[!DNL Adobe I/O] Eventi API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
