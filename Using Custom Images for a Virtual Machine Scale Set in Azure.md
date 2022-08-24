# Using Custom Images for a Virtual Machine Scale Set in Azure
## Introduction
Your company has a custom Linux VM that they want to redeploy to a virtual machine scale set. You must find an efficient way of redeploying this VM without having to manually install all custom packages and settings.

## Solution
Log in to the Azure portal using the credentials provided on the lab page.

## Start Cloud Shell
1. Click the Cloud Shell icon (`>_`) in the upper right.
2. Select `Bash`.
3. Click `Show advanced settings`.
4. Change *Cloud Shell* Region to **West US**.
5. For *Storage account*, select **Use existing**.
6. For *File share*, select **Create new** and give it a name of "fileshare".
7. Click **Create storage**.
## Set Resource Variables in Cloud Shell
1. In the Azure portal, click the listed resource group name.

2. Copy it to your clipboard.

3. In Cloud Shell, set the `RG` variable, replacing `<RESOURCE_GROUP_NAME>` with the name you just copied:
    ```
    RG="<RESOURCE_GROUP_NAME>"
    # or run the below command
    RG=$(az group list --query [].name --output tsv)
    ```
4. In the Azure portal, click the virtual machine provisioned with this lab.

5. In the left menu, click **Properties**.

6. Scroll down and copy the Resource ID to the clipboard.

7. In Cloud Shell, set the `IMAGE` variable, replacing `<RESOURCE_ID>` with the name you just copied:
```
IMAGE="<RESOURCE_ID>"
# or below command
IMAGE=$(az vm list --query [].id -o tsv)
```
## Create an Image from the VM
1. Create an image gallery (**NOTE**: you will need to add some characters to the end of the gallery name to make it unique):
    ```
    az sig create --resource-group $RG --location westus --gallery-name imageGallery
    ```
2. Create an image definition:
    ```
    az sig image-definition create \
    --resource-group $RG \
    --location westus \
    --gallery-name imageGallery \
    --gallery-image-definition imageDefinition \
    --publisher acg \
    --offer ubuntu \
    --sku Ubuntu-1804 \
    --os-type Linux \
    --os-state specialized \
    --features IsAcceleratedNetworkSupported=True
    ```
3. Create an image version:
    ```
    az sig image-version create \
    --resource-group $RG \
    --location westus \
    --gallery-name imageGallery \
    --gallery-image-definition imageDefinition \
    --gallery-image-version 1.0.0 \
    --target-regions "westus=1" "eastus=1" \
    --managed-image $IMAGE
    ```
## Create a Virtual Machine Scale Set from an Image
1. In the Azure portal, scroll the listed resources and click the new image definition file.

2. In the left menu, click **Properties**.

3. Scroll down and copy the Resource ID to the clipboard.

4. In Cloud Shell, create a VM scale set, replacing `<RESOURCE_ID>` in the `image` variable with the name you just copied:
    ```
    az vmss create \
    --resource-group $RG \
    --name myVmss \
    --image "<RESOURCE_ID>" \
    --specialized \
    --generate-ssh-key \
    --location westus
    ```
5. To confirm results in the Azure portal, scroll the listed resources and click **myVMss** > **Instances**. You should see two VMs running.