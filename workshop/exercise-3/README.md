## Deploy our First Application to Knative using the Knative Client (kn)
A relatively new project, the knative client aims to make interacting with Knative a seamless experience for developers. We'll try deploying an application using `kn`, then we'll deploy the same application using `kubectl` and `.yaml` files for additional control. The application for this lab is a simple node.js with express app which returns the first n numbers of the Fibonacci sequence. Once the app is deployed, you can use it by making a GET request to the `/` endpoint with a number as the parameter. 

# Our First Knative Service
Knative Serving supports deploying and serving of serverless applications. Those applications will automatically scale up, and then back down to zero. In this exercise, we'll use the Knative Serving component to deploy our first application from a container image hosted on dockerhub. 

The Knative Service object automatically manages the whole lifecycle for your workload. A service represents your app on Knative. Services control the creation of other objects to ensure that your app has a URL and a new revision for each update of the service.

![](https://github.com/knative/serving/raw/master/docs/spec/images/object_model.png)

# Deploy our Application to Knative using kn
We've already created an image on dockerhub that contains the first version of our Fibonacci application. If we call the `/` endpoint, and pass in a `number` parameter, we should get the first `n` numbers of the Fibonacci sequence.

1. Deploy the application:

    ```
    kn service create --image docker.io/ibmcom/fib-knative fib-knative
    ```

2. Watch the pods initializing as our application gets deployed and starts up:

    ```
    kubectl get pods --watch
    ```

    Note: To exit the watch, use `ctrl + c`.

3. Let's try out our new application! First, let's get the domain name that Knative assigned to the Service we just deployed. Run the following command, and note the value for `domain`. IBM Cloud Kubernetes Service sets the default domain name for Knative to match the domain name of your IBM Cloud Kubernetes Service cluster.

    ```
    kn service get fib-knative
    ```

4. The domain name should look something like `fib-knative.default.bmv-knative-lab.us-south.containers.appdomain.cloud`. We can set an environment variable so that we can use this throughout the lab:

    ```
    export MY_DOMAIN=<your_app_domain_here>
    ```
5. We can now curl this domain to try out our application. Notice that we're calling the `/` endpoint, and passing in a `number` parameter of 5. This should return the first 5 numbers of the fibonacci sequence.

    ```
    curl $MY_DOMAIN/5
    ```

    Expected Output:
    ```
    [1,1,2,3,5]
    ```

6. Congratulations! You've got your first Knative application deployed and responding to requests. Try sending some different number requests. If you stop making requests to the application, you should eventually see that your application scales itself back down to zero. Watch the pod until you see that it is `Terminating`. This should take approximately 90 seconds.

    ```
    kubectl get pods --watch
    ```

    Note: To exit the watch, use `ctrl + c`.

7. We'll redeploy this same application in a different way in the next exercise, so let's clean up.

    ```
    kn service delete fib-knative
    ```

Continue on to [exercise 4](../exercise-4/README.md).