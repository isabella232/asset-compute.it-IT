---
title: Informazioni sull'estensione [!DNL Asset Compute Service]
description: Quando e come estendere la funzionalità [!DNL Asset Compute Service] per eseguire l’elaborazione delle risorse personalizzata.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '254'
ht-degree: 1%

---

# Introduzione all’estensibilità {#introduction-to-extensibilty}

Molti requisiti di rendering, come la conversione in formati e il ridimensionamento delle immagini, sono risolti da [Profili di elaborazione in [!DNL Experience Manager] come a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). I requisiti aziendali più complessi possono richiedere una soluzione personalizzata adatta alle esigenze di un&#39;organizzazione. [!DNL Asset Compute Service] può essere esteso creando applicazioni personalizzate richiamate da Profili elaborazione in  [!DNL Experience Manager]. Queste applicazioni personalizzate soddisfano i [casi d&#39;uso supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l’utilizzo con  [!DNL Experience Manager] come  [!DNL Cloud Service].

Le applicazioni personalizzate sono applicazioni headless [Project Firefly](https://github.com/AdobeDocs/project-firefly). L&#39;estensione di [!DNL Asset Compute Service] con applicazioni personalizzate viene resa semplice tramite gli strumenti per sviluppatori [Asset compute SDK](https://github.com/adobe/asset-compute-sdk) e Project Firefly. Questo consente agli sviluppatori di concentrarsi sulla logica di business. La creazione di applicazioni personalizzate è semplice come la creazione di un&#39;azione runtime senza server semplice [!DNL Adobe I/O]. È una singola funzione JavaScript Node.js. L&#39; [esempio di applicazione personalizzata di base](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo illustra.

## Prerequisiti e requisiti di provisioning {#prerequisites-and-provisioning}

Verifica di soddisfare i seguenti prerequisiti:

* Gli strumenti di Project Firefly sono installati sul computer.
* Un&#39;organizzazione [!DNL Experience Cloud]. Ulteriori informazioni [qui](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials).
* L’organizzazione dell’esperienza deve avere [!DNL Experience Manager] come [!DNL Cloud Service] abilitato.
* [!DNL Adobe Experience Cloud] l’organizzazione fa parte del programma di anteprima per  [!DNL Project Firefly] sviluppatori. Vedere [come richiedere l&#39;accesso](https://www.adobe.io/project-firefly/docs/overview/getting_access/).
* Assicurati i permessi di amministratore o ruolo sviluppatore nell&#39;organizzazione per lo sviluppatore.
* Assicurati che [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) sia installato localmente.

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
