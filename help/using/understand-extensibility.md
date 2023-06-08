---
title: Informazioni sull’estensione [!DNL Asset Compute Service]
description: Quando e come estendere [!DNL Asset Compute Service] per eseguire l’elaborazione personalizzata delle risorse.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 6%

---

# Introduzione all’estensibilità {#introduction-to-extensibilty}

Molti requisiti di rendering, come la conversione in formati e il ridimensionamento delle immagini, sono soddisfatti da [Profili elaborazione in [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=it). I requisiti aziendali più complessi possono richiedere una soluzione personalizzata che soddisfi le esigenze di un&#39;organizzazione. [!DNL Asset Compute Service] possono essere estese creando applicazioni personalizzate chiamate dai profili di elaborazione in [!DNL Experience Manager]. Queste applicazioni personalizzate soddisfano i [casi d’uso supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=it).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l&#39;uso con [!DNL Experience Manager] as a [!DNL Cloud Service].

Le applicazioni personalizzate sono headless [Generatore di app Adobe Developer](https://github.com/AdobeDocs/app-builder) app. Estensione [!DNL Asset Compute Service] con le applicazioni personalizzate viene semplificata [SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk) e gli strumenti per gli sviluppatori di Adobe Developer App Builder. Questo consente agli sviluppatori di concentrarsi sulla logica di business. La creazione di applicazioni personalizzate è semplice come la creazione di un server semplice [!DNL Adobe I/O] Azione runtime. È una singola funzione JavaScript di Node.js. Il [esempio di applicazione personalizzata di base](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) La illustra.

## Prerequisiti e requisiti di provisioning {#prerequisites-and-provisioning}

Assicurati di soddisfare i seguenti prerequisiti:

* Gli strumenti di Adobe Developer App Builder sono installati nel computer.
* Un [!DNL Experience Cloud] organizzazione. Ulteriori informazioni [qui](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* L’organizzazione dell’esperienza deve disporre di [!DNL Experience Manager] as a [!DNL Cloud Service] abilitato.
* [!DNL Adobe Experience Cloud] l&#39;organizzazione fa parte del [!DNL Adobe Developer App Builder] programma di anteprima per sviluppatori. Consulta [come richiedere l’accesso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Assicurati che lo sviluppatore disponga del ruolo di sviluppatore o delle autorizzazioni di amministratore nell’organizzazione.
* Assicurati che [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) è installato localmente.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
