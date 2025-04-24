---
lab:
  title: "Labo\_12\_: Créer une alerte sur l’état du processeur pour SQL Server"
  module: Automate database tasks for Azure SQL
---

# Créer une alerte sur l’état du processeur pour SQL Server sur Azure

**Durée estimée** : 20 minutes

Vous avez été embauché en tant qu’ingénieur Données senior pour aider à automatiser les opérations quotidiennes d’administration de base de données. Cette automatisation a pour objectif de s’assurer que les bases de données pour AdventureWorks continuent de fonctionner à des performances optimales, ainsi que de fournir des méthodes pour générer des alertes en fonction de certains critères.

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

## Créer une alerte quand un processeur dépasse 80 pour cent

1. À partir de la machine virtuelle de labo, ou de votre ordinateur local si elle n’a pas été fournie, démarrez une session de navigateur et accédez à [https://portal.azure.com](https://portal.azure.com/). Connectez-vous au portail à l’aide de vos informations d’identification Azure.

1. Dans la barre de recherche en haut du portail Azure, saisissez **Bases de données SQL** et sélectionnez les **bases de données SQL**. Sélectionnez le nom de la base de données **AdventureWorksLT** dans la liste.

1. Sur le panneau principal de la base de données **AdventureWorksLT**, naviguez jusqu’à la section de surveillance. Sélectionnez **Alertes**.

1. Sélectionnez **Créer une règle d’alerte**.

1. Dans la page **Créer une règle d’alerte**, sélectionnez **Pourcentage UC**.

1. Dans la section **Logique d’alerte**, sélectionnez **Statique** pour le **Type de seuil**. Vérifiez ensuite que le type d’**Agrégation** est **Moyen** et que la propriété **Valeur** est **Supérieure à**. Ensuite, dans **Seuil**, entrez la valeur **80**. Passez en revue les valeurs *Vérifier chaque* et *Période de recherche arrière*.

1. Sélectionnez **Suivant : Actions >**.

1. Sous l’onglet **Actions**, sélectionnez **Créer un groupe d’actions**.

1. Dans l’écran **Groupe d’actions**, tapez **emailgroup** dans les champs **Nom du groupe d’actions** et **Nom d’affichage**, puis sélectionnez **Suivant : Notifications**.

1. Sous l’onglet **Notifications**, entrez les informations suivantes :

    - **Type de notification** : E-mail/SMS message/Push/Voice

        > &#128221; Lorsque vous sélectionnez cette option, un menu volant E-mail/SMS/Notification push/Voix s’affiche. Vérifiez la propriété Email et saisissez le nom d’utilisateur Azure avec lequel vous vous êtes connecté. Cliquez sur **OK**.

    - **Nom** : DemoLab

1. Sélectionnez **Vérifier + créer**, puis sélectionnez **Créer**.

1. De retour dans la page **Créer une règle d’alerte**, sélectionnez **Suivant : Détails** et donnez à la règle d’alerte un nom unique.

1. Sélectionnez **Vérifier + créer**, puis sélectionnez **Créer**.

1. Une fois l’alerte en place, si l’utilisation du processeur dépasse 80 % en moyenne, un e-mail est envoyé.

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

---

Vous avez terminé ce labo.

Les alertes peuvent vous envoyer un e-mail ou appeler un webhook lorsqu’une métrique (taille de la base de données ou utilisation du processeur, par exemple) atteint un seuil que vous avez défini. Vous venez de voir comment vous pouvez facilement configurer des alertes pour les bases de données Azure SQL.
