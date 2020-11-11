---
title: Introduzione alla [!DNL Asset Compute Service].
description: '[!DNL Asset Compute Service] è un servizio di elaborazione delle risorse nativo del cloud che riduce la complessità e migliora la scalabilità.'
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '321'
ht-degree: 0%

---


# Panoramica [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] è un servizio scalabile ed estensibile di [!DNL Adobe Experience Cloud] elaborazione delle risorse digitali. Può trasformare immagini, video, documenti e altri formati di file in diverse rappresentazioni, come miniature, testo estratto e metadati, nonché archivi.

Gli sviluppatori possono collegare applicazioni di risorse personalizzate (o &quot;lavoratori personalizzati&quot;) per risolvere casi di utilizzo personalizzati. Il servizio funziona in fase di [!DNL Adobe I/O] esecuzione. È estensibile tramite app [!DNL Project Firefly] headless scritte in Node.js. che possono eseguire operazioni personalizzate, ad esempio la chiamata di API esterne per eseguire operazioni sulle immagini o il [!DNL Adobe Sensei] supporto.

[!DNL Project Firefly] è un framework per creare e distribuire applicazioni Web personalizzate in fase di [!DNL Adobe I/O] esecuzione per estendere le soluzioni Adobe Experience Cloud. Per creare applicazioni personalizzate, gli sviluppatori possono utilizzare [!DNL React Spectrum] ( toolkit dell&#39;interfaccia utente del Adobe), creare microservizi, creare eventi personalizzati e orchestrare API. Consulta [la documentazione di Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

>[!NOTE]
>
>Al momento, [!DNL Asset Compute Service] è possibile utilizzare il modulo solo [!DNL Experience Manager] come Cloud Service. Gli amministratori creano profili di elaborazione che possono chiamare il [!DNL Asset Compute Service] gruppo per passare le risorse per l’elaborazione. See [use asset microservices and processing profiles](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Casi di utilizzo supportati di [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] supporta alcuni casi d&#39;uso commerciale comuni, come l&#39;elaborazione di immagini di base;  conversioni specifiche dell&#39;applicazione Adobe; e la creazione di applicazioni personalizzate che gestiscano i complessi requisiti aziendali.

Potete utilizzare [!DNL Asset Compute] il servizio Web per generare miniature per tipi di file diversi, rappresentazioni di immagini di alta qualità per i formati [file](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)supportati. Consultate [casi di utilizzo supportati tramite configurazione](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)personalizzata.

>[!NOTE]
>
>Il servizio non fornisce l&#39;archiviazione delle risorse. Gli utenti forniscono i riferimenti alle posizioni dei file di origine e di rappresentazione nell&#39;archiviazione cloud.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use Adobe I/O Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Panoramica dell’elaborazione delle risorse con i microservizi delle risorse in Adobe Experience Manager come Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Documentazione del progetto Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [Formati di file supportati per l&#39;elaborazione](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).
>* [Note sulla versione del servizio di Asset compute](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
