---
ms.openlocfilehash: 17394dd6283464eabcbea1f60c48640412b55431
ms.sourcegitcommit: 9d0d10a9e8e5a1d80382d89bc412df287bee03f3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822380"
---
<!-- markdownlint-disable MD002 MD041 -->

Dans cette section, vous allez incorporer Microsoft Graph dans l’application. Pour cette application, vous allez utiliser la [bibliothèque cliente Microsoft Graph pour .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) pour effectuer des appels à Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Récupérer les événements de calendrier à partir d’Outlook

Commencez par créer un nouveau contrôleur pour les affichages de calendrier.

1. Ajoutez un nouveau fichier nommé **CalendarController.cs** dans le répertoire **./Controllers** et ajoutez le code suivant.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Graph;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        public class CalendarController : Controller
        {
            private readonly GraphServiceClient _graphClient;
            private readonly ILogger<HomeController> _logger;

            public CalendarController(
                GraphServiceClient graphClient,
                ILogger<HomeController> logger)
            {
                _graphClient = graphClient;
                _logger = logger;
            }
        }
    }
    ```

1. Ajoutez les fonctions suivantes à la `CalendarController` classe pour obtenir l’affichage Calendrier de l’utilisateur.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    Examinez le contenu du code `GetUserWeekCalendar` .

    - Il utilise le fuseau horaire de l’utilisateur pour obtenir les valeurs de date/heure de début et de fin UTC de la semaine.
    - Il interroge l' [affichage Calendrier](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) de l’utilisateur pour obtenir tous les événements compris entre les dates de début et de fin. L’utilisation d’un affichage Calendrier au lieu de [répertorier les événements](/graph/api/user-list-events?view=graph-rest-1.0) développe les événements périodiques, en renvoyant les occurrences qui se produisent dans la fenêtre de temps spécifiée.
    - Il utilise l' `Prefer: outlook.timezone` en-tête pour récupérer les résultats dans le fuseau horaire de l’utilisateur.
    - Il utilise `Select` pour limiter les champs qui reviennent à ceux utilisés par l’application.
    - Il est utilisé `OrderBy` pour trier les résultats par ordre chronologique.
    - Il utilise une `PageIterator` [page to via la collection Events](/graph/sdks/paging). Cela gère le cas où l’utilisateur dispose d’un plus grand nombre d’événements sur son calendrier que la taille de page demandée.

1. Ajoutez la fonction suivante à la `CalendarController` classe pour implémenter une vue temporaire des données renvoyées.

    ```csharp
    // Minimum permission scope needed for this view
    [AuthorizeForScopes(Scopes = new[] { "Calendars.Read" })]
    public async Task<IActionResult> Index()
    {
        try
        {
            var userTimeZone = TZConvert.GetTimeZoneInfo(
                User.GetUserGraphTimeZone());
            var startOfWeek = CalendarController.GetUtcStartOfWeekInTimeZone(
                DateTime.Today, userTimeZone);

            var events = await GetUserWeekCalendar(startOfWeek);

            // Return a JSON dump of events
            return new ContentResult {
                Content = _graphClient.HttpProvider.Serializer.SerializeObject(events),
                ContentType = "application/json"
            };
        }
        catch (ServiceException ex)
        {
            if (ex.InnerException is MicrosoftIdentityWebChallengeUserException)
            {
                throw ex;
            }

            return new ContentResult {
                Content = $"Error getting calendar view: {ex.Message}",
                ContentType = "text/plain"
            };
        }
    }
    ```

1. Démarrez l’application, connectez-vous, puis cliquez sur le lien **Calendrier** dans la barre de navigation. Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

Vous pouvez désormais ajouter une vue pour afficher les résultats de façon plus parlante.

### <a name="create-view-models"></a>Créer des modèles d’affichage

1. Créez un fichier nommé **CalendarViewEvent.cs** dans le répertoire **./Models** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. Créez un fichier nommé **DailyViewModel.cs** dans le répertoire **./Models** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. Créez un fichier nommé **CalendarViewModel.cs** dans le répertoire **./Models** et ajoutez le code suivant.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a>Créer des vues

1. Créez un répertoire nommé **calendrier** dans le répertoire **./views** .

1. Créez un fichier nommé **_DailyEventsPartial. cshtml** dans le répertoire **./views/Calendar** et ajoutez le code suivant.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. Créez un fichier nommé **index. cshtml** dans le répertoire **./views/Calendar** et ajoutez le code suivant.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a>Mettre à jour le contrôleur de calendrier

1. Ouvrez **./Controllers/CalendarController.cs** et remplacez la `Index` fonction existante par ce qui suit.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. Démarrez l’application, connectez-vous, puis cliquez sur le lien **Calendrier**. L’application doit désormais afficher un tableau des événements.

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
