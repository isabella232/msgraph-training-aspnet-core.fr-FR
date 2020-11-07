---
ms.openlocfilehash: 30398bafbf47aaa374a200dd1834c9f2003e967f
ms.sourcegitcommit: 9d0d10a9e8e5a1d80382d89bc412df287bee03f3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822353"
---
# <a name="how-to-run-the-completed-project"></a>Exécution du projet terminé

## <a name="prerequisites"></a>Conditions préalables

Pour exécuter le projet terminé dans ce dossier, vous avez besoin des éléments suivants :

- Le [Kit de développement logiciel .net Core](https://dotnet.microsoft.com/download) installé sur votre ordinateur de développement. ( **Remarque :** ce didacticiel a été écrit avec .net Core SDK version 3.1.201. Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais cela n’a pas été testé.)
- Soit un compte Microsoft personnel avec une boîte aux lettres sur Outlook.com, soit un compte professionnel ou scolaire Microsoft.

Si vous n’avez pas de compte Microsoft, vous disposez de deux options pour obtenir un compte gratuit :

- Vous pouvez vous [inscrire pour obtenir un nouveau compte Microsoft personnel](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Vous pouvez vous [inscrire au programme pour les développeurs office 365](https://developer.microsoft.com/office/dev-program) pour obtenir un abonnement gratuit à Office 365.

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a>Enregistrer une application Web avec le centre d’administration Azure Active Directory

1. Ouvrez un navigateur et accédez au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com). Connectez-vous à l’aide d’un **compte personnel** (compte Microsoft) ou d’un **compte professionnel ou scolaire**.

1. Sélectionnez **Azure Active Directory** dans le volet de navigation gauche, puis sélectionnez **Inscriptions d’applications** sous **Gérer**.

    ![Une capture d’écran des inscriptions d’applications ](../tutorial/images/aad-portal-app-registrations.png)

1. Sélectionnez **Nouvelle inscription**. Sur la page **Inscrire une application** , définissez les valeurs comme suit.

    - Définissez le **Nom** sur `ASP.NET Core Graph Tutorial`.
    - Définissez les **Types de comptes pris en charge** sur **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
    - Sous **URI de redirection** , définissez la première flèche déroulante sur `Web`, et la valeur sur `https://localhost:5001/`.

    ![Capture d’écran de la page Inscrire une application](../tutorial/images/aad-register-an-app.png)

1. Sélectionner **Inscription**. Sur la page du **didacticiel Graph de microsoft ASP.net** , copiez la valeur de l' **ID d’application (client)** et enregistrez-la, vous en aurez besoin à l’étape suivante.

    ![Une capture d’écran de l’ID d’application de la nouvelle inscription d'application](../tutorial/images/aad-application-id.png)

1. Sous **Gérer** , sélectionnez **Authentification**. Sous **URI de redirection** , ajoutez un URI avec la valeur `https://localhost:5001/signin-oidc` .

1. Définissez l' **URL de déconnexion** sur `https://localhost:5001/signout-oidc` .

1. Recherchez la section **Octroi implicite** , puis activez **Jetons d’ID**. Sélectionnez **Enregistrer**.

    ![Capture d’écran des paramètres de plateforme Web dans le portail Azure](../tutorial/images/aad-web-platform.png)

1. Sélectionnez **Certificats et secrets** sous **Gérer**. Sélectionnez le bouton **Nouveau secret client**. Entrez une valeur dans **Description** , sélectionnez une des options pour **Expire le** , puis sélectionnez **Ajouter**.

    ![Une capture d’écran de la boîte de dialogue Ajouter une clé secrète client](../tutorial/images/aad-new-client-secret.png)

1. Copiez la valeur due la clé secrète client avant de quitter cette page. Vous en aurez besoin à l’étape suivante.

    > [!IMPORTANT]
    > Ce secret client n’apparaîtra plus jamais, aussi veillez à le copier maintenant.

    ![Une capture d’écran de la clé secrète client nouvellement ajoutée](../tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a>Configurer l’exemple

1. Ouvrez votre interface de ligne de commande (CLI) dans le répertoire où se trouve **GraphTutorial. csproj** , puis exécutez les commandes suivantes, en remplaçant `YOUR_APP_ID` votre ID d’application dans le portail Azure, et `YOUR_APP_SECRET` votre secret d’application.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="run-the-sample"></a>Exécution de l’exemple

Dans votre interface CLI, exécutez la commande suivante pour démarrer l’application.

```Shell
dotnet run
```
