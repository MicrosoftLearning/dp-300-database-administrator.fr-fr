---
lab:
  title: "Labo\_2\_: Provisionner une base de données Azure SQL"
  module: Plan and Implement Data Platform Resources
---

# Provisionner une base de données Azure SQL

**Durée estimée : 40 minutes**

Les participants configureront les ressources de base nécessaires au déploiement d’une base de données Azure SQL avec un point de terminaison de réseau virtuel. La connectivité à la base de données SQL sera validée à l’aide de SQL Server Management Studio à partir de la machine virtuelle de labo si disponible ou à partir de votre configuration de machine locale.

En tant qu’administrateur de base de données pour AdventureWorks, vous allez configurer une nouvelle base de données SQL Database, avec un point de terminaison de réseau virtuel pour augmenter et simplifier la sécurité du déploiement. SQL Server Management Studio permet d’évaluer l’utilisation d’un notebook SQL pour l’interrogation des données et la rétention des résultats.

## Accéder au Portail Azure

1. Sur la machine virtuelle de labo si elle est fournie, sinon sur votre ordinateur local, ouvrez une fenêtre de navigateur.

1. Accédez au portail Azure à l’adresse suivante : [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail Azure à l’aide de votre compte Azure ou des informations d’identification fournies le cas échéant.

1. Depuis le portail Azure, recherchez *groupes de ressources* dans la zone de recherche située en haut, puis sélectionnez **Groupes de ressources** dans la liste des options.

1. Dans la page **Groupe de ressources**, si elle est fournie, sélectionnez le groupe de ressources commençant par *contoso-rg*. Si ce groupe de ressources n’existe pas, créez un groupe de ressources nommé *contoso-rg* dans votre région locale, ou utilisez un groupe de ressources existant et notez sa région.

## Création d'un réseau virtuel

1. Sur la page d’accueil du Portail Azure, sélectionnez le menu de gauche.  

1. Dans le volet de navigation de gauche, sélectionner **Réseaux virtuels**  

1. Sélectionnez **+ Créer** pour ouvrir la page **Créer un réseau virtuel**. Sous l’onglet **Informations de base**, renseignez les informations suivantes :

    - **Abonnement** : &lt;Votre abonnement&gt;
    - **Groupe de ressources :** commençant par *DP300* ou le groupe de ressources que vous avez sélectionné précédemment.
    - **Nom :** lab02-vnet
    - **Région :** sélectionnez la même région que celle dans laquelle votre groupe de ressources a été créé

1. Sélectionnez **Examiner et créer**, vérifiez les paramètres du nouveau réseau virtuel, puis sélectionnez **Créer**.

## Approvisionner une base de données Azure SQL Database dans le portail Azure

1. Dans le portail Azure, recherchez *bases de données SQL* dans le champ de recherche situé en haut de la page, puis sélectionnez **Bases de données SQL** dans la liste des options.

1. Dans le panneau **Bases de données SQL**, sélectionnez **+ Créer**.

1. Sur la page **Créer une base de données SQL**, sélectionnez les options suivantes dans l’onglet **Informations de base**, puis sélectionnez **Suivant : mise en réseau**.

    - **Abonnement** : &lt;Votre abonnement&gt;
    - **Groupe de ressources :** commençant par *DP300* ou le groupe de ressources que vous avez sélectionné précédemment.
    - **Nom de la base de données** : AdventureWorksLT
    - **Serveur :**  sélectionnez le lien **Créer nouveau**. La page **Créer un serveur SQL Database** s’ouvre. Fournissez les détails du serveur comme suit :
        - **Nom du serveur :** dp300-lab-&lt;vos initiales (minuscules)&gt; et, si nécessaire, un nombre aléatoire de 5 chiffres (le nom du serveur doit être globalement unique).
        - **Emplacement :** &lt;votre région locale, identique à la région sélectionnée pour votre groupe de ressources, sinon l’opération risque d’échouer.&gt;
        - **Méthode d’authentification** : utilisez l’authentification SQL.
        - **Identifiant de connexion au serveur de l’administrateur** : dp300admin
        - **Mot de passe :** sélectionnez un mot de passe complexe et notez-le.
        - **Confirmer le mot de passe :** sélectionnez le même mot de passe précédemment sélectionné.
    - Sélectionnez **OK** pour revenir à la page **Créer une base de données SQL**.
    - **Vous souhaitez utiliser un pool élastique ?** défini sur **Non**.
    - **Environnement de charge de travail** : développement
    - Dans l’option **Calcul + stockage**, sélectionnez le lien **Configurer la base de données**. Dans la page **Configurer**, pour la liste déroulante **Niveau de service**, sélectionnez **De base**, puis **Appliquer**.

1. Pour l’option **Redondance du stockage de sauvegarde**, conservez la valeur par défaut : **stockage de sauvegarde localement redondant**.

1. Puis sélectionnez **Suivant : mise en réseau**.

1. Dans l’onglet **Mise en réseau**, pour l’option **Connectivité du réseau**, sélectionnez le bouton d’option **Point de terminaison privé**.

1. Sélectionnez ensuite le lien **+ Ajouter un point de terminaison privé** sous l’option **Points de terminaison privés**.

1. Complétez le volet droit **Créer un point de terminaison privé** comme suit :

    - **Abonnement** : &lt;Votre abonnement&gt;
    - **Groupe de ressources :** commençant par *DP300* ou le groupe de ressources que vous avez sélectionné précédemment.
    - **Emplacement :** &lt;votre région locale, identique à la région sélectionnée pour votre groupe de ressources, sinon l’opération risque d’échouer.&gt;
    - **Nom :** DP-300-SQL-Endpoint
    - **Sous-ressource cible** : SqlServer
    - **Réseau virtuel (Vnet)**  : lab02-vnet
    - **Sous-réseau :** lab02-vnet/default (10.x.0.0/24)
    - **Intégrer à une zone DNS privée** : Oui
    - **Zone DNS privé :** conserver la valeur par défaut
    - Passez en revue les paramètres, puis sélectionnez **OK**.  

1. Le nouveau point de terminaison apparaît dans la liste des **Points de terminaison privés**.

1. Sélectionnez **Suivant : sécurité**, puis **Suivant : paramètres supplémentaires**.  

1. Sur la page **Paramètres supplémentaires**, sélectionnez **Échantillon** dans l’option **Utiliser des données existantes**. Sélectionnez **OK** si un message contextuel s’affiche pour l’échantillon de base de données.

1. Sélectionnez **Vérifier + créer**.

1. Passez en revue les paramètres avant de sélectionner **Créer**.

1. Une fois le déploiement effectué, sélectionnez **Accéder à la ressource**.

## Activer l’accès à une base de données Azure SQL.

1. Dans la page **Base de données SQL**, sélectionnez la section **Vue d’ensemble**, puis sélectionnez le lien du nom du serveur dans la section supérieure.

1. Sur le panneau de navigation des serveurs SQL, sélectionnez **Mise en réseau** dans la section **Sécurité**.

1. Sous l’onglet **Accès public**, sélectionnez **Réseaux sélectionnés**.

1. Sélectionnez **+ Ajouter votre adresse IPv4 cliente**. Cela ajoute une règle de pare-feu pour autoriser votre adresse IP actuelle à accéder au serveur SQL Server.

1. Cochez la propriété **Autoriser les services et les ressources Azure à accéder à ce serveur**.

1. Cliquez sur **Enregistrer**.

---

## Connectez-vous à la base de données Azure SQL à l’aide de SQL Server Management Studio.

1. Dans le volet sur le côté gauche du portail Azure, sélectionnez **Bases de données SQL**. Sélectionnez ensuite la base de données **AdventureWorksLT**.

1. Dans la page **Vue d’ensemble**, copiez la valeur **Nom du serveur**.

1. Lancez SQL Server Management Studio à partir de la machine virtuelle de labo si elle est fournie ou de votre ordinateur local si ce n’est pas le cas.

1. Dans la boîte de dialogue **Se connecter au serveur**, collez la valeur **Nom de serveur** que vous avez copiée à partir du portail Azure.

1. Dans la liste déroulante **Authentification**, sélectionnez **Authentification SQL Server**.

1. Dans le champ **Connexion**, entrez **dp300admin**.

1. Dans le champ **Mot de passe**, entrez le mot de passe sélectionné lors de la création du serveur SQL Server.

1. Sélectionnez **Connecter**.

1. SQL Server Management Studio se connectera à votre serveur Azure SQL Database. Vous pouvez développer le serveur, puis le nœud **Bases de données** pour afficher la base de données *AdventureWorksLT*.

## Interroger une base de données Azure SQL Database à l’aide de SQL Server Management Studio

1. Dans SQL Server Management Studi, cliquez avec le bouton droit sur la base de données *AdventureWorksLT*, puis sélectionnez **Nouvelle requête**.

1. Collez l’instruction SQL suivante dans la fenêtre Requête :

    ```sql
    SELECT TOP 10 cust.[CustomerID], 
        cust.[CompanyName], 
        SUM(sohead.[SubTotal]) as OverallOrderSubTotal
    FROM [SalesLT].[Customer] cust
        INNER JOIN [SalesLT].[SalesOrderHeader] sohead
             ON sohead.[CustomerID] = cust.[CustomerID]
    GROUP BY cust.[CustomerID], cust.[CompanyName]
    ORDER BY [OverallOrderSubTotal] DESC
    ```

1. Sélectionnez le bouton **Exécuter** dans la barre d’outils pour exécuter la requête.

1. Passez en revue les résultats de la requête dans le volet **Résultats**.

1. Cliquez avec le bouton droit sur la base de données *AdventureWorksLT*, puis sélectionnez **Nouvelle requête**.

1. Collez l’instruction SQL suivante dans la fenêtre Requête :

    ```sql
    SELECT TOP 10 cat.[Name] AS ProductCategory, 
        SUM(detail.[OrderQty]) AS OrderedQuantity
    FROM salesLT.[ProductCategory] cat
        INNER JOIN [SalesLT].[Product] prod
            ON prod.[ProductCategoryID] = cat.[ProductCategoryID]
        INNER JOIN [SalesLT].[SalesOrderDetail] detail
            ON detail.[ProductID] = prod.[ProductID]
    GROUP BY cat.[name]
    ORDER BY [OrderedQuantity] DESC
    ```

1. Sélectionnez le bouton **Exécuter** dans la barre d’outils pour exécuter la requête.

1. Passez en revue les résultats de la requête dans le volet **Résultats**.

1. Fermez SQL Server Management Studio. Si vous recevez une invitation à enregistrer vos modifications, sélectionnez **Non**.

---

## Nettoyer les ressources

Si vous n’utilisez pas la machine virtuelle à d’autres fins, vous pouvez nettoyer les ressources que vous avez créées dans ce labo.

### Supprimer le groupe de ressources

Si vous avez créé un groupe de ressources pour ce labo, vous pouvez supprimer le groupe de ressources pour supprimer toutes les ressources créées dans ce labo.

1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le volet de navigation de gauche ou recherchez les **Groupes de ressources** dans la barre de recherche et sélectionnez-les dans les résultats.

1. Accédez au groupe de ressources que vous avez créé pour ce labo. Le groupe de ressources contient la machine virtuelle et d’autres ressources créées dans ce labo.

1. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

1. Dans la boîte de dialogue **Supprimer un groupe de ressources**, entrez le nom de votre groupe de ressources pour confirmer, puis sélectionnez **Supprimer**.

1. Attendez que le groupe de ressources soit supprimé.

1. Fermez le portail Azure.

### Supprimez les ressources de labo uniquement.

Si vous n’avez pas créé de groupe de ressources pour ce labo et que vous souhaitez laisser le groupe de ressources et ses ressources précédentes intactes, vous pouvez toujours supprimer les ressources créées dans ce labo.

1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le volet de navigation de gauche ou recherchez les **Groupes de ressources** dans la barre de recherche et sélectionnez-les dans les résultats.

1. Accédez au groupe de ressources que vous avez créé pour ce labo. Le groupe de ressources contient la machine virtuelle et d’autres ressources créées dans ce labo.

1. Sélectionnez toutes les ressources précédées du nom SQL Server que vous avez spécifié précédemment dans le labo. En outre, sélectionnez le réseau virtuel et la zone DNS privée que vous avez créés.

1. Sélectionnez **Supprimer** dans le menu en haut.

1. Dans la boîte de dialogue **Supprimer les ressources**, tapez **Supprimer** et sélectionnez **Supprimer**.

1. Pour confirmer la suppression des ressources, sélectionnez **Supprimer**.

1. Attendez que les ressources soient supprimées.

1. Fermez le portail Azure.

---

Vous avez terminé ce labo.

Dans cet exercice, vous avez vu comment déployer une base de données Azure SQL avec un point de terminaison de réseau virtuel. Vous avez également pu vous connecter à la base de données SQL que vous avez créée à l’aide de SQL Server Management Studio.
