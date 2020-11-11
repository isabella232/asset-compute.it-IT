---
title: Informazioni sull'estensione [!DNL Asset Compute Service].
description: Quando e come estendere [!DNL Asset Compute Service] le funzionalità per l’elaborazione delle risorse personalizzata.
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '271'
ht-degree: 1%

---


# Introduzione all&#39;estensibilità {#introduction-to-extensibilty}

Numerosi requisiti di rappresentazione, come la conversione in formati e il ridimensionamento delle immagini, vengono soddisfatti mediante l’ [elaborazione [!DNL Experience Manager] di profili come Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). Requisiti aziendali più complessi possono richiedere una soluzione personalizzata in base alle esigenze di un&#39;organizzazione. [!DNL Asset Compute Service] può essere esteso creando applicazioni personalizzate richiamate da Profili di elaborazione in [!DNL Experience Manager]. Queste applicazioni personalizzate soddisfano i casi di utilizzo [supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l&#39;uso con [!DNL Experience Manager] come Cloud Service.

Le applicazioni personalizzate sono app [Project Firefly](https://github.com/AdobeDocs/project-firefly) headless. L&#39;estensione [!DNL Asset Compute Service] con applicazioni personalizzate è resa semplice dall&#39; [SDK](https://github.com/adobe/asset-compute-sdk) Asset compute e dagli strumenti per sviluppatori Project Firefly. Questo consente agli sviluppatori di concentrarsi sulla logica di business. La creazione di applicazioni personalizzate è semplice come la creazione di un&#39;azione Adobe I/O Runtime senza server semplice. È una funzione JavaScript Node.js singola. L&#39;esempio [di applicazione personalizzata](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) di base illustra la situazione.

## Prerequisiti e requisiti di provisioning {#prerequisites-and-provisioning}

Assicuratevi di soddisfare i seguenti prerequisiti:

* Gli strumenti di Project Firefly sono installati nel computer.
* Un&#39; [!DNL Experience Cloud] organizzazione. Maggiori informazioni [qui](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* L&#39;organizzazione dell&#39;esperienza deve avere come Cloud Service abilitato [!DNL Experience Manager] .
* [!DNL Adobe Experience Cloud] l&#39;organizzazione fa parte del programma di anteprima [!DNL Project Firefly] per sviluppatori. Scopri [come richiedere l’accesso](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
* Assicuratevi che lo sviluppatore abbia il ruolo di sviluppatore o le autorizzazioni di amministratore nell&#39;organizzazione.
* Assicurarsi che [CLI](https://github.com/adobe/aio-cli) I/O Adobe sia installato localmente.

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
