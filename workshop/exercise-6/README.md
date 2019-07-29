## (OPTIONAL, FOR ADVANCED USERS) Set up a Private Container Registry and Obtain Credentials

In the previous exercises, we deployed a container to Knative directly from dockerhub. What if we want to build our own container from source? In this lab, we'll use the [IBM Container Registry](https://console.bluemix.net/docs/services/Registry/registry_overview.html#registry_overview) to host our container images since you already have access to this through your IBM Cloud Account. IBM Container Registry enables you to store and distribute container images in a fully managed private registry.

1. You will need to first create an IBM Container Registry namespace. Let's confirm you're logged in.

    ```
    ibmcloud login
    ```

    When prompted, select your own account, and then enter the number for the region `us-south`.


1. Add a namespace to your account. You must set at least one namespace to store images in IBM Cloud Container Registry. Choose a unique name for your first namespace. A namespace is a collection of related repositories (which in turn are made up of individual images). You can set up multiple namespaces as well as control access to your namespaces by using IAM policies.

    ```
    ibmcloud cr namespace-add <my_namespace>
    ```

2. Create a token. This token is a non-expiring token with read and write access to all namespaces in the region. The automated build processes you'll be setting up will use this token to access your images.

    ```
    ibmcloud cr token-add --description "knative read write token for all namespaces" --non-expiring --readwrite
    ```

3. Create an environment variable containing your token.

    ```
    export MYTOKEN=<your_token_value>
    ```

4. The CLI output should include a Token Identifier and the Token. Make note of the Token for later in this lab. You will not need the Token Identifier. You can verify that the token was created by listing all tokens.

    ```
    ibmcloud cr token-list
    ```

### Provide Container Registry Credentials to Cluster
This lab will need credentials for authenticating to your private container registry. First, we'll need to create a `Secret` to store the credentials for this registry. This secret will be used for the knative-serving component to pull down an image from the container registry.

A `Secret` is a Kubernetes object containing sensitive data such as a password, a token, or a key. You can also read more about [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).

1. Let's create a secret, which will be a `docker-registry` type secret. This type of secret is used to authenticate with a container registry to pull down a private image. We can create this via the commandline. For username, simply use the string `token`. For `<token_value>`, use the token you made note of earlier.

    ```
    kubectl create secret docker-registry ibm-cr-secret --docker-server=https://registry.ng.bluemix.net --docker-username=token --docker-password=$MYTOKEN
    ```

2. We will also need a secret for the build process to have credentials to push the built image to the container registry. To create this object, we'll first need to base64 encode our username and password for IBM Container Registry. For username, you will again use the string `token`. The base64 encoding of `token` should be: `dG9rZW4=` - which is already in the yaml file.  Let's base64 encode our token, and copy it.

    ```
    echo -n "$MYTOKEN" | base64 -w0  # Linux
    echo -n "$MYTOKEN" | base64 -b0  # MacOS
    ```

3. This time we'll create the secret via a .yaml file. Update the `docker-secret.yaml` file with your base64 encoded password.

5. You can find the password field near the end of the file. Username (`dG9rZW4=`) is already provided for you.  Replace `<base_64_encoded_token_value>` with your own base64 encoded token value, and ensure you've saved the file.

6. Apply the secret to your cluster:

    ```
    kubectl apply --filename docker-secret.yaml
    ```

7. A `Service Account` provides an identity for processes that run in a Pod. This Service Account will be used to link the build process for Knative to the Secrets you just created. View the service account file, and notice that it's using the credentials you created earlier for pulling from and pushing to the container registry:

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
     - name: basic-user-pass
     imagePullSecrets:
     - name: ibm-cr-secret
    ```


8. Apply the service account to your cluster:

    ```
    kubectl apply --filename service-account.yaml
    ```

Congratulations! You've set up some required credentials that the Knative build process will use to have access to push to your container registry. In the next exercise, you will build the container, push it to your container registry, and then deploy the app to knative. The goal of this exercise was to set up some required credentials for that flow.


Continue on to [exercise 7](../exercise-7/README.md).