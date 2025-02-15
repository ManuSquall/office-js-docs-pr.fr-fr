---
title: Extraire des chaînes d’entités d’un élément Outlook
description: Découvrez comment extraire des chaînes d’entités d’un élément Outlook dans un complément Outlook.
ms.date: 10/31/2019
localization_priority: Normal
ms.openlocfilehash: 987ba7626acb95bd5090e2f2350f71ecc8701e59
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53348971"
---
# <a name="extract-entity-strings-from-an-outlook-item"></a>Extraire des chaînes d’entité d’un élément Outlook

Cet article décrit comment créer un complément Outlook pour l’**affichage des entités** qui extrait des instances de chaînes d’entités connues prises en charge dans l’objet et le corps de l’élément Outlook sélectionné. Cet élément peut être un rendez-vous, un message électronique ou encore une demande, une réponse ou une annulation de réunion.

Les entités prises en charge incluent notamment :

- **Adresse** : une adresse postale aux États-Unis, qui a au moins un sous-ensemble des éléments de numéro de rue, nom de rue, ville, état et code postal.
    
- **Contact** : informations de contact d’une personne, dans le contexte d’autres entités, telles qu’une adresse ou un nom commercial.
    
- **Adresse électronique** : une adresse électronique SMTP.
    
- **Suggestion de réunion** : une suggestion de réunion, par exemple une référence à un événement. Notez que seuls les messages (pas les rendez-vous) prennent en charge l’extraction de suggestion de réunion.
    
- **Numéro de téléphone** : numéro de téléphone nord-américain.
    
- **Suggestion de tâche** : une suggestion de tâche, généralement exprimée dans une phrase associée à une action.
    
- **URL**
    
La plupart de ces entités s’appuient sur la reconnaissance vocale en langage naturel qui est basée sur l’apprentissage machine de grandes quantités de données. Par conséquent, la reconnaissance est non déterministe et dépend parfois du contexte de l’élément Outlook.

Outlook active le complément des entités lorsque l’utilisateur sélectionne un rendez-vous, un message électronique ou une demande, réponse ou annulation de réunion à afficher. Lors de l’initialisation, l’exemple de complément d’entités lit toutes les instances des entités prises en charge à partir de l’élément actif. 

Le complément comporte des boutons permettant à l’utilisateur de choisir un type d’entité. Quand l’utilisateur sélectionne une entité, le complément affiche les instances de l’entité sélectionnée dans le volet de complément. Les sections suivantes répertorient les manifestes XML, les fichiers HTML et JavaScript du complément des entités, et met en évidence le code qui prend en charge l’extraction de l’entité associée.

## <a name="xml-manifest"></a>Manifeste XML

Le complément pour entités a deux règles d’activation jointes par une opération OR logique. 

```xml
<!-- Activate the add-in if the current item in Outlook is an email or appointment item. -->
<Rule xsi:type="RuleCollection" Mode="Or">
    <Rule xsi:type="ItemIs" ItemType="Message"/>
    <Rule xsi:type="ItemIs" ItemType="Appointment"/>
</Rule>
```

Ces règles spécifient qu’Outlook doit activer ce complément lorsque l’élément sélectionné dans l’inspecteur ou le volet de lecture est un rendez-vous ou un message (notamment un message électronique, ou une demande, une réponse ou une annulation de réunion).

Voici le manifeste du complément pour entités. Il utilise la version 1.1 du schéma pour les manifestes des Compléments Office.

```xml
<?xml version="1.0" encoding="utf-8"?>
<OfficeApp xmlns="http://schemas.microsoft.com/office/appforoffice/1.1" 
xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" 
xsi:type="MailApp">
  <Id>6880A140-1C4F-11E1-BDDB-0800200C9A68</Id>
  <Version>1.0</Version>
  <ProviderName>Microsoft</ProviderName>
  <DefaultLocale>EN-US</DefaultLocale>
  <DisplayName DefaultValue="Display entities"/>
  <Description DefaultValue=
     "Display known entities on the selected item."/>
  <Hosts>
    <Host Name="Mailbox" />
  </Hosts>
  <Requirements>
    <Sets DefaultMinVersion="1.1">
      <Set Name="Mailbox" />
    </Sets>
  </Requirements>
  <FormSettings>
    <Form xsi:type="ItemRead">
      <DesktopSettings>
        <!-- Change the following line to specify the web -->
        <!-- server where the HTML file is hosted. -->
        <SourceLocation DefaultValue=
          "http://webserver/default_entities/default_entities.html"/>
        <RequestedHeight>350</RequestedHeight>
      </DesktopSettings>
    </Form>
  </FormSettings>
  <Permissions>ReadItem</Permissions>
  <!-- Activate the add-in if the current item in Outlook is -->
  <!-- an email or appointment item. -->
  <Rule xsi:type="RuleCollection" Mode="Or">
    <Rule xsi:type="ItemIs" ItemType="Message"/>
    <Rule xsi:type="ItemIs" ItemType="Appointment"/>
  </Rule>
  <DisableEntityHighlighting>false</DisableEntityHighlighting>
</OfficeApp>
```


## <a name="html-implementation"></a>Implémentation HTML

Le fichier HTML du complément pour entités spécifie les boutons permettant à l’utilisateur de sélectionner chaque type d’entité, et un autre bouton pour effacer les instances affichées d’une entité. Il inclut un fichier JavaScript, default_entities.js, qui est décrit dans la section suivante sous [Implémentation JavaScript](#javascript-implementation). Le fichier JavaScript inclut le gestionnaire d’événements pour chacun des boutons.

Notez que tous les compléments Outlook doivent comprendre le fichier office.js. Le fichier HTML suivant inclut la version 1.1 du fichier office.js sur le réseau de distribution de contenu (CDN). 

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="X-UA-Compatible" content="IE=Edge" >
    <title>standard_item_properties</title>
    <link rel="stylesheet" type="text/css" media="all" href="default_entities.css" />
    <script type="text/javascript" src="MicrosoftAjax.js"></script>
    <!-- Use the CDN reference to Office.js. -->
    <script src="https://appsforoffice.microsoft.com/lib/1/hosted/Office.js" type="text/javascript"></script>
    <script type="text/javascript"  src="default_entities.js"></script>
</head>

<body>
    <div id="container">
        <div id="button">
        <input type="button" value="clear" 
            onclick="myClearEntitiesBox();">
        <input type="button" value="Get Addresses" 
            onclick="myGetAddresses();">
        <input type="button" value="Get Contact Information" 
            onclick="myGetContacts();">
        <input type="button" value="Get Email Addresses" 
            onclick="myGetEmailAddresses();">
        <input type="button" value="Get Meeting Suggestions" 
            onclick="myGetMeetingSuggestions();">
        <input type="button" value="Get Phone Numbers" 
            onclick="myGetPhoneNumbers();">
        <input type="button" value="Get Task Suggestions" 
            onclick="myGetTaskSuggestions();">
        <input type="button" value="Get URLs" 
            onclick="myGetUrls();">
        </div>
        <div id="entities_box"></div>
    </div>
</body>
</html>
```


## <a name="style-sheet"></a>Feuille de style


Le complément pour entités utilise un fichier CSS facultatif, default_entities.css, pour spécifier la mise en forme de la sortie. Le fichier CSS est présenté ci-dessous.


```CSS
*
{
    color: #FFFFFF;
    margin: 0px;
    padding: 0px;
    font-family: Arial, Sans-serif;
}
html 
{
    scrollbar-base-color: #FFFFFF;
    scrollbar-arrow-color: #ABABAB; 
    scrollbar-lightshadow-color: #ABABAB; 
    scrollbar-highlight-color: #ABABAB; 
    scrollbar-darkshadow-color: #FFFFFF; 
    scrollbar-track-color: #FFFFFF;
}
body
{
    background: #4E9258;
}
input
{
    color: #000000;
    padding: 5px;
}
span
{
    color: #FFFF00;
}
div#container
{
    height: 100%;
    padding: 2px;
    overflow: auto;
}
div#container td
{
    border-bottom: 1px solid #CCCCCC;
}
td.property-name
{
    padding: 0px 5px 0px 0px;
    border-right: 1px solid #CCCCCC;
}
div#meeting_suggestions
{
    border-top: 1px solid #CCCCCC;
}
```


## <a name="javascript-implementation"></a>Implémentation JavaScript

Les sections suivantes expliquent comment l’exemple suivant (le fichier default_entities.js) extrait des entités connues de l’objet et du corps du message ou du rendez-vous consulté par l’utilisateur.

## <a name="extracting-entities-upon-initialization"></a>Extraction d’entités lors de l’initialisation

Lors de l’événement [Office.initialize](/javascript/api/office#office-initialize-reason-), le complément pour entités appelle la méthode [getEntities](../reference/objectmodel/preview-requirement-set/office.context.mailbox.item.md#methods) de l’élément actuel. La `getEntities` méthode renvoie à la variable globale un tableau `_MyEntities` d’instances d’entités pris en charge. Le code JavaScript associé est présenté ci-dessous.


```js
// Global variables
var _Item;
var _MyEntities;

// The initialize function is required for all add-ins.
Office.initialize = function () {
    var _mailbox = Office.context.mailbox;
    // Obtains the current item.
    Item = _mailbox.item;
    // Reads all instances of supported entities from the subject 
    // and body of the current item.
    MyEntities = _Item.getEntities();
    
    // Checks for the DOM to load using the jQuery ready function.
    $(document).ready(function () {
    // After the DOM is loaded, app-specific code can run.
    });
}

```


## <a name="extracting-addresses"></a>Extraction d’adresses


Lorsque l’utilisateur clique sur le bouton **Obtenir les adresses**, le gestionnaire d’événements `myGetAddresses` obtient un tableau d’adresses à partir de la propriété [adresses](/javascript/api/outlook/office.entities#addresses) de l’objet `_MyEntities`, si une adresse a été extraite. Toute adresse extraite est stockée comme chaîne dans le tableau. `myGetAddresses` forme une chaîne HTML locale dans `htmlText` pour afficher la liste des adresses extraites. Le code JavaScript associé est présenté ci-dessous.


```js
// Gets instances of the Address entity on the item.
function myGetAddresses()
{
    var htmlText = "";

    // Gets an array of postal addresses. Each address is a string.
    var addressesArray = _MyEntities.addresses;
    for (var i = 0; i < addressesArray.length; i++)
    {
        htmlText += "Address : <span>" + addressesArray[i] + "</span><br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}
```


## <a name="extracting-contact-information"></a>Extraction d’informations de contact


Lorsque l’utilisateur  clique sur le bouton Obtenir les informations de contact, le handler d’événements obtient un tableau de contacts avec leurs informations à partir de la propriété contacts de l’objet, si des contacts ont été `myGetContacts` [](/javascript/api/outlook/office.entities#contacts) `_MyEntities` extraits. Chaque contact extrait est stocké en tant [qu’objet Contact](/javascript/api/outlook/office.contact) dans le tableau. `myGetContacts` obtient des données supplémentaires sur chaque contact. Notez que le contexte détermine si Outlook peut extraire un contact d’un élément d’une signature à la fin d’un message électronique, ou au moins certaines des informations suivantes doivent exister à proximité du &mdash; contact.


- La chaîne représentant le nom du contact à partir de la propriété [Contact.personName](/javascript/api/outlook/office.contact#personname).

- La chaîne représentant le nom de l’entreprise associée au contact à partir de la propriété [Contact.businessName](/javascript/api/outlook/office.contact#businessname).

- Le tableau des numéros de téléphone associés au contact à partir de la propriété [Contact.phoneNumbers](/javascript/api/outlook/office.contact#phonenumbers). Chaque numéro de téléphone est représenté par un objet [PhoneNumber](/javascript/api/outlook/office.phonenumber).

- Pour chaque membre **PhoneNumber** dans le tableau des numéros de téléphone, la chaîne représentant le numéro de téléphone à partir de la propriété [PhoneNumber.phoneString](/javascript/api/outlook/office.phonenumber#phonestring).

- Le tableau des URL associées au contact à partir de la propriété [Contact.urls](/javascript/api/outlook/office.contact#urls). Chaque URL est représentée sous la forme d’une chaîne dans un membre de tableau.

- Le tableau des adresses électroniques associées au contact à partir de la propriété [Contact.emailAddresses](/javascript/api/outlook/office.contact#emailaddresses). Chaque adresse électronique est représentée sous la forme d’une chaîne dans un membre de tableau.

- Le tableau des adresses postales associées au contact à partir de la propriété [Contact.addresses](/javascript/api/outlook/office.contact#addresses). Chaque adresse postale est représentée sous la forme d’une chaîne dans un membre de tableau.

`myGetContacts` forme une chaîne HTML locale dans `htmlText` pour afficher les données pour chaque contact. Le code JavaScript associé est présenté ci-dessous.




```js
// Gets instances of the Contact entity on the item.
function myGetContacts()
{
    var htmlText = "";

    // Gets an array of contacts and their information.
    var contactsArray = _MyEntities.contacts;
    for (var i = 0; i < contactsArray.length; i++)
    {
        // Gets the name of the person. The name is a string.
        htmlText += "Name : <span>" + contactsArray[i].personName +
            "</span><br/>";

        // Gets the company name associated with the contact.
        htmlText += "Business : <span>" + 
        contactsArray[i].businessName + "</span><br/>";

        // Gets an array of phone numbers associated with the 
        // contact. Each phone number is represented by a 
        // PhoneNumber object.
        var phoneNumbersArray = contactsArray[i].phoneNumbers;
        for (var j = 0; j < phoneNumbersArray.length; j++)
        {
            htmlText += "PhoneString : <span>" + 
                phoneNumbersArray[j].phoneString + "</span><br/>";
            htmlText += "OriginalPhoneString : <span>" + 
                phoneNumbersArray[j].originalPhoneString +
                "</span><br/>";
        }

        // Gets the URLs associated with the contact.
        var urlsArray = contactsArray[i].urls;
        for (var j = 0; j < urlsArray.length; j++)
        {
            htmlText += "Url : <span>" + urlsArray[j] + 
                "</span><br/>";
        }

        // Gets the email addresses of the contact.
        var emailAddressesArray = contactsArray[i].emailAddresses;
        for (var j = 0; j < emailAddressesArray.length; j++)
        {
           htmlText += "E-mail Address : <span>" + 
               emailAddressesArray[j] + "</span><br/>";
        }

        // Gets postal addresses of the contact.
        var addressesArray = contactsArray[i].addresses;
        for (var j = 0; j < addressesArray.length; j++)
        {
          htmlText += "Address : <span>" + addressesArray[j] + 
              "</span><br/>";
        }

        htmlText += "<hr/>";
        }

    document.getElementById("entities_box").innerHTML = htmlText;
}
```


## <a name="extracting-email-addresses"></a>Extraction des adresses électroniques


Lorsque l’utilisateur clique sur le bouton **Obtenir des adresses électroniques**, le gestionnaire d’événements `myGetEmailAddresses` obtient un tableau d’adresses électroniques SMTP à partir de la propriété [emailAddresses](/javascript/api/outlook/office.entities#emailaddresses) de l’objet `_MyEntities`, si des adresses ont été extraites. Chaque adresse électronique extraite est stockée sous la forme d’une chaîne dans le tableau. `myGetEmailAddresses` forme une chaîne HTML locale dans `htmlText` pour afficher la liste des adresses électroniques extraites. Le code JavaScript associé est présenté ci-dessous.


```js
// Gets instances of the EmailAddress entity on the item.
function myGetEmailAddresses() {
    var htmlText = "";

    // Gets an array of email addresses. Each email address is a 
    // string.
    var emailAddressesArray = _MyEntities.emailAddresses;
    for (var i = 0; i < emailAddressesArray.length; i++) {
        htmlText += "E-mail Address : <span>" + emailAddressesArray[i] + "</span><br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}
```


## <a name="extracting-meeting-suggestions"></a>Extraction de suggestions de réunion


Lorsque l’utilisateur clique sur le bouton **Obtenir des suggestions de réunion**, le gestionnaire d’événements `myGetMeetingSuggestions` obtient un tableau de suggestions de réunion à partir de la propriété [meetingSuggestions](/javascript/api/outlook/office.entities#meetingsuggestions) de l’objet `_MyEntities`, si des suggestions ont été extraites.


 > [!NOTE]
 > Seuls les messages, mais pas les rendez-vous, ne peuvent prendre `MeetingSuggestion` en charge le type d’entité.

Chaque suggestion de réunion extraite est stockée sous la forme d’un objet [MeetingSuggestion](/javascript/api/outlook/office.meetingsuggestion) dans le tableau. `myGetMeetingSuggestions` obtient d’autres données sur chaque suggestion de réunion :


- La chaîne identifiée comme suggestion de réunion à partir de la propriété [MeetingSuggestion.meetingString](/javascript/api/outlook/office.meetingsuggestion#meetingstring).

- Le tableau des participants de la réunion à partir de la propriété [MeetingSuggestion.attendees](/javascript/api/outlook/office.meetingsuggestion#attendees). Chaque participant est représenté par un objet [EmailUser](/javascript/api/outlook/office.emailuser).

- Le nom de chaque participant à partir de la propriété [EmailUser.displayName](/javascript/api/outlook/office.emailuser#displayname).

- L’adresse SMTP de chaque participant à partir de la propriété [EmailUser.emailAddress](/javascript/api/outlook/office.emailuser#emailaddress).

- La chaîne représentant l’emplacement de la suggestion de réunion à partir de la propriété [MeetingSuggestion.location](/javascript/api/outlook/office.meetingsuggestion#location).

- La chaîne représentant l’objet de la suggestion de réunion à partir de la propriété [MeetingSuggestion.subject](/javascript/api/outlook/office.meetingsuggestion#subject).

- La chaîne représentant l’heure de début de la suggestion de réunion à partir de la propriété [MeetingSuggestion.start](/javascript/api/outlook/office.meetingsuggestion#start).

- La chaîne représentant l’heure de fin de la suggestion de réunion à partir de la propriété [MeetingSuggestion.end](/javascript/api/outlook/office.meetingsuggestion#end).

`myGetMeetingSuggestions` forme une chaîne HTML locale dans `htmlText` pour afficher les données pour chacune des suggestions de réunion. Le code JavaScript associé est présenté ci-dessous.




```js
// Gets instances of the MeetingSuggestion entity on the 
// message item.
function myGetMeetingSuggestions() {
    var htmlText = "";

    // Gets an array of MeetingSuggestion objects, each array 
    // element containing an instance of a meeting suggestion 
    // entity from the current item.
    var meetingsArray = _MyEntities.meetingSuggestions;

    // Iterates through each instance of a meeting suggestion.
    for (var i = 0; i < meetingsArray.length; i++) {
        // Gets the string that was identified as a meeting suggestion.
        htmlText += "MeetingString : <span>" + meetingsArray[i].meetingString + "</span><br/>";

        // Gets an array of attendees for that instance of a 
        // meeting suggestion. Each attendee is represented 
        // by an EmailUser object.
        var attendeesArray = meetingsArray[i].attendees;
        for (var j = 0; j < attendeesArray.length; j++) {
            htmlText += "Attendee : ( ";

            // Gets the displayName property of the attendee.
            htmlText += "displayName = <span>" + attendeesArray[j].displayName + "</span> , ";

            // Gets the emailAddress property of each attendee.
            // This is the SMTP address of the attendee.
            htmlText += "emailAddress = <span>" + attendeesArray[j].emailAddress + "</span>";

            htmlText += " )<br/>";
        }

        // Gets the location of the meeting suggestion.
        htmlText += "Location : <span>" + meetingsArray[i].location + "</span><br/>";

        // Gets the subject of the meeting suggestion.
        htmlText += "Subject : <span>" + meetingsArray[i].subject + "</span><br/>";

        // Gets the start time of the meeting suggestion.
        htmlText += "Start time : <span>" + meetingsArray[i].start + "</span><br/>";

        // Gets the end time of the meeting suggestion.
        htmlText += "End time : <span>" + meetingsArray[i].end + "</span><br/>";

        htmlText += "<hr/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}
```


## <a name="extracting-phone-numbers"></a>Extraction de numéros de téléphone


Lorsque l’utilisateur clique sur le bouton **Obtenir des numéros de téléphone**, le gestionnaire d’événements `myGetPhoneNumbers` obtient un tableau de numéros de téléphone à partir de la propriété [phoneNumbers](/javascript/api/outlook/office.entities#phonenumbers) de l’objet `_MyEntities`, si des numéros de téléphone ont été extraits. Chaque numéro de téléphone extrait est stocké sous la forme d’un objet [PhoneNumber](/javascript/api/outlook/office.phonenumber) dans le tableau. `myGetPhoneNumbers` obtient d’autres données sur chaque numéro de téléphone :


- La chaîne représentant le type de numéro de téléphone (par exemple, numéro de téléphone du domicile) à partir de la propriété [PhoneNumber.type](/javascript/api/outlook/office.phonenumber#type).

- La chaîne représentant le numéro de téléphone réel à partir de la propriété [PhoneNumber.phoneString](/javascript/api/outlook/office.phonenumber#phonestring).

- La chaîne qui a été initialement identifiée comme le numéro de téléphone à partir de la propriété [PhoneNumber.originalPhoneString](/javascript/api/outlook/office.phonenumber#originalphonestring).

`myGetPhoneNumbers` forme une chaîne HTML locale dans `htmlText` pour afficher les données pour chacun des numéros de téléphone. Le code JavaScript associé est présenté ci-dessous.




```js
// Gets instances of the phone number entity on the item.
function myGetPhoneNumbers()
{
    var htmlText = "";

    // Gets an array of phone numbers. 
    // Each phone number is a PhoneNumber object.
    var phoneNumbersArray = _MyEntities.phoneNumbers;
    for (var i = 0; i < phoneNumbersArray.length; i++)
    {
        htmlText += "Phone Number : ( ";
        // Gets the type of phone number, for example, home, office.
        htmlText += "type = <span>" + phoneNumbersArray[i].type + 
           "</span> , ";

        // Gets the actual phone number represented by a string.
        htmlText += "phone string = <span>" + 
            phoneNumbersArray[i].phoneString + "</span> , ";

        // Gets the original text that was identified in the item 
        // as a phone number. 
        htmlText += "original phone string = <span>" + 
            phoneNumbersArray[i].originalPhoneString + "</span>";

        htmlText += " )<br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}

```


## <a name="extracting-task-suggestions"></a>Extraction de suggestions de tâches


Lorsque l’utilisateur clique sur le bouton **Obtenir des suggestions de tâches**, le gestionnaire d’événements `myGetTaskSuggestions` obtient un tableau de suggestions de tâches à partir de la propriété [taskSuggestions](/javascript/api/outlook/office.entities#tasksuggestions) de l’objet `_MyEntities`, si des suggestions ont été extraites. Chaque suggestion de tâche extraite est stockée sous la forme d’un objet [TaskSuggestion](/javascript/api/outlook/office.tasksuggestion) dans le tableau. `myGetTaskSuggestions` obtient d’autres données sur chaque suggestion de tâche :


- La chaîne qui a été initialement identifiée comme une suggestion de tâche à partir de la propriété [TaskSuggestion.taskString](/javascript/api/outlook/office.tasksuggestion#taskstring).

- Le tableau des cessionnaires de tâches à partir de la propriété [TaskSuggestion.assignees](/javascript/api/outlook/office.tasksuggestion#assignees). Chaque cessionnaire est représenté par un objet [EmailUser](/javascript/api/outlook/office.emailuser).

- Le nom de chaque cessionnaire à partir de la propriété [EmailUser.displayName](/javascript/api/outlook/office.emailuser#displayname).

- L’adresse SMTP de chaque cessionnaire à partir de la propriété [EmailUser.emailAddress](/javascript/api/outlook/office.emailuser#emailaddress).

`myGetTaskSuggestions` forme une chaîne HTML locale dans `htmlText` pour afficher les données pour chaque suggestion de tâche. Le code JavaScript associé est présenté ci-dessous.




```js
// Gets instances of the task suggestion entity on the item.
function myGetTaskSuggestions()
{
    var htmlText = "";

    // Gets an array of TaskSuggestion objects, each array element 
    // containing an instance of a task suggestion entity from 
    // the current item.
    var tasksArray = _MyEntities.taskSuggestions;

    // Iterates through each instance of a task suggestion.
    for (var i = 0; i < tasksArray.length; i++)
    {
        // Gets the string that was identified as a task suggestion.
        htmlText += "TaskString : <span>" + 
           tasksArray[i].taskString + "</span><br/>";

        // Gets an array of assignees for that instance of a task 
        // suggestion. Each assignee is represented by an 
        // EmailUser object.
        var assigneesArray = tasksArray[i].assignees;
        for (var j = 0; j < assigneesArray.length; j++)
        {
            htmlText += "Assignee : ( ";
            // Gets the displayName property of the assignee.
            htmlText += "displayName = <span>" + assigneesArray[j].displayName + 
               "</span> , ";

            // Gets the emailAddress property of each assignee.
            // This is the SMTP address of the assignee.
            htmlText += "emailAddress = <span>" + assigneesArray[j].emailAddress + 
                "</span>";

            htmlText += " )<br/>";
        }

        htmlText += "<hr/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}

```


## <a name="extracting-urls"></a>Extraction d’URL


Lorsque l’utilisateur clique sur le bouton **Obtenir des URL**, le gestionnaire d’événements `myGetUrls` obtient un tableau d’URL à partir de la propriété [urls](/javascript/api/outlook/office.entities#urls) de l’objet `_MyEntities`, si des URL ont été extraites. Chaque URL extraite est stockée sous la forme d’une chaîne dans le tableau. `myGetUrls` forme une chaîne HTML locale dans `htmlText` pour afficher la liste des URL extraites.


```js
// Gets instances of the URL entity on the item.
function myGetUrls()
{
    var htmlText = "";

    // Gets an array of URLs. Each URL is a string.
    var urlArray = _MyEntities.urls;
    for (var i = 0; i < urlArray.length; i++)
    {
        htmlText += "Url : <span>" + urlArray[i] + "</span><br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}

```


## <a name="clearing-displayed-entity-strings"></a>Effacement des chaînes d’entités affichées


Enfin, le complément pour entités spécifie un gestionnaire d’événements  `myClearEntitiesBox` qui efface les chaînes affichées. Le code associé est présenté ci-dessous.


```js
// Clears the div with id="entities_box".
function myClearEntitiesBox()
{
    document.getElementById("entities_box").innerHTML = "";
}
```


## <a name="javascript-listing"></a>Listing JavaScript


Le listing complet de l’implémentation JavaScript est présenté ci-dessous.


```js
// Global variables
var _Item;
var _MyEntities;

// Initializes the add-in.
Office.initialize = function () {
    var _mailbox = Office.context.mailbox;
    // Obtains the current item.
    _Item = _mailbox.item;
    // Reads all instances of supported entities from the subject 
    // and body of the current item.
    _MyEntities = _Item.getEntities();

    // Checks for the DOM to load using the jQuery ready function.
    $(document).ready(function () {
    // After the DOM is loaded, app-specific code can run.
    });
}


// Clears the div with id="entities_box".
function myClearEntitiesBox()
{
    document.getElementById("entities_box").innerHTML = "";
}

// Gets instances of the Address entity on the item.
function myGetAddresses()
{
    var htmlText = "";

    // Gets an array of postal addresses. Each address is a string.
    var addressesArray = _MyEntities.addresses;
    for (var i = 0; i < addressesArray.length; i++)
    {
        htmlText += "Address : <span>" + addressesArray[i] + 
            "</span><br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}


// Gets instances of the EmailAddress entity on the item.
function myGetEmailAddresses()
{
    var htmlText = "";

    // Gets an array of email addresses. Each email address is a 
    // string.
    var emailAddressesArray = _MyEntities.emailAddresses;
    for (var i = 0; i < emailAddressesArray.length; i++)
    {
        htmlText += "E-mail Address : <span>" + 
            emailAddressesArray[i] + "</span><br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}

// Gets instances of the MeetingSuggestion entity on the 
// message item.
function myGetMeetingSuggestions()
{
    var htmlText = "";

    // Gets an array of MeetingSuggestion objects, each array 
    // element containing an instance of a meeting suggestion 
    // entity from the current item.
    var meetingsArray = _MyEntities.meetingSuggestions;

    // Iterates through each instance of a meeting suggestion.
    for (var i = 0; i < meetingsArray.length; i++)
    {
        // Gets the string that was identified as a meeting 
        // suggestion.
        htmlText += "MeetingString : <span>" + 
            meetingsArray[i].meetingString + "</span><br/>";

        // Gets an array of attendees for that instance of a 
        // meeting suggestion.
        // Each attendee is represented by an EmailUser object.
        var attendeesArray = meetingsArray[i].attendees;
        for (var j = 0; j < attendeesArray.length; j++)
        {
            htmlText += "Attendee : ( ";
            // Gets the displayName property of the attendee.
            htmlText += "displayName = <span>" + attendeesArray[j].displayName + 
                "</span> , ";

            // Gets the emailAddress property of each attendee.
            // This is the SMTP address of the attendee.
            htmlText += "emailAddress = <span>" + attendeesArray[j].emailAddress + 
                "</span>";

            htmlText += " )<br/>";
        }

        // Gets the location of the meeting suggestion.
        htmlText += "Location : <span>" + 
            meetingsArray[i].location + "</span><br/>";

        // Gets the subject of the meeting suggestion.
        htmlText += "Subject : <span>" + 
            meetingsArray[i].subject + "</span><br/>";

        // Gets the start time of the meeting suggestion.
        htmlText += "Start time : <span>" + 
           meetingsArray[i].start + "</span><br/>";

        // Gets the end time of the meeting suggestion.
        htmlText += "End time : <span>" + 
            meetingsArray[i].end + "</span><br/>";

        htmlText += "<hr/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}


// Gets instances of the phone number entity on the item.
function myGetPhoneNumbers()
{
    var htmlText = "";

    // Gets an array of phone numbers. 
    // Each phone number is a PhoneNumber object.
    var phoneNumbersArray = _MyEntities.phoneNumbers;
    for (var i = 0; i < phoneNumbersArray.length; i++)
    {
        htmlText += "Phone Number : ( ";
        // Gets the type of phone number, for example, home, office.
        htmlText += "type = <span>" + phoneNumbersArray[i].type + 
            "</span> , ";

        // Gets the actual phone number represented by a string.
        htmlText += "phone string = <span>" + 
            phoneNumbersArray[i].phoneString + "</span> , ";

        // Gets the original text that was identified in the item 
        // as a phone number. 
        htmlText += "original phone string = <span>" + 
           phoneNumbersArray[i].originalPhoneString + "</span>";

        htmlText += " )<br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}

// Gets instances of the task suggestion entity on the item.
function myGetTaskSuggestions()
{
    var htmlText = "";

    // Gets an array of TaskSuggestion objects, each array element 
    // containing an instance of a task suggestion entity from the 
    // current item.
    var tasksArray = _MyEntities.taskSuggestions;

    // Iterates through each instance of a task suggestion.
    for (var i = 0; i < tasksArray.length; i++)
    {
        // Gets the string that was identified as a task suggestion.
        htmlText += "TaskString : <span>" + 
            tasksArray[i].taskString + "</span><br/>";

        // Gets an array of assignees for that instance of a task 
        // suggestion. Each assignee is represented by an 
        // EmailUser object.
        var assigneesArray = tasksArray[i].assignees;
        for (var j = 0; j < assigneesArray.length; j++)
        {
            htmlText += "Assignee : ( ";
            // Gets the displayName property of the assignee.
            htmlText += "displayName = <span>" + assigneesArray[j].displayName + 
                "</span> , ";

            // Gets the emailAddress property of each assignee.
            // This is the SMTP address of the assignee.
            htmlText += "emailAddress = <span>" + assigneesArray[j].emailAddress + 
                "</span>";

            htmlText += " )<br/>";
        }

        htmlText += "<hr/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}

// Gets instances of the URL entity on the item.
function myGetUrls()
{
    var htmlText = "";

    // Gets an array of URLs. Each URL is a string.
    var urlArray = _MyEntities.urls;
    for (var i = 0; i < urlArray.length; i++)
    {
        htmlText += "Url : <span>" + urlArray[i] + "</span><br/>";
    }

    document.getElementById("entities_box").innerHTML = htmlText;
}

```


## <a name="see-also"></a>Voir aussi

- [Créer des compléments Outlook pour des formulaires de lecture](read-scenario.md)
- [Mettre en correspondance des chaînes dans un élément Outlook en tant qu’entités connues](match-strings-in-an-item-as-well-known-entities.md)
- Méthode [item.getEntities](../reference/objectmodel/preview-requirement-set/office.context.mailbox.item.md#methods)
