---
title: Sviluppare per [!DNL Asset Compute Service].
description: Creazione di applicazioni personalizzate tramite [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 127895cf1bab59546f9ba0be2b3b7a935627effb
workflow-type: tm+mt
source-wordcount: '1496'
ht-degree: 0%

---


# Sviluppo di un&#39;applicazione personalizzata {#develop}

Prima di iniziare a sviluppare un&#39;applicazione personalizzata:

* Verificate che tutti i [prerequisiti](/help/understand-extensibility.md#prerequisites-and-provisioning) siano soddisfatti.
* Installate gli strumenti [software](/help/setup-environment.md#create-dev-environment)richiesti.
* Consultate [Configurare l&#39;ambiente](setup-environment.md) per essere certi di essere pronti per creare un&#39;applicazione personalizzata.

## Creare un&#39;applicazione personalizzata {#create-custom-application}

Assicurarsi che l&#39;interfaccia CLI [di I/O di](https://github.com/adobe/aio-cli) Adobe sia installata localmente.

1. Per creare un&#39;applicazione personalizzata, [create un&#39;app](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)Firefly. Per farlo, eseguire `aio app init <app-name>` nel terminale.

   Se non avete già eseguito l&#39;accesso, questo comando richiede a un browser di accedere a [Adobe Developer Console](https://console.adobe.io/) con l&#39;Adobe ID . Per [ulteriori informazioni sull&#39;accesso dal client, consulta](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) .

    Adobe consiglia di effettuare l’accesso. In caso di problemi, segui le istruzioni [per creare un&#39;app senza effettuare l&#39;accesso](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Dopo l&#39;accesso, seguire i prompt in CLI e selezionare `Organization`, `Project`, e `Workspace` da utilizzare per l&#39;applicazione. Scegliete il progetto e l’area di lavoro creati al momento della [configurazione dell’ambiente](setup-environment.md).

   ```sh
   $ aio app init <app-name>
   Retrieving information from Adobe I/O Console..
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. Quando viene richiesto, `Which Adobe I/O App features do you want to enable for this project?`selezionate almeno `Actions`:

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Quando viene richiesto, `Which type of sample actions do you want to create?`accertatevi di selezionare `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Seguire il resto dei prompt e aprire la nuova applicazione in Visual Studio Code (o l&#39;editor di codice preferito). Contiene l&#39;impalcatura e il codice di esempio per un&#39;applicazione personalizzata.

   Leggi qui i componenti [principali di un&#39;app](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)Firefly.

   L’applicazione modello sfrutta l’SDK [per il calcolo delle](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) risorse per il caricamento, il download e l’orchestrazione di rappresentazioni dell’applicazione, pertanto gli sviluppatori devono solo implementare la logica dell’applicazione personalizzata. All’interno della `actions/<worker-name>` cartella, il `index.js` file è il punto in cui aggiungere il codice dell’applicazione personalizzato.

Per esempi e idee per applicazioni personalizzate, consultate [esempi di applicazioni](#try-sample) personalizzate.

### Aggiungi credenziali {#add-credentials}

Durante l&#39;accesso al momento della creazione dell&#39;applicazione, la maggior parte delle credenziali Firefly vengono raccolte nel file ENV. Tuttavia, l&#39;utilizzo dello strumento di sviluppo richiede credenziali aggiuntive.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenziali di archiviazione dello strumento per sviluppatori {#developer-tool-credentials}

Lo strumento di sviluppo utilizzato per testare applicazioni personalizzate con l&#39;effettivo [!DNL Asset Compute service] richiede un contenitore di archiviazione cloud per ospitare i file di prova e per ricevere e visualizzare le rappresentazioni generate dalle applicazioni.

>[!NOTE]
>
>Questo è separato dall&#39;archiviazione cloud di [!DNL Adobe Experience Manager] come Cloud Service. Si applica solo allo sviluppo e al test con lo strumento di sviluppo Asset Compute.

Assicuratevi di avere accesso a un contenitore [di archiviazione cloud](https://github.com/adobe/asset-compute-devtool#prerequisites)supportato. Questo contenitore può essere condiviso da più sviluppatori in diversi progetti, a seconda delle necessità.

#### Aggiungere credenziali al file ENV {#add-credentials-env-file}

Aggiungete le seguenti credenziali per lo strumento di sviluppo al file ENV nella directory principale del progetto Firefly:

1. Aggiungi il percorso assoluto del file di chiave privata creato durante l&#39;aggiunta di servizi al progetto Firefly:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Aggiungere le credenziali di archiviazione S3 o Azure. È necessario avere accesso a una sola soluzione di archiviazione cloud.

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

## Esecuzione dell&#39;applicazione {#run-custom-application}

Prima di eseguire l’applicazione con Asset Compute Developer Tool, configurate correttamente le [credenziali](#developer-tool-credentials).

Per eseguire l&#39;applicazione nello strumento di sviluppo, utilizzare `aio app run` command. Consente di distribuire l&#39;azione ad Adobe I/O Runtime e avviare lo strumento di sviluppo sul computer locale. Questo strumento viene utilizzato per testare le richieste di applicazione durante lo sviluppo. Esempio di richiesta di rappresentazione:

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
>Non utilizzare il `--local` flag con il `run` comando. Non funziona con le applicazioni [!DNL Asset Compute] personalizzate e lo strumento Asset Compute Developer. Le applicazioni personalizzate vengono richiamate dall&#39;utente [!DNL Asset Compute Service] che non può accedere alle azioni eseguite sui computer locali dello sviluppatore.

Vedere [qui](test-custom-application.md) come eseguire il test e il debug dell&#39;applicazione. Al termine dello sviluppo dell&#39;applicazione personalizzata, [distribuite l&#39;applicazione](deploy-custom-application.md)personalizzata.

## Provare l&#39;applicazione di esempio fornita dal Adobe  {#try-sample}

Di seguito sono riportati alcuni esempi di applicazioni personalizzate:

* [lavoratore-base](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [immagini di operai-animali](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Applicazione personalizzata modello {#template-custom-application}

Il [lavoratore-base](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) è un modello di applicazione. Viene generata una rappresentazione semplicemente copiando il file sorgente. Il contenuto di questa applicazione è il modello ricevuto quando si sceglie `Adobe Asset Compute` nella creazione dell&#39;app aio.

Il file dell&#39;applicazione, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) utilizza l&#39;applicazione [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) per scaricare il file di origine, orchestrare ogni elaborazione di rappresentazione e caricare di nuovo i rendering risultanti nell&#39;archiviazione cloud.

All&#39;interno del codice dell&#39;applicazione è [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) definito il punto in cui eseguire tutta la logica di elaborazione dell&#39;applicazione. Il callback di rappresentazione in `worker-basic` copia semplicemente il contenuto del file sorgente nel file di rappresentazione.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Chiamare un&#39;API esterna {#call-external-api}

Nel codice dell&#39;applicazione potete effettuare chiamate API esterne per facilitare l&#39;elaborazione dell&#39;applicazione. Di seguito è riportato un file di esempio di applicazione che richiama l&#39;API esterna.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Ad esempio, [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) effettua una richiesta di recupero a un URL statico da Wikimedia tramite la [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) libreria.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Passa parametri personalizzati {#pass-custom-parameters}

È possibile trasmettere parametri definiti personalizzati attraverso gli oggetti di rappresentazione. All&#39;interno dell&#39;applicazione è possibile farvi riferimento nelle [`rendition` istruzioni](https://github.com/adobe/asset-compute-sdk#rendition). Un esempio di oggetto di rappresentazione è:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Un esempio di file applicazione che accede al parametro personalizzato è:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

Il file `example-worker-animal-pictures` passa un parametro personalizzato [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) per determinare quale file recuperare da Wikimedia.

## Supporto per autenticazione e autorizzazione {#authentication-authorization-support}

Per impostazione predefinita, le applicazioni personalizzate Asset Compute sono provviste di controlli di autorizzazione e autenticazione per le applicazioni Firefly. Questa opzione è attivata impostando l’ `require-adobe-auth` annotazione su `true` nella `manifest.yml`.

### Accesso ad altre API  Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Aggiungi i servizi API all&#39;area di lavoro della [!DNL Asset Compute] console creata nella configurazione. Questi servizi fanno parte del token di accesso JWT generato da [!DNL Asset Compute Service]. Il token e le altre credenziali sono accessibili all&#39;interno dell&#39; `params` oggetto dell&#39;azione dell&#39;applicazione.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Trasmettere le credenziali per i sistemi di terze parti {#pass-credentials-for-tp}

Per gestire le credenziali per altri servizi esterni, trasmettetele come parametri predefiniti sulle azioni. Questi vengono criptati automaticamente in transito. Per ulteriori informazioni, vedete [Creazione di azioni nella guida](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)per gli sviluppatori Runtime. Quindi impostateli utilizzando le variabili di ambiente durante la distribuzione. Questi parametri sono accessibili nell&#39; `params` oggetto contenuto nell&#39;azione.

Impostate i parametri predefiniti all’interno `inputs` della `manifest.yml`:

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

L&#39; `$VAR` espressione legge il valore da una variabile di ambiente denominata `VAR`.

Durante lo sviluppo, il valore può essere impostato nel file ENV locale, in quanto legge `aio` automaticamente le variabili di ambiente dai file ENV oltre alle variabili impostate dalla shell di chiamata. In questo esempio, il file ENV si presenta come:

```CONF
#...
SECRET_KEY=secret-value
```

Per la distribuzione della produzione è possibile impostare le variabili di ambiente nel sistema CI, ad esempio utilizzando i segreti in Azioni GitHub. Infine, potete accedere ai parametri predefiniti all&#39;interno dell&#39;applicazione:

```javascript
const key = params.secretKey;
```

## Ridimensionamento delle applicazioni {#sizing-workers}

Un&#39;applicazione viene eseguita in un contenitore in Adobe I/O Runtime con [limiti](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) configurabili tramite `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

A causa dell&#39;elaborazione più ampia solitamente eseguita dalle applicazioni Asset Compute, è più probabile che si debbano regolare questi limiti per ottenere prestazioni ottimali (abbastanza grandi da gestire risorse binarie) ed efficienza (non spreco di risorse a causa di memoria contenitore inutilizzata).

Il timeout predefinito per le azioni in Runtime è di un minuto, ma può essere aumentato impostando il `timeout` limite (in millisecondi). Se prevedete di elaborare file più grandi, aumentate questo tempo. Considerate il tempo totale necessario per scaricare l’origine, elaborare il file e caricare la rappresentazione. Se un&#39;azione si interrompe, ovvero non restituisce l&#39;attivazione prima del limite di timeout specificato, Runtime scarta il contenitore e non lo riutilizza.

Le applicazioni di calcolo delle risorse tendono ad essere associate a un binding di I/O in rete e su disco. Il file di origine deve essere scaricato per primo, l&#39;elaborazione è spesso pesante in I/O e le rappresentazioni risultanti vengono caricate di nuovo.

La memoria disponibile per un contenitore di azioni è specificata da `memorySize` in MB. Attualmente questo definisce anche l&#39;accesso della CPU al contenitore, e soprattutto è un elemento chiave del costo dell&#39;utilizzo di Runtime (contenitori più grandi costano di più). Utilizzate un valore più elevato quando l&#39;elaborazione richiede più memoria o CPU, ma prestate attenzione a non sprecare risorse poiché più grandi sono i contenitori, più bassa è la velocità effettiva complessiva.

Inoltre, è possibile controllare la concorrenza dell&#39;azione all&#39;interno di un contenitore utilizzando l&#39; `concurrency` impostazione. Si tratta del numero di attivazioni simultanee ottenute da un singolo contenitore (della stessa azione). In questo modello, il contenitore di azioni è simile a un server Node.js che riceve più richieste simultanee, fino a tale limite. Se non è impostato, il valore predefinito in Runtime è 200, che è ottimo per le azioni Firefly più piccole, ma in genere troppo grande per le applicazioni Asset Compute, data la maggiore attività di elaborazione locale e disco. Alcune applicazioni, a seconda della loro implementazione, potrebbero anche non funzionare correttamente con l&#39;attività concorrente. L’SDK per elaborazione risorse garantisce che le attivazioni siano separate scrivendo file in diverse cartelle univoche.

Test delle applicazioni per trovare i numeri ottimali per `concurrency` e `memorySize`. Contenitori più grandi = limite di memoria più alto potrebbe consentire una maggiore concorrenza ma potrebbe anche essere utile per il traffico più basso.
