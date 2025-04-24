---
lab:
  title: "Labo\_5\_: Activer Microsoft Defender pour SQL et la classification des données"
  module: Implement a Secure Environment for a Database Service
---

# Activer Microsoft Defender pour SQL et la classification des données

**Durée estimée : 30 minutes**

Les participants utiliseront les informations acquises dans les leçons pour configurer puis mettre en œuvre la sécurité dans le Portail Azure et dans la base de données AdventureWorks.

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

## Activer Microsoft Defender pour le SQL

1. À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session de navigateur et accédez à [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide de vos informations d’identification Azure.

1. Dans le portail Azure, recherchez *serveurs SQL* dans le champ de recherche situé en haut de la page, puis cliquez sur **Serveurs SQL** dans la liste des options.

1. Sélectionnez le serveur SQL Server **dp300-lab-xxxxxxxx**, où *xxxxxxxx* est une chaîne numérique aléatoire.

    > &#128221; Notez que si vous utilisez votre propre serveur Azure SQL Server non créé par ce labo, sélectionnez le nom de ce serveur SQL Server.

1. Dans le panneau *Vue d’ensemble*, sélectionnez **Non configuré** en regard de *Microsoft Defender pour SQL*.

1. Sélectionnez le **X** en haut à droite pour fermer le volet Vue d’ensemble *Microsoft Defender for Cloud*.

1. Sélectionnez **Activer** sous *Microsoft Defender pour SQL*.

1. Dans un environnement de production, il doit y avoir plusieurs recommandations répertoriées. Vous souhaitez sélectionner **Afficher toutes les recommandations dans Defender for Cloud** et passer en revue toutes les recommandations de *Microsoft Defender* répertoriées pour votre serveur Azure SQL Server et les implémenter selon les besoins.

## Évaluation des vulnérabilités

1. Depuis le panneau principal de votre serveur Azure SQL Server, accédez à la section **Paramètres** et sélectionnez **Bases de données SQL**, puis le nom de la base de données nommée **AdventureWorksLT**.

1. Sélectionnez le paramètre **Microsoft Defender for Cloud** sous **Sécurité**.

1. Sélectionnez le **X** en haut à droite pour fermer le volet Vue d’ensemble de *Microsoft Defender for Cloud* et afficher le tableau de bord **Microsoft Defender for Cloud** de votre base de données `AdventureWorksLT`.

1. Pour commencer à examiner les fonctionnalités Évaluation des vulnérabilités, sous **Résultats de l’évaluation des vulnérabilités**, sélectionnez **Afficher des résultats supplémentaires dans Évaluation des vulnérabilités**.  

1. Sélectionnez **Analyser** pour obtenir les résultats les plus récents de l’évaluation des vulnérabilités. Ce processus prend quelques instants pour permettre à l’évaluation des vulnérabilités d’analyser la base de données.

1. Chaque risque de sécurité est associé à un niveau de risque (élevé, moyen ou faible) et à des informations supplémentaires. Les règles en vigueur s’appuient sur des évaluations fournies par le [Center For Internet Security](https://www.cisecurity.org/benchmark/microsoft_sql_server/?azure-portal=true). Sous l’onglet **Résultats**, sélectionnez une vulnérabilité. Notez l’**ID** de la vulnérabilité, par exemple **VA1143** (s’il est répertorié).

1. En fonction du contrôle de sécurité, d’autres affichages et recommandations sont disponibles. Examinez les informations fournies. Pour cette vérification de sécurité, vous pouvez sélectionner le bouton **Ajouter tous les résultats comme base de référence**, puis sélectionner **Oui** pour définir la base de référence. À présent qu’une ligne de base est en place, ce contrôle de sécurité échouera lors des prochaines analyses dont les résultats s’écarteront de la ligne de base. Sélectionnez le bouton **X** dans le coin supérieur droit pour fermer le volet de la règle spécifique.  

1. Nous allons réexécuter l’**Analyse** pour confirmer que la vulnérabilité sélectionnée s’affiche désormais sous la forme d’une vérification de sécurité *Passée*.

    Si vous sélectionnez la vérification de sécurité réussie précédente, vous devez pouvoir voir la base de référence que vous avez configurée. Si quelque chose change à l’avenir, les analyses d’évaluation des vulnérabilités le détecteront et la vérification de sécurité échouera.  

## Advanced Threat Protection

1. Sélectionnez le **X** en haut à droite pour fermer le volet Évaluation des vulnérabilités et revenir au tableau de bord **Microsoft Defender pour le cloud** de votre base de données. Sous **Incidents et alertes de sécurité**, vous ne devriez rien voir. Cela signifie que **Advanced Threat Protection** n’a détecté aucun problème. Advanced Threat Protection détecte les activités anormales qui indiquent des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des bases de données.  

    > &#128221; À ce stade, ne vous attendez pas à voir des alertes de sécurité. À l’étape suivante, vous allez exécuter un test qui va déclencher une alerte afin de vous permettre de passer en revue les résultats dans Advanced Threat Protection.  

    Vous pouvez utiliser Advanced Threat Protection pour identifier les menaces et vous alerter en cas de survenue de l’un des événements suivants :  

    - Injection de code SQL
    - Vulnérabilité à l’injection de code SQL
    - Exfiltration de données
    - Action non sécurisée
    - Force brute
    - Connexion cliente anormale

    Dans cette section, vous allez découvrir comment une alerte sur une injection de code SQL peut être déclenchée au travers du SSMS. Les alertes d’injection de code SQL sont destinées aux applications personnalisées, pas à des outils standard comme SSMS. Par conséquent, pour déclencher une alerte par le biais de SSMS en guise de test pour une injection de code SQL, vous devez « définir » la valeur **Nom d’application**, qui est une propriété de connexion des clients se connectant à SQL Server ou à Azure SQL.

1. À partir de la machine virtuelle de labo ou de votre ordinateur local si elle n’a pas été fournie, ouvrez SQL Server Management Studio (SSMS). Dans la boîte de dialogue Se connecter au serveur, collez le nom de votre serveur Azure SQL Database et connectez-vous à l’aide des identifiants de connexion ci-dessous :

    - **Nom du serveur** : &lt;_collez le nom de votre serveur Azure SQL Database ici._&gt;
    - **Authentification :** authentification SQL Server
    - **Identifiant d’administration du serveur :** votre identifiant d’administration du serveur Azure SQL Database
    - **Mot de passe :** votre mot de passe d’administration du serveur Azure SQL Database

1. Sélectionnez **Connecter**.

1. Dans SSMS, sélectionnez **Fichier** > **Nouveau** > **Requête de moteur de base de données** pour créer une requête à l’aide d’une nouvelle connexion.  

1. Dans la fenêtre de connexion principale, connectez-vous à la base de données **AdventureWorksLT** comme vous le feriez généralement, avec l’authentification SQL et vos informations d’identification d’administration et de nom du serveur Azure SQL Server. Avant de vous connecter, sélectionnez **Options >>** > **Propriétés de connexion**. Tapez **AdventureWorksLT** pour l’option **Connexion à une base de données**.  

1. Sélectionnez l’onglet **Paramètres de connexion supplémentaires**, puis insérez la chaîne de connexion suivante dans la zone de texte :  

    ```sql
    Application Name=webappname
    ```

1. Sélectionnez **Se connecter**.  

1. Dans la nouvelle fenêtre de requête, collez la requête suivante et sélectionnez **Exécuter** :  

    ```sql
    SELECT * FROM sys.databases WHERE database_id like '' or 1 = 1 --' and family = 'test1';
    ```

1. Dans le portail Azure, accédez à votre base de données **AdventureWorksLT**. Dans le volet gauche, sous **Sécurité**, sélectionnez **Microsoft Defender pour le cloud**.

1. Sous **Incidents et alertes de sécurité**, sélectionnez **Rechercher des alertes sur ces ressources dans Microsoft Defender for Cloud**.  

1. Vous pouvez maintenant voir l’ensemble des alertes de sécurité.  

1. Sélectionnez **Injection de code SQL potentielle** pour afficher des alertes plus spécifiques et recevoir des suggestions d’étapes d’investigation.

1. Sélectionnez **Afficher les détails complets** pour afficher les détails de l’alerte.

1. Sous l’onglet **Détails de l’alerte**, notez que l’*Instruction vulnérable* est affichée. Il s’agit de l’instruction SQL exécutée pour déclencher l’alerte. Il s’agissait également de l’instruction SQL exécutée dans SSMS. Notez également que l’**Application cliente** est affichée en tant que **webappname**. Il s’agit du nom que vous avez spécifié dans la chaîne de connexion dans SSMS.

1. En guise d’étape de nettoyage, envisagez de fermer tous les éditeurs de requête dans SSMS et de supprimer toutes les connexions afin d’éviter de déclencher accidentellement des alertes supplémentaires dans l’exercice suivant.

## Activer la classification des données

1. Depuis le panneau principal de votre serveur Azure SQL Server, accédez à la section **Paramètres** et sélectionnez **Bases de données SQL**, puis le nom de la base de données nommée **AdventureWorksLT**.

1. Sur le panneau principal de la base de données **AdventureWorksLT**, accédez à la section **Sécurité**, puis sélectionnez **Découverte et classification des données**.

1. Sur la page **Découverte et classification des données**, un message d’information indique : **Nous utilisons actuellement la stratégie de protection des données SQL. Nous avons trouvé 15 colonnes avec des recommandations de classification**. Sélectionnez ce lien.

1. Dans l’écran suivant **Découverte et classification des données**, cochez la case **Sélectionner tout**, sélectionnez **Accepter les recommandations sélectionnées**, puis sélectionnez **Enregistrer** pour enregistrer les classifications dans la base de données.

1. Revenez à l’écran **Découverte et classification des données** et remarquez que quinze colonnes ont été classées dans cinq tables différentes. Passez en revue le *Type d’informations* et l’*Étiquette de confidentialité* pour chacune des colonnes.

## Configurer la classification et le masquage des données

1. Dans le portail Azure, accédez à votre instance Azure SQL Database (pas au serveur logique) **AdventureWorksLT**.

1. Dans le menu gauche, sous **Sécurité**, sélectionnez **Découverte et classification des données**.  

1. Dans la table des clients SalesLT, le service *Découverte et classification des données* a identifié que `FirstName` et `LastName` devaient être classifiés, mais pas `MiddleName`. Utilisez les listes déroulantes pour l’ajouter maintenant. Sélectionnez le **Nom** du *Type d’informations* et **Confidentiel - RGPD** pour l’*Étiquette de confidentialité*, puis sélectionnez **Ajouter une classification**.  

1. Cliquez sur **Enregistrer**.

1. Vérifiez que la classification est correctement ajoutée en affichant l’onglet **Vue d’ensemble** pour vous assurer que `MiddleName` figure maintenant dans la liste des colonnes classifiées du schéma SalesLT.

1. Dans le volet gauche, sélectionnez **Vue d’ensemble** pour revenir à la vue d’ensemble de votre base de données.  

   Dynamic Data Masking (DDM) est disponible à la fois dans Azure SQL et SQL Server. DDM limite l’exposition des données en masquant les données sensibles aux utilisateurs non privilégiés au niveau de SQL Server plutôt qu’au niveau de l’application, où vous devez coder ces types de règles. Azure SQL vous recommande des éléments à masquer. Vous pouvez également ajouter des masques manuellement.

   Dans les étapes suivantes, vous allez masquer les colonnes `FirstName`, `MiddleName` et `LastName` que vous avez révisées à l’étape précédente.  

1. Dans le portail Azure, accédez à votre instance Azure SQL Database. Dans le volet gauche, sous **Sécurité**, sélectionnez **Dynamic Data Masking**, puis **Ajouter un masque**.  

1. Dans les listes déroulantes, sélectionnez le schéma **SalesLT**, la table **Customer** et la colonne **FirstName**. Vous pouvez ensuite passer en revue les options de masquage, mais l’option par défaut convient pour ce scénario. Sélectionnez **Ajouter** pour ajouter la règle de masquage.  

1. Répétez l’étape précédente pour les colonnes **MiddleName** et **LastName** dans cette table.  

    Vous avez maintenant trois règles de masquage.  

1. Cliquez sur **Enregistrer**.

    > &#128221; Notez que si votre nom Azure SQL Server n’est pas constitué uniquement de lettres minuscules, de chiffres et de tirets, cette étape échoue et vous ne pourrez pas continuer avec les sections de masquage des données.

1. Dans le volet gauche, sélectionnez **Vue d’ensemble** pour revenir à la vue d’ensemble de votre base de données.

## Récupérer des données classifiées et masquées

Vous simulez ensuite une personne interrogeant les colonnes classifiées et découvrant Dynamic Data Masking en action.

1. Ouvrez SQL Server Management Studio (SSMS), connectez-vous à votre instance Azure SQL Server et ouvrez une nouvelle fenêtre de requête.

1. Cliquez avec le bouton droit sur la base de données **AdventureWorksLT**, puis sélectionnez **Nouvelle requête**.  

1. Lancez la requête suivante pour obtenir les données classifiées (et, dans certains cas, des colonnes marquées pour des données masquées). Sélectionnez **Exécuter** pour exécuter la requête.

    ```sql
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    ```

    Votre résultat doit présenter les 10 premiers noms, sans aucun masquage appliqué. Pourquoi ? Parce que vous êtes l’administrateur de ce serveur logique Azure SQL Database.  

1. Dans la requête suivante, vous créez un utilisateur et exécutez la requête précédente en tant que cet utilisateur. Vous allez également utiliser `EXECUTE AS` pour emprunter l’identité de `Bob`. Lors de l’exécution d’une instruction `EXECUTE AS`, le contexte d’exécution de la session bascule vers la connexion ou l’utilisateur. Cela signifie que les autorisations sont vérifiées par rapport à la connexion ou l’utilisateur plutôt que par rapport à la personne exécutant la commande `EXECUTE AS` (en l’occurrence, vous). `REVERT` est ensuite utilisé pour arrêter l’emprunt d’identité de la connexion ou de l’utilisateur.  

    Vous reconnaîtrez peut-être les premières parties des commandes qui suivent, car elles sont une répétition d’un exercice précédent. Créez une requête avec les commandes suivantes, puis sélectionnez **Exécuter** pour exécuter la requête, enfin observez les résultats.

    ```sql
    -- Create a new SQL user and give them a password
    CREATE USER Bob WITH PASSWORD = 'c0mpl3xPassword!';

    -- Until you run the following two lines, Bob has no access to read or write data
    ALTER ROLE db_datareader ADD MEMBER Bob;
    ALTER ROLE db_datawriter ADD MEMBER Bob;

    -- Execute as our new, low-privilege user, Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;
    ```

    Le résultat doit maintenant présenter les 10 premiers noms, mais avec le masquage appliqué. Bob n’a pas été autorisé à accéder à la forme non masquée de ces données.  

    Que se passe-t-il si, pour une raison quelconque, Bob a besoin d’accéder aux noms et y est autorisé ?  

    Vous pouvez mettre à jour les utilisateurs exclus du masquage dans le portail Azure en accédant au volet **Masquage dynamique des données**, sous **Sécurité**, ainsi qu’en utilisant T-SQL.

1. Cliquez avec le bouton droit sur la base de données **AdventureWorksLT** et sélectionnez **Nouvelle requête**, puis entrez la requête suivante pour permettre à Bob d’interroger les résultats de noms sans masquage. Sélectionnez **Exécuter** pour exécuter la requête.

    ```sql
    GRANT UNMASK TO Bob;  
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Vos résultats devraient inclure les noms en entier.  

1. Vous pouvez également retirer à un utilisateur ses privilèges de démasquage et confirmer cette action en exécutant les commandes T-SQL suivantes dans une nouvelle requête :  

    ```sql
    -- Remove unmasking privilege
    REVOKE UNMASK TO Bob;  

    -- Execute as Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Vos résultats devraient inclure les noms masqués.  

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

Dans cet exercice, vous avez renforcé la sécurité d’une base de données Azure SQL en activant Microsoft Defender pour SQL. Vous avez également créé des colonnes classifiées en fonction des recommandations du Portail Azure.
