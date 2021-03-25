---
title: Sviluppa per [!DNL Asset Compute Service]
description: Crea applicazioni personalizzate utilizzando [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 7ae47fdb7ff91e1388d2037d90abe35fe5218216
workflow-type: tm+mt
source-wordcount: '1615'
ht-degree: 0%

---


# Sviluppa un&#39;applicazione personalizzata {#develop}

Prima di iniziare a sviluppare un’applicazione personalizzata:

* Assicurati che tutti i [prerequisiti](/help/understand-extensibility.md#prerequisites-and-provisioning) siano soddisfatti.
* Installare gli [strumenti software richiesti](/help/setup-environment.md#create-dev-environment).
* Consulta [configurare l&#39;ambiente](setup-environment.md) per essere sicuri di essere pronti per creare un&#39;applicazione personalizzata.

## Creare un&#39;applicazione personalizzata {#create-custom-application}

Assicurati di aver installato [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) localmente.

1. Per creare un&#39;applicazione personalizzata, [crea un&#39;app Firefly](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli). A questo scopo, esegui `aio app init <app-name>` nel terminale.

   Se non hai già effettuato l’accesso, questo comando richiede a un browser di accedere alla [Console per sviluppatori di Adobe](https://console.adobe.io/) con il tuo Adobe ID. Per ulteriori informazioni sull&#39;accesso dal client, consulta [qui](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) .

   Adobe consiglia di effettuare l&#39;accesso. In caso di problemi, segui le istruzioni [per creare un&#39;app senza effettuare l&#39;accesso](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Dopo aver effettuato l’accesso, segui le istruzioni contenute in CLI e seleziona `Organization`, `Project` e `Workspace` da utilizzare per l’applicazione. Scegli il progetto e l&#39;area di lavoro creati quando [imposti l&#39;ambiente](setup-environment.md).

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. Quando richiesto con `Which Adobe I/O App features do you want to enable for this project?`, seleziona `Actions`. Accertati di deselezionare l’opzione `Web Assets` in quanto le risorse web utilizzano diversi controlli di autenticazione e autorizzazione.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Quando viene richiesto `Which type of sample actions do you want to create?`, assicurati di selezionare `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Seguire il resto dei prompt e aprire la nuova applicazione in Visual Studio Code (o nell&#39;editor di codice preferito). Contiene l’impalcatura e il codice di esempio per un’applicazione personalizzata.

   Leggi qui i [componenti principali di un&#39;app Firefly](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

   L&#39;applicazione modello sfrutta il nostro [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) per il caricamento, il download e l&#39;orchestrazione dei rendering dell&#39;applicazione, pertanto gli sviluppatori devono solo implementare la logica dell&#39;applicazione personalizzata. All&#39;interno della cartella `actions/<worker-name>`, il file `index.js` è il punto in cui aggiungere il codice personalizzato dell&#39;applicazione.

Per esempi e idee per le applicazioni personalizzate, consulta [esempi di applicazioni personalizzate](#try-sample) .

### Aggiungi credenziali {#add-credentials}

Durante l&#39;accesso durante la creazione dell&#39;applicazione, la maggior parte delle credenziali Firefly viene raccolta nel file ENV. Tuttavia, l’utilizzo dello strumento per sviluppatori richiede credenziali aggiuntive.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenziali di archiviazione degli strumenti per sviluppatori {#developer-tool-credentials}

Lo strumento per sviluppatori utilizzato per testare le applicazioni personalizzate con l&#39;effettivo [!DNL Asset Compute service] richiede un contenitore di archiviazione cloud per ospitare i file di test e per ricevere e visualizzare le rappresentazioni generate dalle applicazioni.

>[!NOTE]
>
>Questo è separato dall’archiviazione cloud di [!DNL Adobe Experience Manager] come [!DNL Cloud Service]. Si applica solo per lo sviluppo e il test con lo strumento per sviluppatori Asset compute.

Assicurati di avere accesso a un [contenitore di archiviazione cloud supportato](https://github.com/adobe/asset-compute-devtool#prerequisites). Se necessario, questo contenitore può essere condiviso da più sviluppatori tra progetti diversi.

#### Aggiungi credenziali al file ENV {#add-credentials-env-file}

Aggiungi le seguenti credenziali per lo strumento sviluppatore al file ENV nella directory principale del progetto Firefly:

1. Aggiungi il percorso assoluto al file di chiave privata creato durante l&#39;aggiunta di servizi al progetto Firefly:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Scarica il file da Adobe Developer Console. Vai alla directory principale del progetto e fai clic su &quot;Scarica tutto&quot; nell’angolo in alto a destra. Il file viene scaricato con `<namespace>-<workspace>.json` come nome del file. Effettua una delle operazioni seguenti:

   * Rinomina il file come `console.json` e spostalo nella directory principale del progetto.
   * Facoltativamente, puoi aggiungere il percorso assoluto al file JSON dell’integrazione Adobe Developer Console. Si tratta dello stesso file [`console.json`](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user) scaricato nell’area di lavoro del progetto.

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. Aggiungi le credenziali di archiviazione S3 o Azure. È necessario accedere a una sola soluzione di archiviazione cloud.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>Il file `config.json` contiene credenziali. Dall’interno del progetto, aggiungi il file JSON al file `.gitignore` per impedirne la condivisione. Lo stesso vale per i file .env e .aio.

## Esegui l&#39;applicazione {#run-custom-application}

Prima di eseguire l&#39;applicazione con Asset compute Developer Tool, configura correttamente le [credenziali](#developer-tool-credentials).

Per eseguire l&#39;applicazione nello strumento sviluppatore, utilizza il comando `aio app run` . Distribuisce l’azione in [!DNL Adobe I/O] Runtime e avvia lo strumento di sviluppo sul computer locale. Questo strumento viene utilizzato per testare le richieste dell&#39;applicazione durante lo sviluppo. Ecco un esempio di richiesta di rendering:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>Non utilizzare il flag `--local` con il comando `run`. Non funziona con le applicazioni personalizzate [!DNL Asset Compute] e lo strumento Asset compute Developer. Le applicazioni personalizzate vengono richiamate da [!DNL Asset Compute Service] che non possono accedere alle azioni eseguite sui computer locali dello sviluppatore.

Vedi [qui](test-custom-application.md) come testare ed eseguire il debug dell&#39;applicazione. Al termine dello sviluppo dell&#39;applicazione personalizzata, [distribuire l&#39;applicazione personalizzata](deploy-custom-application.md).

## Prova l&#39;applicazione di esempio fornita dall&#39;Adobe {#try-sample}

Di seguito sono riportati alcuni esempi di applicazioni personalizzate:

* [lavoratore-base](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [foto di operaio-animale](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Applicazione personalizzata del modello {#template-custom-application}

Il [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) è un&#39;applicazione modello. Genera un rendering semplicemente copiando il file di origine. Il contenuto di questa applicazione è il modello ricevuto quando si sceglie `Adobe Asset Compute` nella creazione dell&#39;app aio.

Il file dell&#39;applicazione [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) utilizza [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) per scaricare il file di origine, orchestrare ogni elaborazione del rendering e caricare di nuovo i rendering risultanti nell&#39;archiviazione cloud.

Il [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) definito all&#39;interno del codice dell&#39;applicazione è il punto in cui eseguire tutta la logica di elaborazione dell&#39;applicazione. Il callback di rendering in `worker-basic` copia semplicemente il contenuto del file di origine nel file di rendering.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Chiamare un&#39;API esterna {#call-external-api}

Nel codice dell’applicazione puoi effettuare chiamate API esterne per facilitare l’elaborazione dell’applicazione. Di seguito è riportato un file di applicazione di esempio che richiama l’API esterna.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Ad esempio, il [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) invia una richiesta di recupero a un URL statico da Wikimedia utilizzando la libreria [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer).

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Passa parametri personalizzati {#pass-custom-parameters}

È possibile trasmettere parametri definiti personalizzati attraverso gli oggetti di rendering. È possibile farvi riferimento all&#39;interno dell&#39;applicazione in [`rendition` istruzioni](https://github.com/adobe/asset-compute-sdk#rendition). Un esempio di oggetto rendering è:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Un esempio di file di applicazione che accede al parametro personalizzato è:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

Il `example-worker-animal-pictures` passa un parametro personalizzato [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) per determinare quale file recuperare da Wikimedia.

## Supporto per autenticazione e autorizzazione {#authentication-authorization-support}

Per impostazione predefinita, le applicazioni personalizzate Asset compute sono dotate di controlli di autorizzazione e autenticazione per le applicazioni Firefly. Per attivarlo, imposta l’annotazione `require-adobe-auth` su `true` in `manifest.yml`.

### Accedere ad altre API di Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Aggiungi i servizi API all’area di lavoro della console [!DNL Asset Compute] creata nella configurazione. Questi servizi fanno parte del token di accesso JWT generato da [!DNL Asset Compute Service]. Il token e le altre credenziali sono accessibili all&#39;interno dell&#39;oggetto `params` dell&#39;azione dell&#39;applicazione.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Passa le credenziali per i sistemi di terze parti {#pass-credentials-for-tp}

Per gestire le credenziali per altri servizi esterni, trasmetterle come parametri predefiniti sulle azioni. Queste vengono criptate automaticamente durante il transito. Per ulteriori informazioni, consulta [creazione di azioni nella guida per sviluppatori di Runtime](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). Quindi impostali utilizzando le variabili di ambiente durante la distribuzione. Questi parametri sono accessibili nell’oggetto `params` all’interno dell’azione .

Imposta i parametri predefiniti all&#39;interno di `inputs` in `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

L&#39;espressione `$VAR` legge il valore da una variabile di ambiente denominata `VAR`.

Durante lo sviluppo, il valore può essere impostato nel file ENV locale come `aio` legge automaticamente le variabili di ambiente dai file ENV, oltre alle variabili impostate dalla shell di chiamata. In questo esempio, il file ENV si presenta come:

```CONF
#...
SECRET_KEY=secret-value
```

Per la distribuzione di produzione è possibile impostare le variabili di ambiente nel sistema CI, ad esempio utilizzando i segreti nelle azioni GitHub. Infine, accedi ai parametri predefiniti all&#39;interno dell&#39;applicazione in quanto tali:

```javascript
const key = params.secretKey;
```

## Ridimensionamento delle applicazioni {#sizing-workers}

Un&#39;applicazione viene eseguita in un contenitore in [!DNL Adobe I/O] Runtime con [limits](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) configurabile tramite `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

A causa dell&#39;elaborazione più estesa generalmente eseguita dalle applicazioni Asset compute, è più probabile che si debbano regolare questi limiti per ottenere prestazioni ottimali (abbastanza grandi da gestire le risorse binarie) ed efficienza (senza sprecare risorse a causa della memoria contenitore inutilizzata).

Il timeout predefinito per le azioni in Runtime è di un minuto, ma può essere aumentato impostando il limite `timeout` (in millisecondi). Se si prevede di elaborare file di dimensioni maggiori, aumenta questo tempo. Considera il tempo totale necessario per scaricare l’origine, elaborare il file e caricare il rendering. Se un&#39;azione si interrompe, ovvero non restituisce l&#39;attivazione prima del limite di timeout specificato, Runtime elimina il contenitore e non lo riutilizza.

Le applicazioni di Asset compute tendono ad essere associate a rete e disco Input o output. Il file di origine deve essere scaricato per primo, l’elaborazione richiede spesso un uso intensivo di risorse e quindi le rappresentazioni risultanti vengono caricate di nuovo.

La memoria disponibile per un contenitore di azioni è specificata da `memorySize` in MB. Attualmente questo definisce anche l’accesso alla CPU ottenuto dal contenitore e, cosa più importante, è un elemento chiave del costo dell’utilizzo di Runtime (contenitori più grandi costano di più). Utilizza un valore più grande in questo caso quando l’elaborazione richiede più memoria o CPU, ma fai attenzione a non sprecare risorse poiché più grandi sono i contenitori, più basso è il throughput complessivo.

Inoltre, è possibile controllare la concorrenza delle azioni all’interno di un contenitore utilizzando l’impostazione `concurrency` . È il numero di attivazioni simultanee ottenute da un singolo contenitore (della stessa azione). In questo modello, il contenitore di azioni è simile a un server Node.js che riceve più richieste simultanee, fino a quel limite. Se non è impostato, il valore predefinito in Runtime è 200, ideale per le azioni Firefly più piccole, ma di solito troppo grande per le applicazioni di Asset compute, data la loro elaborazione locale più intensiva e l&#39;attività del disco. Anche alcune applicazioni, a seconda della loro implementazione, potrebbero non funzionare bene con l’attività concorrente. L&#39;SDK di Asset compute assicura che le attivazioni siano separate dalla scrittura di file in cartelle univoche diverse.

Testa le applicazioni per trovare i numeri ottimali per `concurrency` e `memorySize`. Contenitori più grandi = un limite di memoria più alto potrebbe consentire una maggiore concorrenza ma potrebbe anche essere utile per ridurre il traffico.
