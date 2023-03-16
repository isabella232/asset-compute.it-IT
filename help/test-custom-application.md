---
title: Test e debug [!DNL Asset Compute Service] applicazione personalizzata
description: Test e debug [!DNL Asset Compute Service] applicazione personalizzata.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# Test ed esecuzione del debug di un&#39;applicazione personalizzata {#test-debug-custom-worker}

## Esecuzione di unit test per un&#39;applicazione personalizzata {#test-custom-worker}

Installa [Docker Desktop](https://www.docker.com/get-started) sulla tua macchina. Per testare un processo di lavoro personalizzato, eseguire il comando seguente nella directory principale dell&#39;applicazione:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Viene eseguito un framework di test di unità personalizzata per Asset compute le azioni dell’applicazione nel progetto come descritto di seguito. È collegato tramite una configurazione nel `package.json` file. È anche possibile avere unit test JavaScript come Jest. `aio app test` esegue entrambe.

La [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) il plug-in è incorporato come dipendenza di sviluppo nell’app dell’applicazione personalizzata in modo che non debba essere installato nei sistemi di build/test.

### Quadro di prova dell&#39;unità applicativa {#unit-test-framework}

Il framework di test dell&#39;unità applicativa di Asset compute consente di testare le applicazioni senza scrivere alcun codice. Si basa sul principio di origine del file di rendering delle applicazioni. È necessario impostare una determinata struttura di file e cartelle per definire i casi di test con i file di origine dei test, i parametri facoltativi, le rappresentazioni previste e gli script di convalida personalizzati. Per impostazione predefinita, le rappresentazioni vengono confrontate per l’uguaglianza dei byte. Inoltre, i servizi HTTP esterni possono essere facilmente mascherati utilizzando semplici file JSON.

### Aggiungere test {#add-tests}

I test sono attesi all’interno del `test` a livello principale del [!DNL Adobe I/O] progetto. I casi di prova per ciascuna applicazione devono essere nel percorso `test/asset-compute/<worker-name>`, con una cartella per ogni test case:

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

Date un&#39;occhiata [esempio di applicazioni personalizzate](https://github.com/adobe/asset-compute-example-workers/) per alcuni esempi. Di seguito è riportato un riferimento dettagliato.

### Uscita del test {#test-output}

L’output dettagliato del test, compresi i registri dell’applicazione personalizzata, è disponibile nella sezione `build` nella directory principale dell’app Adobe Developer App Builder, come illustrato nella `aio app test` output.

### Bloccare i servizi esterni {#mock-external-services}

È possibile prendere in giro le chiamate al servizio esterno nelle tue azioni definendo `mock-<HOST_NAME>.json` file nei casi di test, dove HOST_NAME è l&#39;host che desideri prendere in giro. Un esempio di caso d&#39;uso è un&#39;applicazione che effettua una chiamata separata a S3. La nuova struttura del test sarà simile alla seguente:

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

Il file fittizio è una risposta http in formato JSON. Per ulteriori informazioni, consulta [questa documentazione](https://www.mock-server.com/mock_server/creating_expectations.html). Se ci sono più nomi host da simulare, definisci più `mock-<mocked-host>.json` file. Di seguito è riportato un esempio di file di simulazione per `google.com` denominato `mock-google.com.json`:

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

L&#39;esempio `worker-animal-pictures` contiene [file simulato](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) per il servizio Wikimedia con cui interagisce.

#### Condividere i file tra i casi di test {#share-files-across-test-cases}

Si consiglia di utilizzare i symlink relativi se si condivide `file.*`, `params.json` o `validate` script per più test. Sono supportati con Git. Assicurati di assegnare ai file condivisi un nome univoco, in quanto potrebbero essere presenti file diversi. Nell&#39;esempio seguente i test stanno combinando e confrontando alcuni file condivisi e i loro:

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

I casi di test di errore non devono contenere `rendition.*` e deve definire il `errorReason` all&#39;interno del `params.json` file.

>[!NOTE]
>
>Se un test case non contiene un `rendition.*` e non definisce il `errorReason` all&#39;interno del `params.json` file, si presume sia un caso di errore con qualsiasi `errorReason`.

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

Vedi l&#39;elenco completo e la descrizione di [asset compute di motivi di errore](https://github.com/adobe/asset-compute-commons#error-reasons).

## Debug di un&#39;applicazione personalizzata {#debug-custom-worker}

I passaggi seguenti mostrano come eseguire il debug dell&#39;applicazione personalizzata utilizzando Visual Studio Code. Consente di visualizzare log in tempo reale, punti di interruzione e codice dettagliato, nonché di ricaricare in tempo reale le modifiche al codice locale a ogni attivazione.

Molti di questi passaggi sono in genere automatizzati da `aio` out-of-the-box, consulta la sezione Debug dell’applicazione in [Documentazione di Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Per il momento, i passaggi seguenti includono una soluzione alternativa.

1. Installa la versione più recente [wskdebug](https://github.com/apache/openwhisk-wskdebug) da GitHub e l’facoltativo [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Aggiungi al file JSON delle impostazioni utente. Continua a utilizzare il vecchio debugger del codice VS, il nuovo ha [alcuni problemi](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Chiudi le istanze di app aperte tramite `aio app run`.
1. Distribuisci il codice più recente utilizzando `aio app deploy`.
1. Esegui solo lo strumento Devtool Asset compute utilizzando `aio asset-compute devtool`. Tenetela aperta.
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

   Recupera la `ACTION NAME` dall&#39;uscita di `aio app deploy`.

1. Seleziona `wskdebug worker` dalla configurazione di esecuzione/debug e premi l&#39;icona di riproduzione. Attendi che inizi finché non viene visualizzato **[!UICONTROL Pronti per le attivazioni]** in **[!UICONTROL Console di debug]** finestra.

1. Fai clic su **[!UICONTROL eseguire]** nel Devtool. Puoi vedere le azioni in esecuzione nell’editor di codice VS e i registri iniziano a essere visualizzati.

1. Imposta un punto di interruzione nel codice, esegui di nuovo e dovrebbe colpire.

Tutte le modifiche al codice vengono caricate in tempo reale e hanno effetto non appena si verifica l’attivazione successiva.

>[!NOTE]
>
>Sono presenti due attivazioni per ogni richiesta nelle applicazioni personalizzate. La prima richiesta è un’azione web che si chiama in modo asincrono nel codice SDK. La seconda attivazione è quella che colpisce il codice.
