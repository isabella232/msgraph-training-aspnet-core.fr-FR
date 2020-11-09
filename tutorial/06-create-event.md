---
ms.openlocfilehash: f70714a0bc2588fc67d63096b4ab746380bb8e2c
ms.sourcegitcommit: 9d0d10a9e8e5a1d80382d89bc412df287bee03f3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822329"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="3f54f-101">Dans cette section, vous allez ajouter la possibilité de créer des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="3f54f-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="create-model"></a><span data-ttu-id="3f54f-102">Créer un modèle</span><span class="sxs-lookup"><span data-stu-id="3f54f-102">Create model</span></span>

1. <span data-ttu-id="3f54f-103">Créez un fichier nommé **NewEvent.cs** dans le répertoire **./Models** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="3f54f-103">Create a new file named **NewEvent.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/NewEvent.cs" id="NewEventSnippet":::

## <a name="create-view"></a><span data-ttu-id="3f54f-104">Créer un affichage</span><span class="sxs-lookup"><span data-stu-id="3f54f-104">Create view</span></span>

1. <span data-ttu-id="3f54f-105">Créez un fichier nommé **New. cshtml** dans le répertoire HE **./views/Calendar** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="3f54f-105">Create a new file named **New.cshtml** in he **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/New.cshtml" id="NewFormSnippet":::

## <a name="add-controller-actions"></a><span data-ttu-id="3f54f-106">Ajouter des actions de contrôleur</span><span class="sxs-lookup"><span data-stu-id="3f54f-106">Add controller actions</span></span>

1. <span data-ttu-id="3f54f-107">Ouvrez **./Controllers/CalendarController.cs** et ajoutez l’action suivante à la `CalendarController` classe pour afficher le nouveau formulaire d’événement.</span><span class="sxs-lookup"><span data-stu-id="3f54f-107">Open **./Controllers/CalendarController.cs** and add the following action to the `CalendarController` class to render the new event form.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="CalendarNewGetSnippet":::

1. <span data-ttu-id="3f54f-108">Ajoutez l’action suivante à la `CalendarController` classe pour recevoir le nouvel événement à partir du formulaire lorsque l’utilisateur clique sur **Enregistrer** et utilisez Microsoft Graph pour ajouter l’événement au calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="3f54f-108">Add the following action to the `CalendarController` class to receive the new event from the form when the user clicks **Save** and use Microsoft Graph to add the event to the user's calendar.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="CalendarNewPostSnippet":::

1. <span data-ttu-id="3f54f-109">Démarrez l’application, connectez-vous, puis cliquez sur le lien **Calendrier**.</span><span class="sxs-lookup"><span data-stu-id="3f54f-109">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="3f54f-110">Cliquez sur le bouton **nouvel événement** , remplissez le formulaire, puis cliquez sur **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="3f54f-110">Click the **New event** button, fill in the form, and click **Save**.</span></span>

    ![Capture d’écran du nouveau formulaire d’événement](./images/create-event-01.png)
