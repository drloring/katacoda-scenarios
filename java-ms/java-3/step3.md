Let's explore some more features of kubernetes.  There are a lot of training courses available on Katacoda [here](https://www.katacoda.com/courses/kubernetes).

But we're going to walk through some kubernetes essentials to get you familiar with debugging and troubleshooting your microservice application.

First off, the way we port forwarded the port to the pod is not something you would do in a production environment, as pod's come and go throughout the life-cycle of the application.  Rather, we're going to leverage the Kubernetes Service type, which acts as a proxy to one or more of your running pods.

Start, by opening the `ws/templates/service.yaml`{{open}} file.  Then change the targetPort to <pre class="file" data-filename="ws/templates/service.yaml" data-target="insert" data-marker="      targetPort: http">      targetPort: 8080</pre> to reflect our pod's exposed port.

Next, we have to change the type of port the service is creating from ClusterIP to NodePort, so that it's available external to the cluster.  ClusterIP is used for service-to-service communications, when you need to access the service externally, NodePort is an option for that.  Open `ws/values.yaml`{{open}} again, and change <pre class="file" data-filename="ws/values.yaml" data-target="insert" data-marker="  type: ClusterIP">  type: NodePort</pre>

Apply the change to the cluster with `h3 upgrade ws ws`{{execute}}.

Following the instructions afterwards, `export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services ws)`{{execute}}

and `export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")`{{execute}}

Now, run `curl $NODE_IP:$NODE_PORT/greeting`{{execute}} to see your web application accessed through the service proxy.

By accessing your service this way, when the pod name or IP changes, those changes are insulated by the service proxy.  Let's kill our pod to demonstrate that.  You can effectively restart your pod by scaling it down to 0 deployments then scaling it back up. Run `kubectl get deployment -owide`{{execute}} to see your deployment.

Next run `kubectl scale deployment ws --replicas=0`{{execute}}

Depending on how fast you can run `kubectl get pods`{{execute}}, you'll either see the pod in the 'Terminating' state or not running at all.

Next, scale it back up to a two pod service `kubectl scale deployment ws --replicas=2`{{execute}}

Now when you run `kubectl get pods`{{execute}}, you'll see two pods starting, and when you run `curl $NODE_IP:$NODE_PORT/greeting`{{execute}} again, the requests are actually being directed to each of the pods in a round-robin pattern.

