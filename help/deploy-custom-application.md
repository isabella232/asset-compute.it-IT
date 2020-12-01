---
title: Implementare l'applicazione personalizzata [!DNL Asset Compute Service] .
description: Implementare l'applicazione personalizzata [!DNL Asset Compute Service] .
translation-type: tm+mt
source-git-commit: 78c1246f5fc42006013701a6cf4d375a1d8c9fd8
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 0%

---


# Implementare un&#39;applicazione personalizzata {#deploy-custom-application}

Per distribuire l&#39;applicazione, utilizzare il comando [distribuisci app aio](https://github.com/adobe/aio-cli#aio-appdeploy). Nel terminale, il comando visualizza un URL per accedere all&#39;applicazione personalizzata. L&#39;URL è nel formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Per ottenere lo stesso URL senza ridistribuire l&#39;applicazione, utilizzare il comando [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action).

Utilizzate l&#39;URL in un [Profilo di elaborazione in [!DNL Experience Manager] come  [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) per integrare l&#39;applicazione con [!DNL Experience Manager] come [!DNL Cloud Service].

Verificate che il progetto Firefly e l&#39;area di lavoro corrispondano a [!DNL Experience Manager] come ambiente [!DNL Cloud Service] in cui desiderate utilizzare l&#39;azione. Dispone di ambienti diversi per lo sviluppo, l&#39;installazione e la produzione. È possibile verificare l&#39;ambiente controllando le credenziali `AIO_runtime_*` definite all&#39;interno del file ENV nella radice dell&#39;applicazione Firefly. Ad esempio, per eseguire la distribuzione in un&#39;area di lavoro `Stage`, `AIO_runtime_namespace` è nel formato `xxxxxx_xxxxxxxxx_stage`. Per effettuare l&#39;integrazione con [!DNL Experience Manager] come ambiente di produzione [!DNL Cloud Service], utilizzate gli URL dell&#39;applicazione dall&#39;area di lavoro Firefly `Production`.

>[!CAUTION]
>
>Non utilizzate un&#39;area di lavoro personale in ambienti [!DNL Experience Manager] critici.

>[!MORELIKETHIS]
>
>* [Comprendere e gestire gli ambienti  [!DNL Experience Manager] come [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

