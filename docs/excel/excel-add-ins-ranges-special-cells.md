---
title: Rechercher des cellules spéciales dans une plage à l’aide de l’API JavaScript pour Excel
description: Découvrez comment utiliser l’API JavaScript excel pour rechercher des cellules spéciales, telles que des cellules avec des formules, des erreurs ou des nombres.
ms.date: 04/02/2021
ms.prod: excel
localization_priority: Normal
ms.openlocfilehash: 6504873bcd8ab50bd4c03fe4f54b71d0bd920c5b
ms.sourcegitcommit: 54fef33bfc7d18a35b3159310bbd8b1c8312f845
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/09/2021
ms.locfileid: "51652824"
---
# <a name="find-special-cells-within-a-range-using-the-excel-javascript-api"></a>Rechercher des cellules spéciales dans une plage à l’aide de l’API JavaScript pour Excel

Cet article fournit des exemples de code qui recherchent des cellules spéciales dans une plage à l’aide de l’API JavaScript pour Excel. Pour obtenir la liste complète des propriétés et des méthodes que l’objet prend en charge, voir `Range` la classe [Excel.Range.](/javascript/api/excel/excel.range)

## <a name="find-ranges-with-special-cells"></a>Rechercher des plages avec des cellules spéciales

Les méthodes [Range.getSpecialCells](/javascript/api/excel/excel.range#getspecialcells-celltype--cellvaluetype-) et [Range.getSpecialCellsOrNullObject](/javascript/api/excel/excel.range#getspecialcellsornullobject-celltype--cellvaluetype-) recherchent des plages en fonction des caractéristiques de leurs cellules et des types de valeurs de leurs cellules. Ces deux méthodes renvoient à des`RangeAreas`objets. Voici les signatures des méthodes à partir des types de fichiers de données TypeScript:

```typescript
getSpecialCells(cellType: Excel.SpecialCellType, cellValueType?: Excel.SpecialCellValueType): Excel.RangeAreas;
```

```typescript
getSpecialCellsOrNullObject(cellType: Excel.SpecialCellType, cellValueType?: Excel.SpecialCellValueType): Excel.RangeAreas;
```

L’exemple de code suivant utilise `getSpecialCells` la méthode pour rechercher toutes les cellules avec des formules. Tenez compte du code suivant:

- Cela limite la partie de la feuille qui nécessite d’être recherchée en appelant d’abord`Worksheet.getUsedRange`et en appelant`getSpecialCells`uniquement pour cette plage.
- La`getSpecialCells`méthode renvoie un`RangeAreas`objet, toutes les cellules alors dotées de formules seront colorées en rose même si elles ne sont pas adjacentes.

```js
Excel.run(function (context) {
    var sheet = context.workbook.worksheets.getActiveWorksheet();
    var usedRange = sheet.getUsedRange();
    var formulaRanges = usedRange.getSpecialCells(Excel.SpecialCellType.formulas);
    formulaRanges.format.fill.color = "pink";

    return context.sync();
})
```

Si aucune cellule avec la caractéristique ciblée n’existe dans la plage `getSpecialCells` lève une erreur **ItemNotFound**. Cela dévie le flux de contrôle vers un(e)`catch`bloc/méthode, s’il en existe. S’il n’y a `catch` pas de bloc, l’erreur arrête la méthode.

Si vous attendez que des cellules avec la caractéristique ciblée existent toujours, vous souhaiterez probablement que votre code  lève une erreur si ces cellules ne sont pas là. Mais dans les scénarios où les cellules ne correspondent pas; votre code doit vérifier cette possibilité et le gérer gracieusement sans émettre d’erreur. Vous pouvez obtenir ce comportement avec la `getSpecialCellsOrNullObject`méthode et sa propriété renvoyée`isNullObject`. L’exemple de code suivant utilise ce modèle. Tenez compte du code suivant :

- La méthode renvoie toujours un objet proxy, donc elle `getSpecialCellsOrNullObject` n’est jamais dans le sens `null` JavaScript ordinaire. Mais si les cellules non correspondantes sont introuvables, la propriété`isNullObject` de l’objet est établi à`true`.
- Il appelle`context.sync`*avant* de tester la propriété`isNullObject`. Il s’agit d’une condition avec toutes les méthodes et propriétés`*OrNullObject`, car vous devez toujours télécharger et synchroniser une propriété afin de le lire.  Toutefois, il n’est pas nécessaire de *charger explicitement* la `isNullObject` propriété. Il est automatiquement chargé par le même `context.sync` s’il `load` n’est pas appelé sur l’objet. Pour plus d’informations, [ \* voir méthodes et propriétés OrNullObject.](../develop/application-specific-api-model.md#ornullobject-methods-and-properties)
- Vous pouvez tester ce code en sélectionnant d’abord une plage qui n’a pas de cellules de formule et en l’exécutant. Puis sélectionnez une plage qui dispose au moins d’une cellule dotée d’une formule et en l’exécutant à nouveau.

```js
Excel.run(function (context) {
    var range = context.workbook.getSelectedRange();
    var formulaRanges = range.getSpecialCellsOrNullObject(Excel.SpecialCellType.formulas);
    return context.sync()
        .then(function() {
            if (formulaRanges.isNullObject) {
                console.log("No cells have formulas");
            }
            else {
                formulaRanges.format.fill.color = "pink";
            }
        })
        .then(context.sync);
})
```

Par souci de simplicité, tous les autres exemples de code de cet article utilisent la `getSpecialCells` méthode au lieu de  `getSpecialCellsOrNullObject` .

## <a name="narrow-the-target-cells-with-cell-value-types"></a>Réduisez les cellules cibles avec les types de valeur de cellule

Les méthodes`Range.getSpecialCells()` et `Range.getSpecialCellsOrNullObject()`acceptent un deuxième paramètre facultatif utilisé pour affiner davantage les cellules ciblées. Ce deuxième paramètre est un`Excel.SpecialCellValueType` que vous utilisez afin de spécifier que vous souhaitez uniquement les cellules qui contiennent certains types de valeurs.

> [!NOTE]
> Le paramètre `Excel.SpecialCellValueType` peut uniquement être utilisé si le paramètre `Excel.SpecialCellType` est défini sur `Excel.SpecialCellType.formulas`ou `Excel.SpecialCellType.constants`.

### <a name="test-for-a-single-cell-value-type"></a>Test d’un type de valeur de cellule unique

Le `Excel.SpecialCellValueType` enum dispose de ces quatre types de base (outre les autres valeurs combinées décrites plus loin dans cette section):

- `Excel.SpecialCellValueType.errors`
- `Excel.SpecialCellValueType.logical` (ce qui signifie booléen)
- `Excel.SpecialCellValueType.numbers`
- `Excel.SpecialCellValueType.text`

L’exemple de code suivant recherche des cellules spéciales qui sont des constantes numériques et colore ces cellules en rose. Tenez compte du code suivant :

- Il met uniquement en évidence les cellules qui ont une valeur de nombre littérale. Il ne surligne pas les cellules qui ont une formule (même si le résultat est un nombre) ou des cellules booléles, de texte ou d’état d’erreur.
- Pour tester le code, assurez-vous que la feuille de calcul dispose de certaines cellules avec des valeurs de nombre littérales, certaines avec d’autres sortes de valeurs littérales, et certaines avec des formules.

```js
Excel.run(function (context) {
    var sheet = context.workbook.worksheets.getActiveWorksheet();
    var usedRange = sheet.getUsedRange();
    var constantNumberRanges = usedRange.getSpecialCells(
        Excel.SpecialCellType.constants,
        Excel.SpecialCellValueType.numbers);
    constantNumberRanges.format.fill.color = "pink";

    return context.sync();
})
```

### <a name="test-for-multiple-cell-value-types"></a>Test d’un type de valeur de cellule multiple

Parfois, vous avez besoin d’exécuter plus d’un type de valeur de cellule, tel que toutes les cellules à valeur de texte et à valeur booléen (`Excel.SpecialCellValueType.logical`). Le `Excel.SpecialCellValueType` enum comporte des valeurs avec les types combinés. Par exemple,`Excel.SpecialCellValueType.logicalText`cible toutes les cellules à valeur texte et booléen. `Excel.SpecialCellValueType.all` est la valeur par défaut, ce qui ne limite pas les types de valeur de cellule renvoyés. L’exemple de code suivant colore toutes les cellules avec des formules qui produisent une valeur de nombre ou booléen.

```js
Excel.run(function (context) {
    var sheet = context.workbook.worksheets.getActiveWorksheet();
    var usedRange = sheet.getUsedRange();
    var formulaLogicalNumberRanges = usedRange.getSpecialCells(
        Excel.SpecialCellType.formulas,
        Excel.SpecialCellValueType.logicalNumbers);
    formulaLogicalNumberRanges.format.fill.color = "pink";

    return context.sync();
})
```

## <a name="see-also"></a>Voir aussi

- [Modèle d’objet JavaScript Excel dans les compléments Office](excel-add-ins-core-concepts.md)
- [Utiliser des cellules à l’aide de l’API JavaScript pour Excel](excel-add-ins-cells.md)
- [Rechercher une chaîne à l’aide de l’API JavaScript pour Excel](excel-add-ins-ranges-string-match.md)
- [Travailler simultanément avec plusieurs plages dans des compléments Excel](excel-add-ins-multiple-ranges.md)
