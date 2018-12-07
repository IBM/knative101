## Create IBM Cloud Account and Get Cluster

If you already have `ibmcloud` installed with the `ibmcloud cs` plugin, you can skip these steps.

### Installing the IBM Cloud developer tools

1. Download and install the `ibmcloud` command line tool:
    https://console.bluemix.net/docs/cli/index.html#overview

1. Install the `cs` (container-service) plugin:
    ```bash
    ibmcloud plugin install container-service -r Bluemix
    ```
1. Authorize `ibmcloud`:
    ```bash
    ibmcloud login
    ```

### Create a standard cluster
This lab requires a standard (paid) cluster. Create a new standard cluster from the [IBM Cloud UI](https://console.bluemix.net/containers-kubernetes/catalog/cluster/create).
1. To ensure the cluster is large enough to host all the Knative and Istio
components, the recommended configuration for a cluster is:
  - Kubernetes version 1.10 or later
  - 4 vCPU nodes with 16GB memory (`b2c.4x16`)

2. It is required to select the worker zone, as well as to create a unique cluster name for your cluster.

3. Click `Create Cluster`.

4. Wait while your cluster is fully deployed. Repeat this command until the state of the cluster is `normal`.

    ```
    ibmcloud cs clusters | grep $CLUSTER_NAME
    ```

### Set context for kubectl
Set the context for your cluster in your CLI. Every time you log in to the CLI to work with the cluster, you must run this command to set a path to the cluster's configuration file as a session variable. The Kubernetes CLI uses this variable to find a local configuration file and certificates that are necessary to connect with the cluster in IBM Cloud.

1. Download the configuration file and certificates for your cluster using the `cluster-config` command.

    ```shell
    ibmcloud cs cluster-config <your_cluster_name>
    ```

2. Copy and paste the output command from the previous step to set the `KUBECONFIG` environment variable and configure your CLI to run `kubectl` commands against your cluster.

    Example:
    ```shell
    export KUBECONFIG=/Users...
    ```

3. Validate access to your cluster, by viewing nodes in the cluster.

    ```shell
    kubectl get node
    ```
