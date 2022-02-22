In this course, we're going to expand on the work that we previously did with Redis in Kubernetes [here](https://katacoda.com/ng-dloring/courses/java-ms-config/java-2).
Previously, we deployed our Redis cluster with `--noauth`, not requiring a password.  That's not really realistic in production environments, so we're going to see how to create and use secrets in kubernetes to manage our passwords.

In the previous course, we used the Bitnami helm charts to quickly get up to speed.  However, in this course, we're going to re-create some of the features used in the bitnami charts to learn the basics of deploying production charts.

First, though, we have to install `curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}.

Not that we have helm installed, let's create a basic chart with `helm create redis`{{execute}}.  You'll recall that the redis charts reside in the redis directory.  If we open `redis/values.yaml`{{open}}, and change the `repository` from `nginx` to `redis` and tag from `""` to `"latest"`, we'll have a chart that deploys a simple redis database.

Run `helm install redis redis`{{execute}} and `kubectl get all`{{execute}}, we'll see our service and pods running in kubernetes.  If we `kubectl -it exec redis -- /bin/bash`{{execute}}, we can use the CLI to access the running redis service `redis-cli connect localhost`{{execute}}.




