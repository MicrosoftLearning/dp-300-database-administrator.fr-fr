---
lab:
  title: "Labo\_12\_: Créer une alerte sur l’état du processeur pour SQL Server"
  module: Automate database tasks for Azure SQL
---

# Créer une alerte sur l’état du processeur pour SQL Server sur Azure

**Durée estimée : 30 minutes**

Vous avez été embauché en tant qu’ingénieur Données senior pour aider à automatiser les opérations quotidiennes d’administration de base de données. Cette automatisation a pour objectif de s’assurer que les bases de données pour AdventureWorks continuent de fonctionner à des performances optimales, ainsi que de fournir des méthodes pour générer des alertes en fonction de certains critères.

## Créer une alerte quand un processeur dépasse 80 pour cent

1. Dans la barre de recherche en haut du portail Azure, saisissez **SQL** et sélectionnez **bases de données SQL**. Sélectionnez le nom de la base de données **AdventureWorksLT** dans la liste.

    ![Capture d’écran de la sélection du nom du serveur SQL](../images/dp-300-module-12-lab-01.png)

1. Sur le panneau principal de la base de données **AdventureWorksLT**, naviguez jusqu’à la section de surveillance. Sélectionnez **Alertes**.

    ![Capture d’écran de sélection des alertes sur la page vue d’ensemble de la base de données SQL](../images/dp-300-module-12-lab-02.png)

1. Sélectionnez **Créer une règle d’alerte**.

    ![Capture d’écran de la sélection d’une nouvelle règle d’alerte](../images/dp-300-module-12-lab-03.png)

1. Dans la section **Sélectionner un signal**, sélectionnez **Pourcentage du processeur**.

    ![Capture d’écran de la sélection du pourcentage du processeur](../images/dp-300-module-12-lab-04.png)

1. Dans la section **Configurer le signal**, sélectionnez **Statique** pour la propriété **Seuil**. Vérifiez ensuite que la propriété **Opérateur** est **Supérieur à** et que le **Type d’agrégation** est **Moyen**. Ensuite, dans **valeur de seuil** , entrez la valeur **80**. Cliquez sur **Terminé**.

    ![Capture d’écran de la saisie de 80 et de la sélection de Terminé](../images/dp-300-module-12-lab-05.png)

1. Sélectionnez l’onglet **Actions**.

    ![Capture d’écran de la sélection du lien Sélectionner un groupe d’actions](../images/dp-300-module-12-lab-06.png)

1. Sous l’onglet **Actions**, sélectionnez **Créer un groupe d’actions**.

    ![Capture d’écran de la sélection Créer un groupe d’actions](../images/dp-300-module-12-lab-07.png)

1. Dans l’écran **Groupe d’actions**, tapez **emailgroup** dans le champ **Nom du groupe d’actions**, puis sélectionnez **Suivant : Notifications**.

    ![Capture d’écran de la saisie de emailgroup puis de la sélection Suivant : Notifications](../images/dp-300-module-12-lab-08.png)

1. Sous l’onglet **Notifications**, entrez les informations suivantes :

    - **Type de notification** : E-mail/SMS message/Push/Voice
        - **Remarque :** lorsque vous sélectionnez cette option, un menu volant Email/SMS message/Push/Voice s’affiche. Vérifiez la propriété Email et saisissez le nom d’utilisateur Azure avec lequel vous vous êtes connecté.
    - **Nom** : DemoLab

    ![Capture d’écran de la page créer un groupe d’actions avec les informations ajoutées](../images/dp-300-module-12-lab-09.png)

1. Sélectionnez **Vérifier + créer**, puis sélectionnez **Créer**.

    ![Capture d’écran de la page Créer une règle d’alerte en sélectionnant créer une règle d’alerte](../images/dp-300-module-12-lab-10.png)

    **Remarque :** avant de sélectionner **Créer**, vous pouvez également sélectionner le **Groupe d’actions de test (préversion)** pour tester l’alerte.

1. Un e-mail de ce type est envoyé à l’adresse de messagerie que vous avez entrée, une fois la règle créée.

    ![Capture d’écran de la confirmation par e-mail](../images/dp-300-module-12-lab-11.png)

    Une fois l’alerte en place, si l’utilisation du processeur en moyenne dépasse 80 %, un e-mail comme celui-ci est envoyé.

    ![Capture d’écran de l’e-mail d’alerte](../images/dp-300-module-12-lab-12.png)

Les alertes peuvent vous envoyer un e-mail ou appeler un webhook lorsqu’une métrique (taille de la base de données ou utilisation du processeur, par exemple) atteint un seuil que vous avez défini. Vous venez de voir comment vous pouvez facilement configurer des alertes pour les bases de données Azure SQL.
