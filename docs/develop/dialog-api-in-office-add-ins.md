---
title: Utiliser l’API de boîte de dialogue Office dans vos compléments Office
description: Découvrez les principes de base de la création d’une boîte de dialogue dans un Office de recherche.
ms.date: 01/28/2021
localization_priority: Normal
ms.openlocfilehash: 878bdeaa6752e37f8d3c67f32b42e2a5a7b962cb
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53349916"
---
# <a name="use-the-office-dialog-api-in-office-add-ins"></a>Utiliser l’API de boîte de dialogue Office dans les compléments Office

Vous pouvez utiliser l’[API de dialogue Office](/javascript/api/office/office.ui) pour ouvrir des boîtes de dialogue dans votre complément Office. Cet article fournit des conseils concernant l’utilisation de l’API de dialogue dans votre complément Office.

> [!NOTE]
> Pour plus d’informations sur les compléments où l’API de dialogue est actuellement prise en charge, consultez la rubrique relative aux [ensembles de conditions requises de l’API de dialogue](../reference/requirement-sets/dialog-api-requirement-sets.md). L’API de dialogue est actuellement prise en charge pour Excel, PowerPoint et Word. Outlook prise en charge est incluse dans différents ensembles de conditions requises de boîte aux lettres, consultez la référence &mdash; d’API pour plus d’informations.

Un scénario principal pour l’API de dialogue consiste à activer l’authentification à l'aide d'une ressource telle que Google, Facebook, ou Microsoft Graph. Pour plus d’informations, voir [S’authentifier auprès de l'API de boîte de dialogue Office](auth-with-office-dialog-api.md) *une fois* que vous êtes familiarisé(e) avec cet article.

Envisagez d’ouvrir une boîte de dialogue à partir d’un volet Office, d’un complément de contenu ou d’un [complément de commande](../design/add-in-commands.md) pour effectuer les opérations suivantes :

- afficher les pages de connexion qui ne peuvent pas être ouvertes directement dans un volet Office ;
- fournir davantage d’espace à l’écran, ou même un plein écran, pour certaines tâches exécutées dans votre complément ;
- héberger une vidéo qui serait trop petite si elle était limitée à un volet Office.

> [!NOTE]
> Comme des éléments d’interface utilisateur qui se chevauchent peuvent gêner des utilisateurs, évitez d’ouvrir une boîte de dialogue à partir d’un volet Office à moins que votre scénario l’exige. Lorsque vous envisagez d’utiliser la surface d’exposition d’un volet Office, tenez compte du fait que les volets Office peuvent être affichés sous forme d’onglets. Pour obtenir un exemple de volet De tâches à onglets, voir l’exemple Excel de [salestracker JavaScript pour le](https://github.com/OfficeDev/Excel-Add-in-JavaScript-SalesTracker) module de développement.

L’image suivante montre un exemple de boîte de dialogue.

![Capture d’écran montrant la boîte de dialogue avec 3 options de personnalisation affichées devant Word.](../images/auth-o-dialog-open.png)

Notez que la boîte de dialogue s’ouvre toujours au centre de l’écran. L’utilisateur peut la déplacer et la redimensionner. La fenêtre *n’est* pasmodale : un utilisateur peut continuer à interagir à la fois avec le document dans l’application Office et avec la page dans le volet Des tâches, s’il en existe un.

## <a name="open-a-dialog-box-from-a-host-page"></a>Ouvrir une boîte de dialogue à partir d’une page hôte

Les API JavaScript Office incluent un objet [Dialog](/javascript/api/office/office.dialog) et deux fonctions dans l’[espace de noms Office.context.ui](/javascript/api/office/office.ui).

Pour ouvrir une boîte de dialogue, généralement une page dans un volet des tâches, votre code appelle la méthode [displayDialogAsync](/javascript/api/office/office.ui) et lui transmet l’URL de la ressource que vous voulez ouvrir. La page sur laquelle cette méthode est appelée est connue sous le nom de « page hôte ». Par exemple, si vous appelez cette méthode dans le script sur index.html d'un volet de tâches, la page index.html correspond à la page hôte de la boîte de dialogue ouverte par la méthode.

La ressource ouverte dans la boîte de dialogue correspond généralement à une page, mais ce peut être une méthode du contrôleur dans une application MVC, un itinéraire, une méthode de service web ou toute autre ressource. Dans cet article, les termes « page » ou « site web » font référence à la ressource dans la boîte de dialogue. Le code suivant est un exemple simple.

```js
Office.context.ui.displayDialogAsync('https://myAddinDomain/myDialog.html');
```

> [!NOTE]
> - L’URL utilise le protocole HTTP **S**. Ceci est obligatoire pour toutes les pages chargées dans une boîte de dialogue, pas seulement la première page chargée.
> - Le domaine de la boîte de dialogue est le même que celui de la page hôte, qui peut être la page d’un volet Office ou le [fichier de fonctions](../reference/manifest/functionfile.md) d’une commande de complément. Obligatoire : la page, la méthode du contrôleur ou toute autre ressource qui est transmise à la méthode `displayDialogAsync` doit se trouver dans le même domaine que la page hôte.

> [!IMPORTANT]
> La page hôte et les ressources s'ouvrant dans la boîte de dialogue doivent avoir le même domaine complet. Si vous tentez de transmettre `displayDialogAsync` à un sous-domaine du domaine du complément, cela ne fonctionnera pas. Le domaine complet et tous les sous-domaines doivent être exactement les mêmes.

Une fois que la première page (ou toute autre ressource) est chargée, un utilisateur peut utiliser des liens ou une autre interface utilisateur pour accéder à n’importe quel site web (ou n’importe quelle autre ressource) qui utilise le protocole HTTPS. Vous pouvez également concevoir la première page de façon à ce que l’utilisateur soit immédiatement redirigé vers un autre site.

Par défaut, la boîte de dialogue occupera 80 % de la hauteur et de la largeur de l’écran de l’appareil, mais vous pouvez définir des pourcentages différents en transmettant un objet de configuration à la méthode, comme indiqué dans l’exemple suivant.

```js
Office.context.ui.displayDialogAsync('https://myDomain/myDialog.html', {height: 30, width: 20});
```

Pour voir un exemple de complément qui effectue ce type d’action, consultez la rubrique relative à l’[exemple d’API de dialogue de complément Office](https://github.com/OfficeDev/Office-Add-in-Dialog-API-Simple-Example). Pour plus d’exemples qui `displayDialogAsync` utilisent, voir [Exemples.](#samples)

Définissez les deux valeurs sur 100 % pour bénéficier d’une réelle d’expérience de plein écran. (Le maximum réel est de 99,5 %, et la fenêtre peut toujours être déplacée et redimensionnée.)

> [!NOTE]
> Vous ne pouvez ouvrir qu’une seule boîte de dialogue à partir d’une fenêtre hôte. Toute tentative d’ouverture d’une autre boîte de dialogue génère une erreur. Par exemple, si un utilisateur ouvre une boîte de dialogue à partir d’un volet Office, il ne peut pas ouvrir une seconde boîte de dialogue à partir d’une autre page dans le volet Office. Toutefois, quand une boîte de dialogue est ouverte à partir d’une [commande de complément](../design/add-in-commands.md), la commande ouvre un nouveau fichier HTML (mais invisible) chaque fois qu’elle est sélectionnée. Cela crée une nouvelle fenêtre hôte (invisible), afin que chaque fenêtre de ce type puisse lancer sa propre boîte de dialogue. Pour plus d’informations, reportez-vous à [Erreurs provenant de displayDialogAsync](dialog-handle-errors-events.md#errors-from-displaydialogasync).

### <a name="take-advantage-of-a-performance-option-in-office-on-the-web"></a>Tirer parti d’une option de performances dans Office sur le web

La propriété `displayInIframe` est une propriété supplémentaire dans l’objet de configuration que vous pouvez transmettre à `displayDialogAsync`. Lorsque cette propriété est définie sur `true` et que le complément est en cours d’exécution dans un document ouvert dans Office sur le web, la boîte de dialogue s’ouvre sous la forme d’un iframe flottant et non d’une fenêtre indépendante ; elle s’ouvre ainsi plus rapidement. Voici un exemple.

```js
Office.context.ui.displayDialogAsync('https://myDomain/myDialog.html', {height: 30, width: 20, displayInIframe: true});
```

La valeur par défaut est `false`, ce qui revient à omettre entièrement la propriété. Si le complément n’est pas exécuté dans Office sur le Web, le `displayInIframe` est ignoré.

> [!NOTE]
> Vous ne devez **pas** utiliser `displayInIframe: true` si la boîte de dialogue redirige à un moment donné l’utilisateur vers une page qui ne peut pas être ouverte dans un IFrame. Par exemple, les pages de signature de nombreux services web populaires, tels que les comptes Google et Microsoft, ne peuvent pas être ouvertes dans un iFrame.

## <a name="send-information-from-the-dialog-box-to-the-host-page"></a>Envoi d’informations à la page hôte à partir de la boîte de dialogue

La boîte de dialogue ne peut pas communiquer avec la page hôte dans le volet Office, sauf si :

- la page active dans la boîte de dialogue se trouve dans le même domaine que la page hôte ;
- La Office’API JavaScript est chargée dans la page. (Comme pour toute page qui utilise la bibliothèque d’API JavaScript Office, le script de la page doit affecter une méthode à la propriété, bien qu’il puisse s’agit `Office.initialize` d’une méthode vide. Pour plus d’informations, [voir Initialize your Office Add-in](initialize-add-in.md).)

Le code de la boîte de dialogue utilise [la fonction messageParent](/javascript/api/office/office.ui#messageparent-message-) pour envoyer un message de chaîne à la page hôte. La chaîne peut être un mot, une phrase, un blob XML, un JSON stringified ou toute autre chaîne qui peut être sérialisée en chaîne ou castée en chaîne. Voici un exemple.

```js
if (loginSuccess) {
    Office.context.ui.messageParent(true.toString());
}
```

> [!IMPORTANT]
> - La fonction `messageParent` peut uniquement être appelée sur une page ayant le même domaine (y compris les mêmes protocole et port) que la page hôte.
> - La `messageParent` fonction est l’une *des* deux seules API JS Office qui peuvent être appelées dans la boîte de dialogue.
> - L’autre API JS qui peut être appelée dans la boîte de dialogue est `Office.context.requirements.isSetSupported` . Pour plus d’informations à ce sujet, voir Spécifier les Office [applications et les api requises.](specify-office-hosts-and-api-requirements.md) Toutefois, dans la boîte de dialogue, cette API n’est pas prise en charge dans Outlook 2016'achat à prix seul (c’est-à-dire, la version MSI).

Dans l’exemple suivant, `googleProfile` est une version convertie en chaîne du profil Google de l’utilisateur.

```js
if (loginSuccess) {
    Office.context.ui.messageParent(googleProfile);
}
```

La page hôte doit être configurée de façon à recevoir le message. Pour ce faire, ajoutez un paramètre de rappel à l’appel d’origine de `displayDialogAsync`. Le rappel attribue un gestionnaire à l’événement `DialogMessageReceived`. Voici un exemple.

```js
var dialog;
Office.context.ui.displayDialogAsync('https://myDomain/myDialog.html', {height: 30, width: 20},
    function (asyncResult) {
        dialog = asyncResult.value;
        dialog.addEventHandler(Office.EventType.DialogMessageReceived, processMessage);
    }
);
```

> [!NOTE]
> - Office transmet un objet [AsyncResult](/javascript/api/office/office.asyncresult) au rappel. Il représente le résultat de la tentative d’ouverture de la boîte de dialogue. Il ne représente pas le résultat de tous les événements dans la boîte de dialogue. Pour plus d’informations sur cette distinction, consultez la [Gestion des erreurs et des événements](dialog-handle-errors-events.md).
> - La propriété `value` de `asyncResult` est définie sur un objet [Dialog](/javascript/api/office/office.dialog), qui existe dans la page hôte, pas dans le contexte d’exécution de la boîte de dialogue.
> - `processMessage` est la fonction qui gère l’événement. Vous pouvez lui donner le nom que vous souhaitez.
> - La variable `dialog` est déclarée avec une portée plus large que le rappel, car elle est également référencée dans `processMessage`.

Voici un exemple simple de gestionnaire pour l’événement `DialogMessageReceived`.

```js
function processMessage(arg) {
    var messageFromDialog = JSON.parse(arg.message);
    showUserName(messageFromDialog.name);
}
```

> [!NOTE]
> - Office transmet l’objet `arg` au gestionnaire. Sa `message` propriété est la chaîne envoyée par l’appel de la boîte de `messageParent` dialogue. Dans cet exemple, il s’agit d’une représentation sous la chaîne du profil d’un utilisateur à partir d’un service tel qu’un compte Microsoft ou Google, de sorte qu’il est désérialisé à un objet avec `JSON.parse` .
> - L’implémentation `showUserName` n’est pas visible. Elle peut afficher un message de bienvenue personnalisé dans le volet Office.

Lorsque l’intervention de l’utilisateur sur la boîte de dialogue est terminée, votre gestionnaire de messages doit fermer la boîte de dialogue, comme indiqué dans cet exemple.

```js
function processMessage(arg) {
    dialog.close();
    // message processing code goes here;
}
```

> [!NOTE]
> - L’objet `dialog` doit être le même que celui renvoyé par l’appel de `displayDialogAsync`.
> - L’appel de `dialog.close` indique à Office de fermer immédiatement la boîte de dialogue.

Pour voir un exemple de complément qui utilise ces techniques, consultez la rubrique relative à l’[exemple d’API de dialogue de complément Office](https://github.com/OfficeDev/Office-Add-in-Dialog-API-Simple-Example).

Si le complément a besoin d’ouvrir une autre page du volet Office après avoir reçu le message, vous pouvez utiliser la méthode `window.location.replace` (ou `window.location.href`) en tant que dernière ligne du gestionnaire. Voici un exemple.

```js
function processMessage(arg) {
    // message processing code goes here;
    window.location.replace("/newPage.html");
    // Alternatively ...
    // window.location.href = "/newPage.html";
}
```

Pour voir un exemple de complément qui effectue ce type d’action, consultez l’article relatif à l’exemple [Insérer des graphiques Excel à l’aide de Microsoft Graph dans un complément PowerPoint](https://github.com/OfficeDev/PowerPoint-Add-in-Microsoft-Graph-ASPNET-InsertChart).

### <a name="conditional-messaging"></a>Messagerie conditionnelle

Étant donné que vous pouvez envoyer plusieurs appels `messageParent` à partir de la boîte de dialogue, mais que vous n’avez qu’un seul gestionnaire dans la page hôte pour l’événement `DialogMessageReceived`, le gestionnaire doit utiliser la logique conditionnelle pour distinguer les différents messages. Par exemple, si la boîte de dialogue invite un utilisateur à se connecter à un fournisseur d’identité tel qu’un compte Microsoft ou Google, elle envoie le profil de l’utilisateur en tant que message. Si l’authentification échoue, la boîte de dialogue envoie des informations d’erreur à la page hôte, comme dans l’exemple suivant.

```js
if (loginSuccess) {
    var userProfile = getProfile();
    var messageObject = {messageType: "signinSuccess", profile: userProfile};
    var jsonMessage = JSON.stringify(messageObject);
    Office.context.ui.messageParent(jsonMessage);
} else {
    var errorDetails = getError();
    var messageObject = {messageType: "signinFailure", error: errorDetails};
    var jsonMessage = JSON.stringify(messageObject);
    Office.context.ui.messageParent(jsonMessage);
}
```

> [!NOTE]
> - La variable `loginSuccess` serait initialisée en lisant la réponse HTTP à partir du fournisseur d’identité.
> - L’implémentation des fonctions `getProfile` et `getError` n’est pas affichée. Chacune obtient des données à partir d’un paramètre de requête ou du corps de la réponse HTTP.
> - Des objets anonymes de différents types sont envoyés selon que la connexion a réussi ou non. Tous deux ont une propriété `messageType`, mais un a une propriété `profile` et l’autre une propriété `error`.

Le code du gestionnaire dans la page hôte utilise la valeur de la propriété `messageType` pour créer une branche comme le montre l’exemple suivant. Notez que la fonction `showUserName` est identique à celle de l’exemple précédent et que la fonction `showNotification` affiche l’erreur dans l’interface utilisateur de la page hôte.

```js
function processMessage(arg) {
    var messageFromDialog = JSON.parse(arg.message);
    if (messageFromDialog.messageType === "signinSuccess") {
        dialog.close();
        showUserName(messageFromDialog.profile.name);
        window.location.replace("/newPage.html");
    } else {
        dialog.close();
        showNotification("Unable to authenticate user: " + messageFromDialog.error);
    }
}
```

> [!NOTE]
> L'implémentation `showNotification` n'est pas montrée dans l'exemple de code fourni par cet article. Pour un exemple d'implémentation de cette fonction dans votre complément, voir [Exemple d'API de dialogue de complément Office](https://github.com/OfficeDev/Office-Add-in-Dialog-API-Simple-Example).

## <a name="pass-information-to-the-dialog-box"></a>Transmission d’informations à la boîte de dialogue

Votre add-in peut envoyer des messages à partir de la [page hôte](dialog-api-in-office-add-ins.md#open-a-dialog-box-from-a-host-page) vers une boîte de dialogue à l’aide [de Dialog.messageChild](/javascript/api/office/office.dialog#messagechild-message-).

### <a name="use-messagechild-from-the-host-page"></a>Utilisation `messageChild()` à partir de la page hôte

Lorsque vous appelez l’API Office boîte de dialogue pour ouvrir une boîte de dialogue, un objet [Dialog](/javascript/api/office/office.dialog) est renvoyé. Elle doit être affectée à une variable dont l’étendue est supérieure à celle de la méthode [displayDialogAsync,](/javascript/api/office/office.ui#displaydialogasync-startaddress--callback-) car l’objet sera référencé par d’autres méthodes. Voici un exemple.

```javascript
var dialog;
Office.context.ui.displayDialogAsync('https://myDomain/myDialog.html',
    function (asyncResult) {
        dialog = asyncResult.value;
        dialog.addEventHandler(Office.EventType.DialogMessageReceived, processMessage);
    }
);

function processMessage(arg) {
    dialog.close();

  // message processing code goes here;

}
```

Cet objet possède une méthode messageChild qui envoie toute chaîne, y compris les données `Dialog` stringified, [](/javascript/api/office/office.dialog#messagechild-message-) à la boîte de dialogue. Cela lève un événement `DialogParentMessageReceived` dans la boîte de dialogue. Votre code doit gérer cet événement, comme illustré dans la section suivante.

Envisagez un scénario dans lequel l’interface utilisateur de la boîte de dialogue est liée à la feuille de calcul active et à sa position par rapport aux autres feuilles de calcul. Dans l’exemple suivant, `sheetPropertiesChanged` Excel propriétés de feuille de calcul à la boîte de dialogue. Dans ce cas, la feuille de calcul actuelle est nommée « My Sheet » et il s’agit de la deuxième feuille du workbook. Les données sont encapsulées dans un objet et stringified afin de pouvoir être transmises à `messageChild` .

```javascript
function sheetPropertiesChanged() {
    var messageToDialog = JSON.stringify({
                               name: "My Sheet",
                               position: 2
                           });

    dialog.messageChild(messageToDialog);
}
```

### <a name="handle-dialogparentmessagereceived-in-the-dialog-box"></a>Gérer DialogParentMessageReceived dans la boîte de dialogue

Dans le javaScript de la boîte de dialogue, inscrivez un handler pour l’événement à l’auprès de la méthode `DialogParentMessageReceived` [UI.addHandlerAsync.](/javascript/api/office/office.ui#addhandlerasync-eventtype--handler--options--callback-) Cette méthode est généralement effectuée dans [les méthodes Office.onReady ou Office.initialize,](initialize-add-in.md)comme illustré ci-après. (Vous trouverez ci-dessous un exemple plus robuste.)

```javascript
Office.onReady()
    .then(function() {
        Office.context.ui.addHandlerAsync(
            Office.EventType.DialogParentMessageReceived,
            onMessageFromParent);
    });
```

Ensuite, définissez `onMessageFromParent` le handler. Le code suivant poursuit l’exemple de la section précédente. Notez Office passe un argument au handler et que la propriété de l’objet argument contient la chaîne de `message` la page hôte. Dans cet exemple, le message est reconverti en objet et jQuery est utilisé pour définir le titre supérieur de la boîte de dialogue afin qu’il corresponde au nouveau nom de feuille de calcul.

```javascript
function onMessageFromParent(event) {
    var messageFromParent = JSON.parse(event.message);
    $('h1').text(messageFromParent.name);
}
```

Il est préférable de vérifier que votre handler est correctement enregistré. Vous pouvez le faire en passant un rappel à la `addHandlerAsync` méthode. Cela s’exécute lorsque la tentative d’inscription du handler est terminée. Utilisez le handler pour enregistrer ou afficher une erreur si le handler n’a pas été enregistré avec succès. Voici un exemple. Notez `reportError` qu’il s’agit d’une fonction, qui n’est pas définie ici, qui enregistre ou affiche l’erreur.

```javascript
Office.onReady()
    .then(function() {
        Office.context.ui.addHandlerAsync(
            Office.EventType.DialogParentMessageReceived,
            onMessageFromParent,
            onRegisterMessageComplete);
    });

function onRegisterMessageComplete(asyncResult) {
    if (asyncResult.status !== Office.AsyncResultStatus.Succeeded) {
        reportError(asyncResult.error.message);
    }
}
```

### <a name="conditional-messaging-from-parent-page-to-dialog-box"></a>Messagerie conditionnelle d’une page parent à une boîte de dialogue

Étant donné que vous pouvez effectuer plusieurs appels à partir de la page hôte, mais que vous n’avez qu’un seul responsable dans la boîte de dialogue pour l’événement, le responsable doit utiliser une logique conditionnelle pour distinguer les `messageChild` `DialogParentMessageReceived` différents messages. Vous pouvez le faire d’une manière qui est précisément parallèle à la façon dont vous structureriez la messagerie conditionnelle lorsque la boîte de dialogue envoie un message à la page hôte comme décrit dans la messagerie [conditionnelle.](#conditional-messaging)

> [!NOTE]
> Dans certains cas, l’API, qui fait partie de l’ensemble de conditions `messageChild` [requises DialogApi 1.2,](../reference/requirement-sets/dialog-api-requirement-sets.md)peut ne pas être prise en charge. D’autres méthodes de messagerie de parent à boîte de dialogue sont décrites de manière alternative pour transmettre des messages à une boîte de dialogue à partir de [sa page hôte.](parent-to-dialog.md)

> [!IMPORTANT]
> L’ensemble de conditions [requises DialogApi 1.2](../reference/requirement-sets/dialog-api-requirement-sets.md) ne peut pas être spécifié dans la `<Requirements>` section d’un manifeste de add-in. Vous devez vérifier la prise en charge de DialogApi 1.2 lors de l’runtime à l’aide de la méthode [isSetSupported.](specify-office-hosts-and-api-requirements.md#use-runtime-checks-in-your-javascript-code) La prise en charge des exigences de manifeste est en cours de développement.

## <a name="closing-the-dialog-box"></a>Fermeture de la boîte de dialogue

Vous pouvez implémenter un bouton de fermeture dans la boîte de dialogue. Pour ce faire, le gestionnaire d'événements Click du bouton doit utiliser `messageParent` pour indiquer à la page hôte que vous avez cliqué sur le bouton. Voici un exemple.

```js
function closeButtonClick() {
    var messageObject = {messageType: "dialogClosed"};
    var jsonMessage = JSON.stringify(messageObject);
    Office.context.ui.messageParent(jsonMessage);
}
```

Le gestionnaire de la page hôte pour `DialogMessageReceived` appelle `dialog.close`, comme dans cet exemple. (consultez les exemples précédents qui montrent comment l’objet `dialog` est initialisé).

```js
function processMessage(arg) {
    var messageFromDialog = JSON.parse(arg.message);
    if (messageFromDialog.messageType === "dialogClosed") {
       dialog.close();
    }
}
```

Même lorsque vous ne disposez pas de votre propre interface utilisateur de fermeture de boîte de dialogue, un utilisateur final peut fermer la boîte de dialogue en choisissant le **X** dans le coin supérieur droit. Cette action déclenche l’événement `DialogEventReceived`. Si votre volet hôte a besoin de savoir quand cela se produit, il doit déclarer un gestionnaire pour cet événement. Pour plus d’informations, consultez la section [Erreurs et événements dans la boîte de dialogue](dialog-handle-errors-events.md#errors-and-events-in-the-dialog-box).

## <a name="advanced-topics-and-special-scenarios"></a>Rubriques plus complexes et scénarios spéciaux

### <a name="use-the-dialog-api-to-show-a-video"></a>Utilisation d'un API de boîte de dialogue pour afficher une vidéo

Voir [Utiliser la boîte de dialogue Office pour afficher une vidéo](dialog-video.md).

### <a name="use-the-dialog-apis-in-an-authentication-flow"></a>Utilisation des API de boîte de dialogue dans un flux d’authentification

Voir [Authentifier avec l’API de boîte de dialogue Office](auth-with-office-dialog-api.md).

### <a name="using-the-office-dialog-api-with-single-page-applications-and-client-side-routing"></a>Utilisation de l’API de boîte de dialogue Office avec des applications à page unique et routage côté client

Les authentifications par mot de passe (SPA) et le routage client doivent être gérés avec précaution lorsque vous utilisez l’API de boîte de dialogue Office. Consultez les [Pratiques recommandées pour l’utilisation de l’API de boîte de dialogue Office dans une SPA](dialog-best-practices.md#best-practices-for-using-the-office-dialog-api-in-an-spa).

### <a name="error-and-event-handling"></a>Gestion d'erreurs et d'événements

Voir [Gestion des erreurs et des événements dans la boîte de dialogue Office](dialog-handle-errors-events.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez les pièges et pratiques recommandées pour l’API de boîte de dialogue Office dans les [Meilleures pratiques et règles pour l’API de boîte de dialogue Office](dialog-best-practices.md).

## <a name="samples"></a>Exemples

Tous les exemples suivants utilisent `displayDialogAsync` . Certains ont des serveurs NodeJS et d’autres des serveurs basés sur ASP.NET/IIS, mais la logique d’utilisation de la méthode est la même quelle que soit la façon dont le côté serveur du module est implémenté.

**Informations de base :**

- [Exemple d’API de dialogue pour compléments Office](https://github.com/OfficeDev/Office-Add-in-Dialog-API-Simple-Example)
- [Contenu de formation/ Création de add-ins (plusieurs exemples)](https://github.com/OfficeDev/TrainingContent/tree/2db14a16774e1539a3eebae7dada4798142b8493/OfficeAddin)

**Exemples plus complexes :**

- [Office Add-in Microsoft Graph ASPNET](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Office-Add-in-Microsoft-Graph-ASPNET)
- [Complément Office Microsoft Graph React](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Office-Add-in-Microsoft-Graph-React)
- [SSO NodeJS pour complément Office](https://github.com/OfficeDev/Office-Add-in-NodeJS-SSO)
- [Office Add-in ASPNET SSO](https://github.com/OfficeDev/Office-Add-in-ASPNET-SSO)
- [Office Exemple de monétisation SAAS pour le add-in](https://github.com/OfficeDev/office-add-in-saas-monetization-sample)
- [Outlook Add-in Microsoft Graph ASPNET](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Outlook-Add-in-Microsoft-Graph-ASPNET)
- [Outlook Add-in SSO](https://github.com/OfficeDev/Outlook-Add-in-SSO)
- [Outlook Visionneuse de jetons de add-in](https://github.com/OfficeDev/Outlook-Add-In-Token-Viewer)
- [Outlook Message actionnable du add-in](https://github.com/OfficeDev/Outlook-Add-In-Actionable-Message)
- [Outlook Partage de OneDrive](https://github.com/OfficeDev/Outlook-Add-in-Sharing-to-OneDrive)
- [PowerPoint Add-in Microsoft Graph ASPNET InsertChart](https://github.com/OfficeDev/PowerPoint-Add-in-Microsoft-Graph-ASPNET-InsertChart)
- [Excel Scénario d’runtime partagé](https://github.com/OfficeDev/PnP-OfficeAddins/tree/900b5769bca9bbcff79d6cd6106d9fcc55c70d5a/Samples/excel-shared-runtime-scenario)
- [Excel Add-in ASPNET QuickBooks](https://github.com/OfficeDev/Excel-Add-in-ASPNET-QuickBooks)
- [Word Add-in JS Redact](https://github.com/OfficeDev/Word-Add-in-JS-Redact)
- [Word Add-in JS SpecKit](https://github.com/OfficeDev/Word-Add-in-JS-SpecKit)
- [Word Add-in AngularJS Client OAuth](https://github.com/OfficeDev/Word-Add-in-AngularJS-Client-OAuth)
- [Complément Office dans Auth0](https://github.com/OfficeDev/Office-Add-in-Auth0)
- [Office Ajout de OAuth.io](https://github.com/OfficeDev/Office-Add-in-OAuth.io)
- [Office Code de modèles de conception de l’UX de l’UX de l’ajout](https://github.com/OfficeDev/Office-Add-in-UX-Design-Patterns-Code)
