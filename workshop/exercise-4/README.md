## Deploy Our First Knative Application
Knative Serving supports deploying and serving of serverless applications and functions. Those applications and functions will automatically scale up, and then back down to zero. In this exercise, we'll use the Knative Serving component to deploy our first application from a container image hosted on dockerhub.

### Create a Service Definition
Knative defines some objects for each component as Kubernetes [Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources)(CRDs). A CRD is used to define a new resource type in Kubernetes. Knative [Serving](https://github.com/knative/docs/tree/master/docs/serving#serving-resources) includes a number of Custom Resource Definitions, including Service, Route, Configuration, and Revision.

Because Knative is built on top of Kubernetes, you can use kubectl along with a service definition file to create a new Service definition for your application.

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
      runLatest:
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

4. Let's try out our new application! First, let's get the domain name that Knative assigned to the Service we just deployed. Run the following command, and note the value for `domain`. IBM Cloud Kubernetes Service sets the default domain name for Knative to match the domain name of your IBM Cloud Kubernetes Service cluster. It will also set up the Istio ingress to route all incoming requests targeted to that domain to Knative.

    ```
    kubectl get ksvc fib-knative
    ```

5. The domain name should look something like `fib-knative.default.bmv-kubeflow.us-south.containers.appdomain.cloud`. We can set an environment variable so that we can use this throughout the lab:

    ```
    export MY_DOMAIN=<your_app_domain_here>
    ```

6. We can now curl this domain to try out our application. Notice that we're calling the `/` endpoint, and passing in a `number` parameter of 5. This should return the first 5 numbers of the fibonacci sequence.

    ```
     curl $MY_DOMAIN/5
    ```

    Expected Output:
    ```
    [1,1,2,3,5]
    ```

7. Congratulations! You've got your first Knative application deployed and responding to requests. Try sending some different number requests. If you stop making requests to the application, you should eventually see that your application scales itself back down to zero. Watch the pod until you see that it is `Terminating`. This should take approximately 90 seconds.

    ```
    kubectl get pods --watch
    ```

    Note: To exit the watch, use `ctrl + c`.

Continue on to [exercise 5](../exercise-5/README.md).
