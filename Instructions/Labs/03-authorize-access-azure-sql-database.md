---
lab:
  title: "Labo\_3\_: autoriser l’accès à Azure SQL Database avec Microsoft Entra ID"
  module: Implement a Secure Environment for a Database Service
---

# Configurer l’authentification et l’autorisation des bases de données

**Durée estimée : 25 minutes**

Les participants utiliseront les informations acquises dans les leçons pour configurer puis mettre en œuvre la sécurité dans le portail Azure et dans la base de données *AdventureWorksLT*.

Vous avez été embauché en tant qu’administrateur de base de données senior dans le but de sécuriser l’environnement de base de données.

> &#128221; Ces exercices vous demandent de copier et coller du code T-SQL et d’utiliser des ressources SQL existantes. Vérifiez que le code a été copié correctement, avant de l’exécuter.

## Configurez l’environnement.

Si votre machine virtuelle de labo a été fournie et préconfigurée, vous devez trouver les fichiers de labo prêts dans le dossier **C :\LabFiles**. *Prenez un moment pour vérifier et si les fichiers sont déjà là, ignorez cette section*. Toutefois, si vous utilisez votre propre ordinateur ou que les fichiers de labo sont manquants, vous devez les cloner à partir de *GitHub* pour continuer.

1. À partir de la machine virtuelle du labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session Visual Studio Code.

1. Ouvrez la palette de commandes (Ctrl+Maj+P), puis tapez **Git: Clone**. Sélectionnez l’option **Git: Clone**.

1. Collez l’URL suivante dans le champ **URL du référentiel**, puis sélectionnez **Entrée**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Enregistrez le référentiel dans le dossier **C :\LabFiles** sur la machine virtuelle de labo ou votre ordinateur local si elle n’a pas été fournie (créez le dossier s’il n’existe pas).

## Configurer votre serveur SQL Server dans Azure

Connectez-vous à Azure et vérifiez si vous disposez d’une instance Azure SQL Server existante s’exécutant dans Azure. *Ignorez cette section si vous disposez déjà d’une instance SQL Server en cours d’exécution dans Azure*.

1. À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session Visual Studio Code et accédez au référentiel cloné à partir de la section précédente.

1. Cliquez avec le bouton droit sur le dossier **/Allfiles/Labs**, puis sélectionnez **Ouvrir dans le terminal intégré**.

1. Connectons-nous à Azure à l’aide de l’interface Azure CLI. Entrez la commande suivante, puis sélectionnez **Entrée**.

    ```bash
    az login
    ```

    > &#128221; Notez qu’une fenêtre de navigateur s’ouvre. Utilisez vos informations d’identification Azure pour vous connecter.

1. Une fois la connexion établie à Azure, il est temps de créer un groupe de ressources s’il n’existe pas déjà et de créer un serveur SQL et une base de données sous ce groupe de ressources. Entrez la commande suivante, puis sélectionnez **Entrée**. *L’exécution du script prend quelques minutes*.

    ```bash
    cd ./Setup
    ./deploy-sql-database.ps1
    ```

    > &#128221; Notez que par défaut, ce script crée un groupe de ressources appelé **contoso-rg** ou utilise une ressource dont le nom commence par *contoso-rg* s’il existe. Par défaut, il crée également toutes les ressources sur la région **USA Ouest 2** (westus2). Enfin, il génère un mot de passe aléatoire de 12 caractères pour le **mot de passe administrateur SQL**. Vous pouvez modifier ces valeurs à l’aide d’un ou plusieurs des paramètres **-rgName**, **-location** et **-sqlAdminPw** avec vos propres valeurs. Le mot de passe doit répondre aux exigences de complexité du mot de passe Azure SQL, au moins 12 caractères et contenir au moins 1 lettre majuscule, 1 lettre minuscule, 1 chiffre et 1 caractère spécial.

    > &#128221; Notez que le script ajoute votre adresse IP publique actuelle aux règles de pare-feu du serveur SQL.

1. Une fois le script terminé, il renvoie le nom du groupe de ressources, le nom du serveur SQL et le nom de la base de données, ainsi que le nom d’utilisateur ou d’utilisatrice et le mot de passe de l’utilisateur administrateur ou de l’utilisatrice administratrice. Notez ces valeurs, car vous en aurez besoin plus loin dans le labo.

---

## Autoriser l’accès à Azure SQL Database avec Microsoft Entra

Vous pouvez créer des connexions à partir de comptes Microsoft Entra en tant qu’utilisateur de base de données autonome à l’aide de la syntaxe T-SQL `CREATE USER [anna@contoso.com] FROM EXTERNAL PROVIDER`. Un utilisateur de base de données autonome mappe à une identité dans le répertoire Microsoft Entra associé à la base de données, et n’a pas d’informations de connexion dans la base de données `master`.

Avec l’introduction de connexions de serveur Microsoft Entra dans Azure SQL Database, vous pouvez créer des connexions à partir de principaux Microsoft Entra dans la base de données `master` virtuelle d’une SQL Database. Vous pouvez créer des connexions Microsoft Entra à partir des *utilisateurs, groupes et principaux de service* de Microsoft Entra. Pour plus d’informations, consultez [Principaux du serveur Microsoft Entra](/azure/azure-sql/database/authentication-azure-ad-logins)

En outre, vous pouvez utiliser le Portail Azure uniquement pour créer des administrateurs et les rôles de contrôle d’accès en fonction du rôle Azure ne se propagent pas aux serveurs logiques Azure SQL Database. Vous devez accorder des autorisations de serveur et de base de données supplémentaires avec Transact-SQL (T-SQL). Nous allons créer un administrateur Microsoft Entra pour le serveur SQL Server.

1. À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session de navigateur et accédez à [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide de vos informations d’identification Azure.

1. Sur la page d’accueil du portail Azure, recherchez et sélectionnez **Serveurs SQL**, puis sélectionnez-le.

1. Sélectionnez le serveur SQL Server **dp300-lab-xxxxxxxx**, où *xxxxxxxx* est une chaîne numérique aléatoire.

    > &#128221; Notez que si vous utilisez votre propre serveur Azure SQL Server non créé par ce labo, sélectionnez le nom de ce serveur SQL Server.

1. Dans le panneau *Vue d’ensemble*, sélectionnez **Non configuré** en regard d’*Administrateur Microsoft Entra*.

1. Dans l’écran suivant, sélectionnez **Définir l’administrateur**.

1. Dans la barre latérale **Microsoft Entra ID**, recherchez le nom d’utilisateur ou d’utilisatrice Azure avec lequel vous avez effectué votre connexion au portail Azure, puis cliquez sur **Sélectionner**.

1. Sélectionnez **Enregistrer** pour terminer la procédure. Ainsi, votre nom d’utilisateur ou d’utilisatrice désigne l’administrateur ou l’administratrice Microsoft Entra pour le serveur.

1. Sur la gauche, sélectionnez **Vue d’ensemble**, puis copiez le **nom du serveur**.

1. Ouvrez SQL Server Management Studio (SSMS) et sélectionnez **Se connecter** > **Moteur de base de données**. Dans **Nom du serveur**, collez le nom de votre serveur. Modifiez le type d’authentification par **Microsoft Entra MFA**.

1. Sélectionnez **Connecter**.

## Gérer l’accès aux objets de base de données

Dans cette tâche, vous allez gérer l’accès à la base de données et à ses objets. La première chose à faire est de créer deux utilisateurs dans la base de données *AdventureWorksLT*.

1. À partir de la machine virtuelle de labo ou de votre ordinateur local si elle n’a pas été fournie, dans SSMS, connectez-vous à la base de données *AdventureWorksLT* à l’aide du compte d’administration du serveur Azure ou du compte d’administration Microsoft Entra.

1. Utilisez l’**Explorateur d’objets** pour développer **Bases de données**.

1. Cliquez avec le bouton droit sur **AdventureWorksLT**, puis sélectionnez **Nouvelle requête**.

1. Dans la nouvelle fenêtre de requête, copiez et collez le code T-SQL ci-dessous. Exécutez la requête pour créer les deux utilisateurs.

    ```sql
    CREATE USER [DP300User1] WITH PASSWORD = 'Azur3Pa$$';
    GO

    CREATE USER [DP300User2] WITH PASSWORD = 'Azur3Pa$$';
    GO
    ```

    **Remarque** : notez que ces utilisateurs sont créés dans l’étendue de la base de données AdventureWorksLT. Ensuite, vous allez créer un rôle personnalisé et y ajouter les utilisateurs.

1. Exécutez le code T-SQL suivant dans la même fenêtre de requête.

    ```sql
    CREATE ROLE [SalesReader];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User1];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User2];
    GO
    ```

    Créez ensuite une procédure stockée dans le schéma **SalesLT**.

1. Exécutez le code T-SQL ci-dessous dans votre fenêtre de requête.

    ```sql
    CREATE OR ALTER PROCEDURE SalesLT.DemoProc
    AS
    SELECT P.Name, Sum(SOD.LineTotal) as TotalSales ,SOH.OrderDate
    FROM SalesLT.Product P
    INNER JOIN SalesLT.SalesOrderDetail SOD on SOD.ProductID = P.ProductID
    INNER JOIN SalesLT.SalesOrderHeader SOH on SOH.SalesOrderID = SOD.SalesOrderID
    GROUP BY P.Name, SOH.OrderDate
    ORDER BY TotalSales DESC
    GO
    ```

    Utilisez ensuite la syntaxe `EXECUTE AS USER` pour tester la sécurité. Cela permet au moteur de base de données d’exécuter une requête dans le contexte de votre utilisateur.

1. Exécutez le code T-SQL suivant.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    L’opération va échouer avec le message :

    <span style="color:red">Msg 229, Level 14, State 5, Procedure SalesLT.DemoProc, Line 1 [Batch Start Line 0] L’autorisation EXECUTE a été refusée sur l’objet « DemoProc », base de données « AdventureWorksLT », schéma « SalesLT ».</span>

1. Ensuite, accordez des autorisations au rôle pour lui permettre d’exécuter la procédure stockée. Exécutez le code T-SQL ci-dessous.

    ```sql
    REVERT;
    GRANT EXECUTE ON SCHEMA::SalesLT TO [SalesReader];
    GO
    ```

    La première commande rétablit le contexte d’exécution sur le propriétaire de la base de données.

1. Réexécutez le code T-SQL précédent.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

---

## Nettoyer les ressources

Si vous n’utilisez pas Azure SQL Server à d’autres fins, vous pouvez nettoyer les ressources que vous avez créées dans ce labo.

### Supprimer le groupe de ressources

Si vous avez créé un groupe de ressources pour ce labo, vous pouvez supprimer le groupe de ressources pour supprimer toutes les ressources créées dans ce labo.

1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le volet de navigation de gauche ou recherchez les **Groupes de ressources** dans la barre de recherche et sélectionnez-les dans les résultats.

1. Accédez au groupe de ressources que vous avez créé pour ce labo. Le groupe de ressources contient Azure SQL Server et d’autres ressources créées dans ce labo.

1. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

1. Dans la boîte de dialogue **Supprimer un groupe de ressources**, entrez le nom de votre groupe de ressources pour confirmer, puis sélectionnez **Supprimer**.

1. Attendez que le groupe de ressources soit supprimé.

1. Fermez le portail Azure.

### Supprimez les ressources de labo uniquement.

Si vous n’avez pas créé de groupe de ressources pour ce labo et que vous souhaitez laisser le groupe de ressources et ses ressources précédentes intactes, vous pouvez toujours supprimer les ressources créées dans ce labo.

1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le volet de navigation de gauche ou recherchez les **Groupes de ressources** dans la barre de recherche et sélectionnez-les dans les résultats.

1. Accédez au groupe de ressources que vous avez créé pour ce labo. Le groupe de ressources contient Azure SQL Server et d’autres ressources créées dans ce labo.

1. Sélectionnez toutes les ressources précédées du nom SQL Server que vous avez spécifié précédemment dans le labo.

1. Sélectionnez **Supprimer** dans le menu en haut.

1. Dans la boîte de dialogue **Supprimer les ressources**, tapez **Supprimer** et sélectionnez **Supprimer**.

1. Pour confirmer la suppression des ressources, sélectionnez **Supprimer**.

1. Attendez que les ressources soient supprimées.

1. Fermez le portail Azure.

### Supprimer le dossier LabFiles

Si vous avez créé un dossier LabFiles pour ce labo et que vous n’en avez plus besoin, vous pouvez supprimer le dossier LabFiles pour supprimer tous les fichiers créés dans ce labo.

1. À partir de la machine virtuelle de labo ou de votre ordinateur local si elle n’a pas été fournie, ouvrez l’explorateur de fichiers et accédez au lecteur **C:\\**.
1. Cliquez avec le bouton droit sur le dossier **LabFiles** et sélectionnez **Supprimer**.
1. Sélectionnez **Oui** pour confirmer la suppression du dossier.

---

Vous avez terminé ce labo.

Dans cet exercice, vous avez vu comment vous pouvez utiliser Microsoft Entra ID pour accorder aux informations d’identification Azure l’accès à une instance SQL Server hébergée dans Azure. Vous avez également utilisé l’instruction T-SQL pour créer des utilisateurs de base de données et leur accorder des autorisations pour exécuter les procédures stockées.
