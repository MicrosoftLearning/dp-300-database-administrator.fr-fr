---
lab:
  title: "Labo\_10\_: Isoler les zones à problème des requêtes peu performantes dans SQL Database"
  module: Optimize query performance in Azure SQL
---

# Isoler les zones à problème des requêtes peu performantes dans SQL Database

**Durée estimée : 30 minutes**

Vous avez été embauché en tant qu’administrateur de base de données senior pour résoudre les problèmes de performances qui ont lieu lorsque les utilisateurs interrogent la base de données *AdventureWorks2017*. Votre tâche consiste à identifier les problèmes de performances des requêtes et à les résoudre à l’aide des techniques apprises dans ce module.

Vous allez exécuter des requêtes qui ont des performances non optimales, examiner les plans de requête et tenter d’apporter des améliorations dans la base de données.

> &#128221; Ces exercices vous demandent de copier et coller du code T-SQL. Vérifiez que le code a été copié correctement, avant de l’exécuter.

## Configurez l’environnement.

Si votre machine virtuelle de labo a été fournie et préconfigurée, vous devez trouver les fichiers de labo prêts dans le dossier **C :\LabFiles**. *Prenez un moment pour vérifier et si les fichiers sont déjà là, ignorez cette section*. Toutefois, si vous utilisez votre propre ordinateur ou que les fichiers de labo sont manquants, vous devez les cloner à partir de *GitHub* pour continuer.

1. À partir de la machine virtuelle du labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session Visual Studio Code.

1. Ouvrez la palette de commandes (Ctrl+Maj+P), puis tapez **Git: Clone**. Sélectionnez l’option **Git: Clone**.

1. Collez l’URL suivante dans le champ **URL du référentiel**, puis sélectionnez **Entrée**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Enregistrez le référentiel dans le dossier **C :\LabFiles** sur la machine virtuelle de labo ou votre ordinateur local si elle n’a pas été fournie (créez le dossier s’il n’existe pas).

---

## Restaurer une base de données

Si vous avez déjà restauré la base de données **AdventureWorks2017**, vous pouvez ignorer cette section.

1. À partir de la machine virtuelle du labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session SQL Server Management Studio (SSMS).

1. Lorsque SSMS s’ouvre, par défaut, la boîte de dialogue **Se connecter au serveur** s’affiche. Choisissez l’instance par défaut, puis sélectionnez **Se connecter**. Vous devrez peut-être cocher la case **Faire confiance au certificat de serveur**.

    > &#128221; Notez que si vous utilisez votre propre instance SQL Server, vous devez vous y connecter à l’aide du nom et des informations d’identification appropriés de l’instance de serveur.

1. Sélectionnez le dossier **Bases de données**, puis **Nouvelle requête**.

1. Dans la fenêtre de la nouvelle requête, copiez et collez le code T-SQL ci-dessous. Exécutez la requête pour restaurer la base de données.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; Vous devez disposer d’un dossier nommé **C:\LabFiles**. Si vous n’avez pas ce dossier, créez-le ou spécifiez un autre emplacement pour la base de données et les fichiers de sauvegarde.

1. Sous l’onglet **Messages**, vous devez voir un message indiquant que la base de données a été restaurée avec succès.

## Générer un vrai plan d’exécution

Il existe plusieurs façons de générer un plan d’exécution dans SQL Server Management Studio.

1. Sélectionnez **Nouvelle requête**. Copiez et collez le code T-SQL suivant dans la fenêtre de l’éditeur. Sélectionnez **Exécuter** pour exécuter cette requête.

    **Remarque** : utilisez **SHOWPLAN_ALL** pour afficher une version texte du plan d’exécution d’une requête dans le volet de résultats, plutôt que sous forme graphique dans un onglet séparé.

    ```sql
    USE AdventureWorks2017;

    GO

    SET SHOWPLAN_ALL ON;

    GO

    SELECT BusinessEntityID
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';

    GO

    SET SHOWPLAN_ALL OFF;

    GO
    ```

    Dans le volet des résultats, vous verrez une version texte du plan d’exécution, au lieu des résultats réels de la requête pour l’instruction **SELECT**.

1. Prenez un moment pour examiner le texte de la deuxième ligne de la colonne **StmtText** :

    ```console
    |--Index Seek(OBJECT:([AdventureWorks2017].[HumanResources].[Employee].[AK_Employee_NationalIDNumber]), SEEK:([AdventureWorks2017].[HumanResources].[Employee].[NationalIDNumber]=CONVERT_IMPLICIT(nvarchar(4000),[@1],0)) ORDERED FORWARD)
    ```

    Le texte ci-dessus explique que le plan d’exécution utilise une **Recherche d’index** dans la clé **AK_Employee_NationalIDNumber**. Il montre également que le plan d’exécution a eu besoin d’effectuer une étape **CONVERT_IMPLICIT**.

    L’optimiseur de requête a pu localiser un index approprié pour extraire les enregistrements requis.

## Résoudre un plan de requête non optimal

1. Copiez-collez le code ci-dessous dans une nouvelle fenêtre de requête.

    Sélectionnez l’icône **Inclure le plan d’exécution réel** à droite du bouton Exécuter, ou appuyez sur <kbd>Ctrl</kbd>+<kbd>M</kbd>. Exécutez la requête en sélectionnant **Exécuter** ou appuyez sur <kbd>F5</kbd>. Prenez note du plan d’exécution et des lectures logiques sous l’onglet Messages.

    ```sql
    SET STATISTICS IO, TIME ON;

    SELECT [SalesOrderID] ,[CarrierTrackingNumber] ,[OrderQty] ,[ProductID], [UnitPrice] ,[ModifiedDate]
    FROM [AdventureWorks2017].[Sales].[SalesOrderDetail]
    WHERE [ModifiedDate] > '2012/01/01' AND [ProductID] = 772;
    ```

    Lorsque vous examinerez le plan d’exécution, vous remarquerez une **recherche de clé**. Si vous placez le curseur de la souris sur l’icône, vous verrez que les propriétés indiquent que la recherche est effectuée pour chaque ligne extraite par la requête. Vous pouvez voir que le plan d’exécution effectue une opération de **recherche de clé**.

    Remarquez les colonnes de la section **Liste de sortie**. Comment amélioreriez-vous cette requête ?

    Pour identifier l’index qui doit être modifié afin de supprimer la recherche de clé, vous devez examiner la recherche d’index située au-dessus de celle-ci. Placez le curseur de la souris sur l’opérateur de recherche d’index pour afficher les propriétés de l’opérateur.

1. Vous pouvez supprimer les **recherches de clés** en ajoutant un index de couverture qui inclut tous les champs retournés ou recherchés dans la requête. Dans cet exemple, l’index utilise uniquement la colonne **ProductID**. Voici la définition actuelle de l’index, notez que la colonne **ProductID** est la seule colonne clé qui force une **Recherche de clé** à récupérer les autres colonnes requises par la requête.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID] ON [Sales].[SalesOrderDetail]
    ([ProductID] ASC)
    ```

    Si nous ajoutons les champs **Liste de résultat** à l’index en tant que colonnes incluses, la **recherche de clé** sera supprimée. Étant donné que l’index existe déjà, vous devez soit supprimer l’index (DROP) puis le recréer, soit définir **DROP_EXISTING=ON** de manière à ajouter les colonnes. Notez que la colonne **ProductID** fait déjà partie de l’index et qu’il n’est pas nécessaire de l’ajouter en tant que colonne incluse. Nous pouvons également améliorer les performances de l’index en ajoutant **ModifiedDate**. Ouvrez une fenêtre **Nouvelle requête** et exécutez le script suivant pour supprimer et recréer l’index.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID]
    ON [Sales].[SalesOrderDetail] ([ProductID],[ModifiedDate])
    INCLUDE ([CarrierTrackingNumber],[OrderQty],[UnitPrice])
    WITH (DROP_EXISTING = on);
    GO
    ```

1. Réexécutez la requête de l’étape 1. Notez les modifications apportées aux lectures logiques et au plan d’exécution. Le plan a seulement besoin d’utiliser l’index non-cluster que nous avons créé.

> &#128221; En examinant le plan d’exécution, vous remarquerez que la **recherche de clé** est maintenant terminée et que nous utilisons uniquement l’index non cluster.

## Utiliser le Magasin des requêtes pour détecter et gérer la régression

Maintenant, vous allez exécuter une charge de travail afin de générer des statistiques de requête pour le magasin des requêtes, examiner le rapport **Principales requêtes consommatrices de ressources** afin de déterminer l’origine des mauvaises performances, et voir comment forcer un meilleur plan d’exécution.

1. Sélectionnez **Nouvelle requête**. Copiez et collez le code T-SQL suivant dans la fenêtre de l’éditeur. Sélectionnez **Exécuter** pour exécuter cette requête.

    Ce script active la fonctionnalité du magasin des requêtes dans la base de données AdventureWorks2017 et configure la base de données avec un niveau de compatibilité de 100.

    ```sql
    USE [master];

    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE = ON;

    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE (OPERATION_MODE = READ_WRITE);

    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 100;

    GO
    ```

    La modification du niveau de compatibilité est un retour en arrière pour la base de données. En effet, cette opération restreint les fonctionnalités que SQL Server peut utiliser, car seules les fonctionnalités de SQL Server 2008 sont disponibles.

1. Sélectionnez **Fichier** > **Ouvrir** > **Fichier** dans SQL Server Management Studio.

1. Accédez au fichier **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\10\CreateRandomWorkloadGenerator.sql**.

1. Une fois ouvert dans SQL Server Management Studio, sélectionnez **Exécuter** pour exécuter la requête.

1. Dans un nouvel éditeur de requête, ouvrez le fichier **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\10\ExecuteRandomWorkload.sql**, puis sélectionnez **Exécuter** pour exécuter la requête.

1. Une fois l’exécution terminée, exécutez le script une deuxième fois pour créer une charge supplémentaire sur le serveur. Laissez l’onglet Requête ouvert pour cette requête.

1. Copiez-collez le code ci-dessous dans une nouvelle fenêtre de requête, puis exécutez-le en sélectionnant **Exécuter**.

    Ce script remplace le mode de compatibilité de la base de données par SQL Server 2022 (**160**). Toutes les fonctionnalités et améliorations apportées depuis SQL Server 2008 seront désormais disponibles pour la base de données.

    ```sql
    USE [master];

    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 160;

    GO
    ```

1. Retournez à l’onglet de la requête du fichier **ExecuteRandomWorkload.sql** et exécutez-la à nouveau.

## Examiner le rapport Principales requêtes consommatrices de ressources

1. Pour afficher le nœud Magasin des requêtes, vous devez actualiser la base de données AdventureWorks2017 dans SQL Server Management Studio. Cliquez avec le bouton droit sur le nom de la base de données, puis sélectionnez **Actualiser**. Le nœud Magasin des requêtes s’affiche sous la base de données.

1. Développez le nœud **Magasin des requêtes** pour afficher tous les rapports disponibles. Sélectionnez le rapport **Principales requêtes consommatrices de ressources**.

1. Lorsque le rapport s’ouvre, sélectionnez la liste déroulante du menu, puis sélectionnez **Configurer** dans le coin supérieur droit du rapport.

1. Dans l’écran de configuration, remplacez le filtre pour réduire le nombre minimal de plans de requête à 2 plans. Sélectionnez ensuite **OK**.

1. Choisissez la requête dont la durée est la plus longue en sélectionnant la barre située la plus à gauche du graphique à barres, dans la partie supérieure gauche du rapport.

    Cela vous montrera le résumé de la requête et du plan pour votre requête de longue durée dans le Magasin des requêtes. Consultez le graphique *Résumé de plan* dans le coin supérieur droit du rapport et le *Plan de requête* en bas du rapport.

## Forcer un meilleur plan d’exécution

1. Accédez à la section Résumé du plan du rapport, comme indiqué ci-dessous. Vous verrez qu’il y a deux plans d’exécution avec un grand écart de durée.

1. Sélectionnez l’ID de plan dont la durée est la plus courte (cela est indiqué par une position basse sur l’axe des ordonnées du graphique) dans la fenêtre située en haut à droite du rapport. Sélectionnez l’ID de plan en regard du graphique Résumé de plan.

1. Sélectionnez **Forcer le plan** sous le graphique de synthèse. Une fenêtre de confirmation s’affiche, sélectionnez **Oui**.

    Une fois le plan forcé, vous verrez que le **plan forcé** est désormais grisé et que le plan dans la fenêtre de résumé du plan contient maintenant une coche indiquant qu’il est forcé.

    Il peut arriver que l’optimiseur de requête ne fasse pas un bon choix quant au plan d’exécution à utiliser. Lorsque cela se produit, vous pouvez forcer SQL Server à utiliser le plan de votre choix si vous savez que celui-ci fonctionne mieux.

## Utiliser les indicateurs de requête pour avoir un impact sur les performances

Exécutez ensuite une charge de travail, modifiez la requête pour utiliser un paramètre, appliquez un indice de requête à la requête et exécutez-la à nouveau.

Avant de poursuivre l’exercice, fermez toutes les fenêtres de requête ouvertes en sélectionnant le menu **Fenêtre**, puis sélectionnez **Fermer tous les documents**. Dans le menu contextuel, sélectionnez **Non**.

1. Sélectionnez **Nouvelle requête**, puis sélectionnez l’icône **Inclure le plan d’exécution réel** avant d’exécuter la requête, ou utilisez <kbd>CTRL</kbd>+<kbd>M</kbd>.

1. Exécutez la requête ci-dessous. Notez que le plan d’exécution affiche un opérateur de recherche d’index.

    ```sql
    USE AdventureWorks2017;

    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=288;
    ```

1. Dans une nouvelle fenêtre de requête, exécutez la requête suivante. Comparez les deux plans d’exécution.

    ```sql
    USE AdventureWorks2017;
    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=277;
    ```

    La seule différence cette fois-ci est que la valeur SalesPersonID est définie sur 277. Notez l’opération d’analyse d’index cluster dans le plan d’exécution.

Comme nous pouvons le voir, selon les statistiques d’index, l’optimiseur de requête a choisi un plan d’exécution différent en raison des différentes valeurs de la clause **WHERE**.

Pourquoi avons-nous des plans différents si nous avons uniquement modifié la valeur *SalesPersonID* ?

Cette requête utilise une constante dans sa clause **WHERE**, l’optimiseur considère chacune de ces requêtes comme étant uniques et génère à chaque fois un plan d’exécution différent.

## Modifier la requête pour utiliser une variable et utiliser un indicateur de requête

1. Modifiez la requête afin d’utiliser une valeur de variable pour SalesPersonID.

1. Utilisez l’instruction T-SQL **DECLARE** pour déclarer <strong>@SalesPersonID</strong>. Cela vous permet de passer une valeur au lieu de coder en dur cette valeur dans la clause **WHERE**. Vous devez vérifier que le type de données de votre variable correspond au type de données de la colonne dans la table cible afin d’éviter une conversion implicite. Exécutez la requête avec le plan de requête réel activé.

    ```sql
    USE AdventureWorks2017;

    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID;
    ```

    Si vous examinez le plan d’exécution, vous verrez qu’il utilise une analyse d’index pour obtenir les résultats. L’optimiseur de requête n’a pas pu effectuer de bonnes optimisations, car il ne peut pas connaître la valeur de la variable locale jusqu’au runtime.

1. Vous pouvez aider l’optimiseur de requête à améliorer vos choix en fournissant un indicateur de requête. Réexécutez la requête ci-dessus avec **OPTION (RECOMPILE)**  :

    ```sql
    USE AdventureWorks2017

    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID
    OPTION (RECOMPILE);
    ```

    Notez que l’optimiseur de requête a pu choisir un plan d’exécution plus efficace. L’option **RECOMPILE** force le compilateur de requête à remplacer la variable par sa valeur.

    En comparant les statistiques, vous pouvez voir que le nombre de lectures logiques est **68 %** plus élevé (689 contre 409) pour la requête qui ne contient pas d’indicateur de requête.

---

## Nettoyage

Si vous n’utilisez pas la base de données ou les fichiers de labo à d’autres fins, vous pouvez nettoyer les objets que vous avez créés dans ce labo.

### Supprimer le dossier C:\LabFiles

1. À partir de la machine virtuelle du labo ou de votre ordinateur local si elle n’a pas été fournie, ouvrez l’**Explorateur de fichiers**.
1. Accédez à **C:\\**.
1. Supprimez le dossier **C:\LabFiles**.

### Supprimer la base de données AdventureWorks2017

1. À partir de la machine virtuelle du labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session SQL Server Management Studio (SSMS).
1. Lorsque SSMS s’ouvre, par défaut, la boîte de dialogue **Se connecter au serveur** s’affiche. Choisissez l’instance par défaut, puis sélectionnez **Se connecter**. Vous devrez peut-être cocher la case **Faire confiance au certificat de serveur**.
1. Dans l’**Explorateur d’objets**, développez le dossier **Bases de données**.
1. Cliquez avec le bouton droit sur la base de données **AdventureWorks2017**, puis sélectionnez **Supprimer**.
1. Dans la boîte de dialogue **Supprimer l’objet**, cochez la case **Fermer les connexions existantes**.
1. Cliquez sur **OK**.

---

Vous avez terminé ce labo.

Dans cet exercice, vous avez appris à identifier les problèmes de requête et à les résoudre pour améliorer le plan de requête.
