---
title: Note sulla versione di [!DNL Asset Compute Service].
description: Nuove funzioni, miglioramenti e problemi noti in [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '193'
ht-degree: 0%

---


# Note sulla versione [!DNL Asset Compute Service] {#release-notes}

L&#39;ultima versione di [!DNL Asset Compute Service] è rilasciata il 30 luglio 2020.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## What is new {#what-is-new}

Questa è la prima versione di [!DNL Asset Compute Service]. È un servizio scalabile ed estensibile di [!DNL Adobe Experience Cloud] elaborazione delle risorse digitali. Può trasformare immagini, video, documenti e altri formati di file in diverse rappresentazioni, come miniature, testo estratto e metadati, nonché archivi.

Al momento, [!DNL Asset Compute Service] è possibile utilizzare [!DNL Experience Manager] come Cloud Service solo l&#39;oggetto.

## Limitazioni e problemi noti {#known-limitations}

Per testare l’applicazione personalizzata con lo strumento [](https://github.com/adobe/asset-compute-devtool)di sviluppo, è necessario accedere a un contenitore [di archiviazione](https://github.com/adobe/asset-compute-devtool#prerequisites)cloud.

* L&#39;accesso all&#39;archiviazione cloud (diverso da quello [!DNL Experience Manager] del BLOB) è necessario solo per lo strumento di sviluppo. Potete comunque creare, testare e distribuire applicazioni personalizzate senza lo strumento di sviluppo.
* Può essere un contenitore condiviso utilizzato da più sviluppatori in diversi progetti.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] l&#39;estensibilità è sviluppata nell&#39;ambito di un modello di sviluppo aperto su [github.com/adobe](https://github.com/adobe) che accoglie i contributi degli sviluppatori di estensioni. Tutti i componenti rilevanti per lo sviluppo, la creazione, il test e la distribuzione di applicazioni personalizzate sono open source. Scopri [come e dove contribuire al servizio](contribute-to-compute-service.md)di elaborazione.

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
