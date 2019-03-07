## Deploy vnext Version Using knctl

Did you notice that the Fibonacci sequence started with 1? Most would argue that the sequence should actually start with 0. There's a vnext version of the application at the vnext branch in the github project. We'll deploy that as v2 of our app, but instead of using kubectl, let's try the knctl tool for interacting with Knative.

### Deploy vnext
1. Let's deploy vnext, but instead of kubectl with the service.yaml file, let's use knctl. By providing Knative with the source of our app and the image to push to the container registry, we'll get an application with a URL we can access.

    ```
    knctl deploy \
        --service fib-knative \
        --git-url https://github.com/IBM/fib-knative \
        --git-revision vnext \
        --service-account build-bot \
        --image registry.ng.bluemix.net/<NAMESPACE>/fib-knative:vnext \
        --managed-route=false
    ```

	This command will tell Knative to go out to github, find the code, build it into a container, and push that container to the IBM Container Registry. One thing you'll notice if you follow the output logs is that this deploy command also tags my app versions with a `latest` and a `previous` tag.

2. See the revisions using knctl.

	```
	knctl revisions list
	```
	Expected output:
	
	```
    Revisions

    Service      Name               Tags      Annotations  Conditions  Age  Traffic  
    fib-knative  fib-knative-00003  latest    -            5 OK / 5    45s  100% -> fib-knative.default.mycluster6.us-south.containers.appdomain.cloud  
    ~            fib-knative-00002  previous  -            5 OK / 5    3m   -  
    ~            fib-knative-00001  -         -            5 OK / 5    3m   - 

    3 revisions

    Succeeded
    ```

3. Curl the application to see that 100% of the traffic is hitting your new fib-knative revision, starting the sequence with 0. Ensure you've updated the command with your own ingress subdomain.

    ```
    curl http://fib-knative.default.$MYINGRESS/20
    ```


Continue on to [exercise 8](../exercise-8/README.md).
