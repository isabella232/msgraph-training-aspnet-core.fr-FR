---
ms.openlocfilehash: 308938efbedc4618c7b0ca3ea6b2eebc0582da10
ms.sourcegitcommit: 9d0d10a9e8e5a1d80382d89bc412df287bee03f3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822368"
---
<!-- markdownlint-disable MD002 MD041 -->

Commencez par créer une application Web ASP.NET principale.

1. Ouvrez votre interface de ligne de commande (CLI) dans un répertoire où vous souhaitez créer le projet. Exécutez la commande suivante.

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. Une fois le projet créé, vérifiez qu’il fonctionne en remplaçant le répertoire actuel par le répertoire **GraphTutorial** et en exécutant la commande suivante dans votre CLI.

    ```Shell
    dotnet run
    ```

1. Ouvrez votre navigateur et accédez à `https://localhost:5001` . Si tout fonctionne, vous devez voir une page principale de ASP.NET par défaut.

> [!IMPORTANT]
> Si vous recevez un avertissement indiquant que le certificat de **localhost** n’est pas approuvé, vous pouvez utiliser l’infrastructure CLI .net pour installer et approuver le certificat de développement. Pour obtenir des instructions sur des systèmes d’exploitation spécifiques, voir [Enforce https in ASP.net Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-3.1) .

## <a name="add-nuget-packages"></a>Ajouter des packages NuGet

Avant de poursuivre, installez des packages NuGet supplémentaires que vous utiliserez plus tard.

- [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) pour la demande et la gestion des jetons d’accès.
- [Microsoft. Identity. Web. MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) pour l’ajout du kit de développement logiciel (SDK) Microsoft Graph via l’injection de dépendance.
- [Microsoft. Identity. Web. UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) pour l’interface utilisateur de connexion et de déconnexion.
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) pour effectuer des appels Microsoft Graph.
- [TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) pour le traitement des identificateurs zonés par heure sur plusieurs plateformes.

1. Exécutez les commandes suivantes dans votre interface CLI pour installer les dépendances.

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.1.0
    dotnet add package Microsoft.Identity.MicrosoftGraph --version 1.1.0
    dotnet add package Microsoft.Identity.Web.UI --version 1.1.0
    dotnet add package Microsoft.Graph --version 3.18.0
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>Concevoir l’application

Dans cette section, vous allez créer la structure de base de l’interface utilisateur de l’application.

### <a name="implement-alert-extension-methods"></a>Implémenter des méthodes d’extension d’alerte

Dans cette section, vous allez créer des méthodes d’extension pour le `IActionResult` type renvoyé par les vues du contrôleur. Cette extension permet de transmettre des messages d’erreur ou de réussite temporaires à l’affichage.

> [!TIP]
> Vous pouvez utiliser n’importe quel éditeur de texte pour modifier les fichiers sources de ce didacticiel. Toutefois, [Visual Studio code](https://code.visualstudio.com/) fournit des fonctionnalités supplémentaires, telles que le débogage et IntelliSense.

1. Créez un répertoire dans le répertoire **GraphTutorial** nommé **Alerts**.

1. Créez un fichier nommé **WithAlertResult.cs** dans le répertoire **./alerts** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. Créez un fichier nommé **AlertExtensions.cs** dans le répertoire **./alerts** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a>Implémenter des méthodes d’extension de données utilisateur

Dans cette section, vous allez créer des méthodes d’extension pour l' `ClaimsPrincipal` objet généré par la plateforme d’identité Microsoft. Cela vous permettra d’étendre l’identité de l’utilisateur existante avec les données de Microsoft Graph.

> [!NOTE]
> Ce code est juste un espace réservé pour l’instant, vous le terminerez dans une section ultérieure.

1. Créez un répertoire dans le répertoire **GraphTutorial** nommé **Graph**.

1. Créez un fichier nommé **GraphClaimsPrincipalExtensions.cs** et ajoutez le code suivant.

    ```csharp
    using System.Security.Claims;

    namespace GraphTutorial
    {
        public static class GraphClaimTypes {
            public const string DisplayName ="graph_name";
            public const string Email = "graph_email";
            public const string Photo = "graph_photo";
            public const string TimeZone = "graph_timezone";
            public const string DateTimeFormat = "graph_datetimeformat";
        }

        // Helper methods to access Graph user data stored in
        // the claims principal
        public static class GraphClaimsPrincipalExtensions
        {
            public static string GetUserGraphDisplayName(this ClaimsPrincipal claimsPrincipal)
            {
                return "Adele Vance";
            }

            public static string GetUserGraphEmail(this ClaimsPrincipal claimsPrincipal)
            {
                return "adelev@contoso.com";
            }

            public static string GetUserGraphPhoto(this ClaimsPrincipal claimsPrincipal)
            {
                return "/img/no-profile-photo.png";
            }
        }
    }
    ```

### <a name="create-views"></a>Créer des vues

Dans cette section, vous allez implémenter les vues Razor pour l’application.

1. Ajoutez un nouveau fichier nommé **_LoginPartial. cshtml** dans le répertoire **./Views/Shared** et ajoutez le code suivant.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. Ajoutez un nouveau fichier nommé **_AlertPartial. cshtml** dans le répertoire **./Views/Shared** et ajoutez le code suivant.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. Ouvrez le fichier **./Views/Shared/_Layout.cshtml** et remplacez l’intégralité de son contenu par le code suivant pour mettre à jour la disposition globale de l’application.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. Ouvrez **./wwwroot/CSS/site.CSS** et ajoutez le code suivant en bas du fichier.

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. Ouvrez le fichier **./Views/Home/index.cshtml** et remplacez son contenu par ce qui suit.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. Créez un répertoire dans le répertoire **./wwwroot** nommé **img**. Ajoutez un fichier image de votre choix nommé **no-profile-photo.png** dans ce répertoire. Cette image sera utilisée comme photo de l’utilisateur lorsque l’utilisateur n’aura pas de photo dans Microsoft Graph.

    > [!TIP]
    > Vous pouvez télécharger l’image utilisée dans ces captures d’écran à partir de [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png).

1. Enregistrez toutes vos modifications et redémarrez le serveur ( `dotnet run` ). À présent, l’application doit être très différente.

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
