# Create Web App from Docker Container in Azure
## Introduction
You have a custom application that your company wants you to deploy. This application is optimized for containers, and you have been given a Dockerfile. The company mandates that all container images be stored in Azure Container Registry. Find the best solution to host your containerized application in Azure App Service.

## Solution
Log in to the Azure portal using the credentials provided on the lab page.

## Start Cloud Shell
1. Click the Cloud Shell icon (**>_**) in the upper right.
2. Select **Bash**.
3. Click **Show advanced settings**.
4. Change *Cloud Shell Region* to the same location as your lab provided resource group.
5. For *Storage account*, select **Use existing**.
6. For *File share*, select **Create new** and give it a name of "fileshare".
7. Click **Create storage**.
## Set Resource Variables in Cloud Shell
1. Set the **ACR** (Azure Container Registry) variable. You can use any unique meaningful name for the "<ACR_NAME>":
    ```
    ACR="<ACR_NAME>"
    ```
2. In the Azure portal, click the listed resource group name.

3. Copy it to your clipboard.

4. In Cloud Shell, set the **RG** variable, replacing **<RESOURCE_GROUP_NAME>** with the name you just copied:
    ```
    RG="<RESOURCE_GROUP_NAME>"
    # or run the below command
    RG=$(az group list --query [].name --output tsv)
    ```
## Create a New Container Registry
1. Create a new Azure Container Registry:
    ```
    az acr create --resource-group $RG --name $ACR --sku Basic --admin-enabled true
    ```
## Build an Image and Push to ACR
1. Change the directory to **clouddrive**:
    ```
    cd /home/cloud/clouddrive
    ```
2. Clone the **js-docker** branch of the [github repository](https://github.com/linuxacademy/content-AZ-104-Microsoft-Azure-Administrator/tree/js-docker):
    ```
    git clone --branch js-docker https://github.com/linuxacademy/content-AZ-104-Microsoft-Azure-Administrator.git ./js-docker
    ```
3. Change the directory to **js-docker**:
    ```
    cd js-docker/
    ```
4. Build and push the image to Azure Container Registry using ACR Tasks and the Dockerfile provided (<IMAGE_NAME> can be anything you like e.g. js-docker:v1):
    ```
    az acr build --image <IMAGE_NAME> --registry $ACR --file Dockerfile .
    ```
In the Azure portal, refresh All resources to view the newly created container registry with the **js-docker** repository and image.

## Create and Deploy a New Web App
1. In the Azure portal, open the upper left menu and click **App Services**.

2. In the upper left, click **+Create**.

3. On the *Create Web App* page, set the following values:

    * *Resource Group*: Existing resource group
    * *Name*: Unique name
    * *Publish*: Docker Container
4. In *Sku* and *size*, click **Change size**.

5. Click See additional options and select the Standard S1 size.

6. Click **Apply**.

7. Click **Next: Docker**.

8. In *Image Source*, select **Azure Container Registry**, make sure your image is selected and click **Review + create**.

Once the web app has successfully been deployed, click **Go to resource**, copy the provided URL, and load the URL in the browser to see your new web app.