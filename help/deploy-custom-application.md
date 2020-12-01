---
title: Implementare l'applicazione personalizzata [!DNL Asset Compute Service] .
description: Implementare l'applicazione personalizzata [!DNL Asset Compute Service] .
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '197'
ht-degree: 0%

---


# Implementare un&#39;applicazione personalizzata {#deploy-custom-application}

Per distribuire l&#39;applicazione, utilizzare il comando [distribuisci app aio](https://github.com/adobe/aio-cli#aio-appdeploy). Nel terminale, il comando visualizza un URL per accedere all&#39;applicazione personalizzata. L&#39;URL è nel formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Per ottenere lo stesso URL senza ridistribuire l&#39;applicazione, utilizzare il comando [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action).

Utilizzate l&#39;URL in un [Profilo di elaborazione in  Experience Manager come Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) per integrare l&#39;applicazione con [!DNL Experience Manager] come Cloud Service.

Assicurati che il progetto Firefly e l&#39;area di lavoro corrispondano a [!DNL Experience Manager] come ambiente di Cloud Service in cui desideri usare l&#39;azione. Dispone di ambienti diversi per lo sviluppo, l&#39;installazione e la produzione. È possibile verificare l&#39;ambiente controllando le credenziali `AIO_runtime_*` definite all&#39;interno del file ENV nella radice dell&#39;applicazione Firefly. Ad esempio, per eseguire la distribuzione in un&#39;area di lavoro `Stage`, `AIO_runtime_namespace` è nel formato `xxxxxx_xxxxxxxxx_stage`. Per l&#39;integrazione con [!DNL Experience Manager] come ambiente di produzione Cloud Service, utilizzate gli URL dell&#39;applicazione dall&#39;area di lavoro di Firefly `Production`.

>[!CAUTION]
>
>Non utilizzate un&#39;area di lavoro personale in ambienti [!DNL Experience Manager] critici.

>[!MORELIKETHIS]
>
>* [Comprendere e gestire gli ambienti in  Experience Manager come Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

