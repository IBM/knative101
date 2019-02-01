## Create IBM Cloud Account and Install Developer Tools

If you already have an IBM Cloud account with the following tools installed, you can skip these steps:

- `ibmcloud` CLI for interacting with IBM Cloud
- `ibmcloud ks` plugin IBM Kubernetes Service
- `ibmcloud cr` plugin for IBM Container Registry
- `kubectl` CLI for interacting with Kubernetes
- `knctl` CLI for interacting with Knative

### Create an IBM Cloud Account
1. Create an account on [IBM Cloud](https://cloud.ibm.com/registration).

### Installing the IBM Cloud developer tools and plugins

1. Download and install the `ibmcloud` command line tool:
    https://cloud.ibm.com/docs/cli/index.html#overview

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

### Install knctl CLI
The Knative command-line tool, knctl provides a set of commands to interact with a Knative installation. Let's install knctl as well.

1. Install knctl using one of the prebuilt binaries from their [release page](https://github.com/cppforlife/knctl/releases), and run the following commands:

    ```
    shasum -a 265 ~/Downloads/knctl-*
    # Compare checksum output to what's included in the release notes

    mv ~/Downloads/knctl-* /usr/local/bin/knctl

    chmod +x /usr/local/bin/knctl
    ```

Continue on to [exercise 1](../exercise-1/README.md).
