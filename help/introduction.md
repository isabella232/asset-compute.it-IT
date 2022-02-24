---
title: Introduzione al [!DNL Asset Compute Service]
description: '"[!DNL Asset Compute Service] è un servizio di elaborazione delle risorse nativo per il cloud che riduce la complessità e migliora la scalabilità".'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '307'
ht-degree: 0%

---

# Panoramica [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] è un servizio scalabile ed estensibile di [!DNL Adobe Experience Cloud] per elaborare le risorse digitali. Può trasformare immagini, video, documenti e altri formati di file in diverse rappresentazioni, quali miniature, testo e metadati estratti e archivi.

Gli sviluppatori possono collegare applicazioni per risorse personalizzate (o processi di lavoro personalizzati) per risolvere problemi di utilizzo personalizzati. Il servizio funziona sul [!DNL Adobe I/O] runtime. È estendibile attraverso [!DNL Project Firefly] app headless scritte in Node.js. Possono eseguire operazioni personalizzate, ad esempio richiamare API esterne per eseguire operazioni sulle immagini o sfruttarle [!DNL Adobe Sensei] supporto.

[!DNL Project Firefly] è un framework per la creazione e la distribuzione di applicazioni web personalizzate su [!DNL Adobe I/O] runtime per estendere le soluzioni Adobe Experience Cloud. Per creare applicazioni personalizzate, gli sviluppatori possono sfruttare [!DNL React Spectrum] (toolkit per l’interfaccia utente di Adobe), crea microservizi, crea eventi personalizzati e orchestra API. Vedi [documentazione del progetto Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

>[!NOTE]
>
>Attualmente, il [!DNL Asset Compute Service] può essere utilizzato solo tramite [!DNL Experience Manager] come [!DNL Cloud Service]. Gli amministratori creano profili di elaborazione che possono richiamare [!DNL Asset Compute Service] passare le risorse per l’elaborazione. Vedi [utilizzare i microservizi per le risorse e i profili di elaborazione](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Casi d’uso supportati di [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] supporta alcuni casi d&#39;uso comuni, come l&#39;elaborazione di immagini di base; Adobe di conversioni specifiche dell&#39;applicazione; creazione di applicazioni personalizzate che orchestrano requisiti aziendali complessi.

È possibile utilizzare [!DNL Asset Compute] servizio web per generare miniature per diversi tipi di file, rendering di immagini di alta qualità per [formati di file supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Vedi [casi d’uso supportati dalla configurazione personalizzata](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>Il servizio non fornisce archiviazione delle risorse. Gli utenti forniscono tali informazioni e riferimenti alle posizioni dei file di origine e di rendering nell’archiviazione cloud.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Panoramica dell’elaborazione delle risorse con i microservizi per le risorse in [!DNL Adobe Experience Manager] come [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Documentazione del progetto Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [Formati di file supportati per l’elaborazione](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
