---
title: Test e debug dell'applicazione personalizzata [!DNL Asset Compute Service] applicazione personalizzata
description: Test e debug dell'applicazione personalizzata [!DNL Asset Compute Service] .
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
translation-type: tm+mt
source-git-commit: 9bc1534671c81a05798f98ae556d348bc771d975
workflow-type: tm+mt
source-wordcount: '782'
ht-degree: 0%

---

# Test ed esecuzione del debug di un&#39;applicazione personalizzata {#test-debug-custom-worker}

## Esegui unit test per un&#39;applicazione personalizzata {#test-custom-worker}

Installa [Docker Desktop](https://www.docker.com/get-started) sul computer. Per testare un processo di lavoro personalizzato, eseguire il comando seguente nella directory principale dell&#39;applicazione:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Viene eseguito un framework di test di unità personalizzata per Asset compute le azioni dell’applicazione nel progetto come descritto di seguito. Viene collegato tramite una configurazione nel file `package.json` . È anche possibile avere unit test JavaScript come Jest. `aio app test` esegue entrambe.

Il plug-in [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) è incorporato come dipendenza di sviluppo nell&#39;app dell&#39;applicazione personalizzata in modo che non sia necessario installarlo nei sistemi di build/test.

### Struttura del test dell&#39;unità applicativa {#unit-test-framework}

Il framework di test dell&#39;unità applicativa di Asset compute consente di testare le applicazioni senza scrivere alcun codice. Si basa sul principio di origine del file di rendering delle applicazioni. È necessario impostare una determinata struttura di file e cartelle per definire i casi di test con i file di origine dei test, i parametri facoltativi, le rappresentazioni previste e gli script di convalida personalizzati. Per impostazione predefinita, le rappresentazioni vengono confrontate per l’uguaglianza dei byte. Inoltre, i servizi HTTP esterni possono essere facilmente mascherati utilizzando semplici file JSON.

### Aggiungi test {#add-tests}

Sono previsti test all&#39;interno della cartella `test` al livello principale del progetto [!DNL Adobe I/O]. I casi di prova per ciascuna applicazione devono trovarsi nel percorso `test/asset-compute/<worker-name>`, con una cartella per ogni test case:

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

Per alcuni esempi, consulta l’ [esempio applicazioni personalizzate](https://github.com/adobe/asset-compute-example-workers/) . Di seguito è riportato un riferimento dettagliato.

### Uscita del test {#test-output}

L’output dettagliato del test, compresi i registri dell’applicazione personalizzata, è disponibile nella cartella `build` nella directory principale dell’app Firefly, come illustrato nell’ `aio app test` output.

### Bloccare i servizi esterni {#mock-external-services}

È possibile simulare le chiamate al servizio esterno nelle azioni definendo i file `mock-<HOST_NAME>.json` nei casi di test, dove HOST_NAME è l&#39;host di cui si desidera tenere traccia. Un esempio di caso d&#39;uso è un&#39;applicazione che effettua una chiamata separata a S3. La nuova struttura del test sarà simile alla seguente:

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

Il file fittizio è una risposta http in formato JSON. Per ulteriori informazioni, consulta [questa documentazione](https://www.mock-server.com/mock_server/creating_expectations.html). Se ci sono più nomi host da simulare, definisci più file `mock-<mocked-host>.json`. Di seguito è riportato un esempio di file di simulazione per `google.com` denominato `mock-google.com.json`:

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

L&#39;esempio `worker-animal-pictures` contiene un [file di tag](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) per il servizio Wikimedia con cui interagisce.

#### Condividere file tra test case {#share-files-across-test-cases}

Si consiglia di utilizzare i collegamenti simbolici relativi se si condividono script `file.*`, `params.json` o `validate` in più test. Sono supportati con Git. Assicurati di assegnare ai file condivisi un nome univoco, in quanto potrebbero essere presenti file diversi. Nell&#39;esempio seguente i test stanno combinando e confrontando alcuni file condivisi e i loro:

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

I casi di test di errore non devono contenere un file `rendition.*` previsto e devono definire il `errorReason` previsto all&#39;interno del file `params.json`.

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

Vedi l&#39;elenco completo e la descrizione di [Asset compute di motivi di errore](https://github.com/adobe/asset-compute-commons#error-reasons).

## Debug di un&#39;applicazione personalizzata {#debug-custom-worker}

I passaggi seguenti mostrano come eseguire il debug dell&#39;applicazione personalizzata utilizzando Visual Studio Code. Consente di visualizzare log in tempo reale, punti di interruzione e codice dettagliato, nonché di ricaricare in tempo reale le modifiche al codice locale a ogni attivazione.

Molti di questi passaggi sono generalmente automatizzati da `aio` out of the box, vedi la sezione Debug dell&#39;applicazione nella [documentazione Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Per il momento, i passaggi seguenti includono una soluzione alternativa.

1. Installa l&#39;ultimo [wskdebug](https://github.com/apache/openwhisk-wskdebug) da GitHub e l&#39;facoltativo [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Aggiungi al file JSON delle impostazioni utente. Continua a utilizzare il vecchio debugger del codice VS, il nuovo ha [alcuni problemi](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Chiudi tutte le istanze delle app aperte tramite `aio app run`.
1. Distribuisci il codice più recente utilizzando `aio app deploy`.
1. Esegui solo lo strumento Devtool di Asset compute utilizzando `aio asset-compute devtool`. Tenetela aperta.
1. In Editor di codice VS, aggiungi la seguente configurazione di debug al tuo `launch.json`:

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

   Recupera il `ACTION NAME` dall&#39;output di `aio app deploy`.

1. Seleziona `wskdebug worker` dalla configurazione di esecuzione/debug e premi l&#39;icona di riproduzione. Attendi che inizi finché non viene visualizzato **[!UICONTROL Pronto per le attivazioni]** nella finestra **[!UICONTROL Console di debug]**.

1. Fare clic su **[!UICONTROL esegui]** nel Devtool. Puoi vedere le azioni in esecuzione nell’editor di codice VS e i registri iniziano a essere visualizzati.

1. Imposta un punto di interruzione nel codice, esegui di nuovo e dovrebbe colpire.

Tutte le modifiche al codice vengono caricate in tempo reale e hanno effetto non appena si verifica l’attivazione successiva.

>[!NOTE]
>
>Sono presenti due attivazioni per ogni richiesta nelle applicazioni personalizzate. La prima richiesta è un’azione web che si chiama in modo asincrono nel codice SDK. La seconda attivazione è quella che colpisce il codice.
