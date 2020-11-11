---
title: '[!DNL Asset Compute Service] API HTTP.'
description: '[!DNL Asset Compute Service] API HTTP per creare applicazioni personalizzate.'
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '2925'
ht-degree: 2%

---


# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

L&#39;utilizzo dell&#39;API è limitato a scopi di sviluppo. L&#39;API viene fornita come contesto quando si sviluppano applicazioni personalizzate. [!DNL Adobe Experience Manager] come Cloud Service utilizza l&#39;API per trasmettere le informazioni di elaborazione a un&#39;applicazione personalizzata. Per ulteriori informazioni, consultate [Utilizzare i microservizi delle risorse e i profili](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)di elaborazione.

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l&#39;uso con [!DNL Experience Manager] come Cloud Service.

Qualsiasi client dell&#39;API [!DNL Asset Compute Service] HTTP deve seguire questo flusso di alto livello:

1. Il provisioning di un cliente come [!DNL Adobe Developer Console] progetto in un&#39;organizzazione IMS. Ciascun client (sistema o ambiente) separato richiede un proprio progetto separato per separare il flusso di dati dell&#39;evento.

1. Un client genera un token di accesso per l&#39;account tecnico utilizzando l&#39;autenticazione [](https://www.adobe.io/authentication/auth-methods.html)JWT (Service Account).

1. Un client chiama [`/register`](#register) solo una volta per recuperare l’URL del giornale di registrazione.

1. Un client chiama [`/process`](#process-request) per ogni risorsa per la quale desidera generare delle rappresentazioni. La chiamata è asincrona.

1. Un cliente controlla regolarmente il giornale di registrazione per [ricevere gli eventi](#asynchronous-events). Riceve eventi per ciascuna rappresentazione richiesta quando la rappresentazione viene elaborata correttamente (tipo di`rendition_created` evento) o se si verifica un errore (tipo di`rendition_failed` evento).

Il modulo [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) semplifica l&#39;utilizzo dell&#39;API nel codice Node.js.

## Autenticazione e autorizzazione {#authentication-and-authorization}

Tutte le API richiedono l&#39;autenticazione del token di accesso. Le richieste devono impostare le seguenti intestazioni:

1. `Authorization` intestazione con token portatore, che è il token tecnico dell’account, ricevuto tramite [JWT Exchange](https://www.adobe.io/authentication/auth-methods.html) dal progetto  Developer Console di Adobe. Gli [ambiti](#scopes) sono documentati di seguito.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
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

Ciò richiede la sottoscrizione del [!DNL Adobe Developer Console] progetto a `Asset Compute`, `I/O Events`e `I/O Management API` servizi. La suddivisione dei singoli ambiti è la seguente:

* Base
   * scopes: `openid,AdobeID`

* asset compute 
   * metascope: `asset_compute_meta`
   * scopes: `asset_compute,read_organizations`

*  eventi I/O Adobe
   * metascope: `event_receiver_api`
   * scopes: `event_receiver,event_receiver_api`

*  API di gestione I/O Adobe
   * metascope: `ent_adobeio_sdk`
   * scopes: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registrazione {#register}

Ogni client del [!DNL Asset Compute service] - un [!DNL Adobe Developer Console] progetto unico sottoscritto al servizio - deve [registrarsi](#register-request) prima di effettuare le richieste di elaborazione. Il passaggio di registrazione restituisce il giornale di registrazione eventi univoco richiesto per recuperare gli eventi asincroni dall&#39;elaborazione delle rappresentazioni.

Al termine del ciclo di vita, un client può [annullare la registrazione](#unregister-request).

### Richiesta di registrazione {#register-request}

Questa chiamata API imposta un [!DNL Asset Compute] client e fornisce l&#39;URL del giornale di registrazione eventi. Si tratta di un&#39;operazione molto efficace e deve essere chiamata una sola volta per ogni client. Può essere richiamata di nuovo per recuperare l&#39;URL del journal.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/register` |
| Intestazione `Authorization` | Tutte le intestazioni [relative all&#39;](#authentication-and-authorization)autorizzazione. |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione tra sistemi. |
| Corpo della richiesta | Deve essere vuoto. |

### Registra risposta {#register-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale all’intestazione della `X-Request-Id` richiesta o generata in modo univoco. Utilizzate per identificare le richieste tra sistemi e/o richieste di assistenza. |
| Corpo di risposta | Un oggetto JSON con `journal`, `ok` e/o `requestId` campi. |

I codici di stato HTTP sono:

* **200 Successo**: Quando la richiesta ha esito positivo. Contiene l&#39; `journal` URL a cui viene notificato qualsiasi risultato dell&#39;elaborazione asincrona attivata tramite `/process` (come tipo di evento `rendition_created` quando l&#39;esito è positivo o `rendition_failed` in caso di errore).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 Non autorizzato**: si verifica quando la richiesta non dispone di un&#39; [autenticazione](#authentication-and-authorization)valida. Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 Vietato**: si verifica quando la richiesta non dispone di un&#39; [autorizzazione](#authentication-and-authorization)valida. Un esempio potrebbe essere un token di accesso valido, ma il progetto  Developer Console (account tecnico) non è iscritto a tutti i servizi richiesti.

* **429 Troppe richieste**: si verifica quando il sistema viene sovraccaricato dal client o in altro modo. I client devono riprovare con un backoff [esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **Errore** 4xx: Quando si è verificato un altro errore del client e la registrazione non è riuscita. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx errore**: si verifica quando si è verificato un altro errore lato server e la registrazione non è riuscita. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### Annulla registrazione richiesta {#unregister-request}

Questa chiamata API annulla la registrazione di un [!DNL Asset Compute] client. Dopo questo non è più possibile chiamare `/process`. L&#39;utilizzo della chiamata API per un client non registrato o un client ancora da registrare restituisce un `404` errore.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/unregister` |
| Intestazione `Authorization` | Tutte le intestazioni [relative all&#39;](#authentication-and-authorization)autorizzazione. |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione tra sistemi. |
| Corpo della richiesta | Vuoto. |

### Annulla registrazione risposta {#unregister-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale all’intestazione della `X-Request-Id` richiesta o generata in modo univoco. Utilizzate per identificare le richieste tra sistemi e/o richieste di assistenza. |
| Corpo di risposta | Un oggetto JSON con `ok` e `requestId` campi. |

I codici di stato sono:

* **200 Successo**: si verifica quando la registrazione e il giornale di registrazione vengono trovati e rimossi.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 Non autorizzato**: si verifica quando la richiesta non dispone di un&#39; [autenticazione](#authentication-and-authorization)valida. Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 Vietato**: si verifica quando la richiesta non dispone di un&#39; [autorizzazione](#authentication-and-authorization)valida. Un esempio potrebbe essere un token di accesso valido, ma il progetto  Developer Console (account tecnico) non è iscritto a tutti i servizi richiesti.

* **404 Non trovato**: si verifica quando non esiste una registrazione corrente per le credenziali specificate.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 Troppe richieste**: si verifica quando il sistema è sovraccarico. I client devono riprovare con un backoff [esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.

* **Errore** 4xx: si verifica quando si è verificato un altro errore client e l&#39;annullamento della registrazione non è riuscita. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx errore**: si verifica quando si è verificato un altro errore lato server e la registrazione non è riuscita. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## Richiesta di elaborazione {#process-request}

L&#39; `process` operazione invia un processo che trasforma una risorsa di origine in più rappresentazioni, in base alle istruzioni fornite nella richiesta. Le notifiche relative al completamento corretto (tipo di evento `rendition_created`) o a eventuali errori (tipo di evento `rendition_failed`) vengono inviate a un giornale di registrazione eventi che deve essere recuperato una volta utilizzando [/registrate](#register) prima di eseguire un numero qualsiasi di `/process` richieste. Le richieste formate in modo errato non riescono immediatamente con un codice di errore 400.

Ai file binari viene fatto riferimento tramite URL, come  URL Amazon AWS S3 pre-firmati o URL SAS Azure Blob Storage, sia per la lettura della `source` risorsa (`GET` URL) che per la scrittura delle rappresentazioni (`PUT` URL). Il client è responsabile della generazione di questi URL pre-firmati.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/process` |
| Tipo MIME | `application/json` |
| Intestazione `Authorization` | Tutte le intestazioni [relative all&#39;](#authentication-and-authorization)autorizzazione. |
| Intestazione `x-request-id` | Facoltativo, può essere impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione tra sistemi. |
| Corpo della richiesta | Deve essere nel formato JSON della richiesta di processo come descritto di seguito. Fornisce istruzioni su quale risorsa elaborare e quali rappresentazioni generare. |

### JSON richiesta di elaborazione {#process-request-json}

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
| `source` | `string` | URL della risorsa di origine da elaborare. Facoltativo basato sul formato di rappresentazione richiesto (ad es. `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descrizione della risorsa di origine da elaborare. Vedere la descrizione dei campi [oggetto](#source-object-fields) Source di seguito. Facoltativo basato sul formato di rappresentazione richiesto (ad es. `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Rappresentazioni da generare dal file di origine. Ciascun oggetto di rappresentazione supporta le istruzioni [di](#rendition-instructions)rappresentazione. Obbligatorio. | `[{ "target": "https://....", "fmt": "png" }]` |

Può `source` essere un URL `<string>` che viene visto come URL oppure può essere un campo `<object>` con un altro. Le seguenti varianti sono simili:

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
| `name` | `string` | Nome del file della risorsa di origine. L&#39;estensione del file nel nome potrebbe essere utilizzata se non è possibile rilevare alcun tipo MIME. Ha precedenza rispetto al nome del file nel percorso URL o nel nome del file nell&#39; `content-disposition` intestazione della risorsa binaria. Il valore predefinito è &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Dimensione del file della risorsa di origine in byte. Ha precedenza rispetto all&#39; `content-length` intestazione della risorsa binaria. | `10234` |
| `mimetype` | `string` | Tipo MIME del file della risorsa di origine. Ha la precedenza sull&#39; `content-type` intestazione della risorsa binaria. | `"image/jpeg"` |

### Esempio di `process` richiesta completa {#complete-process-request-example}

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

## Risposta al processo {#process-response}

La `/process` richiesta restituisce immediatamente un esito positivo o negativo in base alla convalida della richiesta di base. L&#39;elaborazione effettiva delle risorse avviene in modo asincrono.

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale all’intestazione della `X-Request-Id` richiesta o generata in modo univoco. Utilizzate per identificare le richieste tra sistemi e/o richieste di assistenza. |
| Corpo di risposta | Un oggetto JSON con `ok` e `requestId` campi. |

Codici di stato:

* **200 Successo**: Se la richiesta è stata inviata correttamente. Il JSON di risposta include `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Richiesta** non valida: Se la richiesta non è formata correttamente, ad esempio i campi obbligatori mancanti nel JSON della richiesta. Il JSON di risposta include `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 Non autorizzato**: Se la richiesta non dispone di un&#39; [autenticazione](#authentication-and-authorization)valida. Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.
* **403 Vietato**: Se la richiesta non dispone di un&#39; [autorizzazione](#authentication-and-authorization)valida. Un esempio potrebbe essere un token di accesso valido, ma il progetto  Developer Console (account tecnico) non è iscritto a tutti i servizi richiesti.
* **429 Troppe richieste**: Quando il sistema è sovraccarico da questo client o in generale. I client possono riprovare con un backoff [esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **Errore** 4xx: In caso di altri errori del client. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx errore**: Quando si è verificato un altro errore lato server. Solitamente viene restituita una risposta JSON come questa, anche se non è garantita per tutti gli errori:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

È probabile che la maggior parte dei client siano inclini a ripetere la stessa richiesta con il backoff [](https://en.wikipedia.org/wiki/Exponential_backoff) esponenziale su qualsiasi errore, *ad eccezione* di problemi di configurazione come 401 o 403, o richieste non valide come 400. Oltre alla limitazione regolare della tariffa tramite 429 risposte, un&#39;interruzione o una limitazione temporanea del servizio potrebbe causare errori 5xx. È quindi consigliabile riprovare dopo un periodo di tempo.

Tutte le risposte JSON (se presenti) includono il valore `requestId` che corrisponde all’ `X-Request-Id` intestazione. Si consiglia di leggere dall’intestazione, in quanto è sempre presente. L&#39;evento `requestId` viene restituito anche in tutti gli eventi correlati all&#39;elaborazione delle richieste come `requestId`. I client non devono prendere in considerazione il formato di questa stringa, ma un identificatore di stringa opaco.

## Consenso alla post-elaborazione {#opt-in-to-post-processing}

L’SDK [per Asset compute](https://github.com/adobe/asset-compute-sdk) supporta una serie di opzioni di base per la post-elaborazione delle immagini. I lavoratori personalizzati possono scegliere esplicitamente di postelaborazione impostando il campo `postProcess` sull&#39;oggetto di rappresentazione su `true`.

I casi di utilizzo supportati sono:

* Ritagliare una rappresentazione in un rettangolo i cui limiti sono definiti da ritaglio.w, ritaglio.h, ritaglio.x e ritaglio.y. È definito dall&#39;oggetto `instructions.crop` di rappresentazione.
* Ridimensionate le immagini utilizzando larghezza, altezza o entrambi. È definito da `instructions.width` e `instructions.height` nell&#39;oggetto di rappresentazione. Per ridimensionare i dati utilizzando solo larghezza o altezza, impostate un solo valore. Il servizio di elaborazione conserva le proporzioni.
* Impostate la qualità per un’immagine JPEG. È definito dall&#39;oggetto `instructions.quality` di rappresentazione. La qualità migliore è indicata da `100` valori più piccoli che indicano una qualità ridotta.
* Creare immagini interlacciate. È definito dall&#39;oggetto `instructions.interlace` di rappresentazione.
* Impostate DPI per regolare le dimensioni di cui è stato effettuato il rendering per la pubblicazione desktop regolando la scala applicata ai pixel. È definito dall&#39;oggetto `instructions.dpi` di rappresentazione per modificare la risoluzione dpi. Tuttavia, per ridimensionare l’immagine in modo che abbia le stesse dimensioni a una risoluzione diversa, utilizzate le `convertToDpi` istruzioni.
* Ridimensionate l’immagine in modo che la larghezza o l’altezza di cui è stato effettuato il rendering resti uguale all’originale alla risoluzione di destinazione specificata (DPI). È definito dall&#39;oggetto `instructions.convertToDpi` di rappresentazione.

## Risorse di filigrana {#add-watermark}

L’SDK [per](https://github.com/adobe/asset-compute-sdk) Asset compute supporta l’aggiunta di una filigrana ai file di immagine PNG, JPEG, TIFF e GIF. La filigrana viene aggiunta seguendo le istruzioni di rappresentazione nell&#39; `watermark` oggetto sulla rappresentazione.

La filigrana viene applicata durante la fase di post-elaborazione del rendering. Per applicare una filigrana alle risorse, il lavoratore personalizzato [sceglie di eseguire la post-elaborazione](#opt-in-to-post-processing) impostando il campo `postProcess` sull&#39;oggetto di rappresentazione su `true`. Se il lavoratore non opta, la filigrana non viene applicata, anche se l&#39;oggetto filigrana è impostato sull&#39;oggetto di rappresentazione nella richiesta.

## Istruzioni per il rendering {#rendition-instructions}

Queste sono le opzioni disponibili per l&#39; `renditions` array in [/process](#process-request).

### Campi comuni {#common-fields}

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Il formato di destinazione delle rappresentazioni può essere anche `text` per l’estrazione del testo e `xmp` per l’estrazione dei metadati XMP come xml. Vedere [Formati supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL di un&#39;applicazione [](develop-custom-application.md)personalizzata. Deve essere un `https://` URL. Se questo campo è presente, la rappresentazione viene creata da un&#39;applicazione personalizzata. Qualsiasi altro campo di rappresentazione impostato viene quindi utilizzato nell&#39;applicazione personalizzata. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL al quale la rappresentazione generata deve essere caricata tramite PUT HTTP. | `http://w.com/img.jpg` |
| `target` | `object` | Informazioni sul caricamento di più URL pre-firmati per la rappresentazione generata. Questo è per il caricamento binario diretto [AEM/Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con questo comportamento [di caricamento](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)multiparte.<br>espandibili:<ul><li>`urls`: array di stringhe, uno per ciascun URL parte pre-firmato</li><li>`minPartSize`: la dimensione minima da utilizzare per una parte = url</li><li>`maxPartSize`: la dimensione massima da utilizzare per una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Spazio riservato facoltativo controllato dal client e trasmesso come per gli eventi di rappresentazione. Consente ai client di aggiungere informazioni personalizzate per identificare gli eventi di rappresentazione. Non deve essere modificato o utilizzato nelle applicazioni personalizzate, in quanto i client possono modificarlo in qualsiasi momento. | `{ ... }` |

### Campi specifici per la rappresentazione {#rendition-specific-fields}

Per un elenco dei formati di file attualmente supportati, vedere Formati [di file](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)supportati.

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `*` | `*` | È possibile aggiungere campi avanzati e personalizzati comprensibili a un&#39;applicazione [](develop-custom-application.md) personalizzata. |  |
| `embedBinaryLimit` | `number` in byte | Se questo valore è impostato e la dimensione del file della rappresentazione è inferiore a tale valore, la rappresentazione viene incorporata nell&#39;evento inviato al termine della generazione della rappresentazione. La dimensione massima consentita per l&#39;incorporamento è 32 KB (32 x 1024 byte). Se una rappresentazione ha dimensioni maggiori del `embedBinaryLimit` limite, viene inserita in una posizione nell&#39;archiviazione cloud e non viene incorporata nell&#39;evento. | `3276` |
| `width` | `number` | Larghezza in pixel. solo per rappresentazioni di immagini. | `200` |
| `height` | `number` | Altezza in pixel. solo per rappresentazioni di immagini. | `200` |
|  |  | Le proporzioni vengono sempre mantenute se: <ul> <li> Entrambi `width` e `height` sono specificati, quindi l&#39;immagine si adatta alle dimensioni mantenendo le proporzioni </li><li> Solo `width` o `height` solo se specificato, l’immagine risultante utilizza la dimensione corrispondente mantenendo le proporzioni</li><li> Se non `width` viene specificato né `height` viene specificata, viene utilizzata la dimensione in pixel dell’immagine originale. Dipende dal tipo di origine. Per alcuni formati, ad esempio i file PDF, viene utilizzata una dimensione predefinita. Può essere presente un limite massimo di dimensioni.</li></ul> |  |
| `quality` | `number` | Specificate la qualità jpeg nell’intervallo tra `1` a `100`. Applicabile solo alle rappresentazioni delle immagini. | `90` |
| `xmp` | `string` | Utilizzata solo XMP reimpostazione dei metadati, XMP codificati di base64 per la riscrittura alla rappresentazione specificata. |  |
| `interlace` | `bool` | Per creare file PNG o GIF interlacciati o JPEG progressivi, impostateli su `true`. Non ha alcun effetto sugli altri formati di file. |  |
| `jpegSize` | `number` | Dimensione approssimativa del file JPEG in byte. Ignora qualsiasi `quality` impostazione. Non ha alcun effetto su altri formati. |  |
| `dpi` | `number` o `object` | Impostare i valori DPI x e y. Per semplicità, può anche essere impostato su un singolo numero utilizzato sia per x che per y. Non ha alcun effetto sull&#39;immagine stessa. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` o `object` | i valori di x e y DPI vengono ricampionati mantenendo le dimensioni fisiche. Per semplicità, può anche essere impostato su un singolo numero utilizzato sia per x che per y. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Elenco di file da includere nell&#39;archivio ZIP (`fmt=zip`). Ogni voce può essere una stringa URL o un oggetto con i campi:<ul><li>`url`: URL per il download del file</li><li>`path`: Memorizza file in questo percorso nella ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Gestione duplicata per archivi ZIP (`fmt=zip`). Per impostazione predefinita, più file memorizzati nello stesso percorso nello ZIP generano un errore. L&#39;impostazione `duplicate` su `ignore` restituisce solo la prima risorsa da memorizzare e gli altri da ignorare. | `ignore` |
| `watermark` | `object` | Contiene istruzioni sulla [filigrana](#watermark-specific-fields). |  |

### Campi specifici per le filigrane {#watermark-specific-fields}

Il formato PNG viene utilizzato come filigrana.

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Scala della filigrana, tra `0.0` e `1.0`. `1.0` indica che la filigrana ha la scala originale (1:1) e che i valori inferiori riducono le dimensioni della filigrana. | Un valore pari a `0.5` metà della dimensione originale. |
| `image` | `url` | URL del file PNG da usare per la filigrana. |  |

## Eventi asincroni {#asynchronous-events}

Al termine dell&#39;elaborazione di una rappresentazione o al verificarsi di un errore, un evento viene inviato a un [Adobe I/O Events Journal](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). I client devono ascoltare l&#39;URL del journal fornito tramite [/register](#register). La risposta del giornale di registrazione include un `event` array composto da un oggetto per ogni evento, del quale il `event` campo include il payload dell&#39;evento effettivo.

Il tipo di evento di I/O di  Adobe per tutti gli eventi del [!DNL Asset Compute Service] è `asset_compute`. Il giornale di registrazione viene sottoscritto automaticamente solo per questo tipo di evento e non è più necessario filtrare in base al tipo di evento I/O del Adobe . I tipi di evento specifici del servizio sono disponibili nella `type` proprietà dell&#39;evento.

### Event types {#event-types}

| Evento | Descrizione |
|---------------------|-------------|
| `rendition_created` | Inviato per ciascuna rappresentazione elaborata e caricata correttamente. |
| `rendition_failed` | Inviato per ogni rappresentazione che non è stata elaborata o caricata. |

### Attributi evento {#event-attributes}

| Attributo | Tipo | Evento | Descrizione |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Timestamp dell&#39;invio dell&#39;evento in formato semplificato esteso [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) , come definito da JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | L’ID della richiesta originale a, `/process`come `X-Request-Id` intestazione. |
| `source` | `object` | `*` | La `source` `/process` richiesta. |
| `userData` | `object` | `*` | Il rendering `userData` `/process` della richiesta, se impostato. |
| `rendition` | `object` | `rendition_*` | L&#39;oggetto di rappresentazione corrispondente passato `/process`. |
| `metadata` | `object` | `rendition_created` | Proprietà [dei metadati](#metadata) della rappresentazione. |
| `errorReason` | `string` | `rendition_failed` | Eventuale [motivo](#error-reasons) di errore della rappresentazione. |
| `errorMessage` | `string` | `rendition_failed` | Testo che fornisce maggiori dettagli sull’eventuale errore di rappresentazione. |

### Metadati {#metadata}

| Proprietà | Descrizione |
|--------|-------------|
| `repo:size` | La dimensione del rendering in byte. |
| `repo:sha1` | Il riassunto sha1 della rappresentazione. |
| `dc:format` | Il tipo MIME della rappresentazione. |
| `repo:encoding` | La codifica charset della rappresentazione, se è un formato basato su testo. |
| `tiff:ImageWidth` | Larghezza della rappresentazione in pixel. Presente solo per rappresentazioni di immagini. |
| `tiff:ImageLength` | La lunghezza della rappresentazione, in pixel. Presente solo per rappresentazioni di immagini. |

### Motivi errore {#error-reasons}

| Motivo | Descrizione |
|---------|-------------|
| `RenditionFormatUnsupported` | Il formato di rappresentazione richiesto non è supportato per l&#39;origine specificata. |
| `SourceUnsupported` | L&#39;origine specifica non è supportata anche se il tipo è supportato. |
| `SourceCorrupt` | I dati di origine sono danneggiati. Include file vuoti. |
| `RenditionTooLarge` | Impossibile caricare il rendering con gli URL pre-firmati forniti in `target`. La dimensione effettiva della rappresentazione è disponibile come metadati in `repo:size` e può essere utilizzata dal client per rielaborare la rappresentazione con il numero corretto di URL pre-firmati. |
| `GenericError` | Qualsiasi altro errore imprevisto. |
