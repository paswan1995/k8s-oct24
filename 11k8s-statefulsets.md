# Lens

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
     * = `db-0.headless service ` it will reaching to the write endpoint . 
     *  to reach each pod specifically in a StatefulSet for your MySQL cluster, you can use their unique names (mysql-0, mysql-1, etc.) along with the service name for direct access. This setup enables clear and reliable communication with each pod based on its role (write or read).

*  Generally, whenever your organization wants to run databases inside a Kubernetes cluster, we use StatefulSets.
   
    * StatefulSets expand on the Idea of PV. If you make the replicas 5 then will get 5 PVs. we get 5 pods and if you delete 1 pod you will get other pod with exactly same name. we can get same consistence name so we can access our Applications and then even a Pod is getting deleted & recreating we have a PV so we don't loosing the data this is the exact fit for running the statefulSet Application that's the whole reason we have a stateless or a statefulSet. when you want run a Application Generally we will go with Deployment & when you want run Databases, It is better i will go is the statefulSets.  

    *  StatefulSets expand on the idea of Persistent Volumes (PVs). If you set the replicas to 5, you will get 5 PVs and 5 pods. If you delete 1 pod, a new pod will be created with exactly the same name. This consistency in naming allows us to access our applications reliably. Even if a pod is deleted and recreated, we have a PV associated with it, so we don't lose any data. This makes StatefulSets an ideal fit for running stateful applications. Generally, when you want to run an application, we use Deployments; however, when running databases, it is better to use StatefulSets.

# Scheduling pods on Nodes

 * In Kubernetes, scheduling pods on nodes is essential to ensure resources are utilized efficiently and application requirements are met. Here are key techniques and strategies used for scheduling pods:

   * 1. Node Selector
          * The simplest way to control which nodes a pod is scheduled on.
          * Specify nodeSelector in the pod specification to define node labels the pod requires.
          * Only nodes with the specified labels will be considered for the pod.

```
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
 
```
yaml
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
 
```
yaml
affinity:
podAffinity:
requiredDuringSchedulingIgnoredDuringExecution:
- labelSelector:
matchLabels:
app: frontend
topologyKey: "kubernetes.io/hostname"
```

4. Taints and Tolerations
   * __Taints:__ Applied to nodes to repel certain pods, setting constraints on which nodes can host which pods.
   
   * __Tolerations:__ Allow pods to “tolerate” taints, enabling them to be scheduled on tainted nodes.
   
   * Useful for workloads requiring dedicated nodes or isolation.

```
yaml
tolerations:
- key: "example-key"
operator: "Equal"
value: "example-value"
effect: "NoSchedule"

```

5. __Resource Requests and Limits__
     * Ensure nodes have the required CPU and memory resources for each pod.
     * Pods with high resource requests will only be scheduled on nodes with sufficient capacity, preventing overloading.

```
yaml
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

```
yaml
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

# Administrative Activity: Upgrading K8s clusters

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

  



