---
lab:
  title: "Lab\_4\_: Configurer des règles de pare-feu Azure SQL Database"
  module: Implement a Secure Environment for a Database Service
---

# Implémenter un environnement sécurisé

**Durée estimée : 30 minutes**

Les participants utiliseront les informations acquises dans les leçons pour configurer puis mettre en œuvre la sécurité dans le portail Azure et dans la base de données *AdventureWorksLT*.

Vous avez été embauché en tant qu’administrateur de base de données senior dans le but de sécuriser l’environnement de base de données. Ces tâches se concentreront sur Azure SQL Database.

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

1. Une fois le script terminé, il renvoie le nom du groupe de ressources, le nom du serveur SQL et le nom de la base de données, ainsi que le nom d’utilisateur ou d’utilisatrice et le mot de passe de l’utilisateur administrateur ou de l’utilisatrice administratrice. *Notez ces valeurs, car vous en aurez besoin plus loin dans le labo*.

---

## Configurer des règles de pare-feu Azure SQL Database

1. À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session de navigateur et accédez à [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide de vos informations d’identification Azure.

1. Dans le portail Azure, recherchez *serveurs SQL* dans le champ de recherche situé en haut de la page, puis cliquez sur **Serveurs SQL** dans la liste des options.

1. Sélectionnez le serveur SQL Server **dp300-lab-xxxxxxxx**, où *xxxxxxxx* est une chaîne numérique aléatoire.

    > &#128221; Notez que si vous utilisez votre propre serveur Azure SQL Server non créé par ce labo, sélectionnez le nom de ce serveur SQL Server.

1. Dans l’écran *Vue d’ensemble* de votre serveur SQL, à droite du nom du serveur, sélectionnez le bouton **Copier dans le presse-papiers**.

1. Sélectionnez **Afficher les paramètres de mise en réseau**.

1. Dans la page **Mise en réseau**, sous **Règles de pare-feu**, passez en revue la liste et vérifiez que votre adresse IP cliente est répertoriée. Si elle n’est pas répertoriée, sélectionnez-la sur **+ Ajouter votre adresse IPv4 cliente (votre adresse IP)**, puis sélectionnez **Enregistrer**.

    > &#128221; Notez que votre adresse IP cliente a été automatiquement entrée pour vous. L’ajout de votre adresse IP cliente à la liste vous permettra de vous connecter à votre base de données Azure SQL Database à l’aide de SQL Server Management Studio (SSMS) ou de tout autre outil client. **Prenez note de votre adresse IP client, vous l’utiliserez plus tard.**

1. Ouvrez SQL Server Management Studio. Dans la boîte de dialogue Se connecter au serveur, collez le nom de votre serveur Azure SQL Database et connectez-vous à l’aide des identifiants de connexion ci-dessous :

    - **Nom du serveur** : &lt;_collez le nom de votre serveur Azure SQL Database ici._&gt;
    - **Authentification :** authentification SQL Server
    - **Identifiant d’administration du serveur :** votre identifiant d’administration du serveur Azure SQL Database
    - **Mot de passe :** votre mot de passe d’administration du serveur Azure SQL Database

1. Sélectionnez **Connecter**.

1. Dans l’Explorateur d’objets, développez le nœud du serveur, cliquez avec le bouton droit sur **Bases de données**. Sélectionnez **Importer une application de la couche Données**.

1. Dans la boîte de dialogue **Importer une application de couche Données**, cliquez sur **Suivant** sur le premier écran.

1. Sur l’écran **Importer les paramètres**, cliquez sur **Parcourir** et accédez au dossier **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\04** et cliquez sur le fichier **AdventureWorksLT.bacpac**, puis ssur **Ouvrir**. Revenez à l’écran **Importer l’application de la couche Données**, puis sélectionnez **Suivant**.

1. Dans l’écran **Paramètres de la base de données**, apportez les modifications ci-dessous :

    - **Nom de la base de données** : AdventureWorksFromBacpac
    - **Édition de Microsoft Azure SQL Database** : De base

1. Cliquez sur **Suivant**.

1. Sur l’écran **Récapitulatif**, sélectionnez **Terminer**. Cette opération peut prendre quelques minutes. Une fois l’importation terminée, les résultats ci-dessous s’affichent. Sélectionnez **Fermer**.

1. Revenez à SQL Server Management Studio, dans l’**Explorateur d’objets**, et développez le dossier **Bases de données**. Cliquez ensuite avec le bouton droit de la souris sur la base de données **AdventureWorksFromBacpac**, puis sur **Nouvelle requête**.

1. Exécutez la requête T-SQL suivante en collant le texte dans votre fenêtre de requête.
    1. **Important :** remplacez **000.000.000.000** par votre adresse IP cliente. Sélectionnez **Exécuter**.

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.000', 
            @end_ip_address = '000.000.000.000'
    ```

1. Ensuite, vous allez créer un utilisateur contenu dans la base de données **AdventureWorksFromBacpac**. Sélectionnez **Nouvelle requête** et exécutez le code T-SQL suivant.

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    > &#128221; Cette commande crée un profil utilisateur contenu dans la base de données **AdventureWorksFromBacpac**. Nous allons tester ces informations d’identification à l’étape suivante.

1. Naviguez jusqu’à l’**Explorateur d’objets**. Cliquez sur **Connecter**, puis sur **Moteur de base de données**.

1. Essayez de vous connecter avec les informations d’identification que vous avez créées à l’étape précédente. Vous devez utiliser les informations suivantes :

    - **Identifiant de connexion** : ContainedDemo
    - **Mot de passe** : P@ssw0rd01

     Cliquez sur **Connecter**.

     Vous recevez l’erreur suivante.

    <span style="color:red">Login failed for user 'ContainedDemo'. (Microsoft SQL Server, Error: 18456)</span>

    > &#128221; Cette erreur se produit parce que la connexion a tenté de se connecter à la base de données *principale* et non à **AdventureWorksFromBacpac **où le profil utilisateur a été créé. Modifiez le contexte de connexion en sélectionnant **OK** pour quitter le message d’erreur, puis en sélectionnant **Options >>** dans la boîte de dialogue **Se connecter au serveur**.

1. Dans l’onglet **Propriétés de la connexion**, saisissez le nom de la base de données **AdventureWorksFromBacpac**, puis sélectionnez **Se connecter**.

1. Remarquez que vous avez réussi à vous authentifier en utilisant l’utilisateur **ContainedDemo**. Cette fois, vous êtes directement connecté à **AdventureWorksFromBacpac**, qui est la seule base de données à laquelle le nouvel utilisateur a accès.

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

Dans cet exercice, vous avez configuré des règles de pare-feu pour le serveur et la base de données afin d’accéder à une base de données hébergée sur Azure SQL Database. Vous avez également utilisé des instructions T-SQL pour créer un utilisateur contenu et utilisé SQL Server Management Studio pour vérifier l’accès.
