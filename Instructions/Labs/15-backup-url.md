---
lab:
  title: "Labo\_15\_: Sauvegarder sur une URL et restaurer à partir d’une URL"
  module: Plan and implement a high availability and disaster recovery solution
---

# Sauvegarde vers une URL

**Durée estimée : 30 minutes**

En tant qu’administrateur de base données pour AdventureWorks, vous devez sauvegarder une base de données vers une URL dans Azure et la restaurer à partir du stockage Blob Azure à la suite d’une erreur humaine.

## Configurez l’environnement.

Si votre machine virtuelle de labo a été fournie et préconfigurée, vous devez trouver les fichiers de labo prêts dans le dossier **C :\LabFiles**. *Prenez un moment pour vérifier et si les fichiers sont déjà là, ignorez cette section*. Toutefois, si vous utilisez votre propre ordinateur ou que les fichiers de labo sont manquants, vous devez les cloner à partir de *GitHub* pour continuer.

1. À partir de la machine virtuelle du labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session Visual Studio Code.

1. Ouvrez la palette de commandes (Ctrl+Maj+P), puis tapez **Git: Clone**. Sélectionnez l’option **Git: Clone**.

1. Collez l’URL suivante dans le champ **URL du référentiel**, puis sélectionnez **Entrée**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Enregistrez le référentiel dans le dossier **C :\LabFiles** sur la machine virtuelle de labo ou votre ordinateur local si elle n’a pas été fournie (créez le dossier s’il n’existe pas).

## Restaurer la base de données

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

## Configurer la sauvegarde vers une URL

1. À partir de la machine virtuelle du labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session Visual Studio Code.

1. Ouvrez le référentiel cloné à l’adresse **C:\LabFiles\dp-300-database-administrator**.

1. Cliquez avec le bouton droit sur le dossier **Allfiles**, puis sélectionnez **Ouvrir dans le terminal intégré**. Cette opération ouvre une fenêtre de terminal à l’emplacement approprié.

1. Dans le terminal, tapez la commande suivante, puis appuyez sur **Entrée**.

    ```bash
    az login
    ```

1. Vous recevez une invitation à ouvrir un navigateur sur et à fournir un code. Suivez les instructions pour vous connecter à votre compte Azure.

1. *Si vous avez déjà un groupe de ressources, ignorez cette étape*. Si vous n’avez pas de groupe de ressources, créez-en un à l’aide de la commande suivante dans le terminal. Remplacez *contoso-rgXXX######* par un nom unique pour votre groupe de ressources. Le nom doit être unique dans tout Azure. Remplacez votre emplacement (-l) par l’emplacement de votre groupe de ressources.

    ```bash
    az group create -n "contoso-rglod#######" -l eastus2
    ```

    Remplacez **######** pour certains caractères aléatoires.

1. Dans le terminal, tapez ce qui suit et appuyez sur **Entrée** pour créer un compte de stockage. Assurez-vous que le nom du compte de stockage est unique. *Le nom doit contenir entre 3 et 24 caractères, et uniquement des lettres minuscules et des chiffres*. Remplacez *########* par 8 caractères numériques aléatoires. Le nom doit être unique dans tout Azure. Remplacez contoso-rgXXX###### par le nom de votre groupe de ressources. Enfin, remplacez votre emplacement (-l) par l’emplacement de votre groupe de ressources.

    ```bash
    az storage account create -n "dp300bckupstrg########" -g "contoso-rgXXX########" --kind StorageV2 -l eastus2
    ```

1. Vous obtiendrez ensuite les clés de votre compte de stockage, que vous utiliserez dans les étapes suivantes. Exécutez le code suivant dans le terminal en utilisant le nom unique de votre compte de stockage et de votre groupe de ressources.

    ```bash
    az storage account keys list -g contoso-rgXXX######## -n dp300bckupstrg########
    ```

    Votre clé de compte figurera dans les résultats de la commande ci-dessus. Veillez à utiliser le même nom (après le **-n**) et le même groupe de ressources (après le **-g**) que ceux utilisés dans la commande précédente. Copiez la valeur renvoyée pour **key1** (sans les guillemets doubles).

1. La sauvegarde d’une base de données dans SQL Server vers une URL utilise un conteneur dans un compte de stockage. Vous allez créer un conteneur spécifiquement pour le stockage de sauvegarde dans cette étape. Pour ce faire, exécutez les commandes ci-dessous.

    ```bash
    az storage container create --name "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --fail-on-exist
    ```

    Où **dp300bckupstrg########** est le nom unique du compte de stockage utilisé à sa création et **storage_key** la clé générée précédemment. La sortie doit retourner la valeur **true**.

1. Pour vérifier si les sauvegardes du conteneur ont été créées correctement, exécutez :

    ```bash
    az storage container list --account-name "dp300bckupstrg########" --account-key "storage_key"
    ```

    Où **dp300bckupstrg########** est le nom unique du compte de stockage utilisé à sa création et **storage_key** la clé générée.

1. Une signature d’accès partagé doit être générée au niveau du conteneur à des fins de sécurité. Dans le terminal, exécutez la commande suivante :

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    Où **dp300bckupstrg########** est le nom unique du compte de stockage utilisé lors de la création du compte de stockage, **storage_key** est la clé générée et **date_in_the_future** est une date ultérieure à aujourd’hui. La valeur **date_in_the_future** doit être au format UTC, par exemple **2025-12-31T00:00Z** pour spécifier une expiration le 31 décembre 2025 à minuit.

    La sortie doit renvoyer quelque chose de similaire à ceci. Copiez l’intégralité de la signature d’accès partagé et collez-la dans le **bloc-notes**. Elle sera utilisée pour la tâche suivante.

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

## Créer des informations d’identification

La fonctionnalité est maintenant configurée. Vous pouvez à présent générer un fichier de sauvegarde sous forme d’objet blob dans le compte de stockage Azure.

1. Démarrez **SSMS (SQL Server Management Studio)**.

1. Vous êtes invité à vous connecter à SQL Server. Vérifiez que l’option **Authentification Windows** est sélectionnée, puis sélectionnez **Se connecter**.

1. Sélectionnez **Nouvelle requête**.

1. Créez les informations d’identification qui seront utilisées pour accéder au stockage dans le cloud avec le code Transact-SQL suivant. Renseignez les valeurs appropriées, puis sélectionnez **Exécuter**.

    ```sql
    IF NOT EXISTS  
    (SELECT * 
        FROM sys.credentials  
        WHERE name = 'https://<storage_account_name>.blob.core.windows.net/backups')  
    BEGIN
        CREATE CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]
        WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<key_value>'
    END;
    GO  
    ```

    Les deux occurrences de **<storage_account_name>** correspondent au nom unique du compte de stockage et **<key_value>** est la valeur générée à la fin de la tâche précédente, dans ce format :

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

1. Vous pouvez vérifier si les informations d’identification ont été créées correctement en accédant à **Sécurité -> Informations d’identification** dans l’explorateur d’objets dans SSMS.

1. En cas d’erreur de saisie, si vous devez recréer les informations d’identification, vous pouvez les supprimer à l’aide de la commande suivante, en veillant à modifier le nom du compte de stockage :

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## Sauvegarder une base de données vers une URL

1. À l’aide de SSMS, sauvegardez la base de données **AdventureWorks2017** dans Azure avec la commande Transact-SQL suivante :

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    Où **<storage_account_name>** est le nom unique de compte de stockage utilisé. 

    Si une erreur se produit, vérifiez que vous n’avez pas fait d’erreur de saisie lors de la création des informations d’identification et que tout a bien été créé.

## Valider la sauvegarde via Azure CLI

Pour vérifier si le fichier est effectivement dans Azure, vous pouvez utiliser l’Explorateur Stockage (préversion) ou Azure Cloud Shell.

1. Dans le terminal Visual Studio Code, exécutez cettecommande Azure CLI :

    ```bash
    az storage blob list -c "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --output table
    ```

    Veillez à utiliser le même nom de compte de stockage unique (après **--account-name**) et la même clé de compte (après **--account-key**) que dans les commandes précédentes.

    Nous pouvons confirmer que le fichier de sauvegarde a été correctement généré.

## Valider la sauvegarde via le navigateur de stockage

1. Dans une fenêtre de navigateur, accédez au portail Azure et recherchez et sélectionnez **Comptes de stockage**.

1. Sélectionnez le nom unique du compte de stockage que vous avez créé pour les sauvegardes.

1. Dans le volet de navigation de gauche, sélectionnez **Navigateur de stockage**. Développez **Conteneurs d’objets blob**.

1. Sélectionnez **Sauvegardes**.

1. Notez que le fichier de sauvegarde est stocké dans le conteneur.

## Restaurer à partir de l'URL

Cette tâche vous montre comment restaurer une base de données à partir d’un Stockage Blob Azure.

1. À partir de **SQL Server Management Studio (SSMS),** sélectionnez **Nouvelle requête**, puis collez et exécutez la requête suivante.

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

1. Exécutez cette commande pour modifier l’adresse de ce client.

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. Exécutez de nouveau l’**étape 1** pour vérifier que l’adresse a été modifiée. À présent, imaginez que quelqu’un ait modifié des milliers ou des millions de lignes sans clause WHERE ou avec une clause WHERE incorrecte. L’une des solutions consiste à restaurer la base de données à partir de la dernière sauvegarde disponible.

1. Pour restaurer la base de données à l’état précédent la modification par erreur du nom du client, procédez comme suit.

    > &#128221; La syntaxe **SET SINGLE_USER WITH ROLLBACK IMMEDIATE** permet que toutes les transactions ouvertes soient restaurées. Cela permet d’éviter que la restauration n’échoue en raison de connexions actives.

    ```sql
    USE [master]
    GO

    ALTER DATABASE AdventureWorks2017 SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    GO

    RESTORE DATABASE AdventureWorks2017 
    FROM URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak'
    GO

    ALTER DATABASE AdventureWorks2017 SET MULTI_USER
    GO
    ```

    Où **<storage_account_name>** est le nom unique de compte de stockage que vous avez créé.

1. Exécutez de nouveau l’**étape 1** pour vérifier que le nom du client a été restauré.

Il est important de comprendre les composants et leur interaction pour effectuer une sauvegarde ou une restauration à l’aide du service Stockage Blob Azure.

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

Si vous n’utilisez pas la base de données ou les fichiers de labo à d’autres fins, vous pouvez nettoyer les objets que vous avez créés dans ce labo.

### Supprimer le dossier C:\LabFiles

1. À partir de la machine virtuelle du labo ou de votre ordinateur local si elle n’a pas été fournie, ouvrez l’**Explorateur de fichiers**.
1. Accédez à **C:\\**.
1. Supprimez le dossier **C:\LabFiles**.

## Supprimer la base de données AdventureWorks2017

1. À partir de la machine virtuelle du labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session SQL Server Management Studio (SSMS).
1. Lorsque SSMS s’ouvre, par défaut, la boîte de dialogue **Se connecter au serveur** s’affiche. Choisissez l’instance par défaut, puis sélectionnez **Se connecter**. Vous devrez peut-être cocher la case **Faire confiance au certificat de serveur**.
1. Dans l’**Explorateur d’objets**, développez le dossier **Bases de données**.
1. Cliquez avec le bouton droit sur la base de données **AdventureWorks2017**, puis sélectionnez **Supprimer**.
1. Dans la boîte de dialogue **Supprimer l’objet**, cochez la case **Fermer les connexions existantes**.
1. Cliquez sur **OK**.

---

Vous avez terminé ce labo.

Vous avez vu que vous pouvez sauvegarder une base de données vers une URL dans Azure et, si nécessaire, la restaurer.
