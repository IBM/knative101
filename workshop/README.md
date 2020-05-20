# Knative 101

In this lab, you'll learn about Knative, a new open source collaboration from IBM, Google, Pivotal, Red Hat, Cisco, and others. Knative is based on the Kubernetes platform and is used for building, deploying, and managing serverless workloads. It enables developers to focus on what really matters by abstracting away many of the underlying details of building, deploying, and managing an application. With Knative, you can serve your applications, manage traffic, autoscale, and react to events. 

As you work through this lab, you'll create a new cluster on IBM Kubernetes Service (IKS), install Knative on that cluster, and then deploy a Node.js Fibonacci application using Knative. 

We'll use the Knative client, `kn`, which makes interacting with Knative simple and straightforward. We will also explore Tekton Pipelines, a project providing kubernetes-style resources for declaring CI/CD pipelines.

The Knative application you'll create is a Fibonacci sequence app. When provided with the number n, it will return the first n numbers of the Fibonacci sequence: 1, 1, 2, 3.... You'll also deploy a vnext of the application, which starts the Fibonacci sequence with 0 instead of 1: 0, 1, 1, 2, 3.... The application will be given a URL, which you'll be able to curl to get the Fibonacci results. We'll also explore how to route varying percentages of your incoming traffic to each of these versions of the application.


Get started with the [workshop](./exercise-0/README.md).
