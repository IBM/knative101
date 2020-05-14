## Deploy vnext Version and Apply Traffic Shifting

Did you notice that the Fibonacci sequence started with 1? Most would argue that the sequence should actually start with 0. There's a vnext version of the application at the vnext branch in the github project that starts the sequence with 0 instead of 1. This container image has been built and deployed to dockerhub, and tagged as vnext. We'll deploy that as v2 of our app.

### Deploy vnext
1. Let's deploy vnext, again using a docker image on dockerhub. Maybe we want to slowly roll users over from our old version to the new version, or do some A/B testing of the new version to see what users like better. Let's see what the yaml for this looks like.

    ```
    cat fib-service2.yaml
    ```

    Expected Output:
    ```
    apiVersion: serving.knative.dev/v1
    kind: Service
    metadata:
      name: fib-knative
      namespace: default
    spec:
      template:
        metadata:
          name: fib-knative-zero
        spec:
          containers:
            - image: docker.io/ibmcom/fib-knative:vnext
      traffic:
      - revisionName: fib-knative-one
        percent: 90
      - revisionName: fib-knative-zero
        percent: 10
    ```

	First, notice that this revision is named `fib-knative-zero`, since the fibonacci sequence will now start with zero. You can see that we're using the vnext version of our application. 
  
  Next, take a look at the `traffic` block. You can see that we'll send 10% of our traffic to the new revision (`fib-knative-zero`), and 90% of our traffic to the old revision (`fib-knative-one`).

2. Apply this new configuration to the cluster.

    ```
    kubectl apply -f fib-service2.yaml
    ```

3. Let's use `kn service describe` to see some details about our updated service.

    ```
    kn service describe fib-knative
    ```
    
    Example Output:
    ```
    Name:       fib-knative
    Namespace:  default
    Age:        2m
    URL:        http://fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud

    Revisions:  
      10%  fib-knative-zero (current @latest) [2] (1m)
            Image:  docker.io/ibmcom/fib-knative:vnext (at c13569)
      90%  fib-knative-one [1] (2m)
            Image:  docker.io/ibmcom/fib-knative (at 9eac25)

    Conditions:  
      OK TYPE                   AGE REASON
      ++ Ready                   1m 
      ++ ConfigurationsReady     1m 
      ++ RoutesReady             1m 
    ```
    
4. Notice the revisions section. You can see that 10% of the traffic will be sent to `fib-knative-zero`, and 90% of the traffic will be sent to `fib-knative-one`.

5. Let's run some load against the app, just asking for the first number in the Fibonacci sequence so that we can clearly see which revision is being called.

	```
	while sleep 0.5; do curl "$MY_APP_URL/1"; done
	```

    Expected Output:
    ```
    [1][1][0][1][1][1][1][1][1][1][1]
    ```
    
6. We should see that the curl requests are routed approximately 90/10 between the two revisions. Let's kill this process using `ctrl + c`.


Congratulations! You've completed the lab! You've deployed a new version of fib-knative, and updated the rollout percentage of your application.