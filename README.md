# aci-aks-workshop


1. Create a resource group named “jumbox” in south india region
2. Create a Ubuntu 18.04 LTS VM on Azure Portal (Note:the password off the root password) in the above RG.
3. Note down the public IP address of the VM.
4. Open putty, configure the public IP address and the password to connect to the SSH console of the VM.
5. Install Docker Daemon:
    1. sudo apt-get update
    2. sudo apt install docker.io
    3. sudo systemctl start docker
    4. sudo systemctl enable docker
    5. sudo docker —version (This should show the version properly)

Lets Create a container and run it on the VM:
1. sudo docker pull gkranga/sumnode:v1
2. sudo docker run -p 80:80 -tid gkranga/sumnode:v1
3. Explore the container:
    1. Sudo docker ps
    2. Sudo docker exec -ti <<container ID>> sh
4. Open the browser and hit the URL: http://<<VM IP>>/?a=10&b=20
5. 
Quiz Questions:

1. What is the IP address of the container?
2. Login into container: sudo docker exec -ti <<container ID>> sh and try to store a file. Can you store a file inside the container?
3. Kill the container: sudo docker kill <<containerID>>
4. Run the container  again sudo docker run -p 80:80 -tid gkranga/sumnode:v1
5. Login to container and check if the file exists? Explain why?
6. Where is the node process of the container running?
Create ACR on azure portal:
1. Create Resource Group
2. Run the commando on desktop: az group create --name rangaaci --location eastus
3. Setting ACR
    1. az acr create --resource-group <<resuorce group name>> --name <<acrname>> --sku Basic admin-enabled true
    2. sudo az acr login --name <<acrname>>
    3. az acr show --name <<acrname>> --query loginServer --output table
    4. 
4. Sudo docker tag gkranga.azurecr.io/sumnode:v1 <<acrname>>.azurecr.io/sumnode:v1
5. Sudo docker push <<acrname>>azurecr.io/sumnode:v1
6. az acr repository list --name <<acrname>> --output table
7. az acr credential show --name <<acrname>> --query "passwords[0].value"
Note down the password <<password>>
8. Create a new resource group for act
az container create --resource-group <<aci resource group name>>i --name aci-nodesum-app --image <<acrname>>azurecr.io/sumnode:v1 --cpu 1 --memory 1 --registry-password “<<password>>" --ip-address public --ports 80


Whats the Value of ACI?

9. Create Replica of ACR
10. Check Security Center for image vulnerabilities
11. Enable Security Center for standard tier: security centre—>Coverage (under policy & compliance) 





