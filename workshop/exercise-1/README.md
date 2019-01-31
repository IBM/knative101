## Install Istio, Knative, and Kaniko Build Template

Knative is currently built on top of both Kubernetes and Istio. You will need to install Istio to install Knative. It's not required for this lab, but you can learn more about Istio by completing the [Istio 101 lab](https://github.com/IBM/istio101/tree/master/workshop).

### Install Istio

1. A Custom Resource Definition enables you to create custom resources, extensions to the Kubernetes API on your Kubernetes cluster. Istio needs these CRDs to be created before we can install.  Install Istio’s CRDs via kubectl apply, and wait a few seconds for the CRDs to be committed in the kube-apiserver.

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.3.0/istio-crds.yaml
	```

2. Install Istio:

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.3.0/istio.yaml
	```
3. Label the default namespace with `istio-injection=enabled`:

	```
	kubectl label namespace default istio-injection=enabled
	```

4. Monitor the Istio components until all of the components show a `STATUS` of
    `Running` or `Completed`:

    ```
    kubectl get pods --namespace istio-system --watch
    ```

### Install Knative

After installing Istio, you can install Knative. For this lab, we will install all available Knative components.

1. Install Knative:

    ```
    kubectl apply --filename https://github.com/knative/serving/releases/download/v0.3.0/serving.yaml \
    --filename https://github.com/knative/build/releases/download/v0.3.0/release.yaml \
    --filename https://github.com/knative/eventing/releases/download/v0.3.0/release.yaml \
    --filename https://github.com/knative/eventing-sources/releases/download/v0.3.0/release.yaml \
    --filename https://github.com/knative/serving/releases/download/v0.3.0/monitoring.yaml
    ```

2. Monitor the Knative components until all of the components show a `STATUS` of `Running` or `Completed`:

	Commands:
    ```
    kubectl get pods --namespace knative-serving
    kubectl get pods --namespace knative-build
    kubectl get pods --namespace knative-eventing
    kubectl get pods --namespace knative-sources
    kubectl get pods --namespace knative-monitoring
    ```
    Example Ouput:

    ```
    NAME                          READY   STATUS    RESTARTS   AGE
    activator-df78cb6f9-jpvs7     2/2     Running   0          38s
    activator-df78cb6f9-nhzhf     2/2     Running   0          37s
    activator-df78cb6f9-qjg8w     2/2     Running   0          37s
    autoscaler-6fccb66768-m4f2q   2/2     Running   0          37s
    controller-56cf5965f5-8pwcg   1/1     Running   0          35s
    webhook-5dcbf967cd-lxzmk      1/1     Running   0          35s
    ```

    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    build-controller-747b8fd966-4n8b2   1/1     Running   0          47s
    build-webhook-6dc78d8f6d-gsm4k      1/1     Running   0          47s
    ```

### Install Kaniko Build Template

As a part of this lab, we will use the kaniko build template for building source into a container image from a Dockerfile, inside a container or a Kubernetes cluster. Typically, to build a container image, it is required to run a Docker daemon with root access. According to the [Kaniko github project](https://github.com/GoogleContainerTools/kaniko), "Kaniko doesn't depend on a Docker daemon and executes each command within a Dockerfile completely in userspace. This enables building container images in environments that can't easily or securely run a Docker daemon, such as a standard Kubernetes cluster."


1. Install the kaniko build template to your cluster.

    ```
    kubectl apply --filename https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml
    ```

2. Use kubectl to confirm you installed the kaniko build template, as well as to see some more details about it.  You'll see that this build template accepts parameters of `IMAGE` and `DOCKERFILE`.  `IMAGE` is the name of the image you will push to the container registry, and `DOCKERFILE` is the path to the Dockerfile that will be built.

	Command:
	```
	kubectl get BuildTemplate kaniko -o yaml
	```

	Example Output:
	```yaml
      spec:
        generation: 1
        parameters:
        - description: The name of the image to push
          name: IMAGE
        - default: /workspace/Dockerfile
          description: Path to the Dockerfile to build.
          name: DOCKERFILE
        steps:
        - args:
          - --dockerfile=${DOCKERFILE}
          - --destination=${IMAGE}
          image: gcr.io/kaniko-project/executor
          name: build-and-push
	```


Continue on to [exercise 2](../exercise-2/README.md).
