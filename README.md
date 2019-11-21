# aci-aks-workshop


1. Create a resource group named `<give a custom name>` in Eastus region
1. Create a Ubuntu 18.04 LTS VM on Azure Portal (Note the password of the root user) in the above resource group.
1. Note down the public IP address of the VM.
1. Open putty, configure the public IP address and the password to connect to the SSH console of the VM.
1. Install Docker Daemon:
    1. `sudo apt-get update`
    1. `sudo apt install docker.io`
    1. `sudo systemctl start docker`
    1. `sudo systemctl enable docker`
    1. `sudo docker —version` (This should show the version properly)
1. Install Azure CLI by running the below command
    `curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash`

## Lets Create a container and run it on the VM: 
1. Connect to the VM on SSH Terminal using putty. If Putty does not work use the cloud shell by clicking:
![by Clicking](https://github.com/gkranga/aci-aks-workshop/blob/master/Cloud%20Shell.png) on the Azure Portal. [Refer here for details](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) 

To connect from the cloud shell to the VM run the command `ssh username@public_ip_address_of_the_vm`.

Now execute the following commands to run the sample container called sumnode (This is the Container I have built and published on Dockerhub. This runs a node.js application which takes 2 URL paramaters a and b as numbers and gives the HTTP reponse with the values of a and b added

1. `sudo docker pull gkranga/sumnode:v1`
1. `sudo docker run -p 80:80 -tid gkranga/sumnode:v1`
1. Explore the container with the commands
    1. `sudo docker ps` (copy the container ID from the output)
    1. `sudo docker exec -ti <<container ID>> sh`(Note that you are inside the container, explore the processes, network address, etc.. and exit out of the container with the command `exit` )
1. On the Azure Portal click on the VM, On the left blade click Networking, click `add inbound port rule` button and appropriately add the rule to allow port 80.
1. Open the browser and hit the URL `http://<<Public IP address of the VM>>/?a=10&b=20`

If you successfully get the result. You have successfully  run the container on a server.

Quiz Questions:

1. What is the IP address of the container?
1. Login into container and run the command: `sudo docker exec -ti <<container ID>> sh` and try to create a file with the command `touch <<any filename you want >>` 
1. Can you store a file inside the container?
1. Kill the container with the command `sudo docker kill <<containerID>>`
1. Run the container  again with the command `sudo docker run -p 80:80 -tid gkranga/sumnode:v1`
1. Login to container and check if the file youo had created exists? Explain why?
1. Where is the node process of the container running?
1. The container that you have run on the server, where did it originally come from? 
1. If yuo want to make some changes to the conatiner image how will you do?
1. If you want to distribute this container to some one else how?


## Create ACR on azure portal:

1. Create a new Resource Group on Azure portal for ACR service. You can als create the resource with the command by running the command below on the Ubuntu VM 

`az group create --name <<any name of yoour choiced>> --location eastus`

1. Provision the ACR with thefollowing commands on Ubuntu VM:
    1. `az acr create --resource-group <<resuorce group name from the above>> --name <<any acrname of your choice>> --sku Basic --admin-enabled true`
    1. On the Azure portal, click on the ACR that has been created, click on `Access Keys` on the laft blade and note down the username and password (any one of the 2 passwords)
    1. `sudo az acr login --name <<acrname>> -u <<username from the above>>`
    1. `az acr repository list --name <<acrname from the above>> --output table` (This list should be empty)
    
    ##Push the sumnode container to ACR 
    
    Run the fllowing commands in sequence on the Ubuntu VM:
    
1. `sudo docker tag gkranga/sumnode:v1 <<acrname>>.azurecr.io/sumnode:v1`
1. `sudo docker push <<acrname you have given>>azurecr.io/sumnode:v1`

Explore the repository on the ACR and validare you can see the container image.

  
  ## Lets run the same container on ACI 
  
Now will run the container not a server but on a PaaS service called Azure Container instance (ACI). The image for this container is pulled from the ACR. Following are the steps:

1.. Create a new resource group for ACI
2. Run this command on the Ubunutu VM: 
`az container create --resource-group <<aci resource group name as abve>> --name <<name of your chioce>> --image <<acrname>>azurecr.io/sumnode:v1 --cpu 1 --memory 1 --registry-password “<<password>>" --ip-address public --ports 80`


Question time again
1. whats the value of the ACI and ACR?

Enterprise Security Capabilty:

1. Create Replica of ACR on Azure portal. Whats the value of this? Can yuo image a new way of DR?
1. Check Security Center for image vulnerabilities. How is the code vulnerability detected in container image? (To do this you must enable Security Center for standard tier.  Click security centre, click Coverage in the left blade and click n upgrade to catch the vulerabilities in the image. 

