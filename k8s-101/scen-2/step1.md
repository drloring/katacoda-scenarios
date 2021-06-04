Kubectl is the command line interface to interact with your Kubernetes cluster.

Type `kubectl get nodes` at the terminal prompt for click `kubectl get nodes`{{execute}} to send it to your terminal.

Initially, you'll see

`The connection to the server 172.17.0.15:8443 was refused - did you specify the right host or port?` messages.  That means Kubernetes is starting up.

Eventually, you should see something like:

`NAME       STATUS   ROLES    AGE   VERSION
minikube   NotReady    master   12m   v1.17.3`

If the status is NotReady, keep retrying the command until it becomes ready.

Kubernetes Pods are the application runtimes for your containers in the cluster.

Next let's see if any pod are running.

Type `kubectl get po` or `kubectl get pods` at the terminal prompt for click `kubectl get po`{{execute}} to send it to your terminal.

Notice there are no pods in the default namespace.  

Namespaces are the way clusters are organized in kubernetes, the default namespace if primarily for the kubernetes control plane.
