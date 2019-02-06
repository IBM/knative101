## Setup: Install Istio and Knative on Your Cluster

Knative is currently built on top of both Kubernetes and Istio. You will need to install Istio to install Knative. If you want to learn more, you can check out the labs [Kube101](https://github.com/IBM/kube101/tree/master/workshop) and [Istio101](https://github.com/IBM/istio101/tree/master/workshop).

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

1. Install Knative Serving & Build components:

    ```
    kubectl apply --filename https://github.com/knative/serving/releases/download/v0.3.0/serving.yaml \
    --filename https://github.com/knative/build/releases/download/v0.3.0/release.yaml
    ```

2. Monitor the Knative components until all of the components show a `STATUS` of `Running` or `Completed`:

	Commands:
    ```
    kubectl get pods --namespace knative-serving
    kubectl get pods --namespace knative-build
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


Continue on to [exercise 4](../exercise-4/README.md).
