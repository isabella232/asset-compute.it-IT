---
title: Imposta l'ambiente di sviluppo necessario per [!DNL Asset Compute Service]
description: Configurazione dell’ambiente per sviluppatori per [!DNL Asset Compute Service] per iniziare a creare e testare il codice personalizzato.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: a50a3bdb520cbe608c5710716df80ac6e3b486e5
workflow-type: tm+mt
source-wordcount: '364'
ht-degree: 3%

---

# Configurare un ambiente di sviluppo {#create-dev-environment}

Per creare una configurazione che consente di sviluppare [!DNL Asset Compute Service], seguire questi requisiti e istruzioni.

1. [Acquisire accesso e credenziali](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) per [!DNL Project Firefly].

1. [Configurare l’ambiente locale](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) e gli strumenti necessari.

1. Altri strumenti che ti aiutano a iniziare lo sviluppo senza problemi sono:

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (versioni da v12 a v14 LTS, non consigliate versioni dispari) e [NPM](https://www.npmjs.com). Utente di OSX HomeBrew può fare `brew install node` per installare entrambi. In caso contrario, scaricalo dal [Pagina di download di NodeJS](https://nodejs.org/it/)
   * Un IDE adatto a NodeJS, ti consigliamo [Codice di Visual Studio (codice VS)](https://code.visualstudio.com) poiché è l’IDE supportato per il debugger. Puoi utilizzare qualsiasi altro IDE come editor di codice, ma l’utilizzo avanzato (ad esempio, debugger) non è ancora supportato
   * Installa la versione più recente[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Assicurati di soddisfare le [prerequisiti](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Configurare un progetto App Builder {#create-App-Builder-project}

1. Assicurati il ruolo di amministratore di sistema o sviluppatore nel [!DNL Experience Cloud] organizzazione. Questo è configurato da un amministratore di sistema nel [Admin Console](https://adminconsole.adobe.com/overview).

1. Accedi al [Console Adobe Developer](https://console.adobe.io/). Assicurati di fare parte della stessa [!DNL Experience Cloud] come organizzazione [!DNL Experience Manager] come [!DNL Cloud Service] integrazione. Per ulteriori informazioni sulla console Adobe Developer, consulta [Documentazione della console](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Creare un progetto App Builder](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Fai clic su **[!UICONTROL Crea nuovo progetto]** > **[!UICONTROL Progetto da modello]**. Seleziona App Builder. Crea un nuovo progetto App Builder con due aree di lavoro: `Production` e `Stage`. Aggiungi altre aree di lavoro, ad esempio `Development`, in base alle esigenze.

1. In Progetto App Builder, seleziona un’area di lavoro e abbonati ai servizi necessari, ad Asset compute. Fai clic su **Aggiungi a progetto** > **API** e aggiungere `Asset Compute`, `IO Events`e `IO Events Management` servizi. Quando si aggiunge la prima API, viene richiesto di creare una chiave privata. Salvare queste informazioni sul computer in quanto è necessaria questa chiave per testare l’applicazione personalizzata con lo strumento sviluppatore.

## Passaggio successivo {#next-step}

Ora che il tuo ambiente è configurato, sei pronto a [creare un&#39;applicazione personalizzata](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
