---
lab:
  title: "Labo\_7\_: Détecter et corriger les problèmes de fragmentation"
  module: Monitor and optimize operational resources in Azure SQL
---

# Détecter et corriger les problèmes de fragmentation

**Durée estimée** : 20 minutes

Les participants utiliseront les informations acquises dans les leçons pour définir les produits livrables d’un projet de transformation numérique au sein d’AdventureWorks. En examinant le Portail Azure ainsi que d’autres outils, les participants détermineront comment utiliser les outils natifs pour identifier et résoudre les problèmes liés aux performances. Enfin, les participants seront en mesure d’identifier la fragmentation dans la base de données et d’apprendre les étapes pour la résoudre de manière appropriée.

Vous avez été embauché en tant qu’administrateur de base de données pour identifier les problèmes liés aux performances et fournir des solutions viables aux problèmes détectés. AdventureWorks vend des vélos et des pièces de vélos directement aux consommateurs et aux distributeurs depuis plus de dix ans. Récemment, l’entreprise a constaté une dégradation des performances de ses produits utilisés pour répondre aux demandes des clients. Vous devez utiliser des outils SQL pour identifier les problèmes de performances et suggérer des méthodes pour les résoudre.

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

1. La fragmentation d’index peut être due à un certain nombre de facteurs, notamment :

    - Mises à jour fréquentes de la table ou de l’index.
    - Insertions ou suppressions fréquentes dans la table ou l’index.
    - Fractionnements de pages.

    Pour augmenter le niveau de fragmentation de la table Person.Address et de ses index, vous allez ajouter de nombreux nouveaux enregistrements. Pour effectuer cette opération, exécutez la requête suivante.

    Sélectionnez **Nouvelle requête**. Copiez et collez le code T-SQL suivant dans la fenêtre de l’éditeur. Sélectionnez **Exécuter** pour exécuter cette requête.

    ```sql
    USE AdventureWorks2017

    GO
    
    -- Insert 60000 records into the Address table    

    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Split Avenue ' + CAST(v1.number AS VARCHAR(10)), 
        'Apt ' + CAST(v2.number AS VARCHAR(10)), 
        'PageSplitTown', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '88' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure unique rowguid
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 300 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;
    GO
    
    -- DELETE 25000 records from the Address table
    DELETE FROM [Person].[Address] WHERE AddressID BETWEEN 35001 AND 60000;

    GO

    -- Insert 40000 records into the Address table
    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Fragmented Street ' + CAST(v1.number AS VARCHAR(10)), 
        'Suite ' + CAST(v2.number AS VARCHAR(10)), 
        'FragmentCity', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '99' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure a unique rowguid per row
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 200 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;

    GO
    ```

    Cette requête augmente le niveau de fragmentation de la table Person.Address et de ses index, en ajoutant et supprimant de nombreux nouveaux enregistrements.

1. Exécutez à nouveau la première requête. Vous devriez maintenant voir quatre index très fragmentés.

1. Sélectionnez une **Nouvelle requête** et copiez et collez le code T-SQL suivant dans la nouvelle fenêtre de requête. Sélectionnez **Exécuter** pour exécuter cette requête.

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

    Sélectionnez l’onglet **Messages** dans le volet de résultats de SQL Server Management Studio. Notez le nombre de lectures logiques effectuées par la requête sur la table **Adresse**.

## Reconstruire les index fragmentés

1. Sélectionnez une **Nouvelle requête** et copiez et collez le code T-SQL suivant dans la nouvelle fenêtre de requête. Sélectionnez **Exécuter** pour exécuter cette requête.

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

1. Sélectionnez une **Nouvelle requête** et exécutez la requête suivante pour confirmer que la fragmentation de l’index **IX_Address_StateProvinceID** est désormais inférieure à 50 %.

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

    En comparant les résultats, on constate que la fragmentation de l’index **IX_Address_StateProvinceI** est passée de 88 % à 0 %.

1. Réexécutez l’instruction SELECT de la section précédente. Prenez note des lectures logiques qui se trouvent sous l’onglet **Messages** du volet **Résultats** dans Management Studio. *Y a-t-il eu un changement par rapport au nombre de lectures logiques qu’il y avait avant la reconstruction de l’index pour la table d’adresse* ?

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

Dans cet exercice, vous avez appris à reconstruire l’index et à analyser les lectures logiques pour augmenter les performances des requêtes.
