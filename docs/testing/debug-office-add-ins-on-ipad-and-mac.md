---
title: Déboguer des compléments Office sur un Mac
description: Découvrez comment utiliser un Mac pour déboguer des Office des macros.
ms.date: 10/16/2020
localization_priority: Normal
ms.openlocfilehash: 98473e7c37b9ef5ee34d35f91688ccef65ac7d78
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53350133"
---
# <a name="debug-office-add-ins-on-a-mac"></a>Déboguer des compléments Office sur un Mac

Étant donné que les compléments sont développés avec du code HTML et JavaScript, ils sont conçus pour fonctionner sur toutes les plateformes, mais il peut y avoir de subtiles différences dans le rendu du code HTML par les différents navigateurs. Cet article décrit la procédure de débogage des compléments qui s’exécutent sur un Mac.

## <a name="debugging-with-safari-web-inspector-on-a-mac"></a>Débogage avec l’inspecteur web Safari sur Mac

Si votre complément affiche une interface utilisateur dans un volet des tâches ou dans un complément de contenu, vous pouvez déboguer un complément Office à l’aide de avec l’inspecteur web Safari.

Pour pouvoir déboguer des Office sur Mac, vous devez avoir Mac OS High Sierra et Mac Office version 16.9.1 (build 18012504) ou version ultérieure. Si vous n’avez pas de build Office Mac, vous pouvez en obtenir une en rejoignant le programme [Microsoft 365 développeur.](https://developer.microsoft.com/office/dev-program)

Pour commencer, ouvrez un terminal, puis définissez la propriété `OfficeWebAddinDeveloperExtras` pour l’application Office pertinente comme suit :

- `defaults write com.microsoft.Word OfficeWebAddinDeveloperExtras -bool true`

- `defaults write com.microsoft.Excel OfficeWebAddinDeveloperExtras -bool true`

- `defaults write com.microsoft.Powerpoint OfficeWebAddinDeveloperExtras -bool true`

- `defaults write com.microsoft.Outlook OfficeWebAddinDeveloperExtras -bool true`

    > [!IMPORTANT]
    > Les builds d’applications du Mac App Store Office ne pas la prise en charge de `OfficeWebAddinDeveloperExtras` l’indicateur.

Ensuite, ouvrez l’application Office et[insérez votre complément](sideload-an-office-add-in-on-ipad-and-mac.md). Cliquez sur le complément. Vous devriez voir l’option **Inspecter l’élément** s’afficher dans le menu contextuel. Sélectionnez cette option pour afficher l’inspecteur dans lequel vous pouvez définir des points d’arrêt et déboguer votre complément.

> [!NOTE]
> Si vous essayez d’utiliser l’inspecteur et si la boîte de dialogue scintille, mettez Office à jour vers la dernière version. Si cela ne résout pas le clignotement, essayez la solution de contournement suivante.
>
> 1. Pour réduire la taille de la boîte de dialogue.
> 1. Sélectionnez l’option **Inspecter l’élément** qui ouvre une nouvelle fenêtre.
> 1. Redimensionner la boîte de dialogue à sa taille d’origine.
> 1. Utiliser l’inspecteur comme requis.

## <a name="clearing-the-office-applications-cache-on-a-mac"></a>Effacement du cache de l’application Office sur un ordinateur Mac

[!include[additional cache folders on Mac](../includes/mac-cache-folders.md)]
