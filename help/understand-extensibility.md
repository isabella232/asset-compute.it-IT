---
title: Informazioni sull'estensione [!DNL Asset Compute Service].
description: Quando e come estendere la funzionalità  [!DNL Asset Compute Service] per eseguire l'elaborazione delle risorse personalizzata.
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '259'
ht-degree: 1%

---


# Introduzione all&#39;estensibilità {#introduction-to-extensibilty}

Numerosi requisiti di rappresentazione, come la conversione in formati e il ridimensionamento delle immagini, vengono soddisfatti da [Profili di elaborazione in [!DNL Experience Manager] come a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). Requisiti aziendali più complessi possono richiedere una soluzione personalizzata in base alle esigenze di un&#39;organizzazione. [!DNL Asset Compute Service] può essere esteso creando applicazioni personalizzate richiamate da Profili di elaborazione in  [!DNL Experience Manager]. Queste applicazioni personalizzate soddisfano i casi di utilizzo [supportati](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l&#39;uso con  [!DNL Experience Manager] come  [!DNL Cloud Service].

Le applicazioni personalizzate sono app [Project Firefly](https://github.com/AdobeDocs/project-firefly) senza testa. L&#39;estensione di [!DNL Asset Compute Service] con le applicazioni personalizzate è resa semplice tramite l&#39;SDK [ Asset compute](https://github.com/adobe/asset-compute-sdk) e gli strumenti per sviluppatori Project Firefly. Questo consente agli sviluppatori di concentrarsi sulla logica di business. La creazione di applicazioni personalizzate è semplice come la creazione di un&#39;azione runtime senza server semplici [!DNL Adobe I/O]. È una funzione JavaScript Node.js singola. L&#39;esempio [di base dell&#39;applicazione personalizzata](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) ne illustra l&#39;esempio.

## Prerequisiti e requisiti di provisioning {#prerequisites-and-provisioning}

Assicuratevi di soddisfare i seguenti prerequisiti:

* Gli strumenti di Project Firefly sono installati nel computer.
* Un&#39;organizzazione [!DNL Experience Cloud]. Ulteriori informazioni [qui](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* L&#39;organizzazione dell&#39;esperienza deve avere [!DNL Experience Manager] abilitato come [!DNL Cloud Service].
* [!DNL Adobe Experience Cloud] l&#39;organizzazione fa parte del programma di anteprima  [!DNL Project Firefly] per sviluppatori. Vedere [come richiedere l&#39;accesso](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
* Assicuratevi che lo sviluppatore abbia il ruolo di sviluppatore o le autorizzazioni di amministratore nell&#39;organizzazione.
* Assicurarsi che [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) sia installato localmente.

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
