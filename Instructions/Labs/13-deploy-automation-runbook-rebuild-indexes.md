---
lab:
  title: "Labo\_13\_: Déployer un runbook Automation pour reconstruire automatiquement les index"
  module: Automate database tasks for Azure SQL
---

# Déployer un runbook Automation pour reconstruire automatiquement les index

**Durée estimée : 30 minutes**

Vous avez été embauché en tant qu’administrateur de bases de données senior pour aider à automatiser les opérations quotidiennes d’administration de base de données. Cette automatisation a pour objectif de s’assurer que les bases de données pour AdventureWorks continuent de fonctionner à des performances optimales, ainsi que de fournir des méthodes pour générer des alertes en fonction de certains critères. AdventureWorks utilise SQL Server à la fois comme infrastructure en tant que service (IaaS) et plateforme en tant que service (PaaS).

**Remarque :** Ces exercices peuvent vous demander de copier et coller du code T-SQL et d’utiliser des ressources SQL existantes. Vérifiez que le code a été copié correctement, avant de l’exécuter.

## Créer un compte Automation

1. Depuis la machine virtuelle du labo, démarrez une session de navigateur et naviguez vers [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide du **nom d’utilisateur** et du **mot de passe** Azure fournis dans l’onglet **Ressources** de cette machine virtuelle de labo.

    ![Capture d’écran de la page de connexion du Portail Azure](../images/dp-300-module-01-lab-01.png)

1. Dans le Portail Azure, dans la barre de recherche, tapez *Automation*, puis sélectionnez **Comptes Automation** dans les résultats de la recherche, puis cliquez sur **+ Créer**.

    ![Capture d’écran de la sélection des comptes Automation.](../images/dp-300-module-13-lab-01.png)

1. Sur la page **Créer un compte Automation**, saisissez les informations ci-dessous, puis sélectionnez **Vérifier + Créer**.

    - **Groupe de ressources** : &lt;Votre groupe de ressources.&gt;
    - **Nom** : autoAccount
    - **Emplacement** : utilisez la valeur par défaut.

    ![Capture d’écran de l’écran Ajouter un compte Automation.](../images/dp-300-module-13-lab-02.png)

1. Dans la page Vérifier, sélectionnez **Créer**.

    ![Capture d’écran de l’écran Ajouter un compte Automation.](../images/dp-300-module-13-lab-29.png)

    > [!NOTE]
    > Votre compte Automation devrait être créé dans les trois minutes.

## Se connecter à une base de données Azure SQL

1. Dans le Portail Azure, accédez à votre base de données en recherchant les **bases de données SQL**.

    ![Capture d’écran de la recherche des bases de données SQL existantes.](../images/dp-300-module-13-lab-03.png)

1. Sélectionnez la base de données SQL **AdventureWorksLT**.

    ![Capture d’écran de la sélection de la base de données SQL AdventureWorks.](../images/dp-300-module-13-lab-04.png)

1. Dans la section principale de la page de votre base de données SQL, sélectionnez **Éditeur de requêtes (préversion)**.

    ![Capture d’écran montrant la sélection de l’Éditeur de requêtes (préversion).](../images/dp-300-module-13-lab-05.png)

1. Vous serez invité à entrer vos informations d’identification pour vous connecter à votre base de données. Utilisez ces informations d’identification :

    - **Identifiant de connexion :** sqladmin
    - **Mot de passe** : P@ssw0rd01

1. Vous devriez obtenir le message d’erreur suivant :

    ![Capture d’écran de l’erreur de connexion.](../images/dp-300-module-13-lab-06.png)

1. Sélectionnez le lien **Liste d’adresses IP autorisées…** fourni à la fin du message d’erreur ci-dessus. Cela ajoutera automatiquement votre adresse IP client comme entrée de règle de pare-feu pour votre base de données SQL.

    ![Capture d’écran de la création de la règle de pare-feu.](../images/dp-300-module-13-lab-07.png)

1. Revenez à l’Éditeur de requête, puis sélectionnez **OK**pour vous connecter à votre base de données.

1. Ouvrez un nouvel onglet dans votre navigateur et naviguez vers la page GitHub pour accéder au script [**AdaptativeIndexDefragmentation**](https://github.com/microsoft/tigertoolbox/blob/master/AdaptiveIndexDefrag/usp_AdaptiveIndexDefrag.sql). Sélectionnez ensuite **Brut**.

    ![Capture d’écran de la sélection de Raw dans GitHub.](../images/dp-300-module-13-lab-08.png)

    Cela affiche le code dans un format que vous pouvez copier. Sélectionnez tout le texte (<kbd>Ctrl</kbd> + <kbd>A</kbd>), puis copiez-le dans le Presse-papiers ( <kbd>Ctrl</kbd> + <kbd>C</kbd>).

    >[!NOTE]
    > Le but de ce script est d’effectuer une défragmentation intelligente sur un ou plusieurs index, ainsi que la mise à jour des statistiques requises, pour une ou plusieurs bases de données.

1. Fermez l’onglet du navigateur GitHub et revenez dans le portail Azure.

1. Collez le texte que vous avez copié dans le volet **Requête 1**.

    ![Capture d’écran du collage du code dans une nouvelle fenêtre de requête.](../images/dp-300-module-13-lab-09.png)

1. Supprimez `USE msdb` et `GO` sur les lignes 5 et 6 de la requête (qui sont mises en surbrillance dans la capture d’écran), puis sélectionnez **Exécuter**.

1. Développez le dossier **Procédures stockées** pour voir ce qui a été créé.

    ![Capture d’écran des nouvelles procédures stockées.](../images/dp-300-module-13-lab-10.png)

## Configurer les ressources du compte Automation

Les étapes suivantes consistent à configurer les ressources nécessaires pour préparer la création du runbook. Sélectionnez ensuite **Comptes Automation**.

1. Dans le portail Azure, dans la zone de recherche supérieure, tapez **automation**.

    ![Capture d’écran de la sélection des comptes Automation.](../images/dp-300-module-13-lab-11.png)

1. Sélectionnez le compte Automation que vous venez de créer.

    ![Capture d’écran de la sélection du compte Automation autoAccount.](../images/dp-300-module-13-lab-12.png)

1. Sélectionnez **Modules** dans la section **Ressources partagées** du volet Automation. Sélectionnez ensuite **Parcourir la galerie**.

    ![Capture d’écran de la sélection du menu Modules.](../images/dp-300-module-13-lab-13.png)

1. Recherchez **sqlserver** dans la Galerie.

    ![Capture d’écran de la sélection du module SqlServer.](../images/dp-300-module-13-lab-14.png)

1. Sélectionnez **SqlServer** afin d’accéder à l’écran suivant, puis sélectionnez **Sélectionner**.

    ![Capture d’écran de la sélection de l’option Sélectionner.](../images/dp-300-module-13-lab-15.png)

1. Sur la page **Ajouter un module**, sélectionnez la dernière version du runtime disponible, puis sélectionnez **Importer**. Le module PowerShell est alors importé dans votre compte Automation.

    ![Capture d’écran de la sélection de l’option Importer.](../images/dp-300-module-13-lab-16.png)

1. Vous devez créer des informations d’identification pour vous connecter de manière sécurisée à votre base de données. Dans le volet du compte Automation, accédez à la section **Ressources partagées** et sélectionnez **Informations d’identification**.

    ![Capture d’écran de la sélection de l’option Informations d’identification.](../images/dp-300-module-13-lab-17.png)

1. Sélectionnez **+ Ajouter des informations d’identification**, entrez les informations ci-dessous, puis sélectionnez **Créer**.

    - Nom : **SQLUser**
    - Nom d’utilisateur : **sqladmin**
    - Mot de passe : **P@ssw0rd01**
    - Confirmer le mot de passe : **P@ssw0rd01**

    ![Capture d’écran de l’ajout d’informations d’identification de compte.](../images/dp-300-module-13-lab-18.png)

## Créer un runbook PowerShell

1. Dans le Portail Azure, accédez à votre base de données en recherchant les **bases de données SQL**.

    ![Capture d’écran de la recherche des bases de données SQL existantes.](../images/dp-300-module-13-lab-03.png)

1. Sélectionnez la base de données SQL **AdventureWorksLT**.

    ![Capture d’écran de la sélection de la base de données SQL AdventureWorks.](../images/dp-300-module-13-lab-04.png)

1. Sur la page **Vue d’ensemble**, copiez le **Nom du serveur** de votre Azure SQL Database comme indiqué ci-dessous. (Le nom de votre serveur doit commencer par *dp300-lab*). Vous le collerez lors des étapes ultérieures.

    ![Capture d’écran de la copie du nom du serveur.](../images/dp-300-module-13-lab-19.png)

1. Dans le portail Azure, dans la zone de recherche supérieure, tapez **automation**.

    ![Capture d’écran de la sélection des comptes Automation.](../images/dp-300-module-13-lab-11.png)

1. Sélectionnez le compte Automation que vous venez de créer.

    ![Capture d’écran de la sélection du compte Automation autoAccount.](../images/dp-300-module-13-lab-12.png)

1. Faites défiler jusqu’à la section **Automatisation des processus** du volet Compte Automation et sélectionnez **Runbooks**, puis **+ Créer un runbook**.

    ![Capture d’écran de la page Runbooks, avec sélection de Créer un runbook.](../images/dp-300-module-13-lab-20.png)

    >[!NOTE]
    > Comme nous l’avons appris, notez qu’il existe deux runbooks existants créés. Ils ont été créés automatiquement lors du déploiement du compte Automation.

1. Entrez **IndexMaintenance** comme nom du runbook et **PowerShell** comme type de runbook. Sélectionnez la dernière version du runtime disponible, puis sélectionnez **Créer**.

    ![Capture d’écran de la création d’un runbook.](../images/dp-300-module-13-lab-21.png)

1. Une fois le runbook créé, copiez et collez l’extrait de code PowerShell ci-dessous dans votre éditeur de runbook. Sur la première ligne du script, collez le nom du serveur que vous avez copié dans les étapes ci-dessus. Sélectionnez **Enregistrer**, puis **Publier**.

    **Remarque** : veuillez vérifier que le code a été copié correctement avant d’enregistrer le runbook.

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    ![Capture d’écran du collage de l’extrait de code.](../images/dp-300-module-13-lab-22.png)

1. Si tout se passe bien, vous devriez recevoir un message de réussite.

    ![Capture d’écran d’un message de création de runbook.](../images/dp-300-module-13-lab-23.png)

## Créer une planification pour un runbook

Ensuite, vous allez planifier l’exécution régulière du runbook.

1. Sous **Ressources** dans le volet de navigation gauche de votre runbook **IndexMaintenance**, sélectionnez **Planifications**. Sélectionnez **+ Ajouter une planification**.

    ![Capture d’écran de la page Planifications, avec sélection de l’option Ajouter une planification.](../images/dp-300-module-13-lab-24.png)

1. Sélectionnez **Associer une planification à votre runbook**.

    ![Capture d’écran de la sélection de Associer une planification à votre runbook.](../images/dp-300-module-13-lab-25.png)

1. Sélectionnez **+ Ajouter une planification**.

    ![Capture d’écran du lien Créer une planification.](../images/dp-300-module-13-lab-26.png)

1. Spécifiez un nom de planification descriptif et une description si vous le souhaitez.

1. Spécifiez comme heure de début **4:00** le jour suivant, dans le fuseau horaire **Heure du Pacifique**. Configurez une fréquence de répétition tous les **1** jour. Ne définissez pas d’expiration.

    ![Capture d’écran de la fenêtre contextuelle Nouvelle planification renseignée avec des exemples d’informations.](../images/dp-300-module-13-lab-27.png)

1. Sélectionnez **Créer**, puis **OK**.

1. La planification est maintenant créée et associée au runbook. Cliquez sur **OK**.

    ![Capture d’écran de la planification créée.](../images/dp-300-module-13-lab-28.png)

Azure Automation fournit un service de configuration et d’automatisation basé sur le cloud qui prend en charge la gestion cohérente dans vos environnements Azure et non-Azure.

Cet exercice vous a permis d’automatiser la défragmentation d’index sur une base de données SQL Server pour qu’elle s’exécute tous les jours à quatre heures du matin.