## Deploy Our first Knative application

Let's get our first Knative application up & running. Using the Build & Serving components of Knative, we can go from some source code on github to a container image built on cluster (using the Kaniko build template), to a container image pushed to a private container registry on IBM Cloud, and ultimately a URL to access our application.


### Update domain configuration for Knative
When a Knative application is deployed, Knative will define a URL for your application. By default, Knative Serving routes use `example.com` as the default domain. The fully qualified domain name for a route by default is `{route}.{namespace}.{default-domain}`, where `{route}` is the name of the application to deploy.

Because we want our application to be accessible at a URL we own, we need to configure Knative to assign new applications to our own domain.

What hostname should we use? Luckily for us, IBM Kubernetes Service gave us an external domain when we created our cluster. We'll first get that URL, tell Knative to assign new applications to that URL, and then forward requests to our Ingress Subdomain to the Knative Istio Gateway.

1. First, let's get the ingress subdomain for our cluster.

	```
	ibmcloud ks cluster-get <my-cluster-name>
	```

	Example Output:
	```
	Ingress Subdomain:      bmv-knative.us-east.containers.appdomain.cloud   
	```
	Ingress is a Kubernetes service that balances network traffic workloads in your cluster by forwarding public or private requests to your apps. This Ingress Subdomain is an externally available and public URL providing access to your cluster. Take note of this Ingress Subdomain.

2. Next, update the default URL for new Knative apps by editing the configuration:

	```
	kubectl edit cm config-domain --namespace knative-serving
	```

3. Change all instances of `example.com` to your ingress subdomain, which should look something like: `bmv-knative.us-east.containers.appdomain.cloud`. There should be two instances of `example.com`, one under `data` and one under `annotations`. New Knative applications will now be assigned a route with this host, rather than `example.com`.

### Forward specific requests coming into IKS ingress to the Knative Ingress Gateway

1. When requests come in to our fibonacci application through the ingress subdomain, we want them to be forwarded to the Knative ingress gateway. Update the `forward-ingress.yaml` file with your own ingress subdomain, prepended with `fib-knative.default`. Remember that the fully qualified domain name for a route has the following form: `{route}.{namespace}.{domain}`.

The file should look something like:

```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: iks-knative-ingress
    namespace: istio-system
    spec:
      rules:
        - host: fib-knative.default.<ingress_subdomain>
          http:
            paths:
              - path: /
                backend:
                  serviceName: knative-ingressgateway
                  servicePort: 80
```

2. Apply the ingress rule.

	```
	kubectl apply --filename forward-ingress.yaml
	```

### Deploy the Fibonacci App Using kubectl and service.yaml

1. Edit the `service.yaml` file to point to your own container registry namespace by replacing instances of <NAMESPACE> with the container registry namespace you created earlier.

2. Apply the `service.yaml` file to your cluster.

	```
	kubectl apply -f service.yaml
	```
3. Run `kubectl get pods --watch` to see the pods initializing.

4. Once all the pods are initialized, you can see that your container image was built and pushed to the IBM Container Registry:

	```
	ibmcloud cr image-list
	```
	Expected output:
	
	```
	registry.ng.bluemix.net/bmv-ibm/fib-knative   latest     bd42179ab4a7     bmv-ibm	1 minute ago	26 MB	No Issues
	```

5. Now that the app is up, we should be able to call it using a number input. We can do that using a curl command against the URL provided to us:

	```
	curl -X POST http://fib-knative.default.<ingress_subdomain>/fib -H 'Content-Type: application/json' -d '{"number":20}'
	```
6. You should see the first 20 Fibonacci numbers!

7. If we left this application alone for some time, it would scale itself back down to 0, and terminate the pods that were created. Run `kubectl get pods --watch` and wait until you see the application scale itself back down to 0. When the application is no longer in use, you should eventually see the pods move from the `Running` to the `Terminating` state.

	Expected Output:
	```
	NAME                                            READY   STATUS      RESTARTS   AGE
	fib-knative-00001-deployment-58dcbdb97c-rrnzc   3/3     Running     0          56s
	fib-knative-00001-deployment-58dcbdb97c-rrnzc   3/3   Terminating   0          89s
	fib-knative-00001-deployment-58dcbdb97c-rrnzc   0/3   Terminating   0          91s
	```

Continue on to [exercise 4](../exercise-4/README.md).
