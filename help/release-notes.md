---
title: Note sulla versione di  [!DNL Asset Compute Service]
description: Nuove funzioni, miglioramenti e problemi noti in [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---


# Note sulla versione di [!DNL Asset Compute Service] {#release-notes}

L&#39;ultima versione di [!DNL Asset Compute Service] è rilasciata il 30 luglio 2020.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## Novità di {#what-is-new}

Questa è la prima versione di [!DNL Asset Compute Service]. È un servizio scalabile ed estensibile di [!DNL Adobe Experience Cloud] per l&#39;elaborazione di risorse digitali. Può trasformare immagini, video, documenti e altri formati di file in diverse rappresentazioni, come miniature, testo estratto e metadati, nonché archivi.

Attualmente, il [!DNL Asset Compute Service] può essere utilizzato solo in [!DNL Experience Manager] come [!DNL Cloud Service].

## Limitazioni e problemi noti {#known-limitations}

Per testare l&#39;applicazione personalizzata con lo [strumento di sviluppo](https://github.com/adobe/asset-compute-devtool), è necessario avere accesso a un [contenitore di archiviazione cloud](https://github.com/adobe/asset-compute-devtool#prerequisites).

* L&#39;accesso all&#39;archiviazione cloud (diverso dall&#39;archivio BLOB [!DNL Experience Manager]) è necessario solo per lo strumento di sviluppo. Potete comunque creare, testare e distribuire applicazioni personalizzate senza lo strumento di sviluppo.
* Può essere un contenitore condiviso utilizzato da più sviluppatori in diversi progetti.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] l&#39;estensibilità è sviluppata nell&#39;ambito di un modello di sviluppo aperto su  [github.com/](https://github.com/adobe) adobeche accoglie i contributi degli sviluppatori di estensioni. Tutti i componenti rilevanti per lo sviluppo, la creazione, il test e la distribuzione di applicazioni personalizzate sono open source. Vedere [come e dove contribuire a Compute Service](contribute-to-compute-service.md).

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
