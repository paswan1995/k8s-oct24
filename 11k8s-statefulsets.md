# Lens
26/Oct/2024

* refer: https://k8slens.dev/



# StatefulSet

* refer: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

* Stateful sets create multiple pods with a predicatable name in a sequential order (0 to n) = `[if i want to  created statefulset of  database with 5 replicas so what will be the pod name and  order of creation is = database-0,1,2,3]`
* Rolling updates will be performed in a reverse order (n-0) [1 to n]  
* Each Pod will raise a PVC to get a PV
* Generally we will have a headless service to access specific pod
* Statefulsets are widely used to create database clusters and any  application with state.
* Since we acces pod using headless service the libarary application will have a DATABASE URI changed to `postgresql://user:password@booksdb-0.booksdb-svc:5432/booksdb`
*  Refer Here for the changes done to move away from replicaset to stateful set for database pods in library application. 
--------------------------------------------------------------------------------------------------------------------

* StatefulSets create a Persistent Volume (PV) for each replica. Therefore, if you specify 4 replicas, it will create 4 PVs; if you specify 10 replicas, it will create 10 PVs.

*  If 3 pods are created in a StatefulSet, they will be named sequentially as web-0, web-1, and web-2. If any of the pods are deleted, they will be recreated with the same previous name. This is one of the primary advantages of using a StatefulSet.

* In Kubernetes, setting the clusterIP field to None in a service definition signifies that the service is a headless service. This means that the service does not have a single IP address assigned to it. Instead of routing traffic through a load balancer or a single IP, a headless service allows direct access to the individual pods associated with it.

* a headless service in Kubernetes is defined by setting clusterIP to None, enabling direct communication with individual pods rather than routing through a single entry point. This feature is essential for applications requiring direct pod-to-pod communication and enhanced control over network traffic.

*  A headless service helps you reach the pods directly. By creating a headless service, you can access a specific pod using the format [pod-name].[service-name].

* Generally, we use services in Kubernetes because they allow us to reach the pods directly. When I access the service, it routes my request to one of the pods, enabling direct communication with any of them.

   *  if we create a StatefulSet, we can specify that we want a 3-pod MySQL cluster. For example, web-0 can serve as my write endpoint, while web-1 can serve as my read endpoint. Alternatively, db-0 can be my write endpoint and db-1 my read endpoint. How do I reach each pod specifically? 
     * = `db-0.headless service` it will reaching to the write endpoint . 
     *  to reach each pod specifically in a StatefulSet for your MySQL cluster, you can use their unique names (mysql-0, mysql-1, etc.) along with the service name for direct access. This setup enables clear and reliable communication with each pod based on its role (write or read).

*  Generally, whenever your organization wants to run databases inside a Kubernetes cluster, we use StatefulSets.
   
    * StatefulSets expand on the Idea of PV. If you make the replicas 5 then will get 5 PVs. we get 5 pods and if you delete 1 pod you will get other pod with exactly same name. we can get same consistence name so we can access our Applications and then even a Pod is getting deleted & recreating we have a PV so we don't loosing the data this is the exact fit for running the statefulSet Application that's the whole reason we have a stateless or a statefulSet. when you want run a Application Generally we will go with Deployment & when you want run Databases, It is better i will go is the statefulSets.  

    *  StatefulSets expand on the idea of Persistent Volumes (PVs). If you set the replicas to 5, you will get 5 PVs and 5 pods. If you delete 1 pod, a new pod will be created with exactly the same name. This consistency in naming allows us to access our applications reliably. Even if a pod is deleted and recreated, we have a PV associated with it, so we don't lose any data. This makes StatefulSets an ideal fit for running stateful applications. Generally, when you want to run an application, we use Deployments; however, when running databases, it is better to use StatefulSets.


*  `Library App v3 is stored in StatefulSet folder in Manifest file folder.`

 
# Scheduling pods on Nodes

 * In Kubernetes, scheduling pods on nodes is essential to ensure resources are utilized efficiently and application requirements are met. Here are key techniques and strategies used for scheduling pods:

   * 1. Node Selector
          * The simplest way to control which nodes a pod is scheduled on.
          * Specify nodeSelector in the pod specification to define node labels the pod requires.
          * Only nodes with the specified labels will be considered for the pod.

```yml
yaml
spec:
nodeSelector:
disktype: ssd
```

2. Node Affinity and Anti-Affinity
   * __Node Affinity:__ A more flexible and expressive method than nodeSelector. Allows specifying rules for preferred and required node selection.
      * Required (hard constraint): The pod will only be scheduled if nodes match.
      * Preferred (soft constraint): The scheduler tries to match but may skip if unavailable.

   * __Anti-Affinity:__ Ensures that certain pods are not placed on the same node or close to others. Useful for  high availability.
 
```yml
affinity:
nodeAffinity:
requiredDuringSchedulingIgnoredDuringExecution:
nodeSelectorTerms:
- matchExpressions:
- key: disktype
operator: In
values:
- ssd

```

3. __Pod Affinity and Anti-Affinity__
     * Controls pod placement based on the presence of other pods.
     
     * __Pod Affinity:__ Specifies pods that should be placed together for performance or resource-sharing needs.
     * __Pod Anti-Affinity:__ Ensures pods are spread out across nodes to prevent resource contention.
 
```yml
affinity:
podAffinity:
requiredDuringSchedulingIgnoredDuringExecution:
- labelSelector:
matchLabels:
app: frontend
topologyKey: "kubernetes.io/hostname"
```

4. **Taints and Tolerations**
   * __Taints:__ Applied to nodes to repel certain pods, setting constraints on which nodes can host which pods.
   
   * __Tolerations:__ Allow pods to “tolerate” taints, enabling them to be scheduled on tainted nodes.
   
   * Useful for workloads requiring dedicated nodes or isolation.

```yml
tolerations:
- key: "example-key"
operator: "Equal"
value: "example-value"
effect: "NoSchedule"

```

5. __Resource Requests and Limits__
     * Ensure nodes have the required CPU and memory resources for each pod.
     * Pods with high resource requests will only be scheduled on nodes with sufficient capacity, preventing overloading.

```yml
resources:
requests:
memory: "64Mi"
cpu: "250m"
limits:
memory: "128Mi"
cpu: "500m"
```

# 6. Priority and Preemption
   * Allows higher-priority pods to preempt lower-priority ones if the cluster is under resource pressure.
   * Priority classes define which pods are most important, ensuring critical applications are scheduled even in tight resource situations.

# 7. Topology Spread Constraints
   * Enables even distribution of pods across nodes in the cluster, reducing single points of failure.
   * Can control pod spread across various zones or availability domains.

```yml
topologySpreadConstraints:
- maxSkew: 1
topologyKey: "zone"
whenUnsatisfiable: DoNotSchedule
labelSelector:
matchLabels:
app: frontend
```
     
# 8. Custom Schedulers

   * Kubernetes allows using custom schedulers for unique workload requirements.
   * Useful for applications needing custom placement strategies, such as data locality.

   * Real-World Examples
      * GPU Workloads: Schedule AI/ML jobs on GPU-enabled nodes while considering GPU type, memory, and CUDA cores.
       
      * Cost Optimization: Use strategies like MostAllocated for better bin-packing and reduced cloud costs.

      * Cloud Bursting: Provision jobs in external clouds if local resources are insufficient.

   * Benefits of Custom Schedulers

      * Tailored workload placement for unique requirements.

      * Enhanced resource utilization and cost efficiency.

      * Support for advanced scheduling strategies not available in the default scheduler.

* Custom schedulers provide flexibility and control, making them essential for complex Kubernetes deployments in industries like AI/ML, finance, healthcare, and more

# Here is a comparison of Cordon and Drain in Kubernetes in tabular form:

Here is a comparison of **Cordon** and **Drain** in Kubernetes in tabular form:

| **Aspect**            | **Cordon**                                                                                  | **Drain**                                                                                           |
|------------------------|---------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| **Purpose**           | Marks a node as **unschedulable**, preventing new pods from being scheduled on it.           | Evicts all pods from a node and marks it as unschedulable, preparing it for maintenance or shutdown. |
| **Effect on Existing Pods** | Existing pods on the node remain running and unaffected.                                     | Existing pods are evicted (moved to other nodes) gracefully.                                        |
| **Command Syntax**    | `kubectl cordon `                                                                 | `kubectl drain  [options]`                                                              |
| **Use Case**          | Used for light maintenance or as a preparatory step before draining a node.                  | Used for major maintenance or when a node needs to be shut down or removed from the cluster.        |
| **DaemonSet Pods**    | Does not affect DaemonSet pods; they continue running.                                       | By default, ignores DaemonSet pods unless `--ignore-daemonsets` is specified.                      |
| **Pod Scheduling**    | Prevents new pods from being scheduled on the node but does not reschedule existing ones.    | Evicts all schedulable pods and ensures they are rescheduled to other nodes in the cluster.         |
| **Graceful Termination** | Not applicable, as existing pods are not evicted.                                             | Ensures graceful termination of pods with respect to their PodDisruptionBudgets and grace periods.  |
| **Common Options**    | No additional options required.                                                             | Options include `--ignore-daemonsets`, `--delete-emptydir-data`, `--force`, and `--grace-period`.   |
| **When to Use**       | - Quick fixes or updates that do not require pod eviction.- As preparation before draining. | - Node shutdown, hardware upgrades, or OS/kernel updates.- Cluster scaling or decommissioning. |

### Summary:
- Use **Cordon** when you want to stop scheduling new pods on a node while keeping existing workloads running.
- Use **Drain** when you need to evict all workloads from a node to perform significant maintenance or shut it down safely.




# Administrative Activity: Upgrading K8s clusters

  **Self hosted** 

   * Always go through release notes to figure out what has changed in the new version
   * backup the etcd cluster and persistent volumes

   * cordon the node
   * drain the node

   * upgrade by executing linux commands
   * uncordon and make it available for scheduling

 **Managed k8s cluster**

   * [EKS](https://aws.amazon.com/eks/)

Here is a detailed comparison of Amazon Elastic Kubernetes Service (EKS), Azure Kubernetes Service (AKS), and Google Kubernetes Engine (GKE) in tabular form:

| **Feature/Aspect**           | **Amazon EKS**                                                                                  | **Azure AKS**                                                                                     | **Google GKE**                                                                                   |
|-------------------------------|-----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| **Year Released**             | 2018                                                                                         | 2017                                                                                            | 2014                                                                                           |
| **Ease of Use**               | Requires manual configuration for some features; integrates well with AWS ecosystem          | User-friendly interface with Azure Portal; seamless integration with Azure services             | Offers Autopilot mode for simplified management; deep integration with Google Cloud services    |
| **Control Plane Cost**        | $0.10 per hour per cluster                                                                    | Free                                                                                            | Free for one zonal cluster; $0.10/hour for multi-zonal or regional clusters                     |
| **Worker Node Management**    | Managed Node Groups simplify node provisioning and scaling                                    | Uses Virtual Machine Scale Sets (VMSS) for high availability and scaling                        | Offers both manual and automated node management; Autopilot mode manages nodes automatically    |
| **Scaling Options**           | Horizontal Pod Autoscaler, Cluster Autoscaler                                                | Horizontal Pod Autoscaler, Cluster Autoscaler                                                   | Horizontal Pod Autoscaler, Cluster Autoscaler, Vertical Pod Autoscaler                         |
| **Max Nodes per Cluster**     | 1,000                                                                                        | 1,000                                                                                           | 5,000                                                                                          |
| **High Availability**         | Control plane is highly available across multiple zones                                       | High availability for worker nodes via VMSS; control plane lacks built-in HA                    | Regional clusters provide high availability for both control plane and worker nodes             |
| **Kubernetes Upgrades**       | Manual upgrades for control plane and worker nodes                                           | Manual upgrades required                                                                        | Automated upgrades enabled by default                                                           |
| **Integration with Cloud Services** | Deep integration with AWS tools like CloudWatch, IAM, and S3                                    | Integrates with Azure AD, Azure Monitor, and Azure DevOps                                        | Integrates with Google Cloud services like Stackdriver Monitoring and Config Connector          |
| **Security Features**         | Supports IAM roles for service accounts; Pod Security Policies                               | Role-based access control (RBAC) integrated with Azure Active Directory                         | Native RBAC; integrates with Google Cloud IAM                                                   |
| **Windows Container Support** | Supported                                                                                    | Supported                                                                                       | Supported                                                                                        |
| **Monitoring Tools**          | AWS CloudWatch Container Insights                                                             | Azure Monitor                                                                                   | Google Cloud Operations Suite (formerly Stackdriver)                                            |
| **Deployment Options**        | Primarily on AWS; supports hybrid setups via Outposts                                        | Primarily on Azure; supports hybrid and multi-cloud deployments via Azure Arc                   | Primarily on Google Cloud; supports hybrid deployments via Anthos                               |
| **Cost Efficiency**           | Higher costs due to control plane charges; discounts available via Savings Plans or Reserved Instances  | Free control plane; costs depend on underlying Azure resources                                  | Competitive pricing for compute and storage; free tier for small clusters                       |
| **Unique Features**           | Bare-metal node support; Elastic Fabric Adapter (EFA) for high-performance networking         | Integration with GitOps workflows using Azure DevOps pipelines                                  | Early access to Kubernetes features due to Google's involvement in Kubernetes development       |

### Key Takeaways:
1. **Amazon EKS**: Best suited for organizations heavily invested in AWS. Offers robust networking features but requires more manual configurations.
2. **Azure AKS**: Ideal for those already using Microsoft Azure services. Provides ease of use through its portal but requires manual upgrades.
3. **Google GKE**: The most mature service, offering advanced features like Autopilot mode and high scalability. Ideal for data-heavy or multi-cloud applications.

The choice between these services depends on your existing cloud environment, workload requirements, and cost considerations.

# [AKS vs EKS](https://www.pluralsight.com/resources/blog/cloud/aks-vs-eks-vs-gke-managed-kubernetes-services-compared)



* To address your query, here are detailed steps for migrating a Kubernetes cluster from version 1.27 to 1.31, backing up etcd and persistent volumes, upgrading AKS and EKS clusters, and upgrading GKE clusters.

---

## **Migrating a Kubernetes Cluster (Self-Managed) from 1.27 to 1.31**

Kubernetes does not support skipping minor versions during upgrades. You must upgrade sequentially: 1.27 → 1.28 → 1.29 → 1.30 → 1.31.

### Steps:
1. **Backup the Cluster**:
   - Backup etcd (see below for details).
   - Backup Persistent Volumes (see below for details).

2. **Upgrade Control Plane**:
   - On the first control plane node:
     ```bash
     sudo apt-mark unhold kubeadm && \
     sudo apt-get update && \
     sudo apt-get install -y kubeadm=1.x.y-* && \
     sudo apt-mark hold kubeadm
     ```
     Replace `x.y` with the target version (e.g., `1.28` for the first step).
   - Verify the upgrade plan:
     ```bash
     sudo kubeadm upgrade plan
     ```
   - Apply the upgrade:
     ```bash
     sudo kubeadm upgrade apply v1.x.y
     ```

3. **Upgrade Worker Nodes**:
   - Upgrade kubelet and kubectl on each worker node:
     ```bash
     sudo apt-mark unhold kubelet kubectl && \
     sudo apt-get update && \
     sudo apt-get install -y kubelet=1.x.y-* kubectl=1.x.y-* && \
     sudo apt-mark hold kubelet kubectl
     ```
   - Restart kubelet:
     ```bash
     sudo systemctl restart kubelet
     ```

4. **Repeat for Subsequent Versions** until reaching v1.31[1][2].

---

## **Backup of etcd Cluster**

### Steps to Backup etcd (Kubeadm):
1. Install `etcdctl` on the control plane node:
   ```bash
   sudo apt install etcd-client
   ```
2. Take a snapshot of etcd:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key \
   snapshot save /opt/etcd.db
   ```
3. Verify the snapshot:
   ```bash
   ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/etcd.db
   ```
4. Store the snapshot securely[3][4].

---

## **Backup Persistent Volumes**

Persistent Volume backups can be automated using CSI VolumeSnapshot or manually using tools like Velero or Commvault.

### Steps (CSI VolumeSnapshot):
1. Create a `VolumeSnapshotClass` object.
2. Use `kubectl` to create snapshots of PVCs:
   ```yaml
   apiVersion: snapshot.storage.k8s.io/v1
   kind: VolumeSnapshot
   metadata:
     name: pvc-snapshot
     namespace: default
   spec:
     volumeSnapshotClassName: 
     source:
       persistentVolumeClaimName: 
   ```
3. Verify snapshots using `kubectl get volumesnapshots`[5][6].

---

## **Backup AKS Cluster**

### Steps:
1. Install Azure Backup extension in the AKS cluster via Azure Portal.
2. Configure backup policies and storage locations.
3. Backups can be scheduled daily or more frequently (e.g., every 4 hours).
4. Persistent volumes are backed up as snapshots stored in a resource group[7].

---

## **Upgrade AKS Cluster from 1.29 to 1.30**

AKS upgrades must follow sequential minor versions.

### Steps:
1. Check available upgrades:
   ```bash
   az aks get-upgrades --resource-group  --name 
   ```
2. Upgrade the cluster:
   ```bash
   az aks upgrade --resource-group  --name  --kubernetes-version 
   ```
3. Monitor the upgrade process[8].

---

## **Upgrade EKS Cluster**

### Steps:
1. Update control plane using `eksctl` or AWS CLI:
   ```bash
   eksctl upgrade cluster --name  --version  --approve
   ```
2. Upgrade worker nodes by creating new node groups with updated versions or manually upgrading node pools.
3. Ensure compatibility between control plane and nodes[9].

---

## **Upgrade GKE Cluster**

### Steps:
#### Control Plane Upgrade:
- Using Google Cloud Console:
  1. Go to **Google Kubernetes Engine**.
  2. Select your cluster and click **Upgrade Available** under **Cluster basics**.
- Using CLI:
  ```bash
  gcloud container clusters upgrade  --master --cluster-version 
  ```

#### Node Pool Upgrade:
- Using Google Cloud Console or CLI, upgrade node pools sequentially to ensure compatibility with the control plane[10].

---

* By following these steps, you can successfully migrate your Kubernetes cluster across versions, ensure backups are in place, and perform upgrades on managed Kubernetes services like AKS, EKS, and GKE effectively while minimizing downtime and risks associated with version mismatches or data loss during upgrades.



 * __Self hosted__
     * Always go through release notes to figure out what has changed in the new version
     * backup the etcd cluster and persistent volumes
     * cordon the node
     * drain the node
     * upgrade by executing linux commands
     * uncordon and make it available for scheduling


# Managed  k8s cluster

 * Follow the cloud documentations
  
# Manifests
   * K8s manifests are static in nature, so every change has to be done in the file and then added to version control
   * Reusability becomes a problem
   * To solve this problem, we have two options
      * Kustomize
      * Helm: (Package manager for kubernetes):

* __NextSteps:__
      * ingress and ingress controller
      * Authentication and Authorization
      * Autoscaling:
           * Horizontal Pod Autoscaling
           * Vertical Pod Autoscaling
           * Node Autoscaling (Managed  K8s Clusters)

# Extra

* Lens = lens is a k8s tool which help us to indentify the issuses, debug the problem easly and this from cri dockerd mirantis.

* aks statefulset examples

```
subPath= data 
how to exec into postgress container?
exec with username with password?
json web token
k8s metrics server 
kubectl top pods 
```

* Here is a comparison of **Stateless (Deployment)** and **StatefulSet** in Kubernetes in tabular form, along with examples:

| **Aspect**              | **Stateless (Deployment)**                                                                                  | **StatefulSet**                                                                                     |
|--------------------------|-------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| **Definition**           | Manages stateless applications, where pods are interchangeable and do not retain persistent state.          | Manages stateful applications, where pods have unique identities and persistent storage.            |
| **Pod Identity**         | Pods are treated as identical and interchangeable.                                                         | Each pod has a unique, predictable identity (e.g., `statefulset-name-0`, `statefulset-name-1`).     |
| **Storage**              | Uses ephemeral storage; data is lost when pods are deleted or restarted.                                   | Uses Persistent Volume Claims (PVCs) to ensure data persists across pod restarts or rescheduling.   |
| **Network Identity**     | Pods have dynamically assigned network identities; no stable hostname or IP address is guaranteed.          | Pods have stable network identities, which remain consistent even if a pod is rescheduled.          |
| **Scaling Behavior**     | Pods can be scaled up or down in any order; no guarantees on startup or shutdown sequence.                  | Pods are scaled in a controlled, ordered manner (e.g., `pod-0` starts before `pod-1`).             |
| **Rolling Updates**      | Updates all pods simultaneously or based on a rolling update strategy without any specific order.           | Updates pods one at a time in a defined order, ensuring application stability during updates.        |
| **Use Cases**            | Ideal for stateless applications like web servers, APIs, or microservices that don’t require persistent data. | Ideal for stateful applications like databases (e.g., MySQL, Cassandra) or distributed systems requiring persistence. |
| **Example YAML**         | Example Deployment:                                                                                        | Example StatefulSet:                                                                                |
|                          | ```yaml                                                                                                    | ```
|                          | apiVersion: apps/v1                                                                                        | apiVersion: apps/v1                                                                                |
|                          | kind: Deployment                                                                                           | kind: StatefulSet                                                                                  |
|                          | metadata:                                                                                                  | metadata:                                                                                          |
|                          |   name: nginx-deployment                                                                                   |   name: nginx-statefulset                                                                          |
|                          | spec:                                                                                                      | spec:                                                                                              |
|                          |   replicas: 3                                                                                             |   replicas: 3                                                                                      |
|                          |   selector:                                                                                               |   serviceName: "nginx"                                                                             |
|                          |     matchLabels:                                                                                          |   selector:                                                                                        |
|                          |       app: nginx                                                                                          |     matchLabels:                                                                                   |
|                          |   template:                                                                                               |       app: nginx                                                                                   |
|                          |     metadata:                                                                                            |   template:                                                                                        |
|                          |       labels:                                                                                            |     metadata:                                                                                      |
|                          |         app: nginx                                                                                        |       labels:                                                                                      |
|                          |     spec:                                                                                                |         app: nginx                                                                                 |
|                          |       containers:                                                                                        |     spec:                                                                                          |
|                          |       - name: nginx                                                                                      |       containers:                                                                                  |
|                          |         image: nginx                                                                                     |       - name: nginx                                                                                |
|                          |         ports:                                                                                           |         image: nginx                                                                               |
|                          |         - containerPort: 80                                                                              |         ports:                                                                                     |
|                          | ```yaml...```



* Here’s a detailed explanation of the topics you mentioned, including scheduling pods on nodes, static manifests, Helm/Kustomize, and working with Helm for specific use cases like MySQL replication and PostgreSQL clusters.

---

## **Scheduling Pods on Nodes**

Kubernetes allows scheduling pods on specific nodes using labels and node selectors. Here's how to achieve this:

### **Node Labeling**
1. **View current labels on nodes**:
   ```bash
   kubectl get nodes --show-labels
   ```
2. **Add or update a label on a node**:
   ```bash
   kubectl label node [node-name] env=qa
   ```
3. **Verify the label**:
   ```bash
   kubectl get nodes --show-labels
   ```

### **Pod Scheduling Using Node Selectors**
To schedule a pod on a specific node labeled `env=qa`, include the following in your Pod manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    env: qa
```

This ensures the pod will only be scheduled on nodes with the label `env=qa`.

---

## **Static Manifests**

Static manifests are YAML files used to define Kubernetes objects (e.g., Pods, Deployments, Services). These files are static in nature, meaning they are manually created and managed.

### Key Points:
- Static manifests are stored locally and applied using `kubectl` commands.
- Example of a static manifest for a Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21.6
        ports:
        - containerPort: 80
```

Apply the manifest using:
```bash
kubectl apply -f deployment.yaml
```

---

## **Helm and Kustomize**

### **Helm**
Helm is a package manager for Kubernetes that simplifies application deployment using charts.

* search given below:  
   * helm mysql replication / helm postgress cluster 
   * Kustomize overlays for production and qa 
   * simple helm chart 
   **Layer 4  knoes only port n ip address**
   * and Layer 7 Load balancer

[Kustomize](https://www.densify.com/kubernetes-tools/kustomize/)




#### **MySQL Replication in Helm**
To deploy MySQL replication using Helm, you can use the Bitnami MySQL Helm chart:

1. Add the Bitnami repository:
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```
2. Install MySQL with replication enabled:
   ```bash
   helm install my-mysql bitnami/mysql --set architecture=replication --set auth.rootPassword=my-root-password --set auth.replicationPassword=my-replication-password
   ```

#### **PostgreSQL Cluster in Helm**
Use the Bitnami PostgreSQL chart to deploy a PostgreSQL cluster:

1. Add the Bitnami repository (if not already added):
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```
2. Install PostgreSQL with replication enabled:
   ```bash
   helm install my-postgresql bitnami/postgresql-ha --set postgresql.password=my-password --set postgresql.replication.password=my-replication-password --set postgresql.replicaCount=2
   ```

#### **Overlays in Helm**
Helm does not natively support overlays like Kustomize, but you can achieve similar functionality by using Helm values files.

1. Create multiple `values.yaml` files for different environments (e.g., `values-dev.yaml`, `values-prod.yaml`).
2. Deploy with specific values file:
   ```bash
   helm install my-app ./my-chart -f values-dev.yaml
   ```

### **Kustomize**
Kustomize allows you to customize Kubernetes objects without modifying the original YAML files.

#### Example of Overlays in Kustomize:
1. Base YAML (`base/deployment.yaml`):
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
     labels:
       app: my-app
   spec:
     replicas: 2
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-app-container
           image: my-app-image:v1.0.0
           ports:
           - containerPort: 80
   ```

2. Overlay for production (`overlays/prod/kustomization.yaml`):
   ```yaml
   resources:
     - ../../base/deployment.yaml

   patchesStrategicMerge:
     - patch.yaml

3. Patch file (`overlays/prod/patch.yaml`):
```
spec

---

