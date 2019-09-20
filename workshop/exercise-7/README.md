## (OPTIONAL, FOR ADVANCED USERS) Build and Deploy our Knative Application

We've seen how to deploy an application from an image that already exists, so let's build the image ourselves from source code. We'll be using Tekton Pipelines, a project providing Kubernetes-style resources for declaring CI/CD pipelines. We'll fetch our source code from a github repository, build a container image from it, push it to a private container image registry on IBM Cloud, and then ultimately get a URL to access our application.

### Tekton Components Overview

First, let's understand a few of the Tekton components we will be using in this exercise. A `Task` defines work that needs to be executed, and is made up of a series of `steps` that will be executed sequentially.

A `TaskRun` will run the `Task` that you defined. You can then view output from the `TaskRun`.

The main goal of Tekton Pipelines is to run your `Task` individually or as a part of a `Pipeline`. Each `Task` runs as a Pod on your Kubernetes cluster with each step as its own container.

A `PipelineResource` is used to define artifacts that are passed into and out of a `Task`. There are a few sytem defined resource types ready to use, such as `git` or `image` resources.  

### Create the Required PipelineResources

1. Take a look at the `git-resource.yaml` file. 
    ```
    apiVersion: tekton.dev/v1alpha1
    kind: PipelineResource
    metadata:
      name: fib-knative
    spec:
      type: git
      params:
        - name: revision
          value: master
        - name: url
          value: https://github.com/IBM/fib-knative
    ```
    You can see that this file is defining the git resource we will be building. It includes the git url and revision for our code.

2. Apply this yaml file to your kubernetes cluster.
    ```
    kubectl apply -f git-resource.yaml
    ```

3. Take a look at the `image-resource.yaml` file.
    ```
    apiVersion: tekton.dev/v1alpha1
    kind: PipelineResource
    metadata:
      name: fib-knative
    spec:
      type: image
      params:
        - name: url
          value: us.icr.io/<NAMESPACE>/fib-knative
    ```
    This file defines the location where our built image will be stored. In our case, we'll be storing it on the IBM Cloud Container Registry. 

4. Make sure you update the image value with your own container registry namespace, which you defined in exercise 6. Replace <NAMESPACE> with your own namespace.

5. Apply this yaml file to your kubernetes cluster.
    ```
    kubectl apply -f image-resource.yaml
    ```

### Build Image From Source and Push to Container Registry

1. Take a look at the `image-from-source-task.yaml` file. This `Task` has inputs and outputs. The input resource is the github repository we just defined, and the output is the image that will be produced from that source. You'll notice that the build-and-push step of this `Task` uses Kaniko to build the source into a container image from a Dockerfile. Kaniko doesn't depend on a Docker engine and instead executes each command within the Dockerfile completely in userspace. This enables building container images in environments that can't easily or securely run a Docker engine, such as Kubernetes.

    ```
    apiVersion: tekton.dev/v1alpha1
    kind: Task
    metadata:
      name: build-docker-image-from-git-source
    spec:
      inputs:
        resources:
          - name: docker-source
            type: git
        params:
          - name: pathToDockerFile
            type: string
            description: The path to the dockerfile to build
            default: /workspace/docker-source/Dockerfile
          - name: pathToContext
            type: string
            description:
              The build context used by Kaniko
              (https://github.com/GoogleContainerTools/kaniko#kaniko-build-contexts)
            default: /workspace/docker-source
      outputs:
        resources:
          - name: builtImage
            type: image
      steps:
        - name: build-and-push
          image: gcr.io/kaniko-project/executor:v0.9.0
          # specifying DOCKER_CONFIG is required to allow kaniko to detect docker credential
          env:
            - name: "DOCKER_CONFIG"
              value: "/builder/home/.docker/"
          command:
            - /kaniko/executor
          args:
            - --dockerfile=${inputs.params.pathToDockerFile}
            - --destination=${outputs.resources.builtImage.url}
            - --context=${inputs.params.pathToContext}
    ```

2. Apply this yaml file to your kubernetes cluster.
    ```
    kubectl apply -f image-from-source-task.yaml
    ```
  
3. Now that our `Task` is defined, let's define a `TaskRun` that will run the `Task`. This `TaskRun` will bind inputs and outputs to our earlier defined resources, `fib-knative` and `fib-knative-git`. This `TaskRun` will also execute the `steps` we defined for our `Task`. Take a look at this file, called `image-from-source-task-run.yaml`.

    ```
    apiVersion: tekton.dev/v1alpha1
    kind: TaskRun
    metadata:
      name: build-docker-image-from-git-source-task-run
    spec:
      serviceAccount: build-bot
      taskRef:
        name: build-docker-image-from-git-source
      inputs:
        resources:
          - name: docker-source
            resourceRef:
              name: fib-knative-git
        params:
          - name: pathToDockerFile
            value: Dockerfile
          - name: pathToContext
            value: /workspace/docker-source/
      outputs:
        resources:
          - name: builtImage
            resourceRef:
              name: fib-knative
    ```

4. Apply this yaml file to your kubernetes cluster.
    ```
    kubectl apply -f image-from-source-task-run.yaml
    ```

5. To see the output of the `TaskRun`, you can use the following command:
    ```
    kubectl get taskruns/build-docker-image-from-git-source-task-run -o yaml
    ```

    You should see that the steps executed successfully. Congratulations! At this point, you've successfully used Tekton Pipelines to build a docker image from some source code on github. Next, let's deploy that docker image to knative!

### Deploy the Fibonacci App Using kubectl and service.yaml

1. Edit the `service.yaml` file to point to your own container registry namespace by replacing the instance of `<NAMESPACE>` with the container registry namespace you created earlier. 

2. Apply the `service.yaml` file to your cluster.

  ```
  kubectl apply -f service.yaml
  ```
3. Run `kubectl get pods --watch` to see the pods initializing. Note: To exit the watch, use `ctrl + c`.

4. Take a look at the `service.yaml` file again:

  ```
  cat service.yaml
  ```
  You should see values for the serviceAccount that you created earlier, as well as the image that we're deploying to knative. This is the image you just built using Tekton.

5. This application has a different name, and will have a different domain. Let's get that URL now.

  ```
  kn service describe fib-knative-built
  ```

6. The route should begin with `fib-knative-built`, and look something like `fib-knative-built-default.bmv-knative-lab.us-south.containers.appdomain.cloud`. Save that in an environment variable now:

  ```
  export MY_BUILT_DOMAIN=fib-knative-built....
  ```

7. Now that the app is up, we should be able to call it using a number input. We can do that using a curl command against the URL provided to us. Ensure you've updated the command with your own ingress subdomain.

  ```
  curl $MY_BUILT_DOMAIN/20
  ```
6. You should see the first 20 Fibonacci numbers!

7. If we left this application alone for some time, it would scale itself back down to 0, and terminate the pods that were created. Run `kubectl get pods --watch` and wait until you see the application scale itself back down to 0. When the application is no longer in use, you should eventually see the pods move from the `Running` to the `Terminating` state. Note: To exit the watch, use `ctrl + c`.

  Expected Output:
  ```
  NAME                                            READY   STATUS      RESTARTS   AGE
  fib-knative-00002-deployment-58dcbdb97c-rrnzc   3/3     Running     0          56s
  fib-knative-00002-deployment-58dcbdb97c-rrnzc   3/3   Terminating   0          89s
  fib-knative-00002-deployment-58dcbdb97c-rrnzc   0/3   Terminating   0          91s
  ```
