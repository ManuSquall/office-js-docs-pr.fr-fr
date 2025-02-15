---
title: Valider un jeton d’identité de complément Outlook
description: Votre complément Outlook peut vous envoyer un jeton d’identité d’utilisateur Exchange, mais avant de faire confiance à la requête, vous devez valider le jeton pour vous assurer qu’il provient du serveur Exchange attendu.
ms.date: 07/07/2020
localization_priority: Normal
ms.openlocfilehash: ba499fa2ece03a326eabb1a48bb19e33c3feea94
ms.sourcegitcommit: 883f71d395b19ccfc6874a0d5942a7016eb49e2c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "53348838"
---
# <a name="validate-an-exchange-identity-token"></a>Valider un jeton d’identité Exchange

Votre complément Outlook peut vous envoyer un jeton d’identité d’utilisateur Exchange, mais avant de faire confiance à la requête, vous devez valider le jeton pour vous assurer qu’il provient du serveur Exchange attendu. Les jetons d’identité d’utilisateur Exchange sont des jetons Web JSON (JWT). Les étapes nécessaires pour valider un jeton JWT sont décrites dans le document [RFC 7519 JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519.txt).

Nous vous suggérons d’utiliser un processus en quatre étapes pour valider le jeton d’identité et obtenir l’identificateur unique de l’utilisateur. Dans un premier temps, extrayez le jeton JWT (JSON Web Token) à partir d’une chaîne d’URL encodée au format base64. Dans un deuxième temps, assurez-vous que le jeton est bien formé, c’est-à-dire qu’il est adapté à votre complément Outlook, qu’il n’a pas expiré et que vous pouvez extraire une URL valide pour le document de métadonnées d’authentification. Dans un troisième temps, récupérez le document de métadonnées d’authentification sur le serveur Exchange et validez la signature jointe au jeton d’identité. Enfin, calculez un identificateur unique pour l’utilisateur en concatenant l’ID Exchange de l’utilisateur avec l’URL du document de métadonnées d’authentification.

## <a name="extract-the-json-web-token"></a>Extraction du jeton Web JSON

Le jeton renvoyé par [getUserIdentityTokenAsync](../reference/objectmodel/preview-requirement-set/office.context.mailbox.md#methods) est une représentation de chaîne encodée du jeton. Dans ce formulaire, conformément au document RFC 7519, tous les jetons JWT se composent de trois parties, séparées par un point. Le format est comme suit.

```json
{header}.{payload}.{signature}
```

L’en-tête et la charge utile doivent être décodés au format base64 pour obtenir une représentation JSON de chaque partie. La signature doit être décodée au format base64 pour obtenir un tableau d’octets contenant la signature binaire.

Pour plus d’informations sur le contenu du jeton, consultez la section [Présentation du jeton d’identité Exchange](inside-the-identity-token.md).

Une fois les trois composants décodés, vous pouvez poursuivre avec la validation du contenu du jeton.

## <a name="validate-token-contents"></a>Validation du contenu du jeton

Pour valider le contenu du jeton, vous devez vérifier les éléments suivants :

- Vérifiez l’en-tête et vérifiez que :
  - `typ` est définie sur `JWT` .
  - `alg` est définie sur `RS256` .
  - `x5t` est présente.

- Vérifiez la charge utile et vérifiez que :
  - `amurl` est définie sur l’emplacement d’un fichier manifeste de clé de signature de jetons `appctx` autorisé. Par exemple, la valeur `amurl` attendue pour Microsoft 365 est https://outlook.office365.com:443/autodiscover/metadata/json/1 . Voir la section suivante [Vérifier le domaine pour](#verify-the-domain) plus d’informations.
  - L’heure actuelle se trouve entre les heures spécifiées dans les `nbf` `exp` revendications. La revendication `nbf` spécifie le début de la période où le jeton est considéré comme valide et la revendication `exp` spécifie le délai d’expiration pour le jeton. Ceci est recommandé pour permettre certains écarts dans les paramètres de l’horloge entre les serveurs.
  - `aud` est l’URL attendue pour votre add-in.
  - `version` à `appctx` l’intérieur de la revendication est définie sur `ExIdTok.V1` .

### <a name="verify-the-domain"></a>Vérifier le domaine

Lorsque vous implémentez la logique de vérification décrite précédemment dans cette section, vous devez également exiger que le domaine de la revendication corresponde au domaine de découverte automatique `amurl` de l’utilisateur. Pour ce faire, vous devez utiliser ou implémenter la découverte automatique. Pour en savoir plus, vous pouvez commencer par la découverte automatique [pour Exchange](/exchange/client-developer/exchange-web-services/autodiscover-for-exchange).

## <a name="validate-the-identity-token-signature"></a>Validation de la signature du jeton d’identité

Une fois que vous savez que le jeton JWT contient les revendications requises, poursuivez avec la validation de la signature du jeton.

### <a name="retrieve-the-public-signing-key"></a>Récupération de la clé de signature publique

La première étape consiste à récupérer la clé publique qui correspond au certificat que le serveur Exchange a utilisé pour signer le jeton. La clé est disponible dans le document de métadonnées d’authentification. Ce document est un fichier JSON hébergé dans l’URL spécifiée dans la réclamation `amurl`.

Le document de métadonnées d’authentification utilise le format suivant.

```json
{
    "id": "_70b34511-d105-4e2b-9675-39f53305bb01",
    "version": "1.0",
    "name": "Exchange",
    "realm": "*",
    "serviceName": "00000002-0000-0ff1-ce00-000000000000",
    "issuer": "00000002-0000-0ff1-ce00-000000000000@*",
    "allowedAudiences": [
        "00000002-0000-0ff1-ce00-000000000000@*"
    ],
    "keys": [
        {
            "usage": "signing",
            "keyinfo": {
                "x5t": "enh9BJrVPU5ijV1qjZjV-fL2bco"
            },
            "keyvalue": {
                "type": "x509Certificate",
                "value": "MIIHNTCC..."
            }
        }
    ],
    "endpoints": [
        {
            "location": "https://by2pr06mb2229.namprd06.prod.outlook.com:444/autodiscover/metadata/json/1",
            "protocol": "OAuth2",
            "usage": "metadata"
        }
    ]
}
```

Les clés de signature disponibles sont dans le tableau `keys`. Sélectionnez la clé correcte en vérifiant que la valeur `x5t` dans la propriété `keyinfo` correspond à la valeur `x5t` dans l’en-tête du jeton. La clé publique est à l’intérieur de la propriété `value` dans la propriété `keyvalue`. Elle est stockée sous la forme d’un tableau d’octets codé au format base64.

Une fois que vous avez trouvé la bonne clé publique, vérifiez la signature. Les données signées correspondent aux deux premières parties du jeton codé, séparées par un point :

```json
{header}.{payload}
```

## <a name="compute-the-unique-id-for-an-exchange-account"></a>Calculer l’ID unique d’un compte Exchange

Vous pouvez créer un identificateur unique pour un compte Exchange en concatenant l’URL du document de métadonnées d’authentification avec l’identificateur Exchange du compte. Lorsque vous avez cet identificateur unique, vous pouvez l’utiliser pour créer un système de connexion unique (SSO) pour le service Web de votre complément Outlook. Pour plus d’informations sur l’utilisation de l’identificateur unique pour l’authentification unique, consultez la section [Authentifier un utilisateur avec un jeton d’identité pour Exchange](authenticate-a-user-with-an-identity-token.md).

## <a name="use-a-library-to-validate-the-token"></a>Utiliser une bibliothèque pour valider le jeton

Il existe un certain nombre de bibliothèques qui permettent une analyse et une validation générales du jeton JWT. Microsoft fournit la bibliothèque qui peut être utilisée pour valider les `System.IdentityModel.Tokens.Jwt` jetons Exchange’identité de l’utilisateur.

> [!IMPORTANT]
> Nous ne recommandons plus l’API gérée des services web Exchange, car la Microsoft.Exchange.WebServices.Auth.dll, bien que toujours disponible, est désormais obsolète et s’appuie sur des bibliothèques non pris en charge telles que Microsoft.IdentityModel.Extensions.dll.

### <a name="systemidentitymodeltokensjwt"></a>System.IdentityModel.Tokens.Jwt

La bibliothèque [System.IdentityModels.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) peut analyser le jeton et également effectuer la validation, même si vous devez analyser la réclamation `appctx` vous-même et récupérer la clé de signature publique.

```cs
// Load the encoded token
string encodedToken = "...";
JwtSecurityToken jwt = new JwtSecurityToken(encodedToken);

// Parse the appctx claim to get the auth metadata url
string authMetadataUrl = string.Empty;
var appctx = jwt.Claims.FirstOrDefault(claim => claim.Type == "appctx");
if (appctx != null)
{
    var AppContext = JsonConvert.DeserializeObject<ExchangeAppContext>(appctx.Value);

    // Token version check
    if (string.Compare(AppContext.Version, "ExIdTok.V1", StringComparison.InvariantCulture) != 0) {
        // Fail validation
    }

    authMetadataUrl = AppContext.MetadataUrl;
}

// Use System.IdentityModel.Tokens.Jwt library to validate standard parts
JwtSecurityTokenHandler tokenHandler = new JwtSecurityTokenHandler();
TokenValidationParameters tvp = new TokenValidationParameters();

tvp.ValidateIssuer = false;
tvp.ValidateAudience = true;
tvp.ValidAudience = "{URL to add-in}";
tvp.ValidateIssuerSigningKey = true;
// GetSigningKeys downloads the auth metadata doc and
// returns a List<SecurityKey>
tvp.IssuerSigningKeys = GetSigningKeys(authMetadataUrl);
tvp.ValidateLifetime = true;

try
{
    var claimsPrincipal = tokenHandler.ValidateToken(encodedToken, tvp, out SecurityToken validatedToken);

    // If no exception, all standard checks passed
}
catch (SecurityTokenValidationException ex)
{
    // Validation failed
}
```

<br/>

La classe `ExchangeAppContext` est définie comme suit :

```cs
using Newtonsoft.Json;

/// <summary>
/// Representation of the appctx claim in an Exchange user identity token.
/// </summary>
public class ExchangeAppContext
{
    /// <summary>
    /// The Exchange identifier for the user
    /// </summary>
    [JsonProperty("msexchuid")]
    public string ExchangeUid { get; set; }

    /// <summary>
    /// The token version
    /// </summary>
    public string Version { get; set; }

    /// <summary>
    /// The URL to download authentication metadata
    /// </summary>
    [JsonProperty("amurl")]
    public string MetadataUrl { get; set; }
}
```

Pour un exemple qui utilise cette bibliothèque pour valider les jetons Exchange et qui a une implémentation de `GetSigningKeys`, reportez-vous à [Outlook-Add-In-Token-Viewer](https://github.com/OfficeDev/Outlook-Add-In-Token-Viewer).

## <a name="see-also"></a>Voir aussi

- [Outlook-Add-In-Token-Viewer](https://github.com/OfficeDev/Outlook-Add-In-Token-Viewer)
- [Outlook-Add-in-JavaScript-ValidateIdentityToken](https://github.com/OfficeDev/Outlook-Add-in-JavaScript-ValidateIdentityToken)
