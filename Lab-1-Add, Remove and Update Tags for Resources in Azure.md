# Add, Remove and Update Tags for Resources in Azure
## Introduction
In the scenario for this hands-on lab, the finance department has reached out to you. They are requesting additional taxonomy information on a recent Azure bill, including who created the resources, which department budget should be used for the resources, and if the resources are necessary for running business critical systems.

If there are any non-essential business systems, they ask that you signify that in some way.

## Solution
Log in to the Azure Portal.

- In the Azure Portal, click the Cloud Shell icon (>_ ) in the upper right.
- Select **PowerShell**.
- Click **Show advanced settings**.
- Use the same region as the lab provided storage account and the existing resource group. Create a new Storage account (use a globally unique name).
- For File share, select **Create new** and give it a name of "fileshare".
- Click **Create storage**.
- Note: If you have trouble, try using the existing storage account).
## Add Tags to the Resource Group
1. In Azure Cloud Shell, switch to `bash`:

2. List the resource groups:
    ```
    az group list
    ```
3. Copy the group `name`.

4. Update the resource group tags:
    ```
    az group update --resource-group "<RESOURCE_GROUP_NAME>" --tags "Environment=Production" "Dept=IT" "CreatedBy=<YourName>"
    ```
## Remove Tags for VM and Mark for Deletion
1. In the Cloud Shell, list the existing virtual machines:
    ```
    az vm list --query '[].{name:name, resourceGroup:resourceGroup, tags:tags}' -o json
    ```
2. Remove the existing tags from the VM:
    ```
    az vm update -g "<RESOURCE_GROUP_NAME>" -n webvm1 --remove tags.defaultExperience
    ```
3. Mark the VM for deletion
    ```
    az vm update -g "<RESOURCE_GROUP_NAME>" -n webvm1 --set tags.MarkForDeletion=Yes
    ```
## Change Tags for the Virtual Network
1. In the Cloud Shell, list the virtual networks:
    ```
    az network vnet list --query '[].{name:name, resourceGroup:resourceGroup, tags:tags}' -o json
    ```
2. Overwrite the existing tags:
    ```
    az resource tag --tags "Dept=IT" "Environment=Production" "CreatedBy=<YourName>" --resource-group "<RESOURCE_GROUP_NAME>" -n "vnet1" --resource-type "Microsoft.Network/virtualNetworks"
    ```