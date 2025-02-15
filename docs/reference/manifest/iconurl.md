---
title: Élément IconUrl dans le fichier manifeste
description: L’élément IconUrl spécifie l’URL de l’image qui représente votre add-in Office dans l’UX d’insertion et l’Office Store.
ms.date: 03/30/2021
localization_priority: Normal
ms.openlocfilehash: 68a449b40f6084d26140d59fec61967e163196df
ms.sourcegitcommit: 0bff0411d8cfefd4bb00c189643358e6fb1df95e
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/07/2021
ms.locfileid: "51604638"
---
# <a name="iconurl-element"></a>IconUrl, élément

Spécifie l’URL de l’image utilisée pour représenter votre complément Office dans l’UX d’insertion UX et l’Office Store.

**Type de complément :** application de contenu, de volet Office, de messagerie

## <a name="syntax"></a>Syntaxe

```XML
<IconUrl DefaultValue="string" />
```

## <a name="can-contain"></a>Peut contenir

[Override](override.md)

## <a name="attributes"></a>Attributs

|Attribut|Type|Requis|Description|
|:-----|:-----|:-----|:-----|
|DefaultValue|chaîne|obligatoire|Spécifie la valeur par défaut de ce paramètre, exprimée pour les paramètres régionaux spécifiés dans l’élément [DefaultLocale](defaultlocale.md).|

## <a name="remarks"></a>Remarques

Pour un module de messagerie, l’icône s’affiche dans l’interface utilisateur de gestion des fichiers (Outlook) ou dans l’interface utilisateur de gestion des paramètres des applications  >     >   (Outlook sur le web). Pour un complément de contenu ou de volet Office, l’icône s’affiche dans l’interface utilisateur, sous **Insérer** > **Compléments**. Pour tous les types de modules, l’icône est également utilisée dans [AppSource,](https://appsource.microsoft.com)si vous publiez votre application dans AppSource.

L’image doit être dans un des formats de fichier suivants : GIF, JPG, PNG, EXIF, BMP ou TIFF. Pour les applications de volet de tâches et de contenu, l’image spécifiée doit contenir 32 x 32 pixels. Pour les applications de messagerie, la résolution d’image doit être de 64 x 64 pixels. Vous devez également spécifier une icône à utiliser avec les applications clientes Office qui s’exécutent sur des écrans HAUTESPI à l’aide de l’élément [HighResolutionIconUrl.](highresolutioniconurl.md) Pour plus d’informations, reportez-vous à la section _Créer une identité visuelle cohérente pour votre application_ dans [Création de listings efficaces dans AppSource et dans Office](/office/dev/store/create-effective-office-store-listings#create-a-consistent-visual-identity).

La modification de la valeur de `IconUrl` l’élément au moment de l’runtime n’est pas prise en charge actuellement.
