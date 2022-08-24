# Snapshot an Azure VM Disk using PowerShell
## Introduction
Your company has a legal requirement to keep all backups for a period of time to ensure data can be restored if needed. Your manager has asked you to take a snapshot of a disk in Azure, and make sure that you can access it.

Use PowerShell to take a snapshot of the VM disk and send it to a storage account.

## Solution
Log in to the Azure Portal using the credentials provided on the lab instructions page.

* In the Azure Portal, click the **Cloud Shell** icon (>_ ) in the upper right.
* Select **PowerShell**.
* Click **Show advanced settings**.
* Use the existing storage account.
* For File share, select **Create new** and give it a name of "fileshare".
* Click **Create storage**.
## Stop the Virtual Machine
1. In the Azure Portal, click **All resources** and copy the name of the pre-provisioned resource group.

2. In the Cloud Shell, create a new variable:
    ```
    $rg = "<RESOURCE_GROUP_NAME>"
    # or the the below command if you have one resource group
    $rg = (Get-AzResourceGroup).ResourceGroupName
    ```
3. In the Azure Portal, copy the name of the VM disk and create a new variable:
    ```
    $diskname = "<VIRTUAL_MACHINE_DISK_NAME>"
    # if you have one disk run the below command
    $diskanme = (Get-AzDisk).Name
    ```
4. Create a variable for `sasExpiryDuration`:
    ```
    $sasExpiryDuration = "3600"
    ```
5. In the Azure Portal, copy the storage account name, and create a new variable:
    ```
    $storageAccountName = "<STORAGE_ACCOUNT_NAME>"
    # or run the below command
    $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $rg).StorageAccountName
    ```
6. In the Azure Portal, copy the storage account key for `key1`, and create a new variable:
    ```
    $storageAccountKey = "<KEY1_STORAGE_ACCOUNT_KEY>"
    # or run the below command
    $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $rg -AccountName $storageAccountName) | Where {$_.KeyName -eq "Key1"}
    ```
7. Create additional variables:
    ```
    $storageContainerName = "container1"
    $destinationVHDFileName = "disk1.vhd"
    $useAzCopy = 1
    $vmName = "winVM"
    ```
8. Stop the VM:
    ```
    Stop-AzVM -ResourceGroupName $rg -Name $vmName
    ```
## Take a Snapshot of the Disk
1. Once the VM has stopped, grant access to the disk:
    ```
    $sas = Grant-AzDiskAccess -ResourceGroupName $rg -DiskName $diskName -DurationInSecond $sasExpiryDuration -Access Read
    ```
2. Create an Azure Storage context:
    ```
    $destinationContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
    ```
## Copy the Snapshot to the Container
1. Using AzCopy, send the snapshot to `container1`:
    ```
    if($useAzCopy -eq 1)
    {
    $containerSASURI = New-AzStorageContainerSASToken -Context $destinationContext -ExpiryTime(get-date).AddSeconds($sasExpiryDuration) -FullUri -Name $storageContainerName -Permission rw
    azcopy copy $sas.AccessSAS $containerSASURI

    }else{

    Start-AzStorageBlobCopy -AbsoluteUri $sas.AccessSAS -DestContainer $storageContainerName -DestContext $destinationContext -DestBlob $destinationVHDFileName
    }
    ```