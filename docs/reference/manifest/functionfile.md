---
title: Élément FunctionFile dans le fichier manifest
description: Spécifie le fichier de code source pour les opérations qu’un complément expose via les commandes de complément qui exécutent une fonction JavaScript au lieu d’afficher l’interface utilisateur.
ms.date: 11/06/2020
localization_priority: Normal
ms.openlocfilehash: f31a1bc7a561305a89f5388102a4985aaa31fe37
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53348299"
---
# <a name="functionfile-element"></a>Élément FunctionFile

Spécifie le fichier de code source pour les opérations qu’un add-in expose de l’une des manières suivantes.

* Commandes de add-in qui exécutent une fonction JavaScript au lieu d’afficher l’interface utilisateur.
* Raccourcis clavier qui exécutent une fonction JavaScript.

`FunctionFile`L’élément est un élément enfant de [DesktopFormFactor](desktopformfactor.md) ou [MobileFormFactor](mobileformfactor.md). L’attribut de l’élément ne peut pas être plus de 32 caractères et est défini sur la valeur de l’attribut d’un élément dans l’élément qui contient l’URL d’un fichier HTML qui contient ou charge toutes les fonctions JavaScript utilisées par les boutons de commande de l’interface utilisateur sans interface `resid` `FunctionFile` `id` `Url` `Resources` utilisateur, [](control.md)comme défini par l’élément Control .

Voici un exemple de `FunctionFile` l’élément.

```XML
<DesktopFormFactor>
  <FunctionFile resid="residDesktopFuncUrl" />
  <ExtensionPoint xsi:type="PrimaryCommandSurface">
    <!-- information about this extension point -->
  </ExtensionPoint>

  <!-- You can define more than one ExtensionPoint element as needed -->

</DesktopFormFactor>
```

Le code JavaScript dans le fichier HTML indiqué par l’élément doit appeler et définir des fonctions nommées qui `FunctionFile` `Office.initialize` prennent un seul paramètre : `event` . Les fonctions doivent utiliser l’`item.notificationMessages` API pour indiquer l’avancement, réussite ou Échec de l’utilisateur. Elle doit également appeler `event.completed` lorsqu’il a fini d’exécution. Le nom des fonctions est utilisé dans l’élément pour les boutons sans interface `FunctionName` utilisateur.

Voici un exemple de fichier HTML définissant une `trackMessage` fonction.

```js
Office.initialize = function () {
    doAuth();
}

function trackMessage (event) {
    var buttonId = event.source.id;    
    var itemId = Office.context.mailbox.item.id;
    // save this message
    event.completed();
}
```

Le code suivant montre comment implémenter la fonction utilisée par `FunctionName` .

```js
// The initialize function must be run each time a new page is loaded.
(function () {
    Office.initialize = function (reason) {
        // If you need to initialize something you can do so here.
    };
})();

// Your function must be in the global namespace.
function writeText(event) {

    // Implement your custom code here. The following code is a simple example.

    Office.context.document.setSelectedDataAsync("ExecuteFunction works. Button ID=" + event.source.id,
        function (asyncResult) {
            var error = asyncResult.error;
            if (asyncResult.status === "failed") {
                // Show error message.
            }
            else {
                // Show success message.
            }
        });
    // Calling event.completed is required. event.completed lets the platform know that processing has completed.
    event.completed();
}
```

> [!IMPORTANT]
> L’appel `event.completed` aux signaux que vous avez géré l’événement avec succès. Lorsqu’une fonction est appelée à plusieurs reprises, par exemple lorsque l’utilisateur clique plusieurs fois sur une même commande de complément, tous les événements sont automatiquement mis en file d’attente. Le premier événement s’exécute automatiquement, tandis que les autres événements restent dans la file d’attente. Lorsque votre fonction appelle, l’appel mis en file `event.completed` d’attente suivant à cette fonction s’exécute. Vous devez appeler `event.completed` ; sinon, votre fonction ne s’exécutera pas.
