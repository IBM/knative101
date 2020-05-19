## Deploy our First Application to Knative using the Knative Client (kn)
The Knative client, `kn`, aims to make interacting with Knative a seamless experience for developers. In this section we'll try deploying an application using `kn`. The application for this lab is a simple Node.js with Express app which returns the first n numbers of the Fibonacci sequence. Once the app is deployed, you can use it by making a GET request to the `/` endpoint with a number as the parameter. 

### Our First Knative Service
Knative Serving enables rapid deploying and serving of serverless applications. Those applications will automatically scale up, and then back down to zero. In this exercise, we'll use the Knative Serving component to deploy our first application from a container image hosted on dockerhub. 

The Knative Service object automatically manages the whole lifecycle for your workload. A Service represents your app on Knative. Services control the creation of other objects to ensure that your app has a URL and a new Revision for each update of the service.

![](https://github.com/knative/serving/raw/master/docs/spec/images/fibknativev1.png)

### Deploy our Application to Knative using kn
We've already created an image on dockerhub that contains the first version of our Fibonacci application. If we call the `/` endpoint, and pass in a `number` parameter, we should get the first `n` numbers of the Fibonacci sequence.

1. Deploy the application. We will create a Knative Service named fib-knative which will run our fib-knative image on dockerhub.

    ```
    kn service create fib-knative --image docker.io/ibmcom/fib-knative
    ```

2. You should see some output indicating that the service was created. You should also be given the URL where your application will be available. It should look something like `htpp://fib-knative.default.bmv-knative-lab.us-south.containers.appdomain.cloud`. 

    ```
    Creating service 'fib-knative' in namespace 'default':

    0.316s The Route is still working to reflect the latest desired specification.
    0.637s Configuration "fib-knative" is waiting for a Revision to become ready.
    5.305s ...
    5.400s Ingress has not yet been reconciled.
    6.424s Ready to serve.

    Service 'fib-knative' created to latest revision 'fib-knative-ywvgm-1' is available at URL:
    http://fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud
    ```

    You should see your cluster name as a part of the URL, since IBM Cloud Kubernetes Service sets the default domain name for Knative to match the domain name of your cluster.

3. Save this URL as an environment variable, so that we can use it throughout the lab:

    ```
    export MY_APP_URL=<your_application_url_here>
    ```

4. Let's take a look at some details about the service we just created using the `kn service describe` command.

    ```
    kn service describe fib-knative
    ```

    Example Output:
    ```
    Name:       fib-knative
    Namespace:  default
    Age:        11s
    URL:        http://fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud

    Revisions:  
      100%  @latest (fib-knative-lhghx-1) [1] (11s)
            Image:  docker.io/ibmcom/fib-knative (pinned to 9eac25)

    Conditions:  
      OK TYPE                   AGE REASON
      ++ Ready                   6s 
      ++ ConfigurationsReady     7s 
      ++ RoutesReady             6s 
    ```

    We can see the URL, the name of the service, and some status information about the service. We can also look at the various revisions for this service. Currently, we only have one revision with 100% of the traffic being routed there.

5. Let's try out our application! We can curl the URL to try out our application. Notice that we're calling the `/` endpoint, and passing in a `number` parameter of 5. This should return the first 5 numbers of the Fibonacci sequence.

    ```
    curl $MY_APP_URL/5
    ```

    Expected Output:
    ```
    [1,1,2,3,5]
    ```

6. Congratulations! You've got your first Knative application deployed and responding to requests. Try sending some different number requests.

Continue on to [exercise 4](../exercise-4/README.md).