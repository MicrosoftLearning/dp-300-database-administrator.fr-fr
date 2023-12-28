---
lab:
  title: "Labo\_11\_: Déployer Azure SQL Database avec un modèle Azure Resource Manager"
  module: Automate database tasks for Azure SQL
---

# Déployer une base de données Azure SQL à partir d’un modèle

**Durée estimée** : 15 minutes

Vous avez été embauché en tant qu’ingénieur Données senior pour automatiser les opérations d’administration de base de données quotidiennes. Cette automatisation permet de s’assurer que les bases de données pour AdventureWorks continuent de fonctionner à des performances optimales, ainsi que de fournir des méthodes de création d’alertes en fonction de certains critères. AdventureWorks utilise SQL Server à la fois comme infrastructure en tant que service (IaaS) et plateforme en tant que service (PaaS).

## Explorer le modèle Azure Resource Manager

1. Dans Microsoft Edge, ouvrez un nouvel onglet et accédez au chemin suivant dans un dépôt GitHub qui contient un modèle ARM pour déployer une ressource SQL Database.

    ```
    https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-database
    ```

1. Cliquez avec le bouton droit sur **azuredeploy.json**, puis sélectionnez **Ouvrir le lien dans un nouvel onglet** pour afficher le modèle ARM, lequel doit ressembler à ceci :

    ```JSON
    {
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "serverName": {
        "type": "string",
        "defaultValue": "[uniqueString('sql', resourceGroup().id)]",
        "metadata": {
            "description": "The name of the SQL logical server."
        }
        },
        "sqlDBName": {
        "type": "string",
        "defaultValue": "SampleDB",
        "metadata": {
            "description": "The name of the SQL Database."
        }
        },
        "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
            "description": "Location for all resources."
        }
        },
        "administratorLogin": {
        "type": "string",
        "metadata": {
            "description": "The administrator username of the SQL logical server."
        }
        },
        "administratorLoginPassword": {
        "type": "securestring",
        "metadata": {
            "description": "The administrator password of the SQL logical server."
        }
        }
    },
    "variables": {},
    "resources": [
        {
        "type": "Microsoft.Sql/servers",
        "apiVersion": "2020-02-02-preview",
        "name": "[parameters('serverName')]",
        "location": "[parameters('location')]",
        "properties": {
            "administratorLogin": "[parameters('administratorLogin')]",
            "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
        },
        "resources": [
            {
            "type": "databases",
            "apiVersion": "2020-08-01-preview",
            "name": "[parameters('sqlDBName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "dependsOn": [
                "[resourceId('Microsoft.Sql/servers', concat(parameters('serverName')))]"
            ]
            }
        ]
        }
    ]
    }
    ```

1. Passez en revue et observez les propriétés JSON.

1. Fermez l’onglet **azuredeploy.json** et revenez à l’onglet contenant le dossier GitHub **sql-database**. Faites défiler vers le bas et sélectionnez **Déployer sur Azure**.

    ![Bouton Déployer dans Azure](../images/dp-300-module-11-lab-01.png)

1. La page du modèle de démarrage rapide **Créer un serveur SQL et une base de données** s’ouvre sur le Portail Azure. Les informations de la ressource sont partiellement remplies à partir du modèle ARM. Complétez les champs vides avec les informations suivantes :

    - **Groupe de ressources** : doit commencer par *contoso-rg*.
    - **Identifiant de connexion de l’administrateur SQL** : labadmin
    - **Mot de passe de connexion de l’administrateur SQL** : &lt;entrez un mot de passe fort.&gt;

1. Sélectionnez **Examiner + créer**, puis sélectionnez **Créer**. Votre déploiement prend environ 5 minutes.

    ![Image 2](../images/dp-300-module-11-lab-02.png)

1. Une fois le déploiement terminé, sélectionnez **Accéder au groupe de ressources**. Vous êtes dirigé vers votre groupe de ressources Azure qui contient une ressource **SQL Server** nommée de façon aléatoire, créée par le déploiement.

    ![Image 3](../images/dp-300-module-11-lab-03.png)

Vous venez de voir comment, en un seul clic sur un lien de modèle Azure Resource Manager, vous pouvez créer un serveur et une base de données Azure SQL en toute simplicité.
