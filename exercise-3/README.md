# Update Domain Configurations and Ingress Forwarding

When a Knative application is deployed, Knative will define a URL for your application.  By default, this url is "default.example.com." Because we want our application to be accessible at a URL we own, we need to configure Knative to assign new applications to our own hostname.

What hostname should we use? Luckily for us, IBM Kubernetes Service gave us an external domain when we created our cluster.  We'll first get that URL, tell Knative to assign new applications to that URL, and then forward requests to our Ingress Subdomain to the Knative Istio Gateway.

### Update domain configuration for Knative
1. First, let's get our ingress subdomain for our cluster.

	```
	ibmcloud ks cluster-get my-cluster-name
	```

2. Next, update the default URL for new Knative apps by editing the configuration:

	```
	kubectl edit cm config-domain --namespace knative-serving
	```

3. Change instances of `default.example.com` to your ingress subdomain, which should look like: `bmv-knative.us-east.containers.appdomain.cloud`.

### Forward requests coming into IKS ingress to the Knative Istio Gateway

1. Update the forward-ingress.yaml file with your own ingress subdomain, prepended with fib-knative, or whatever subdomain you would like your application to live at.  The file should look something like:

	```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: iks-knative-ingress
    namespace: istio-system
    spec:
      rules:
        - host: fib-knative.default.bmv-knative.us-east.containers.appdomain.cloud
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
