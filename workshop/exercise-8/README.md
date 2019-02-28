## A/B Testing with knctl

Maybe we want to slowly roll users over from our old version to the new version, or do some A/B testing of the new version. We can use the knctl rollout command to route traffic percentages to our revisions.

1. Check the current route percentages:

	```
	knctl route list
	```

	Output:
	```
	Routes in namespace 'default'

	Name         Domain                                                              Traffic                   Annotations  Conditions  Age  
	fib-knative  fib-knative.default.mycluster6.us-south.containers.appdomain.cloud  100% -> fib-knative  -            3 OK / 3    20h  

	1 routes

	Succeeded
	```

2. Send 50% of the traffic to the latest revision, and 50% to the previous revision. Notice that we're using the previous and latest tags that were created for us as a part of the knctl deploy command.

	```
	knctl rollout --route fib-knative -p fib-knative:latest=50% -p fib-knative:previous=50%
	```

3. Check the route percentages to confirm they've updated:

	```
	knctl route list
	```

	Output:
	```
	Routes in namespace 'default'

	Name         Domain                                                              Traffic                   Annotations  Conditions  Age  
	fib-knative  fib-knative.default.mycluster6.us-south.containers.appdomain.cloud  50% -> fib-knative-00003  -            3 OK / 3    15h  
                                                                                 	 50% -> fib-knative-00002                             

	1 routes

	Succeeded
	```

3. Let's run some load against the app, just asking for the first number in the Fibonacci sequence so that we can clearly see which revision is being called. Ensure you've replaced `<ingress_subdomain>` with your own ingress subdomain.

	```
	while sleep 0.5; do curl "http://fib-knative.default.<ingress_subdomain>/1" ; done
	```

4. We should see that the curl requests are routed approximately 50/50 between the two applications. Let's kill this process using `ctrl + c`.


At this point, you should feel that you've had a whirlwind tour of Knative. We installed Istio & Knative onto a Kubernetes cluster. We deployed a serverless application and saw it scale up and then back down when it was no longer in use. We also explored the knctl tool to easily create routing rules and deploy serverless applications. Please reach out should you have any questions or issues going through this lab!
