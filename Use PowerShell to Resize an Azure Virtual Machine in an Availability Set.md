# Use PowerShell to Resize an Azure Virtual Machine in an Availability Set
## Introduction
Your company's finance department explains that the cost of Azure virtual machines (VMs) is over budget. You must find an easy solution to decrease the cost of Azure VMs in your subscription. Using only PowerShell, how do you identify the VMs with low CPU utilization and change their size?

## Solution
Log in to the Azure portal using the credentials provided on the lab page.

## Start Cloud Shell
1. Click the Cloud Shell icon (>_) in the upper right.
2. Select **PowerShell**.
3. Click **Show advanced settings**.
4. Set the cloud shell region to the same location as the resource group
5. For *Storage account*, select **Create new** and give it a globally unique name (e.g., "cloudshell" with a series of numbers at the end).
6. For *File share*, select **Create new** and give it a name of "fileshare".
7. Click **Create storage**.
## Get the VM Size and CPU Metrics
1. Find the current size of the VMs:
    ```
    Get-AzVM
    ```
2. Get the VM subscription name:
    ```
    Get-AzSubscription
    ```
3. Check VM CPU metrics, replacing `<ID>` with the VM subscription name and `<RESOURCE_GROUP_NAME> `with the group name you just copied:
    ```
    az monitor metrics list --resource /subscriptions/<ID>/resourceGroups/<RESOURCE_GROUP_NAME>/providers/Microsoft.Compute/virtualMachines/labVM0
    ```
## Resize VMs in the Availability Set
1. Assign the value to the $vm variable:
    ```
    $vm = Get-AzVM -ResourceGroupName <RESOURCE_GROUP_NAME> -VMName labVM0
    ```
2. Perform the PowerShell commands to resize the VM(s):
    ```
    $vm.HardwareProfile.VmSize = "Standard_B1s"
    Update-AzVM -VM $vm -ResourceGroupName <RESOURCE_GROUP_NAME>
    ```
3. To view results of the change, enter ```Get-AzVM```. We should see `VMSize` for `labVM0` changed to `Standard_B1s`.