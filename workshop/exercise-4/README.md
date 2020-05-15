## Deploy vnext Version and Apply Traffic Shifting

Did you notice that the Fibonacci sequence started with 1? Most would argue that the sequence should actually start with 0. There's a vnext version of the application that starts the sequence with 0 instead of 1. This container image has been built and deployed to dockerhub, and tagged as vnext. We'll deploy that as v2 of our app.

### Deploy vnext
1. Let's deploy vnext, again using a docker image on dockerhub.

    ```
     kn service update fib-knative --image docker.io/ibmcom/fib-knative:vnext --revision-name fib-knative-zero 
    ```

	First, notice that this revision is named `fib-knative-zero`, since the fibonacci sequence will now start with zero. You can see that we're using the vnext version of our application.

2. Let's see the details for our Service, again using `kn service describe`

    ```
    kn service describe fib-knative
    ```

    Example output:
    ```
    Name:       fib-knative
    Namespace:  default
    Age:        1m
    URL:        http://fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud

    Revisions:  
      100%  @latest (fib-knative-zero) [2] (9s)
            Image:  docker.io/ibmcom/fib-knative:vnext (pinned to c13569)

    Conditions:  
      OK TYPE                   AGE REASON
      ++ Ready                   5s 
      ++ ConfigurationsReady     6s 
      ++ RoutesReady             5s 
    ```
  
    We can see that our latest revision (fib-knative-zero) has 100% of the traffic being routed to it.

3. We can try out the application by curling the URL. We should see that the sequence now starts with zero.

    ```
    curl $MY_APP_URL/5
    ```

    Expected Output:
    ```
    [0, 1, 1, 2, 3]
    ```

  
4. What if, instead of 100% of the traffic going to the new revision, we wanted a different percentage? Maybe we want to slowly roll users over from our old version to the new version, or do some A/B testing of the new version to see what users like better. Let's update the service to send 10% of the traffic to our new revision (`fib-knative-zero`), and 90% to the old revision (`fib-knative-one`)

    ```
    kn service update fib-knative --traffic fib-knative-zero=10 --traffic fib-knative-one=90
    ```

5. Again, we can use `kn service describe` to see these changes. Notice the revisions section. You can see that 10% of the traffic will be sent to `fib-knative-zero`, and 90% of the traffic will be sent to `fib-knative-one`.

    ```
    kn service describe fib-knative
    ```

    Expected Output:
    ```
    Revisions:  
      10%  fib-knative-zero (current @latest) [2] (58m)
            Image:  docker.io/ibmcom/fib-knative:vnext (pinned to c13569)
      90%  fib-knative-one [1] (59m)
            Image:  docker.io/ibmcom/fib-knative (pinned to 9eac25)
    ```

6. Let's run some load against the app, just asking for the first number in the Fibonacci sequence so that we can clearly see which revision is being called.

	```
	while sleep 0.5; do curl "$MY_APP_URL/1"; done
	```

    Expected Output:
    ```
    [1][1][0][1][1][1][1][1][1][1][1]
    ```
    
7. We should see that the curl requests are routed approximately 90/10 between the two revisions. Let's kill this process using `ctrl + c`.


Congratulations! You've deployed two versions of the fib-knative application, and then updated the rollout percentage of your application. Continue on to [exercise 5](../exercise-5/README.md).