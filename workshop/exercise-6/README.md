## Knative from the Kubernetes Layer
Up until this point, we've used the Knative CLI to deploy, update, and interact with our application. While our application is running on Kubernetes, we haven't had to interact with any of the underlying Kubernetes components. If we did want to control or view the application at this layer of the stack, we certainly could.

Knative defines some objects for each component as Kubernetes [Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources)(CRDs). A CRD is used to define a new resource type in Kubernetes. Knative [Serving](https://github.com/knative/docs/tree/master/docs/serving#serving-resources) includes a number of Custom Resource Definitions, including Service, Route, Configuration, and Revision.

Because Knative is built on top of Kubernetes, we can access all of these resource types from the Kubernetes layer in the stack.

### Explore Application Deployment Using Kubectl
1. Our application may have already scaled down to zero. Let's curl the application, and then view the pods our application is running on.

    ```
    curl $MY_APP_URL/4
    ```
    ```
    kubectl get pods --watch
    ```

    You should see some pods to indicate that your application is running. You may see pods for both versions of your application (-zero and -one). Wait about 90 seconds, and you should see that your application scales back down to zero and the pods terminate due to lack of use. You should eventually see the pods move from the `Running` to the `Terminating` state.

    Example Output:

    ```
    NAME                                                    READY   STATUS      RESTARTS   AGE
    fib-knative-one-deployment-79d6cb9cbd-v4th5             2/2     Running     0          43s
    fib-knative-zero-deployment-968c8896-8mhkx              2/2     Running     0          60s
    fib-knative-one-deployment-79d6cb9cbd-v4th5             2/2     Terminating   0          86s
    fib-knative-one-deployment-79d6cb9cbd-v4th5             0/2     Terminating   0          96s
    fib-knative-zero-deployment-968c8896-8mhkx              0/2     Terminating   0          96s
    ```
   
    Note: To exit the watch, use `ctrl + c`.

2. We can also view some details about the Configuration for our Service. Every Service has a Configuration and a Route -- each time a Configuration is updated, a new Revision is created. Let's see a list of Configurations, first.

    ```
    kubectl get configuration
    ```

    Example Output:
    ```
    NAME                LATESTCREATED             LATESTREADY               READY   REASON
    fib-knative         fib-knative-zero          fib-knative-zero          True    
    ```

3. Next, let's see some of the details of the fib-knative Configuration.    

    ```
    kubectl get configuration fib-knative -o yaml
    ```

    Example Output:
    ```
    apiVersion: serving.knative.dev/v1
    kind: Configuration
    metadata:
      annotations:
        serving.knative.dev/creator: IAM#beemarie@us.ibm.com
        serving.knative.dev/lastModifier: IAM#beemarie@us.ibm.com
      creationTimestamp: "2020-05-18T18:21:10Z"
      generation: 3
      labels:
        serving.knative.dev/route: fib-knative
        serving.knative.dev/service: fib-knative
      name: fib-knative
      namespace: default
      ownerReferences:
      - apiVersion: serving.knative.dev/v1alpha1
        blockOwnerDeletion: true
        controller: true
        kind: Service
        name: fib-knative
        uid: 8fb26d78-1e95-4140-9a11-d0ca288011e7
      resourceVersion: "1123399"
      selfLink: /apis/serving.knative.dev/v1/namespaces/default/configurations/fib-knative
      uid: d4e043ac-439c-4dc4-90e5-e71d88d1b4ea
    spec:
      template:
        metadata:
          annotations:
            client.knative.dev/user-image: docker.io/ibmcom/fib-knative:vnext
          creationTimestamp: null
          name: fib-knative-zero
        spec:
          containerConcurrency: 0
          containers:
          - image: docker.io/ibmcom/fib-knative:vnext
            name: user-container
            readinessProbe:
              successThreshold: 1
              tcpSocket:
                port: 0
            resources: {}
          timeoutSeconds: 300
    status:
      conditions:
      - lastTransitionTime: "2020-05-18T18:38:33Z"
        status: "True"
        type: Ready
      latestCreatedRevisionName: fib-knative-zero
      latestReadyRevisionName: fib-knative-zero
      observedGeneration: 3
    ```
    
    The Configuration shows the desired state for our application. You can see the image that our Service is using, as well as the namespace the Service is in, the status of the Service, and some other configuration information.

4. When creating your Service, some other objects were created in Kubernetes as well, such as Routes or Revisions. Let's check out the Route. In the `traffic` section you can see the two URLs that were created when we tagged the revisions, as well as the percentage of traffic going to each Revision.

    ```
    kubectl get route fib-knative -o yaml
    ```

    Example Output:
    ```
    ...
      traffic:
      - latestRevision: false
        percent: 10
        revisionName: fib-knative-zero
        tag: zero
        url: http://zero-fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud
      - latestRevision: false
        percent: 90
        revisionName: fib-knative-one
        tag: one
        url: http://one-fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud
      url: http://fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud
      ```
5. We can also see a list of Revisions created in Kubernetes:

    ```
    kubectl get revision
    ```
    
    Example Output:
    ```
    NAME                      CONFIG NAME         K8S SERVICE NAME          GENERATION   READY   REASON
    fib-knative-one           fib-knative         fib-knative-one           2            True    
    fib-knative-ywvgm-1       fib-knative         fib-knative-ywvgm-1       1            True    
    fib-knative-zero          fib-knative         fib-knative-zero          3            True    
    ```

6. In the next section, we'll be deploying the same application, but with a different method, so let's clean up our service.

    ```
    kn service delete fib-knative
    ```

Congratulations! You've completed the first part of the Knative 101 lab. You have had a quick overview of deploying applications to Knative using the `kn` CLI as well as using `kubectl` to do some advanced debugging and exploration. You can continue to the advanced portion of this lab with [exercise 7](../exercise-7/README.md) or you can go to the [Knative Eventing Lab](https://github.com/IBM/knative101-eventing) to learn more about the eventing component.