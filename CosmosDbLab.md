

# Deploying Database Instances in Azure


## Exercise 1: Deploy a Cosmos DB database

#### Task 1: Open the Azure Portal

1. Open a browser window and navigate to the **Azure Portal** (<https://portal.azure.com>).

1. If prompted, authenticate with the user account account that has the owner role in the Azure subscription you will be using in this lab.

#### Task 2: Create a Cosmos DB database and collection

1. In the upper left corner of the Azure portal, click **Create a resource**.

1. At the top of the **New** blade, in the **Search the Marketplace** text box, type **Cosmos DB** and press **Enter**.

1. On the **Everything** blade, in the search results, click **Azure Cosmos DB**.

1. On the **Azure Cosmos DB** blade, click the **Create** button.

1. On the new **Azure Cosmos DB** blade, perform the following tasks:

    - On the "Select API Option" Page press the "Create" button for "Core (SQL) - Recommended"
    
    - Leave the **Subscription** drop-down list entry set to its default value.

    - Resource group: ensure that the **Create new** option is selected and then, in the text box, type **AADesignLab0701-RG**.

    - In the **Account Name** text box, type a globally unique value.

    - In the **Location** drop-down list, select the Azure region in which you want to deploy resources in this lab.

    - Leave all remaining settings with their default values.

    - Click the **Review + create** button and then click the **Create** button.

1. Wait for the provisioning to complete before you proceed to the next step.

    > **Note**: The deployment could take up to 15 minutes.

1. Navigate to the blade of the newly created Cosmos DB account and click **Keys**.

1. On the Cosmos DB account Keys blade, note the value of the **PRIMARY CONNECTION STRING**. You will need it in the third exercise of this lab.

1. At the top of the portal, click the **Cloud Shell** icon to open a new shell instance.

    > **Note**: The **Cloud Shell** icon is a symbol that is constructed of the combination of the *greater than* and *underscore* characters.
    > Ensure that you choose BASH  (not Powershell)


1. Wait for the **Cloud Shell** to finish its first-time setup procedures before you proceed to the next task.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the resource group that contains the Azure Cosmos DB account you deployed earlier in this task:

    ```
    RESOURCE_GROUP='AADesignLab0701-RG'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the CosmosDB account you created earlier in this task:

    ```sh
    COSMOSDB_NAME=$(az cosmosdb list --resource-group $RESOURCE_GROUP --query "[0].name" --output tsv)
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new CosmosDB database named **FinancialClubDatabase**:

    ```sh
    az cosmosdb sql  database create  --name 'FinancialClubDatabase' --resource-group $RESOURCE_GROUP --account-name $COSMOSDB_NAME
    ```
    
    
1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the primary key of the CosmosDB account you created earlier in this task:

    ```sh
    PRIMARY_KEY=$(az cosmosdb keys list --resource-group $RESOURCE_GROUP --name $COSMOSDB_NAME --output json | jq -r '.primaryMasterKey')
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the URI of the CosmosDB account you created earlier in this task:

    ```sh
    URI="https://$COSMOSDB_NAME.documents.azure.com:443/"
    ```


1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a fixed collection named **MemberCollection** in the newly created database:

    ```sh
    az cosmosdb sql container create --account-name $COSMOSDB_NAME --database-name FinancialClubDatabase --name MemberCollection --partition-key-path '/firstName/lastName' --resource-group $RESOURCE_GROUP
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to display the value of the PRIMARY_KEY variable:

    ```sh
    echo $PRIMARY_KEY
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to display the value of the URI variable:

    ```sh
    echo $URI
    ```

    > **Note**: Take a note of these values - you will need them in the third exercise of this lab.

#### Task 3: Create and query documents in Cosmos DB

1. On the left side of the Azure Cosmos DB account blade, click **Data Explorer**.

1. In the **Data Explorer** pane, if necessary, refresh the pane and then click the **MemberCollection** child node of the **FinancialClubDatabase** node.

1. Click the **New SQL Query** button at the top of the **Data Explorer** pane.

1. In the **Query 1** tab that opened, view the default query:

    ```sql
    SELECT * FROM c
    ```

1. Click the **Execute Query** button at the top of the query editor and verify that the query does not return any results.

1. In the left pane of the Data Explorer, expand the **MemberCollection** node.

1. Click the **Items** child node within the **MemberCollection** node.

1. In the new **Items** tab that opened, click the **New Item** button at the top of the tab.

1. In the **Items** tab, replace the existing document with the following document:

    ```json
    {
        "firstName": "Pennington",
        "lastName": "Oneal",
        "age": 26,
        "salary": 90000.00,
        "company": "Veraq",
        "isVested": false
    }
    ```

1. Click the **Save** button at the top of the **Items** tab (you might need to first click the ellipsis toolbar button).

1. In the **Items** tab, click the **New Item** button at the top of the tab.

1. In the **Items** tab, replace the existing document with the following document:

    ```json
    {
        "firstName": "Suzanne",
        "lastName": "Oneal",
        "company": "Veraq"
    }
    ```

1. Click the **Save** button at the top of the **Items** tab.

1. Switch back to the **Query 1** tab, re-run the default query `SELECT * FROM c` by clicking the **Execute Query** button at the top of the query editor, and review the results.

1. In the query editor, replace the default query with the following query:

    ```sql
    SELECT
        c.id,
        c.firstName,
        c.lastName,
        c.isVested,
        c.company
    FROM
        c
    WHERE
        IS_DEFINED(c.isVested)
    ```

1. Click the **Execute Query** button at the top of the query editor and review the results.

1. In the query editor, replace the existing query with the following query:

    ```sql
    SELECT
        c.id,
        c.firstName,
        c.lastName,
        c.age
    FROM
        c
    WHERE
        c.age > 20
    ```

1. Click the **Execute Query** button at the top of the query editor and review the results.

1. In the query editor, replace the existing query with the following query:

    ```sql
    SELECT VALUE
        c.id
    FROM
        c
    ```

1. Click the **Execute Query** button at the top of the query editor and review the results.

1. In the query editor, replace the existing query with the following query:

    ```sql
    SELECT VALUE {
        "badgeNumber": SUBSTRING(c.id, 0, 8),
        "company": c.company,
        "fullName": CONCAT(c.firstName, " ", c.lastName)
    } FROM c
    ```

1. Click the **Execute Query** button at the top of the query editor and review the results.

> **Review**: In this exercise, you created a new Cosmos DB account, database, and collection, added sample items to the collection, and run sample queries targeting these items.

## Exercise 2: Deploy Application using Cosmos DB

#### Task 1: Deploy API App code using Azure Resource Manager templates and GitHub

1. Download the **api.json** file from [Here](https://bigpopcat.z5.web.core.windows.net/CosmosDB-Lab.zip). Unzip and save **api.json** to your local machine.

1. In the upper left corner of the Azure portal, click **Create a resource**.

1. At the top of the **New** blade, in the **Search the Marketplace** text box, type **Template Deployment** and press **Enter**.

1. On the **Everything** blade, in the search results, click **Template Deployment**.

1. On the **Template deployment** blade, click the **Create** button.

1. On the **Custom deployment** blade, click the **Build your own template in the editor** link.

1. On the **Edit template** blade, click the **Load file** link.

1. In the **Open** file dialog that appears, navigate to the **api.json** file you downloaded.

1. Select the **api.json** file.

1. Click the **Open** button.

1. Back on the **Edit template** blade, click the **Save** button to persist the template.

1. Back on the **Custom deployment** blade, follow the process the create the resources.

1. Wait for the deployment to complete before you proceed to the next task.

    > **Note**: Deployment from source control can take up to 10 minutes.

#### Task 2: Validate API App

1. In the hub menu in the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **AADesignLab0701-RG**.

1. On the **AADesignLab0701-RG** blade, click the entry representing the newly created App Service API app.

1. On the API app blade, under **Settings**, click **Configuration**.

1. On the Application Settings blade, scroll down to the **Application settings** section and perform the following tasks:

    - Set the value of the **CosmosDB:AuthorizationKey** setting to the value of the **PRIMARY KEY** setting of the **Cosmos DB** account you created earlier in this lab.

    - Update the value of the **CosmosDB:EndpointUrl** setting to the value of the **URI** setting of the **Cosmos DB** instance you created earlier in this lab.

    - Click the **Save** button at the top of the pane (if prompted, click **Continue**).

1. On the left-side of the API app blade, click **Overview**.

1. Click the **Restart** button at the top of the blade and, when prompted to confirm, click **Yes**.

1. Click the **Browse** button at the top of the blade. This will open a new browser tab displaying the **Swagger UI** homepage.

    > **Note**: If you click the **Browse** button before the API app has fully restarted, you may not be able to follow the remaining steps in this task. If this happens, refresh your browser until the API app is running again.

1. On the **Swagger UI** homepage, click **GET/Documents**.

1. Click the **Try it out!** button.

1. Review the results of the request (the results should include 2 items).

1. Back on the **Swagger UI** homepage, click **POST/Populate**.

1. In the **Parameters** section, in the **Value** field for the **options** parameter, paste in the following JSON content:

    ```json
    {
        "quantity": 50
    }
    ```

1. In the **Response Messages** section, click the **Try it out!** button.

1. Review the results of the request (the results should include 50 items).

1. Back on the **Swagger UI** homepage, click **GET/Documents**.

1. Locate the **Response Messages** section. Click the **Try it out!** button.

1. Review the results of the request (the results should include 52 items).

1. Close the new browser tab and return to the browser tab displaying the Azure portal.

> **Review**: In this exercise, you created a new API App that uses the .NET Core DocumentDB SDK to connect to Azure Cosmos DB collection and manage its documents.



## Exercise 3: Remove lab resources

#### Task 1: Delete the resource group

1. In the hub menu in the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **AADesignLab0701-RG**.

1. On the **AADesignLab0701-RG** blade, click **Delete resource group**.

1. In the **Are you sure you want to delete "AADesignLab0701-RG"?** pane, in the **TYPE THE RESOURCE GROUP NAME** text box, type **AADesignLab0701-RG** and click **Delete**.


> **Review**: In this exercise, you removed the resources used in this lab.