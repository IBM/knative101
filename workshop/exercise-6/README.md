## Build and Deploy our Knative Application

We've seen how to deploy an application from a container that already exists, so let's build the container image ourselves from source code. Using the Build & Serving components of Knative, we can build the image and push it to a private container registry on IBM Cloud, and then ultimately get a URL to access our application.

### Install Kaniko Build Template

As a part of this lab, we will use the kaniko build template for building source into a container image from a Dockerfile. Kaniko doesn't depend on a Docker engine and instead executes each command within the Dockerfile completely in userspace. This enables building container images in environments that can't easily or securely run a Docker engine, such as Kubernetes.

A Knative BuildTemplate encapsulates a shareable build process with some limited parameterization capabilities.

1. Install the kaniko build template to your cluster.

    ```
    kubectl apply --filename https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml
    ```

2. Use kubectl to confirm you installed the kaniko build template, as well as to see some more details about it.  You'll see that this build template accepts parameters of `IMAGE` and `DOCKERFILE`.  `IMAGE` is the name of the image you will push to the container registry, and `DOCKERFILE` is the path to the Dockerfile that will be built.

	Command:
	```
	kubectl get BuildTemplate kaniko -o yaml
	```

	Example Output:
	```yaml
      spec:
        generation: 1
        parameters:
        - description: The name of the image to push
          name: IMAGE
        - default: /workspace/Dockerfile
          description: Path to the Dockerfile to build.
          name: DOCKERFILE
        steps:
        - args:
          - --dockerfile=${DOCKERFILE}
          - --destination=${IMAGE}
          image: gcr.io/kaniko-project/executor
          name: build-and-push
	```


### Deploy the Fibonacci App Using kubectl and service.yaml

1. Edit the `service.yaml` file to point to your own container registry namespace by replacing instances of `<NAMESPACE>` with the container registry namespace you created earlier.

2. Apply the `service.yaml` file to your cluster.

	```
	kubectl apply -f service.yaml
	```
3. Run `kubectl get pods --watch` to see the pods initializing. Note: To exit the watch, use `ctrl + c`.

4. Take a look at the `service.yaml` file again. The service.yaml file defines the required Knative components to build the application from source code in the git repository and push the built container image to the private container registry. Then it'll replace the currently running version of the application with this new one.

5. Now that the app is up, we should be able to call it using a number input. We can do that using a curl command against the URL provided to us. Esnure you've updated the command with your own ingress subdomain.

	```
	curl http://fib-knative.default.<ingress_subdomain>/20
	```
6. You should see the first 20 Fibonacci numbers!

7. If we left this application alone for some time, it would scale itself back down to 0, and terminate the pods that were created. Run `kubectl get pods --watch` and wait until you see the application scale itself back down to 0. When the application is no longer in use, you should eventually see the pods move from the `Running` to the `Terminating` state. Note: To exit the watch, use `ctrl + c`.

	Expected Output:
	```
	NAME                                            READY   STATUS      RESTARTS   AGE
	fib-knative-00002-deployment-58dcbdb97c-rrnzc   3/3     Running     0          56s
	fib-knative-00002-deployment-58dcbdb97c-rrnzc   3/3   Terminating   0          89s
	fib-knative-00002-deployment-58dcbdb97c-rrnzc   0/3   Terminating   0          91s
	```

Continue on to [exercise 7](../exercise-7/README.md).
