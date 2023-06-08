---
title: Imposta l’ambiente di sviluppo richiesto per [!DNL Asset Compute Service]
description: Configurazione dell’ambiente di sviluppo per [!DNL Asset Compute Service] per iniziare a creare e testare il codice personalizzato.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 3%

---

# Configurare un ambiente per sviluppatori {#create-dev-environment}

Per creare una configurazione che consente di sviluppare per [!DNL Asset Compute Service], attieniti ai requisiti e alle istruzioni seguenti.

1. [Acquisire accesso e credenziali](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) per [!DNL Adobe Developer App Builder].

1. [Configurare l’ambiente locale](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) e gli strumenti richiesti.

1. Altri strumenti utili per iniziare a sviluppare in modo fluido sono:

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, le versioni dispari non sono consigliate) e [NPM](https://www.npmjs.com). L&#39;utente di OSX HomeBrew può fare `brew install node` per installare entrambi. In caso contrario, scaricalo da [Pagina di download di NodeJS](https://nodejs.org/it/)
   * Un IDE adatto a NodeJS, consigliamo [Codice Visual Studio (codice VS)](https://code.visualstudio.com) in quanto è l’IDE supportato per il debugger. Puoi utilizzare qualsiasi altro IDE come editor di codice, ma l’utilizzo avanzato (ad esempio, debugger) non è ancora supportato
   * Installa la versione più recente[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Assicurati di soddisfare i requisiti [prerequisiti](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Configurare un progetto App Builder {#create-App-Builder-project}

1. Assicura il ruolo di amministratore di sistema o sviluppatore nella [!DNL Experience Cloud] organizzazione. Viene configurato da un amministratore di sistema in [Admin Console](https://adminconsole.adobe.com/overview).

1. Accedi a [Console Adobe Developer](https://console.adobe.io/). Assicurati di fare parte della stessa [!DNL Experience Cloud] organizzazione come [!DNL Experience Manager] as a [!DNL Cloud Service] integrazione. Per ulteriori informazioni sulla console Adobe Developer, consulta [Documentazione della console](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Creare un progetto App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Clic **[!UICONTROL Crea nuovo progetto]** > **[!UICONTROL Progetto da modello]**. Seleziona App Builder. Crea un nuovo progetto App Builder con due aree di lavoro: `Production` e `Stage`. Aggiungi altre aree di lavoro, ad esempio `Development`, come richiesto.

1. Nel progetto App Builder, seleziona un’area di lavoro e abbonati ai servizi necessari, ad Asset compute. Clic **Aggiungi al progetto** > **API** e aggiungi `Asset Compute`, `IO Events`, e `IO Events Management` servizi. Quando si aggiunge la prima API, viene richiesto di creare una chiave privata. Salva queste informazioni sul computer quando necessario per testare l&#39;applicazione personalizzata con lo strumento per sviluppatori.

## Passaggio successivo {#next-step}

Ora che l’ambiente è configurato, è possibile: [creare un&#39;applicazione personalizzata](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
