# Build and Run a Container using Azure ACR Tasks
## Introduction
Your manager asks you to run a container, but you don't have Docker installed on your desktop. You've recently learned about Azure ACR Tasks built into Cloud Shell and decide to give it a try. Your goal is to create a new container registry and use ACR Tasks to build, push and run the container in Azure.

## Solution
Log in to the Azure portal using the credentials provided on the lab page.

## Start Cloud Shell
1. Click the Cloud Shell icon (**>_**) in the upper right.
2. Select **Bash**.
3. Click **Show advanced settings**.
4. Change *Cloud Shell Region* to the same region as the existing resource group.
5. For *Storage account*, select **Use existing**.
6. For *File share*, select **Create new** and give it a name of "fileshare".
7. Click **Create storage**.
## Create a New Container Registry
1. In the Azure portal, click the listed resource group name.

2. Copy it to your clipboard.

3. Create a new container registry, replacing **<RESOURCE_GROUP_NAME>** with the name you just copied:
    ```
    az acr create --resource-group <RESOURCE_GROUP_NAME> \
    --name acrbuildcontainer11 --sku Basic --admin-enabled true
    ```
4. Change the directory to **clouddrive**:
    ```
    cd /home/cloud/clouddrive
    ```
5. Create a Dockerfile:
    ```
    echo "FROM hello-world" > Dockerfile
    ```
## Build an Image and Push to ACR
Build and push the image to Azure Container Registry using the newly created Dockerfile. Give this step extra time to complete:
```
az acr build --image sample/hello-world:v1 --registry acrbuildcontainer11 \
--file Dockerfile .
```
In the Azure portal, refresh All resources to view the newly created container registry with the **sample/hello-world** repository.

## Run the New Container
Run the container:
```
az acr run --registry acrbuildcontainer11 --cmd '$Registry/sample/hello-world:v1' /dev/null
```