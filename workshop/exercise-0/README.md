## Setup: Create IBM Cloud Account and Install Developer Tools

If you already have an IBM Cloud account with the following tools installed, you can skip these steps:

- `ibmcloud` CLI for interacting with IBM Cloud
- `ibmcloud ks` plugin for IBM Kubernetes Service
- `ibmcloud cr` plugin for IBM Container Registry
- `kubectl` CLI for interacting with Kubernetes
- `kn` CLI for interacting with Knative

### Create an IBM Cloud Account
1. Create an account on [IBM Cloud](https://cloud.ibm.com/registration).

### Installing the IBM Cloud developer tools and plugins

1. Download and install the `ibmcloud` [command line tool.](https://cloud.ibm.com/docs/cli/index.html#overview)

1. Install the `ks` (kubernetes-service) plugin:

    ```
    ibmcloud plugin install kubernetes-service
    ```
1. Install the `cr` (container-registry) plugin:

    ```
    ibmcloud plugin install container-registry
    ```
1. Authorize `ibmcloud`:

    ```
    ibmcloud login
    ```

### Install Kubectl
The Kubernetes command-line tool, kubectl, can be used to deploy and manage applications on Kubernetes. Using kubectl, you can inspect cluster resources; create, delete, and update components; look at your new cluster; and bring up example apps.

1. Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)

### Install kn CLI
The Knative command-line tool, kn provides a set of commands to interact with a Knative installation. Let's install kn as well.

1. Install the kn client by following their [instructions for installation](https://github.com/knative/client/blob/master/docs/README.md#installing-kn). You should be able to download the latest binary executable, and add it to your path, and ensure the file is executable. To ensure the file is executable, run the following command:

    ```
    chmod +x kn
    ```


Continue on to [exercise 1](../exercise-1/README.md).
