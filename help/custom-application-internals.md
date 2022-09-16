---
title: Comprendere il funzionamento di un’applicazione personalizzata
description: Lavoro interno [!DNL Asset Compute Service] applicazione personalizzata per comprendere il funzionamento.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: a121b48d480b45405259c2061ac86b9ab46b89cb
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# Interni di un&#39;applicazione personalizzata {#how-custom-application-works}

Utilizza l’illustrazione seguente per comprendere il flusso di lavoro end-to-end quando una risorsa digitale viene elaborata utilizzando un’applicazione personalizzata da un client.

![Flusso di lavoro dell’applicazione personalizzata](assets/customworker.png)

*Figura: Passaggi coinvolti nell’elaborazione di una risorsa utilizzando [!DNL Asset Compute Service].*

## Registrazione {#registration}

Il client deve chiamare [`/register`](api.md#register) una volta prima della prima richiesta a [`/process`](api.md#process-request) per impostare e recuperare l&#39;URL del giornale di registrazione per la ricezione [!DNL Adobe I/O] Eventi, ad Adobe.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

La [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) La libreria JavaScript può essere utilizzata nelle applicazioni NodeJS per gestire tutti i passaggi necessari dalla registrazione, dall’elaborazione alla gestione asincrona degli eventi. Per ulteriori informazioni sulle intestazioni richieste, vedi [Autenticazione e autorizzazione](api.md).

## Elaborazione {#processing}

Il client invia un [elaborazione](api.md#process-request) richiesta.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Il client è responsabile della corretta formattazione dei rendering con URL pre-firmati. La [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) La libreria JavaScript può essere utilizzata nelle applicazioni NodeJS per pre-firmare gli URL. Attualmente la libreria supporta solo l’archiviazione BLOB di Azure e i contenitori AWS S3.

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

La [!DNL Asset Compute Service] invia le richieste di rendering dell’applicazione personalizzata all’applicazione personalizzata. Utilizza un POST HTTP per l&#39;URL dell&#39;applicazione fornito, che è l&#39;URL dell&#39;azione web protetto da App Builder. Tutte le richieste utilizzano il protocollo HTTPS per massimizzare la sicurezza dei dati.

La [asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) utilizzato da un&#39;applicazione personalizzata per gestire la richiesta HTTP POST. Gestisce anche il download della sorgente, il caricamento di rappresentazioni, l&#39;invio [!DNL Adobe I/O] eventi e gestione degli errori.

<!-- TBD: Add the application diagram. -->

### Codice applicazione {#application-code}

Il codice personalizzato deve solo fornire un callback che prende il file di origine disponibile localmente (`source.path`). La `rendition.path` è la posizione in cui inserire il risultato finale di una richiesta di elaborazione delle risorse. L&#39;applicazione personalizzata utilizza il callback per trasformare i file di origine disponibili localmente in un file di rendering utilizzando il nome passato (`rendition.path`). Un&#39;applicazione personalizzata deve scrivere in `rendition.path` per creare un rendering:

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

Un&#39;applicazione personalizzata si occupa solo dei file locali. Il download del file di origine viene gestito dalla [asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Creazione di copie trasformate {#rendition-creation}

L’SDK chiama un [funzione di callback di rendering](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) per ogni rendering.

La funzione di callback ha accesso al [source](https://github.com/adobe/asset-compute-sdk#source) e [rendering](https://github.com/adobe/asset-compute-sdk#rendition) oggetti. La `source.path` esiste già ed è il percorso della copia locale del file di origine. La `rendition.path` è il percorso in cui deve essere memorizzato il rendering elaborato. A meno che [flag disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) è impostato, l&#39;applicazione deve utilizzare esattamente il `rendition.path`. In caso contrario, l&#39;SDK non può individuare o identificare il file di rendering e non riesce.

La semplificazione eccessiva dell&#39;esempio viene fatta per illustrare e concentrarsi sull&#39;anatomia di un&#39;applicazione personalizzata. L&#39;applicazione copia semplicemente il file di origine nella destinazione di rendering.

Per ulteriori informazioni sui parametri di callback di rendering, vedi [API SDK di Asset compute](https://github.com/adobe/asset-compute-sdk#api-details).

### Caricare rappresentazioni {#upload-rendition}

Dopo ogni rendering viene creato e memorizzato in un file con il percorso fornito da `rendition.path`, [asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carica ogni rendering in un archivio cloud (AWS o Azure). Un&#39;applicazione personalizzata ottiene più rappresentazioni contemporaneamente se e solo se, la richiesta in arrivo ha più rappresentazioni che puntano allo stesso URL dell&#39;applicazione. Il caricamento nell’archiviazione cloud viene eseguito dopo ogni rendering e prima di eseguire il callback per il rendering successivo.

La `batchWorker()` ha un comportamento diverso, in quanto elabora effettivamente tutte le rappresentazioni e solo dopo che tutte le sono state elaborate le carica.

## [!DNL Adobe I/O] Eventi {#aio-events}

L’SDK invia [!DNL Adobe I/O] Eventi per ogni rappresentazione. Questi eventi possono essere di tipo `rendition_created` o `rendition_failed` a seconda del risultato. Vedi [asset compute di eventi asincroni](api.md#asynchronous-events) per i dettagli sugli eventi.

## Ricezione [!DNL Adobe I/O] Eventi {#receive-aio-events}

Il client controlla il [[!DNL Adobe I/O] Journal eventi](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) secondo la sua logica di consumo. L&#39;URL del journal iniziale è quello fornito nella `/register` Risposta API. Gli eventi possono essere identificati utilizzando `requestId` presente negli eventi ed è lo stesso restituito in `/process`. Ogni rendering dispone di un evento separato che viene inviato non appena il rendering è stato caricato (o non riuscito). Una volta ricevuto un evento corrispondente, il client può visualizzare o gestire in altro modo i rendering risultanti.

Libreria JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) rende il polling del journal semplice utilizzando `waitActivation()` per ottenere tutti gli eventi.

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

Per informazioni dettagliate su come ottenere gli eventi del giornale, vedere [[!DNL Adobe I/O] API eventi](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
