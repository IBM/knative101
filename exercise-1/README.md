## Install Istio, Knative, and Kaniko Build Template

Knative is currently built on top of both Kubernetes and Istio. You will need to install Istio.

1. Install Istio:

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.2/istio.yaml
	```
2. Label the default namespace with `istio-injection=enabled`:

	```
	kubectl label namespace default istio-injection=enabled
	```

3. Monitor the Istio components until all of the components show a `STATUS` of
    `Running` or `Completed`:

    ```
    kubectl get pods --namespace istio-system --watch
    ```

After installing Istio, you can install Knative. For this lab, we will install both the Build & Serving components of Knative.

1. Install Knative:

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.2/release.yaml
	```

2. Monitor the Knative components until all of the components show a `STATUS` of `Running` or `Completed`:

	```
	kubectl get pods --namespace knative-serving
	kubectl get pods --namespace knative-build
  ```

As a part of this lab, we will use the Kaniko Build Template for building source into a container image from a Dockerfile, inside a container or a Kubernetes cluster. Typically, to build a container image, it is required to run a Docker daemon with root access. According to the [Kaniko github project](https://github.com/GoogleContainerTools/kaniko), "Kaniko doesn't depend on a Docker daemon and executes each command within a Dockerfile completely in userspace. This enables building container images in environments that can't easily or securely run a Docker daemon, such as a standard Kubernetes cluster."

1. Install the Kaniko Build Template to your cluster.

  ```
  kubectl apply --filename https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml
  ```
