---
lab:
  title: "Labo\_14\_: Configurer la géoréplication pour Azure SQL Database"
  module: Plan and implement a high availability and disaster recovery solution
---

# Configurer la géoréplication pour Azure SQL Database

**Durée estimée : 30 minutes**

En tant qu’administrateur de base de données au sein d’AdventureWorks, vous devez activer la géoréplication pour Azure SQL Database, et vous assurer qu’elle fonctionne correctement. De plus, vous le basculerez manuellement vers une autre région à l’aide du portail.

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

## Activer la géoréplication

1.  À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session de navigateur et accédez à [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide de vos informations d’identification Azure.

1. Dans le Portail Azure, accédez à votre base de données en recherchant les **bases de données SQL**.

1. Sélectionnez la base de données SQL **AdventureWorksLT**.

1. Dans le panneau de la base de données, dans la section **Gestion des données**, sélectionnez **Réplicas**.

1. Sélectionnez **+ Créer un réplica**.

1. Dans la page **Créer une base de données SQL - Géo-réplica**, notez que les sections **Détails du projet** et **Base de données primaire** sont déjà renseignés avec l’abonnement, le groupe de ressources et le nom de la base de données.

1. Pour la section **Configuration du réplica**, sélectionnez **Géo-réplica** pour le *Type de réplica*.

1. Dans **Détails de la base de données géosecondaire**, saisissez les valeurs suivantes :

    - **Abonnement** : &lt;nom de votre abonnement&gt; (identique à la base de données primaire).
    - **Groupe de ressources** : &lt;sélectionnez le même groupe de ressources que la base de données primaire.&gt; 
    - **Nom de la base de données** : le nom de la base de données est grisé et sera identique au nom de la base de données primaire.
    - **Serveur :** sélectionnez **Créer nouveau**.
    - Dans la page **Créer une base de données SQL**, entrez les valeurs suivantes :

        - **Nom du serveur** : tapez un nom unique pour le serveur secondaire. Le nom doit être unique pour tous les serveurs Azure SQL Database.
        - **Emplacement** : sélectionnez une autre région dans la base de données primaire. Notez que votre abonnement n’a peut-être pas toutes les régions disponibles.
        - Cochez la case **Autoriser les services Azure à accéder au serveur**. Notez que dans un environnement de production, vous pouvez restreindre l’accès au serveur.
        - Pour l’authentification, sélectionnez **Authentification SQL**. Notez que dans un environnement de production, vous pouvez utiliser l’authentification **Microsoft Entra uniquement**. Entrez **sqladmin* pour le nom de connexion d’administration et un mot de passe sécurisé. Le mot de passe doit répondre aux exigences de complexité du mot de passe Azure SQL, au moins 12 caractères et contenir au moins 1 lettre majuscule, 1 lettre minuscule, 1 chiffre et 1 caractère spécial.
        - Sélectionnez **OK** pour créer le serveur.

    - **Vous souhaitez utiliser un pool élastique ?**  : non.
    - **Calcul + stockage** : usage général, Gen 5, 2 vCores, stockage de 32 Go.
    - **Redondance du stockage de sauvegarde** : stockage localement redondant (LRS). Notez que dans un environnement de production, vous pouvez utiliser **Stockage géoredondant (GRS)**.

1. Sélectionnez **Vérifier + créer**.

1. Sélectionnez **Créer**. La création du serveur secondaire et de la base de données prend quelques minutes. Une fois l’opération terminée, la progression passe de **Déploiement en cours** à **Votre déploiement est terminé**.

1. Sélectionnez **Accéder à la ressource** pour accéder à la base de données du serveur secondaire pour l’étape suivante.

## Basculement d’une base de données SQL vers une région secondaire

Une fois le réplica Azure SQL Database créé, vous allez effectuer un basculement.

1. Si ce n’est pas déjà le cas sur la base de données du serveur secondaire, recherchez des **bases de données SQL** dans le portail Azure et sélectionnez la base de données SQL **AdventureWorksLT** sur le serveur secondaire.

1. Dans le panneau principal de la base de données SQL, dans la section **Gestion des données**, sélectionnez **Réplicas**.

1. Notez que le lien de géoréplication est maintenant établi. La valeur *État du réplica* de la base de données primaire est **En ligne** et la valeur *État du réplica* des réplicas géographiques est **Lisible**.

1. Sélectionnez le menu **…** du serveur réplica géographique secondaire, puis sélectionnez **Basculement forcé**.

    > &#128221 Le basculement forcé fait passer la base de données secondaire au rôle principal. Toutes les sessions sont déconnectées pendant cette opération.

1. Lorsque le message d’avertissement vous y invite, cliquez sur **Oui**.

1. L’état du réplica principal passe à **En attente** et celui du réplica secondaire à **Basculement**. 

     > &#128221; Notez que, étant donné que la base de données est petite, le basculement sera rapide. Dans un environnement de production, ce processus peut prendre quelques minutes.

1. Lorsqu’il est terminé, les rôles basculent avec le réplica secondaire qui devient le nouveau principal, et l’ancien principal qui se retrouve le secondaire. Il se peut que vous deviez actualiser la page pour afficher le nouveau statut.

Nous avons vu que la base de données secondaire accessible en lecture peut être dans la même région Azure que la base de données primaire, ou, plus communément, dans une autre région. Ce type de base de données secondaire accessible en lecture est également appelé géo-secondaires ou géo-réplicas.

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

Vous avez maintenant vu comment activer les géo-réplicas pour Azure SQL Database et la basculer manuellement vers une autre région à l’aide du portail.
