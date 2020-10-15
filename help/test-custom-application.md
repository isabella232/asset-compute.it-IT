---
title: Applicazione personalizzata per test e [!DNL Asset Compute Service] debug.
description: Applicazione personalizzata per test e [!DNL Asset Compute Service] debug.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '788'
ht-degree: 0%

---


# Test ed esecuzione del debug di un&#39;applicazione personalizzata {#test-debug-custom-worker}

## Esecuzione di unit test per un&#39;applicazione personalizzata {#test-custom-worker}

Installate [Docker Desktop](https://www.docker.com/get-started) nel computer. Per testare un lavoratore personalizzato, eseguire il comando seguente nella directory principale dell&#39;applicazione:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Viene eseguito un framework di unit test personalizzato per le azioni dell&#39;applicazione Asset Compute nel progetto come descritto di seguito. Viene collegato tramite una configurazione nel `package.json` file. È inoltre possibile disporre di unit test JavaScript come Jest. `aio app test` esegue entrambi.

Il plug-in per elaborazione risorse [aio-cli-plugin](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) è incorporato come dipendenza di sviluppo nell&#39;app dell&#39;applicazione personalizzata in modo che non sia necessario installarlo nei sistemi di build/test.

### Framework di test dell&#39;unità applicativa {#unit-test-framework}

Il framework di test dell&#39;unità applicativa Asset Compute consente di testare le applicazioni senza scrivere codice. Si basa sul principio del file di origine per la rappresentazione delle applicazioni. È necessario impostare una determinata struttura di file e cartelle per definire i casi di test con i file di origine dei test, i parametri facoltativi, le rappresentazioni previste e gli script di convalida personalizzati. Per impostazione predefinita, le rappresentazioni vengono confrontate per l&#39;uguaglianza dei byte. Inoltre, i servizi HTTP esterni possono essere facilmente mascherati utilizzando semplici file JSON.

### Aggiungere test {#add-tests}

Sono previsti test all&#39;interno della `test` cartella al livello principale del progetto AIO. I casi di test per ciascuna applicazione devono trovarsi nel percorso `test/asset-compute/<worker-name>`, con una cartella per ogni test case:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Per alcuni esempi, consultate [alcune applicazioni](https://github.com/adobe/asset-compute-example-workers/) personalizzate. Di seguito è riportato un riferimento dettagliato.

### Test output {#test-output}

L&#39;output dettagliato del test, compresi i registri dell&#39;applicazione personalizzata, sono disponibili nella `build` cartella principale dell&#39;app Firefly come dimostrato nell&#39; `aio app test` output.

### Servizi esterni {#mock-external-services}

È possibile simulare le chiamate di servizio esterne nelle azioni definendo `mock-<HOST_NAME>.json` i file nei casi di test, dove HOST_NAME è l&#39;host che si desidera controllare. Un esempio di utilizzo è un&#39;applicazione che esegue una chiamata separata a S3. La nuova struttura del test sarà simile alla seguente:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

Il file del mock è una risposta HTTP in formato JSON. Per ulteriori informazioni, consulta [questa documentazione](https://www.mock-server.com/mock_server/creating_expectations.html). Se vi sono più nomi host da simulare, definite più `mock-<mocked-host>.json` file. Di seguito è riportato un esempio di file di modello per `google.com` denominato `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

L&#39;esempio `worker-animal-pictures` contiene un [file](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) fittizio per il servizio Wikimedia con cui interagisce.

#### Condivisione di file tra i test case {#share-files-across-test-cases}

Si consiglia di utilizzare i collegamenti di tipo relativo se si condividono `file.*`, `params.json` o `validate` script tra più test. Sono supportati con git. Accertatevi di assegnare ai file condivisi un nome univoco, in quanto potrebbero essere presenti file diversi. Nell&#39;esempio seguente, i test combinano e corrispondono ad alcuni file condivisi, e i relativi:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Test degli errori previsti {#test-unexpected-errors}

I casi di test di errore non devono contenere un `rendition.*` file previsto e devono definire il file previsto `errorReason` all&#39;interno del `params.json` file.

Struttura del caso di test di errore:

```json
<error_test_case>/
    file.jpg
    params.json
```

File di parametro con motivo di errore:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Consultate l’elenco completo e la descrizione dei motivi [di errore di calcolo delle](https://github.com/adobe/asset-compute-commons#error-reasons)risorse.

## Debug di un&#39;applicazione personalizzata {#debug-custom-worker}

Nei passaggi seguenti viene illustrato come eseguire il debug dell&#39;applicazione personalizzata utilizzando Visual Studio Code. Consente di visualizzare i log in diretta, i punti di interruzione degli hit e i passaggi del codice, nonché il ricaricamento in tempo reale delle modifiche del codice locale a ogni attivazione.

Molti di questi passaggi sono generalmente automatizzati `aio` dal prodotto, vedere la sezione Debug dell&#39;applicazione nella documentazione [](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)Firefly. Per il momento, i passaggi seguenti includono una soluzione alternativa.

1. Installate la versione [wskdebug](https://github.com/apache/openwhisk-wskdebug) più recente da GitHub e dal [gruppo](https://www.npmjs.com/package/ngrok)opzionale.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Aggiungi al file JSON delle impostazioni utente. Continua a utilizzare il vecchio VS Code Debugger, il nuovo ha [alcuni problemi](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Chiudete tutte le istanze di app aperte tramite `aio app run`.
1. Distribuisci il codice più recente utilizzando `aio app deploy`.
1. Eseguire solo lo strumento di calcolo risorse utilizzando `npx adobe-asset-compute devtool`. Lo tenga aperto.
1. Nell&#39;editor del codice VS, aggiungi la seguente configurazione di debug al `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Recupera il NOME AZIONE dall&#39;output di `aio app deploy`. Sembra `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`.

1. Selezionare `wskdebug worker` dalla configurazione di esecuzione/debug e premere l&#39;icona di riproduzione. Attendi che inizi finché non viene visualizzato **[!UICONTROL Pronto per le attivazioni]** nella finestra **[!UICONTROL Console]** di debug.

1. Fare clic su **[!UICONTROL Esegui]** in Devtool. Potete visualizzare le azioni eseguite nell&#39;editor del codice VS e i registri iniziano a essere visualizzati.

1. Imposta un punto di interruzione nel codice, esegui di nuovo e dovrebbe colpire.

Qualsiasi modifica del codice viene caricata in tempo reale e ha effetto non appena si verifica la successiva attivazione.

>[!NOTE]
>
>Sono presenti due attivazioni per ogni richiesta in applicazioni personalizzate. La prima richiesta è un&#39;azione Web che si richiama in modo asincrono nel codice SDK. La seconda attivazione è quella che colpisce il codice.
