First we have to exit out of the anchore-cli pod with `exit` or `Ctrl+D`, now let's get the web service we created in past courses `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}} and the Dockerfile `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}

Now, we'll build the docker image with `docker build -t java-ws .`{{execute}}

We need to expose the anchore-engine outside of the kubernetes cluster, but first we need the anchore-engine-api pod name `kubectl get po`{{execute}}.  Copy the pod name and run `kubectl expose pod <podname> --type=NodePort`.

Once that's exposed, run the following command to get the list of services`kubectl get svc`{{execute}} and take note of the NodePort address allocated.

Then  `curl -s https://ci-tools.anchore.io/inline_scan-latest | bash -s -- analyze -u admin -p $ANCHORE_CLI_PASS -r http://localhost:<NodePort> -g java-ws:latest`

Once that process finishes, you'll see that the results are put back into the anchore-engine running Kubernetes, so we'll have to exec into the pod to access the Anchore CLI again `kubectl exec -it anchore-cli -- bash`{{execute}}

Once in the pod, you can see the java-ws by running `anchore-cli image list`{{execute}}.

From there, you can perform the analysis on the `java-ws` container just like we did with the redis container, e.g. `anchore-cli evaluate check localbuild/java-ws`{{execute}}.

Notice that we have are failing the security scan, run `anchore-cli evaluate check localbuild/java-ws --detail`{{execute}} to view the specific errors.

Next we're going to look into customizing security policies.


