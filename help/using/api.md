---
title: "[!DNL Asset Compute Service] API HTTP"
description: "[!DNL Asset Compute Service] API HTTP per creare applicazioni personalizzate."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 3%

---

# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

L’utilizzo dell’API è limitato a scopi di sviluppo. L’API viene fornita come contesto durante lo sviluppo di applicazioni personalizzate. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] utilizza l’API per passare le informazioni di elaborazione a un’applicazione personalizzata. Per ulteriori informazioni, consulta [Utilizzare i microservizi delle risorse e i profili di elaborazione](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=it).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l&#39;uso con [!DNL Experience Manager] as a [!DNL Cloud Service].

Qualsiasi client del [!DNL Asset Compute Service] L’API HTTP deve seguire questo flusso di alto livello:

1. È stato eseguito il provisioning di un client come [!DNL Adobe Developer Console] progetto in un’organizzazione IMS. Ogni client separato (sistema o ambiente) richiede un proprio progetto separato al fine di separare il flusso di dati dell’evento.

1. Un client genera un token di accesso per l’account tecnico utilizzando [Autenticazione JWT (account di servizio)](https://www.adobe.io/authentication/auth-methods.html).

1. Un client chiama [`/register`](#register) una sola volta per recuperare l&#39;URL del diario.

1. Un client chiama [`/process`](#process-request) per ogni risorsa per la quale desidera generare rappresentazioni. La chiamata è asincrona.

1. Un cliente esegue regolarmente polling del giornale di registrazione a [ricevere eventi](#asynchronous-events). Riceve eventi per ogni rappresentazione richiesta quando la rappresentazione viene elaborata correttamente (`rendition_created` (tipo di evento) o in caso di errore (`rendition_failed` tipo di evento).

Il [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) Questo modulo semplifica l’utilizzo dell’API nel codice di Node.js.

## Autenticazione e autorizzazione {#authentication-and-authorization}

Tutte le API richiedono l’autenticazione tramite token di accesso. Le richieste devono impostare le seguenti intestazioni:

1. `Authorization` intestazione con token bearer, che è il token dell’account tecnico, ricevuto tramite [scambio JWT](https://www.adobe.io/authentication/auth-methods.html) dal progetto della console Adobe Developer. Il [ambiti](#scopes) sono documentati di seguito.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` con l’ID organizzazione IMS.

1. `x-api-key` con l’ID client di [!DNL Adobe Developers Console] progetto.

### Ambiti {#scopes}

Verifica che il token di accesso sia conforme ai seguenti ambiti:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Questi richiedono [!DNL Adobe Developer Console] progetto a cui iscriversi `Asset Compute`, `I/O Events`, e `I/O Management API` servizi. La ripartizione dei singoli ambiti è la seguente:

* Base
   * ambiti: `openid,AdobeID`

* Asset compute
   * metascope: `asset_compute_meta`
   * ambiti: `asset_compute,read_organizations`

* [!DNL Adobe I/O] Eventi
   * metascope: `event_receiver_api`
   * ambiti: `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] API di gestione
   * metascope: `ent_adobeio_sdk`
   * ambiti: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registrazione {#register}

Ogni client del [!DNL Asset Compute service] - una [!DNL Adobe Developer Console] progetto abbonato al servizio - obbligatorio [registrare](#register-request) prima di elaborare le richieste. Il passaggio di registrazione restituisce il giornale di registrazione eventi univoco necessario per recuperare gli eventi asincroni dall’elaborazione della rappresentazione.

Al termine del suo ciclo di vita, un client può [annulla registrazione](#unregister-request).

### Registra richiesta {#register-request}

Questa chiamata API configura una [!DNL Asset Compute] e fornisce l&#39;URL del giornale di registrazione eventi. Si tratta di un’operazione idempotente e deve essere chiamata solo una volta per ogni client. Può essere richiamato nuovamente per recuperare l’URL del diario.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/register` |
| Intestazione `Authorization` | Tutti [intestazioni relative alle autorizzazioni](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione tra i sistemi. |
| Corpo della richiesta | Deve essere vuoto. |

### Registra risposta {#register-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale al `X-Request-Id` intestazione di richiesta o un’intestazione generata in modo univoco. Utilizza per identificare le richieste tra sistemi e/o richieste di supporto. |
| Corpo della risposta | Un oggetto JSON con `journal`, `ok` e/o `requestId` campi. |

I codici di stato HTTP sono:

* **200 riuscito**: quando la richiesta ha esito positivo. Contiene il `journal` URL a cui devono essere notificati eventuali risultati dell’elaborazione asincrona attivata tramite `/process` (come tipo di eventi `rendition_created` in caso di esito positivo, oppure `rendition_failed` in caso di errore).

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Non autorizzato**: si verifica quando la richiesta non ha un valore valido [autenticazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 - Non consentito**: si verifica quando la richiesta non ha un valore valido [autorizzazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è abbonato a tutti i servizi richiesti.

* **429 Troppe richieste**: si verifica quando il sistema viene sovraccaricato da questo client o in altro modo. I client devono riprovare con un [backoff esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **Errore 4xx**: quando si è verificato un altro errore del client e la registrazione non è riuscita. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Errore 5xx**: si verifica in presenza di un errore sul lato server e registrazione non riuscita. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Annulla registrazione richiesta {#unregister-request}

Questa chiamata API annulla la registrazione di un [!DNL Asset Compute] client. In seguito non sarà più possibile chiamare `/process`. L’utilizzo della chiamata API per un client non registrato o per un client non ancora registrato restituisce un `404` errore.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/unregister` |
| Intestazione `Authorization` | Tutti [intestazioni relative alle autorizzazioni](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione tra i sistemi. |
| Corpo della richiesta | Vuoto. |

### Annulla registrazione risposta {#unregister-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale al `X-Request-Id` intestazione di richiesta o un’intestazione generata in modo univoco. Utilizza per identificare le richieste tra sistemi e/o richieste di supporto. |
| Corpo della risposta | Un oggetto JSON con `ok` e `requestId` campi. |

I codici di stato sono:

* **200 riuscito**: si verifica quando la registrazione e il giornale di registrazione vengono trovati e rimossi.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Non autorizzato**: si verifica quando la richiesta non ha un valore valido [autenticazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 - Non consentito**: si verifica quando la richiesta non ha un valore valido [autorizzazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è abbonato a tutti i servizi richiesti.

* **404 Non trovato**: si verifica in assenza di registrazione corrente per le credenziali specificate.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Troppe richieste**: si verifica quando il sistema è sovraccarico. I client devono riprovare con un [backoff esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.

* **Errore 4xx**: si verifica quando si è verificato un altro errore del client e l’annullamento della registrazione non è riuscito. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Errore 5xx**: si verifica in presenza di un errore sul lato server e registrazione non riuscita. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Richiesta di elaborazione {#process-request}

Il `process` L&#39;operazione sottomette un processo che trasforma una risorsa di origine in più rappresentazioni, in base alle istruzioni contenute nella richiesta. Notifiche di completamento (tipo di evento) `rendition_created`) o eventuali errori (tipo di evento `rendition_failed`) vengono inviate a un giornale di registrazione eventi che deve essere recuperato utilizzando [/register](#register) una volta prima di effettuare un numero qualsiasi di `/process` richieste. Le richieste formate in modo errato hanno immediatamente esito negativo con un codice di errore 400.

I riferimenti binari vengono eseguiti utilizzando URL, ad esempio URL prefirmati di Amazon AWS S3 o URL SAS di Azure Blob Storage, per entrambe le operazioni di lettura di `source` risorsa (`GET` URL e scrittura delle rappresentazioni (`PUT` URL). Il client è responsabile della generazione di questi URL prefirmati.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/process` |
| Tipo MIME | `application/json` |
| Intestazione `Authorization` | Tutti [intestazioni relative alle autorizzazioni](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione tra i sistemi. |
| Corpo della richiesta | Deve essere nel formato JSON della richiesta del processo come descritto di seguito. Fornisce istruzioni su quale risorsa elaborare e quali rappresentazioni generare. |

### Elabora JSON richiesta {#process-request-json}

Il corpo della richiesta di `/process` è un oggetto JSON con questo schema di alto livello:

```json
{
    "source": "",
    "renditions" : []
}
```

I campi disponibili sono:

| Nome | Tipo | Descrizione | Esempio |
|--------------|----------|-------------|---------|
| `source` | `string` | URL della risorsa di origine da elaborare. Facoltativo in base al formato di rendering richiesto (ad esempio `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descrizione della risorsa di origine da elaborare. Vedi la descrizione di [Campi oggetto di origine](#source-object-fields) di seguito. Facoltativo in base al formato di rendering richiesto (ad esempio `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Rappresentazioni da generare dal file di origine. Ogni oggetto di rendering supporta [istruzione di rendering](#rendition-instructions). Obbligatorio. | `[{ "target": "https://....", "fmt": "png" }]` |

Il `source` può essere un `<string>` che viene visualizzato come URL o può essere un `<object>` con un campo aggiuntivo. Le seguenti varianti sono simili:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Campi oggetto di origine {#source-object-fields}

| Nome | Tipo | Descrizione | Esempio |
|-----------|----------|-------------|---------|
| `url` | `string` | URL della risorsa di origine da elaborare. Obbligatorio. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nome del file della risorsa di origine. Se non è possibile rilevare alcun tipo MIME, è possibile utilizzare l’estensione file nel nome. Ha la precedenza sul nome file nel percorso URL o nel nome file in `content-disposition` intestazione della risorsa binaria. Impostazione predefinita: &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Dimensione del file della risorsa di origine in byte. Ha la precedenza su `content-length` intestazione della risorsa binaria. | `10234` |
| `mimetype` | `string` | Tipo MIME del file della risorsa di origine. Ha la precedenza sul `content-type` intestazione della risorsa binaria. | `"image/jpeg"` |

### Un completo `process` esempio di richiesta {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Risposta processo {#process-response}

Il `/process` La richiesta restituisce immediatamente un risultato positivo o negativo in base alla convalida della richiesta di base. L’elaborazione effettiva delle risorse avviene in modo asincrono.

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale al `X-Request-Id` intestazione di richiesta o un’intestazione generata in modo univoco. Utilizza per identificare le richieste tra sistemi e/o richieste di supporto. |
| Corpo della risposta | Un oggetto JSON con `ok` e `requestId` campi. |

Codici di stato:

* **200 riuscito**: se la richiesta è stata inviata correttamente. JSON di risposta include `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Richiesta non valida**: se la richiesta non è formata correttamente, ad esempio se mancano campi obbligatori nel JSON della richiesta. JSON di risposta include `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Non autorizzato**: quando la richiesta non ha un valore valido [autenticazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.
* **403 - Non consentito**: quando la richiesta non ha un valore valido [autorizzazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è abbonato a tutti i servizi richiesti.
* **429 Troppe richieste**: quando il sistema è sovraccarico da questo client o in generale. I client possono riprovare con un [backoff esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **Errore 4xx**: quando si è verificato un altro errore del client. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Errore 5xx**: quando si è verificato un altro errore lato server. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

La maggior parte dei client è probabilmente incline a riprovare la stessa richiesta con [backoff esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff) in caso di errori *eccetto* problemi di configurazione come 401 o 403, oppure richieste non valide come 400. A parte la limitazione regolare della frequenza tramite le risposte 429, un’interruzione temporanea del servizio o una limitazione potrebbero causare errori 5xx. Sarebbe quindi consigliabile riprovare dopo un certo periodo di tempo.

Tutte le risposte JSON (se presenti) includono `requestId` che è lo stesso valore del `X-Request-Id` intestazione. Si consiglia di leggere dall’intestazione, poiché è sempre presente. Il `requestId` viene restituito anche in tutti gli eventi relativi all’elaborazione delle richieste come `requestId`. I client non devono fare alcuna supposizione sul formato di questa stringa, è un identificatore di stringa opaco.

## Consenso alla post-elaborazione {#opt-in-to-post-processing}

Il [SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk) supporta una serie di opzioni di base per la post-elaborazione delle immagini. I lavoratori personalizzati possono acconsentire esplicitamente alla post-elaborazione impostando il campo `postProcess` sull&#39;oggetto rendering a `true`.

I casi d’uso supportati sono:

* Ritagliare una rappresentazione in un rettangolo i cui limiti sono definiti da crop.w, crop.h, crop.x e crop.y. È definito da `instructions.crop` nell&#39;oggetto rendering.
* Ridimensionare le immagini utilizzando la larghezza, l&#39;altezza o entrambe. È definito da `instructions.width` e `instructions.height` nell&#39;oggetto rendering. Per ridimensionare utilizzando solo la larghezza o l&#39;altezza, impostate un solo valore. Il servizio di elaborazione conserva le proporzioni.
* Imposta la qualità per un&#39;immagine JPEG. È definito da `instructions.quality` nell&#39;oggetto rendering. La qualità migliore è indicata da `100` e valori più bassi indicano una qualità ridotta.
* Creazione di immagini interlacciate. È definito da `instructions.interlace` nell&#39;oggetto rendering.
* Impostate DPI per regolare le dimensioni di rendering per la pubblicazione desktop regolando la scala applicata ai pixel. È definito da `instructions.dpi` nell&#39;oggetto rendering per modificare la risoluzione dpi. Tuttavia, per ridimensionare l&#39;immagine in modo che abbia le stesse dimensioni e una risoluzione diversa, utilizzare `convertToDpi` istruzioni.
* Ridimensiona l’immagine in modo che la larghezza o l’altezza di cui è stato eseguito il rendering rimanga invariata rispetto all’originale alla risoluzione di destinazione specificata (DPI). È definito da `instructions.convertToDpi` nell&#39;oggetto rendering.

## Risorse filigrana {#add-watermark}

Il [SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk) supporta l’aggiunta di una filigrana ai file immagine PNG, JPEG, TIFF e GIF. La filigrana viene aggiunta seguendo le istruzioni per la rappresentazione contenute nella sezione `watermark` nella rappresentazione.

La filigrana viene eseguita durante la post-elaborazione della rappresentazione. Per applicare una filigrana alle risorse, il lavoratore personalizzato [opta per la post-elaborazione](#opt-in-to-post-processing) impostando il campo `postProcess` sull&#39;oggetto rendering a `true`. Se il processo di lavoro non acconsente, la filigrana non viene applicata, anche se l&#39;oggetto filigrana è impostato sull&#39;oggetto di rendering nella richiesta.

## Istruzioni per la rappresentazione {#rendition-instructions}

Queste sono le opzioni disponibili per `renditions` array in [/process](#process-request).

### Campi comuni {#common-fields}

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Il formato di destinazione delle rappresentazioni, può anche essere `text` per l’estrazione di testo e `xmp` per l’estrazione di metadati XMP in formato xml. Consulta [formati supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL di un [applicazione personalizzata](develop-custom-application.md). Deve essere un `https://` URL. Se questo campo è presente, la rappresentazione viene creata da un’applicazione personalizzata. Qualsiasi altro campo di rendering impostato viene quindi utilizzato nell’applicazione personalizzata. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL in cui deve essere caricata la rappresentazione generata utilizzando HTTP PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Informazioni sul caricamento di URL prefirmati in più parti per la rappresentazione generata. Questo è per [Caricamento binario diretto AEM/Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con questo [comportamento caricamento multipart](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Campi:<ul><li>`urls`: array di stringhe, uno per ogni URL della parte prefirmato</li><li>`minPartSize`: dimensione minima da utilizzare per una parte = url</li><li>`maxPartSize`: dimensione massima da utilizzare per una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Spazio riservato facoltativo controllato dal client e trasmesso così com’è agli eventi di rendering. Consente ai client di aggiungere informazioni personalizzate per identificare gli eventi di rendering. Non deve essere modificata o utilizzata nelle applicazioni personalizzate, in quanto i client sono liberi di modificarla in qualsiasi momento. | `{ ... }` |

### Campi specifici della rappresentazione {#rendition-specific-fields}

Per un elenco dei formati di file attualmente supportati, consulta [formati di file supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `*` | `*` | È possibile aggiungere campi personalizzati avanzati che [applicazione personalizzata](develop-custom-application.md) capisce. | |
| `embedBinaryLimit` | `number` in byte | Se questo valore è impostato e la dimensione del file della rappresentazione è inferiore a questo valore, la rappresentazione viene incorporata nell’evento inviato una volta completata la generazione della rappresentazione. La dimensione massima consentita per l’incorporamento è 32 KB (32 x 1024 byte). Se le dimensioni di una rappresentazione sono maggiori di quelle `embedBinaryLimit` , viene inserito in una posizione nell’archiviazione cloud e non è incorporato nell’evento. | `3276` |
| `width` | `number` | Larghezza in pixel. solo per le rappresentazioni di immagini. | `200` |
| `height` | `number` | Altezza in pixel. solo per le rappresentazioni di immagini. | `200` |
|                   |          | Le proporzioni vengono sempre mantenute se: <ul> <li> Entrambi `width` e `height` , l&#39;immagine si adatta alle dimensioni mantenendo le proporzioni </li><li> Solo `width` o solo `height` , l&#39;immagine risultante utilizza la dimensione corrispondente mantenendo le proporzioni</li><li> Se nessuno dei due `width` né `height` viene utilizzata la dimensione in pixel dell&#39;immagine originale. Dipende dal tipo di origine. Per alcuni formati, ad esempio i file PDF, viene utilizzata una dimensione predefinita. Può essere presente un limite di dimensioni massimo.</li></ul> | |
| `quality` | `number` | Specifica la qualità JPEG nell&#39;intervallo `1` a `100`. Applicabile solo alle rappresentazioni di immagini. | `90` |
| `xmp` | `string` | Utilizzato solo dal writeback dei metadati dell&#39;XMP, l&#39;XMP con codifica base64 viene riscritto nella rappresentazione specificata. | |
| `interlace` | `bool` | Creare un file PNG o GIF interlacciato o progressive JPEG impostandolo su `true`. Non ha alcun effetto su altri formati di file. | |
| `jpegSize` | `number` | Dimensione approssimativa del file JPEG in byte. Sostituisce qualsiasi `quality` impostazione. Non ha alcun effetto su altri formati. | |
| `dpi` | `number` oppure `object` | Impostare x e y DPI. Per semplicità, può anche essere impostato su un singolo numero che viene utilizzato sia per x che per y. Non ha alcun effetto sull’immagine stessa. | `96` oppure `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` oppure `object` | x e y DPI ricampiona i valori mantenendo le dimensioni fisiche. Per semplicità, può anche essere impostato su un singolo numero che viene utilizzato sia per x che per y. | `96` oppure `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Elenco dei file da includere nell&#39;archivio ZIP (`fmt=zip`). Ogni voce può essere una stringa URL o un oggetto con i campi:<ul><li>`url`: URL per scaricare il file</li><li>`path`: archivia il file nel percorso specificato nel file ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Gestione duplicata per archivi ZIP (`fmt=zip`). Per impostazione predefinita, più file memorizzati nello stesso percorso nel file ZIP generano un errore. Impostazione `duplicate` a `ignore` fa sì che venga memorizzata solo la prima risorsa e ignorato il resto. | `ignore` |
| `watermark` | `object` | Contiene istruzioni su [filigrana](#watermark-specific-fields). |  |

### Campi specifici della filigrana {#watermark-specific-fields}

Il formato PNG viene utilizzato come filigrana.

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Scala della filigrana, tra `0.0` e `1.0`. `1.0` indica che la filigrana ha la scala originale (1:1) e che i valori più bassi ne riducono la dimensione. | Un valore di `0.5` significa metà delle dimensioni originali. |
| `image` | `url` | URL del file PNG da utilizzare per la filigrana. | |

## Eventi asincroni {#asynchronous-events}

Al termine dell’elaborazione di una rappresentazione o quando si verifica un errore, un evento viene inviato a un [[!DNL Adobe I/O] Giornale di registrazione eventi](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). I client devono ascoltare l&#39;URL del diario fornito tramite [/register](#register). La risposta del giornale di registrazione include `event` array costituito da un oggetto per ogni evento, di cui `event` include il payload dell’evento effettivo.

Il [!DNL Adobe I/O] Tipo di evento per tutti gli eventi del [!DNL Asset Compute Service] è `asset_compute`. Il giornale di registrazione viene automaticamente iscritto solo a questo tipo di evento e non è necessario filtrare ulteriormente in base al [!DNL Adobe I/O] Tipo di evento. I tipi di evento specifici del servizio sono disponibili nel `type` proprietà dell’evento.

### Tipi di evento {#event-types}

| Evento | Descrizione |
|---------------------|-------------|
| `rendition_created` | Inviato per ogni rappresentazione elaborata e caricata correttamente. |
| `rendition_failed` | Inviato per ogni rappresentazione che non è stata elaborata o caricata. |

### Attributi evento {#event-attributes}

| Attributo | Tipo | Evento | Descrizione |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Timestamp in cui l’evento è stato inviato in modalità semplificata estesa [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) formato, come definito da JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | ID della richiesta originale a `/process`, come `X-Request-Id` intestazione. |
| `source` | `object` | `*` | Il `source` del `/process` richiesta. |
| `userData` | `object` | `*` | Il `userData` della rappresentazione dal `/process` richiesta se impostata. |
| `rendition` | `object` | `rendition_*` | Oggetto di rendering corrispondente trasmesso `/process`. |
| `metadata` | `object` | `rendition_created` | Il [metadati](#metadata) proprietà della rappresentazione. |
| `errorReason` | `string` | `rendition_failed` | Errore rappresentazione [motivo](#error-reasons) se presente. |
| `errorMessage` | `string` | `rendition_failed` | Testo con maggiori dettagli sull’eventuale errore di rappresentazione. |

### Metadati {#metadata}

| Proprietà | Descrizione |
|--------|-------------|
| `repo:size` | Dimensione della rappresentazione in byte. |
| `repo:sha1` | Il digest sha1 della rappresentazione. |
| `dc:format` | Tipo MIME della rappresentazione. |
| `repo:encoding` | La codifica charset della rappresentazione nel caso sia un formato basato su testo. |
| `tiff:ImageWidth` | Larghezza della rappresentazione in pixel. Presente solo per le rappresentazioni di immagini. |
| `tiff:ImageLength` | Lunghezza della rappresentazione in pixel. Presente solo per le rappresentazioni di immagini. |

### Motivi di errore {#error-reasons}

| Motivo | Descrizione |
|---------|-------------|
| `RenditionFormatUnsupported` | Il formato di rappresentazione richiesto non è supportato per l’origine specificata. |
| `SourceUnsupported` | L’origine specifica non è supportata anche se il tipo è supportato. |
| `SourceCorrupt` | Dati di origine danneggiati. Include i file vuoti. |
| `RenditionTooLarge` | Impossibile caricare la rappresentazione utilizzando gli URL prefirmati forniti in `target`. La dimensione effettiva della rappresentazione è disponibile come metadati in `repo:size` e possono essere utilizzati dal client per rielaborare la rappresentazione con il numero corretto di URL prefirmati. |
| `GenericError` | Qualsiasi altro errore imprevisto. |
