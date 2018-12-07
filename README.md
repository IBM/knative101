# knative101
Lab instructions for knative101 - Knative on IKS


## Create IBM Cloud Account and Get Cluster
> If you already have `ibmcloud` installed with the `ibmcloud cs` plugin, you
> can skip these steps.

### Installing the IBM Cloud developer tools

1.  Download and install the `ibmcloud` command line tool:
    https://console.bluemix.net/docs/cli/index.html#overview

1.  Install the `cs` (container-service) plugin:
    ```bash
    ibmcloud plugin install container-service -r Bluemix
    ```
1.  Authorize `ibmcloud`:
    ```bash
    ibmcloud login
    ```

### Create a standard cluster
This lab requires a standard (paid) cluster. Create a new standard cluster from the [IBM Cloud UI](https://console.bluemix.net/containers-kubernetes/catalog/cluster/create).
1. To ensure the cluster is large enough to host all the Knative and Istio
components, the recommended configuration for a cluster is:
  - Kubernetes version 1.10 or later
  - 4 vCPU nodes with 16GB memory (`b2c.4x16`)

2. It is required to select the worker zone, as well as to create a unique cluster name for your cluster.

3. Click `Create Cluster`.

4. Wait while your cluster is fully deployed. Repeat this command until the state of the cluster is `normal`.

    ```
    ibmcloud cs clusters | grep $CLUSTER_NAME
    ```

### Set context for kubectl
Set the context for your cluster in your CLI.  Every time you log in to the CLI to work with the cluster, you must run this command to set a path to the cluster's configuration file as a session variable. The Kubernetes CLI uses this variable to find a local configuration file and certificates that are necessary to connect with the cluster in IBM Cloud.

1. Download the configuration file and certificates for your cluster using the `cluster-config` command.

    ```shell
    ibmcloud cs cluster-config <your_cluster_name>
    ```

2. Copy and paste the output command from the previous step to set the `KUBECONFIG` environment variable and configure your CLI to run `kubectl` commands against your cluster.

    Example:
    ```shell
    export KUBECONFIG=/Users/user-name/.bluemix/plugins/container-service/clusters/mycluster/kube-config-hou02-mycluster.yml
    ```

3. Validate access to your cluster, by viewing nodes in the cluster.

    ```shell
    kubectl get node
    ```

## Install Istio, Knative, and Kaniko Build Template

Knative is currently built on top of both Kubernetes and Istio.  You will need to install Istio.

1. Install Istio:

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.2/istio.yaml
	```
2. Label the default namespace with `istio-injection=enabled`:

	```
	kubectl label namespace default istio-injection=enabled
	```

3.  Monitor the Istio components until all of the components show a `STATUS` of
    `Running` or `Completed`:

    ```
    kubectl get pods --namespace istio-system --watch
    ```

After installing Istio, you can install Knative.  For this lab, we will install both the Build & Serving components of Knative.

1. Install Knative:

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.2/release.yaml
	```

2. Monitor the Knative components until all of the components show a `STATUS` of `Running` or `Completed`:

	```
	kubectl get pods --namespace knative-serving
	kubectl get pods --namespace knative-build
  ```

As a part of this lab, we will use the Kaniko Build Template for building source into a container image from a Dockerfile, inside a container or a Kubernetes cluster. Typically, to build a container image, it is required to run a Docker daemon with root access. According to the [Kaniko github project](https://github.com/GoogleContainerTools/kaniko), "Kaniko doesn't depend on a Docker daemon and executes each command within a Dockerfile completely in userspace.  This enables building container images in environments that can't easily or securely run a Docker daemon, such as a standard Kubernetes cluster."

1. Install the Kaniko Build Template to your cluster.

  ```
  kubectl apply --filename https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml
  ```

## Clone the lab repo
The application for this lab is a simple node.js with express app which returns the first n numbers of the fibonacci sequence.  To use the app, host it, and simply make a POST request to the `/fib` endpoint with the JSON data: `{"number":10}`

1. Clone the git repository:

	```
	git clone https://github.com/beemarie/fib-knative.git
	```
2. Change directories to the fib-knative folder.

	```
	cd fib-knative
	```

## Provide Container Registry Credentials
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

## Update Domain Configurations and Ingress Forwarding
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


## Deploy app using Kubectl and service.yaml
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

## Deploy vnext version using Knctl
Did you notice that the fibonacci sequence started with 1? Some would argue that the sequence should actually start with 0, 1, 1, 2.  There's a vnext version of the application living in the vnext branch in the github project.  We'll deploy that as v2 of our app, but instead of using kubectl, let's try a new tool.

knctl is a new Knative CLI providing a simple set of commands to interact with a Knative installation.  Let's try it out.

### Installing knctl
1. Install knctl using one of the prebuilt binaries from their [release page](https://github.com/cppforlife/knctl/releases), and run the following commands:

  ```
  shasum -a 265 ~/Downloads/knctl-*
  # Compare checksum output to what's included in the release notes

  mv ~/Downloads/knctl-* /usr/local/bin/knctl

  chmod +x /usr/local/bin/knctl
  ```

### Deploy vnext
1. We can deploy vnext in the same way as we did with kubectl with the service.yaml configuration file, except this time we'll use knctl. By providing knative with the source of our app and the image to push to dockerhub, we'll get an application with a URL we can access.

	```
	knctl deploy \
	    --service fib-knative \
	    --git-url https://github.com/beemarie/fib-knative \
	    --git-revision vnext \
	    --service-account build-bot \
	    --image index.docker.io/beemarie/fib-knative:vnext \
	    --managed-route=false
	```
	This command will tell knative to go out to github, find my code, build it into a container, and push that conatiner to dockerhub. One thing you'll notice is that this deploy command also tags my app versions with a `latest` and a `previous` tag.

2. See the revisions using knctl.

	```
	knctl revisions list
	```


## Canary Testing with Knctl
Maybe we want to slowly roll over from our old version to the new version, or do some AB testing of the new version. We can use the knctl rollout command to route traffic percentages to our revisions.

1. Check the current route percentages:

	```
	knctl route list
	```

2. Send 50% of the traffic to the latest revision, and 50% to the previous revision:

	```
	knctl rollout --route fib-knative -p fib-knative:latest=50% -p fib-knative:previous=50%
	```

3. Check the route percentages to confirm they've updated:

	```
	knctl route list
	```

3. Let's run some load against the app, just asking for the first number in the fibonacci sequence so that we can clearly see which revision is being called.

	```
	while sleep 0.5; do curl -X POST http://fib-knative.default.bmv-knative.us-east.containers.appdomain.cloud/fib" -H 'Content-Type: application/json'   -d '{"number":1}' ; done
	```

4. We should see that the curl requests are routed approximately 50/50 between the two applications. Let's kill this process using ctrl-c.


At this point, you should feel that you've gotten a whirlwind tour of knative and knctl.  Please reach out should you have any questions or issues going through this lab!


TODO:
- remove steps from access clusters
- Kaniko bulid Template
- show more output from knctl & kubectl commands
- add cleanup section at bottom
