---
lab:
  title: "Labo\_7\_: Détecter et corriger les problèmes de fragmentation"
  module: Monitor and optimize operational resources in Azure SQL
---

# Détecter et corriger les problèmes de fragmentation

**Durée estimée** : 15 minutes

Les participants utiliseront les informations acquises dans les leçons pour définir les produits livrables d’un projet de transformation numérique au sein d’AdventureWorks. En examinant le Portail Azure ainsi que d’autres outils, les participants détermineront comment utiliser les outils natifs pour identifier et résoudre les problèmes liés aux performances. Enfin, les participants seront en mesure d’identifier la fragmentation dans la base de données et d’apprendre les étapes pour la résoudre de manière appropriée.

Vous avez été embauché en tant qu’administrateur de base de données pour identifier les problèmes liés aux performances et fournir des solutions viables aux problèmes détectés. AdventureWorks vend des vélos et des pièces de vélos directement aux consommateurs et aux distributeurs depuis plus de dix ans. Récemment, l’entreprise a constaté une dégradation des performances de ses produits utilisés pour répondre aux demandes des clients. Vous devez utiliser des outils SQL pour identifier les problèmes de performances et suggérer des méthodes pour les résoudre.

**Remarque :** ces exercices vous demandent de copier et coller du code T-SQL. Vérifiez que le code a été copié correctement, avant de l’exécuter.

## Restaurer une base de données

1. Téléchargez le fichier de sauvegarde de la base de données situé sur **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** à l’emplacement **C:\LabFiles\Monitor and optimize** sur la machine virtuelle du labo (créez la structure du dossier si elle n’existe pas).

    ![Image 03](../images/dp-300-module-07-lab-03.png)

1. Sélectionnez le bouton Démarrer de Windows et tapez SSMS. Sélectionnez **Microsoft SQL Server Management Studio 18** dans la liste.  

    ![Image 01](../images/dp-300-module-01-lab-34.png)

1. Lorsque SSMS s’ouvre, remarquez que la boîte de dialogue **Se connecter au serveur** est préremplie avec le nom de l’instance par défaut. Cliquez sur **Se connecter**.

    ![Image 02](../images/dp-300-module-07-lab-01.png)

1. Sélectionnez le dossier **Bases de données**, puis **Nouvelle requête**.

    ![Image 03](../images/dp-300-module-07-lab-04.png)

1. Dans la nouvelle fenêtre de requête, copiez et collez le code T-SQL ci-dessous. Exécutez la requête pour restaurer la base de données.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017_log.ldf';
    ```

    **Remarque** : le nom et le chemin du fichier de sauvegarde de la base de données doivent correspondre à ceux que vous avez téléchargés à l’étape 1, sinon la commande échouera.

1. Vous devriez voir un message de réussite une fois la restauration terminée.

    ![Image 03](../images/dp-300-module-07-lab-05.png)

## Investiguer la fragmentation d’index

1. Sélectionnez **Nouvelle requête**. Copiez et collez le code T-SQL suivant dans la fenêtre de l’éditeur. Sélectionnez **Exécuter** pour exécuter cette requête.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT i.name Index_Name
     , avg_fragmentation_in_percent
     , db_name(database_id)
     , i.object_id
     , i.index_id
     , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
     INNER JOIN sys.indexes i ON ps.object_id = i.object_id 
     AND ps.index_id = i.index_id
    WHERE avg_fragmentation_in_percent > 50 -- find indexes where fragmentation is greater than 50%
    ```

    Cette requête signale tous les index dont la fragmentation est supérieure à **50 %**. La requête ne doit pas renvoyer de résultat.

1. Sélectionnez **Nouvelle requête**. Copiez et collez le code T-SQL suivant dans la fenêtre de l’éditeur. Sélectionnez **Exécuter** pour exécuter cette requête.

    ```sql
    USE AdventureWorks2017
    GO
        
    INSERT INTO [Person].[Address]
        ([AddressLine1]
        ,[AddressLine2]
        ,[City]
        ,[StateProvinceID]
        ,[PostalCode]
        ,[SpatialLocation]
        ,[rowguid]
        ,[ModifiedDate])
        
    SELECT AddressLine1,
        AddressLine2, 
        'Amsterdam',
        StateProvinceID, 
        PostalCode, 
        SpatialLocation, 
        newid(), 
        getdate()
    FROM Person.Address;
    
    GO
    ```

    Cette requête augmente le niveau de fragmentation de la table Person.Address et de ses index, en ajoutant de nombreux nouveaux enregistrements.

1. Exécutez à nouveau la première requête. Vous devriez maintenant voir quatre index très fragmentés.

    ![Image 03](../images/dp-300-module-07-lab-06.png)

1. Copiez et collez le code T-SQL suivant dans la fenêtre de l’éditeur. Sélectionnez **Exécuter** pour exécuter cette requête.

    ```sql
    SET STATISTICS IO,TIME ON
    GO
        
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

    Cliquez sur l’onglet **Messages** dans le volet de résultats de SQL Server Management Studio. Notez le nombre de lectures logiques effectuées par la requête.

    ![Image 03](../images/dp-300-module-07-lab-07.png)

## Reconstruire les index fragmentés

1. Copiez et collez le code T-SQL suivant dans la fenêtre de l’éditeur. Sélectionnez **Exécuter** pour exécuter cette requête.

    ```sql
    USE AdventureWorks2017
    GO
    
    ALTER INDEX [IX_Address_StateProvinceID] ON [Person].[Address] REBUILD PARTITION = ALL 
    WITH (PAD_INDEX = OFF, 
        STATISTICS_NORECOMPUTE = OFF, 
        SORT_IN_TEMPDB = OFF, 
        IGNORE_DUP_KEY = OFF, 
        ONLINE = OFF, 
        ALLOW_ROW_LOCKS = ON, 
        ALLOW_PAGE_LOCKS = ON)
    ```

1. Exécutez la requête ci-dessous pour confirmer que la fragmentation de l’index **IX_Address_StateProvinceID** est inférieure à 50 %.

    ```sql
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT i.name Index_Name
        , avg_fragmentation_in_percent
        , db_name(database_id)
        , i.object_id
        , i.index_id
        , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
        INNER JOIN sys.indexes i ON (ps.object_id = i.object_id AND ps.index_id = i.index_id)
    WHERE i.name = 'IX_Address_StateProvinceID'
    ```

    En comparant les résultats, on constate que la fragmentation est passée de 81 % à 0 %.

1. Réexécutez l’instruction SELECT de la section précédente. Prenez note des lectures logiques qui se trouvent sous l’onglet **Messages** du volet **Résultats** dans Management Studio. Y a-t-il eu un changement par rapport au nombre de lectures logiques qu’il y avait avant la reconstruction de l’index ?

    ```sql
    SET STATISTICS IO,TIME ON
    GO
        
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

Étant donné que l’index a été reconstruit, il sera aussi efficace que possible et le nombre de lectures logiques sera réduit. Vous avez maintenant vu que la maintenance des index peut avoir un impact sur les performances des requêtes.

Dans cet exercice, vous avez appris à reconstruire l’index et à analyser les lectures logiques pour augmenter les performances des requêtes.
