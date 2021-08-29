---
title: Imposta l'ambiente di sviluppo richiesto per [!DNL Asset Compute Service]
description: Configurazione dell'ambiente per sviluppatori per [!DNL Asset Compute Service] per iniziare a creare e testare il codice personalizzato.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 9404ffcc66a3b6ba206155d1b1a5c16a43e22a39
workflow-type: tm+mt
source-wordcount: '370'
ht-degree: 3%

---

# Configurare un ambiente di sviluppo {#create-dev-environment}

Per creare una configurazione che ti consenta di sviluppare per [!DNL Asset Compute Service], segui questi requisiti e istruzioni.

1. [Acquisisci ](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) credenziali e accesso per  [!DNL Project Firefly].

1. [Imposta l&#39;](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) ambiente locale e gli strumenti necessari.

1. Altri strumenti che ti aiutano a iniziare lo sviluppo senza problemi sono:

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org)  (da v10 a v12 LTS, non sono consigliate versioni dispari) e  [NPM](https://www.npmjs.com). L&#39;utente di OSX HomeBrew può fare `brew install node` per installare entrambi. In caso contrario, scaricalo dalla [pagina di download di NodeJS](https://nodejs.org/it/).
   * Un IDE adatto a NodeJS, si consiglia [Codice di Visual Studio (Codice VS)](https://code.visualstudio.com) in quanto è l’IDE supportato per il debugger. Puoi utilizzare qualsiasi altro IDE come editor di codice, ma l’utilizzo avanzato (ad esempio, debugger) non è ancora supportato.
   * [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`) - installa utilizzando  `npm install -g @adobe/aio-cli@7.1.0`.

1. Assicurati di soddisfare i [prerequisiti](/help/understand-extensibility.md#prerequisites-and-provisioning).

>[!NOTE]
>
>Per il momento, utilizza [!DNL Adobe I/O] CLI v7.1.0 di e non utilizza [!DNL Adobe I/O] CLI v8.

## Configurare un progetto Firefly {#create-firefly-project}

1. Assicurati il ruolo di amministratore di sistema o sviluppatore nell&#39;organizzazione [!DNL Experience Cloud]. Questo è configurato da un amministratore di sistema nell&#39; [Admin Console](https://adminconsole.adobe.com/overview).

1. Accedi a [Adobe Developer Console](https://console.adobe.io/). Assicurati di far parte della stessa organizzazione [!DNL Experience Cloud] di [!DNL Experience Manager] come integrazione [!DNL Cloud Service]. Per ulteriori informazioni su Adobe Developer Console, consulta la [documentazione della console](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Crea un progetto](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md) Firefly. Fai clic su **[!UICONTROL Crea nuovo progetto]** > **[!UICONTROL Progetto da modello]**. Selezionare Firefly. Crea un nuovo progetto Firefly con due aree di lavoro: `Production` e `Stage`. Aggiungi altre aree di lavoro, ad esempio `Development`, in base alle esigenze.

1. Nel progetto Firefly, seleziona un’area di lavoro e abbonati ai servizi necessari per Asset compute. Fai clic su **Aggiungi al progetto** > **API** e aggiungi i servizi `Asset Compute`, `IO Events` e `IO Events Management` . Quando si aggiunge la prima API, viene richiesto di creare una chiave privata. Salvare queste informazioni sul computer in quanto è necessaria questa chiave per testare l’applicazione personalizzata con lo strumento sviluppatore.

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
