---
title: Authentifier et autoriser avec l’API de dialogue Office
description: Découvrez comment utiliser l’API de boîte de dialogue Office pour permettre aux utilisateurs de se connecter à Google, Facebook, Microsoft 365 ainsi qu'à d’autres services protégés par la plateforme Microsoft Identity.
ms.date: 09/24/2020
localization_priority: Priority
ms.openlocfilehash: 85576c50c69332e16c0636586461392021b2e2a4
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53350049"
---
# <a name="authenticate-and-authorize-with-the-office-dialog-api"></a>Authentifier et autoriser avec l’API de dialogue Office

De nombreuses autorités d’identité, également appelées service d’émission de jeton de sécurité (STS), empêchent leur page de connexion de s’ouvrir dans un IFRAME. Il s'agit notamment de Google, Facebook et des services protégés par Microsoft Identity Platform (anciennement Azure AD V 2.0) tels qu'un compte Microsoft, un compte Microsoft 365 Éducation ou travail, ou tout autre compte commun. Cela a pour effet de créer un problème pour les compléments Office, car lorsque le complément est exécuté dans **Office sur le Web**, le volet Office est un IFRAME. Les utilisateurs d’un complément peuvent se connecter à l’un de ces services uniquement si le complément peut ouvrir une instance de navigateur entièrement distincte. C’est la raison pour laquelle Office fournit son [API de boîte de dialogue Office](dialog-api-in-office-add-ins.md), à savoir la méthode [displayDialogAsync](/javascript/api/office/office.ui).

> [!NOTE]
> Cet article part du principe que vous êtes familiarisé(e) avec [Utiliser l’API de boîte de dialogue Office](dialog-api-in-office-add-ins.md) dans vos compléments Office.

La boîte de dialogue ouverte avec cette API présente les caractéristiques suivantes.

- C'est [non modal](https://en.wikipedia.org/wiki/Dialog_box).
- Il s’agit d’une instance de navigateur totalement distincte du volet de tâches, ce qui signifie :
  - Elle possède ses propres environnements d’exécution JavaScript et objets de fenêtre et variables globales.
  - Il n’existe pas d’environnement d’exécution partagé dans le volet des tâches.
  - Il ne partage pas le même espace de stockage de session (la propriété [Window.sessionStorage](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage)) que le volet Office.
- La première page ouverte dans la boîte de dialogue doit être hébergée dans le même domaine que le volet des tâches, y compris le protocole, les sous-domaines et le port, le cas échéant.
- La boîte de dialogue peut renvoyer les informations au volet des tâches à l’aide de la méthode [messageParent](/javascript/api/office/office.ui#messageparent-message-), mais cette méthode ne peut être appelée que depuis une page hébergée dans le même domaine que le volet des tâches, y compris le protocole, les sous-domaines et le port.

Lorsque la boîte de dialogue n’est pas un IFRAME (qui est la valeur par défaut), elle peut ouvrir la page de connexion d’un fournisseur d’identité. Comme vous le verrez dans la section ci-dessous, les caractéristiques de la boîte de dialogue Office ont une incidence sur la manière dont vous utilisez les bibliothèques d’authentification ou d’autorisation telles que MSAL et Passport.

> [!NOTE]
> Vous pouvez configurer la boîte de dialogue pour qu’elle s’ouvre dans un IFRAME flottant : vous pouvez simplement transmettre l’ `displayInIframe: true`option dans l’appel à`displayDialogAsync`. Ne le faites *pas* lorsque vous utilisez l’API de boîte de dialogue Office pour la connexion.

## <a name="authentication-flow-with-the-office-dialog-box"></a>Flux d’authentification avec la boîte de dialogue Office

Voici un flux d’authentification simple et standard. Les détails sont répertoriés après le diagramme.

![Diagramme montrant la relation entre le volet des tâches et les processus de navigation des boîtes de dialogue.](../images/taskpane-dialog-processes.gif)

1. La première page qui s’ouvre dans la boîte de dialogue est une page (ou toute autre ressource) qui est hébergée dans le domaine du complément ; autrement dit, le même domaine que la fenêtre du volet des tâches. Cette page peut avoir une IU simple indiquant « Veuillez patienter, nous allons vous rediriger vers la page sur laquelle vous pouvez vous connecter à *NOM DU FOURNISSEUR* ». Le code dans cette page construit l’URL de la page de connexion du fournisseur d’identité en utilisant les informations transmises à la boîte de dialogue, comme décrit dans [Transmission d’informations à la boîte de dialogue](dialog-api-in-office-add-ins.md#pass-information-to-the-dialog-box) ou est codée en dur dans un fichier de configuration du complément, tel qu’un fichier web.config.
2. La fenêtre de la boîte de dialogue redirige alors l’utilisateur vers la page de connexion. L’URL inclut un paramètre de requête qui indique au fournisseur d’identité de rediriger la fenêtre de la boîte de dialogue une fois que l’utilisateur s’est connecté à une page spécifique. Dans cet article, nous appellerons cette page **redirectPage.html**. *Il doit s’agir d’une page se trouvant dans le même domaine que la fenêtre hôte*, afin que les résultats de la tentative de connexion puissent être transférés au volet des tâches avec un appel de`messageParent`.
3. Le service du fournisseur d’identité traite la requête GET entrante à partir de la fenêtre de la boîte de dialogue. Si l’utilisateur est déjà connecté, il redirige immédiatement la fenêtre vers **redirectPage.html** et inclut les données utilisateur sous la forme d’un paramètre de requête. Si l’utilisateur n’est pas encore connecté, la page de connexion du fournisseur apparaît dans la fenêtre et l’utilisateur se connecte. Pour la plupart des fournisseurs, si l’utilisateur ne parvient pas à se connecter, le fournisseur affiche une page d’erreur dans la fenêtre de la boîte de dialogue et ne redirige pas vers **redirectPage.html**. L’utilisateur doit fermer la fenêtre en sélectionnant le **X** dans le coin. Si l’utilisateur se connecte avec succès, la fenêtre de la boîte de dialogue est redirigée vers **redirectPage.html** et les données utilisateur sont incluses sous la forme d’un paramètre de requête.
4. Lorsque la page **redirectPage.html** s’ouvre, elle appelle`messageParent` pour indiquer le succès ou l’échec au volet des tâches et éventuellement indiquer également des données utilisateur ou des données d’erreur. Les autres messages possibles incluent le passage d’un jeton d’accès ou le volet des tâches dans lequel le jeton est stocké.
5. L’événement `DialogMessageReceived` se déclenche dans le volet des tâches, et son gestionnaire ferme la fenêtre de la boîte de dialogue et effectue éventuellement d’autres traitements du message.

#### <a name="support-multiple-identity-providers"></a>Prise en charge de plusieurs fournisseurs d’identité

Si votre complément offre à l’utilisateur le choix entre plusieurs fournisseurs, tels qu’un compte Microsoft, Google ou Facebook, vous avez besoin d’une première page locale (voir section précédente) qui fournit une interface utilisateur permettant à l’utilisateur de sélectionner un fournisseur. La sélection déclenche la construction de l’URL de connexion et la redirection vers celle-ci.

#### <a name="authorization-of-the-add-in-to-an-external-resource"></a>Autorisation du complément pour une ressource externe

Sur le web nouvelle génération, les applications web sont des principaux de sécurité. L’application a sa propre identité et ses propres autorisations pour une ressource en ligne comme Microsoft 365, Google Plus, Facebook ou LinkedIn. L’application est inscrite auprès du fournisseur de ressources avant d’être déployée. L’inscription inclut :

- La liste des autorisations dont l’application a besoin.
- l’URL à laquelle le service de ressources doit renvoyer un jeton d’accès lorsque l’application accède au service.  

Lorsqu’un utilisateur appelle une fonction dans l’application qui accède aux données de l’utilisateur dans le service de ressources, l’utilisateur est invité à se connecter au service, puis à accorder à l’application les autorisations dont elle a besoin pour les ressources de l’utilisateur. Ensuite, le service redirige la fenêtre de connexion vers l’URL précédemment inscrite et transmet le jeton d’accès. L’application utilise le jeton d’accès pour accéder aux ressources de l’utilisateur.

Vous pouvez utiliser les API de dialogue Office pour gérer ce processus à l’aide d’un flux semblable à celui décrit pour la connexion des utilisateurs. Les seules différences sont les suivantes :

- Si l’utilisateur n’a pas préalablement accordé à l’application les autorisations nécessaires, il est invité à le faire dans la boîte de dialogue après la connexion.
- La fenêtre de la boîte de dialogue envoie le jeton d’accès vers la fenêtre hôte en utilisant `messageParent` pour envoyer le jeton d’accès converti en chaîne ou en stockant jeton d’accès à un emplacement où la fenêtre hôte peut le récupérer (et utilise `messageParent` pour indiquer à la fenêtre hôte que le jeton est disponible). Le jeton a une limite de temps, mais tant qu’elle n’est pas écoulée, la fenêtre hôte peut l’utiliser pour accéder directement aux ressources de l’utilisateur sans demander d’autre confirmation.

Quelques exemples de compléments d’authentification qui utilisent l’API de boîte de dialogue Office à cet effet sont répertoriés dans les [exemples](#samples).

## <a name="using-authentication-libraries-with-the-dialog-box"></a>Utilisation de bibliothèques d’authentification avec la boîte de dialogue

Le fait que la boîte de dialogue Office et le volet des tâches s’exécutent dans différents navigateurs, et instances JavaScript Runtime, signifie que vous devez utiliser de nombreuses bibliothèques d’authentification et d’autorisation de manière différente que celle utilisée lorsque l’authentification et l’autorisation peuvent être effectuées dans la même fenêtre. Les sections suivantes décrivent les principales façons dont vous ne pouvez généralement pas utiliser ces bibliothèques et la *manière* de les utiliser.

### <a name="you-usually-cannot-use-the-librarys-internal-cache-to-store-tokens"></a>En général, vous ne pouvez pas utiliser le cache interne de la bibliothèque pour stocker des jetons

En règle générale, les bibliothèques associées à l’authentification fournissent un cache en mémoire pour stocker le jeton d’accès. Si des appels ultérieurs au fournisseur de ressources (par exemple, Google, Microsoft Graph, Facebook, etc.) sont apportés, la bibliothèque vérifie tout d’abord si le jeton dans son cache a expiré. Si celui-ci n’a pas expiré, la bibliothèque renvoie le jeton mis en cache plutôt que d’effectuer un autre aller-retour vers le SJS pour un nouveau jeton. Mais ce modèle n’est pas utilisable dans les compléments Office. Dans la mesure où la connexion a lieu dans l’instance du navigateur de la boîte de dialogue Office, le cache du jeton est dans cette instance.

Ceci est lié au fait qu’une bibliothèque fournit généralement des méthodes à la fois interactives et «silencieuses» pour obtenir un jeton. Lorsque vous pouvez effectuer les deux appels d’authentification et de données à la ressource dans la même instance de navigateur, votre code appelle la méthode silencieuse pour obtenir un jeton juste avant que votre code n’ajoute le jeton à l’appel de données. La méthode silencieuse vérifie la présence d’un jeton non expiré dans le cache et le renvoie, le cas échéant. Dans le cas contraire, la méthode silencieuse appelle la méthode interactive qui redirige vers la connexion de STS. Une fois la connexion terminée, la méthode interactive renvoie le jeton, mais le met en cache dans la mémoire. En revanche, lorsque l’API de boîte de dialogue Office est utilisée, les données appellent la ressource, qui appellent la méthode silencieuse, se trouvent dans l’instance de navigateur du volet des tâches. Le cache de jetons de la bibliothèque n’existe pas dans cette instance.

En guise d’alternative, l’instance de navigateur de la boîte de dialogue de votre complément peut appeler directement la méthode interactive de la bibliothèque. Lorsque cette méthode renvoie un jeton, votre code doit stocker de manière explicite le jeton à l’endroit où l’instance de navigateur du volet des tâches peut le récupérer (par exemple, stockage local\* ou une base de données côté serveur). Une autre option consiste à transmettre le jeton au volet des tâches avec la méthode`messageParent`. Cette alternative est uniquement possible si la méthode interactive stocke le jeton d’accès à un endroit où votre code peut le lire. Parfois, la méthode interactive d’une bibliothèque est conçue pour stocker le jeton dans une propriété privée d’un objet qui n’est pas accessible à votre code.

> [!NOTE]
> \* Un bogue peut affecter votre stratégie de gestion des jetons. Si le complément s’exécute dans **Office sur le web** dans le navigateur Safari ou Edge, la boîte de dialogue et le volet des tâches Office ne partagent pas le même stockage local, il ne peut donc pas être utilisé pour communiquer entre eux.

### <a name="you-usually-cannot-use-the-librarys-auth-context-object"></a>En général, vous ne pouvez pas utiliser l’objet «contexte d’authentification» de la bibliothèque.

Il arrive souvent qu’une bibliothèque liée à l’authentification ait une méthode qui récupère un jeton de façon interactive et crée également un objet «contexte d’authentification» que la méthode renvoie. Le jeton est une propriété de l’objet (potentiellement privé et inaccessible directement à partir de votre code). Cet objet possède les méthodes pour recevoir les données de la ressource. Ces méthodes incluent le jeton dans les requêtes HTTP qu’ils font au fournisseur de ressources (par exemple, Google, Microsoft Graph, Facebook, etc.).

Ces objets de contexte d’authentification, ainsi que les méthodes qui les créent, ne sont pas utilisables dans les compléments Office. Dans la mesure où la connexion a lieu dans l’instance du navigateur de la boîte de dialogue Office, l’objet devrait être créé à cet emplacement. Mais les appels de données à la ressource se trouvent dans l’instance de navigateur du volet des tâches et il n’est pas possible d’utiliser l’objet d’une instance à l’autre. Par exemple, vous ne pouvez pas passer l'objet avec`messageParent` car `messageParent`peut uniquement transmettre des valeurs de chaînes. Un objet JavaScript avec des méthodes ne peut pas être mis en chaîne de façon fiable.

### <a name="how-you-can-use-libraries-with-the-office-dialog-api"></a>Utilisation des bibliothèques avec l’API de boîte de dialogue Office

En plus ou au lieu de, des objets «contexte d’authentification» monolithiques, la plupart des bibliothèques fournissent des API à un niveau d’abstraction inférieur qui permettent à votre code de créer moins d’objets d’assistance monolithiques. Par exemple, [MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#conceptual-documentation) v. 3. x. x a une API pour créer une URL de connexion, et une autre API qui crée un objet AuthResult qui contient un jeton d’accès dans une propriété accessible à votre code. Pour consulter des exemples d’MSAL.net dans un complément Office, voir :[complément Office Microsoft Graph ASP.NET](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Office-Add-in-Microsoft-Graph-ASPNET) et [complément Outlook Microsoft Graph ASP.NET](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Outlook-Add-in-Microsoft-Graph-ASPNET). Pour obtenir un exemple d’utilisation [msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js) dans un complément, voir [complément Office Microsoft Graph React](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Office-Add-in-Microsoft-Graph-React).

Pour plus d’informations sur les bibliothèques d’authentification et d’autorisation, voir [Microsoft Graph : bibliothèques recommandées](authorize-to-microsoft-graph-without-sso.md#recommended-libraries-and-samples) et [autres services externes : bibliothèques](auth-external-add-ins.md#libraries).

## <a name="samples"></a>Exemples

- [Complément Office Microsoft Graph ASP.NET](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Office-Add-in-Microsoft-Graph-ASPNET): complément ASP.net (Excel, Word, ou PowerPoint) qui utilise la bibliothèque MSAL.net et le flux de code d’autorisation pour se connecter et obtenir un jeton d’accès pour les données Microsoft Graph.
- [Complément Outlook Microsoft Graph ASP.NET](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Outlook-Add-in-Microsoft-Graph-ASPNET): comme celui ci-dessus, mais l’application Office est Outlook.
- [Complément Office Microsoft Graph React](https://github.com/OfficeDev/PnP-OfficeAddins/tree/master/Samples/auth/Office-Add-in-Microsoft-Graph-React): complément NodeJS (Excel, Word ou PowerPoint) qui utilise la bibliothèque msal.js et le flux implicite pour se connecter et obtenir un jeton d’accès pour les données Microsoft Graph.


Pour plus d’informations, voir :
- [Autoriser des services externes dans votre complément Office](auth-external-add-ins.md)
- [Utiliser l’API de boîte de dialogue Office dans vos compléments Office](dialog-api-in-office-add-ins.md)
