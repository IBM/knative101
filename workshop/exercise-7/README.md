## (OPTIONAL, FOR ADVANCED USERS) Set up a Private Container Image Registry and Obtain Credentials

In the previous exercises, we deployed a container to Knative directly from dockerhub. What if we want to build our own container from source? In this lab, we'll use the [IBM Container Registry](https://cloud.ibm.com/docs/services/Registry?topic=registry-registry_overview#registry_overview) to host our container images since you already have access to this through your IBM Cloud Account. IBM Container Registry enables you to store and distribute container images in a fully managed private registry.

### Clone the application repo
Let's first get the code we'll use for this portion of the lab. This repository contains the code for the Fibonacci application as well as various .yaml files we'll be using.

1. Clone the git repository:

    ```
    git clone https://github.com/IBM/fib-knative.git
    ```

2. Change directories to the fib-knative folder.

    ```
    cd fib-knative
    ```

### Create an IBM Container Registry namespace
We will later use this container registry to store the image that we're building from our git repository.

3. You will now need to create an IBM Container Registry namespace. Let's confirm you're logged in.

    ```
    ibmcloud login
    ```

    When prompted, select your own account, and then enter the number for the region `us-south`.


4. Add a namespace to your account. You must create at least one namespace to store images in IBM Cloud Container Registry. Choose a unique name for your first namespace. A namespace is a collection of related repositories (which in turn are made up of individual images). You can create multiple namespaces as well as control access to your namespaces by using IAM policies.

    ```
    ibmcloud cr namespace-add <my_namespace>
    ```

5. Create an API key. This API key can be used to automate pushing and pulling of Docker images to and from your namespaces. The automated build processes you'll be setting up will use this key to access your images.

    ```
    ibmcloud iam api-key-create mykey -d "API key for IBM Cloud"
    ```

6. The CLI output should include the API Key value. Create an environment variable containing your key.

    ```
    export MYAPIKEY=<your_api_key_value>
    ```

### Provide Container Registry Credentials to Cluster
This lab will need credentials for authenticating to your private container registry. First, we'll need to create a `Secret` to store the credentials for this registry. This secret will be used for the knative-serving component to pull down an image from the container registry.

A `Secret` is a Kubernetes object containing sensitive data such as a password, a token, or a key. You can also read more about [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).

1. Let's create a secret, which will be a `docker-registry` type secret. This type of secret is used to authenticate with a container registry to pull down a private image. We can create this via the commandline. For username, simply use the string `iamapikey`. For `<api_key_value>`, use the api key you made note of earlier.

    ```
    kubectl --namespace default create secret docker-registry ibm-cr-secret  --docker-server=us.icr.io --docker-username=iamapikey --docker-password=$MYAPIKEY
    ```

2. We will also need a secret for the build process to have credentials to push the built image to the container registry. To create this secret, we'll first need to base64 encode our username and password for IBM Container Registry. For username, you will use the string `iamapikey`. The base64 encoding of `iamapikey` should be: `aWFtYXBpa2V5` - which is already in the yaml file.  Let's base64 encode our apikey, and copy it.

    ```
    echo -n $MYAPIKEY | base64 -w0  # Linux
    echo -n $MYAPIKEY | base64 -b0  # MacOS
    ```

3. Update the `docker-secret.yaml` file with your base64 encoded password. You can find the password field near the end of the file. Username (`aWFtYXBpa2V5=`) is already provided for you.  Replace `<base_64_encoded_api_key_value>` with your own base64 encoded apikey value, and ensure you've saved the file.

4. Apply the secret to your cluster:

    ```
    kubectl apply --filename docker-secret.yaml
    ```

5. A `Service Account` provides an identity for processes that run in a Pod. This Service Account will be used to link the build process for Knative to the Secrets you just created. View the service account file, and notice that it's using the credentials you created earlier for pulling from and pushing to the container registry:

    ```
    cat service-account.yaml
    ```

    Example output:
    ```yaml
     apiVersion: v1
     kind: ServiceAccount
     metadata:
       name: build-bot
     secrets:
     - name: ibm-cr-push-secret
     imagePullSecrets:
     - name: ibm-cr-secret
    ```


6. Apply the service account to your cluster:

    ```
    kubectl apply --filename service-account.yaml
    ```

Congratulations! You've set up some required credentials that the Tekton Pipeline process will use to have access to push to your container image registry. In the next exercise, you will build the image, push it to the registry, and then deploy the app to knative using Tekton Pipelines. The goal of this exercise was to set up some required credentials for that flow.


Continue on to [exercise 8](../exercise-8/README.md).
