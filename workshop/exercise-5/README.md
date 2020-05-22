## Tag revisions to generate custom app URLs

What if we wanted to create URLs that are specific for each of the two revisions we created earlier? Maybe one of the revisions is for staging, and one is for production. You can automatically create a URL for a specific revision by using the --tag option.

### Tag revisions
1. Let's tag both of our revisions. We'll tag `fib-knative-zero` as `zero`, and `fib-knative-one` as `one`.

    ```
    kn service update fib-knative --tag fib-knative-zero=zero --tag fib-knative-one=one
    ```

2. When we tagged the revisions, Knative should have created two new URLs, one for each of the revisions. The new URLs will be the same format as your old URL, but with `zero-` or `one-` prepended before the service name in the URL. Let's get the old URL now, and then build the new URLs.

    ```
    echo $MY_APP_URL
    ```

    Your old URL will look something like this:
    ```
    http://fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud
    ```

    Add `zero-` or `one-` before the service name to get your new URLs, and then save these as environment variables. The new URLs will look something like this:
    ```
    export ZERO_URL=http://zero-fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud
    ```
    ```
    export ONE_URL=http://one-fib-knative-default.bmv-dev-16-5290c8c8e5797924dc1ad5d1b85b37c0-0000.us-south.containers.appdomain.cloud
    ```


3. Let's try out our URLs to confirm that we're getting the expected results! 

    Try the URL for version 1 of the app:
    ```
    curl $ZERO_URL/4
    ```
    
    Example output:
    ```
    [0,1,1,2]
    ```

    Try the URL for version 2 of the app:
    ```
    curl $ONE_URL/4
    ```
    
    Example output:
    ```
    [1,1,2,3]
    ```


Congratulations! You've tagged each revision of your fib-knative application resulting in customized URLs for each version of our application. Continue on to [exercise 6](../exercise-6/README.md).