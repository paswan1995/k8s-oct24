# Managed Kubernetes clusters

*  Cloud providers offer managed  kubernetes clusters

* Cloud providers manage the control plane i.e. we need not install k8s and the control plane will be a black box

# why should i used manage k8s over self hosted k8s
 
  * Choosing between managed Kubernetes and self-hosted Kubernetes involves weighing several factors related to operational efficiency, control, and resource allocation. Here are the key reasons to consider using a managed Kubernetes service:
  
  __1. Reduced Operational Burden__:  Managed Kubernetes services, such as Amazon EKS, Azure AKS, and Google GKE, handle the complexities of managing the Kubernetes control plane. This includes tasks like patching, scaling, and ensuring high availability of the control components, which can significantly reduce the operational burden on your engineering teams1

  __2. Simplified Setup and Maintenance:__  Setting up a self-hosted Kubernetes environment requires substantial expertise in configuring and managing various components, including networking and storage solutions. Managed services provide a more streamlined setup process, allowing teams to focus on deploying applications rather than managing infrastructure3

  __3. Enhanced Security Features:__  Managed Kubernetes providers typically implement additional security measures and configurations that may be challenging to replicate in a self-hosted environment. This includes automated updates for security patches, compliance with industry standards, and built-in monitoring tools5

  __4. Scalability and Flexibility:__  Managed services offer dynamic scaling capabilities that allow organizations to automatically adjust resources based on workload demands. This flexibility is particularly beneficial for businesses experiencing fluctuating traffic patterns1

  __5. Access to Advanced Features:__  Many managed Kubernetes offerings come with integrated tools for monitoring, logging, and performance management that can enhance operational efficiency. These tools are often pre-configured to work seamlessly with the managed environment4

  __6. Focus on Development Over Operations:__  By leveraging a managed service, engineering teams can allocate more time to developing applications rather than dealing with infrastructure management. This aligns with the DevOps philosophy of "more Dev, less Ops," enabling faster deployment cycles and innovation1

  __7. Cost Considerations:__  While self-hosting might seem cost-effective initially, the hidden costs related to maintenance, staffing specialized personnel, and managing outages can accumulate quickly. Managed services often provide predictable pricing models that can help in budgeting effectively2

  __Conclusion:__  In summary, opting for a managed Kubernetes service is generally advisable for organizations lacking extensive Kubernetes expertise or those looking to minimize operational overhead while maximizing security and scalability. Conversely, self-hosted Kubernetes may be more suitable for teams seeking complete control over their infrastructure configuration and management processes.     

* 
![preview](images/46.png)

 * Lower-level detail relevant to creating or administering a Kubernetes cluster.
 * refer: https://kubernetes.io/docs/concepts/cluster-administration/

# Azure Kubernetes Services (AKS)

 * AKS is managed k8s offered by Azure 
    * Refer Here: https://learn.microsoft.com/en-us/azure/aks/what-is-aks

 * Lets setup AKS Cluster
 * Pre-reqs: 
    * Azure CLI is installed and configured
        * refer here for steps :https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli 
        * install azure cli then configure it
        * Azure cli user data
```
#!/bin/bash
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -sLS https://packages.microsoft.com/keys/microsoft.asc |
  gpg --dearmor | sudo tee /etc/apt/keyrings/microsoft.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/microsoft.gpg
AZ_DIST=$(lsb_release -cs)
echo "Types: deb
URIs: https://packages.microsoft.com/repos/azure-cli/
Suites: ${AZ_DIST}
Components: main
Architectures: $(dpkg --print-architecture)
Signed-by: /etc/apt/keyrings/microsoft.gpg" | sudo tee /etc/apt/sources.list.d/azure-cli.sources
sudo apt-get update
sudo apt-get install azure-cli -y
```

* Commands to create AKS cluster from bash (linux/mac)

```
# variables 
export MY_RESOURCE_GROUP_NAME="myAKSResourceGroup"
export REGION="eastus"
export MY_AKS_CLUSTER_NAME="myAKSCluster"
export MY_DNS_LABEL="mydnslabel"
# Create a resource group
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
az aks create --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME --node-count 1 --node-vm-size "Standard_B2ms" --generate-ssh-keys
# install kubectl
az aks install-cli 
# get kube config
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME
```

* Commands to create AKS cluster from powershell using Azure CLI only in windows powershell is accepting `$` symbol in front of commands

```
$MY_RESOURCE_GROUP_NAME="myAKSResourceGroup"
$REGION="eastus"
$MY_AKS_CLUSTER_NAME="myAKSCluster"
$MY_DNS_LABEL="mydnslabel"
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
az aks create --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME --node-count 1 --node-vm-size "Standard_B2ms" --generate-ssh-keys
az aks install-cli 
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME

```
* az aks create search on google
* refer here: https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create
* if you want to remove kube.config file `rm ~/.kube/config`  whatever is present in my system

* `kubectl version` in powershell 
* # firstly login to the cloud shell through azure portal
# execute below commands in cloud shell
# defining enviornment variables
```
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="sampleAks$RANDOM_ID"
export REGION="eastus2"
export MY_AKS_CLUSTER_NAME="myAKSCluster$RANDOM_ID"
export MY_DNS_LABEL="mydnslabel$RANDOM_ID"
``` 
# creating resource group
`az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION`
 
# creating AKS Cluster with new key
# here you can increase the nodes and size of the vm
`az aks create --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $MY_AKS_CLUSTER_NAME \
    --node-count 2 \
    --generate-ssh-keys --node-vm-size 'Standard_B4ms'`
 
# conneting to the cluster

# getting credentials
`az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME`
 
# for getting credentials on powershell execute below commandon
`echo "az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME"`
# run the output on power shell
 
# to delete the cluster delete the resource group
`az group delete --name $MY_RESOURCE_GROUP_NAME --yes --no-wait`
 
# run below command on git bash to get auto-complete every time
source <(kubectl completion bash)

![preview](images/47.png)
![preview](images/48.png)
![preview](images/49.png)
![preview](images/50.png)
![preview](images/51.png)
![preview](images/52.png)
![preview](images/53.png)
![preview](images/54.png)

* __Lets deploy spc using replicaset and a service of type load balancer__

```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: spc-rs
  labels:
    app: spc
    version: v5.55
spec:
  minReadySeconds: 5
  replicas: 2
  selector:
    matchLabels:
      app: spc-build
  template:
    metadata:
      labels:
        app: spc-build 
    spec:
      containers:
        - name: spc-c
          image: redfiree/spc-project
          ports:
            - containerPort: 8080
          resources:
            limits:
              memory: 500Mi
              cpu: 500m  
---
apiVersion: v1
kind: Service
metadata:
  name: aks-spc-svc
spec:
  type: LoadBalancer
  selector:
    app: spc-build
  ports:
    - port: 80
      targetPort: 8080

---
```
![preview](images/55.png)
![preview](images/56.png)

* refer: https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/introduce-eshoponcontainers-reference-app

# Library Application

 * Architecture
  ![preview](images/57.png)

 *  Technical stack
   
   * users-service: REST API 
     * python
     * fast api
   
   * user-db: 
     * postgres
     
   * books-service: REST API 
     * python
     * fast api
     
   * user-db: 
     * postgres
     
   * library-frontend: 
     * react js
        
   * To run books database 
     * image: postgres:15-alpine
     * environmental variables: 
       * POSTGRES_USER: user
       * POSTGRES_PASSWORD: password
       * POSTGRES_DB: booksdb
     
     * port: 5432 
     
     *  To run books service
      
       * image: shaikkhajaibrahim/libbookssvc:1.0
       * environmental variables:    
           * DATABASE_URL: “postgresql://:@:5432/“
           * SECRET_KEY: ‘YtDEVWnL35aAIP-5yxeLjAZ49R920-mMNDfwPyWULu63HFsYzo0f-LO2InxC8eu428k’
       * port: 8000     
       
     * To run user database
       
       * image: postgres:15-alpine
       * environmental variables:     
           * POSTGRES_USER: user
           * POSTGRES_PASSWORD: password
           * POSTGRES_DB: usersdb

       * port: 5432 
       
       * To run users servics: 
         * image: shaikkhajaibrahim/libuserssvc:1.0
         * environmental variables: 
           * DATABASE_URL: “postgresql://:@:5432/“
           * SECRET_KEY: ‘YtDEVWnL35aAIP-5yxeLjAZ49R920-mMNDfwPyWULu63HFsYzo0f-LO2InxC8eu428k’
```YML
1. to run users service 
  image: 
  environmental variables: 
    * DATABASE_URL: "postgresql://
	<POSTGRES_USER>:<POSTGRES_PASSWORD>@<name or ip of db container>:5432/<POSTGRES_DB>"
           |             |                        |                 |        |
           |             |                        |                 |        |
      user name       postgres password       service name        port    postgre db name
                                               
```
       * port: 8000    

       * To run library webstore 
        * image: shaikkhajaibrahim/libwebstore:1.0
        * environmental variables 
           * REACT_APP_BACKEND_API_URL: http://:8000/api/v1
           * REACT_APP_BOOKS_API_URL: http://:8000/api/v1/books
           * REACT_APP_USERS_API_URL: http://:8000/api/v1/users
       
       * port: 3000
       
       ![preview](images/58.png)


# First Version of Library:


  * Lets create a users database replicaset

```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: booksdb-rs
spec:
  minReadySeconds: 5
  replicas: 1
  selector:
    matchLabels:
      db: books
  template:
    metadata:
      labels:
       db: books 
    spec: 
      containers: 
        - name: booksdb-c
          image: postgres:15-alpine
          env:
            - name: POSTGRES_USER
              value: user
            - name: POSTGRES_PASSWORD
              value: password
            - name: POSTGRES_DB
              value: booksdb 
          ports:
            - name: booksdb-p
              containerPort: 5432
          resources:
            requests: 
              memory: 128M
              cpu: 125m
            limits:
              memory: 250M 
              cpu: 250m 

---
apiVersion: v1 
kind: Service
metadata: 
  name: booksdb-svc
spec: 
  type: ClusterIP
  selector: 
    db: books
  ports:
    - port: 5432
      targetPort: 5432
```
  * Lets create a books database replicaset 

```yml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: usersdb-rs
spec:
  minReadySeconds: 5
  replicas: 1
  selector:
    matchLabels:
      db: users
  template:
    metadata:
      labels:
       db: users 
    spec: 
      containers: 
        - name: usersdb-c
          image: postgres:15-alpine
          env:
            - name: POSTGRES_USER
              value: user
            - name: POSTGRES_PASSWORD
              value: password
            - name: POSTGRES_DB
              value: usersdb 
          ports:
            - name: usersdb-p
              containerPort: 5432
          resources:
            requests: 
              memory: 128M
              cpu: 125m
            limits:
              memory: 250M 
              cpu: 250m 

---
apiVersion: v1 
kind: Service
metadata: 
  name: usersdb-svc
spec: 
  type: ClusterIP
  selector: 
    db: users
  ports:
    - port: 5432
      targetPort: 5432
```
* Now lets create rs for users service and books service Refer Here for changes

```yml
---
apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: users-svc-rs
spec:
  minReadySeconds: 5
  replicas: 1
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users 
    spec:
      containers:
        - name: users-svc-c
          image: redfiree/libuserssvc:1.0
          env:
            - name: DATABASE_URL
              value:  "postgresql://user:password@usersdb-svc:5432/usersdb"
            - name: SECRET_KEY
              value: "YtDEVWnL35aAIP-5yxeLjAZ49R920-mMNDfwPyWULu63HFsYzo0f-LO2InxC8eu428k"
          ports:
            - containerPort: 8000
          resources:
            requests:
              memory: 128M
              cpu: 125m 
            limits:
              memory: 256M 
              cpu: 250m 

---
apiVersion: v1
kind: Service
metadata:
  name: users-svc
spec:
  type: ClusterIP
  selector:
    app: users
  ports:
  - port: 8000
    targetPort: 8000
```
* bookssvc

```yml
---
apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: books-svc-rs
spec:
  minReadySeconds: 5
  replicas: 1
  selector:
    matchLabels:
      app: books
  template:
    metadata:
      labels:
        app: books 
    spec:
      containers:
        - name: books-svc-c
          image: redfiree/libbookssvc:1.0
          env:
            - name: DATABASE_URL
              value:  "postgresql://user:password@booksdb-svc:5432/booksdb"
            - name: SECRET_KEY
              value: "YtDEVWnL35aAIP-5yxeLjAZ49R920-mMNDfwPyWULu63HFsYzo0f-LO2InxC8eu428k"
          ports:
            - containerPort: 8000
          resources:
            requests:
              memory: 128M
              cpu: 125m 
            limits:
              memory: 256M 
              cpu: 250m 

---
apiVersion: v1
kind: Service
metadata:
  name: books-svc
spec:
  type: ClusterIP
  selector:
    app: books
  ports:
  - port: 8000
    targetPort: 8000
```
* Libstore app yaml manifest file  for frontend 

```yml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: libstore-rs
  labels:
    app: libstore
spec:
  minReadySeconds: 5
  replicas: 1
  selector: 
    matchLabels:
      app: libstore 
  template:
    metadata:
      labels:
        app: libstore
    spec:
      containers:
        - name: libstore-c
          image: redfiree/libstore:1.0
          env:
            - name: REACT_APP_BACKEND_API_URL
              value: http://users-svc:8000/api/v1
            - name: REACT_APP_BOOKS_API_URL
              value: http://books-svc:8000/api/v1/books 
            - name: REACT_APP_USERS_API_URL
              value: http://users-svc:8000/api/v1/users
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: 128M 
              cpu: 250m
            limits:
              memory: 500M 
              cpu: 500m  

---
apiVersion: v1 
kind: Service
metadata:
  name: libstore-svc 
spec:
  type: NodePort  #while using cloud you should give Loadbalancer instead of NodePort
  selector: 
    app: libstore
  ports: 
    - port: 80
      targetPort: 3000 
```

* always create first database then services.  Always follow this rule `least dependent to most dependent` 
![preview](images/62.png)
*  Lets add rs and service for library store
![preview](images/63.png)
![preview](images/64.png)
![preview](images/65.png)
*  To create a admin user run this from any container 
```
curl -X 'POST' \
  'http://users-svc:8000/api/v1/users/' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "username": "qtdevops",
  "email": "admin@admin.com",
  "user_type": "administrator",
  "password": "admin@123"
}'
```

 # namespace exercise
   
   * create a qa namespace (use manifest)

```yaml
---
apiVersion: v1
kind: Namespace
metadata: 
  name: qa
```

![preview](images/60.png)

   * create a nginx deployment with 4 replicas in qa namespace

```yaml

---
apiVersion: v1
kind: Namespace
metadata: 
  name: qa

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: qa
  name: nginx-deploy
spec:
  minReadySeconds: 5
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers: 
        - name: nginx-c
          image: nginx
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: 64Mi
              cpu: 250m 
            limits:
              memory: 528Mi
              cpu: 500m
---
```
![preview](images/66.png)