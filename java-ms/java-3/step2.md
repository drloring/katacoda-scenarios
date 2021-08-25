Next we're going to run our installer to check out our deployment in kubernetes.

First, dry run the install with the following command `h3 install ws ws --dry-run`{{execute}}.

This shows you what the final kubernetes yml files will look like prior to installing in the cluster.

When ready, run the installer again, this time with non --dry-run `h3 install ws ws`{{execute}}

Run `kubectl get pods`{{execute}} and ensure that your pod is installed. 


