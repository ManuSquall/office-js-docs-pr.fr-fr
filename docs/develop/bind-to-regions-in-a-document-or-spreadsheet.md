---
title: Lier des régions dans un document ou une feuille de calcul
description: Découvrez comment utiliser la liaison pour garantir un accès cohérent à une région ou à un élément spécifique d’un document ou d’une feuille de calcul via un identificateur.
ms.date: 06/20/2019
localization_priority: Normal
ms.openlocfilehash: c9a658653c562de446f3b8e5f1ea192ddfcf3b21
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53350000"
---
# <a name="bind-to-regions-in-a-document-or-spreadsheet"></a>Lier des régions dans un document ou une feuille de calcul

L’accès aux données basées sur une liaison permet aux compléments de contenu et du volet Office d’accéder de façon cohérente à une zone particulière d’un document ou d’une feuille de calcul au moyen d’un identificateur. Le complément doit d’abord établir la liaison en appelant l’une des méthodes qui associent une partie du document à un identificateur unique : [addFromPromptAsync], [addFromSelectionAsync] ou [addFromNamedItemAsync]. Une fois la liaison établie, le complément peut utiliser l’identificateur fourni pour accéder aux données contenues dans la région associée du document ou de la feuille de calcul. La création de liaisons fournit la valeur suivante à votre add-in.

- Elle permet l’accès aux structures de données communes sur les applications Office prises en charge, telles que : tableaux, plages ou texte (série contiguë de caractères).
- Elle permet les opérations de lecture/écriture sans exiger que l’utilisateur effectue une sélection.
- Elle établit une relation entre le complément et les données du document. Les liaisons persistent dans le document et sont accessibles par la suite.

L’établissement d’une liaison vous permet également de vous abonner aux données et aux événements de changement de sélection qui sont concernés par cette région particulière du document ou de la feuille de calcul. Cela signifie que le complément est seulement notifié des changements qui surviennent dans la région délimitée, par opposition aux changements généraux affectant l’ensemble du document ou de la feuille de calcul.

L’objet [Bindings] expose une méthode [getAllAsync] qui donne accès à toutes les liaisons établies dans le document ou la feuille de calcul. Une liaison individuelle est accessible par son ID à l’aide de la méthode [Bindings.getBindingByIdAsync] ou [Office.select]. Vous pouvez établir de nouvelles liaisons et supprimer des liaisons existantes en utilisant l’une des méthodes suivantes de l’objet [Bindings] : [addFromSelectionAsync], [addFromPromptAsync], [addFromNamedItemAsync] ou [releaseByIdAsync].

## <a name="binding-types"></a>Types de liaison

Il existe [trois types de liaisons][Office. BindingType] que vous spécifiez avec le paramètre _bindingType_ lorsque vous créez une liaison avec les méthodes [addFromSelectionAsync], [addFromPromptAsync] ou [addFromNamedItemAsync] :

1. **[Liaison de texte][TextBinding]** - Établit une liaison à une zone du document qui est représentée en tant que texte.

    Dans Word, la plupart des sélections contiguës sont valides, tandis que dans Excel, seules les sélections de cellules uniques peuvent être la cible d’une liaison de texte. Dans Excel, seul le texte brut est pris en charge. Dans Word, trois formats sont pris en charge : texte brut, HTML et Open XML pour Office.

2. **[Matrix Binding][MatrixBinding]** : se lie à une région fixe d’un document qui contient des données tabulaires sans en-têtes. Les données d’une liaison de matrice sont écrites ou lues sous la forme d’un tableau à deux **dimensions,** qui en JavaScript est implémenté sous la forme d’un tableau de tableaux. Par exemple, deux lignes de valeurs **string** dans deux colonnes peuvent être écrites ou lues comme `[['a', 'b'], ['c', 'd']]`, et une colonne unique de trois lignes peut être écrite ou lue comme `[['a'], ['b'], ['c']]`.

    Dans Excel, toute sélection contiguë de cellules peut être utilisée pour établir une liaison de matrice. Dans Word, seuls les tableaux prennent en charge la liaison de matrice.

3. **[Liaison de tableau][TableBinding]** - Établit une liaison à une zone d’un document qui contient un tableau avec des en-têtes. Les données dans une liaison de tableau sont écrites ou lues comme un objet [TableData](/javascript/api/office/office.tabledata). L’objet `TableData` expose les données via les propriétés `headers` et `rows`.

    Tout tableau Excel ou Word peut être la base d’une liaison de tableau. Une fois que vous établissez une liaison de tableau, chaque nouvelle ligne ou colonne qu’un utilisateur ajoute au tableau est automatiquement incluse dans la liaison.

Après la création d’une liaison à l’aide de l’une des trois méthodes « addFrom » de l’objet, vous pouvez utiliser les données et propriétés de la liaison à l’aide des méthodes de `Bindings` l’objet correspondant : [MatrixBinding,] [TableBinding]ou [TextBinding]. Ces trois objets héritent des méthodes [getDataAsync] et [setDataAsync] de l’objet `Binding` qui vous permettent d’interagir avec les données liées.

> [!NOTE]
> **Quand devez-vous utiliser une liaison de matrice ou une liaison de tableau ?** Lorsque les données tabulaires avec lesquelles vous travaillez contiennent une ligne de total, vous devez utiliser une liaison de matrice si le script de votre complément doit accéder aux valeurs figurant dans la ligne de total ou détecter que la sélection de l’utilisateur figure dans la ligne de total. Si vous établissez une liaison de tableau pour des données tabulaires qui contiennent une ligne de total, la propriété [TableBinding.rowCount] et les propriétés `rowCount` et `startRow` de l’objet [BindingSelectionChangedEventArgs] dans les gestionnaires d’événements ne reflèteront pas la ligne de total dans leurs valeurs. Pour contourner cette limitation, vous devez établir une liaison de matrice pour travailler avec la ligne de total.

## <a name="add-a-binding-to-the-users-current-selection"></a>Ajout d’une liaison à la sélection actuelle de l’utilisateur

L’exemple suivant montre comment ajouter une liaison de texte nommée `myBinding` à la sélection actuelle dans un document à l’aide de la méthode [addFromSelectionAsync].

```js
Office.context.document.bindings.addFromSelectionAsync(Office.BindingType.Text, { id: 'myBinding' }, function (asyncResult) {
    if (asyncResult.status == Office.AsyncResultStatus.Failed) {
        write('Action failed. Error: ' + asyncResult.error.message);
    } else {
        write('Added new binding with type: ' + asyncResult.value.type + ' and id: ' + asyncResult.value.id);
    }
});

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```

Dans cet exemple, le type de liaison spécifié est « Text ». Cela signifie qu’un objet [TextBinding] sera créé pour la sélection. Différents types de liaison exposent différentes données et opérations. [Office.BindingType] est une énumération des valeurs de types de liaison disponibles.

Le deuxième paramètre facultatif est un objet qui spécifie l’ID de la nouvelle liaison créée. Si un ID n’est pas spécifié, un ID est généré automatiquement.

La fonction anonyme qui est passée dans la fonction comme paramètre final _callback_ est exécutée lorsque la création de la liaison est terminée. La fonction est appelée avec un seul paramètre, `asyncResult`, ce qui donne accès à un objet [AsyncResult] qui fournit l’état de l’appel. La propriété `AsyncResult.value` contient une référence à un objet [Binding] du type spécifié pour la liaison créée récemment. Vous pouvez utiliser cet objet [Binding] pour obtenir et définir les données.

## <a name="add-a-binding-from-a-prompt"></a>Ajout d’une liaison à partir d’une invite

L’exemple suivant indique comment ajouter une liaison de texte appelée `myBinding` à l’aide de la méthode [addFromPromptAsync]. Cette méthode permet à l’utilisateur de spécifier la plage pour la liaison à l’aide de l’invite de sélection de plage intégrée.

```js
function bindFromPrompt() {
    Office.context.document.bindings.addFromPromptAsync(Office.BindingType.Text, { id: 'myBinding' }, function (asyncResult) {
        if (asyncResult.status == Office.AsyncResultStatus.Failed) {
            write('Action failed. Error: ' + asyncResult.error.message);
        } else {
            write('Added new binding with type: ' + asyncResult.value.type + ' and id: ' + asyncResult.value.id);
        }
    });
}

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message;
}
```

Dans cet exemple, le type de liaison spécifié est « Text ». Cela signifie qu’un objet [TextBinding] sera créé pour la sélection que l’utilisateur spécifie dans l’invite.

Le deuxième paramètre est un objet qui contient l’ID de la nouvelle liaison créée. Si un ID n’est pas spécifié, un ID est généré automatiquement.

La fonction anonyme passée dans la fonction en tant que _troisième_ paramètre de rappel est exécutée lorsque la création de la liaison est terminée. Lorsque la fonction de rappel s’exécute, l’objet [AsyncResult] contient le statut de l’appel et la nouvelle liaison.

La figure 1 montre l’invite de sélection de plage intégrée dans Excel.

*Figure 1. Interface utilisateur de sélection de données dans Excel*

![Capture d’écran montrant la boîte de dialogue Sélectionner des données.](../images/agave-api-overview-excel-selection-ui.png)

## <a name="add-a-binding-to-a-named-item"></a>Ajout d’une liaison à un élément nommé

L’exemple suivant montre comment ajouter une liaison à l’élément nommé existant en tant que liaison « matrix » à l’aide de la méthode `myRange` [addFromNamedItemAsync,] et affecte la liaison en tant que `id` « myMatrix ».

```js
function bindNamedItem() {
    Office.context.document.bindings.addFromNamedItemAsync("myRange", "matrix", {id:'myMatrix'}, function (result) {
        if (result.status == 'succeeded'){
            write('Added new binding with type: ' + result.value.type + ' and id: ' + result.value.id);
            }
        else
            write('Error: ' + result.error.message);
    });
}

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}

```

**Pour Excel**, le paramètre de la méthode `itemName` [addFromNamedItemAsync] peut faire référence à une plage nommée existante, à une plage spécifiée avec le style de référence ou à un `A1` `("A1:A3")` tableau. Par défaut, l’ajout d’un tableau dans Excel entraîne l’affectation du nom « Tableau1 » pour le premier tableau que vous ajoutez, « Tableau2 » pour le deuxième tableau que vous ajoutez, et ainsi de suite. Pour attribuer un nom significatif à une table dans l’interface utilisateur Excel, utilisez la propriété dans la | `Table Name` **Onglet** Création du ruban.

> [!NOTE]
> Dans Excel, lorsque vous spécifiez un tableau en tant qu’élément nommé, vous devez qualifier complètement le nom pour inclure le nom de la feuille de calcul dans le nom du tableau au format suivant :`"Sheet1!Table1"`

L’exemple suivant crée une liaison dans Excel aux trois premières cellules de la colonne A ( ), affecte l’ID, puis écrit trois noms de ville dans `"A1:A3"` `"MyCities"` cette liaison.

```js
 function bindingFromA1Range() {
    Office.context.document.bindings.addFromNamedItemAsync("A1:A3", "matrix", {id: "MyCities" },
        function (asyncResult) {
            if (asyncResult.status == "failed") {
                write('Error: ' + asyncResult.error.message);
            }
            else {
                // Write data to the new binding.
                Office.select("bindings#MyCities").setDataAsync([['Berlin'], ['Munich'], ['Duisburg']], { coercionType: "matrix" },
                    function (asyncResult) {
                        if (asyncResult.status == "failed") {
                            write('Error: ' + asyncResult.error.message);
                        }
                    });
            }
        });
}
// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```

**Pour Word**, le paramètre de la méthode `itemName` [addFromNamedItemAsync] fait référence à la propriété `Title` d’un `Rich Text` contrôle de contenu. (Vous ne pouvez réaliser de liaison avec des contrôles de contenu différents du contrôle de contenu `Rich Text`.)

Par défaut, aucune valeur n’est affectée à un contrôle `Title*` de contenu. Pour affecter un nom significatif dans l’interface utilisateur de Word, après l’insertion d’un contrôle de contenu **Texte enrichi** à partir du groupe **Contrôles** sur l’onglet **Développeur** du ruban, utilisez la commande **Propriétés** du groupe **Contrôles** pour afficher la boîte de dialogue **Propriétés du contrôle de contenu**. Définissez ensuite la propriété du contrôle de contenu sur le nom que `Title` vous souhaitez référencer à partir de votre code.

L’exemple suivant crée une liaison de texte dans Word à un contrôle de contenu de texte enrichi nommé , affecte l’ID, puis affiche `"FirstName"`  `"firstName"` ces informations.

```js
function bindContentControl() {
    Office.context.document.bindings.addFromNamedItemAsync('FirstName', 
        Office.BindingType.Text, {id:'firstName'},
        function (result) {
            if (result.status === Office.AsyncResultStatus.Succeeded) {
                write('Control bound. Binding.id: '
                    + result.value.id + ' Binding.type: ' + result.value.type);
            } else {
                write('Error:', result.error.message);
            }
    });
}
// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```

## <a name="get-all-bindings"></a>Obtention de toutes les liaisons

L’exemple suivant montre comment obtenir toutes les liaisons dans un document en utilisant la méthode Bindings.[getAllAsync].

```js
Office.context.document.bindings.getAllAsync(function (asyncResult) {
    var bindingString = '';
    for (var i in asyncResult.value) {
        bindingString += asyncResult.value[i].id + '\n';
    }
    write('Existing bindings: ' + bindingString);
});

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```

La fonction anonyme qui est passée dans la fonction en tant que paramètre est `callback` exécutée lorsque l’opération est terminée. La fonction est appelée avec un seul paramètre, qui contient un `asyncResult` tableau des liaisons dans le document. Le tableau est répété pour générer une chaîne qui contient les ID des liaisons. La chaîne est ensuite affichée dans une boîte de message.

## <a name="get-a-binding-by-id-using-the-getbyidasync-method-of-the-bindings-object"></a>Obtention d’une liaison par ID en utilisant la méthode getByIdAsync de l’objet Bindings

L’exemple suivant indique comment utiliser la méthode [getByIdAsync] pour obtenir une liaison dans un document en spécifiant son ID. Cet exemple suppose qu’une liaison nommée `'myBinding'` a été ajoutée au document à l’aide des méthodes décrites plus haut dans cette rubrique.

```js
Office.context.document.bindings.getByIdAsync('myBinding', function (asyncResult) {
    if (asyncResult.status == Office.AsyncResultStatus.Failed) {
        write('Action failed. Error: ' + asyncResult.error.message);
    }
    else {
        write('Retrieved binding with type: ' + asyncResult.value.type + ' and id: ' + asyncResult.value.id);
    }
});

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```

Dans l’exemple, le premier `id` paramètre est l’ID de la liaison à récupérer.

La fonction anonyme qui est passée dans la fonction en tant que _deuxième_ paramètre de rappel est exécutée lorsque l’opération est terminée. La fonction est appelée avec un seul paramètre, _asyncResult_, qui contient le statut de l’appel et la liaison avec l’ID « myBinding ».

## <a name="get-a-binding-by-id-using-the-select-method-of-the-office-object"></a>Obtention d’une liaison par ID en utilisant la méthode Select de l’objet Office

L’exemple suivant montre comment utiliser la méthode [Office.select] pour obtenir une promesse d’objet [Binding] dans un document en spécifiant son ID dans une chaîne de sélecteur. Il appelle ensuite la méthode [Binding.getDataAsync] pour obtenir des données à partir de la liaison spécifiée. Cet exemple suppose qu’une liaison nommée `'myBinding'` a été ajoutée au document à l’aide des méthodes décrites plus haut dans cette rubrique.

```js
Office.select("bindings#myBinding", function onError(){}).getDataAsync(function (asyncResult) {
    if (asyncResult.status == Office.AsyncResultStatus.Failed) {
        write('Action failed. Error: ' + asyncResult.error.message);
    } else {
        write(asyncResult.value);
    }
});

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```


> [!NOTE]
> Si la promesse de méthode renvoie correctement un objet Binding, cet objet expose uniquement les quatre méthodes suivantes de l’objet : `select` [getDataAsync], [setDataAsync], [addHandlerAsync]et [removeHandlerAsync]. [] Si la promesse ne peut pas renvoyer un objet Binding, le rappel peut être utilisé pour accéder à un objet `onError` .error [asyncResult]pour obtenir plus d’informations. Si vous devez appeler un membre de l’objet Binding autre que les quatre méthodes exposées par la promesse d’objet [Binding] renvoyée par la méthode, utilisez plutôt la méthode `select` [getByIdAsync] à l’aide de la propriété [Document.bindings] et de Bindings.[ méthode getByIdAsync] pour récupérer [l’objet Binding.]

## <a name="release-a-binding-by-id"></a>Publication d’une liaison par ID

L’exemple suivant montre comment utiliser la méthode [releaseByIdAsync] pour publier une liaison dans un document en spécifiant son ID.

```js
Office.context.document.bindings.releaseByIdAsync('myBinding', function (asyncResult) {
    write('Released myBinding!');
});

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```

Dans l’exemple, le premier paramètre `id` est l’ID de la liaison à publier.

La fonction anonyme qui est passée dans la fonction comme le deuxième paramètre est un rappel qui est exécuté lorsque l’opération est terminée. La fonction est appelée avec un seul paramètre,  [asyncResult], qui contient le statut de l’appel.

## <a name="read-data-from-a-binding"></a>Lecture de données à partir d’une liaison

L’exemple suivant montre comment utiliser la méthode [getDataAsync] pour obtenir des données à partir d’une liaison existante.

```js
myBinding.getDataAsync(function (asyncResult) {
    if (asyncResult.status == Office.AsyncResultStatus.Failed) {
        write('Action failed. Error: ' + asyncResult.error.message);
    } else {
        write(asyncResult.value);
    }
});

// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```

`myBinding` est une variable qui contient une liaison de texte existante dans le document. Vous pouvez également utiliser [Office.select] pour accéder à la liaison avec son identifiant et commencer à appeler la méthode [getDataAsync] de la manière suivante : 

```js
Office.select("bindings#myBindingID").getDataAsync
```

La fonction anonyme qui est passée dans la fonction est un rappel qui est exécuté lorsque l’opération est terminée. La propriété [AsyncResult].value contient les données dans `myBinding`. Le type de valeur dépend du type de liaison. La liaison dans cet exemple est une liaison de texte. Par conséquent, la valeur contiendra une chaîne. Pour obtenir des exemples supplémentaires concernant l’utilisation des liaisons de matrice et de tableau, consultez la rubrique sur la méthode [getDataAsync].

## <a name="write-data-to-a-binding"></a>Écriture de données dans une liaison

L’exemple suivant montre comment utiliser la méthode [setDataAsync] pour définir des données dans une liaison existante.

```js
myBinding.setDataAsync('Hello World!', function (asyncResult) { });
```

`myBinding` est une variable qui contient une liaison de texte existante dans le document.

Dans l’exemple, le premier paramètre est la valeur à définir `myBinding` sur . Comme il s’agit d’une liaison de texte, la valeur est de type `string`. Différents types de liaisons acceptent divers types de données.

La fonction anonyme qui est passée dans la fonction est un rappel qui est exécuté lorsque l’opération est terminée. La fonction est appelée avec un seul paramètre, `asyncResult` qui contient l’état du résultat.

> [!NOTE]
> Depuis la publication d’Excel 2013 SP1 et de la version correspondante d’Excel sur le web, vous pouvez désormais [définir la mise en forme lors de l’écriture et de la mise à jour des données dans des tableaux liés](../excel/excel-add-ins-tables.md).

## <a name="detect-changes-to-data-or-the-selection-in-a-binding"></a>Détection des modifications apportées aux données ou à la section dans une liaison

L’exemple suivant montre comment lier un gestionnaire d’événements à l’événement [DataChanged](/javascript/api/office/office.binding) d’une liaison ayant l’ID « MyBinding ».

```js
function addHandler() {
Office.select("bindings#MyBinding").addHandlerAsync(
    Office.EventType.BindingDataChanged, dataChanged);
}
function dataChanged(eventArgs) {
    write('Bound data changed in binding: ' + eventArgs.binding.id);
}
// Function that writes to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message;
}
```

`myBinding` est une variable qui contient une liaison de texte existante dans le document.

Le premier _paramètre eventType_ de la méthode [addHandlerAsync] spécifie le nom de l’événement à abonner. [Office.EventType] est une énumération des valeurs de types d’événement disponibles. `Office.EventType.BindingDataChanged` évalue la chaîne « bindingDataChanged ».

La fonction qui est passée dans la fonction en tant que deuxième paramètre de handler est un handler d’événements qui est exécuté lorsque les données de la `dataChanged` liaison sont  modifiées. La fonction est appelée avec un seul paramètre, _eventArgs_, qui contient une référence à la liaison. Cette liaison peut être utilisée pour récupérer les données mises à jour.

De même, vous pouvez détecter lorsqu’un utilisateur modifie la sélection dans une liaison en ajoutant un gestionnaire d’événements à l’événement [SelectionChanged] d’une liaison. Pour ce faire, spécifiez le paramètre `eventType` de la méthode [addHandlerAsync] comme `Office.EventType.BindingSelectionChanged` ou `"bindingSelectionChanged"`.

Vous pouvez ajouter plusieurs gestionnaires d’événements pour un événement donné en appelant à nouveau la méthode [addHandlerAsync] et en transmettant une fonction de gestionnaire d’événements supplémentaire pour le paramètre `handler`. Cela fonctionnera correctement tant que le nom de chaque fonction de gestionnaire d’événements est unique.

### <a name="remove-an-event-handler"></a>Suppression d’un gestionnaire d’événements

Pour supprimer un gestionnaire d’événements pour un événement, appelez la méthode [removeHandlerAsync] en transmettant le type d’événement en tant que premier paramètre _eventType_, puis le nom de la fonction de gestionnaire d’événements à supprimer comme deuxième paramètre _handler_. Par exemple, la fonction suivante supprimera la fonction de gestionnaire d’événements `dataChanged` ajoutée dans l’exemple de la section précédente.

```js
function removeEventHandlerFromBinding() {
    Office.select("bindings#MyBinding").removeHandlerAsync(
        Office.EventType.BindingDataChanged, {handler:dataChanged});
}
```

> [!IMPORTANT]
> Si le paramètre de _handler_ facultatif est omis lors de l’appel de la méthode [removeHandlerAsync,] tous les handlers d’événements pour le paramètre spécifié `eventType` sont supprimés.

## <a name="see-also"></a>Voir aussi

- [Compréhension de l’API JavaScript pour Office](understanding-the-javascript-api-for-office.md)
- [Programmation asynchrone dans des compléments Office](asynchronous-programming-in-office-add-ins.md)
- [Lecture et écriture de données dans la sélection active d’un document ou d’une feuille de calcul](read-and-write-data-to-the-active-selection-in-a-document-or-spreadsheet.md)

[Binding]:               /javascript/api/office/office.binding
[MatrixBinding]:         /javascript/api/office/office.matrixbinding
[TableBinding]:          /javascript/api/office/office.tablebinding
[TextBinding]:           /javascript/api/office/office.textbinding
[getDataAsync]:          /javascript/api/office/Office.Binding#getdataasync-options--callback-
[setDataAsync]:          /javascript/api/office/Office.Binding#setdataasync-data--options--callback-
[SelectionChanged]:      /javascript/api/office/office.bindingselectionchangedeventargs
[addHandlerAsync]:       /javascript/api/office/Office.Binding#addhandlerasync-eventtype--handler--options--callback-
[removeHandlerAsync]:    /javascript/api/office/Office.Binding#removehandlerasync-eventtype--options--callback-

[Bindings]:              /javascript/api/office/office.bindings
[getByIdAsync]:          /javascript/api/office/office.bindings#getbyidasync-id--options--callback- 
[getAllAsync]:           /javascript/api/office/office.bindings#getallasync-options--callback-
[addFromNamedItemAsync]: /javascript/api/office/office.bindings#addfromnameditemasync-itemname--bindingtype--options--callback-
[addFromSelectionAsync]: /javascript/api/office/office.bindings#addfromselectionasync-bindingtype--options--callback-
[addFromPromptAsync]:    /javascript/api/office/office.bindings#addfrompromptasync-bindingtype--options--callback-
[releaseByIdAsync]:      /javascript/api/office/office.bindings#releasebyidasync-id--options--callback-

[AsyncResult]:          /javascript/api/office/office.asyncresult
[Office.BindingType]:   /javascript/api/office/office.bindingtype
[Office.select]:        /javascript/api/office 
[Office.EventType]:     /javascript/api/office/office.eventtype 
[Document.bindings]:    /javascript/api/office/office.document

[TableBinding.rowCount]: /javascript/api/office/office.tablebinding
[BindingSelectionChangedEventArgs]: /javascript/api/office/office.bindingselectionchangedeventargs
