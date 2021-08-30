Let's explore some more features of kubernetes.  There are a lot of training courses available on Katacoda [here](https://www.katacoda.com/courses/kubernetes).

But we're going to walk through some kubernetes essentials to get you familiar with debugging and troubleshooting your microservice application.

First off, the way we port forwarded the port to the pod in the last step is not something you would do in a production environment, since pod's come and go throughout the life-cycle of the application.  Rather, we're going to leverage the Kubernetes Service type, which acts as a proxy to one or more of your running pods.

There's already a service configured for this application, but we'll have to make a few tweaks to get it working.  Run `kubectl get svc -owide`{{execute}} to see the running ws service.  Notice the selector column, that's how service knows which pods are associated to this service.  In our case, `app.kubernetes.io/name=ws` is the name applied to each  pod in the application. 

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

Now when you run `kubectl get pods`{{execute}}, you'll see two pods starting, and when they are in the Running state, you can run `curl $NODE_IP:$NODE_PORT/greeting`{{execute}} again, the requests are actually being directed to each of the pods in a round-robin pattern.

Running the describe command on anything is a great way to learn about your running (or not running) pods, services, jobs, etc.  Run `kubectl describe svc/ws`{{execute}} to see the details of the ws service.  Now, export your first pod's name again `export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=ws,app.kubernetes.io/instance=ws" -o jsonpath="{.items[0].metadata.name}")`{{execute}}, then run `kubectl describe pod/$POD_NAME`{{execute}} to see the details of the lifetime of the pod.  This is where you typically start to debug the running pod.

Another way to see what your pod is doing is by inspecting the log files.  Run `kubectl logs $POD_NAME`{{execute}} to see your logs in the pod.

And finally, when you really need to see what's going on within your running pod, you can exec into the pod's runtime with `kubectl exec -it $POD_NAME -- /bin/bash`{{execute}}.

Once you're inside the pod, you can run `curl localhost:8080/greeting`{{execute}} to check if your service is running.  Then, `exit`{{execute}} to get our of the pod's terminal.

That's about it for this scenario, hopefully, this course has given you enough information to help you create cloud native web services deployed to kubernetes on your own. 


