---
title: Informazioni sull’estensione [!DNL Asset Compute Service]
description: Quando e come estendere il [!DNL Asset Compute Service] funzionalità per l’elaborazione personalizzata delle risorse.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 6%

---

# Introduzione all’estensibilità {#introduction-to-extensibilty}

Molti requisiti di rendering, come la conversione in formati e il ridimensionamento delle immagini, sono risolti da [Profili di elaborazione in [!DNL Experience Manager] come [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=it). I requisiti aziendali più complessi possono richiedere una soluzione personalizzata adatta alle esigenze di un&#39;organizzazione. [!DNL Asset Compute Service] può essere esteso creando applicazioni personalizzate richiamate da Profili elaborazione in [!DNL Experience Manager]. Queste applicazioni personalizzate rispondono alle [casi di utilizzo supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=it).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo con [!DNL Experience Manager] come [!DNL Cloud Service].

Le applicazioni personalizzate sono headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) app. Estensione [!DNL Asset Compute Service] con le applicazioni personalizzate è possibile semplificare le [asset compute SDK](https://github.com/adobe/asset-compute-sdk) e strumenti per sviluppatori di Adobe Developer App Builder. Questo consente agli sviluppatori di concentrarsi sulla logica di business. La creazione di applicazioni personalizzate è semplice come la creazione di un server semplice [!DNL Adobe I/O] Azione runtime. È una singola funzione JavaScript Node.js. La [esempio di applicazione personalizzata di base](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo illustra.

## Prerequisiti e requisiti di provisioning {#prerequisites-and-provisioning}

Verifica di soddisfare i seguenti prerequisiti:

* Gli strumenti di Adobe Developer App Builder sono installati nel computer.
* Un [!DNL Experience Cloud] organizzazione. Ulteriori informazioni [qui](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* L’organizzazione dell’esperienza deve avere [!DNL Experience Manager] come [!DNL Cloud Service] abilitato.
* [!DNL Adobe Experience Cloud] l&#39;organizzazione fa parte del [!DNL Adobe Developer App Builder] programma di anteprima per sviluppatori. Vedi [come richiedere l&#39;accesso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Assicurati i permessi di amministratore o ruolo sviluppatore nell&#39;organizzazione per lo sviluppatore.
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
