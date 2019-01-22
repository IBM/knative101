## Deploy vnext Version Using knctl

Did you notice that the Fibonacci sequence started with 1? Most would argue that the sequence should actually start with 0. There's a vnext version of the application at the vnext branch in the github project. We'll deploy that as v2 of our app, but instead of using kubectl, let's try a new tool.

knctl is a new Knative CLI providing a simple set of commands to interact with a Knative installation. Let's try it out.

### Installing knctl
1. Install knctl using one of the prebuilt binaries from their [release page](https://github.com/cppforlife/knctl/releases), and run the following commands:

  ```
  shasum -a 265 ~/Downloads/knctl-*
  # Compare checksum output to what's included in the release notes

  mv ~/Downloads/knctl-* /usr/local/bin/knctl

  chmod +x /usr/local/bin/knctl
  ```

### Deploy vnext
1. Let's deploy vnext, but instead of kubectl with the service.yaml file, let's use knctl. By providing Knative with the source of our app and the image to push to dockerhub, we'll get an application with a URL we can access.

	```
	knctl deploy \
	    --service fib-knative \
	    --git-url https://github.com/beemarie/fib-knative \
	    --git-revision vnext \
	    --service-account build-bot \
	    --image index.docker.io/beemarie/fib-knative:vnext \
	    --managed-route=false
	```
	This command will tell Knative to go out to github, find my code, build it into a container, and push that container to dockerhub. One thing you'll notice if you follow the output logs is that this deploy command also tags my app versions with a `latest` and a `previous` tag.

2. See the revisions using knctl.

	```
	knctl revisions list
	```

  Output:
  ```
  Revisions

  Service      Name               Tags      Annotations  Conditions  Age  Traffic  
  fib-knative  fib-knative-00002  latest    -            5 OK / 5    20h  100% -> fib-knative.default.bmv-knative.us-east.containers.appdomain.cloud  
  ~            fib-knative-00001  previous  -            5 OK / 5    20h  -

  2 revisions

  Succeeded
  ```
3. Curl the application to see that 100% of the traffic is hitting your new fib-knative revision, starting the sequence with 0.

  ```
  curl -X POST http://fib-knative.default.bmv-knative.us-east.containers.appdomain.cloud/fib -H 'Content-Type: application/json' -d '{"number":3}'
  ```


Continue on to [exercise 5](../exercise-5/README.md).
