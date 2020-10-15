---
title: Impostare l'ambiente di sviluppo richiesto per [!DNL Asset Compute Service].
description: Configurazione dell'ambiente [!DNL Asset Compute Service] per sviluppatori per iniziare a creare e testare codice personalizzato.
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '376'
ht-degree: 3%

---


# Impostazione di un ambiente di sviluppo {#create-dev-environment}

Per creare una configurazione che consenta di sviluppare [!DNL Asset Compute Service], segui questi requisiti e istruzioni.

1. [Acquisisci accesso e credenziali](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) per Project Firefly.

1. [Configurare l’ambiente](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) locale e gli strumenti richiesti.

1. Altri strumenti utili per iniziare a sviluppare in modo uniforme sono:

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (da v10 a v12 LTS, non sono consigliate versioni dispari) e [NPM](https://www.npmjs.com). Utente di OSX HomeBrew può fare `brew install node` per installare entrambi. In caso contrario, scaricatelo dalla pagina [di download](https://nodejs.org/it/)NodeJS.
   * Un IDE adatto a NodeJS, si consiglia di utilizzare il codice [Visual Studio (VS Code)](https://code.visualstudio.com) in quanto è l&#39;IDE supportato per il debugger. È possibile utilizzare qualsiasi altro IDE come editor di codice, ma l&#39;utilizzo avanzato (ad es. debugger) non è ancora supportato.
   * [CLI](https://github.com/adobe/aio-cli) dell&#39;AIO (`aio`) - installare utilizzando `npm install -g @adobe/aio-cli`.

1. Assicuratevi di soddisfare i [prerequisiti](/help/understand-extensibility.md#prerequisites-and-provisioning).

## Impostazione di un progetto Firefly {#create-firefly-project}

1. Accedere al ruolo di amministratore di sistema o sviluppatore nell&#39;organizzazione dell&#39;esperienza. Può essere impostato da un amministratore di sistema nel Admin Console [](https://adminconsole.adobe.com/overview).

1. Accedete alla [console](https://console.adobe.io/)Sviluppatore di Adobe. Assicurati di far parte della stessa organizzazione Adobe Experience Cloud AEM come integrazione Cloud Service. Per ulteriori informazioni  console per sviluppatori di Adobe, consulta la documentazione [della](https://www.adobe.io/apis/experienceplatform/console/docs.html)console.

1. [Crea un progetto](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)Firefly. Fate clic su **[!UICONTROL Crea nuovo progetto]** > **[!UICONTROL Progetto dal modello]**. Selezionare lucciola. Crea un nuovo progetto Firefly con due aree di lavoro: `Production` e `Stage`. Se necessario, potete aggiungere altre aree di lavoro, ad esempio `Development`.

1. Nel progetto Firefly, selezionate un’area di lavoro e iscrivetevi ai servizi necessari per il calcolo delle risorse. Fate clic su **Aggiungi a progetto** > **API** e aggiungete `Asset Compute`, `IO Events`e `IO Events Management` servizi. Quando si aggiunge la prima API, viene richiesto di creare una chiave privata. Salvate queste informazioni sul computer in base alle esigenze di questa chiave per testare l&#39;applicazione personalizzata con lo strumento di sviluppo.

## Next step {#next-step}

Ora che l&#39;ambiente è configurato, è possibile [creare un&#39;applicazione](develop-custom-application.md)personalizzata.

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
