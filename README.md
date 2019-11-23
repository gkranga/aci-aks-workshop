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
    1. download the binaries 
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/ `curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
    ```
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

**Quiz Questions**

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
    
    ## Push the sumnode container to ACR 
    
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

**Question time again**

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
    
        **Question Time**
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
  1. `kubectl run sumnode --image=gkranga/sumnode:v1 --restart=Never --port=80 --dry-run -o yaml > sumnode_pod.yaml `
  1. `cat sumnode_pod.yaml`
  1. Explain the objects of the yaml file. 
  1. `kubectl delete po sumnode`
  1. `kubectl create -f ./sumnode_pod.yaml`
  1. Validate if the pod is running
  
  ## How do I get rid of the proxy business ##
  
  1. `kubectl run sumnode --image=gkranga/sumnode:v1 --restart=Never --port=80 --expose`
  1. Observe the output carefully. Service is also created.
  1. `kubectl get svc` notice the service? Get the External-IP address 
  1. `kubectl get svc sumnoode-service --export=true`. If the External-IP is pending, the provisining is not complete. Wait till it gets over.
  1. lets get the service object `kubectl get svc sumnode-service --export=true -o yaml > sumnode-service.yaml`
  1. `cat sumnode-service.yaml`
  
   **Question Time**
   
    1. How is service architecture working?
    1. How did the network and IP addresss dangling get solved?
    1. Check the resource group MC_* and notice any changes in the resources? explain what is added?
    1. Types of Services? Cluster, Loadbalancer, NodePort.
    1. Can you think of few more operational challenges of running apps on K8S? Monitoring, alerting, logging?
    1. Deployment patterns: 
    1. release managemnt Canary releases, Dark Launching, AB Testing, Progressive exposure deployment, Blue green deployments
    
   **Architecturally what is Kubernetes. Relate to Object Oriented Programming.**
    
    
**Take a coffee Break **
    
  ## Use ACR image in K8S ##
  
  1. `kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>`
  1. `kubectl get secret`
  1. `kubectl describe secret regcred`
  1. `kubectl delete pod sumnode`
  1. `kubectl delete service sumnode-service`
  1. Edit the sumnode_pod.yaml to change the image and also inlude imagepull secrets. The YAML File should like as below:
   
   ```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sumnode
  name: sumnode
spec:
  containers:
  - image: rangaacr.azurecr.io/sumnode:v1
    imagePullPolicy: IfNotPresent
    name: sumnode
    ports:
    - containerPort: 80
    resources: {}
  imagePullSecrets:
  - name: regcred
  dnsPolicy: ClusterFirst
  restartPolicy: Never
   ```
  1. Validate that the container is running.
  
  ## Lets create a 3 tier architecture app on Container ##
  
1. Create a resource group 
1. Create a mySql database:
    Perform the following steps to create and configure Azure Database for MySQL used by this app:

### Step A –

Create an Azure Database for MySQL server by using the Azure portal
Please refer to [https://docs.microsoft.com/en-us/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal](https://docs.microsoft.com/en-us/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal) for creation of database
** Please ensure that database is created in the same resource group and location as the app. This consideration reduces database latency across geo

### Step B -

We need to setup firewall settings for the app to access the database. In the Azure Database for MySQL settings blade Under the Firewall rules heading, select the blank text box in the Rule Name column to begin creating the firewall rule. 
For this Quick start, let's allow all IP addresses into the server by filling in boxes in each column with the following values:

Rule name -  AllowAllIps

Start IP -   0.0.0.0

End IP   -   255.255.255.255

Alternatively: you can choose to allow only the 10.x.x.x IP range of the kubernetes vnet.

Allowing all IP addresses is not secure. This example is provided for simplicity, but in a real-world scenario, you need to know the precise IP address ranges to add for your applications and users. 
Please refer to [https://docs.microsoft.com/en-us/azure/mysql/howto-manage-firewall-using-portal](https://docs.microsoft.com/en-us/azure/mysql/howto-manage-firewall-using-portal)


### Step C – 

Connect to the database created in step 1 using cloudshell and run the command 
`mysql -h <<hostname>>` -p
refer to the screen shot below
![](https://github.com/gkranga/teamcgbb/blob/master/images/mysql%20password%20prompt.png)

enter the password you had provided when provisioning the mysql DB.

![](https://github.com/gkranga/teamcgbb/blob/master/images/mysql%20login%20screen.png)

after the successful login you will get the mysql prompt

run the query `show databases;` and verify that you are getting responses.

![](https://github.com/gkranga/teamcgbb/blob/master/images/showdatabases.png)


### Step D -
Now run the command `create database` _`<<name of your choice>>`_ to create a database of your choice of name 

## Create Kubernetes secret for DB connectin string ##
On the console of the VM from step1 run the following commands:

From the previous step, copy the following data points:
1. The JDBC URL for connecting to the database. Copy the text as shown in the highlighted section of the text. In the coped string, replace the _`{your_database}`_ with the name you gave in the previous step while creating the Database. 
![](https://github.com/gkranga/teamcgbb/blob/master/images/JDBC.png)
1. username of the DB
![](https://github.com/gkranga/teamcgbb/blob/master/images/DB%20Username.png)
1. mysql password that you have configured.

Now run the following commands to get the base 64 encoding of the above 3 data (url, username & password)

`echo -n <<jdbc url edited from the above>> | base64`

`echo -n <<username from the above>> | base64`

`echo -n <<mysql password>> | base64`

copy paste the output of the above in any file/note (ensure that you don't add any extra spaces while copying)

Perform the following steps

1. Edit the file secret.yml using your choice of editor.

against the 3 keys "username", "password" and "url" paste the corresponding values from above and save the file. Ensure that you don't have any extra spaces at the end or beginning of the text being pasted. The edited file would look as give below
![](https://github.com/gkranga/teamcgbb/blob/master/images/secret%20yaml.png)

 run the following command

`kubectl create -f ./secret.yml`

validate that the secrets are properly configured by running the command 

`kubectl get secret mysecret -o yaml`
![](https://github.com/gkranga/teamcgbb/blob/master/images/secret%20validate.png)



Now you are all set to run the application and connect to the database. 

1. Create the ecommerce POD
1. Create a YAML file with the following code
```
apiVersion: v1
kind: Pod
metadata:
  name: ecommerce-pod
  labels: 
    app: ecommerce
spec:
      containers:
        - name: ecommerce-container
          image: index.docker.io/gkranga/ecommerce:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
          - name: SECRET_USERNAME
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: username
          - name: SECRET_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: password
          - name: URL
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: url
      restartPolicy: Never
```

## Expose the Service for the app ##

1. create the service by creating the ecommerce-svc.yaml with the following content:
```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: ecommerce-service
  name: ecommerce-service
spec:
  ports:
    # The port that this service should serve on.
    - port: 8080
      targetPort: 8080
  # Label keys and values that must match in order to receive traffic for this service.
  selector:
    app: ecommerce
  type: LoadBalancer
```
1. `kubectl apply -f ./ecommerce-svc.yaml`
It will take a few minutes for the external IP to be assigned for the service. Until then the external IP status will be shown as Pending.

After this, open the browser and go to the URL http://external IP:8080/

This should load the tomcat screen

The URL http://external IP:8080/OnlineShoppingPortal will load the application page

![](https://github.com/gkranga/teamcgbb/blob/master/images/App%20Login%20page.png)


## Create DB Tables to start the complete ecommerce application on Kubernetes ##
 
 Run the following queries on the mysql prompt on cloud shell (Refer to previous steps)
```
insert into role values(1,'USER');

insert into role values(2,'ADMIN');

insert into products values('S001','Java',150,20);

insert into products values('S002','Java8',160,1);

insert into products values('S003','node',170,0);

insert into products values('S004','angular',180,2);

insert into products values('S005','jsp',190,5);

insert into products values('S006','spring',120,10);

insert into products values('S007','hibernate',110,19);

```
Now go the application URL page, register user and experience the ecommerce flows and also validate the data updates in the DB.
        
    
## Virtual Nodes ##
1. Delete the resuorce groups previously created for K8S
2. Follow the instructions [here](https://docs.microsoft.com/en-us/azure/aks/virtual-nodes-portal) 


## Introducing Azure DevOps ##

