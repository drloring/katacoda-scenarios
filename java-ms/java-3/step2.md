Next we're going to run our installer to check out our deployment in kubernetes.

First, dry run the install with the following command `h3 install ws ws --dry-run`{{execute}}.

This shows you what the final kubernetes yml files will look like prior to installing in the cluster.

When ready, run the installer again, this time with non --dry-run `h3 install ws ws`{{execute}}

Run `kubectl get pods`{{execute}} and ensure that your pod is installed. Notice that your pod has a system generated name.

Now, follow the instructions on the screen to export some environment variables so that you can access your pod from the command line.

`export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=ws,app.kubernetes.io/instance=ws" -o jsonpath="{.items[0].metadata.name}")`{{execute}}

and `export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")`{{execute}}

then run `kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT &`{{execute}} to forward the port outside of the cluster.

And finally, you can run `curl http://localhost:8080/greeting`{{execute}} to ensure your service is running in Kubernetes!
