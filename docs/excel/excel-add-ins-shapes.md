---
title: Utiliser des formes à l’aide Excel API JavaScript
description: Découvrez comment Excel définit les formes comme n’importe quel objet qui se trouve sur la couche de dessin de Excel.
ms.date: 01/14/2020
localization_priority: Normal
ms.openlocfilehash: eeb6a1f76c839e4b550662b28b717bfd1bcca4e8
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53349447"
---
# <a name="work-with-shapes-using-the-excel-javascript-api"></a>Utiliser des formes à l’aide Excel API JavaScript

Excel définit les formes comme n’importe quel objet qui se trouve sur la couche de dessin de Excel. Cela signifie que tout ce qui se trouve en dehors d’une cellule est une forme. Cet article explique comment utiliser des formes géométriques, des lignes et des images conjointement avec les API [Shape](/javascript/api/excel/excel.shape) et [ShapeCollection.](/javascript/api/excel/excel.shapecollection) [Les](/javascript/api/excel/excel.chart) graphiques sont abordés dans leur propre article, [Work with charts using the Excel JavaScript API](excel-add-ins-charts.md).

L’image suivante montre les formes qui forment un thermomètre.
![Image d’un thermomètre en forme de Excel forme.](../images/excel-shapes.png)

## <a name="create-shapes"></a>Créer des formes

Les formes sont créées par le biais et stockées dans la collection de formes d’une feuille de calcul ( `Worksheet.shapes` ). `ShapeCollection` a plusieurs `.add*` méthodes à cet effet. Toutes les formes ont des noms et des ID générés pour elles lorsqu’elles sont ajoutées à la collection. Ce sont `name` respectivement les `id` propriétés et les propriétés. `name` peut être définie par votre add-in pour faciliter l’extraction avec la `ShapeCollection.getItem(name)` méthode.

Les types de formes suivants sont ajoutés à l’aide de la méthode associée.

| Forme | Add, méthode | Signature |
|-------|------------|-----------|
| Forme géométrique | [addGeometricShape](/javascript/api/excel/excel.shapecollection#addgeometricshape-geometricshapetype-) | `addGeometricShape(geometricShapeType: Excel.GeometricShapeType): Excel.Shape` |
| Image (JPEG ou PNG) | [addImage](/javascript/api/excel/excel.shapecollection#addimage-base64imagestring-) | `addImage(base64ImageString: string): Excel.Shape` |
| Trait | [addLine](/javascript/api/excel/excel.shapecollection#addline-startleft--starttop--endleft--endtop--connectortype-) | `addLine(startLeft: number, startTop: number, endLeft: number, endTop: number, connectorType?: Excel.ConnectorType): Excel.Shape` |
| SVG | [addSvg](/javascript/api/excel/excel.shapecollection#addsvg-xml-) | `addSvg(xml: string): Excel.Shape` |
| Zone de texte | [addTextBox](/javascript/api/excel/excel.shapecollection#addtextbox-text-) | `addTextBox(text?: string): Excel.Shape` |

### <a name="geometric-shapes"></a>Formes géométriques

Une forme géométrique est créée avec `ShapeCollection.addGeometricShape` . Cette méthode prend une [enum GeometricShapeType](/javascript/api/excel/excel.geometricshapetype) comme argument.

L’exemple de code suivant crée un rectangle de 150 x 150 pixels nommé « **Square** » placé à 100 pixels des côtés supérieur et gauche de la feuille de calcul.

```js
// This sample creates a rectangle positioned 100 pixels from the top and left sides
// of the worksheet and is 150x150 pixels.
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var rectangle = shapes.addGeometricShape(Excel.GeometricShapeType.rectangle);
    rectangle.left = 100;
    rectangle.top = 100;
    rectangle.height = 150;
    rectangle.width = 150;
    rectangle.name = "Square";
    return context.sync();
}).catch(errorHandlerFunction);
```

### <a name="images"></a>Des images

Les images JPEG, PNG et SVG peuvent être insérées dans une feuille de calcul sous forme de formes. La méthode prend comme argument une chaîne `ShapeCollection.addImage` codée en base 64. Il s’agit d’une image JPEG ou PNG sous forme de chaîne. `ShapeCollection.addSvg` prend également une chaîne, bien que cet argument soit du XML qui définit le graphique.

L’exemple de code suivant montre un fichier image chargé par [un FileReader](https://developer.mozilla.org/docs/Web/API/FileReader) sous la mesure d’une chaîne. La chaîne a les métadonnées « base64 » supprimées avant la création de la forme.

```js
// This sample creates an image as a Shape object in the worksheet.
var myFile = document.getElementById("selectedFile");
var reader = new FileReader();

reader.onload = (event) => {
    Excel.run(function (context) {
        var startIndex = reader.result.toString().indexOf("base64,");
        var myBase64 = reader.result.toString().substr(startIndex + 7);
        var sheet = context.workbook.worksheets.getItem("MyWorksheet");
        var image = sheet.shapes.addImage(myBase64);
        image.name = "Image";
        return context.sync();
    }).catch(errorHandlerFunction);
};

// Read in the image file as a data URL.
reader.readAsDataURL(myFile.files[0]);
```

### <a name="lines"></a>Lines

Une ligne est créée avec `ShapeCollection.addLine` . Cette méthode a besoin des marges gauche et supérieure des points de début et de fin de la ligne. Il prend également une enum [ConnectorType](/javascript/api/excel/excel.connectortype) pour spécifier la façon dont la ligne est contorte entre les points de terminaison. L’exemple de code suivant crée une ligne droite sur la feuille de calcul.

```js
// This sample creates a straight line from [200,50] to [300,150] on the worksheet
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var line = shapes.addLine(200, 50, 300, 150, Excel.ConnectorType.straight);
    line.name = "StraightLine";
    return context.sync();
}).catch(errorHandlerFunction);
```

Les lignes peuvent être connectées à d’autres objets Shape. Les méthodes attachent le début et la fin d’une ligne aux formes aux `connectBeginShape` points de connexion `connectEndShape` spécifiés. Les emplacements de ces points varient en fonction de la forme, mais ils peuvent être utilisés pour vous assurer que votre module ne se connecte pas à un point hors `Shape.connectionSiteCount` limites. Une ligne est déconnectée de toutes les formes attachées à l’aide `disconnectBeginShape` des méthodes et des `disconnectEndShape` méthodes.

L’exemple de code suivant connecte la ligne « **MyLine** » à deux formes nommées **« LeftShape** » et **« RightShape**».

```js
// This sample connects a line between two shapes at connection points '0' and '3'.
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var line = shapes.getItem("MyLine").line;
    line.connectBeginShape(shapes.getItem("LeftShape"), 0);
    line.connectEndShape(shapes.getItem("RightShape"), 3);
    return context.sync();
}).catch(errorHandlerFunction);
```

## <a name="move-and-resize-shapes"></a>Déplacer et re tailler des formes

Les formes sont au-dessus de la feuille de calcul. Leur placement est défini par la `left` propriété `top` et la propriété. Elles agissent comme des marges des bords respectifs de la feuille de calcul, [0, 0] étant le coin supérieur gauche. Celles-ci peuvent être définies directement ou ajustées à partir de leur position actuelle avec les `incrementLeft` `incrementTop` méthodes et les méthodes. La quantité de rotation d’une forme par rapport à la position par défaut est également établie de cette manière, la propriété étant la quantité absolue et la méthode ajustant la `rotation` `incrementRotation` rotation existante.

La profondeur d’une forme par rapport aux autres formes est définie par la `zorderPosition` propriété. Ceci est définie à `setZOrder` l’aide de la méthode, qui prend [un ShapeZOrder](/javascript/api/excel/excel.shapezorder). `setZOrder` ajuste l’ordre de la forme actuelle par rapport aux autres formes.

Votre add-in dispose de deux options pour modifier la hauteur et la largeur des formes. La définition de `height` la ou de la propriété modifie la dimension `width` spécifiée sans modifier l’autre dimension. L’et ajuster les dimensions respectives de la forme par rapport à la taille actuelle ou d’origine (en fonction de la valeur de `scaleHeight` `scaleWidth` [l’shapeScaleType fourni](/javascript/api/excel/excel.shapescaletype)). Un paramètre [ShapeScaleFrom](/javascript/api/excel/excel.shapescalefrom) facultatif spécifie l’endroit où la forme est mise à l’échelle (coin supérieur gauche, milieu ou coin inférieur droit). Si la propriété est true, les méthodes d’échelle conservent les proportions actuelles de la forme en ajustant également `lockAspectRatio` l’autre dimension.

> [!NOTE]
> Les modifications directes apportées aux propriétés affectent uniquement cette propriété, quelle que `height` soit la valeur de la `width` `lockAspectRatio` propriété.

L’exemple de code suivant montre une forme mise à l’échelle jusqu’à 1,25 fois sa taille d’origine et pivotée de 30 degrés.

```js
// In this sample, the shape "Octagon" is rotated 30 degrees clockwise
// and scaled 25% larger, with the upper-left corner remaining in place.
Excel.run(function (context) {
    var sheet = context.workbook.worksheets.getItem("MyWorksheet");
    var shape = sheet.shapes.getItem("Octagon");
    shape.incrementRotation(30);
    shape.lockAspectRatio = true;
    shape.scaleWidth(
        1.25,
        Excel.ShapeScaleType.currentSize,
        Excel.ShapeScaleFrom.scaleFromTopLeft);
    return context.sync();
}).catch(errorHandlerFunction);
```

## <a name="text-in-shapes"></a>Texte dans les formes

Les formes géométriques peuvent contenir du texte. Les formes ont `textFrame` une propriété de type [TextFrame](/javascript/api/excel/excel.textframe). `TextFrame`L’objet gère les options d’affichage de texte (telles que les marges et le dépassement de texte). `TextFrame.textRange` est un [objet TextRange](/javascript/api/excel/excel.textrange) avec le contenu du texte et les paramètres de police.

L’exemple de code suivant crée une forme géométrique nommée « Wave » avec le texte « Shape Text ». Il ajuste également les couleurs de la forme et du texte, et définit l’alignement horizontal du texte sur le centre.

```js
// This sample creates a light-blue wave shape and adds the purple text "Shape text" to the center.
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var wave = shapes.addGeometricShape(Excel.GeometricShapeType.wave);
    wave.left = 100;
    wave.top = 400;
    wave.height = 50;
    wave.width = 150;
    wave.name = "Wave";
    wave.fill.setSolidColor("lightblue");
    wave.textFrame.textRange.text = "Shape text";
    wave.textFrame.textRange.font.color = "purple";
    wave.textFrame.horizontalAlignment = Excel.ShapeTextHorizontalAlignment.center;
    return context.sync();
}).catch(errorHandlerFunction);
```

La `addTextBox` méthode de création `ShapeCollection` `GeometricShape` d’un type avec un `Rectangle` arrière-plan blanc et du texte noir. Ceci est identique à ce qui est créé par Excel bouton **Zone** de texte sous l’onglet **Insertion.** `addTextBox` prend un argument de chaîne pour définir le texte du `TextRange` .

L’exemple de code suivant montre la création d’une zone de texte avec le texte « Hello! ».

```js
// This sample creates a text box with the text "Hello!" and sizes it appropriately.
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var textbox = shapes.addTextBox("Hello!");
    textbox.left = 100;
    textbox.top = 100;
    textbox.height = 20;
    textbox.width = 45;
    textbox.name = "Textbox";
    return context.sync();
}).catch(errorHandlerFunction);
```

## <a name="shape-groups"></a>Groupes de formes

Les formes peuvent être regroupées. Cela permet à un utilisateur de les traiter comme une entité unique pour le positionnement, le resserrement et d’autres tâches connexes. Un [shapeGroup](/javascript/api/excel/excel.shapegroup) est un type de `Shape` , donc votre add-in traite le groupe comme une forme unique.

L’exemple de code suivant montre trois formes regroupées. L’exemple de code suivant montre que le groupe de formes est déplacé vers la droite de 50 pixels.

```js
// This sample takes three previously-created shapes ("Square", "Pentagon", and "Octagon")
// and groups them into a single ShapeGroup.
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var square = shapes.getItem("Square");
    var pentagon = shapes.getItem("Pentagon");
    var octagon = shapes.getItem("Octagon");

    var shapeGroup = shapes.addGroup([square, pentagon, octagon]);
    shapeGroup.name = "Group";
    console.log("Shapes grouped");

    return context.sync();
}).catch(errorHandlerFunction);

// This sample moves the previously created shape group to the right by 50 pixels.
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var shapeGroup = sheet.shapes.getItem("Group");
    shapeGroup.incrementLeft(50);
    return context.sync();
}).catch(errorHandlerFunction);
```

> [!IMPORTANT]
> Les formes individuelles au sein du groupe sont référencés par le biais de la `ShapeGroup.shapes` propriété, qui est de type [GroupShapeCollection](/javascript/api/excel/excel.GroupShapeCollection). Elles ne sont plus accessibles via la collection de formes de la feuille de calcul après avoir été regroupées. Par exemple, si votre feuille de calcul avait trois formes et qu’elles étaient toutes regroupées, la méthode de la feuille de calcul retournerait le nombre `shapes.getCount` 1.

## <a name="export-shapes-as-images"></a>Exporter des formes en tant qu’images

Tout `Shape` objet peut être converti en image. [Shape.getAsImage](/javascript/api/excel/excel.shape#getasimage-format-) renvoie une chaîne codée en base 64. Le format de l’image est spécifié en tant qu’enum [PictureFormat](/javascript/api/excel/excel.pictureformat) transmis à `getAsImage` .

```js
Excel.run(function (context) {
    var shapes = context.workbook.worksheets.getItem("MyWorksheet").shapes;
    var shape = sheet.shapes.getItem("Image");
    var stringResult = shape.getAsImage(Excel.PictureFormat.png);

    return context.sync().then(function () {
        console.log(stringResult.value);
        // Instead of logging, your add-in may use the base64-encoded string to save the image as a file or insert it in HTML.
    });
}).catch(errorHandlerFunction);
```

## <a name="delete-shapes"></a>Supprimer des formes

Les formes sont supprimées de la feuille de calcul à `Shape` l’aide de la méthode de `delete` l’objet. Aucune autre métadonnée n’est nécessaire.

L’exemple de code suivant supprime toutes les formes de **MyWorksheet**.

```js
// This deletes all the shapes from "MyWorksheet".
Excel.run(function (context) {
    var sheet = context.workbook.worksheets.getItem("MyWorksheet");
    var shapes = sheet.shapes;

    // We'll load all the shapes in the collection without loading their properties.
    shapes.load("items/$none");
    return context.sync().then(function () {
        shapes.items.forEach(function (shape) {
            shape.delete()
        });
        return context.sync();
    }).catch(errorHandlerFunction);
}).catch(errorHandlerFunction);
```

## <a name="see-also"></a>Voir aussi

- [Concepts fondamentaux de programmation avec l’API JavaScript pour Excel](../reference/overview/excel-add-ins-reference-overview.md)
- [Utiliser des graphiques à l’aide de l’API JavaScript pour Excel](excel-add-ins-charts.md)
