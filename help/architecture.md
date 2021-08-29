---
title: Architettura di [!DNL Asset Compute Service]
description: Come  [!DNL Asset Compute Service] API, applicazioni e SDK funzionano insieme per fornire un servizio di elaborazione delle risorse nativo per il cloud.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '485'
ht-degree: 0%

---

# Architettura di [!DNL Asset Compute Service] {#overview}

Il [!DNL Asset Compute Service] è costruito sulla piattaforma runtime senza server [!DNL Adobe I/O]. Fornisce il supporto per le risorse dei servizi di contenuti Adobe Sensei. Al client di chiamata (è supportato solo [!DNL Experience Manager] as a [!DNL Cloud Service]) vengono fornite le informazioni generate da Adobe Sensei richieste per la risorsa. Le informazioni restituite sono in formato JSON.

[!DNL Asset Compute Service] può essere esteso creando applicazioni personalizzate basate su  [!DNL Project Firefly]. Queste applicazioni personalizzate sono [!DNL Project Firefly] headless apps e eseguono attività quali l’aggiunta di strumenti di conversione personalizzati o la chiamata di API esterne per eseguire operazioni sull’immagine.

[!DNL Project Firefly] è un framework per creare e distribuire applicazioni web personalizzate in  [!DNL Adobe I/O] fase di esecuzione. Per creare applicazioni personalizzate, gli sviluppatori possono utilizzare [!DNL React Spectrum] (toolkit dell’interfaccia utente di Adobe), creare microservizi, creare eventi personalizzati e orchestrare API. Consulta la [documentazione di Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

La base su cui si basa l’architettura include:

* La modularità delle applicazioni - che contiene solo ciò che è necessario per una determinata attività - consente di separare le applicazioni tra loro e di mantenerle leggere.

* Il concetto senza server di [!DNL Adobe I/O] Runtime offre numerosi vantaggi: elaborazione asincrona, altamente scalabile, isolata e basata su processi, ideale per l’elaborazione delle risorse.

* L’archiviazione cloud binaria fornisce le funzioni necessarie per memorizzare e accedere ai file di risorse e alle rappresentazioni singolarmente, senza richiedere autorizzazioni di accesso completo all’archiviazione, utilizzando riferimenti URL pre-firmati. L&#39;accelerazione del trasferimento, la memorizzazione in cache CDN e la co-localizzazione di applicazioni di elaborazione con l&#39;archiviazione cloud consentono un accesso ottimale ai contenuti a bassa latenza. Sono supportate sia le cloud AWS che Azure.

![Architettura del servizio Asset compute](assets/architecture-diagram.png)

*Figura: Architettura  [!DNL Asset Compute Service] e modalità di integrazione con le applicazioni di  [!DNL Experience Manager], storage ed elaborazione.*

L&#39;architettura è costituita dalle seguenti parti:

* **Un** livello API e orchestrazione riceve richieste (in formato JSON) che istruiscono il servizio a trasformare una risorsa sorgente in più rappresentazioni. Le richieste sono asincrone e vengono restituite con un ID di attivazione, che è l’ID del processo. Le istruzioni sono puramente dichiarative e per tutti i lavori di elaborazione standard (ad esempio generazione di miniature, estrazione di testo) i consumatori specificano solo il risultato desiderato, ma non le applicazioni che gestiscono determinate rappresentazioni. Le funzionalità API generiche, come l’autenticazione, l’analisi, le limitazioni delle tariffe, vengono gestite utilizzando l’Adobe gateway API di fronte al servizio e gestiscono tutte le richieste che vanno in [!DNL Adobe I/O] Runtime. Il routing dell&#39;applicazione viene eseguito dinamicamente dal livello di orchestrazione. I client possono specificare applicazioni personalizzate per rappresentazioni specifiche e includere parametri personalizzati. L&#39;esecuzione dell&#39;applicazione può essere completamente parallelizzata in quanto sono funzioni diverse senza server in [!DNL Adobe I/O] Runtime.

* **Applicazioni per elaborare** risorse specializzate in determinati tipi di formati di file o rappresentazioni di destinazione. Concettualmente, un&#39;applicazione è come il concetto di tubo Unix: un file di input viene trasformato in uno o più file di output.

* **Una  [comune ](https://github.com/adobe/asset-compute-sdk)** libreria di applicazioni gestisce le attività comuni come scaricare il file sorgente, caricare le rappresentazioni, generare rapporti sugli errori, inviare eventi e monitorare . Questo è progettato in modo che lo sviluppo di un&#39;applicazione rimanga il più semplice possibile, seguendo l&#39;idea senza server, e può essere limitato alle interazioni locali del filesystem.

<!-- TBD:

* About the YAML file?
* See [https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
