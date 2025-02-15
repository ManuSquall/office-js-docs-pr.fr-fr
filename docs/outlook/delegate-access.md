---
title: Activer les dossiers partagés et les scénarios de boîtes aux lettres partagées dans un Outlook de messagerie
description: Explique comment configurer la prise en charge de la prise en charge des dossiers partagés (c’est-à-dire. accès délégué) et boîtes aux lettres partagées.
ms.date: 07/02/2021
localization_priority: Normal
ms.openlocfilehash: 70578f2c78a9dd88efc9ba70d5599a13e121df53
ms.sourcegitcommit: aa73ec6367eaf74399fbf8d6b7776d77895e9982
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/03/2021
ms.locfileid: "53290711"
---
# <a name="enable-shared-folders-and-shared-mailbox-scenarios-in-an-outlook-add-in"></a>Activer les dossiers partagés et les scénarios de boîtes aux lettres partagées dans un Outlook de messagerie

Cet article explique comment activer les scénarios de dossiers partagés [](../reference/objectmodel/preview-requirement-set/outlook-requirement-set-preview.md#shared-mailboxes)(également appelés accès délégué) et de boîtes aux lettres partagées (désormais en prévisualisation) dans votre application Outlook, y compris les autorisations que l’API JavaScript Office prend en charge.

> [!IMPORTANT]
> La prise en charge de cette fonctionnalité a été introduite dans [l’ensemble de conditions requises 1.8](../reference/objectmodel/requirement-set-1.8/outlook-requirement-set-1.8.md). Voir [les clients et les plateformes](../reference/requirement-sets/outlook-api-requirement-sets.md#requirement-sets-supported-by-exchange-servers-and-outlook-clients) qui prennent en charge cet ensemble de conditions requises.

## <a name="supported-setups"></a>Configurations prise en charge

Les sections suivantes décrivent les configurations prise en charge pour les boîtes aux lettres partagées (désormais en prévisualisation) et les dossiers partagés. Les API de fonctionnalité peuvent ne pas fonctionner comme prévu dans d’autres configurations. Sélectionnez la plateforme que vous souhaitez apprendre à configurer.

### <a name="windows"></a>[Windows](#tab/windows)

#### <a name="shared-folders"></a>Dossiers partagés

Le propriétaire de la boîte aux lettres [doit d’abord fournir l’accès à un délégué.](https://support.microsoft.com/office/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926) Le délégué doit ensuite suivre les instructions décrites dans la section « Ajouter la boîte aux lettres d’une autre personne à votre profil » de l’article Gérer les éléments de courrier et de calendrier [d’une autre personne.](https://support.microsoft.com/office/manage-another-person-s-mail-and-calendar-items-afb79d6b-2967-43b9-a944-a6b953190af5)

#### <a name="shared-mailboxes-preview"></a>Boîtes aux lettres partagées (aperçu)

Exchange administrateurs de serveur peuvent créer et gérer des boîtes aux lettres partagées pour des ensembles d’utilisateurs à accéder. Actuellement, [Exchange Online](/exchange/collaboration-exo/shared-mailboxes) est la seule version de serveur prise en charge pour cette fonctionnalité.

Une fonctionnalité Exchange Server appelée « mappage automatique » est mise en [](/microsoft-365/admin/email/create-a-shared-mailbox?view=o365-worldwide&preserve-view=true#add-the-shared-mailbox-to-outlook) place par défaut, ce qui signifie que la boîte aux lettres partagée doit apparaître automatiquement dans l’application Outlook d’un utilisateur après la fermeture et la réouverture de Outlook. Toutefois, si un administrateur a désactivé le mappage automatique, l’utilisateur doit suivre les étapes manuelles décrites dans la section « Ajouter une boîte aux lettres partagée à Outlook » de l’article Ouvrir et utiliser une boîte aux lettres partagée dans [Outlook](https://support.microsoft.com/office/open-and-use-a-shared-mailbox-in-outlook-d94a8e9e-21f1-4240-808b-de9c9c088afd).

> [!WARNING]
> Ne **vous connectez** PAS à la boîte aux lettres partagée avec un mot de passe. Les API de fonctionnalité ne fonctionneront pas dans ce cas.

### <a name="web-browser---modern-outlook"></a>[Navigateur web – Outlook moderne](#tab/modern)

#### <a name="shared-folders"></a>Dossiers partagés

Le propriétaire de la boîte aux lettres doit [d’abord fournir l’accès à un délégué](https://www.microsoft.com/microsoft-365/blog/2013/09/04/configuring-delegate-access-in-outlook-web-app/) en mettant à jour les autorisations du dossier de boîte aux lettres. Le délégué doit ensuite suivre les instructions décrites dans la section « Ajouter la boîte aux lettres d’une autre personne à votre liste de dossiers dans Outlook Web App » de l’article Accéder à la boîte aux lettres [d’une](https://support.microsoft.com/office/access-another-person-s-mailbox-a909ad30-e413-40b5-a487-0ea70b763081)autre personne.

#### <a name="shared-mailboxes-preview"></a>Boîtes aux lettres partagées (aperçu)

Exchange administrateurs de serveur peuvent créer et gérer des boîtes aux lettres partagées pour des ensembles d’utilisateurs à accéder. Actuellement, [Exchange Online](/exchange/collaboration-exo/shared-mailboxes) est la seule version de serveur prise en charge pour cette fonctionnalité.

Après avoir reçu l’accès, un utilisateur de boîte aux lettres partagée doit suivre les étapes décrites dans la section « Ajouter la boîte aux lettres partagée afin qu’elle s’affiche sous votre boîte aux lettres principale » de l’article Ouvrir et utiliser une boîte aux lettres partagée dans [Outlook sur le web](https://support.microsoft.com/office/open-and-use-a-shared-mailbox-in-outlook-on-the-web-98b5a90d-4e38-415d-a030-f09a4cd28207).

> [!WARNING]
> **N’utilisez PAS** d’autres options telles que « Ouvrir une autre boîte aux lettres ». Il se peut que les API de fonctionnalité ne fonctionnent pas correctement.

---

Pour en savoir plus sur l’endroit où les modules sont activés et non activés en général, reportez-vous à la section Éléments de boîte aux lettres disponibles pour les [add-ins](outlook-add-ins-overview.md#mailbox-items-available-to-add-ins) de la page de vue d’ensemble des Outlook.

## <a name="supported-permissions"></a>Autorisations prise en charge

Le tableau suivant décrit les autorisations que l’API JavaScript Office prend en charge pour les délégués et les utilisateurs de boîtes aux lettres partagées.

|Autorisation|Valeur|Description|
|---|---:|---|
|Lecture|1 (000001)|Peut lire des éléments.|
|Write|2 (000010)|Peut créer des éléments.|
|DeleteOwn|4 (000100)|Peut supprimer uniquement les éléments qu’ils ont créés.|
|DeleteAll|8 (001000)|Peut supprimer tous les éléments.|
|EditOwn|16 (010000)|Peut modifier uniquement les éléments qu’ils ont créés.|
|EditAll|32 (100000)|Peut modifier n’importe quel objet.|

> [!NOTE]
> Actuellement, l’API prend en charge l’obtention d’autorisations existantes, mais pas la définition d’autorisations.

[L’objet DelegatePermissions est](/javascript/api/outlook/office.mailboxenums.delegatepermissions) implémenté à l’aide d’un masque de bits pour indiquer les autorisations. Chaque position dans le masque de bits représente une autorisation particulière et si elle est définie sur, l’utilisateur dispose de `1` l’autorisation respective. Par exemple, si le deuxième bit à partir de la droite est `1` , l’utilisateur dispose de **l’autorisation d’écriture.** Vous pouvez voir un exemple de vérification d’une autorisation spécifique dans la [section](#perform-an-operation-as-delegate-or-shared-mailbox-user) Effectuer une opération en tant que délégué ou utilisateur de boîte aux lettres partagée plus loin dans cet article.

## <a name="sync-across-shared-folder-clients"></a>Synchronisation entre les clients de dossiers partagés

Les mises à jour d’un délégué vers la boîte aux lettres du propriétaire sont généralement synchronisées immédiatement entre les boîtes aux lettres.

Toutefois, si des opérations REST ou Exchange Web Services (EWS) ont été utilisées pour définir une propriété étendue sur un élément, la synchronisation de ces modifications peut prendre quelques heures. Nous vous recommandons plutôt d’utiliser [l’objet CustomProperties](/javascript/api/outlook/office.customproperties) et les API associées pour éviter un tel délai. Pour en savoir plus, consultez la [section](metadata-for-an-outlook-add-in.md#custom-data-per-item-in-a-mailbox-custom-properties) des propriétés personnalisées de l’article « Obtenir et définir des métadonnées dans un Outlook de données ».

> [!IMPORTANT]
> Dans un scénario de délégué, vous ne pouvez pas utiliser EWS avec les jetons actuellement fournis par office.js API.

## <a name="configure-the-manifest"></a>Configurer le manifeste

Pour activer les dossiers partagés et les scénarios de boîtes aux lettres partagées dans votre add-in, vous devez définir l’élément [SupportsSharedFolders](../reference/manifest/supportssharedfolders.md) dans le manifeste sous `true` l’élément `DesktopFormFactor` parent. Pour l’instant, les autres facteurs de forme ne sont pas pris en charge.

Pour prendre en charge les appels REST d’un délégué, définissez le nœud [Autorisations](../reference/manifest/permissions.md) dans le manifeste sur `ReadWriteMailbox` .

L’exemple suivant montre `SupportsSharedFolders` l’ensemble `true` d’éléments dans une section du manifeste.

```XML
...
<VersionOverrides xmlns="http://schemas.microsoft.com/office/mailappversionoverrides" xsi:type="VersionOverridesV1_0">
  <VersionOverrides xmlns="http://schemas.microsoft.com/office/mailappversionoverrides/1.1" xsi:type="VersionOverridesV1_1">
    ...
    <Hosts>
      <Host xsi:type="MailHost">
        <DesktopFormFactor>
          <SupportsSharedFolders>true</SupportsSharedFolders>
          <FunctionFile resid="residDesktopFuncUrl" />
          <ExtensionPoint xsi:type="MessageReadCommandSurface">
            <!-- configure selected extension point -->
          </ExtensionPoint>

          <!-- You can define more than one ExtensionPoint element as needed -->

        </DesktopFormFactor>
      </Host>
    </Hosts>
    ...
  </VersionOverrides>
</VersionOverrides>
...
```

## <a name="perform-an-operation-as-delegate-or-shared-mailbox-user"></a>Effectuer une opération en tant qu’utilisateur de boîte aux lettres déléguée ou partagée

Vous pouvez obtenir les propriétés partagées d’un élément en mode Composition ou Lecture en appelant la méthode [item.getSharedPropertiesAsync.](../reference/objectmodel/preview-requirement-set/office.context.mailbox.item.md#methods) Cela renvoie un [objet SharedProperties](/javascript/api/outlook/office.sharedproperties) qui fournit actuellement les autorisations de l’utilisateur, l’adresse e-mail du propriétaire, l’URL de base de l’API REST et la boîte aux lettres cible.

L’exemple suivant montre comment obtenir les propriétés partagées d’un message  ou d’un rendez-vous, vérifier si le délégué ou l’utilisateur de boîte aux lettres partagée dispose d’une autorisation d’écriture et passer un appel REST.

```js
function performOperation() {
  Office.context.mailbox.getCallbackTokenAsync({
      isRest: true
    },
    function (asyncResult) {
      if (asyncResult.status === Office.AsyncResultStatus.Succeeded && asyncResult.value !== "") {
        Office.context.mailbox.item.getSharedPropertiesAsync({
            // Pass auth token along.
            asyncContext: asyncResult.value
          },
          function (asyncResult1) {
            let sharedProperties = asyncResult1.value;
            let delegatePermissions = sharedProperties.delegatePermissions;

            // Determine if user can do the expected operation.
            // E.g., do they have Write permission?
            if ((delegatePermissions & Office.MailboxEnums.DelegatePermissions.Write) != 0) {
              // Construct REST URL for your operation.
              // Update <version> placeholder with actual Outlook REST API version e.g. "v2.0".
              // Update <operation> placeholder with actual operation.
              let rest_url = sharedProperties.targetRestUrl + "/<version>/users/" + sharedProperties.targetMailbox + "/<operation>";
  
              $.ajax({
                  url: rest_url,
                  dataType: 'json',
                  headers:
                  {
                    "Authorization": "Bearer " + asyncResult1.asyncContext
                  }
                }
              ).done(
                function (response) {
                  console.log("success");
                }
              ).fail(
                function (error) {
                  console.log("error message");
                }
              );
            }
          }
        );
      }
    }
  );
}
```

> [!TIP]
> En tant que délégué, vous pouvez utiliser REST pour obtenir le contenu d’un message Outlook joint à un élément Outlook ou un [billet de groupe.](/graph/outlook-get-mime-message#get-mime-content-of-an-outlook-message-attached-to-an-outlook-item-or-group-post)

## <a name="handle-calling-rest-on-shared-and-non-shared-items"></a>Gérer l’appel de REST sur les éléments partagés et non partagés

Si vous souhaitez appeler une opération REST sur un élément, que l’élément soit partagé ou non, vous pouvez utiliser l’API pour déterminer si l’élément `getSharedPropertiesAsync` est partagé. Après cela, vous pouvez construire l’URL REST pour l’opération à l’aide de l’objet approprié.

```js
if (item.getSharedPropertiesAsync) {
  // In Windows, Mac, and the web client, this indicates a shared item so use SharedProperties properties to construct the REST URL.
  // Add-ins don't activate on shared items in mobile so no need to handle.

  // Perform operation for shared item.
} else {
  // In general, this is not a shared item, so construct the REST URL using info from the Call REST APIs article:
  // https://docs.microsoft.com/office/dev/add-ins/outlook/use-rest-api

  // Perform operation for non-shared item.
}
```

## <a name="limitations"></a>Limites

Selon les scénarios de votre add-in, il existe quelques limitations à prendre en compte lors de la gestion des situations de dossier partagé ou de boîte aux lettres partagée.

### <a name="message-compose-mode"></a>Mode composition de message

En mode composition de message, [getSharedPropertiesAsync](/javascript/api/outlook/office.messagecompose#getSharedPropertiesAsync_options__callback_) n’est pas pris en charge dans Outlook sur le web ou sur Windows à moins que les conditions suivantes ne soient remplies.

a. **Accès délégué/Dossiers partagés**

1. Le propriétaire de la boîte aux lettres démarre un message. Il peut s’agit d’un nouveau message, d’une réponse ou d’un forward.
1. Ils enregistrent le message, puis le déplacent de leur propre dossier **Brouillons** vers un dossier partagé avec le délégué.
1. Le délégué ouvre le brouillon à partir du dossier partagé, puis continue la composition.

b. **Boîte aux lettres partagée**

1. Un utilisateur de boîte aux lettres partagée démarre un message. Il peut s’agit d’un nouveau message, d’une réponse ou d’un forward.
1. Ils enregistrent le message, puis le déplacent de leur propre dossier **Brouillons** vers un dossier de la boîte aux lettres partagée.
1. Un autre utilisateur de boîte aux lettres partagée ouvre le brouillon à partir de la boîte aux lettres partagée, puis continue la composition.

Le message se trouve maintenant dans un contexte partagé et les modules qui la prisent en charge de ces scénarios partagés peuvent obtenir les propriétés partagées de l’élément. Une fois le message envoyé, il se trouve généralement  dans le dossier Éléments envoyés de l’expéditeur.

### <a name="rest-and-ews"></a>REST et EWS

Votre application peut utiliser REST et son autorisation doit être définie pour activer l’accès REST à la boîte aux lettres du propriétaire ou à la boîte aux lettres partagée, le `ReadWriteMailbox` cas échéant. EWS n’est pas pris en charge.

### <a name="user-or-shared-mailbox-hidden-from-an-address-list"></a>Boîte aux lettres utilisateur ou partagée masquée dans une liste d’adresses

Si un administrateur a caché un utilisateur ou une adresse de boîte aux lettres partagée à partir d’une liste d’adresses telle que la liste d’adresses globale ,les éléments de courrier affectés ouverts dans le rapport de boîte aux lettres sont `Office.context.mailbox.item` null. Par exemple, si l’utilisateur ouvre un élément de courrier dans une boîte aux lettres partagée qui est masquée dans la liste d’adresses gal, représentant cet élément de `Office.context.mailbox.item` courrier est null.

## <a name="see-also"></a>Voir aussi

- [Autoriser quelqu’un d’autre à gérer votre courrier et votre calendrier](https://support.office.com/article/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926)
- [Partage de calendrier dans Microsoft 365](https://support.office.com/article/calendar-sharing-in-office-365-b576ecc3-0945-4d75-85f1-5efafb8a37b4)
- [Ajouter une boîte aux lettres partagée à Outlook](/microsoft-365/admin/email/create-a-shared-mailbox?view=o365-worldwide&preserve-view=true#add-the-shared-mailbox-to-outlook)
- [Comment commander des éléments de manifeste](../develop/manifest-element-ordering.md)
- [Masque (calcul)](https://en.wikipedia.org/wiki/Mask_(computing))
- [Opérateurs de bits JavaScript](https://www.w3schools.com/js/js_bitwise.asp)