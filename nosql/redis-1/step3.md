This simplest way to try to scale out is to make more replicas.  Let's try that now `kubectl scale --replicas=3 deployment/redis`{{execute}}.  Now if we look at the pods, we'll have three pods running.  However, lets try modifying one of the pods data and see whether it is shared with the other pods.

Let's exec into the first pod with `kubectl -it exec $(kubectl get po -o jsonpath="{.items[0].metadata.name}") -- redis-cli -a $REDIS_PASS`{{execute}}, and `set this other`{{execute}}.  Remember to exit with `Ctrl+D`, then exec into the second pod with `kubectl -it exec $(kubectl get po -o jsonpath="{.items[1].metadata.name}") -- redis-cli -a $REDIS_PASS`{{execute}} (notice the index changed to 1 in the command).  Now if we run `get this`{{execute}}, we see it's still set to "that".  So, while they're sharing the data storage location, they're not sharing updates.

The easiest way to get a clustered redis is using the bitnami helm charts located [here](https://github.com/bitnami/charts/tree/master/bitnami/redis).  Following the instructions on that page, we would run `helm repo add bitnami https://charts.bitnami.com/bitnami`{{execute}} then `helm install my-release bitnami/redis`{{execute}}.

The default configuration contains a single master and three replicas, run `kubectl get pods`{{execute}} to confirm.

Following the instructions after the deployment, we export the `REDIS_PASSWORD` with `export REDIS_PASSWORD=$(kubectl get secret --namespace default my-release-redis -o jsonpath="{.data.redis-password}" | base64 --decode)`{{execute}}, then we can exec into the pods and save some data.  

First, get the pod names with `kubectl get pods`{{execute}}, then we `kubectl exec -it <MASTER_POD_NAME> -- redis-cli -a $REDIS_PASSWORD` replacing the master pod name with yours.  Once in, we `set this that`{{execute}} and exit with `Ctrl+D`.  Now, if we exec into any of the replicas by replacing the previous `kubectl` command with the replicas pod name and run `get this`{{execute}}, we see the data was replicated.

What if we try by modifying data on the replica, however, we'll get an error `(error) READONLY You can't write against a read only replica.`.  So the cluster is configured as a read-repliced cluster, if the master goes down, while kubernetes is bringing the pod back up, the cluster will be not available.  Test it by deleting the master pod and ensuring that the data is retained.

If we wanted to run an active-active cluster configuration, we'll need a different set of helm charts for that.  First, let's delete our redis cluster with `helm uninstall my-release`{{execute}}.

Once those resources have been removed from kubernetes, we'll run the Redis Cluster helm charts with `helm install my-release bitnami/redis-cluster`{{execute}}.  Running `kubectl get pods`{{execute}} reveals that six pods are now being created.  In this configuration, there are three master and three replicated master pods in the system.

We re-export our `REDIS_PASSWORD` with `export REDIS_PASSWORD=$(kubectl get secret --namespace "default" my-release-redis-cluster -o jsonpath="{.data.redis-password}" | base64 --decode)`{{execute}}, then `kubectl exec -it my-release-redis-cluster-0 -- redis-cli -a $REDIS_PASSWORD`{{execute}}.  In this configuration, all other pods are prohibited from executing queries.

In this course, we've learned how to install and configure a redis server to support authentication, durability and scalability with clustering in a kubernetes cluster.  I hope you find this information useful and get the chance to apply it to a project in the near future.
