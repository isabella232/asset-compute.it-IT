---
title: Architettura di [!DNL Asset Compute Service].
description: Come  [!DNL Asset Compute Service] API, applicazioni e SDK funzionano insieme per fornire un servizio di elaborazione delle risorse nativo per il cloud.
translation-type: tm+mt
source-git-commit: 0fb256f7d9f83fbae564d9fd52ee6b2f34c5d7e9
workflow-type: tm+mt
source-wordcount: '496'
ht-degree: 0%

---


# Architettura di [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] è basato sulla piattaforma Adobe I/O Runtime senza server. Fornisce  supporto dei servizi di contenuto Adobe Sensei per le risorse. Il client di chiamata (solo [!DNL Experience Manager] come Cloud Service è supportato) viene fornito con le  informazioni generate da Adobe Sensei richieste per la risorsa. Le informazioni restituite sono in formato JSON.

[!DNL Asset Compute Service] può essere esteso mediante la creazione di applicazioni personalizzate basate su  [!DNL Project Firefly]. Queste applicazioni personalizzate sono [!DNL Project Firefly] app headless e eseguono attività quali l&#39;aggiunta di strumenti di conversione personalizzati o la chiamata di API esterne per eseguire operazioni sulle immagini.

[!DNL Project Firefly] è un framework per creare e distribuire applicazioni Web personalizzate in  [!DNL Adobe I/O] fase di esecuzione. Per creare applicazioni personalizzate, gli sviluppatori possono utilizzare [!DNL React Spectrum] ( toolkit dell&#39;interfaccia utente del Adobe), creare microservizi, creare eventi personalizzati e orchestrare API. Vedere la [documentazione del progetto Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

La base su cui si basa l&#39;architettura comprende:

* La modularità delle applicazioni, che contiene solo ciò che è necessario per una determinata attività, consente di separare le applicazioni e di mantenerle leggere.

* Il concetto di Adobe I/O Runtime senza server offre numerosi vantaggi: elaborazione asincrona, altamente scalabile, isolata e basata su processo, ideale per l’elaborazione delle risorse.

* L&#39;archiviazione cloud binaria fornisce le funzionalità necessarie per memorizzare e accedere singolarmente ai file di risorse e alle rappresentazioni, senza richiedere autorizzazioni di accesso completo all&#39;archivio, utilizzando riferimenti URL pre-firmati. L&#39;accelerazione del trasferimento, il caching CDN e la co-localizzazione di applicazioni di elaborazione con l&#39;archiviazione cloud consentono un accesso ottimale ai contenuti a bassa latenza. Sono supportati sia i cloud AWS che Azure.

![Architettura del servizio  Asset compute](assets/architecture-diagram.png)

*Figura: Architettura  [!DNL Asset Compute Service] e modalità di integrazione con le applicazioni di  [!DNL Experience Manager]storage ed elaborazione.*

L&#39;architettura è costituita dalle seguenti parti:

* **Un** livello API e orchestrazione riceve richieste (in formato JSON) che informano il servizio di trasformare una risorsa sorgente in più rappresentazioni. Queste richieste sono asincrone e restituiscono un ID di attivazione (detto anche &quot;ID processo&quot;). Le istruzioni sono puramente dichiarative e per tutti i lavori di elaborazione standard (ad es. generazione di miniature, estrazione di testo) i consumatori specificano solo il risultato desiderato, ma non le applicazioni che gestiscono determinate rappresentazioni. Le funzionalità API generiche, come autenticazione, analisi, limitazione della tariffa, vengono gestite tramite il gateway API di Adobe  davanti al servizio e gestisce tutte le richieste che vanno al runtime di I/O. Il routing dell&#39;applicazione viene eseguito in modo dinamico dal livello di orchestrazione. I client possono specificare applicazioni personalizzate per rappresentazioni specifiche e includere parametri personalizzati. L&#39;esecuzione dell&#39;applicazione può essere completamente parallelizzata in quanto sono funzioni diverse senza server in I/O Runtime.

* **Applicazioni per l’elaborazione di** risorse specializzate in determinati tipi di formati di file o rappresentazioni di destinazione. Concettualmente, un&#39;applicazione è come il concetto di tubo di Unix: un file di input viene trasformato in uno o più file di output.

* **Una  [comune ](https://github.com/adobe/asset-compute-sdk)** libreria di applicazioni gestisce le attività più comuni come il download del file sorgente, il caricamento delle rappresentazioni, la segnalazione degli errori, l&#39;invio e il monitoraggio degli eventi. Questo è progettato in modo che lo sviluppo di un&#39;applicazione rimanga il più semplice possibile, seguendo l&#39;idea senza server, e può essere limitato alle interazioni locali del filesystem.

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
