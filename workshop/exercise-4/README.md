## Clone the Application Repo and Deploy our Application using Kubectl and yaml

### Clone the application repo
Let's get the code we'll use for today's lab. This repository contains the code for the Fibonacci application as well as various .yaml files we'll use throughout the lab.

1. Clone the git repository:

    ```
    git clone https://github.com/IBM/fib-knative.git
    ```

2. Change directories to the fib-knative folder.

    ```
    cd fib-knative
    ```

## Deploy Our Application using kubectl and yaml
In this exercise, we'll use the Knative Serving component to deploy our application from a container image hosted on dockerhub. Instead of using the `kn` cli, we'll use `kubectl` and `.yaml` files. This should feel familiar if you're a kubernetes user.

### Create the Configuration and Route for our Service
Knative defines some objects for each component as Kubernetes [Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources)(CRDs). A CRD is used to define a new resource type in Kubernetes. Knative [Serving](https://github.com/knative/docs/tree/master/docs/serving#serving-resources) includes a number of Custom Resource Definitions, including Service, Route, Configuration, and Revision.

Because Knative is built on top of Kubernetes, you can use kubectl along with configuration files to create a new Service representing your application.

1. In the `fib-knative` project you cloned earlier, you should see a file called `fib-service.yaml`. Look at the contents of the file:

    ```
    cat fib-service.yaml
    ```

    File Contents:
    ```
    apiVersion: serving.knative.dev/v1alpha1
    kind: Service
    metadata:
        name: fib-knative
        namespace: default
    spec:
        release:
            revisions: ["@latest"]
            configuration:
            revisionTemplate:
                spec:
                    container:
                        image: docker.io/ibmcom/fib-knative
    ```

    The `fib-service.yaml` file describes a Service. Notice that it includes the name (fib-knative), the namespace (default), and a reference to the container image on dockerhub (docker.io/ibmcom/fib-knative). 
    
2. Let's deploy this app into our cluster. Apply the `fib-service.yaml` file.

    ```
    kubectl apply --filename fib-service.yaml
    ```

3. Watch the pods initializing as our application gets deployed and starts up:

    ```
    kubectl get pods --watch
    ```

    Note: To exit the watch, use `ctrl + c`.

4. Let's try out our application again! Because the service name was the same as the application you deployed before, `fib-knative`, the domain name for our service should be the same. You can double check if you want.

    ```
    kn service get fib-knative
    ```

5. The domain name should look something like `fib-knative.default.bmv-knative-lab.us-south.containers.appdomain.cloud`.

6. We can now curl this domain to try out our application. Notice that we're calling the `/` endpoint, and passing in a `number` parameter of 5. This should return the first 5 numbers of the fibonacci sequence.

    ```
    curl $MY_DOMAIN/5
    ```

    Expected Output:
    ```
    [1,1,2,3,5]
    ```

7. Congratulations! You've got your Knative application deployed and responding to requests again, but this time you deployed with `kubectl` and `.yaml` files. Try sending some different number requests. If you stop making requests to the application, you should eventually see that your application scales itself back down to zero. Watch the pod until you see that it is `Terminating`. This should take approximately 90 seconds.

    ```
    kubectl get pods --watch
    ```

    Note: To exit the watch, use `ctrl + c`.

Continue on to [exercise 5](../exercise-5/README.md).
