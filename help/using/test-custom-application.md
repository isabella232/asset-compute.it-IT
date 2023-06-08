---
title: Test ed debug [!DNL Asset Compute Service] applicazione personalizzata
description: Test ed debug [!DNL Asset Compute Service] applicazione personalizzata.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# Testare ed eseguire il debug di un&#39;applicazione personalizzata {#test-debug-custom-worker}

## Eseguire unit test per un&#39;applicazione personalizzata {#test-custom-worker}

Installa [Docker Desktop](https://www.docker.com/get-started) sul tuo computer. Per testare un processo di lavoro personalizzato, eseguire il comando seguente nella radice dell&#39;applicazione:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

In questo modo viene eseguito un framework di unit test personalizzato per azioni dell&#39;applicazione di Asset compute nel progetto, come descritto di seguito. È collegato tramite una configurazione in `package.json` file. È inoltre possibile disporre di unit test JavaScript, ad esempio Jest. `aio app test` esegue entrambi.

Il [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) il plug-in è incorporato come dipendenza dallo sviluppo nell’app dell’applicazione personalizzata, pertanto non deve essere installato sui sistemi di build/test.

### Framework di test delle unità applicative {#unit-test-framework}

Il framework di test di unità applicativa di Asset compute consente di testare le applicazioni senza scrivere codice. Si basa sul principio di origine per il rendering del file delle applicazioni. È necessario configurare una determinata struttura di file e cartelle per definire i casi di test con file di origine di test, parametri facoltativi, rappresentazioni previste e script di convalida personalizzati. Per impostazione predefinita, le rappresentazioni vengono confrontate per l&#39;uguaglianza dei byte. Inoltre, i servizi HTTP esterni possono essere facilmente presi in giro utilizzando semplici file JSON.

### Aggiungi test {#add-tests}

Sono previsti test all’interno del `test` cartella a livello principale della [!DNL Adobe I/O] progetto. I casi di test per ogni applicazione devono essere nel percorso `test/asset-compute/<worker-name>`, con una cartella per ogni test case:

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

Dai un&#39;occhiata a [esempi di applicazioni personalizzate](https://github.com/adobe/asset-compute-example-workers/) per alcuni esempi. Di seguito è riportato un riferimento dettagliato.

### Output di prova {#test-output}

L’output del test dettagliato, compresi i registri dell’applicazione personalizzata, è disponibile nella sezione `build` nella directory principale dell&#39;app Adobe Developer App Builder, come illustrato nella `aio app test` output.

### Mascherare i servizi esterni {#mock-external-services}

È possibile simulare chiamate di servizio esterne nelle azioni definendo `mock-<HOST_NAME>.json` file nei tuoi casi di test, dove HOST_NAME è l&#39;host che desideri simulare. Un esempio di caso d’uso è un’applicazione che effettua una chiamata separata a S3. La nuova struttura di test sarà simile alla seguente:

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

Il file fittizio è una risposta HTTP in formato JSON. Per ulteriori informazioni, consulta [questa documentazione](https://www.mock-server.com/mock_server/creating_expectations.html). Se esistono più nomi host da simulare, definire più `mock-<mocked-host>.json` file. Di seguito è riportato un esempio di file fittizio per `google.com` denominato `mock-google.com.json`:

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

L’esempio `worker-animal-pictures` contiene un [file fittizio](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) per il servizio Wikimedia con cui interagisce.

#### Condivisione di file tra test case {#share-files-across-test-cases}

Si consiglia di utilizzare i symlink relativi se si condividono `file.*`, `params.json` o `validate` script in più test. Sono supportate con Git. Assicurati di assegnare ai file condivisi un nome univoco, in quanto potrebbero essere presenti file diversi. Nell’esempio seguente i test si combinano e corrispondono a pochi file condivisi e ai loro:

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

I casi di test di errore non devono contenere un `rendition.*` e deve definire il `errorReason` all&#39;interno del `params.json` file.

>[!NOTE]
>
>Se un test case non contiene un valore previsto `rendition.*` e non definisce il file previsto `errorReason` all&#39;interno del `params.json` file, si presume che sia un caso di errore con qualsiasi `errorReason`.

Struttura del test case di errore:

```json
<error_test_case>/
    file.jpg
    params.json
```

File di parametri con motivo errore:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Vedi elenco completo e descrizione di [Motivi di errore Asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Eseguire il debug di un&#39;applicazione personalizzata {#debug-custom-worker}

Nei passaggi seguenti viene illustrato come eseguire il debug dell&#39;applicazione personalizzata utilizzando Visual Studio Code. Consente di visualizzare registri live, punti di interruzione di hit e codice step-through, nonché il ricaricamento in tempo reale delle modifiche al codice locale a ogni attivazione.

Molti di questi passaggi sono in genere automatizzati da `aio` con, consulta la sezione Debug dell’applicazione in [Documentazione di Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Per il momento, i passaggi seguenti includono una soluzione alternativa.

1. Installa la versione più recente [wskdebug](https://github.com/apache/openwhisk-wskdebug) da GitHub e le opzioni [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Aggiungi al file JSON delle impostazioni utente. Continua a utilizzare il vecchio debugger VS Code, il nuovo ha [alcuni problemi](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Chiudi tutte le istanze di app aperte tramite `aio app run`.
1. Distribuisci il codice più recente tramite `aio app deploy`.
1. Esegui solo lo strumento di sviluppo Asset compute utilizzando `aio asset-compute devtool`. Tienilo aperto.
1. Nell’editor di codice VS, aggiungi la seguente configurazione di debug al tuo `launch.json`:

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

1. Seleziona `wskdebug worker` dalla configurazione esegui/debug e premi l’icona play. Attendi che inizi finché non viene visualizzato **[!UICONTROL Pronto per le attivazioni]** nel **[!UICONTROL Console di debug]** finestra.

1. Clic **[!UICONTROL eseguire]** in Devtool. Puoi vedere le azioni in esecuzione nell’editor di codice VS e i registri iniziano a essere visualizzati.

1. Imposta un punto di interruzione nel codice, esegui di nuovo e dovrebbe premere.

Eventuali modifiche al codice vengono caricate in tempo reale e diventano effettive non appena si verifica l’attivazione successiva.

>[!NOTE]
>
>Per ogni richiesta nelle applicazioni personalizzate sono presenti due attivazioni. La prima richiesta è un’azione web che si richiama in modo asincrono nel codice SDK. La seconda attivazione è quella che colpisce il codice.
