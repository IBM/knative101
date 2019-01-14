## Clone the Lab Repo & Provide Container Registry Credentials

### Clone the lab repo
The application for this lab is a simple node.js with express app which returns the first n numbers of the Fibonacci sequence. To use the app, host it, and simply make a POST request to the `/fib` endpoint with the JSON data: `{"number":10}`

1. Clone the git repository:

	```
	git clone https://github.com/beemarie/fib-knative.git
	```
2. Change directories to the fib-knative folder.

	```
	cd fib-knative
	```


### Provide Container Registry Credentials
This lab will need credentials for authenticating to your container registry - we'll be using dockerhub. You could also use the IBM Container Registry. First, we'll need to create a `Secret` to store the credentials.

A `Secret` is a Kubernetes object containing sensitive data such as a password, a token, or a key. You can also read more about [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).

1. To create this object, we'll first need to base64 encode our username and password for dockerhub.

	```
	echo -n "username" | base64

	echo -n "password" | base64
	```

2. Update the `docker-secret.yaml` file with your base64 encoded username and password.
3. Apply the secret to your cluster:

      ```
      kubectl apply --filename docker-secret.yaml
      ```

      View the yaml file used to create the Secret:
      ```
      kubectl get secret basic-user-pass -o yaml
      ```

      Example output:

      ```yaml
      apiVersion: v1
        kind: Secret
        metadata:
          name: basic-user-pass
          annotations:
            build.knative.dev/docker-0: https://index.docker.io/v1/
        type: kubernetes.io/basic-auth
        data:
          # Use 'echo -n "username" | base64' to generate this string
          username: your_base_64_username
          # Use 'echo -n "password" | base64' to generate this string
          password: your_base_64_password
      ```
A `Service Account` provides an identity for processes that run in a Pod. This Service Account will be used to link the build process for Knative to the Secret you created earlier.

4. Apply the service account to your cluster:

    ```
    kubectl apply --filename service-account.yaml
    ```

    View the yaml file used to create the Service Account:
    ```
    kubectl get serviceaccount default -o yaml
    ```

    Example output:
    ```yaml
       apiVersion: v1
     kind: ServiceAccount
     metadata:
       name: build-bot
     secrets:
     - name: basic-user-pass
    ```


Congratulations! You've set up some required credentials for the Knative build process to have access to push to your container registry. In the next exercise, you will build & deploy this app. The goal of this exercise was to set up the credentials for your app.
