# Create a Standard Load Balancer with Azure CLI
## Introduction
Your company has an application that is overloaded. Whenever there's a spike in traffic, the application is unresponsive. You must implement a load balancer to save the application from crashing and improve response times.

## Solution
Log in to the Azure Portal using the credentials provided on the lab page.

## Create Virtual Machines
### Start Cloud Shell
1. Click the Cloud Shell icon (**>_**) in the upper right.
2. Select **Bash**.
3. Click **Show advanced settings**.
4. In the Azure portal, click **All resources** to see which region your resources are located in.
5. In the Cloud Shell section, change Cloud Shell region to the one your resources are in.
6. For *Storage account*, select **Use existing**. If you have trouble, choose to create a new storage account with a unique name.
7. For *File share*, select **Create new** and give it a name of "fileshare".
8. Click **Create storage**.
## Set Resource Variables in Cloud Shell
1. In the Azure portal, click the listed resource group name.

2. Copy it to your clipboard.

3. In Cloud Shell, set the **RG** variable, replacing **<RESOURCE_GROUP_NAME>** with the name you just copied:
    ```
    RG="<RESOURCE_GROUP_NAME>"
    # or run the below command
    RG=$(az group list --query [].name --output tsv)
    ```
4. Set the **LOC** variable, replacing **<REGION>** with the one your resources are in (e.g., **eastus**):
    ```
    LOC="<REGION>"
    ```
5. In the Azure portal, scroll in the resources list to find your network security group name (it will look something like **nsg1-fhbmu**) and copy its name.

6. In Cloud Shell, set the **NSG** variable, replacing **<NETWORK_SECURITY_GROUP_NAME>** with the name of your network security group:
    ```
    NSG="<NETWORK_SECURITY_GROUP_NAME>"
    # or run below command
    NSG=$(az network nsg list --query [].name --output tsv)
    ```
7. In the Azure portal, scroll in the resources list to find your VNet (it will look something like **lab-VNet1**).

8. Click it, and copy its name.

9. In Cloud Shell, set the **VNET** variable, replacing **<VNET_NAME>** with the name of your VNet:
    ```
    VNET="<VNET_NAME>"
    # or run the below command
    VNET=$(az network vnet list --query [].name --output tsv)
    ```
10. In the Azure portal, on the VNet page, click **Subnets** in the left-hand menu. We should see a subnet called **default**.

11. In Cloud Shell, set the **SNET** variable:
    ```
    SNET="default"
    ```
## Create Network Interfaces for VMs
1. Create the network interface for the first VM:
    ```
    az network nic create \
    --resource-group $RG \
    --location $LOC \
    --name myNicVM1 \
    --vnet-name $VNET \
    --subnet $SNET \
    --network-security-group $NSG
    ```
It will take a minute or so for the command to complete.

2. Create the network interface for the second VM:
    ```
    az network nic create \
    --resource-group $RG \
    --location $LOC \
    --name myNicVM2 \
    --vnet-name $VNET \
    --subnet $SNET \
    --network-security-group $NSG
    ```
It will take a minute or so for the command to complete.

## Create VMs
1. In order to install packages on VMs during deployment, we'll create a **cloud-init** file:
    ```
    code cloud-init.txt
    ```
2. Enter the following into the file:
    ```
    #cloud-config
    package_upgrade: true
    packages:
    - nginx
    - nodejs
    - npm
    write_files:
    - owner: www-data:www-data
    - path: /etc/nginx/sites-available/default
        content: |
        server {
            listen 80;
            location / {
            proxy_pass http://localhost:3000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection keep-alive;
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            }
        }
    - owner: azureuser:azureuser
    - path: /home/azureuser/myapp/index.js
        content: |
        var express = require('express')
        var app = express()
        var os = require('os');
        app.get('/', function (req, res) {
            res.send('Hello World from host ' + os.hostname() + '!')
        })
        app.listen(3000, function () {
            console.log('Hello world app listening on port 3000!')
        })
    runcmd:
    - service nginx restart
    - cd "/home/azureuser/myapp"
    - npm init
    - npm install express -y
    - nodejs index.js
    ```
3. Click the three-dots icon in the upper right corner of the file, and select **Save**.

4. Click the three-dots icon in the upper right corner of the file, and select **Close Editor**.

5. Create the first VM:
    ```
    az vm create \
    --resource-group $RG \
    --location $LOC \
    --name myVM1 \
    --nics myNicVM1 \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --zone 1 \
    --no-wait
    ```
6. Create the second VM:
    ```
    az vm create \
    --resource-group $RG \
    --location $LOC \
    --name myVM2 \
    --nics myNicVM2 \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --custom-data cloud-init.txt \
    --zone 2 \
    --no-wait
    ```
## Create a Load Balancer
1. Create the load balancer's public IP address:
    ```
    az network public-ip create \
    --resource-group $RG \
    --location $LOC \
    --name myPublicIP \
    --sku Standard
    ```
2. Create the load balancer:
    ```
    az network lb create \
    --resource-group $RG \
    --location $LOC \
    --name myLoadBalancer \
    --sku Standard \
    --public-ip-address myPublicIP \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool
    ```
3. Add a health probe to the load balancer:
    ```
    az network lb probe create \
    --resource-group $RG \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
    ```
4. Create the load balancing rules:
    ```
    az network lb rule create \
    --resource-group $RG \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --disable-outbound-snat true
    ```
5. Add the first VM to the load balancer pool:
    ```
    az network nic ip-config address-pool add \
    --address-pool myBackEndPool \
    --ip-config-name ipconfig1 \
    --nic-name myNicVM1 \
    --resource-group $RG \
    --lb-name myLoadBalancer
    ```
6. Add the second VM to the load balancer pool:
    ```
    az network nic ip-config address-pool add \
    --address-pool myBackEndPool \
    --ip-config-name ipconfig1 \
    --nic-name myNicVM2 \
    --resource-group $RG \
    --lb-name myLoadBalancer
    ```
7. In the Azure portal, navigate to the VM provided with the lab.

8. Click **Networking** in the left-hand menu.

9. Click on the network interface.

10. Copy its name.

11. In Cloud Shell, set the **NIC** variable, replacing **<NIC_NAME>** with the name you just copied:
    ```
    NIC="<NIC_NAME>"
    ```
12. Add the lab-provided VM to the load balancer pool:
    ```
    az network nic ip-config address-pool add \
    --address-pool myBackEndPool \
    --ip-config-name ipconfig1 \
    --nic-name $NIC \
    --resource-group $RG \
    --lb-name myLoadBalancer
    ```
13. Get the public IP of the load balancer:
    ```
    az network public-ip show \
    --resource-group $RG \
    --name myPublicIP \
    --query [ipAddress] \
    --output tsv
    ```
14. Copy the IP address in the output.

15. Enter the IP address in a new browser tab. We should see a "Hello World" message from one of the VMs.

16. Refresh multiple times to see the other VM names. (You may need to close and reopen the browser to see the others.)