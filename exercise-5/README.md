## Deploy vnext Version Using knctl

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

  Output:
  ```
  Revisions

  Service      Name               Tags      Annotations  Conditions  Age  Traffic  
  fib-knative  fib-knative-00002  latest    -            5 OK / 5    20h  100% -> fib-knative.default.bmv-knative.us-east.containers.appdomain.cloud  
  ~            fib-knative-00001  previous  -            5 OK / 5    20h  0%

  2 revisions

  Succeeded
```
