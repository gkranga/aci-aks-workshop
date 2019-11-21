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
1. Install the kubectl
    1. download the binaries `curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl`
    1. `chmod +x ./kubectl`
    1. `sudo mv ./kubectl /usr/local/bin/kubectl`
    1. run the command `kubectl version` verify that the command is working.

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

In this section, you will need to run the `az` commands on the ubuntu VM

1. Create a new Resource Group on Azure portal for ACR service. You can also create the resource with the command by running the command below on the Ubuntu VM 

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
1. Run this command on the Ubunutu VM: 
`az container create --resource-group <<aci resource group name as abve>> --name <<name of your chioce>> --image <<acrname>>azurecr.io/sumnode:v1 --cpu 1 --memory 1 --registry-password “<<password>>" --ip-address public --ports 80`
1. The out put of the command takes a while for the ACI to get provisioned and you will see the public IP for the ACI.
1. Access the application with the browser: `http://<<Public IP address of the ACI>>/?a=10&b=20`

Question time again
1. whats the value of the ACI and ACR?

Enterprise Security Capabilty:

1. Create Replica of ACR on Azure portal. Whats the value of this? Can yuo image a new way of DR?
1. Check Security Center for image vulnerabilities. How is the code vulnerability detected in container image? (To do this you must enable Security Center for standard tier.  Click security centre, click Coverage in the left blade and click n upgrade to catch the vulerabilities in the image. 

# Lets start with Kubernetes

Our Goal is to create the architecture as below:

![Architecture](https://github.com/gkranga/teamcgbb/blob/master/images/DiagramForContainerization.jpg)

We've found out from the previous sessions that containers independently run are not productin ready. We will find out how Kubernetes can solve this problem. Kubernetes is a complex distributed OS. Let's set this up Azure. Fllow the steps sequentially

1. Create a resource group of any name in the East US region.
1. From the ubuntu VM run the commands below
    1. `az aks create -g testrangaaks -n rangaakscluster --generate-ssh-keys --node-count 1`
    1. wait till the cluster gets created. 
    
        **Questin Time**
        1. What is a cluster?
        **Cntainers are Developer Friendly but how about operations?**
        1. Lets re-visit the problem of IP address of the container? how will yuo reach teh container if the server goes down?
        1. How to interface to handle the dynamic routing?
        1. What is the HA for container?
        1. If container instances are stateless then how to provude load balancing? How to make Existing load balancers compatible?
        1. Enterprise systems need external storage systems to be connected with containers. How?
        1. How does Firewall work with containers?
      1.    `az aks get-credentials --name <<name of the cluster from above>> --resource-group <<name of the resource group>>`
      1. run the command `kubectl get nodes`. If you get the output you are all set.
    1. Go to Azure portal and explore resuorce groups. Notice another resource group with name MC_* has been created. explore the architecture. Explain K8S architecture. 
    
 ## Lets Run a container on this cluster ##
 
 1. Run the command `kubectl run nginx --image=nginx --restart=Never`
 1. `kubectl get pod` (POD is nothing but a container in Kubernetes.
 1. `kubectl describe po nginx`
 1. whats the IP address of the POD?
 1. How do we reach the container?
 1. Run the command `kubectl port-forward nginx 8000:80`(dont close the terminal session)
 1. `curl http://127.0.0.1:8000` check that nginx page is loading. 
 
     **Question Time**
    1. Explain the networking? 
    1. What is the IP range? 
    1. What's the proxy?
    1. What's the K8S API Architecture?
    1. where did the nginx pod come from?
    
  ## Lets run our sumnode container on Kubernetes ##
  1. `kubectl delete po nginx`
  1. `kubectl run sumnode --image=gkranga/sumnode:v1 --restart=Never --port=80`
  1. `kubectl port-forward sumnode 8000:80`
  1. `curl "http://127.0.0.1:8000/?a=10&b=40"`
  1. did you get the result?
  
## Working with Kubernetes objects ##
  In real development situation you cannot be running the `kubectl run` always. We will need to work with API objects of kubernetes. What would we do?
  1. `kubectl run sumnode --image=gkranga/sumnode:v1-restart=Never --port=80 --dry-run -o yaml > sumnode_pod.yaml `
  1. `cat sumnode_pod.yaml`
  1. Explain the objects of the yaml file. 
  1. `kubectl delete po sumnode`
  1. `kubectl create -f ./sumnode_pod.yaml`
  1. Validate if the pod is running
  
  ## How do I get rid of the proxy business?
  
  1. `kubectl run sumnode --image=gkranga/sumnode:v1 --restart=Never --port=80 --expose`
  1. Observe the output carefully. Service is also created.
  1. `kubectl get svc` notice the service? Get the External-IP address 
  
  


 
 
        
    
    




