First we have to exit out of the anchore-cli pod with `exit` or `Ctrl+D`, now let's get the web service we created in past courses `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}} and the Dockerfile `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}

Now, we'll build the docker image with `docker build -t java-ws .`{{execute}}

We need to expose the anchore-engine outside of the kubernetes cluster, but first we need the anchore-engine-api pod name `kubectl get po`{{execute}}.  Copy the pod name and run `kubectl expose pod <podname> --type=NodePort`.

Once that's exposed, run the following command to get the list of services`kubectl get svc`{{execute}} and take note of the NodePort address allocated.

Then  `curl -s https://ci-tools.anchore.io/inline_scan-latest | bash -s -- analyze -u admin -p $ANCHORE_CLI_PASS -r http://localhost:<NodePort> -g java-ws:latest`

Once that process finishes, you'll see that the results are put back into the anchore-engine running Kubernetes, so we'll have to exec into the pod to access the Anchore CLI again `kubectl exec -it anchore-cli -- bash`{{execute}}

Once in the pod, you can see the java-ws by running `anchore-cli image list`{{execute}}.

From there, you can perform the analysis on the `java-ws` container just like we did with the redis container, e.g. `anchore-cli evaluate check localbuild/java-ws`{{execute}}.

Notice that we have are failing the security scan, run `anchore-cli evaluate check localbuild/java-ws --detail`{{execute}} to view the specific errors.

UPDATE: inline_scan is deprecated in lieu of syft and grype
To install syft, run `curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin`{{execute}}

To install grype, run `curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin`{{execute}}

These two tools are primarily stateless and don't require anchore-engine to be running.  Let's try it out now, first, run syft and output the SBOM to a json file with `syft packages java-ws:latest -o json > java-ws.json`{{execute}}

Once the SBOM is created, we can run vulnerability checks on it with `grype sbom:./java-ws.json`{{execute}}, to have grype fail (return a 1), just pass in the `--fail-on` flag like `grype sbom:./java-ws.json --fail-on high`{{execute}}.  These two commands allow a much lighter-weight integration into CI/CD pipelines.

Now we'll see how they integrate with anchore-engine.  First we have to export our existing password to syft with `export SYFT_ANCHORE_PASSWORD=$ANCHORE_CLI_PASS`{{execute}}.

Then, run syft again connecting to the server `syft packages java-ws:latest --host http://localhost:<NODE_PORT> -u admin -p $SYFT_ANCHORE_PASSWORD`{{execute}}.

Once that runs, you can exec into the anchore-cli pod again `kubectl exec -it anchore-cli -- bash`{{execute}}, and run `anchore-cli image list`{{execute}} to see the uploaded results.  Take some time to compare those results to the one's performed with the inline_scan tool.

Next we're going to look into customizing security policies and get the image scan to pass.


