---
ms.openlocfilehash: 6e6c476b4ff0901f50d8e35a17f584d73b48b533
ms.sourcegitcommit: 9d0d10a9e8e5a1d80382d89bc412df287bee03f3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822361"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cette opération est obligatoire pour obtenir le jeton d’accès OAuth nécessaire pour appeler l’API Microsoft Graph. Dans cette étape, vous allez configurer la bibliothèque [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) .

> [!IMPORTANT]
> Pour éviter le stockage de l’ID et de la clé secrète de l’application dans la source, utilisez le [Gestionnaire de secret .net](/aspnet/core/security/app-secrets) pour stocker ces valeurs. Le gestionnaire de secrets est destiné uniquement à des fins de développement, les applications de production doivent utiliser un gestionnaire de secret approuvé pour le stockage de secrets.

1. Ouvrez **./appsettings.jssur** et remplacez son contenu par ce qui suit.

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. Ouvrez la CLI dans le répertoire où se trouve **GraphTutorial. csproj** , puis exécutez les commandes suivantes, en remplaçant `YOUR_APP_ID` votre ID d’application dans le portail Azure, et `YOUR_APP_SECRET` votre clé d’application secrète.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Implémentation de la connexion

Commencez par ajouter les services de plateforme d’identité Microsoft à l’application.

1. Créez un fichier nommé **GraphConstants.cs** dans le répertoire **./Graph** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. Ouvrez le fichier **./Startup.cs** et ajoutez les `using` instructions suivantes en haut du fichier.

    ```csharp
    using Microsoft.AspNetCore.Authentication.OpenIdConnect;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc.Authorization;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.UI;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.Graph;
    using System.Net;
    using System.Net.Http.Headers;
    ```

1. Remplacez la fonction `ConfigureServices` existante par ce qui suit.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services
            // Use OpenId authentication
            .AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
            // Specify this is a web app and needs auth code flow
            .AddMicrosoftIdentityWebApp(Configuration)
            // Add ability to call web API (Graph)
            // and get access tokens
            .EnableTokenAcquisitionToCallDownstreamApi(options => {
                Configuration.Bind("AzureAd", options);
            }, GraphConstants.Scopes)
            // Use in-memory token cache
            // See https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization
            .AddInMemoryTokenCaches();

        // Require authentication
        services.AddControllersWithViews(options =>
        {
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .Build();
            options.Filters.Add(new AuthorizeFilter(policy));
        })
        // Add the Microsoft Identity UI pages for signin/out
        .AddMicrosoftIdentityUI();
    }
    ```

1. Dans la `Configure` fonction, ajoutez la ligne suivante au-dessus de la `app.UseAuthorization();` ligne.

    ```csharp
    app.UseAuthentication();
    ```

1. Ouvrez **./Controllers/HomeController.cs** et remplacez son contenu par ce qui suit.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using System.Diagnostics;
    using System.Threading.Tasks;

    namespace GraphTutorial.Controllers
    {
        public class HomeController : Controller
        {
            ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<HomeController> _logger;

            // Get the ITokenAcquisition interface via
            // dependency injection
            public HomeController(
                ITokenAcquisition tokenAcquisition,
                ILogger<HomeController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _logger = logger;
            }

            public async Task<IActionResult> Index()
            {
                // TEMPORARY
                // Get the token and display it
                try
                {
                    string token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(GraphConstants.Scopes);
                    return View().WithInfo("Token acquired", token);
                }
                catch (MicrosoftIdentityWebChallengeUserException)
                {
                    return Challenge();
                }
            }

            public IActionResult Privacy()
            {
                return View();
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            public IActionResult Error()
            {
                return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            [AllowAnonymous]
            public IActionResult ErrorWithMessage(string message, string debug)
            {
                return View("Index").WithError(message, debug);
            }
        }
    }
    ```

1. Enregistrez vos modifications et démarrez le projet. Connectez-vous avec votre compte Microsoft.

1. Examinez l’invite de consentement. La liste des autorisations correspond à la liste des étendues d’autorisations configurées dans **./Graph/GraphConstants.cs**.

    - **Conserver l’accès aux données auxquelles vous avez accordé l’accès à :** ( `offline_access` ) cette autorisation est demandée par MSAL afin de récupérer les jetons d’actualisation.
    - **Connectez-vous et lisez votre profil :** ( `User.Read` ) cette autorisation permet à l’application d’obtenir le profil de l’utilisateur connecté et sa photo de profil.
    - **Lire les paramètres de votre boîte aux lettres :** ( `MailboxSettings.Read` ) cette autorisation permet à l’application de lire les paramètres de boîte aux lettres de l’utilisateur, notamment le fuseau horaire et le format de l’heure.
    - **Avoir un accès total à vos calendriers :** ( `Calendars.ReadWrite` ) cette autorisation permet à l’application de lire des événements sur le calendrier de l’utilisateur, d’ajouter de nouveaux événements et de modifier des événements existants.

    ![Capture d’écran de l’invite de consentement de la plateforme d’identité Microsoft](./images/add-aad-auth-03.png)

    Pour plus d’informations sur le consentement, consultez la rubrique [Understanding Azure ad Presentation Experience](/azure/active-directory/develop/application-consent-experience).

1. Consentement des autorisations demandées. Le navigateur vous redirige vers l’application, affichant le jeton.

### <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

Une fois que l’utilisateur a ouvert une session, vous pouvez obtenir ses informations à partir de Microsoft Graph.

1. Ouvrez **./Graph/GraphClaimsPrincipalExtensions.cs** et remplacez l’intégralité de son contenu par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. Ouvrez **./Startup.cs** et remplacez la `.AddMicrosoftIdentityWebApp(Configuration)` ligne existante par le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    Examinez ce que fait ce code.

    - Il ajoute un gestionnaire d’événements pour l' `OnTokenValidated` événement.
        - Il utilise l' `ITokenAcquisition` interface pour obtenir un jeton d’accès.
        - Il appelle Microsoft Graph pour obtenir le profil et la photo de l’utilisateur.
        - Il ajoute les informations de graphique à l’identité de l’utilisateur.

1. Ajoutez l’appel de fonction suivant après l' `EnableTokenAcquisitionToCallDownstreamApi` appel et avant l' `AddInMemoryTokenCaches` appel.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    Cela rendra un **GraphServiceClient** authentifié accessible aux contrôleurs via l’injection de dépendance.

1. Ouvrez **./Controllers/HomeController.cs** et remplacez la `Index` fonction par ce qui suit.

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. Supprimez toutes les références à `ITokenAcquisition` dans la classe **HomeController** .

1. Enregistrez vos modifications, démarrez l’application et parcourez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** . Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a>Stockage et actualisation des jetons

À ce stade, votre application a un jeton d’accès, qui est envoyé dans l' `Authorization` en-tête des appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Cependant, ce jeton est de courte durée. Le jeton expire une heure après son émission. C’est là que le jeton d’actualisation devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.

Étant donné que l’application utilise la bibliothèque Microsoft. Identity. Web, vous n’avez pas besoin d’implémenter de logique d’actualisation ou de stockage de jetons.

L’application utilise le cache de jetons en mémoire, ce qui est suffisant pour les applications qui n’ont pas besoin de faire persister les jetons lors du redémarrage de l’application. Les applications de production peuvent utiliser à la place les [options de cache distribué](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) dans la bibliothèque Microsoft. Identity. Web.

La `GetAccessTokenForUserAsync` méthode gère l’expiration du jeton et l’actualisation pour vous. Il vérifie d’abord le jeton mis en cache et, s’il n’a pas expiré, il le renvoie. Si elle a expiré, elle utilise le jeton d’actualisation mis en cache pour en obtenir une nouvelle.

Le **GraphServiceClient** que les contrôleurs obtiennent via l’injection de dépendances est préconfiguré avec un fournisseur d’authentification qui s’utilise `GetAccessTokenForUserAsync` pour vous.
