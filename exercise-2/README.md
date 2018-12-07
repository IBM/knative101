## Clone the lab repo & Provide Container Registry Credentials

### Clone the lab repo
The application for this lab is a simple node.js with express app which returns the first n numbers of the fibonacci sequence.  To use the app, host it, and simply make a POST request to the `/fib` endpoint with the JSON data: `{"number":10}`

1. Clone the git repository:

	```
	git clone https://github.com/beemarie/fib-knative.git
	```
2. Change directories to the fib-knative folder.

	```
	cd fib-knative
	```


### Provide Container Registry Credentials
This lab will need credentials for authenticating to your container registry - we'll be using dockerhub.  You could also use the IBM Container Registry. First, we'll need to create a `Secret` to store the credentials.

A `Secret` is a Kubernetes object containing sensitive data such as a password, a token, or a key. You can also read more about [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).

1. To create this object, we'll first need to base64 encode our username and password for dockerhub.

	```
	echo -n "username" | base64 -b 0

	echo -n "password" | base64 -b 0
	```

2. Update the `docker-secret.yaml` file with your base64 encoded username and password.
3. Apply the secret to your cluster:

	```
	kubectl apply --filename docker-secret.yaml

	```

A `Service Account` provides an identity for processes that run in a Pod. This Service Account will be used to link the build process for Knative to the Secret you created earlier.

1.  Apply the service account to your cluster:

	```
	kubectl apply --filename service-account.yaml
	```
