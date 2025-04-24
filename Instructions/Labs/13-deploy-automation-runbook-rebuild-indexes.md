---
lab:
  title: "Labo\_13\_: Déployer un runbook Automation pour reconstruire automatiquement les index"
  module: Automate database tasks for Azure SQL
---

# Déployer un runbook Automation pour reconstruire automatiquement les index

**Durée estimée : 30 minutes**

Vous avez été embauché en tant qu’administrateur de bases de données senior pour aider à automatiser les opérations quotidiennes d’administration de base de données. Cette automatisation a pour objectif de s’assurer que les bases de données pour AdventureWorks continuent de fonctionner à des performances optimales, ainsi que de fournir des méthodes pour générer des alertes en fonction de certains critères. AdventureWorks utilise SQL Server à la fois comme infrastructure en tant que service (IaaS) et plateforme en tant que service (PaaS).

> &#128221; Ces exercices peuvent vous demander de copier et coller du code T-SQL et d’utiliser des ressources SQL existantes. Vérifiez que le code a été copié correctement, avant de l’exécuter.

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

1. Une fois le script terminé, il renvoie le nom du groupe de ressources, le nom du serveur SQL et le nom de la base de données, ainsi que le nom d’utilisateur ou d’utilisatrice et le mot de passe de l’utilisateur administrateur ou de l’utilisatrice administratrice. *Notez ces valeurs, car vous en aurez besoin plus loin dans le labo*.

---

## Créer un compte Automation

1. À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session de navigateur et accédez à [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide de vos informations d’identification Azure.

1. Dans le Portail Azure, dans la barre de recherche, tapez *Automation*, puis sélectionnez **Comptes Automation** dans les résultats de la recherche, puis cliquez sur **+ Créer**.

1. Sur la page **Créer un compte Automation**, saisissez les informations ci-dessous, puis sélectionnez **Vérifier + Créer**.

    - **Groupe de ressources** : &lt;Votre groupe de ressources.&gt;
    - **Nom du compte Automation :** autoAccount
    - **Région :** utilisez la région par défaut.

1. Dans la page Vérifier, sélectionnez **Créer**.

    > &#128221; La création de votre compte Automation peut prendre quelques minutes.

## Se connecter à une base de données Azure SQL

1. Dans le Portail Azure, accédez à votre base de données en recherchant les **bases de données SQL**.

1. Sélectionnez la base de données SQL **AdventureWorksLT**.

1. Dans la section principale de la page de votre base de données SQL, sélectionnez **Éditeur de requêtes (préversion)**.

1. Vous recevrez une invitation à entrer des informations d’identification pour vous connecter à votre base de données à l’aide du compte d’administration de base de données, puis sélectionnez **OK**.

    Cela ouvrira un nouvel onglet dans le navigateur. Sélectionnez **Ajouter une adresse IP cliente**, puis **Enregistrer**. Une fois enregistrée, revenez à l’onglet précédent et sélectionnez à nouveau **OK**.

    > &#128221; Vous pouvez recevoir le message d’erreur *Impossible d’ouvrir le serveur « your-sql-server-name » demandé par la connexion. Le client avec l’adresse IP « xxx.xxx.xxx.xxx » n’est pas autorisé à accéder au serveur.* Si c’est le cas, vous devez ajouter votre adresse IP publique actuelle aux règles de pare-feu du serveur SQL.

    Si vous devez configurer les règles de pare-feu, procédez comme suit :

    1. Dans la page **Vue d’ensemble** de la base de données SQL en haut de la barre de menus, sélectionnez **Définir le pare-feu du serveur**.
    1. Sélectionnez **+ Ajouter votre adresse IPv4 actuelle (xxx.xxx.xxx.xxx)** et **Enregistrer**.
    1. Une fois enregistrée, revenez à la page de base de données **AdventureWorksLT** et sélectionnez à nouveau l’**Éditeur de requête (préversion)**.
    1. Vous recevrez une invitation à entrer des informations d’identification pour vous connecter à votre base de données à l’aide du compte d’administration de base de données, puis sélectionnez **OK**.

1. Dans l’**Éditeur de requête (préversion)**, sélectionnez **Ouvrir une requête**.

1. Sélectionnez l’icône parcourir le *dossier* et accédez au dossier **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Module13**. Sélectionnez le fichier **usp_AdaptiveIndexDefrag.sql**, puis sélectionnez **Ouvrir**, puis **OK**.

1. Supprimez **USE msdb** et **GO** sur les lignes 5 et 6 de la requête, puis sélectionnez **Exécuter**.

1. Développez le dossier **Procédures stockées** pour afficher les procédures stockées nouvellement créées.

## Configurer les ressources du compte Automation

Les étapes suivantes consistent à configurer les ressources nécessaires pour préparer la création du runbook. Sélectionnez ensuite **Comptes Automation**.

1. Dans le portail Azure, dans la zone de recherche supérieure, tapez **automation**, puis sélectionnez **Comptes Automation**.

1. Sélectionnez le compte Automation **autoAccount** que vous venez de créer.

1. Sélectionnez **Modules** dans la section **Ressources partagées** du volet Automation. Sélectionnez ensuite **Parcourir la galerie**.

1. Recherchez **SqlServer** dans la Galerie.

1. Sélectionnez **SqlServer** afin d’accéder à l’écran suivant, puis sélectionnez le bouton **Sélectionner**.

1. Sur la page **Ajouter un module**, sélectionnez la dernière version du runtime disponible, puis sélectionnez **Importer**. Le module PowerShell est alors importé dans votre compte Automation.

1. Vous devez créer des informations d’identification pour vous connecter de manière sécurisée à votre base de données. Dans le volet du *compte Automation*, accédez à la section **Ressources partagées** et sélectionnez **Informations d’identification**.

1. Sélectionnez **+ Ajouter des informations d’identification**, entrez les informations ci-dessous, puis sélectionnez **Créer**.

    - Nom : **SQLUser**
    - Nom d’utilisateur : **sqladmin**
    - Mot de passe : &lt;entrez un mot de passe fort, de 12 caractères et contenant au moins 1 lettre majuscule, 1 lettre minuscule, 1 chiffre et 1 caractère spécial.&gt;
    - Confirmez le mot de passe : &lt;entrez à nouveau le mot de passe que vous avez entré précédemment.&gt;

## Créer un runbook PowerShell

1. Dans le Portail Azure, accédez à votre base de données en recherchant les **bases de données SQL**.

1. Sélectionnez la base de données SQL **AdventureWorksLT**.

1. Sur la page **Vue d’ensemble**, copiez le **Nom du serveur** de votre Azure SQL Database comme indiqué ci-dessous (le nom de votre serveur doit commencer par *dp300-lab*). Vous le collerez lors des étapes ultérieures.

1. Dans le portail Azure, dans la zone de recherche supérieure, tapez **automation**, puis sélectionnez **Comptes Automation**.

1. Sélectionnez le compte Automation **autoAccount**.

1. Faites défiler jusqu’à la section **Automatisation de processus** du volet Compte Automation et sélectionnez **Runbooks**.

1. Sélectionnez **+ Créer un runbook**.

    > &#128221; Comme nous l’avons appris, notez qu’il existe deux runbooks existants créés. Ils ont été créés automatiquement lors du déploiement du compte Automation.

1. Entrez **IndexMaintenance** comme nom du runbook et **PowerShell** comme type de runbook. Sélectionnez la dernière version du runtime disponible, puis sélectionnez **Examiner et créer**.

1. Dans la page **Créer des runbooks**, sélectionnez **Créer**.

1. Une fois le runbook créé, copiez et collez l’extrait de code PowerShell ci-dessous dans votre éditeur de runbook. 

    > &#128221; Veuillez vérifier que le code a été copié correctement avant d’enregistrer le runbook.

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    > &#128221; Notez que le code ci-dessus est un script PowerShell qui exécute la procédure stockée **usp_AdaptiveIndexDefrag** sur la base de données **AdventureWorksLT**. Le script utilise le cmdlet **Invoke-Sqlcmd** pour se connecter au serveur SQL Server et exécuter la procédure stockée. Le cmdlet **Get-AutomationPSCredential** est utilisé pour récupérer les informations d’identification stockées dans le compte Automation.

1. Sur la première ligne du script, collez le nom du serveur que vous avez copié dans les étapes précédentes.

1. Sélectionnez **Enregistrer**, puis **Publier**.

1. Sélectionnez **Oui** pour confirmer l’action de publication.

1. Le runbook *IndexMaintenance* est maintenant publié.

## Créer une planification pour un runbook

Ensuite, vous allez planifier l’exécution régulière du runbook.

1. Sous **Ressources** dans le volet de navigation gauche de votre runbook **IndexMaintenance**, sélectionnez **Planifications**. 

1. Sélectionnez **+ Ajouter une planification**.

1. Sélectionnez **Associer une planification à votre runbook**.

1. Sélectionnez **+ Ajouter une planification**.

1. Entrez les informations ci-dessous, puis sélectionnez **Créer**.

    - **Nom :** DailyIndexDefrag
    - **Description :** défragmentation d’index quotidienne pour la base de données AdventureWorksLT.
    - **Démarrage :** 4h00 (jour suivant)
    - **Fuseau horaire :**&lt;sélectionner le fuseau horaire correspondant à votre emplacement&gt;
    - **Périodicité :** périodique
    - **Fréquence de périodicité :** 1 jour
    - **Définir une expiration :** Non

    > &#128221; Notez que l’heure de début est définie sur 4h00 le lendemain. Le fuseau horaire est défini sur le vôtre. La périodicité est définie tous les jours. N’expire jamais.

1. Sélectionnez **Créer**, puis **OK**.

1. La planification est maintenant créée et associée au runbook. Cliquez sur **OK**.

Azure Automation fournit un service de configuration et d’automatisation basé sur le cloud qui prend en charge la gestion cohérente dans vos environnements Azure et non-Azure.

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

Cet exercice vous a permis d’automatiser la défragmentation d’index sur une base de données SQL Server pour qu’elle s’exécute tous les jours à quatre heures du matin.
