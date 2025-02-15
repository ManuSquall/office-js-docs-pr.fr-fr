---
title: Recommandations en matière d’icônes de style Office des produits
description: Recommandations en matière d’utilisation d’icônes de style Office dans les add-ins.
ms.date: 05/12/2021
localization_priority: Normal
ms.openlocfilehash: e7a06ec25f82215a402bc5eb7fc74fa39430e227
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53349314"
---
# <a name="fresh-style-icon-guidelines-for-office-add-ins"></a>Recommandations en matière d’icônes de style Office des produits

Les Office versions 2013 et 2013 (sans abonnement) Office l’iconographie du style Fresh de Microsoft. Si vous préférez que vos icônes correspondent au style monoligne de Microsoft 365, consultez les instructions relatives aux icônes de style Monoline pour Office [des modules.](add-in-icons-monoline.md)

## <a name="office-fresh-visual-style"></a>Office Style visuel à nouveau

Les icônes Fresh incluent uniquement les éléments de communication essentiels. Les éléments non essentiels, tels que la source de lumière, les dégradés et les perspectives, sont supprimés. Les icônes simplifiées prennent en charge l’analyse rapide des commandes et des contrôles. Suivez ce style pour mieux s’adapter Office clients sans abonnement.

## <a name="best-practices"></a>Meilleures pratiques

Suivez ces instructions lorsque vous créez vos icônes :

|À faire|À ne pas faire|
|:---|:---|
|Conservez des éléments visuels simples et clairs, en vous concentrant sur les éléments clés de la communication.| N’utilisez pas d’artefacts qui rendent votre icône désordonnée.|
|Utilisez le langage d’icône Office pour représenter des comportements ou des concepts.|Ne réaffectez pas les glyphes Fabric Core pour les commandes de application Office du ruban ou des menus contextuels. Les icônes Fabric Core sont stylistiquement différentes et ne correspondent pas.|
|Réutilisez les métaphores visuelles d’Office courantes telles que le pinceau pour mettre en forme ou la loupe pour rechercher.|Ne réutilisez pas les métaphores visuelles pour différentes commandes. L’utilisation de la même icône pour différents comportements et concepts peut semer la confusion. |
|Redessinez vos icônes pour les réduire ou les agrandir. Prenez le temps de redessiner les découpages, les coins et des bords arrondis pour optimiser la netteté de ligne. |Ne redimensionnez pas vos icônes en réduisant ou en agrandissant leurs tailles. Cela peut entraîner une mauvaise qualité visuelle et des actions peu claires. Les icônes complexes créées dans une plus grande taille risquent de perdre en clarté si elles sont redimensionnées pour être réduites sans être redessinées. |
|Utilisez un remplissage blanc pour améliorer l’accessibilité. La plupart des objets dans les icônes nécessitent un arrière-plan blanc pour être lisibles sur les thèmes de l’interface utilisateur d’Office et en mode contraste élevé.  |Évitez de vous fier à votre logo ou marque pour communiquer ce que fait une commande de complément. Les repères de marque ne sont pas toujours reconnaissables sur des icônes de petites tailles et lorsque des modificateurs sont appliqués. Les marques de marque entrent souvent en conflit application Office styles d’icône du ruban et peuvent attirer l’attention des utilisateurs dans un environnement saturé. |
|Utilisez le format PNG avec un arrière-plan transparent. ||
|Évitez le contenu localisable dans les icônes, y compris les caractères typographiques, les paragraphes en drapeau et les points d’interrogation. ||

## <a name="icon-size-recommendations-and-requirements"></a>Configuration requise et recommandations sur la taille des icônes

Les icônes du bureau Office sont des images bitmap. Différentes tailles apparaissent en fonction du paramètre PPP de l’utilisateur et du mode tactile. Incluez les huit tailles prises en charge pour créer la meilleure expérience possible dans tous les contextes et résolutions pris en charge. Les tailles suivantes sont les suivantes : trois sont requises.

- 16 px (obligatoire)
- 20 px
- 24 px
- 32 px (obligatoire)
- 40 px
- 48 px
- 64 px (recommandé, meilleur choix pour Mac)
- 80 px (obligatoire)

> [!IMPORTANT]
> Pour obtenir une image représentant l’icône représentant votre application, voir Créer des listes efficaces dans [AppSource](/office/dev/store/create-effective-office-store-listings#create-an-icon-for-your-add-in) et dans Office pour la taille et d’autres exigences.

Veillez à renouveler les icônes pour chaque taille au lieu de les réduire pour les ajuster.

![Illustration de la recommandation pour redesscrire les icônes par taille plutôt que de réduire les icônes. Par exemple, vous devrez peut-être utiliser moins d’éléments dans une petite icône plutôt que simplement mettre à l’échelle une image plus grande.](../images/icon-resizing.png)

## <a name="icon-anatomy-and-layout"></a>Mise en page et structure de l’icône

Office icônes sont généralement composées d’un élément de base avec des modificateurs d’action et conceptuels superposés. Les modificateurs d’action représentent des concepts tels qu’ajouter, ouvrir, nouveau ou fermer. Les modificateurs conceptuels représentent l’état, l’altération ou une description de l’icône.

Pour créer des commandes qui s’alignent sur l’interface utilisateur d’Office, suivez les instructions de mise en forme pour les éléments de base et les modificateurs. Cela garantit que vos commandes auront un aspect professionnel et que vos clients auront confiance en votre complément. Si vous apportez des exceptions à ces instructions, faites-le intentionnellement.

L’image suivante montre la disposition des éléments de base et modificateurs dans une icône Office.

![Diagramme montrant un élément de base d’icône au centre avec un modificateur en bas à droite et un modificateur d’action dans le coin supérieur gauche.](../images/icon-layouts.png)

- Éléments de base centraux dans le cadre de pixel avec remplissage vide tout autour.
- Placez les modificateurs d’action dans le coin supérieur gauche.
- Placez les modificateurs conceptuels dans la partie inférieure droite.
- Limitez le nombre d’éléments dans les icônes. À 32 px, limitez le nombre de modificateurs à un maximum de deux. À 16 px, limitez le nombre de modificateurs à un.

### <a name="base-element-padding"></a>Remplissage d’un élément de base

Placez les éléments de base de façon cohérente en fonction des tailles. Si les éléments de base ne peuvent pas être centrés dans le cadre, alignez-les en haut à gauche, en laissant les pixels supplémentaires dans la partie inférieure droite. Pour obtenir de meilleurs résultats, appliquez les instructions de remplissage répertoriées dans le tableau de la section suivante.

### <a name="modifiers"></a>Modificateurs

Tous les modificateurs doivent avoir un cutout transparent de 1 px entre chaque élément, y compris l’arrière-plan. Les éléments ne doivent pas se chevaucher directement. Créez des espaces entre les règles et les bords. Les modificateurs peuvent varier légèrement en taille, mais utilisez ces dimensions comme point de départ.

|Taille de l’icône|Remplissage autour de l’élément de base|Taille du modificateur|
|:---|:---|:---|
|16 px|0|9 px|
|20 px|1px|10 px|
|24 px|1px|12 px|
|32 px|2px|14 px|
|40 px|2px|20 px|
|48 px|3px|22 px|
|64 px|5px|29 px|
|80 px|5px|38 px|

## <a name="icon-colors"></a>Couleurs de l’icône

> [!NOTE]
> Les couleurs recommandées concernent les icônes du ruban utilisées dans les [Commandes de complément](add-in-commands.md). Ces icônes ne sont pas restitues avec Fluent’interface utilisateur et la palette de couleurs est différente de la palette décrite dans [Microsoft UI Fabric | Couleurs | Partagé](https://fluentfabric.azurewebsites.net/#/color/shared).

Les icônes Office ont une palette de couleurs limitée. Utilisez les couleurs répertoriées dans le tableau suivant pour garantir une intégration parfaite avec l’interface utilisateur d’Office. Appliquez les instructions suivantes à l’utilisation de la couleur.

- Utilisez la couleur pour véhiculer une signification plutôt que pour embellir. Elle doit mettre en surbrillance ou mettre en évidence une action, un état ou un élément qui différencie explicitement le repère.
- Si possible, n’utilisez qu’une seule couleur supplémentaire au-delà du gris. Limitez les couleurs supplémentaires à deux au maximum.
- Les couleurs ont une apparence cohérente dans toutes les tailles d’icône. Les icônes Office ont des palettes de couleurs légèrement différentes pour des tailles d’icônes différentes. Les icônes de 16 px et plus petites sont légèrement plus sombres et plus dynamiques que les icônes de 32 px et plus grandes. Sans ces ajustements discrets, les couleurs semblent varier en taille.

|Nom de la couleur|RVB|Hex|Couleur|Catégorie|
|:---|:---|:---|:---|:---|
|Texte gris (80)|80, 80, 80|#505050| ![Gris 80 pour le texte.](../images/color-text-gray-80.png) |Texte|
|Texte gris (95)|95, 95, 95|#5F5F5F| ![Gris 95 pour le texte.](../images/color-text-gray-95.png) |Texte|
|Texte gris (105)|105, 105, 105|#696969| ![Gris 105 pour le texte.](../images/color-text-gray-105.png) |Texte|
|Gris foncé 32|128, 128, 128|#808080| ![Gris foncé pour 32 px et plus.](../images/color-dark-gray-32.png) |32 px et supérieures|
|Gris moyen 32|158, 158, 158|#9E9E9E| ![Gris moyen pour 32 px et plus.](../images/color-medium-gray-32.png) |32 px et supérieures|
|TOUT gris clair|179, 179, 179|#B3B3B3| ![Gris clair pour toutes les tailles d’image.](../images/color-light-gray-all.png) |Toutes les tailles|
|Gris foncé 16|114, 114, 114|#727272| ![Gris foncé pour 16 px et moins.](../images/color-dark-gray-16.png) |16 px et inférieur|
|Gris moyen 16|144, 144, 144|#909090| ![Gris moyen pour 16 px et plus petit.](../images/color-medium-gray-16.png) |16 et moins|
|Bleu 32|77, 130, 184|#4d82B8| ![Bleu pour 32 px et plus.](../images/color-blue-32.png) |32 px et supérieures|
|Bleu 16|74, 125, 177|#4A7DB1| ![Bleu pour 16 px et moins.](../images/color-blue-16.png) |16 px et inférieur|
|TOUT jaune|234, 194, 130|#EAC282| ![Jaune pour toutes les tailles d’image.](../images/color-yellow-all.png) |Toutes les tailles|
|Orange 32|231, 142, 70|#E78E46| ![Couleur orange pour 32 px et plus.](../images/color-orange-32.png) |32 px et supérieures|
|Orange 16|227, 142, 70|#E3751C| ![Couleur orange pour 16 px et moins.](../images/color-orange-16.png) |16 px et inférieur|
|TOUT rose|230, 132, 151|#E68497| ![Rose pour toutes les tailles d’image.](../images/color-pink-all.png) |Toutes les tailles|
|Vert 32|118, 167, 151|#76A797| ![Vert pour 32 px et plus.](../images/color-green-32.png) |32 px et supérieures|
|Vert 16|104, 164, 144|#68A490| ![Vert pour 16 px et moins.](../images/color-green-16.png) |16 px et inférieur|
|Rouge 32|216, 99, 68|#D86344| ![Couleur rouge pour 32 px et plus.](../images/color-red-32.png) |32 px et supérieures|
|Rouge 16|214, 85, 50|#D65532| ![Couleur rouge pour 16 px et moins.](../images/color-red-16.png) |16 px et au-dessous|
|Violet 32|152, 104, 185|#9868B9| ![Violet pour 32 px et plus.](../images/color-purple-32.png) |32 px et supérieures|
|Violet 16|137, 89, 171|#8959AB| ![Violet pour 16 px et moins.](../images/color-purple-16.png) |16 px et au-dessous|

## <a name="icons-in-high-contrast-modes"></a>Icônes en modes de contraste élevé

Les icônes Office sont conçues pour un rendu correct en mode de contraste élevé. Les éléments de premier plan sont bien différenciés des arrière-plans pour optimiser la lisibilité et permettre le recoloriage. En modes de contraste élevé, Office recolorie tous les pixels de votre icône avec une valeur rouge, verte ou bleue inférieure à 190 en noir plein. Tous les autres pixels sont blancs. Autrement dit, chaque canal RVB est évalué lorsque les valeurs 0-189 sont noires et les valeurs 190-255 sont blanches. D’autres thèmes à contraste élevé recolorient à l’aide du même seuil de valeur 190 mais avec des règles différentes. Par exemple, le thème blanc à contraste élevé recolorie tous les pixels supérieurs à 190 en opaque, mais tous les autres pixels en transparent. Appliquez les instructions suivantes pour optimiser la lisibilité dans les paramètres à contraste élevé.

- Essayez de différencier les éléments de premier plan et d’arrière-plan par rapport au seuil de valeur 190.
- Suivez les styles visuels des icônes Office.
- Utilisez des couleurs de notre palette d’icônes.
- Évitez d’utiliser des dégradés.
- Évitez les grands blocs de couleur avec des valeurs similaires.

## <a name="see-also"></a>Voir aussi

- [Élément de manifeste d’icône](../reference/manifest/icon.md)
- [Élément manifeste IconUrl](../reference/manifest/iconurl.md)
- [Élément manifeste HighResolutionIconUrl](../reference/manifest/highresolutioniconurl.md)
- [Créer une icône pour votre add-in](/office/dev/store/create-effective-office-store-listings#create-an-icon-for-your-add-in)
