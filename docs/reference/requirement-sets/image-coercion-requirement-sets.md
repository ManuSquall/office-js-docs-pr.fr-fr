---
title: Ensembles de conditions requises de coercition d’image
description: Prise en charge des ensembles de conditions requises pour le foragage d’image avec Office pour les Excel, PowerPoint et Word.
ms.date: 02/19/2021
ms.prod: non-product-specific
localization_priority: Normal
ms.openlocfilehash: 29614718378fd51013360a2a922e11f89bca14b8
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53350217"
---
# <a name="image-coercion-requirement-sets"></a>Ensembles de conditions requises de coercition d’image

Les ensembles de conditions requises sont des groupes nommés des membres de l’API. Les compléments Office utilisent les ensembles de conditions requises spécifiés dans le manifeste ou utilisent une vérification de l’exécution pour déterminer si une application Office prend en charge les API requises par un complément. Pour plus d’informations, consultez la rubrique [Versions d’Office et ensembles de conditions requises](../../develop/office-versions-and-requirement-sets.md).

## <a name="imagecoercion-11"></a>ImageCoercion 1.1

ImageCoercion 1.1 permet la conversion en image ( ) lors de l’écriture de données `Office.CoercionType.Image` à l’aide de la [`Document.setSelectedDataAsync`](/javascript/api/office/office.document#setselecteddataasync-data--options--callback-) méthode. Les applications suivantes sont pris en charge.

- Excel 2013 et les ultérieures Windows
- Excel 2016 et ultérieures sur Mac
- Excel sur iPad
- OneNote sur le web
- PowerPoint 2013 et les Windows
- PowerPoint 2016 et ultérieures sur Mac
- PowerPoint sur le web
- PowerPoint sur iPad
- Word 2013 ou version ultérieure sur Windows
- Word 2016 ou version ultérieure sur Mac
- Word sur le web
- Word sur iPad

## <a name="imagecoercion-12"></a>ImageCoercion 1.2

ImageCoercion 1.2 permet la conversion au format SVG () lors de l’écriture de données `Office.CoercionType.XmlSvg` à l’aide de la [`Document.setSelectedDataAsync`](/javascript/api/office/office.document#setselecteddataasync-data--options--callback-) méthode. Les applications suivantes sont pris en charge.

- Excel sur Windows (connecté à un abonnement Microsoft 365 abonnement)
- Excel mac (connecté à un abonnement Microsoft 365))
- PowerPoint sur Windows (connecté à un abonnement Microsoft 365 abonnement)
- PowerPoint mac (connecté à un abonnement Microsoft 365))
- PowerPoint sur le web
- Word on Windows (connecté à un abonnement Microsoft 365))
- Word sur Mac (connecté à un abonnement Microsoft 365))

## <a name="office-common-api-requirement-sets"></a>Séries de conditions requises des API communes pour Office

Pour plus d’informations sur les ensembles de conditions requises des API communes, voir [Ensembles de conditions requises des API communes pour Office](office-add-in-requirement-sets.md).

## <a name="see-also"></a>Voir aussi

- [Versions d’Office et ensembles de conditions requises](../../develop/office-versions-and-requirement-sets.md)
- [Spécifier les exigences en matière d’applications Office et d’API](../../develop/specify-office-hosts-and-api-requirements.md)
- [Manifeste XML des compléments Office](../../develop/add-in-manifests.md)
