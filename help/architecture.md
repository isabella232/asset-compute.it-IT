---
title: Architettura [!DNL Asset Compute Service]
description: Come [!DNL Asset Compute Service] API, applicazioni e SDK collaborano per fornire un servizio di elaborazione delle risorse nativo per il cloud.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: 0c5ab8ab230e3f033b804849a2c005351542155f
workflow-type: tm+mt
source-wordcount: '486'
ht-degree: 0%

---

# Architettura [!DNL Asset Compute Service] {#overview}

La [!DNL Asset Compute Service] è basato su server [!DNL Adobe I/O] Piattaforma runtime. Fornisce il supporto per le risorse dei servizi di contenuti Adobe Sensei. Client di chiamata (solo [!DNL Experience Manager] come [!DNL Cloud Service] è supportato) viene fornito con le informazioni generate da Adobe Sensei che cercava per la risorsa. Le informazioni restituite sono in formato JSON.

[!DNL Asset Compute Service] è estendibile creando applicazioni personalizzate basate su [!DNL Project Adobe Developer App Builder]. Queste applicazioni personalizzate [!DNL Project Adobe Developer App Builder] le app headless e eseguono attività quali l’aggiunta di strumenti di conversione personalizzati o la chiamata di API esterne per eseguire operazioni sulle immagini.

[!DNL Project Adobe Developer App Builder] è un framework per la creazione e la distribuzione di applicazioni web personalizzate su [!DNL Adobe I/O] runtime. Per creare applicazioni personalizzate, gli sviluppatori possono sfruttare [!DNL React Spectrum] (toolkit per l’interfaccia utente di Adobe), crea microservizi, crea eventi personalizzati e orchestra API. Vedi [documentazione di Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).

La base su cui si basa l’architettura include:

* La modularità delle applicazioni - che contiene solo ciò che è necessario per una determinata attività - consente di separare le applicazioni tra loro e di mantenerle leggere.

* Concetto senza server di [!DNL Adobe I/O] Il runtime offre numerosi vantaggi: elaborazione asincrona, altamente scalabile, isolata e basata su processi, ideale per l’elaborazione delle risorse.

* L’archiviazione cloud binaria fornisce le funzioni necessarie per memorizzare e accedere ai file di risorse e alle rappresentazioni singolarmente, senza richiedere autorizzazioni di accesso completo all’archiviazione, utilizzando riferimenti URL pre-firmati. L&#39;accelerazione del trasferimento, la memorizzazione in cache CDN e la co-localizzazione di applicazioni di elaborazione con l&#39;archiviazione cloud consentono un accesso ottimale ai contenuti a bassa latenza. Sono supportati sia i cloud AWS che Azure.

![Architettura del servizio Asset compute](assets/architecture-diagram.png)

*Figura: Architettura [!DNL Asset Compute Service] e il modo in cui si integra con [!DNL Experience Manager], archiviazione ed elaborazione delle applicazioni.*

L&#39;architettura è costituita dalle seguenti parti:

* **Un livello API e orchestrazione** riceve richieste (in formato JSON) che istruiscono il servizio a trasformare una risorsa sorgente in più rappresentazioni. Le richieste sono asincrone e vengono restituite con un ID di attivazione, che è l’ID del processo. Le istruzioni sono puramente dichiarative e per tutti i lavori di elaborazione standard (ad esempio generazione di miniature, estrazione di testo) i consumatori specificano solo il risultato desiderato, ma non le applicazioni che gestiscono determinate rappresentazioni. Le funzionalità API generiche, come l’autenticazione, l’analisi, le limitazioni delle tariffe, vengono gestite tramite l’Adobe API Gateway di fronte al servizio e gestiscono tutte le richieste che arrivano a [!DNL Adobe I/O] Runtime. Il routing dell&#39;applicazione viene eseguito dinamicamente dal livello di orchestrazione. I client possono specificare applicazioni personalizzate per rappresentazioni specifiche e includere parametri personalizzati. L&#39;esecuzione dell&#39;applicazione può essere completamente parallelizzata in quanto sono funzioni diverse senza server in [!DNL Adobe I/O] Runtime.

* **Applicazioni per elaborare le risorse** che si specializzano in determinati tipi di formati di file o rendering di destinazione. Concettualmente, un&#39;applicazione è come il concetto di tubo Unix: un file di input viene trasformato in uno o più file di output.

* **A [libreria applicazioni comune](https://github.com/adobe/asset-compute-sdk)** gestisce attività comuni come il download del file sorgente, il caricamento dei rendering, la segnalazione degli errori, l’invio degli eventi e il monitoraggio . Questo è progettato in modo che lo sviluppo di un&#39;applicazione rimanga il più semplice possibile, seguendo l&#39;idea senza server, e può essere limitato alle interazioni locali del filesystem.

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
