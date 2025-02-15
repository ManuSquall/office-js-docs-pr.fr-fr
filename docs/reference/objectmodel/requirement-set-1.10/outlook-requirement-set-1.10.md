---
title: Outlook conditions requises de l’API du add-in 1.10
description: Ensemble de conditions requises 1.10 pour Outlook API de votre application.
ms.date: 05/17/2021
localization_priority: Normal
ms.openlocfilehash: f5fda91c4105d56dcf9d20d570e48851c8b6dfeb
ms.sourcegitcommit: 0d9fcdc2aeb160ff475fbe817425279267c7ff31
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/21/2021
ms.locfileid: "52592036"
---
# <a name="outlook-add-in-api-requirement-set-110"></a>Outlook conditions requises de l’API du add-in 1.10

Le sous-ensemble d’API de Outlook de l’API JavaScript Office inclut des objets, des méthodes, des propriétés et des événements que vous pouvez utiliser dans un Outlook.

## <a name="whats-new-in-110"></a>Nouveautés de la 1.10

L’ensemble de conditions requises 1.10 inclut toutes les fonctionnalités de l’ensemble de conditions [requises 1.9](../requirement-set-1.9/outlook-requirement-set-1.9.md). Les fonctionnalités suivantes ont été ajoutées.

- Ajout de nouvelles API pour [l’activation basée](../../../outlook/autolaunch.md) sur des événements et les fonctionnalités de signature électronique.
- Ajout de la possibilité d’inclure une action personnalisée sur un message de notification.

### <a name="change-log"></a>Journal des modifications

- Ajout du [point d’extension LaunchEvent](../../manifest/extensionpoint.md#launchevent): ajoute un nouveau type d’ExtensionPoint pris en charge. Il configure la fonctionnalité d’activation basée sur des événements.
- Ajout de [l’élément de manifeste LaunchEvents](../../manifest/launchevents.md): ajoute un élément manifeste pour prendre en charge la configuration de la fonctionnalité d’activation basée sur les événements.
- Élément [manifeste Runtimes modifié](../../manifest/runtimes.md): ajoute Outlook prise en charge. Il fait référence aux fichiers HTML et JavaScript nécessaires pour la fonctionnalité d’activation basée sur des événements.
- Ajout [Office.context.mailbox.item.body.setSignatureAsync](/javascript/api/outlook/office.body?view=outlook-js-1.10&preserve-view=true#setsignatureasync-data--options--callback-): ajoute une nouvelle fonction à `Body` l’objet. Il ajoute ou remplace la signature dans le corps de l’élément en mode Composition.
- Ajout de [Office.context.mailbox.item.disableClientSignatureAsync](office.context.mailbox.item.md#methods): ajoute une nouvelle fonction qui désactive la signature du client pour la boîte aux lettres d’envoi en mode composition.
- Ajout [Office.context.mailbox.item.getComposeTypeAsync](/javascript/api/outlook/office.messagecompose?view=outlook-js-1.10&preserve-view=true#getcomposetypeasync-options--callback-): ajoute une nouvelle fonction qui obtient le type de composition d’un message en mode composition.
- Ajout de [Office.context.mailbox.item.isClientSignatureEnabledAsync](office.context.mailbox.item.md#methods): ajoute une nouvelle fonction qui vérifie si la signature du client est activée sur l’élément en mode composition.
- Ajout [Office. MailboxEnums.ActionType :](/javascript/api/outlook/office.mailboxenums.actiontype)ajoute une nouvelle enum. Il représente le type d’action personnalisée dans un message de notification.
- Ajout [Office.MailboxEnums.ComposeType](/javascript/api/outlook/office.mailboxenums.composetype?view=outlook-js-1.10&preserve-view=true): ajoute une nouvelle enum disponible en mode Composition.
- Ajout [Office. MailboxEnums.ItemNotificationMessageType.InsightMessage](/javascript/api/outlook/office.mailboxenums.itemnotificationmessagetype): ajoute un nouveau type à `ItemNotificationMessageType` l’enum. Il représente un message de notification avec une action personnalisée.
- Ajout [Office. NotificationMessageAction](/javascript/api/outlook/office.notificationmessageaction): ajoute un nouvel objet afin que vous pouvez définir une action personnalisée pour votre `InsightMessage` notification.
- Ajout [Office. NotificationMessageDetails.actions](/javascript/api/outlook/office.notificationmessagedetails#actions): ajoute une nouvelle propriété qui vous permet d’ajouter une `InsightMessage` notification avec une action personnalisée.

## <a name="see-also"></a>Voir aussi

- [Compléments Outlook](../../../outlook/outlook-add-ins-overview.md)
- [Exemples de code pour les compléments Outlook](https://developer.microsoft.com/outlook/gallery/?filterBy=Outlook,Samples,Add-ins)
- [Prise en main](../../../quickstarts/outlook-quickstart.md)
- [Ensembles de conditions requises et clients pris en charge](../../requirement-sets/outlook-api-requirement-sets.md)
