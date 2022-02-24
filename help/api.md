---
title: '"[!DNL Asset Compute Service] API HTTP"'
description: '"[!DNL Asset Compute Service] API HTTP per creare applicazioni personalizzate."'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---

# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

L’utilizzo dell’API è limitato a scopi di sviluppo. L’API viene fornita come contesto per lo sviluppo di applicazioni personalizzate. [!DNL Adobe Experience Manager] come [!DNL Cloud Service] utilizza l’API per passare le informazioni di elaborazione a un’applicazione personalizzata. Per ulteriori informazioni, consulta [Utilizzare i microservizi delle risorse e i profili di elaborazione](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo con [!DNL Experience Manager] come [!DNL Cloud Service].

Qualsiasi cliente del [!DNL Asset Compute Service] L’API HTTP deve seguire questo flusso di alto livello:

1. Il provisioning di un client viene eseguito come [!DNL Adobe Developer Console] in un’organizzazione IMS. Ogni client separato (sistema o ambiente) richiede un proprio progetto separato per separare il flusso di dati dell’evento.

1. Un client genera un token di accesso per l’account tecnico utilizzando [Autenticazione JWT (account di servizio)](https://www.adobe.io/authentication/auth-methods.html).

1. Chiamate client [`/register`](#register) solo una volta per recuperare l&#39;URL del journal.

1. Chiamate client [`/process`](#process-request) per ogni risorsa per la quale desidera generare rappresentazioni. La chiamata è asincrona.

1. Un cliente esegue regolarmente il polling del giornale di registrazione su [eventi di ricezione](#asynchronous-events). Riceve eventi per ogni rendering richiesto quando il rendering viene elaborato correttamente (`rendition_created` tipo di evento) o se si verifica un errore (`rendition_failed` tipo di evento).

La [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) Il modulo semplifica l’utilizzo dell’API nel codice Node.js.

## Autenticazione e autorizzazione {#authentication-and-authorization}

Tutte le API richiedono l’autenticazione dei token di accesso. Le richieste devono impostare le seguenti intestazioni:

1. `Authorization` intestazione con token portatore, che è il token dell’account tecnico, ricevuto tramite [Scambio JWT](https://www.adobe.io/authentication/auth-methods.html) dal progetto Adobe Developer Console . La [ambiti](#scopes) sono documentati di seguito.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` con l’ID organizzazione IMS.

1. `x-api-key` con l&#39;ID client dal [!DNL Adobe Developers Console] progetto.

### Ambiti {#scopes}

Assicurati i seguenti ambiti per il token di accesso:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Questi richiedono [!DNL Adobe Developer Console] progetto a cui si desidera effettuare l’abbonamento `Asset Compute`, `I/O Events`e `I/O Management API` servizi. La ripartizione dei singoli ambiti è la seguente:

* Base
   * ambiti: `openid,AdobeID`

* asset compute
   * metascopio: `asset_compute_meta`
   * ambiti: `asset_compute,read_organizations`

* [!DNL Adobe I/O] Eventi
   * metascopio: `event_receiver_api`
   * ambiti: `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] API di gestione
   * metascopio: `ent_adobeio_sdk`
   * ambiti: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registrazione {#register}

Ogni cliente del [!DNL Asset Compute service] - un unico [!DNL Adobe Developer Console] progetto abbonato al servizio - deve [registrare](#register-request) prima di effettuare richieste di elaborazione. Il passaggio di registrazione restituisce il giornale di registrazione eventi univoco necessario per recuperare gli eventi asincroni dall’elaborazione del rendering.

Al termine del suo ciclo di vita, un cliente può [annullare la registrazione](#unregister-request).

### Richiesta di registrazione {#register-request}

Questa chiamata API imposta un [!DNL Asset Compute] e fornisce l&#39;URL del giornale di registrazione eventi. Si tratta di un’operazione ideale e deve essere richiamata una sola volta per ogni client. Può essere richiamato nuovamente per recuperare l&#39;URL del journal.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/register` |
| Intestazione `Authorization` | Tutto [intestazioni relative all&#39;autorizzazione](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione su più sistemi. |
| Corpo della richiesta | Deve essere vuoto. |

### Registra risposta {#register-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale al `X-Request-Id` intestazione di richiesta o una generata in modo univoco. Da utilizzare per identificare le richieste tra sistemi e/o richieste di supporto. |
| Corpo di risposta | Un oggetto JSON con `journal`, `ok` e/o `requestId` campi. |

I codici di stato HTTP sono:

* **Successo 200**: Quando la richiesta ha esito positivo. Contiene il `journal` URL a cui viene inviata una notifica relativa ai risultati dell’elaborazione asincrona attivata tramite `/process` (come tipo di evento `rendition_created` in caso di esito positivo, oppure `rendition_failed` in caso di errore).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 Non autorizzato**: si verifica quando la richiesta non è valida [autenticazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 Vietato**: si verifica quando la richiesta non è valida [autorizzazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è iscritto a tutti i servizi richiesti.

* **429 Troppe richieste**: si verifica quando il sistema è sovraccaricato da questo client o in altro modo. I clienti devono riprovare con un [retromarcia esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **Errore 4xx**: Quando si è verificato un altro errore del client e la registrazione non è riuscita. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Errore 5xx**: si verifica quando si è verificato un altro errore lato server e la registrazione non è riuscita. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### Annulla registrazione richiesta {#unregister-request}

Questa chiamata API annulla la registrazione di un [!DNL Asset Compute] client. Dopo questo non è più possibile chiamare `/process`. Se si utilizza la chiamata API per un client non registrato o per un client non ancora registrato, viene restituito un `404` errore.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/unregister` |
| Intestazione `Authorization` | Tutto [intestazioni relative all&#39;autorizzazione](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione su più sistemi. |
| Corpo della richiesta | Vuoto. |

### Annulla registrazione risposta {#unregister-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale al `X-Request-Id` intestazione di richiesta o una generata in modo univoco. Da utilizzare per identificare le richieste tra sistemi e/o richieste di supporto. |
| Corpo di risposta | Un oggetto JSON con `ok` e `requestId` campi. |

I codici di stato sono:

* **Successo 200**: si verifica quando la registrazione e il giornale di registrazione vengono trovati e rimossi.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 Non autorizzato**: si verifica quando la richiesta non è valida [autenticazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 Vietato**: si verifica quando la richiesta non è valida [autorizzazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è iscritto a tutti i servizi richiesti.

* **404 Non trovato**: si verifica quando non è presente alcuna registrazione corrente per le credenziali specificate.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 Troppe richieste**: si verifica quando il sistema è sovraccarico. I clienti devono riprovare con un [retromarcia esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.

* **Errore 4xx**: si verifica quando si è verificato un altro errore del client e l&#39;annullamento della registrazione non è riuscito. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Errore 5xx**: si verifica quando si è verificato un altro errore lato server e la registrazione non è riuscita. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## Richiesta di processo {#process-request}

La `process` invia un processo che trasforma una risorsa sorgente in più rappresentazioni, in base alle istruzioni contenute nella richiesta. Notifiche relative al completamento riuscito (tipo di evento) `rendition_created`) o qualsiasi errore (tipo di evento `rendition_failed`) vengono inviate a un giornale di registrazione eventi che deve essere recuperato utilizzando [/register](#register) prima di effettuare un numero qualsiasi di `/process` richieste. Le richieste formate in modo errato non riescono immediatamente con un codice di errore 400.

I binari sono referenziati utilizzando URL, come URL prefirmati di Amazon AWS S3 o URL SAS di archiviazione BLOB di Azure, sia per la lettura della `source` asset (`GET` URL) e scrittura delle rappresentazioni (`PUT` URL). Il cliente è responsabile della generazione di questi URL pre-firmati.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/process` |
| Tipo MIME | `application/json` |
| Intestazione `Authorization` | Tutto [intestazioni relative all&#39;autorizzazione](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione su più sistemi. |
| Corpo della richiesta | Deve essere nel formato JSON della richiesta del processo come descritto di seguito. Fornisce istruzioni su quale risorsa elaborare e quali rappresentazioni generare. |

### JSON richiesta di processo {#process-request-json}

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
| `source` | `string` | URL della risorsa sorgente da elaborare. Facoltativo basato sul formato di rendering richiesto (ad esempio `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descrizione della risorsa di origine da elaborare. Vedi descrizione [Campi oggetto di origine](#source-object-fields) sotto. Facoltativo basato sul formato di rendering richiesto (ad esempio `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Rappresentazioni da generare dal file di origine. Ogni oggetto di rendering supporta [istruzione di rendering](#rendition-instructions). Obbligatorio. | `[{ "target": "https://....", "fmt": "png" }]` |

La `source` può essere `<string>` visualizzato come URL o può essere un `<object>` con un campo aggiuntivo. Le seguenti varianti sono simili:

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
| `url` | `string` | URL della risorsa sorgente da elaborare. Obbligatorio. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nome file risorsa di origine. L’estensione del file nel nome può essere utilizzata se non è possibile rilevare alcun tipo MIME. Ha la precedenza sul nome del file nel percorso URL o nel nome del file in `content-disposition` intestazione della risorsa binaria. Predefinito in &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Dimensione del file della risorsa di origine in byte. Ha la precedenza `content-length` intestazione della risorsa binaria. | `10234` |
| `mimetype` | `string` | Tipo MIME del file della risorsa di origine. Ha la precedenza sul `content-type` intestazione della risorsa binaria. | `"image/jpeg"` |

### Completo `process` esempio di richiesta {#complete-process-request-example}

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

## Risposta del processo {#process-response}

La `/process` La richiesta restituisce immediatamente con un esito positivo o negativo in base alla convalida della richiesta di base. L’elaborazione effettiva delle risorse avviene in modo asincrono.

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale al `X-Request-Id` intestazione di richiesta o una generata in modo univoco. Da utilizzare per identificare le richieste tra sistemi e/o richieste di supporto. |
| Corpo di risposta | Un oggetto JSON con `ok` e `requestId` campi. |

Codici di stato:

* **Successo 200**: Se la richiesta è stata inviata correttamente. La risposta JSON include `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Richiesta non valida**: Se la richiesta non è formata correttamente, ad esempio campi obbligatori mancanti nella richiesta JSON. La risposta JSON include `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 Non autorizzato**: Se la richiesta non è valida [autenticazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.
* **403 Vietato**: Se la richiesta non è valida [autorizzazione](#authentication-and-authorization). Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è iscritto a tutti i servizi richiesti.
* **429 Troppe richieste**: Quando il sistema è sovraccaricato da questo client o in generale. I client possono riprovare con un [retromarcia esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **Errore 4xx**: Quando si è verificato un altro errore del client. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Errore 5xx**: Quando si è verificato un altro errore lato server. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

È probabile che la maggior parte dei client siano inclini a ripetere la stessa richiesta con [retromarcia esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff) su qualsiasi errore *eccetto* problemi di configurazione come 401 o 403 o richieste non valide come 400. Oltre a limitare regolarmente le tariffe tramite risposte 429, un&#39;interruzione o una limitazione del servizio temporaneo potrebbe causare errori 5xx. Sarebbe quindi consigliabile riprovare dopo un periodo di tempo.

Tutte le risposte JSON (se presenti) includono la `requestId` che è lo stesso valore del `X-Request-Id` intestazione. Si consiglia di leggere dall’intestazione, in quanto è sempre presente. La `requestId` viene restituito anche in tutti gli eventi relativi all’elaborazione delle richieste come `requestId`. I client non devono presupporre il formato di questa stringa, ma si tratta di un identificatore di stringa opaco.

## Opt-in alla post-elaborazione {#opt-in-to-post-processing}

La [asset compute SDK](https://github.com/adobe/asset-compute-sdk) supporta un set di opzioni di base per la post-elaborazione delle immagini. I processi di lavoro personalizzati possono esplicitamente accettare la post-elaborazione impostando il campo `postProcess` sull&#39;oggetto di rendering in `true`.

I casi di utilizzo supportati sono:

* Ritaglia un rendering in un rettangolo i cui limiti sono definiti da crop.w, crop.h, crop.x e crop.y. È definito da `instructions.crop` nell&#39;oggetto di rendering.
* Ridimensiona le immagini utilizzando larghezza, altezza o entrambi. È definito da `instructions.width` e `instructions.height` nell&#39;oggetto di rendering. Per ridimensionare utilizzando solo larghezza o altezza, impostare un solo valore. Compute Service conserva le proporzioni.
* Impostare la qualità di un&#39;immagine JPEG. È definito da `instructions.quality` nell&#39;oggetto di rendering. La migliore qualità è indicata da `100` e valori più piccoli indicano una qualità ridotta.
* Creare immagini interlacciate. È definito da `instructions.interlace` nell&#39;oggetto di rendering.
* Imposta DPI per regolare le dimensioni di cui è stato eseguito il rendering a scopo di pubblicazione desktop regolando la scala applicata ai pixel. È definito da `instructions.dpi` nell&#39;oggetto di rendering per modificare la risoluzione dpi. Tuttavia, per ridimensionare l’immagine in modo che abbia le stesse dimensioni a una risoluzione diversa, utilizza la `convertToDpi` istruzioni.
* Ridimensiona l’immagine in modo che la sua larghezza o altezza di cui è stato eseguito il rendering resti uguale all’originale alla risoluzione di destinazione specificata (DPI). È definito da `instructions.convertToDpi` nell&#39;oggetto di rendering.

## Risorse filigrana {#add-watermark}

La [asset compute SDK](https://github.com/adobe/asset-compute-sdk) supporta l’aggiunta di una filigrana ai file immagine PNG, JPEG, TIFF e GIF. La filigrana viene aggiunta seguendo le istruzioni di rendering in `watermark` sul rendering.

La filigrana viene eseguita durante il rendering post-elaborazione. Per applicare una filigrana alle risorse, eseguire il processo di lavoro personalizzato [effettua l’opt-in dopo-elaborazione](#opt-in-to-post-processing) impostando il campo `postProcess` sull&#39;oggetto di rendering in `true`. Se il processo di lavoro non effettua l’opt-in, la filigrana non viene applicata, anche se l’oggetto filigrana è impostato sull’oggetto di rendering nella richiesta.

## Istruzioni per la rappresentazione {#rendition-instructions}

Sono le opzioni disponibili per la `renditions` in matrice [/process](#process-request).

### Campi comuni {#common-fields}

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Il formato di destinazione delle rappresentazioni può anche essere `text` per l’estrazione del testo e `xmp` estrazione dei metadati XMP come xml. Vedi [formati supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL di un [applicazione personalizzata](develop-custom-application.md). Deve essere `https://` URL. Se questo campo è presente, il rendering viene creato da un&#39;applicazione personalizzata. Qualsiasi altro campo di rendering impostato viene quindi utilizzato nell&#39;applicazione personalizzata. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL a cui il rendering generato deve essere caricato utilizzando HTTP PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Informazioni sul caricamento di URL pre-firmati multiparte per il rendering generato. Questo è per [Caricamento binario diretto AEM/Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con questo [comportamento di caricamento multiparte](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>espandibili:<ul><li>`urls`: array di stringhe, una per ciascun URL di parte pre-firmato</li><li>`minPartSize`: la dimensione minima da utilizzare per una parte = url</li><li>`maxPartSize`: la dimensione massima da utilizzare per una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Spazio riservato facoltativo controllato dal client e trasmesso come agli eventi di rendering. Consente ai client di aggiungere informazioni personalizzate per identificare gli eventi di rendering. Non è necessario modificarlo o basarsi su in applicazioni personalizzate, in quanto i client possono modificarlo in qualsiasi momento. | `{ ... }` |

### Campi specifici della rappresentazione {#rendition-specific-fields}

Per un elenco dei formati di file attualmente supportati, consulta [formati di file supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `*` | `*` | È possibile aggiungere campi personalizzati avanzati che consentono di [applicazione personalizzata](develop-custom-application.md) comprende. |  |
| `embedBinaryLimit` | `number` in byte | Se questo valore è impostato e la dimensione del file del rendering è inferiore a questo valore, il rendering viene incorporato nell&#39;evento inviato al termine della generazione del rendering. La dimensione massima consentita per l&#39;incorporazione è di 32 KB (32 x 1024 byte). Se un rendering ha dimensioni maggiori rispetto al `embedBinaryLimit` limite, viene inserito in una posizione nell&#39;archiviazione cloud e non è incorporato nell&#39;evento. | `3276` |
| `width` | `number` | Larghezza in pixel. solo per rappresentazioni di immagini. | `200` |
| `height` | `number` | Altezza in pixel. solo per rappresentazioni di immagini. | `200` |
|  |  | Il rapporto di formato viene sempre mantenuto se: <ul> <li> Entrambi `width` e `height` vengono specificati, quindi l&#39;immagine si adatta alle dimensioni mantenendo le proporzioni </li><li> Solo `width` o solo `height` viene specificata, l&#39;immagine risultante utilizza la dimensione corrispondente mantenendo le proporzioni</li><li> Se non `width` né `height` viene specificata la dimensione del pixel dell&#39;immagine originale. Dipende dal tipo di origine. Per alcuni formati, ad esempio i file PDF, viene utilizzata una dimensione predefinita. Può esserci un limite massimo di dimensioni.</li></ul> |  |
| `quality` | `number` | Specifica la qualità jpeg nell&#39;intervallo di `1` a `100`. Applicabile solo alle rappresentazioni di immagini. | `90` |
| `xmp` | `string` | Utilizzato solo da XMP write-back di metadati, è XMP codificato base64 per riscrivere al rendering specificato. |  |
| `interlace` | `bool` | Crea PNG interlacciato, GIF o JPEG progressivo impostandolo su `true`. Non ha alcun effetto su altri formati di file. |  |
| `jpegSize` | `number` | Dimensione approssimativa del file JPEG in byte. Ignora qualsiasi `quality` impostazione. Non ha alcun effetto su altri formati. |  |
| `dpi` | `number` oppure `object` | Impostare x e y DPI. Per semplicità, può anche essere impostato su un singolo numero che viene utilizzato sia per x che per y. Non ha alcun effetto sull&#39;immagine stessa. | `96` oppure `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` oppure `object` | i valori di ricampionamento di x e y DPI mantenendo le dimensioni fisiche. Per semplicità, può anche essere impostato su un singolo numero che viene utilizzato sia per x che per y. | `96` oppure `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Elenco di file da includere nell&#39;archivio ZIP (`fmt=zip`). Ogni voce può essere una stringa URL o un oggetto con i campi:<ul><li>`url`: URL per il download del file</li><li>`path`: Archivia file sotto questo percorso nel file ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Gestione dei duplicati per gli archivi ZIP (`fmt=zip`). Per impostazione predefinita, più file memorizzati sotto lo stesso percorso nello ZIP generano un errore. Impostazione `duplicate` a `ignore` determina solo la prima risorsa da memorizzare e il resto da ignorare. | `ignore` |
| `watermark` | `object` | Contiene istruzioni sul [filigrana](#watermark-specific-fields). |  |

### Campi specifici della filigrana {#watermark-specific-fields}

Il formato PNG viene utilizzato come filigrana.

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Scala della filigrana, tra `0.0` e `1.0`. `1.0` significa che la filigrana ha la sua scala originale (1:1) e che i valori inferiori riducono la dimensione della filigrana. | Un valore di `0.5` metà delle dimensioni originali. |
| `image` | `url` | URL del file PNG da utilizzare per la filigrana. |  |

## Eventi asincroni {#asynchronous-events}

Al termine dell&#39;elaborazione di un rendering o al verificarsi di un errore, viene inviato un evento a un [[!DNL Adobe I/O] Journal eventi](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). I clienti devono ascoltare l&#39;URL del journal fornito tramite [/register](#register). La risposta del journal include un `event` array costituito da un oggetto per ogni evento, di cui la `event` Il campo include il payload dell’evento effettivo.

La [!DNL Adobe I/O] Tipo evento per tutti gli eventi del [!DNL Asset Compute Service] è `asset_compute`. Il giornale di registrazione viene automaticamente sottoscritto solo a questo tipo di evento e non è necessario filtrare ulteriormente in base al [!DNL Adobe I/O] Tipo di evento. I tipi di evento specifici del servizio sono disponibili nella variabile `type` proprietà dell&#39;evento.

### Tipi di eventi {#event-types}

| Evento | Descrizione |
|---------------------|-------------|
| `rendition_created` | Inviato per ogni rendering elaborato e caricato correttamente. |
| `rendition_failed` | Inviato per ogni rendering che non è stato in grado di elaborare o caricare. |

### Attributi evento {#event-attributes}

| Attributo | Tipo | Evento | Descrizione |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Timestamp dell’invio dell’evento in versione semplificata estesa [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) , come definito da JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | ID richiesta della richiesta originale a `/process`, come `X-Request-Id` intestazione. |
| `source` | `object` | `*` | La `source` del `/process` richiesta. |
| `userData` | `object` | `*` | La `userData` del rendering dal `/process` se impostato. |
| `rendition` | `object` | `rendition_*` | L&#39;oggetto di rendering corrispondente passato `/process`. |
| `metadata` | `object` | `rendition_created` | La [metadati](#metadata) proprietà del rendering. |
| `errorReason` | `string` | `rendition_failed` | Errore di rendering [motivo](#error-reasons) se del caso. |
| `errorMessage` | `string` | `rendition_failed` | Testo che fornisce ulteriori dettagli sull’eventuale errore di rendering. |

### Metadati {#metadata}

| Proprietà | Descrizione |
|--------|-------------|
| `repo:size` | Dimensione del rendering in byte. |
| `repo:sha1` | Il riassunto sha1 del rendering. |
| `dc:format` | Il tipo MIME del rendering. |
| `repo:encoding` | Codifica charset del rendering nel caso in cui si tratti di un formato basato su testo. |
| `tiff:ImageWidth` | Larghezza del rendering in pixel. Presente solo per rappresentazioni di immagini. |
| `tiff:ImageLength` | Lunghezza del rendering in pixel. Presente solo per rappresentazioni di immagini. |

### Motivi dell&#39;errore {#error-reasons}

| Motivo | Descrizione |
|---------|-------------|
| `RenditionFormatUnsupported` | Il formato di rendering richiesto non è supportato per l&#39;origine specificata. |
| `SourceUnsupported` | L&#39;origine specifica non è supportata anche se il tipo è supportato. |
| `SourceCorrupt` | I dati di origine sono danneggiati. Include file vuoti. |
| `RenditionTooLarge` | Impossibile caricare il rendering utilizzando gli URL pre-firmati forniti in `target`. La dimensione effettiva del rendering è disponibile come metadati in `repo:size` e può essere utilizzato dal client per rielaborare il rendering con il numero corretto di URL pre-firmati. |
| `GenericError` | Qualsiasi altro errore imprevisto. |
