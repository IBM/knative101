## Setup: Install Knative on Your Cluster
Knative extends the capabilites of Kubernetes to add components for deploying, running, and managing serverless applications on Kubernetes. The managed Knative add-on for IBM Cloud utilizes Istio under the covers for its networking layer. If you want to learn more about Kubernetes or Istio, you can check out the labs [Kube101](https://github.com/IBM/kube101/tree/master/workshop) and
[Istio101](https://github.com/IBM/istio101/tree/master/workshop).

When you install Knative on IKS, it will install Istio for you
automatically.

### Install Knative

1. To install Knative, first make sure you have the latest Kubernetes
   plugin:

    ```
    ibmcloud plugin update
    ```

2. Next ask for Knative to be installed on your cluster. The `-y` flag enables all dependencies for the Knative add-on:

    ```
    ibmcloud ks cluster addon enable knative --cluster $MYCLUSTER -y
    ```

3. The install process may take a minute or two. To know when it's done you
   can run two commands - first see if the Istio and Knative namespaces
   are there:

   ```
   kubectl get namespace
   ```

   and you should see something like:

   ```
    NAME                 STATUS   AGE
    default            Active   21h
    ibm-cert-store     Active   21h
    ibm-operators      Active   21h
    ibm-system         Active   21h
    istio-system       Active   8s
    knative-eventing   Active   7s
    knative-serving    Active   7s
    knative-sources    Active   7s
    kube-node-lease    Active   21h
    kube-public        Active   21h
    kube-system        Active   21h
    tekton-pipelines   Active   7s
   ```

   Notice the `istio-system` namespace, and the `knative-...` namespaces.

   Once the namespaces are there, check to see if all of the Istio and
   Knative pods are in the `Running` or `Completed` state:

   ```
   kubectl get pods --namespace istio-system
   kubectl get pods --namespace knative-serving
   kubectl get pods --namespace tekton-pipelines
   ```
   

   Example Ouput:

   ```
   NAME                          READY   STATUS    RESTARTS   AGE
   activator-df78cb6f9-jpvs7     2/2     Running   0          38s
   activator-df78cb6f9-nhzhf     2/2     Running   0          37s
   activator-df78cb6f9-qjg8w     2/2     Running   0          37s
   autoscaler-6fccb66768-m4f2q   2/2     Running   0          37s
   controller-56cf5965f5-8pwcg   1/1     Running   0          35s
   webhook-5dcbf967cd-lxzmk      1/1     Running   0          35s
   ```

   If all of the pods shown are in a `Running` or `Completed` state then you should be all set.

Continue on to [exercise 3](../exercise-3/README.md).