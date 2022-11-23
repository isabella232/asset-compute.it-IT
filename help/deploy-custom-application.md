---
title: Distribuzione [!DNL Asset Compute Service] applicazione personalizzata
description: Distribuzione [!DNL Asset Compute Service] applicazione personalizzata.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 129651ba432b75703bc27baa7081da60302f828d
workflow-type: tm+mt
source-wordcount: '184'
ht-degree: 3%

---

# Distribuire un&#39;applicazione personalizzata {#deploy-custom-application}

Per distribuire l&#39;applicazione, utilizza [distribuzione app aio](https://github.com/adobe/aio-cli#aio-appdeploy) comando. Nel terminale, il comando visualizza un URL per accedere all&#39;applicazione personalizzata. L’URL è nel formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Per ottenere lo stesso URL senza ridistribuire l&#39;applicazione, utilizza [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) comando.

Utilizza l’URL in un [Profilo di elaborazione in [!DNL Experience Manager] come [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=it) per integrare l&#39;applicazione con [!DNL Experience Manager] come [!DNL Cloud Service].

Assicurati che il progetto e l’area di lavoro di App Builder corrispondano a [!DNL Experience Manager] come [!DNL Cloud Service] ambiente in cui desideri utilizzare l’azione. Dispone di ambienti diversi per lo sviluppo, la gestione temporanea e la produzione. Puoi verificare l’ambiente controllando `AIO_runtime_*` credenziali definite all&#39;interno del file ENV nella directory principale dell&#39;applicazione Firefly. Ad esempio, per distribuire in un `Stage` l&#39;area di lavoro `AIO_runtime_namespace` è del formato `xxxxxx_xxxxxxxxx_stage`. Per integrare con [!DNL Experience Manager] come [!DNL Cloud Service] Ambiente di produzione, utilizza gli URL dell&#39;applicazione da Firefly `Production` workspace.

>[!CAUTION]
>
>Non utilizzare un&#39;area di lavoro personale in ambienti critici [!DNL Experience Manager] ambienti.

>[!MORELIKETHIS]
>
>* [Comprendere e gestire gli ambienti in [!DNL Experience Manager] come [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

