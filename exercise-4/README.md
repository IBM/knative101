## Deploy fibonacci app using kubectl and service.yaml

Let's get our first Knative application up & running. Using the Build & Serving components of Knative, we can go from some source code on github to a docker image built on cluster (using the Kaniko build template, to a docker image pushed to dockerhub, and ultimately a URL to access our application.

1. Edit the service.yaml file to point to your own container registry.

2. Apply the service.yaml file to you cluster.

	```
	kubectl apply -f service.yaml
	```
3. Run `kubectl get pods --watch` to see the pods initializing.

4. Once all the pods are initialized, go to dockerhub to see that your container image was built and pushed to dockerhub.

5. Now that the app is up, we should be able to call it using a number input.  We can do that using a curl command against the URL provided us:

	```
	curl -X POST http://fib-knative.default.bmv-knative.us-east.containers.appdomain.cloud/fib -H 'Content-Type: application/json' -d '{"number":20}'
	```

6. If we left this alone for some time, it would scale itself back down to 0, and terminate the pods that were created.  The default for knative scale-to-zero is 5 minutes.  Let's decrease this time by editing the autoscaler:

	```
	kubectl edit cm config-autoscaler --namespace knative-serving
	```

7. Find `scale-to-zero-threshold`, and decrease the time from 5m to 1m.  You can also decrease the `scale-to-zero-graceperiod`.

8. Run `kubectl get pods --watch` and wait to see the application scale itself back down to 0.
