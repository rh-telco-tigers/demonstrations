<h2 id="900100">900.100</h2>
<h3 id="900100usecase">Use Case(s):</h3> 

- Install Red Hat OpenShift Logging for in-cluster logging (not related to remote forwarding)
- Considerations in this use case are for a 3 node compact cluster

<h3 id="900100overview">Tests(s):</h3>

- Setting up OpenShift Elasticsearch Operator
- Setting up [Red Hat OpenShift Logging](https://docs.openshift.com/container-platform/4.10/logging/cluster-logging.html#cluster-logging-about_cluster-logging)
- Configuring the Logging Subsystem for OpenShift

---
|  Operator(s)  |  Sub-Components | Details     |
| ------------- | --------------- | ----------- |
| [Elasticsearch Operator](https://catalog.redhat.com/software/operators/detail/5f32f067651c4c0bcecf1bfe) | N/A | Elasticsearch is maintained in collaboration with Elastic and Red Hat |
| [Logging Subsystem](https://docs.openshift.com/container-platform/4.10/logging/cluster-logging.html) | N/A | The Logging Subsystem Operator maintained by Red Hat |
---

<h4 id="900100deploy1"><b>Step 1: Deploy MetalLB</b></h4>

OpenShift uses what's called a "Logging Subsystem" as a method for providing logs to external resources. With Self-Service in mind, the logging subsystem allows individual teams to leverage both corporate logging standards, as well as their own separate logging solutions for unique purposes. A common use case would be where platform Operations, SRE, or SecOps teams need platform and general project logs, but individual development teams would need application-specific detailed logs.

The Logging Subsystem is a collection of operators. We will install them in the following steps:

1. Using the left-hand administration panel, navigate to "Operators" > "Operator Hub"
2. Search for "OpenShift Elasticsearch Operator", click on it and click on "Install"
  **IMPORTANT:** *You must select the following for a successful deployment of the Logging Subsystem*
   - Update channel: "Stable"
   - Installation mode: "All namespaces on the cluster (default)"
   - Installed Namespace: `openshift-operators-redhat`
   - Select: "Enable Operator recommended cluster monitoring on this Namespace"
   - Update Approval: "Automatic"
3. Click on "Install", and wait until this is complete.
4. Once that is complete, go back into "Operator Hub"
5. Search for "Red Hat OpenShift Logging", click on it and click on "Install"
  **IMPORTANT:** *You must select the following for a successful deployment of the Logging Subsystem*
   - Update channel: "Stable"
   - Installation mode: "A specific namespace on the cluster"
   - Operator recommended Namespace: `openshift-logging`
   - Select: "Enable Operator recommended cluster monitoring on this Namespace"
   - Update Approval: "Automatic"
6. Create an OpenShift-Logging instance with the following steps:
   - Navigate to "Administration" on the left-hand administration panel, and then "Custom Resource Definitions"
   - Search for "Cluster Logging", and click on it
   - Go to the "Instances" tab, and then select "Create ClusterLogging"
7. Replace what is in the YAML form with what is provided below. It's ***extremely important*** to modify both the `logStore.elasticsearch.storage.storageClassName` and `logStore.elasticsearch.storage.size` to match your environment. If ODF is being used, then the `storageClassName` will be: `ocs- storagecluster-ceph-rbd`.
   ```
   apiVersion: "logging.openshift.io/v1"
   kind: "ClusterLogging"
   metadata:
     name: "instance" 
     namespace: "openshift-logging"
   spec:
     managementState: "Managed"  
     logStore:
       type: "elasticsearch"  
       retentionPolicy: 
         application:
           maxAge: 1d
         infra:
           maxAge: 7d
         audit:
           maxAge: 7d
       elasticsearch:
         nodeCount: 3 
         storage:
           storageClassName: "ocs-storagecluster-ceph-rbd" 
           size: 200G
         resources: 
             limits:
               memory: "16Gi"
             requests:
               memory: "16Gi"
         proxy: 
           resources:
             limits:
               memory: 256Mi
             requests:
               memory: 256Mi
         redundancyPolicy: "SingleRedundancy"
     visualization:
       type: "kibana"  
       kibana:
         replicas: 1
     collection:
       logs:
         type: "fluentd"  
         fluentd: {}
   ```
   **NOTE:** To view the status of the rollout, in the webUI on the left-hand administration pane go to "Workloads", "Pods" and make sure the dropdown at the top of the page is selected for "openshift-logging". Here you can see the status of the workload rollout. 
8. Once the installation has completed, using the webUI navigate to "Network" > "Routes" and search for `kibana`. Click on the "Location" and "Allow selected permissions".
9. When asked to create an index pattern, you will want to create two:
   - `app*`
   - `infra*`
10. Once you've created the first pattern, click on "Next Step"
11. Use `@timestamp` as the "Time Filter field name" and click on "Create index pattern" (repeat the same steps 9. and 10. with the second index pattern)
12. Now you are free to use the "Discover" tab (on the left-hand side) to view log data from the OpenShift cluster

You can continue reading through the [Post-installation Tasks](https://docs.openshift.com/container-platform/4.10/logging/cluster-logging-deploying.html#post-installation-tasks-2) to determine what logs you want to view (you will notice both "infra" and "application" indexes have already been generated, once you are logged into Kibana).