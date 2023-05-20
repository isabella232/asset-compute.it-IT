---
title: Comprendere il funzionamento di un’applicazione personalizzata
description: Funzionamento interno di [!DNL Asset Compute Service] applicazione personalizzata per comprendere come funziona.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 2af710443cdc2e5e25e105eca6e779eb58631ae9
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# Interni di un’applicazione personalizzata {#how-custom-application-works}

Utilizza l’illustrazione seguente per comprendere il flusso di lavoro end-to-end quando una risorsa digitale viene elaborata da un client utilizzando un’applicazione personalizzata.

![Flusso di lavoro dell&#39;applicazione personalizzato](assets/customworker.svg)

*Figura: Passaggi necessari per elaborare una risorsa utilizzando [!DNL Asset Compute Service].*

## Registrazione {#registration}

Il client deve chiamare [`/register`](api.md#register) una volta prima della prima richiesta a [`/process`](api.md#process-request) per impostare e recuperare l&#39;URL del giornale di registrazione per la ricezione [!DNL Adobe I/O] Ad Adobe Asset compute, gli eventi.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

Il [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) La libreria JavaScript può essere utilizzata nelle applicazioni NodeJS per gestire tutti i passaggi necessari, dalla registrazione all’elaborazione fino alla gestione asincrona degli eventi. Per ulteriori informazioni sulle intestazioni richieste, consulta [Autenticazione e autorizzazione](api.md).

## Elaborazione {#processing}

Il client invia una [elaborazione](api.md#process-request) richiesta.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Il client è responsabile della corretta formattazione delle rappresentazioni con URL prefirmati. Il [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) La libreria JavaScript può essere utilizzata nelle applicazioni NodeJS per pre-firmare gli URL. Attualmente la libreria supporta solo l’archiviazione BLOB di Azure e i contenitori AWS S3.

La richiesta di elaborazione restituisce un `requestId` che può essere utilizzato per il polling [!DNL Adobe I/O] Eventi.

Di seguito è riportato un esempio di richiesta di elaborazione personalizzata dell’applicazione.

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

Il [!DNL Asset Compute Service] invia le richieste di rendering dell’applicazione personalizzata all’applicazione personalizzata. Utilizza un POST HTTP per l’URL dell’applicazione fornito, che è l’URL dell’azione web protetta di App Builder. Tutte le richieste utilizzano il protocollo HTTPS per massimizzare la sicurezza dei dati.

Il [SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) utilizzato da un’applicazione personalizzata gestisce la richiesta HTTP POST. Gestisce anche il download dell’origine, il caricamento delle rappresentazioni e l’invio [!DNL Adobe I/O] e la gestione degli errori.

<!-- TBD: Add the application diagram. -->

### Codice applicazione {#application-code}

Il codice personalizzato deve fornire solo un callback che accetta il file di origine disponibile localmente (`source.path`). Il `rendition.path` è la posizione in cui inserire il risultato finale di una richiesta di elaborazione di risorse. L&#39;applicazione personalizzata utilizza il callback per trasformare i file di origine disponibili localmente in un file di copia trasformata utilizzando il nome passato (`rendition.path`). Un&#39;applicazione personalizzata deve scrivere in `rendition.path` per creare una rappresentazione:

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

### Scarica i file sorgente {#download-source}

Un’applicazione personalizzata tratta solo i file locali. Il download del file di origine viene gestito da [SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Creazione rappresentazione {#rendition-creation}

L’SDK chiama un’istanza asincrona [funzione di callback della rappresentazione](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) per ogni rappresentazione.

La funzione di callback ha accesso al [sorgente](https://github.com/adobe/asset-compute-sdk#source) e [rendering](https://github.com/adobe/asset-compute-sdk#rendition) oggetti. Il `source.path` esiste già ed è il percorso della copia locale del file di origine. Il `rendition.path` è il percorso in cui deve essere memorizzata la rappresentazione elaborata. A meno che [flag disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) è impostato, l&#39;applicazione deve utilizzare esattamente `rendition.path`. In caso contrario, l’SDK non è in grado di individuare o identificare il file di rappresentazione e genera un errore.

L&#39;eccessiva semplificazione dell&#39;esempio viene eseguita per illustrare e concentrarsi sull&#39;anatomia di un&#39;applicazione personalizzata. L&#39;applicazione copia semplicemente il file di origine nella destinazione della copia trasformata.

Per ulteriori informazioni sui parametri di callback della rappresentazione, consulta [API SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk#api-details).

### Carica rappresentazioni {#upload-rendition}

Dopo aver creato e memorizzato ogni copia trasformata in un file con il percorso fornito da `rendition.path`, il [SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carica ogni rendering in un’archiviazione cloud (AWS o Azure). Un’applicazione personalizzata ottiene più rappresentazioni contemporaneamente se, e solo se, la richiesta in ingresso ha più rappresentazioni che puntano allo stesso URL dell’applicazione. Il caricamento nell’archiviazione cloud viene eseguito dopo ogni rendering e prima di eseguire il callback per il rendering successivo.

Il `batchWorker()` ha un comportamento diverso, in quanto elabora effettivamente tutte le rappresentazioni e solo dopo che tutte sono state elaborate carica quelle.

## [!DNL Adobe I/O] Eventi {#aio-events}

L’SDK invia [!DNL Adobe I/O] Eventi per ogni rappresentazione. Questi eventi sono di entrambi i tipi `rendition_created` o `rendition_failed` a seconda del risultato. Consulta [Asset compute di eventi asincroni](api.md#asynchronous-events) per i dettagli degli eventi.

## Ricezione [!DNL Adobe I/O] Eventi {#receive-aio-events}

Il client esegue il polling di [[!DNL Adobe I/O] Giornale di registrazione eventi](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) in base alla sua logica di consumo. L&#39;URL iniziale del giornale di registrazione è quello fornito in `/register` Risposta API. Gli eventi possono essere identificati utilizzando `requestId` presente negli eventi e uguale a quello restituito in `/process`. Ogni rendering ha un evento separato che viene inviato non appena il rendering è stato caricato (o non è riuscito). Una volta ricevuto un evento corrispondente, il client può visualizzare o gestire in altro modo le rappresentazioni risultanti.

Libreria JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) semplifica il polling del diario utilizzando `waitActivation()` per ottenere tutti gli eventi.

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

Per informazioni dettagliate su come ottenere gli eventi del diario, vedere [[!DNL Adobe I/O] API Eventi](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
