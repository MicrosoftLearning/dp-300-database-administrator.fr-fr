---
lab:
  title: "Labo\_6\_: Isoler les problèmes de performances via la surveillance"
  module: Monitor and optimize operational resources in Azure SQL
---

# Isoler les problèmes de performances via la surveillance

**Durée estimée : 30 minutes**

Les participants utiliseront les informations acquises dans les leçons pour définir les produits livrables d’un projet de transformation numérique au sein d’AdventureWorksLT. En examinant le Portail Azure ainsi que d’autres outils, les participants détermineront comment utiliser les outils pour identifier et résoudre les problèmes liés aux performances.

Vous avez été embauché en tant qu’administrateur de base de données pour identifier les problèmes liés aux performances et fournir des solutions viables aux problèmes détectés. Vous devez utiliser le portail Azure afin d’identifier les problèmes de performances et de suggérer des méthodes pour les résoudre.

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

## Examiner l’utilisation du processeur dans le portail Azure

1. À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session de navigateur et accédez à [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide de vos informations d’identification Azure.

1. Dans le portail Azure, recherchez *serveurs SQL* dans le champ de recherche situé en haut de la page, puis cliquez sur **Serveurs SQL** dans la liste des options.

1. Sélectionnez le serveur SQL Server **dp300-lab-xxxxxxxx**, où *xxxxxxxx* est une chaîne numérique aléatoire.

    > &#128221; Notez que si vous utilisez votre propre serveur Azure SQL Server non créé par ce labo, sélectionnez le nom de ce serveur SQL Server.

1. Dans la page principale Azure SQL Server, sous **Sécurité**, sélectionnez **Mise en réseau**.

1. Dans la page **Mise en réseau**, vérifiez si votre adresse IP publique actuelle est déjà ajoutée à la liste **Règles de pare-feu**, sinon, sélectionnez **+ Ajouter votre adresse IPv4 cliente (votre adresse IP)** pour l’ajouter, puis sélectionnez **Enregistrer**.

1. Depuis le panneau principal de votre serveur Azure SQL Server, accédez à la section **Paramètres** et sélectionnez **Bases de données SQL**, puis sélectionnez la base de données **AdventureWorksLT**.

1. Dans le volet de navigation gauche, sélectionnez **Éditeur de requête (préversion)**.

    **Remarque** : cette fonctionnalité est en préversion.

1. Sélectionnez le nom d’utilisateur ou d’utilisatrice de l’administrateur ou de l’administratrice SQL Server et entrez le mot de passe ou vos informations d’identification Microsoft Entra s’ils sont affectés pour se connecter à la base de données.
    - **Nom du serveur** : &lt;_collez le nom de votre serveur Azure SQL Database ici._&gt;
    - **Authentification :** authentification SQL Server
    - **Identifiant d’administration du serveur :** votre identifiant d’administration du serveur Azure SQL Database
    - **Mot de passe :** votre mot de passe d’administration du serveur Azure SQL Database

1. Dans **Requête 1**, entrez la requête suivante, puis sélectionnez **Exécuter** :

    ```sql
    DECLARE @Counter INT 
    SET @Counter=1
    WHILE ( @Counter <= 10000)
    BEGIN
        SELECT 
             RTRIM(a.Firstname) + ' ' + RTRIM(a.LastName)
            , b.AddressLine1
            , b.AddressLine2
            , RTRIM(b.City) + ', ' + RTRIM(b.StateProvince) + '  ' + RTRIM(b.PostalCode)
            , CountryRegion
            FROM SalesLT.Customer a
            INNER JOIN SalesLT.CustomerAddress c 
                ON a.CustomerID = c.CustomerID
            RIGHT OUTER JOIN SalesLT.Address b
                ON b.AddressID = c.AddressID
        ORDER BY a.LastName ASC
        SET @Counter  = @Counter  + 1
    END
    ```

1. Attendez la fin de la requête.

1. Réexécutez la requête *deux* fois de plus pour générer une charge processeur sur la base de données.

1. Sur le panneau de la base de données **AdventureWorksLT**, sélectionnez l’icône **Mesures** dans la section **Surveillance**.

    Si le message *Vos modifications non enregistrées seront ignorées* s’affiche, sélectionnez **OK**.

1. Modifiez l’option de menu **Mesure** pour refléter le **Pourcentage du processeur**, puis sélectionnez une **Agrégation** de **Moyenne**. Vous obtiendrez ainsi le pourcentage moyen du processeur pour la période donnée.

1. Observez la moyenne du processeur au fil du temps. Vous devez noter un pic d’utilisation du processeur à la fin du graphique lorsque la requête était en cours d’exécution.

## Identifier les requêtes sollicitant beaucoup le processeur

1. Recherchez l’icône **Query Performance Insight** dans la section **Performances intelligentes** du volet de la base de données **AdventureWorksLT**.

1. Sélectionnez **Réinitialiser les paramètres**.

1. Sélectionnez la requête dans la grille qui est située sous le graphique. Si vous ne voyez pas la requête que nous avons exécutée précédemment plusieurs fois, attendez jusqu’à 2 à 5 minutes, puis sélectionnez **Actualiser**.

    > &#128221; S’il existe plusieurs requêtes répertoriées, sélectionnez chacune d’elles pour observer les résultats. Notez la quantité importante d’informations disponibles pour chaque requête.

1. Pour la requête que vous avez exécutée précédemment, notez que la durée totale était de plus d’une minute et qu’elle s’est exécutée environ 30 000 fois.

1. En examinant le texte SQL de la page **Détails de la requête** par rapport à la requête que vous avez exécutée, vous remarquerez que les **Détails de la requête** incluent uniquement l’instruction **SELECT** et non la boucle **WHILE** ou une autre instruction. Cela se produit parce que **Query Performance Insight** s’appuie sur les données du **Magasin des requêtes**, qui effectue uniquement le suivi des instructions DML (langage de manipulation de données), telles que **SELECT, INSERT, UPDATE, DELETE, MERGE** et **BULK INSERT** tout en ignorant les instructions DDL (langage de définition de données).

Tous les problèmes de performances ne sont pas liés à une utilisation élevée du processeur par une seule exécution de requête. Dans ce cas, la requête a été exécutée des milliers de fois, ce qui peut également entraîner une utilisation élevée du processeur.

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

Dans cet exercice, vous avez appris à explorer les ressources du serveur pour une base de données Azure SQL et à identifier les problèmes potentiels de performances des requêtes grâce à Query Performance Insight.
